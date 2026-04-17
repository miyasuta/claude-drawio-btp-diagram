---
name: diagram-review
description: Use after generating diagram.drawio (Step 4) to review the XML against project rules and style docs
---

# Diagram Review

## Overview

draw.io XML を生成した直後に、`rules/` と `styles/` のドキュメントに照らして自動レビューする。問題がなければ「レビュー OK」、問題があれば指摘のみ報告する。

## When to Use

- **Step 4（draw.io XML 生成）完了直後に自動実行する**
- 人間が明示的に「レビューして」と言った場合にも実行可能

## Process

1. 対象の `diagram.drawio` を読む
2. 対応する `diagram.md`（Mermaid）を読む
3. 以下のドキュメントを読み、現在のルールを取得する:
   - `docs/rules/conventions.md`
   - `docs/styles/layout.md`
   - `docs/styles/elements.md`
4. カテゴリごとにチェックし、問題を収集する
5. 問題があれば一覧で報告、なければ「レビュー OK」

## Review Categories

各カテゴリはドキュメント参照で判定する。固定チェックリストではなく、**ドキュメントの現在の内容**がルールになる。

| カテゴリ | 参照先 | チェック観点 |
|---|---|---|
| 構成パターン | `rules/conventions.md` | End User+AC セット配置、CIS 配置位置、オンプレ接続 3 点セット、サービスボックスからの矢印集約 |
| ラベル・言語 | `rules/conventions.md` | 要素名が英語か、矢印ラベルが許可リスト内か |
| コンテナの塗り | `styles/elements.md` | L0-L3 の交互ルール（#EBF8FF / #ffffff）に従っているか |
| アイコン形式 | `styles/elements.md` + `layout.md` | SAP アイコンのサイズ、Generic Icon がプレースホルダ形式か、ラベルが別セルか |
| 座標整合 | `styles/layout.md` | 直線矢印で結ぶ要素の中心座標が一致しているか |
| エッジ接続点 | `styles/layout.md` | trust / authenticate 等の重要エッジで entryX/Y, exitX/Y が明示されているか |
| 必須要素 | `diagram.md` との突合 | Mermaid に定義された全ノード・エッジが XML に存在するか |

## Output Format

問題がある場合:

```
## Diagram Review

- [ ] **構成パターン**: Cloud Connector が S/4HANA の箱の外に配置されている（conventions.md §8）
- [ ] **座標整合**: SPA と DEST の中心 Y が 2px ずれている（layout.md §4）

上記を修正しますか？
```

問題がない場合:

```
## Diagram Review

レビュー OK
```
