# docs/ 構成

Claude Code と協働して draw.io で SAP BTP ダイアグラムを描くためのドキュメント。

## ファイル構成

```
docs/
├── README.md           ← このファイル。索引
├── WORKFLOW.md         ← 人間 & Claude の協働フロー（まずこれを読む）
├── GUIDELINES.md       ← SAP 公式ガイドラインの日本語要約（source of truth）
└── styles/
    ├── layout.md       ← レイアウト原則・標準サイズ・軸揃え・エッジ規則
    └── elements.md     ← 個別要素のスタイル定義（BTP, ロゴ, アイコン, 線 等）
```

## Claude が読むべきタイミング

| ステップ | 読むファイル |
|---|---|
| 会話開始時 | [`WORKFLOW.md`](WORKFLOW.md) — フロー全体と Claude / 人間の分担 |
| ステップ1-3（論理構造 / Mermaid） | [`WORKFLOW.md`](WORKFLOW.md) のみで十分 |
| ステップ4（draw.io XML 生成） | [`styles/layout.md`](styles/layout.md) + [`styles/elements.md`](styles/elements.md) |
| 既存ダイアグラムの修正 | [`styles/layout.md`](styles/layout.md) + [`styles/elements.md`](styles/elements.md) |
| スタイル判断に迷ったとき | [`GUIDELINES.md`](GUIDELINES.md) — SAP 公式準拠の確認 |

## スタイルの source of truth

- **[`GUIDELINES.md`](GUIDELINES.md) = SAP 公式ガイドラインの要約 = source of truth**
- **[`styles/`](styles/) = GUIDELINES を draw.io XML として具体化した実装層**
- `styles/` と公式が食い違う場合、原則として **`styles/` 側を公式に合わせて修正** する。変更の前に必ずユーザーに確認する（過去の iteration で意図的にずらしている可能性があるため）。

## Generic Icon について

`shape=image` + base64 SVG 形式の Generic Icon（End User / Application Clients / 3rd-party Identity Provider など）は **Claude は扱わず、人間が draw.io 上で配置する**。理由と運用は [`WORKFLOW.md`](WORKFLOW.md) の「アイコンの扱い」セクション参照。

Claude は 40×40 の空矩形プレースホルダだけ置けば良い。
