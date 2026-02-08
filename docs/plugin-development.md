# わんこめプラグイン開発ガイド

わんこめ（OneComme）のプラグインを開発するための包括的なガイドです。

## 📋 目次

- [プラグインとは](#プラグインとは)
- [開発環境](#開発環境)
- [プラグインの基本構造](#プラグインの基本構造)
- [API リファレンス](#api-リファレンス)
- [実装例: waiting-count プラグイン](#実装例-waiting-count-プラグイン)
- [デバッグ方法](#デバッグ方法)
- [配布とインストール](#配布とインストール)
- [ベストプラクティス](#ベストプラクティス)

---

## プラグインとは

わんこめプラグインは、わんこめアプリの機能を拡張する Node.js モジュールです。

### できること

- わんこめのデータファイル（waiting.txt 等）を監視・処理
- 外部ファイルへの出力（OBS 連携等）
- カスタムロジックの実装（統計、分析、通知等）

### できないこと

- わんこめ UI の直接操作
- コメント表示のリアルタイム変更（CSS テンプレートを使用）
- ネットワーク越しのリモート制御（セキュリティ制限）

---

## 開発環境

### 必要なもの

- **Node.js**: 14.0.0 以上
- **わんこめ**: プラグイン機能対応版
- **テキストエディタ**: VS Code 推奨

### ディレクトリ構造

```
~/Library/Application Support/onecomme/
├── plugins/                    # プラグインディレクトリ
│   └── your-plugin/            # あなたのプラグイン
│       ├── plugin.js           # プラグイン本体（必須）
│       ├── package.json        # パッケージ情報（推奨）
│       └── README.md           # 説明（推奨）
└── waiting.txt                 # わんこめデータファイル（例）
```

---

## プラグインの基本構造

### 最小構成

**plugin.js**:

```javascript
module.exports = {
  // 必須: プラグイン名
  name: "My Plugin",

  // 必須: 一意なID（逆ドメイン形式推奨）
  uid: "com.example.my-plugin",

  // 必須: バージョン
  version: "1.0.0",

  // オプション: 作者名
  author: "Your Name",

  // オプション: 必要な権限（配列）
  permissions: [],

  // オプション: 初期状態
  defaultState: {},

  /**
   * 必須: 初期化処理
   * わんこめ起動時に1回だけ呼ばれる
   */
  init({ dir, store }) {
    console.log("[MyPlugin] プラグイン起動");
    // ここに初期化処理を書く
  },

  /**
   * オプション: 終了処理
   * わんこめ終了時に呼ばれる
   */
  destroy() {
    console.log("[MyPlugin] プラグイン終了");
    // リソースのクリーンアップ
  },
};
```

### package.json（推奨）

```json
{
  "name": "onecomme-my-plugin",
  "version": "1.0.0",
  "description": "My OneComme Plugin",
  "main": "plugin.js",
  "keywords": ["onecomme", "わんこめ", "plugin"],
  "author": "Your Name",
  "license": "MIT",
  "engines": {
    "node": ">=14.0.0"
  }
}
```

---

## API リファレンス

### プラグインオブジェクト

プラグインは以下のプロパティとメソッドを持つオブジェクトとして定義します。

#### 必須プロパティ

| プロパティ | 型 | 説明 |
|------------|-----|------|
| `name` | `string` | プラグインの名前 |
| `uid` | `string` | 一意な識別子（逆ドメイン形式推奨） |
| `version` | `string` | バージョン（セマンティックバージョニング推奨） |

#### オプションプロパティ

| プロパティ | 型 | 説明 |
|------------|-----|------|
| `author` | `string` | 作者名 |
| `permissions` | `string[]` | 必要な権限（現在は空配列） |
| `defaultState` | `object` | プラグインの初期状態 |

#### ライフサイクルメソッド

##### `init({ dir, store })`

プラグイン初期化時に呼ばれます（わんこめ起動時に1回）。

**引数**:
- `dir` (string): プラグインディレクトリの絶対パス
- `store` (object): わんこめの状態管理オブジェクト

**用途**:
- ファイル監視の開始
- 初期データの読み込み
- リソースの初期化

**例**:

```javascript
init({ dir, store }) {
  this.store = store;
  console.log(`[MyPlugin] プラグインディレクトリ: ${dir}`);

  // ファイル監視開始
  this.startWatching();
}
```

##### `destroy()`

プラグイン終了時に呼ばれます（わんこめ終了時）。

**用途**:
- ファイル監視の停止
- タイマーのクリア
- 一時ファイルの削除

**例**:

```javascript
destroy() {
  if (this.watcher) {
    this.watcher.close();
    this.watcher = null;
  }
  console.log("[MyPlugin] クリーンアップ完了");
}
```

---

## 実装例: waiting-count プラグイン

参加型機能の人数を監視して OBS 用のテキストファイルに出力するプラグインです。

### 完全なコード

```javascript
const fs = require("fs");
const path = require("path");
const os = require("os");

module.exports = {
  name: "わんこめ参加人数表示プラグイン",
  uid: "tokyo.vtube-tools.waiting-count",
  version: "1.0.0",
  author: "VTube Tools",
  permissions: [],
  defaultState: {},

  // 設定（カスタマイズ可能）
  config: {
    prefix: "参加者: ",
    suffix: "人",
    outputFileName: "waiting_count.txt",
    debounceMs: 100,
  },

  // 内部状態
  state: {
    watcher: null,
    currentCount: 0,
    debounceTimer: null,
    waitingFilePath: null,
    outputFilePath: null,
  },

  /**
   * プラグイン初期化
   */
  init({ dir, store }) {
    this.store = store;

    // ファイルパスの設定
    const onecommeDataDir = path.join(
      os.homedir(),
      "Library",
      "Application Support",
      "onecomme"
    );

    this.state.waitingFilePath = path.join(onecommeDataDir, "waiting.txt");
    this.state.outputFilePath = path.join(
      onecommeDataDir,
      this.config.outputFileName
    );

    console.log("[WaitingCount] プラグイン起動");
    console.log(`[WaitingCount] 監視対象: ${this.state.waitingFilePath}`);
    console.log(`[WaitingCount] 出力先: ${this.state.outputFilePath}`);

    // 初回読み込み
    this.updateCount();

    // ファイル監視開始
    this.startWatching();
  },

  /**
   * プラグイン終了処理
   */
  destroy() {
    if (this.state.watcher) {
      this.state.watcher.close();
      this.state.watcher = null;
      console.log("[WaitingCount] ファイル監視停止");
    }

    if (this.state.debounceTimer) {
      clearTimeout(this.state.debounceTimer);
      this.state.debounceTimer = null;
    }

    console.log("[WaitingCount] プラグイン終了");
  },

  /**
   * ファイル監視開始
   */
  startWatching() {
    try {
      this.state.watcher = fs.watch(
        this.state.waitingFilePath,
        { persistent: true },
        (eventType) => {
          if (eventType === "change") {
            this.debouncedUpdate();
          }
        }
      );

      this.state.watcher.on("error", (error) => {
        console.error("[WaitingCount] ファイル監視エラー:", error.message);
      });

      console.log("[WaitingCount] ファイル監視開始");
    } catch (error) {
      console.error("[WaitingCount] ファイル監視開始失敗:", error.message);
    }
  },

  /**
   * デバウンス処理付き更新
   */
  debouncedUpdate() {
    if (this.state.debounceTimer) {
      clearTimeout(this.state.debounceTimer);
    }

    this.state.debounceTimer = setTimeout(() => {
      this.updateCount();
      this.state.debounceTimer = null;
    }, this.config.debounceMs);
  },

  /**
   * waiting.txt を読み込んで人数をカウント
   */
  updateCount() {
    try {
      if (!fs.existsSync(this.state.waitingFilePath)) {
        this.writeCount(0);
        return;
      }

      const content = fs.readFileSync(this.state.waitingFilePath, "utf-8");
      const firstLine = content.split("\n")[0] || "";

      if (!firstLine.trim()) {
        this.writeCount(0);
        return;
      }

      const count = this.parseWaitingLine(firstLine);
      console.log(`[WaitingCount] 参加人数: ${count}人`);
      this.writeCount(count);
    } catch (error) {
      console.error("[WaitingCount] ファイル読み込みエラー:", error.message);
    }
  },

  /**
   * waiting.txt の1行をパースして人数を取得
   */
  parseWaitingLine(line) {
    const parts = line.trim().split(/\s+/);
    const numbers = [];

    for (const part of parts) {
      const match = part.match(/^(\d+):/);
      if (match) {
        const num = parseInt(match[1], 10);
        if (!isNaN(num)) {
          numbers.push(num);
        }
      }
    }

    return numbers.length > 0 ? Math.max(...numbers) : 0;
  },

  /**
   * 人数を出力ファイルに書き込み
   */
  writeCount(count) {
    if (count === this.state.currentCount) {
      return;
    }

    const outputText = `${this.config.prefix}${count}${this.config.suffix}`;

    try {
      // アトミック書き込み（tmpファイル→rename）
      const tmpFilePath = `${this.state.outputFilePath}.tmp`;
      fs.writeFileSync(tmpFilePath, outputText, "utf-8");
      fs.renameSync(tmpFilePath, this.state.outputFilePath);

      this.state.currentCount = count;
      console.log(`[WaitingCount] 書き込み成功: ${outputText}`);
    } catch (error) {
      console.error("[WaitingCount] ファイル書き込みエラー:", error.message);
    }
  },
};
```

### ポイント解説

#### 1. ファイル監視

```javascript
fs.watch(filePath, { persistent: true }, (eventType) => {
  if (eventType === "change") {
    this.debouncedUpdate();
  }
});
```

- `fs.watch()` でファイルの変更を監視
- `persistent: true` でプロセス終了を防ぐ
- イベント駆動なので CPU 使用率が低い

#### 2. デバウンス処理

```javascript
debouncedUpdate() {
  if (this.state.debounceTimer) {
    clearTimeout(this.state.debounceTimer);
  }
  this.state.debounceTimer = setTimeout(() => {
    this.updateCount();
  }, this.config.debounceMs);
}
```

- 連続した変更を1つにまとめる
- 無駄な処理を削減

#### 3. アトミック書き込み

```javascript
const tmpFilePath = `${this.state.outputFilePath}.tmp`;
fs.writeFileSync(tmpFilePath, outputText, "utf-8");
fs.renameSync(tmpFilePath, this.state.outputFilePath);
```

- 一時ファイルに書き込んでからリネーム
- OBS が読み込み途中の破損ファイルを読まないようにする

#### 4. エラー処理

```javascript
try {
  // 処理
} catch (error) {
  console.error("[WaitingCount] エラー:", error.message);
  // 前回の値を維持
}
```

- エラー時も配信事故を防ぐ
- 前回の値を維持してフォールバック

---

## デバッグ方法

### 1. コンソールログ

わんこめアプリの開発者ツールでログを確認します。

**macOS**:
```
わんこめメニュー → 表示 → 開発者ツールを開く
```

**ログ出力**:
```javascript
console.log("[YourPlugin] メッセージ");
console.error("[YourPlugin] エラー:", error);
```

### 2. ファイル出力でデバッグ

```javascript
const debugLog = (message) => {
  const logPath = path.join(os.homedir(), "Desktop", "debug.log");
  fs.appendFileSync(logPath, `${new Date().toISOString()} ${message}\n`);
};

debugLog("[MyPlugin] 初期化完了");
```

### 3. シンボリックリンクで開発

```bash
cd ~/Library/Application\ Support/onecomme/plugins/
ln -s /path/to/your-plugin ./your-plugin
```

- 元ファイルを編集すると即座に反映
- わんこめ再起動で変更を読み込み

---

## 配布とインストール

### ディレクトリ構造

```
your-plugin/
├── plugin.js         # プラグイン本体（必須）
├── package.json      # パッケージ情報（推奨）
├── README.md         # 使い方（推奨）
└── LICENSE           # ライセンス（推奨）
```

### インストール手順（ユーザー向け）

#### 方法A: 手動コピー

```bash
cd ~/Library/Application\ Support/onecomme/plugins/
mkdir -p your-plugin
cp /path/to/your-plugin/* ./your-plugin/
```

#### 方法B: GitHub から直接ダウンロード

```bash
cd ~/Library/Application\ Support/onecomme/plugins/
git clone https://github.com/your-name/your-plugin.git
```

### わんこめを再起動

プラグインが自動で読み込まれます。

---

## ベストプラクティス

### 1. プラグイン名にプレフィックスを付ける

```javascript
console.log("[YourPlugin] メッセージ");
```

- ログが識別しやすい
- 他のプラグインと混同しない

### 2. エラー処理を徹底する

```javascript
try {
  // 処理
} catch (error) {
  console.error("[YourPlugin] エラー:", error.message);
  // フォールバック処理
}
```

- わんこめがクラッシュしないようにする
- 配信中のトラブルを防ぐ

### 3. リソースを適切にクリーンアップ

```javascript
destroy() {
  if (this.watcher) {
    this.watcher.close();
  }
  if (this.timer) {
    clearTimeout(this.timer);
  }
}
```

- メモリリークを防ぐ
- わんこめの終了を遅延させない

### 4. 設定を外部化する

```javascript
config: {
  prefix: "参加者: ",
  suffix: "人",
  debounceMs: 100,
},
```

- ユーザーがカスタマイズしやすい
- コードを編集せずに調整可能

### 5. 一意な UID を使う

```javascript
uid: "com.yourname.your-plugin",
```

- 逆ドメイン形式を推奨
- 他のプラグインと衝突しない

---

## よくある質問

### Q: プラグインが読み込まれない

**確認事項**:
1. ファイル名が `plugin.js` になっているか
2. わんこめを再起動したか
3. コンソールログにエラーが出ていないか

### Q: ファイル監視が動作しない

**確認事項**:
1. ファイルパスが正しいか
2. ファイルが存在するか
3. `fs.watch()` のエラーハンドラを確認

### Q: プラグインを配布したい

**推奨手順**:
1. README.md でインストール手順を明記
2. LICENSE ファイルを含める（MIT 推奨）
3. GitHub でリポジトリを公開
4. package.json に正確なバージョンを記載

---

## 参考リンク

- [わんこめ公式サイト](https://onecomme.com)
- [Node.js ドキュメント](https://nodejs.org/docs/)
- [waiting-count プラグイン実装例](../waiting-count/)

---

**Happy Plugin Development!** 🎭
