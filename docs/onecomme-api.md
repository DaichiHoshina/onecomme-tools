# わんこめ API リファレンス

わんこめ（OneComme）の公開 API（OneSDK、プラグイン API）の仕様をまとめたリファレンスです。

## 📋 目次

- [OneSDK API](#onesdk-api)
  - [セットアップ](#セットアップ)
  - [パーミッション](#パーミッション)
  - [購読](#購読)
  - [接続](#接続)
  - [コメントオブジェクト](#コメントオブジェクト)
- [プラグイン API](#プラグイン-api)
  - [プラグインオブジェクト](#プラグインオブジェクト)
  - [ライフサイクル](#ライフサイクル)
  - [ファイルシステムAPI](#ファイルシステム-api)
- [データファイル仕様](#データファイル仕様)
  - [waiting.txt](#waitingtxt)
  - [その他のファイル](#その他のファイル)

---

## OneSDK API

OneSDK は、わんこめアプリとブラウザソース（CSS テンプレート）間でデータをやり取りするための JavaScript ライブラリです。

### セットアップ

#### 読み込み

```html
<script src="https://onecomme.com/OneSDK.js"></script>
```

#### 初期化

```javascript
OneSDK.setup({
  permissions: OneSDK.usePermission([
    OneSDK.PERM.COMMENT,
    // 必要なパーミッションを配列で指定
  ]),
});
```

---

### パーミッション

OneSDK で使用可能なパーミッション一覧。

#### `OneSDK.PERM.COMMENT`

コメント取得のパーミッション。

**使用例**:

```javascript
OneSDK.setup({
  permissions: OneSDK.usePermission([OneSDK.PERM.COMMENT]),
});
```

#### `OneSDK.PERM.GIFT`

ギフト（スパチャ、メンバーシップ等）取得のパーミッション。

**使用例**:

```javascript
OneSDK.setup({
  permissions: OneSDK.usePermission([
    OneSDK.PERM.COMMENT,
    OneSDK.PERM.GIFT,
  ]),
});
```

---

### 購読

#### `OneSDK.subscribe(options)`

わんこめからのデータを購読します。

**引数**:

- `options` (object): 購読オプション
  - `action` (string): 購読アクション（`"comments"` 等）
  - `callback` (function): データ受信時のコールバック

**使用例**:

```javascript
OneSDK.subscribe({
  action: "comments",
  callback: (comments) => {
    console.log("コメント受信:", comments);
    this.comments = comments;
  },
});
```

#### 購読アクション

##### `"comments"`

コメント一覧を受信します。

**コールバック引数**:
- `comments` (array): コメントオブジェクトの配列

**受信頻度**: コメント変更時（リアルタイム）

---

### 接続

#### `OneSDK.connect()`

わんこめアプリに接続します。

**使用例**:

```javascript
OneSDK.connect();
```

**注意**:
- `OneSDK.setup()` の後に呼び出す
- `OneSDK.subscribe()` の後に呼び出す

#### `OneSDK.ready()`

わんこめアプリとの接続が完了したら解決される Promise を返します。

**戻り値**: Promise

**使用例**:

```javascript
OneSDK.ready().then(() => {
  console.log("わんこめ接続完了");
  app.mount("#container");
});
```

---

### コメントオブジェクト

#### 構造

```javascript
{
  data: {
    id: string,           // 一意なID
    name: string,         // コメント投稿者名
    comment: string,      // コメント内容
    colors: object | null, // スパチャの色情報（通常コメントはnull）
  },
  // カスタムプロパティ（テンプレート側で追加可能）
  commentIndex: number,   // コメント番号（例）
}
```

#### `data.id`

コメントの一意な識別子。

- 型: `string`
- 例: `"youtube-abc123xyz"`

#### `data.name`

コメント投稿者の表示名。

- 型: `string`
- 例: `"山田太郎"`

#### `data.comment`

コメント本文。

- 型: `string`
- 例: `"こんにちは！"`

#### `data.colors`

スパチャ、メンバーシップの色情報。通常コメントは `null`。

- 型: `object | null`
- 構造:
  ```javascript
  {
    bg: string,  // 背景色（例: "#FFA500"）
    fg: string,  // 文字色（例: "#FFFFFF"）
  }
  ```

**使用例**:

```javascript
if (comment.data.colors) {
  // スパチャの場合
  console.log("背景色:", comment.data.colors.bg);
  console.log("文字色:", comment.data.colors.fg);
} else {
  // 通常コメントの場合
}
```

---

## プラグイン API

プラグイン API は、わんこめアプリの機能を拡張する Node.js プラグインのための API です。

### プラグインオブジェクト

プラグインは、以下のプロパティとメソッドを持つオブジェクトとして定義します。

```javascript
module.exports = {
  // 必須プロパティ
  name: string,
  uid: string,
  version: string,

  // オプションプロパティ
  author: string,
  permissions: string[],
  defaultState: object,

  // ライフサイクルメソッド
  init: function({ dir, store }),
  destroy: function(),
};
```

#### 必須プロパティ

##### `name`

プラグインの名前。

- 型: `string`
- 例: `"わんこめ参加人数表示プラグイン"`

##### `uid`

プラグインの一意な識別子（逆ドメイン形式推奨）。

- 型: `string`
- 例: `"tokyo.vtube-tools.waiting-count"`

##### `version`

プラグインのバージョン（セマンティックバージョニング推奨）。

- 型: `string`
- 例: `"1.0.0"`

#### オプションプロパティ

##### `author`

プラグインの作者名。

- 型: `string`
- 例: `"VTube Tools"`

##### `permissions`

プラグインが必要とするパーミッション（現在は未使用）。

- 型: `string[]`
- 例: `[]`

##### `defaultState`

プラグインの初期状態。

- 型: `object`
- 例: `{}`

---

### ライフサイクル

#### `init({ dir, store })`

プラグイン初期化時に呼ばれます（わんこめ起動時に1回）。

**引数**:

- `options` (object):
  - `dir` (string): プラグインディレクトリの絶対パス
  - `store` (object): わんこめの状態管理オブジェクト

**使用例**:

```javascript
init({ dir, store }) {
  this.store = store;
  console.log(`[MyPlugin] プラグインディレクトリ: ${dir}`);

  // 初期化処理
  this.startWatching();
}
```

**用途**:
- ファイル監視の開始
- 初期データの読み込み
- リソースの初期化

#### `destroy()`

プラグイン終了時に呼ばれます（わんこめ終了時）。

**使用例**:

```javascript
destroy() {
  if (this.watcher) {
    this.watcher.close();
    this.watcher = null;
  }
  console.log("[MyPlugin] クリーンアップ完了");
}
```

**用途**:
- ファイル監視の停止
- タイマーのクリア
- 一時ファイルの削除

---

### ファイルシステム API

プラグインは Node.js の標準モジュールを使用してファイルシステムにアクセスできます。

#### わんこめデータディレクトリ

```javascript
const path = require("path");
const os = require("os");

const onecommeDataDir = path.join(
  os.homedir(),
  "Library",
  "Application Support",
  "onecomme"
);
```

- macOS: `~/Library/Application Support/onecomme/`
- Windows: （未確認、要調査）

#### 推奨パターン

##### ファイル監視

```javascript
const fs = require("fs");

const watcher = fs.watch(filePath, { persistent: true }, (eventType) => {
  if (eventType === "change") {
    // ファイル変更時の処理
  }
});

// エラーハンドリング
watcher.on("error", (error) => {
  console.error("ファイル監視エラー:", error.message);
});
```

##### アトミック書き込み

```javascript
const fs = require("fs");

const tmpFilePath = `${outputFilePath}.tmp`;
fs.writeFileSync(tmpFilePath, content, "utf-8");
fs.renameSync(tmpFilePath, outputFilePath);
```

- 一時ファイルに書き込んでからリネーム
- OBS 等の読み込み途中の破損ファイルを防ぐ

---

## データファイル仕様

わんこめが生成するデータファイルの仕様。

### waiting.txt

参加型機能の参加者一覧ファイル。

#### ファイルパス

```
~/Library/Application Support/onecomme/waiting.txt
```

#### フォーマット

```
1:佐藤 2:小林 3:武藤
```

- 1行目に参加者一覧
- スペース区切り
- `番号:名前` 形式
- 番号は1から始まる連番

#### パース例

```javascript
const fs = require("fs");

const content = fs.readFileSync(waitingFilePath, "utf-8");
const firstLine = content.split("\n")[0] || "";
const parts = firstLine.trim().split(/\s+/);

const participants = parts.map((part) => {
  const match = part.match(/^(\d+):(.+)$/);
  if (match) {
    return {
      number: parseInt(match[1], 10),
      name: match[2],
    };
  }
  return null;
}).filter(Boolean);

console.log("参加者:", participants);
// 出力: [{ number: 1, name: "佐藤" }, { number: 2, name: "小林" }, ...]
```

#### 参加人数の取得

```javascript
const numbers = participants.map((p) => p.number);
const count = numbers.length > 0 ? Math.max(...numbers) : 0;
console.log("参加人数:", count);
```

---

### その他のファイル

わんこめが使用する他のファイルは、公式ドキュメントまたはわんこめアプリ内で確認してください。

---

## OneMarquee コンポーネント

OneSDK が提供する Vue.js コンポーネント。長いコメントを流れるテキストで表示します。

### 使用方法

#### 1. コンポーネント登録

```javascript
app.component("one-marquee", window.OneMarquee());
```

#### 2. テンプレートで使用

```html
<one-marquee :comment="comment">
  {{ comment.data.comment }}
</one-marquee>
```

#### Props

- `comment` (object): コメントオブジェクト

---

## エラーハンドリング

### OneSDK

```javascript
OneSDK.subscribe({
  action: "comments",
  callback: (comments) => {
    try {
      this.comments = comments;
    } catch (error) {
      console.error("コメント処理エラー:", error);
    }
  },
});
```

### プラグイン API

```javascript
init({ dir, store }) {
  try {
    // 初期化処理
  } catch (error) {
    console.error("[MyPlugin] 初期化エラー:", error.message);
  }
}
```

---

## ベストプラクティス

### OneSDK

1. **パーミッションを最小限に**
   ```javascript
   OneSDK.setup({
     permissions: OneSDK.usePermission([
       OneSDK.PERM.COMMENT, // 必要なものだけ
     ]),
   });
   ```

2. **エラーハンドリングを徹底**
   ```javascript
   OneSDK.subscribe({
     action: "comments",
     callback: (comments) => {
       try {
         this.comments = comments;
       } catch (error) {
         console.error(error);
       }
     },
   });
   ```

3. **OneSDK.ready() を使用**
   ```javascript
   OneSDK.ready().then(() => {
     app.mount("#container");
   });
   ```

### プラグイン API

1. **一意な UID を使用**
   ```javascript
   uid: "com.yourname.your-plugin",
   ```

2. **リソースをクリーンアップ**
   ```javascript
   destroy() {
     if (this.watcher) {
       this.watcher.close();
     }
   }
   ```

3. **エラーを適切に処理**
   ```javascript
   try {
     // 処理
   } catch (error) {
     console.error("[MyPlugin] エラー:", error.message);
     // フォールバック処理
   }
   ```

---

## よくある質問

### Q: OneSDK のバージョンはどこで確認できますか？

A: 現在、公式ドキュメントで確認してください。OneSDK.js ファイル内にバージョン情報が記載されている可能性があります。

### Q: プラグインから OneSDK を使用できますか？

A: プラグインは Node.js 環境で動作し、OneSDK はブラウザ環境用のため、直接使用できません。ファイルシステム経由で連携する必要があります。

### Q: カスタムパーミッションは作成できますか？

A: 現在、カスタムパーミッションの作成方法は公開されていません。公式ドキュメントを参照してください。

---

## 制限事項

### OneSDK

- ブラウザ環境のみ（Node.js では使用不可）
- わんこめアプリが起動している必要がある
- ネットワーク越しのリモート接続は非サポート

### プラグイン API

- Node.js 環境のみ（ブラウザでは使用不可）
- ファイルシステムアクセスのみ（ネットワークアクセスは制限される可能性）
- わんこめアプリの UI を直接操作できない

---

## 参考リンク

- [わんこめ公式サイト](https://onecomme.com)
- [プラグイン開発ガイド](./plugin-development.md)
- [CSS テンプレートカスタマイズガイド](./css-customization.md)

---

**Happy Coding!** 🎭
