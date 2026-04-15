# SAP ダイアグラム作成ワークフロー

Claude Code と協働して SAP BTP アーキテクチャ図を draw.io で作成するための手順と Tips。

---

## 基本ワークフロー（5ステップ）

```
1. 人間: 設計を言葉で記述
      ↓
2. Claude: Mermaid 図を作成して論理構造を可視化（diagram.md）
      ↓
3. 人間: Mermaid プレビューで論理構造を確認・修正指示
      ↓
4. Claude: diagram.drawio に draw.io XML を生成
      ↓
5. 人間: XML を draw.io に手動で貼り付けてビジュアル確認
```

**なぜ Mermaid を間に挟むか**: draw.io XML は冗長で人間がレビューしづらい。Mermaid で論理構造（ノード・エッジ・グルーピング）だけを先に確定させることで、draw.io XML 生成時はスタイルに集中できる。

**なぜ MCP で直接描画しないか**: MCP 経由の描画はツール呼び出し回数が多く時間がかかる。XML を手貼りする方が速い。ただし細かい修正フェーズでは MCP の `edit-cell` / `delete-cell-by-id` が役立つ。

---

## ステップごとの人間の指示 Tips

### ステップ1: 設計を言葉で伝える

Claude には以下を明示的に伝える：

- **構成要素のリスト**（サービス名・役割）
- **階層・所属関係**（何が何の中に入るか）
- **関係性**（データフロー、trust、認証委譲など）
- **SAP / 非SAP の区別**（色分けに影響）

#### サンプル指示文

そのままプロンプトに貼れる形式：

```text
以下のランドスケープをダイアグラム化してください。

**BTP 内**:
- Subaccount (BTP 内)
  - SAP Build Work Zone
  - SAP Task Center
- SAP Cloud Identity Services (BTP 内、Subaccount 外)

**BTP 外**:
- End User (非SAP)
- Application Clients: Mobile or Desktop (非SAP)
- 3rd-party Identity Provider (非SAP)
- SAP On-Premise Solutions (SAP)
- SAP Cloud Solutions (SAP)

**関係**:
- End User → Application Clients → Build Work Zone → Task Center → On-Prem / Cloud Solutions
- Subaccount ↔ Cloud Identity Services (trust)
- Cloud Identity Services ↔ 3rd-party IdP (delegates auth)
```

曖昧な部分は Claude が聞き返すので、番号付きで回答する：

> 1. はい / 2. BTP 内で Subaccount 外 / 3. BTP 外 / 4. trust のみ / 5. はい

---

### ステップ2-3: Mermaid で論理構造を確認

Claude に「diagram.md に Mermaid で書き出して」と指示する。人間は VSCode のプレビューで確認し、間違っていたら言葉で修正指示。

> CIS は Subaccount の外に出してください。trust 関係は双方向矢印にしてください。

**注意**: Mermaid ではスタイル（色・線種）まで決めない。論理構造だけに集中する。

---

### ステップ4: draw.io XML 生成

Claude に「diagram.drawio に draw.io 形式で書き出して」と指示する。Claude はこのステップで [`styles/layout.md`](styles/layout.md) と [`styles/elements.md`](styles/elements.md) を読んで XML を組み立てる。スタイル定義は人間が直接触る必要はなく、スタイル変更が必要なら「`styles/` のルールに従って」と言うだけで良い。

#### XML 生成前の準備（推奨手順）

XML を書き始める前に、以下を済ませておくと往復が減る。

1. **必要要素の棚卸し**: その図で使うサービスアイコン名・ボックス種別・新規のレイアウトパターンを列挙する
2. **styles / GUIDELINES に存在するか確認**: [`styles/elements.md`](styles/elements.md) の SAPIcon 表、[`styles/layout.md`](styles/layout.md) の標準サイズ表、[`GUIDELINES.md`](GUIDELINES.md) を突き合わせる
3. **不明点を一括でユーザーに確認**: 不明な SAPIcon 名や未定義パターンを **全部まとめて** 聞く（遭遇するたびに聞くと会話が細切れになる）

