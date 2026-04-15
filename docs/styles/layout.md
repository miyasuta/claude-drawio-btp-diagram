# レイアウトガイド

draw.io で SAP BTP ダイアグラムを描くときのレイアウト原則。**Claude はダイアグラム作成時に必ずこのファイルを読む**。

個別要素のスタイル定義（BTP コンテナ、アイコンなど）は [`elements.md`](elements.md) を参照。

このファイルは SAP 公式ガイドラインを draw.io で具体化した実装層。source of truth は [`../GUIDELINES.md`](../GUIDELINES.md)。公式と食い違いを見つけたら `styles/` 側を公式に合わせる（変更前にユーザーに確認）。

---

## ⚠️ レイアウトに関する注意事項（重要）

ダイアグラム全体のレイアウトは、見た目の整列と密度の両方を意識して決めること。

### 1. 一直線に並べられる要素は極力一直線にする

データフローやシーケンスで連続する要素は、**横方向でも縦方向でも** 一直線に揃える。折れ線や段差を入れると視覚的な流れが断たれる。

- **横並びの例**: `Application Clients` → `SAP Build Work Zone` → `SAP Task Center` は同じ Y 座標に揃える
- **縦並びの例**: 上から下に流れるシーケンスは同じ X 座標に揃える

一直線に並んだ要素間の接続は、横方向・縦方向にかかわらず**直線（まっすぐな矢印）**で描く。曲がる必要がないのに orthogonal を使うと不自然なベンドが入るため避ける。一直線に並ばない要素間のみ **orthogonal（折れ曲がった直線）** を使う。

> 関連: 公式 Foundation の Spacing ルール「オブジェクト間は均等に、SAPロゴの高さ程度」 → [`../GUIDELINES.md` の Spacing セクション](../GUIDELINES.md#スペーシングspacing)

### 2. コンテナの無駄な余白を作らない

Subaccount や BTP コンテナのサイズは、内部要素を配置したうえで**実際に必要な大きさに詰める**。アイコン + ラベルだけのサービスは意外に小さいので、旧サイズのまま大きな箱を残すと余白が目立つ。

**ルール**:
- 子要素の配置が決まったら、親コンテナの width / height を見直す
- 縦横両方向で、不要な余白（特に下部・右側）がないか確認する
- 子要素を削除・縮小した後は、必ず親コンテナも縮小する

**コンテナ内の推奨余白**:
- 左右: 各 30〜55 px
- 上: 40 px（ラベルが入る場合は 90〜110 px でラベル領域を確保）
- 下: 40〜55 px
- 子要素同士の水平間隔: 100 px 前後（アイコン左端から次のアイコン左端まで 170 px が目安）

**実測値の参考**（2 サービスを横並びに配置した Subaccount）:
- Subaccount: 330×210
- 内部の SAP サービスアイコン 2 個（56×56）が同じ Y で横並び
- BTP 全体（Subaccount + その下に CIS コンテナ 280×150）: 410×460

### 3. 標準サイズ

> 関連: 公式 Icons ルール（SAP BTP Service Icon はグレー背景円付きバージョン必須 / Generic Icon はニュートラルグレー） → [`../GUIDELINES.md` の Icons セクション](../GUIDELINES.md#iconsアイコン)。公式テキストには具体 px 仕様が無いため、下記の標準サイズはプロジェクト独自の運用値。

| 要素 | 標準サイズ | 備考 |
|---|---|---|
| SAP サービスアイコン（`mxgraph.sap.icon`） | **56×56** | 変更しない |
| Generic Icon（`shape=image` + base64 SVG） | **40×40（正方形）** | Claude はこのサイズで仮配置する。正方形で収まらないアイコンは**人間が後で調整**する |
| SAP On-Premise / Cloud Solutions 矩形 | 120×60 | |
| 3rd-party Identity Provider 矩形 | 120×60 | |

**Generic Icon のプレースホルダ配置**:
- Claude は Generic Icon の base64 データを扱わない（WORKFLOW.md 参照）
- Claude は **40×40 の正方形の空矩形** を placeholder として仮配置するだけで良い
- アスペクト比が正方形でないアイコン（例: Application Clients は横長の SVG）は、人間が draw.io 上で差し替え時にサイズ調整する
- 重要なのは**位置を先に確定させる**こと。アイコン差し替え後に人間が大きさを微調整しても、中心座標さえ合っていればレイアウトは崩れない

### 4. 接続要素の軸揃え（直線矢印を成立させる）

矢印で繋がる一連の要素は、**中心座標（X または Y）を完全に一致**させる。中心が 1 px でもずれると、draw.io は自動的に orthogonal に折れ曲げてしまう。

**横並びの実例**（同じ Y 中心 = 210）:
- `ac`: y=190, height=40 → 中心 Y = 210
- `bwz`: y=182, height=56 → 中心 Y = 210
- `tc`: y=182, height=56 → 中心 Y = 210

**縦並びの実例**（同じ X 中心 = 74.35）:
- `eu`: x=54.35, width=40 → 中心 X = 74.35
- `ac`: x=50, width=48.69 → 中心 X = 74.345

異なるサイズの要素を一直線に揃えるには、`y = center − height/2`（または X も同様）を計算して配置する。

> 関連: 公式 Lines / Connectors の方向とセマンティック色のルール → [`../GUIDELINES.md` の Lines / Connectors セクション](../GUIDELINES.md#lines--connectors線コネクタ)

### 5. エッジの接続点を明示的に指定する

矢印の source / target だけを指定すると、draw.io が自動でルーティングを決めるため、コンテナの位置調整で経路が崩れることがある。重要な矢印には **`entryX` / `entryY`** を明示的に指定する。

| 目的 | 指定 |
|---|---|
| 相手の上辺中央に刺す | `entryX=0.5;entryY=0;entryDx=0;entryDy=0` |
| 相手の下辺中央に刺す | `entryX=0.5;entryY=1;entryDx=0;entryDy=0` |
| 相手の左辺中央に刺す | `entryX=0;entryY=0.5;entryDx=0;entryDy=0` |
| 相手の右辺中央に刺す | `entryX=1;entryY=0.5;entryDx=0;entryDy=0` |

**使用例**:
- TC → On-Premise / Cloud Solutions: `entryX=0;entryY=0.5`（相手の左辺中央）
- Subaccount → CIS コンテナ (trust): `entryX=0.5;entryY=0`（CIS コンテナの上辺中央）
- CIS → 3rd-party IdP (delegates auth): `entryX=0;entryY=0.5`

source 側も同様に `exitX` / `exitY` で指定可能。複数のエッジが同じ辺から出る場合は分散させる。

---

## MCP / 実装上の制約

- `parent_id` でコンテナの子要素にはできない（drawio-mcp-server の制約）→ 絶対座標で重ねる
- `import-diagram` は動作が遅い・タイムアウトする場合あり → `add-rectangle` / `edit-cell` を使う、または XML を人間が手貼り（WORKFLOW.md 参照）
- ロゴサイズを 37×19 より大きくすると `imageAspect=0` の影響で横伸びする
