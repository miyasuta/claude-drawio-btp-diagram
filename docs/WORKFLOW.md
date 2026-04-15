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

> 以下のランドスケープをダイアグラム化してください。
>
> **BTP 内**:
> - Subaccount (BTP 内)
>   - SAP Build Work Zone
>   - SAP Task Center
> - SAP Cloud Identity Services (BTP 内、Subaccount 外)
>
> **BTP 外**:
> - End User (非SAP)
> - Application Clients: Mobile or Desktop (非SAP)
> - 3rd-party Identity Provider (非SAP)
> - SAP On-Premise Solutions (SAP)
> - SAP Cloud Solutions (SAP)
>
> **関係**:
> - End User → Application Clients → Build Work Zone → Task Center → On-Prem / Cloud Solutions
> - Subaccount ↔ Cloud Identity Services (trust)
> - Cloud Identity Services ↔ 3rd-party IdP (delegates auth)

曖昧な部分は Claude が聞き返すので、番号付きで回答する：

> 1. はい / 2. BTP 内で Subaccount 外 / 3. BTP 外 / 4. trust のみ / 5. はい

---

### ステップ2-3: Mermaid で論理構造を確認

Claude に「diagram.md に Mermaid で書き出して」と指示する。人間は VSCode のプレビューで確認し、間違っていたら言葉で修正指示。

> CIS は Subaccount の外に出してください。trust 関係は双方向矢印にしてください。

**注意**: Mermaid ではスタイル（色・線種）まで決めない。論理構造だけに集中する。

---

### ステップ4: draw.io XML 生成

Claude に「diagram.drawio に draw.io 形式で書き出して」と指示する。

#### スタイルの決定

`docs/STYLES.md` が**プロジェクトのスタイル源泉**。Claude はこれを参照して XML を作る。
公式の SAP ガイドライン（`docs/GUIDELINES.md` 経由）は補助資料であり、STYLES.md と食い違った場合は **STYLES.md が優先**。

#### 修正指示の Tips

- **アイコンを触らないでほしい時**: 「アイコンの差し替えはこちらで行うので、それ以外の修正のみ適用してください」
- **埋め込みラベルのスタイルだけ変えたい時**: 「コンテナ埋め込みの状態で、文字のスタイルのみ変更することはできませんか？」（別セル化を防ぐ）
- **線の種類を一括指定**: 「線は横並びになるコンポーネント以外は折れ曲がった直線 (orthogonalEdgeStyle) を使用してください」
- **ルールを STYLES.md に永続化**: 新しく決めたルールは「STYLES.md に追加してください」と明示的に指示すると、次回以降の会話でも反映される

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

## 今日確立したルール（プロジェクト固有）

これらは `docs/STYLES.md` に記載済み：

- **SAP ロゴ必須**: BTP コンテナには必ず SAP ロゴを配置する
- **Subaccount 内の単体サービスは矩形で囲まない**: アイコン + ラベルだけで配置。複数サービスを1つのソリューションとしてグルーピングする場合のみ矩形で囲み、その判断は設計段階で人間が明示する
- **テキストスタイル**: サービス名・コンテナ名は Medium Title (fontSize=12, fontFamily=Helvetica, fontColor=default)
- **線のルール**: 横並び以外は orthogonal（折れ曲がった直線）
- **色のルール**: SAP は #0070F2 ボーダー + #EBF8FF 塗り、非SAP は #475E75 ボーダー + #F5F6F7 塗り
- **trust 線**: #CC00DC (ピンク) 双方向
- **認証委譲線**: #188918 (グリーン) 双方向

---

## ファイル構成

| ファイル | 役割 |
|---|---|
| `diagram.md` | Mermaid 論理構造（ステップ2-3 の成果物） |
| `diagram.drawio` | draw.io XML（ステップ4 の成果物、手貼り用） |
| `docs/STYLES.md` | プロジェクトのスタイル源泉（最優先） |
| `docs/GUIDELINES.md` | SAP 公式ガイドラインへのリンク集 |
| `.claude/CLAUDE.md` | draw.io MCP の使い方・Tips |
| `docs/WORKFLOW.md` | このファイル |

---

## よくある失敗と対策

| 失敗 | 原因 | 対策 |
|---|---|---|
| Subaccount 名が太字で大きい | Claude が独自判断で強調した | Medium Title 指定（fontSize=12, non-bold） |
| ラベルが別セルで作られる | Claude が「スタイル変更 = 新規セル作成」と解釈 | 「コンテナ埋め込みのまま文字スタイルだけ変えて」と指示 |
| trust 線が青い点線 | SAP 公式と STYLES.md を取り違え | STYLES.md が優先と明示 |
| Mermaid から draw.io 変換でスタイルが崩れる | ステップ2で先にスタイルを決めてしまった | Mermaid は論理構造のみ、スタイルはステップ4で |
| サービスが不要な矩形で囲われる | デフォルトで囲う癖 | STYLES.md の「Subaccount 内は囲まない」ルールで防止 |

---

## 次に試したいこと

- Chrome CDP を使った Claude 側自動ビジュアル検証ループ
- SAP 公式アイコンの確実な描画方法（現状はアイコン差し替えを人間側で実施）
- よく使うパターンのテンプレート化（CAP + S/4HANA、Event Mesh など）
