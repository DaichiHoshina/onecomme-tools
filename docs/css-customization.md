# わんこめ CSS テンプレートカスタマイズガイド

わんこめ（OneComme）のコメント表示テンプレートをカスタマイズするための包括的なガイドです。

## 📋 目次

- [CSS テンプレートとは](#css-テンプレートとは)
- [テンプレートの基本構造](#テンプレートの基本構造)
- [OneSDK の使い方](#onesdk-の使い方)
- [カスタマイズ例](#カスタマイズ例)
- [OBS での使用方法](#obs-での使用方法)
- [ベストプラクティス](#ベストプラクティス)
- [トラブルシューティング](#トラブルシューティング)

---

## CSS テンプレートとは

わんこめの CSS テンプレートは、OBS Studio でコメントを表示するための HTML/CSS/JavaScript ファイルセットです。

### できること

- コメントの見た目を完全にカスタマイズ
- スパチャ、メンバーシップの特別表示
- アニメーション、エフェクトの追加
- レイアウトの自由な調整

### 必要な知識

- HTML/CSS の基礎
- JavaScript の基礎（Vue.js を使用）
- OneSDK の基本的な使い方

---

## テンプレートの基本構造

### ファイル構成

```
your-template/
├── index.html        # メインHTML（必須）
├── style.css         # カスタムCSS（必須）
├── script.js         # Vue.js アプリケーション（必須）
├── template.json     # テンプレート情報（必須）
└── thumb.png         # サムネイル画像（オプション）
```

### 最小構成

#### 1. template.json

```json
{
  "name": "your-template",
  "author": "Your Name",
  "link": "https://github.com/your-name/your-template",
  "description": "Template description"
}
```

#### 2. index.html

```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>OneComme Template</title>
  <link rel="stylesheet" href="style.css">
  <script src="https://onecomme.com/OneSDK.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/vue@3"></script>
</head>
<body hidden>
  <div id="container">
    <div v-for="comment in comments" :key="comment.data.id" :class="getClassName(comment)">
      <!-- コメント名前タグ -->
      <div class="name">{{ comment.data.name }}</div>

      <!-- コメント本文 -->
      <div class="message">
        <one-marquee :comment="comment">
          {{ comment.data.comment }}
        </one-marquee>
      </div>
    </div>
  </div>

  <script src="script.js"></script>
</body>
</html>
```

#### 3. script.js

```javascript
const app = Vue.createApp({
  setup() {
    document.body.removeAttribute("hidden");
  },
  data() {
    return {
      comments: [],
    };
  },
  methods: {
    getClassName(comment) {
      // 偶数・奇数でクラスを変える
      if (comment.commentIndex % 2 === 0) {
        return "comment even";
      }
      return "comment odd";
    },
  },
  mounted() {
    let cache = new Map();
    let commentIndex = 0;

    // OneSDK セットアップ
    OneSDK.setup({
      permissions: OneSDK.usePermission([OneSDK.PERM.COMMENT]),
    });

    // コメント受信
    OneSDK.subscribe({
      action: "comments",
      callback: (comments) => {
        const newCache = new Map();
        comments.forEach((comment) => {
          const index = cache.get(comment.data.id);
          if (isNaN(index)) {
            comment.commentIndex = commentIndex;
            newCache.set(comment.data.id, commentIndex);
            ++commentIndex;
          } else {
            comment.commentIndex = index;
            newCache.set(comment.data.id, index);
          }
        });
        cache = newCache;
        this.comments = comments;
      },
    });

    OneSDK.connect();
  },
});

app.component("one-marquee", window.OneMarquee());

OneSDK.ready().then(() => {
  app.mount("#container");
});
```

#### 4. style.css

```css
:root {
  /* カスタマイズ可能な変数 */
  --name-bg: linear-gradient(104deg, #53B7FF 0%, #81E3DD 94.77%);
  --name-color: #FFFFFF;
  --message-bg: #FFFFFF;
  --message-color: #333333;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: "Noto Sans JP", sans-serif;
  background: transparent;
}

#container {
  padding: 20px;
}

.comment {
  margin-bottom: 12px;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.name {
  background: var(--name-bg);
  color: var(--name-color);
  padding: 8px 16px;
  font-weight: bold;
  font-size: 18px;
}

.message {
  background: var(--message-bg);
  color: var(--message-color);
  padding: 12px 16px;
  font-size: 20px;
  line-height: 1.6;
}
```

---

## OneSDK の使い方

OneSDK は、わんこめアプリとブラウザソース間でデータをやり取りするための JavaScript ライブラリです。

### 1. セットアップ

```javascript
OneSDK.setup({
  permissions: OneSDK.usePermission([
    OneSDK.PERM.COMMENT,  // コメント取得
  ]),
});
```

#### 利用可能なパーミッション

- `OneSDK.PERM.COMMENT`: コメント取得
- `OneSDK.PERM.GIFT`: ギフト取得（スパチャ、メンバーシップ等）
- その他は公式ドキュメント参照

### 2. コメント購読

```javascript
OneSDK.subscribe({
  action: "comments",
  callback: (comments) => {
    this.comments = comments;
  },
});
```

#### コメントオブジェクトの構造

```javascript
{
  data: {
    id: "unique-id",           // 一意なID
    name: "ユーザー名",         // コメント投稿者名
    comment: "コメント本文",    // コメント内容
    colors: null,              // 通常コメントの場合null
                               // スパチャの場合: { bg: "#色", fg: "#色" }
  },
  commentIndex: 0,             // コメント番号（カスタムプロパティ）
}
```

### 3. 接続

```javascript
OneSDK.connect();
```

### 4. マウント

```javascript
OneSDK.ready().then(() => {
  app.mount("#container");
});
```

---

## カスタマイズ例

### 1. グラデーション背景を変更

**style.css**:

```css
:root {
  /* 水色グラデーション → ピンクグラデーション */
  --name-bg: linear-gradient(104deg, #FF53B7 0%, #DDA1E3 94.77%);
}
```

### 2. スパチャを目立たせる

**script.js**:

```javascript
methods: {
  getClassName(comment) {
    // スパチャの場合は特別なクラス
    if (comment.data.colors) {
      return "comment superchat";
    }
    return "comment";
  },
  getStyle(comment) {
    // スパチャの色を適用
    if (comment.data.colors) {
      return {
        '--name-bg': comment.data.colors.bg,
        '--message-bg': comment.data.colors.bg,
        '--message-color': comment.data.colors.fg,
      };
    }
    return {};
  },
}
```

**index.html**:

```html
<div v-for="comment in comments"
     :key="comment.data.id"
     :class="getClassName(comment)"
     :style="getStyle(comment)">
  <!-- ... -->
</div>
```

**style.css**:

```css
.comment.superchat {
  box-shadow: 0 0 20px rgba(255, 215, 0, 0.8);
  animation: glow 1s ease-in-out infinite alternate;
}

@keyframes glow {
  from {
    box-shadow: 0 0 20px rgba(255, 215, 0, 0.8);
  }
  to {
    box-shadow: 0 0 30px rgba(255, 215, 0, 1);
  }
}
```

### 3. コメント間の余白を調整

**style.css**:

```css
.comment {
  margin-bottom: 16px;  /* 12px → 16px */
}
```

### 4. フォントを変更

**index.html** (head内):

```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=M+PLUS+Rounded+1c:wght@400;700&display=swap" rel="stylesheet">
```

**style.css**:

```css
body {
  font-family: "M PLUS Rounded 1c", sans-serif;
}
```

### 5. ネオン光彩エフェクト

**style.css**:

```css
:root {
  --neon-color: #00FFFF;
  --neon-shadow: 0 0 3px #fff,
                 0 0 6px var(--neon-color),
                 0 0 9px var(--neon-color);
}

.comment {
  text-shadow: var(--neon-shadow);
  box-shadow: 0 0 10px var(--neon-color);
}
```

---

## OBS での使用方法

### 1. ブラウザソースを追加

1. OBS で「ソース」パネルの「+」をクリック
2. 「ブラウザ」を選択
3. ソース名を「わんこめコメント」等に設定

### 2. URL を設定

#### わんこめアプリから直接読み込む場合

```
http://localhost:11180/
```

- わんこめアプリが起動している必要がある
- ポート番号はわんこめの設定で確認

#### ローカルファイルから読み込む場合

```
file:///path/to/your-template/index.html
```

- テンプレートの絶対パスを指定
- わんこめアプリとの接続は OneSDK が自動で行う

### 3. サイズを設定

- 幅: 1920px
- 高さ: 1080px

（配信解像度に合わせて調整）

### 4. カスタム CSS（通常は不要）

テンプレートに CSS が含まれている場合は設定不要。

### 5. 動作確認

1. わんこめアプリでコメントをテスト送信
2. OBS 画面にコメントが表示されることを確認

---

## ベストプラクティス

### 1. CSS 変数を活用する

```css
:root {
  --primary-color: #53B7FF;
  --secondary-color: #81E3DD;
}

.comment {
  background: var(--primary-color);
}
```

- カスタマイズが簡単
- 一箇所変更で全体に反映

### 2. パフォーマンスを考慮する

```css
/* ❌ 重い */
box-shadow: 0 0 2px #fff, 0 0 4px #fff, 0 0 6px #fff,
            0 0 8px #0ff, 0 0 12px #0ff, 0 0 16px #0ff,
            0 0 20px #0ff, 0 0 32px #0ff;

/* ✅ 軽い */
box-shadow: 0 0 3px #fff, 0 0 6px #0ff, 0 0 9px #0ff;
```

- GPU 負荷を抑える
- レイヤー数を減らす

### 3. レスポンシブデザインにしない

OBS はブラウザソースのサイズを固定で使用するため、レスポンシブ対応は不要。

### 4. 読みやすさを重視する

```css
.message {
  font-size: 20px;        /* 小さすぎない */
  line-height: 1.6;       /* 適切な行間 */
  color: #333333;         /* 背景とのコントラスト */
  background: #FFFFFF;
}
```

### 5. アニメーションを控えめに

```css
/* ❌ 激しいアニメーション */
@keyframes crazy {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

/* ✅ 控えめなアニメーション */
@keyframes subtle-glow {
  from { opacity: 0.9; }
  to { opacity: 1; }
}
```

---

## トラブルシューティング

### Q: コメントが表示されない

**確認事項**:
1. わんこめアプリが起動しているか
2. OBS のブラウザソースの URL が正しいか
3. ブラウザソースのサイズが正しく設定されているか
4. OneSDK が正しく読み込まれているか

**デバッグ方法**:
```javascript
OneSDK.subscribe({
  action: "comments",
  callback: (comments) => {
    console.log("コメント受信:", comments);
    this.comments = comments;
  },
});
```

OBS のブラウザソース上で右クリック → 「ソースと対話」→ 開発者ツールでログ確認。

### Q: スタイルが反映されない

**確認事項**:
1. `style.css` のファイルパスが正しいか
2. CSS の記述にエラーがないか
3. ブラウザキャッシュをクリアしたか

**解決方法**:
- OBS のブラウザソースを削除して再度追加
- わんこめアプリを再起動

### Q: パフォーマンスが悪い

**改善方法**:
1. ネオンエフェクトのレイヤー数を減らす
2. アニメーションを削減する
3. 不要な要素を削除する
4. CSS の `will-change` を使用する

```css
.comment {
  will-change: transform, opacity;
}
```

### Q: 文字が切れる

**解決方法**:
```css
.message {
  word-wrap: break-word;
  overflow-wrap: break-word;
}
```

---

## 参考テンプレート

このリポジトリには以下のサンプルテンプレートが含まれています。

### simple-css

- 水色グラデーションのシンプルなテンプレート
- 読みやすさ重視
- 軽量

### neon-custome

- ネオンエフェクト付きテンプレート
- GPU 負荷を軽減したカスタマイズ版
- 目立つ演出

---

## 参考リンク

- [わんこめ公式サイト](https://onecomme.com)
- [わんこめテンプレート一覧](https://onecomme.com/generator/templates)
- [Vue.js 3 ドキュメント](https://v3.ja.vuejs.org/)
- [OBS Studio](https://obsproject.com/ja)

---

**Happy Customizing!** 🎨
