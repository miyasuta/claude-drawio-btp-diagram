# claude-drawio プロジェクト

## 目的

Claude Code と draw.io MCP を組み合わせて、SAP BTP アーキテクチャ図を効率よく描くワークフローを確立するプロジェクト。人間は言葉で設計を伝え、Claude が Mermaid → draw.io XML に落とし込む。

## スキル化完了（2026-04-24）

このプロジェクトで確立したワークフローは `drawio-btp-diagram-skills` リポジトリにスキル化された。**新規プロジェクトでは `~/.claude/skills/drawio-sap-diagrams`（および `diagram-review`）を使う**。本リポは diagram 作業ワークスペース + 検証サンプル（`diagrams/` 過去成果物）として継続する。

- 新リポ: `/Users/mio/projects/claude-drawio-skill/drawio-btp-diagram-skills/`
- 移行計画: `/Users/mio/projects/claude-drawio-skill/PLAN.md`
- `docs/*` は source of truth を新リポの `skills/drawio-sap-diagrams/references/` に移植済み。当面保持するが、中期的には段階削除予定
- `.claude/skills/diagram-review/` は削除済み（グローバルの symlink を使う）

## 見にいく場所

実際の作業ルール・スタイル定義・公式ガイドラインは `docs/` 配下にある。Claude は会話の内容に応じて以下を読む。

| 目的 | ファイル |
|---|---|
| docs/ の全体索引 | [`docs/README.md`](../docs/README.md) |
| 5 ステップの協働フロー（必ず最初に読む） | [`docs/WORKFLOW.md`](../docs/WORKFLOW.md) |
| SAP 公式ガイドラインの日本語要約（source of truth） | [`docs/GUIDELINES.md`](../docs/GUIDELINES.md) |
| ダイアグラム設計規約（構成パターン・ラベル・言語） | [`docs/rules/conventions.md`](../docs/rules/conventions.md) |
| レイアウト原則・軸揃え・エッジ規則 | [`docs/styles/layout.md`](../docs/styles/layout.md) |
| 個別要素のスタイル定義（BTP / ロゴ / アイコン / 線 等） | [`docs/styles/elements.md`](../docs/styles/elements.md) |

`docs/GUIDELINES.md` が source of truth、`docs/styles/` がその draw.io 実装層。未定義の要素に遭遇したら推測せずユーザーに確認し、再利用可能なら `docs/styles/` に追記していく（詳細は WORKFLOW.md 参照）。

## 出力先ルール

Claude が生成するダイアグラムファイル（`diagram.md`（Mermaid）、`diagram.drawio`（draw.io XML））は以下のディレクトリに出力する。

```
diagrams/<yyyy-mm-dd>-<nn>/
├── diagram.md          ← Mermaid（論理構造）
└── diagram.drawio      ← draw.io XML（貼り付け用）
```

- `yyyy-mm-dd` は作成日（今日の日付を使う）
- `nn` は同じ日の中で作成した図の **2 桁連番（01, 02, 03, ...）**
- 新しい図を作る際は `diagrams/` 配下を `ls` して、その日の最大連番 + 1 を決める
- その日初めての図なら `01` から始める
- 例: `diagrams/2026-04-16-01/`, `diagrams/2026-04-16-02/`

既存の図を修正する場合は、元の日付ディレクトリをそのまま編集する（新しいディレクトリは作らない）。

---

## drawio MCP セットアップ（人間向け）

### 前提条件

- Chrome に [Draw.io MCP Extension](https://chromewebstore.google.com/detail/drawio-mcp-extension/okdbbjbbccdhhfaefmcmekalmmdjjide) がインストール済みであること
- ターミナルで別途 `claude` CLI を起動していないこと（ポート 3333 が競合する）

### 起動手順

1. ターミナルの Claude Code を終了する（起動中の場合）
2. VSCode ウィンドウをリロードする（`Cmd+Shift+P` → `Developer: Reload Window`）
3. draw.io タブで Chrome 拡張機能を開き `Status: Connected`（緑）を確認
4. 新しい会話を開始すると drawio MCP ツールが利用可能になる

### ツールが認識されない場合

```bash
ps aux | grep drawio | grep -v grep   # 残存プロセス確認
pkill -f drawio-mcp-server             # 残っていたら終了
```

その後 VSCode をリロードして再試行。`.mcp.json` に `"type": "stdio"` の明示が無いと VSCode 拡張がサーバーを認識できない点に注意（過去の既知問題・解決済み）。