#### 座標の粒度

Claude が XML を書く際、座標は粗く決めて良い。**最終的な位置微調整は人間が draw.io 上で行う**。Claude が厳密に合わせるべきなのは以下だけ:

- **直線矢印で結ぶ要素の中心座標**（中心が 1 px でもずれると orthogonal に折れ曲がる、[`styles/layout.md`](styles/layout.md) §4 参照）
- **コンテナの内外関係**（BTP の中 / Subaccount の中 / ボックスの中、の所属）

それ以外（コンテナの幅・高さ、余白、矢印のベンド点など）はおおよそで良い。ピクセル単位の計算に時間をかけるより、粗い XML を早く出してユーザーのフィードバックを受ける方が速い。

#### スタイルの source of truth

- **[`GUIDELINES.md`](GUIDELINES.md) = SAP 公式ガイドラインの日本語要約 = source of truth**
- **[`styles/`](styles/) = GUIDELINES を draw.io XML として具体化した実装層**
- `styles/` と公式が食い違う場合は、原則として `styles/` を公式に合わせて修正する。Claude は食い違いを見つけても勝手に直さず、**ユーザーに報告してから対応**する（過去の iteration で意図的にずらしている可能性があるため）。

#### 不明なスタイル・アイコンに遭遇したとき（重要）

`styles/` と `GUIDELINES.md` のどちらにも定義されていない要素（未登録のサービスアイコン `SAPIcon` 名、新種の矩形、初めて使う色 / 線種 など）に遭遇したら、**推測で描かずユーザーに確認する**。確認の粒度：

- **SAP サービスアイコン**: `SAPIcon=...` の正式名が不明なときは、シェイプ名を推測せずユーザーに聞く（誤った名前だと draw.io 側で空表示になる）
- **スタイル仕様**: 色・角丸・線種・フォント等で `styles/` に定義がない要素は、公式 GUIDELINES に該当セクションがあればそれを引用し、無ければユーザーに確認する
- **レイアウト判断**: 新しい配置パターン（スタックコンテナ、interface 記号など）は勝手に採用しない

確認の結果、**再利用可能なルールだと判断したら [`styles/layout.md`](styles/layout.md) または [`styles/elements.md`](styles/elements.md) に追記**する。公式由来のルールなら [`GUIDELINES.md`](GUIDELINES.md) の該当セクションへの相互リンクも一緒に足す。これにより `styles/` が使用と共に育っていき、次回以降は再質問が不要になる。

> Claude への指示例: 「この図で SAP Integration Suite のアイコンが要るけど `styles/elements.md` に `SAPIcon` 名の記載が無い。正しい名前を教えてもらえたら、そのまま `styles/elements.md` の SAPIcon 表に追記してください」

#### Claude / 人間の分担

| 内容 | 担当 |
|---|---|
| SAP サービスアイコン（`mxgraph.sap.icon`）の配置 | **Claude** |
| Generic Icon（`shape=image` + base64 SVG）の配置 | **人間**（Claude は空プレースホルダを置くだけ。サイズは `styles/layout.md` 参照） |
| コンテナ・線・テキスト・スタイル全般 | **Claude** |

**なぜ Generic Icon のみ人間担当か**: base64 エンコードされた SVG を 1 アイコンあたり数 KB、style 属性に埋め込む必要があり、Claude が XML 内で扱うとトークン消費・応答時間が膨れ上がる。SAP サービスアイコンは `SAPIcon=<名前>` の短い識別子で済むため問題ない。

会話の冒頭で「Generic Icon は人間が後から配置するので、SAP サービスアイコンのみ XML に記述してください」と伝えておくと誤解がない。

#### 指示の Tips

