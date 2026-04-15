# 要素別スタイル定義

draw.io で SAP BTP ダイアグラムを描くときの個別要素のスタイル定義集。**Claude はステップ4（draw.io XML 生成）の際にこのファイルを読む**。

レイアウト原則（間隔・標準サイズ・軸揃えなど）は [`layout.md`](layout.md) を参照。

このファイルは SAP 公式ガイドラインを draw.io で具体化した実装層。source of truth は [`../GUIDELINES.md`](../GUIDELINES.md)。公式と食い違いを見つけたら `styles/` 側を公式に合わせる（変更前にユーザーに確認）。

---

## BTP コンテナ（青い矩形）

> 関連: 公式 Areas の色（SAP Border `#0070F2` / Fill `#EBF8FF`）と角丸 16px 固定 → [`../GUIDELINES.md` の Areas セクション](../GUIDELINES.md#areas矩形エリア)。`strokeColor` / `fillColor` は公式に一致。`arcSize=32` + `absoluteArcSize=1` は draw.io 仕様上、実効半径 16 px で公式と一致する（`arcSize` は直径相当として扱われる）。

```
style: rounded=1;whiteSpace=wrap;html=1;strokeColor=#0070F2;fillColor=#EBF8FF;
       arcSize=32;absoluteArcSize=1;strokeWidth=1.5;verticalAlign=top;
       fontSize=14;fontStyle=1;fontColor=#002A86;
text: （空）
```

- ラベルは空にして、ロゴとタイトルを別セルで重ねる

---

## SAP ロゴ

> 関連: 公式 Product Names のルール「SAP 製品名は SAP ロゴとペアで表示 / 同一図内で多数のロゴは非推奨」 → [`../GUIDELINES.md` の Product Names セクション](../GUIDELINES.md#product-names製品名表記)

**⚠️ 必須**: SAP BTP エリア（BTPコンテナ）には必ずSAPロゴを追加すること。ロゴセルは「BTP」というタイトル文字も兼ねる。

```
style: image;image=img/lib/sap/SAP_Logo.svg;editableCssRules=.*;
       verticalLabelPosition=middle;labelBackgroundColor=none;verticalAlign=middle;
       imageAspect=0;labelPosition=right;align=left;
       fontSize=16;fontStyle=1;spacingLeft=0;fontColor=#002A86;spacing=0;
width: 37
height: 19
text: SAP BTP  ← タイトル文字を兼ねる（ロゴ右側に表示される）
```

- `labelPosition=right` によりテキストがロゴ右隣に表示される
- サイズは **37×19** が正しい縦横比（これより大きくすると横伸びする）
- コンテナ内の左上に配置（コンテナ x+15, コンテナ y+11 が目安）
- BTP コンテナ矩形自体の `value` は空にし、ロゴセルで重ねて表示する
- タイトル文字のスタイルはロゴセルに内包されており、単独のスタイル定義は未確立

### タイトル文字のスタイル（ロゴセル内）

| プロパティ | 値 |
|---|---|
| fontSize | 16 |
| fontStyle | 1（太字） |
| fontColor | #002A86 |
| align | left |

---

## Subaccount コンテナ（白地の青枠矩形）

> 関連: 公式 Areas のネスティング規則「塗りの有無を交互にしてコントラストを確保」 → [`../GUIDELINES.md` の Areas セクション](../GUIDELINES.md#areas矩形エリア)。BTP コンテナ（`#EBF8FF` 塗り）の中に白地の Subaccount を置くのは、この alternate fill ルールに沿った実装。

BTP コンテナの中に置く、Subaccount を表す白地の矩形。

```
style: rounded=1;whiteSpace=wrap;html=1;strokeColor=#0070F2;fillColor=#ffffff;
       arcSize=32;absoluteArcSize=1;strokeWidth=1.5;
       verticalAlign=top;align=left;spacingLeft=20;spacingTop=10;
       fontSize=12;fontColor=default;fontFamily=Helvetica;
text: Subaccount
```

- 角の丸みは BTP コンテナと揃える（arcSize=32）
- ラベル「Subaccount」はコンテナに埋め込み、左上寄せ

---

## サービスボックス（白い矩形、グルーピング用）

> 関連: 公式 Areas → [`../GUIDELINES.md` の Areas セクション](../GUIDELINES.md#areas矩形エリア)。角丸半径は公式の 16 px に合わせる（`arcSize=32` + `absoluteArcSize=1`）。

**⚠️ 使用ルール**: Subaccount 内のサービスは基本的に矩形で囲む必要はない（アイコン + ラベルのみで配置する）。**複数のサービスで構成されるソリューションをグルーピングする場合のみ** 矩形で囲む。ソリューション単位のグルーピングが必要かどうかは設計段階で明示的に指示する。

```
style: rounded=1;whiteSpace=wrap;html=1;strokeColor=#0070F2;fillColor=#ffffff;
       arcSize=32;absoluteArcSize=1;imageWidth=64;imageHeight=64;strokeWidth=1.5;
```

---

## テキスト

> 関連: 公式 Text → [`../GUIDELINES.md` の Text セクション](../GUIDELINES.md#textテキスト)。公式は Fiori Horizon 派生の 4 階層テキストスタイルを定義しているが、原文テキストには具体フォントサイズの記載が無く、本ファイルの `fontSize=12` はプロジェクト独自の運用値。

テキストには2通りの使い方がある:

1. **独立した text セルとして配置**（`shape=text`）
2. **矩形/アイコンの `value` に埋め込み、スタイルでフォントを指定**（推奨：コンテナ名・サービス名など、親要素に従属するラベル）

### Medium Title

サブアカウント名、サービス名などに使用する。

**フォントプロパティ（共通）:**

```
fontSize=12;fontColor=default;fontFamily=Helvetica;
```

#### 独立セルとして使う場合

```
style: text;html=1;align=left;verticalAlign=middle;resizable=0;points=[];autosize=1;strokeColor=none;fillColor=none;fontSize=12;fontColor=default;fontFamily=Helvetica;
```

#### コンテナに埋め込む場合（サブアカウント名など）

コンテナ矩形の `value` にタイトルを入れ、スタイルに以下を追加:

```
verticalAlign=top;align=left;spacingLeft=20;spacingTop=10;fontSize=12;fontColor=default;fontFamily=Helvetica;
```

- `verticalAlign=top;align=left` で左上寄せ
- `spacingLeft` / `spacingTop` で内側余白を調整

#### SAP サービスアイコンに埋め込む場合（サービス名）

アイコンセル（`shape=mxgraph.sap.icon`）の `value` にサービス名を入れ、スタイルにフォントプロパティを追加するだけ:

```
...;aspect=fixed;fontSize=12;fontColor=default;fontFamily=Helvetica;SAPIcon=<アイコン名>
```

- アイコン基底スタイルの `verticalLabelPosition=bottom;align=center` により、ラベルはアイコン下に中央配置される

### Additional Information（補助情報テキスト）

注釈・補足情報用のテキスト。

```
style: text;html=1;align=left;verticalAlign=middle;resizable=0;points=[];autosize=1;strokeColor=none;fillColor=none;fontSize=12;fontColor=#1D2D3E;fontFamily=Helvetica;
```

---

## 線（エッジ）

> 関連: 公式 Lines / Connectors → [`../GUIDELINES.md` の Lines / Connectors セクション](../GUIDELINES.md#lines--connectors線コネクタ)。公式は線種で用途を分ける（実線=同期 / 破線=非同期 / 点線=オプション / 太線=ファイアウォール）。

コンポーネント同士をつなぐ線。接続点の指定方法や直線/折れ曲がりの使い分けは [`layout.md`](layout.md) を参照。

### 直線

一直線に並ぶコンポーネントに使用。

```
style: endArrow=blockThin;html=1;strokeColor=#475e75;bendable=1;rounded=0;endFill=1;endSize=4;strokeWidth=1.5;startSize=4;startArrow=none;startFill=0;
```

### 折れ曲がった直線（orthogonal）

一直線ではないコンポーネントに使用。

```
style: edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;strokeColor=#475E75;fontFamily=Helvetica;fontColor=default;targetPerimeterSpacing=15;endSize=4;startSize=4;endArrow=blockThin;endFill=1;strokeWidth=1.5;startArrow=none;startFill=0;
```

### 双方向矢印（trust / delegates auth）

trust 関係や認証委譲など、双方向の関係を表す線。色で意味を区別する。公式のセマンティック色（trust=pink / authentication=green / authorization=indigo）に準拠:

| 関係 | 色 | 例 |
|---|---|---|
| trust | `#CC00DC`（ピンク） | Subaccount ↔ SAP Cloud Identity Services |
| delegates auth | `#188918`（緑） | SAP Cloud Identity Services ↔ 3rd-party IdP |

公式ではこの緑色は **Authentication flows** に割り当てられており、本プロジェクトの「delegates auth（認証委譲）」はその一用法という位置付け。

両端に矢印を出す場合は `startArrow=blockThin;startFill=1` を追加。ラベルの色も線色に合わせる（`fontColor=#CC00DC` など）。

```
style: edgeStyle=orthogonalEdgeStyle;rounded=0;orthogonalLoop=1;jettySize=auto;html=1;
       strokeColor=#CC00DC;fontColor=#CC00DC;fontFamily=Helvetica;
       targetPerimeterSpacing=15;endSize=4;startSize=4;
       endArrow=blockThin;endFill=1;startArrow=blockThin;startFill=1;
       strokeWidth=1.5;fontSize=10;
text: trust
```

---

## SAP サービスアイコン（mxgraph.sap.icon）

> 関連: 公式 Icons のルール「SAP BTP Service Icon は**グレー背景円付きバージョンの使用が必須**」 → [`../GUIDELINES.md` の Icons セクション](../GUIDELINES.md#iconsアイコン)。`mxgraph.sap.icon` シェイプはこの要件を満たす形式。

`shape=mxgraph.sap.icon` を使うと base64 SVG 埋め込み不要でアイコンを指定できる。**Claude が扱うアイコンはこの形式のみ**（Generic Icon は人間担当、WORKFLOW.md 参照）。

### 共通ベーススタイル

```
shape=mxgraph.sap.icon;labelPosition=center;verticalLabelPosition=bottom;
align=center;verticalAlign=top;strokeWidth=1;strokeColor=#D5DADD;
fillColor=#EDEFF0;gradientColor=#FCFCFC;gradientDirection=west;aspect=fixed;
SAPIcon=<アイコン名>
```

サイズは **56×56** 固定（[`layout.md`](layout.md) の標準サイズ表を参照）。

### 主な SAPIcon 名

| サービス | SAPIcon 値 |
|---|---|
| SAP Build Work Zone - Standard Edition | `SAP_Build_Work_Zone_-_Standard_Edition` |
| SAP Cloud Identity Services | `SAP_Cloud_Identity_Service` |
| SAP Task Center | `SAP_Task_Center` |

他のサービスのアイコン名は draw.io のシェイプライブラリから確認できる。
