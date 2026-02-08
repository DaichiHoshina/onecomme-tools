# わんこめツール集

わんこめ（onecomme）の機能を拡張するプラグイン・CSSテンプレート集です。

## 🛠️ ツール一覧

### 📊 waiting-count - 参加人数表示プラグイン

わんこめの参加型機能の人数をOBS配信画面にリアルタイム表示するプラグイン。

**主な機能**:
- ✅ waiting.txtを自動監視
- ✅ 参加人数をOBS用テキストファイルに出力
- ✅ 軽量・高速（イベント駆動）
- ✅ エラー時も前回値を維持

**ドキュメント**: [waiting-count/README.md](./waiting-count/README.md)

---

### 🎨 simple-css - シンプルCSSテンプレート

わんこめのコメント表示用CSSテンプレート（シンプル版）。

**ドキュメント**: [simple-css/README.md](./simple-css/README.md)

---

### 💡 neon-custome - ネオンCSSカスタマイズ

わんこめのネオンテンプレートを個人的にカスタマイズした版。

**ドキュメント**: [neon-custome/README.md](./neon-custome/README.md)

---

## 📚 ドキュメント

わんこめのカスタマイズ・開発に関する包括的なドキュメントを用意しています。

- **[プラグイン開発ガイド](./docs/plugin-development.md)** - わんこめプラグインの作り方
- **[CSS テンプレートカスタマイズガイド](./docs/css-customization.md)** - CSSテンプレートのカスタマイズ方法
- **[わんこめ API リファレンス](./docs/onecomme-api.md)** - OneSDK と プラグイン API の仕様
- **[コメント表示設定ガイド](./docs/comment-settings.md)** - わんこめアプリでのコメント表示設定とカスタマイズ方法

詳細は [docs/](./docs/) ディレクトリを参照してください。

---

## 📦 プロジェクト構造

```
onecomme-tools/
├── README.md           # このファイル
├── docs/               # ドキュメント
│   ├── README.md
│   ├── plugin-development.md
│   ├── css-customization.md
│   ├── onecomme-api.md
│   └── comment-settings.md
├── waiting-count/      # 参加人数表示プラグイン
│   ├── plugin.js
│   ├── package.json
│   ├── README.md
│   ├── PRD.md
│   └── REVIEW.md
├── simple-css/         # シンプルCSSテンプレート
└── neon-custome/       # ネオンCSSカスタマイズ
```

## 🚀 インストール

### waiting-count プラグイン

```bash
cp -r waiting-count ~/Library/Application\ Support/onecomme/plugins/
```

### CSSテンプレート

各ディレクトリのREADME.mdを参照してください。

---

## 🎯 今後の追加予定

- コメント装飾プラグイン
- 統計・分析ツール
- その他のCSSテンプレート

## ⚠️ 免責事項・使用目的

**このプロジェクトは個人使用・学習目的のツール集です**

- わんこめ（OneComme）の機能を個人的に拡張・カスタマイズしたものです
- 実験的なプロジェクトであり、動作保証はありません
- 本ツールの使用によって生じたいかなる損害についても責任を負いません
- **Use at your own risk / ご自身の責任でご使用ください**

### 📌 わんこめ公式
- **公式サイト**: https://onecomme.com
- **公式テンプレート**: https://onecomme.com/generator/templates

---

## 🤝 コントリビューション

バグ報告や機能要望は、Issueからお願いします。

## 📄 ライセンス

MIT License

Copyright (c) 2026 Daichi Hoshina

---

**Made for わんこめ (onecomme) users** 🎭