- **アイコンを触らないでほしい時**: 「アイコンの差し替えはこちらで行うので、それ以外の修正のみ適用してください」
- **埋め込みラベルのスタイルだけ変えたい時**: 「コンテナ埋め込みの状態で、文字のスタイルのみ変更することはできませんか？」（別セル化を防ぐ）
- **新しいルールを永続化したい時**: 「`docs/styles/layout.md`（または `elements.md`）に追加してください」と明示的に指示する

---

### ステップ5: draw.io でビジュアル確認

生成された `diagram.drawio` の内容を draw.io にペーストして目視確認。問題があれば言葉で修正指示に戻る。

**Chrome CDP を使った確認ループ**（Claude 側で実行可能）:

```bash
# 1. draw.io タブの ID を取得
node /Users/mio/.claude/skills/chrome-cdp-skill/skills/chrome-cdp/scripts/cdp.mjs list | grep -i draw

# 2. スクリーンショット取得
node /Users/mio/.claude/skills/chrome-cdp-skill/skills/chrome-cdp/scripts/cdp.mjs shot <タブID> /tmp/drawio-check.png
```

Claude がスクリーンショットを Read で読み込んでビジョンで検証できる。

---

## ファイル構成

| ファイル | 役割 |
|---|---|
| `diagram.md` | Mermaid 論理構造（ステップ2-3 の成果物） |
| `diagram.drawio` | draw.io XML（ステップ4 の成果物、手貼り用） |
| `docs/README.md` | docs/ 配下の索引 |
| `docs/WORKFLOW.md` | このファイル（フロー全体） |
| `docs/GUIDELINES.md` | SAP 公式ガイドラインの日本語要約（source of truth） |
| `docs/styles/layout.md` | レイアウト原則・標準サイズ・軸揃え・エッジ規則（GUIDELINES の実装層） |
| `docs/styles/elements.md` | 個別要素のスタイル定義（GUIDELINES の実装層） |
| `.claude/CLAUDE.md` | draw.io MCP の使い方・Tips |

---

## よくある作業上の失敗と対策

個別のスタイル誤りは `docs/styles/` 側で管理する。ここにはワークフロー運用で起こる失敗だけを載せる。

| 失敗 | 原因 | 対策 |
|---|---|---|
| ステップ2で Mermaid にスタイル情報を入れてしまう | 論理構造と見た目を同時に決めようとした | Mermaid は論理構造のみ、スタイルはステップ4で |
| Claude が独自判断でスタイルを変えてしまう | `styles/` 参照を明示しなかった | ステップ4の冒頭で「`docs/styles/` のルールに従って」と明示する |
| ラベルが別セルで作られてしまう | Claude が「スタイル変更 = 新規セル作成」と解釈 | 「コンテナ埋め込みのまま文字スタイルだけ変えて」と指示 |
| Claude が Generic Icon の base64 を触ってしまう | 役割分担を伝えていない | ステップ4の冒頭で「Generic Icon は人間が配置する」と明示 |
| Claude が新しく決めたルールを忘れる | 会話内だけで合意して永続化しなかった | 「`docs/styles/layout.md`（または `elements.md`）に追加してください」と指示する |
| アイコン名の確認で会話が細切れになる | XML 生成中に未登録アイコンに遭遇するたびユーザーに聞いた | ステップ4冒頭で必要アイコンを列挙し、未登録分を **一括で** 確認する（上記「XML 生成前の準備」参照） |
| Claude が座標計算に時間をかけすぎる | ピクセル単位で完璧に合わせようとした | 最終調整は人間の仕事。Claude は直線矢印の両端と所属関係だけ厳密にし、他は粗で出す |

---

## 次に試したいこと

- Chrome CDP を使った Claude 側自動ビジュアル検証ループ
- SAP 公式アイコンの確実な描画方法（現状はアイコン差し替えを人間側で実施）
- よく使うパターンのテンプレート化（CAP + S/4HANA、Event Mesh など）
