# OneComme Tools - プロジェクト情報

## プロジェクト概要

わんこめ（OneComme）の機能を拡張するプラグイン・CSSテンプレート集です。
配信者が技術的な知識なしに、わんこめを最大限活用できることを目指しています。

### 主な目標
- ✅ わんこめプラグインの開発・提供
- ✅ CSSテンプレートのカスタマイズ・共有
- ✅ 配信者の利便性向上
- ✅ コミュニティへの貢献

---

## ディレクトリ構造

```
onecomme-tools/
├── README.md                 # プロジェクト README
├── .serena/                  # Serena MCP データ
│   └── memories/
│       ├── project_info.md
│       └── onecome-css-refactoring-2026-02-08.md
│
├── waiting-count/            # 参加人数表示プラグイン
│   ├── plugin.js             # メインロジック
│   ├── package.json          # プラグイン定義
│   ├── README.md             # インストール・使用方法
│   ├── PRD.md                # 要件定義書
│   └── REVIEW.md             # レビュー結果
│
├── simple-css/               # シンプルCSSテンプレート
│   ├── index.html            # テンプレート本体
│   ├── style.css             # スタイル定義
│   ├── script.js             # スクリプト
│   ├── template.json         # わんこめテンプレート定義
│   └── README.md             # 使用方法
│
└── neon-custome/             # ネオンCSSカスタマイズ
    ├── index.html
    ├── style.css
    ├── script.js
    ├── template.json
    └── README.md
```

---

## 提供ツール

### 1. waiting-count - 参加人数表示プラグイン

**種類**: わんこめプラグイン
**状態**: ✅ リリース済み

**主な機能**:
- waiting.txtを自動監視（fs.watch、イベント駆動）
- 参加人数をOBS用テキストファイル（waiting_count.txt）に出力
- エラー時も前回値を維持
- 軽量・高速（アイドル時CPU 0%）

**インストール**:
```bash
cp -r waiting-count ~/Library/Application\ Support/onecomme/plugins/
```

**ドキュメント**: `waiting-count/README.md`

---

### 2. simple-css - シンプルCSSテンプレート

**種類**: わんこめCSSテンプレート
**状態**: ✅ リリース済み

**特徴**:
- Aqua Gradientテーマ
- スーパーチャット/メンバーシップ/ギフト対応
- グラデーション表示
- レスポンシブ対応

**リファクタリング履歴**:
- 2026-02-08: CSS重複削減（41%削減）、メンシプギフト対応
- 詳細: `.serena/memories/onecome-css-refactoring-2026-02-08.md`

**使用方法**: わんこめのテンプレート機能でインポート

**ドキュメント**: `simple-css/README.md`

---

### 3. neon-custome - ネオンCSSカスタマイズ

**種類**: わんこめCSSテンプレート（個人カスタマイズ版）
**状態**: ✅ リリース済み

**特徴**:
- ネオンエフェクトを抑えめに調整
- GPU負荷削減（7層→3層、57%削減）
- 背景色追加（#e0e0e0）
- 視認性向上

**注意**: 個人的なカスタマイズバックアップです。
公式テンプレートは https://onecomme.com からご利用ください。

**使用方法**: わんこめのテンプレート機能でインポート

**ドキュメント**: `neon-custome/README.md`

---

## 開発状況

| ツール | 種類 | 状況 | 進捗 |
|--------|------|------|------|
| waiting-count | プラグイン | ✅ リリース済み | 100% |
| simple-css | CSSテンプレート | ✅ リリース済み | 100% |
| neon-custome | CSSテンプレート | ✅ リリース済み | 100% |

---

## 技術スタック

### waiting-count プラグイン
- **言語**: JavaScript（CommonJS）
- **実行環境**: Node.js（わんこめプラグイン環境）
- **ファイル監視**: fs.watch()（イベント駆動）
- **書き込み**: アトミック書き込み（tmpファイル→rename）

### CSSテンプレート
- **言語**: HTML + CSS + JavaScript
- **対象**: わんこめテンプレート機能
- **互換性**: わんこめ最新版

---

## ライセンス

MIT License

---

## リンク

- **GitHub**: https://github.com/DaichiHoshina/onecomme-tools
- **わんこめ公式**: https://onecomme.com

---

## 今後の追加予定

- コメント装飾プラグイン
- 統計・分析ツール
- その他のCSSテンプレート
- プラグインテンプレート

---

## 関連プロジェクト

- **VTube Tools**: https://github.com/obsidian-engine/vtube-tools
  - VTuber・配信者向けツール集の親プロジェクト
  - Discord立ち絵CSS、OBSテキスト表示など

---

**最終更新**: 2026-02-08
