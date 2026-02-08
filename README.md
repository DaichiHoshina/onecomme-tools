# わんこめツール集

わんこめ（onecomme）の機能を拡張するプラグイン・ツール集です。

## 🛠️ ツール一覧

### waiting-count - 参加人数表示プラグイン

わんこめの参加型機能の人数をOBS配信画面にリアルタイム表示するプラグイン。

- **ディレクトリ**: `waiting-count/`
- **ドキュメント**: [waiting-count/README.md](./waiting-count/README.md)

**主な機能**:
- ✅ waiting.txtを自動監視
- ✅ 参加人数をOBS用テキストファイルに出力
- ✅ 軽量・高速（イベント駆動）
- ✅ エラー時も前回値を維持

**インストール**:
```bash
cp -r waiting-count ~/Library/Application\ Support/onecomme/plugins/
```

詳細は[waiting-count/README.md](./waiting-count/README.md)を参照してください。

---

## 📦 プロジェクト構造

```
onecomme-tools/
├── README.md           # このファイル
└── waiting-count/      # 参加人数表示プラグイン
    ├── plugin.js
    ├── package.json
    ├── README.md
    ├── PRD.md
    └── REVIEW.md
```

## 🚀 今後の追加予定

- コメント装飾プラグイン
- 統計・分析ツール
- カスタムテンプレート

## 🤝 コントリビューション

バグ報告や機能要望は、Issueからお願いします。

## 📄 ライセンス

MIT License

---

**Made for わんこめ (onecomme) users** 🎭
