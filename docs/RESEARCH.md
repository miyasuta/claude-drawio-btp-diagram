# SAP シェイプ調査結果

## 調査日
2026-04-14

## 目的
draw.io MCP を使って SAP BTP ソリューションダイアグラムを描く際に、SAP 公式アイコン・シェイプを使う方法を確立する。

---

## 調査結果

### 1. MCP ツールからは SAP シェイプを取得できない

`mcp__drawio__get-shape-categories` で返ってくるカテゴリは以下のみ：

```
general, mxgraph.aws4.analytics, mxgraph.aws4.compute,
mxgraph.aws4.containers, mxgraph.aws4.network,
mxgraph.aws4.application_integration, mxgraph.aws4.database
```

SAP 関連のカテゴリは含まれていない。

### 2. シェイプ名での指定も機能しない

```
mcp__drawio__get-shape-by-name("mxgraph.sap.icons.btp") → null
mcp__drawio__get-shape-by-name("SAP Cloud Identity Services") → null
```

`mcp__drawio__add-cell-of-shape` で SAP シェイプ名を指定しても動作しない。

### 3. draw.io の内部スタイルレジストリも空

```javascript
mxStencilRegistry.stencils  // 0 stencils
```

draw.io はシェイプを遅延ロード（サイドバーを展開したときに読み込む）するため、
ページ読み込み直後はスタイルレジストリが空になっている。

---

## 根本原因

**SAP 公式の `.drawio` ファイルを直接調査した結果**（`SAP_Task_Center_L0.drawio` 等）、
SAP アイコンは draw.io のシェイプライブラリ（stencil）として実装されておらず、
**SVG を base64 エンコードして style 属性に直接埋め込む形式**で実装されていることが判明した。

例（SAP Build Work Zone アイコンのスタイル）：
```
style="shape=image;verticalLabelPosition=bottom;...;
image=data:image/svg+xml,PHN2ZyB4bWxucz0i...(base64エンコードされたSVG)...;
strokeWidth=1.5;"
```

---

## SAP シェイプを使う現実的な方法

### 方法A: SAP 公式 `.drawio` ファイルをインポートする（推奨）

SAP 公式リポジトリ（[SAP/btp-solution-diagrams](https://github.com/SAP/btp-solution-diagrams)）に
編集可能な `.drawio` ファイルが公開されている。

```bash
# GitHub CLI でファイルを取得
gh api repos/SAP/btp-solution-diagrams/contents/assets/editable-diagram-examples/SAP_Task_Center_L0.drawio \
  --jq '.content' | base64 -d > /tmp/sap_diagram.drawio
```

その後 `import-diagram` ツールで流し込む。

**注意点**: ファイルが大きい（base64 SVG 埋め込みのため 60KB 超）。
コンテキストを圧迫するため、必要な部分だけ抽出するか、小さいファイルを選ぶこと。

利用可能なファイル一覧（editable-diagram-examples）：
- `SAP_Task_Center_L0.drawio` — シンプルな L0 ダイアグラム
- `SAP_Task_Center_L1.drawio`
- `SAP_Task_Center_L2.drawio`
- `SAP_Build_Work_Zone_L2.drawio`
- `SAP_Cloud_Identity_Services_Authentication_L2.drawio`
- `SAP_Build_Process_Automation_L2.drawio`
- `SAP_Start_L2.drawio`
- `SAP_Private_Link_Service_L2.drawio`

### 方法B: SAP シェイプライブラリ XML をインポートして使う

SAP/btp-solution-diagrams の `shape-libraries-and-editable-presets/draw.io/` 以下に
シェイプライブラリ XML が公開されている。これを draw.io に読み込ませることで、
サイドバーから SAP シェイプを選択できるようになる。

ただし MCP 経由でスタイルライブラリを有効化する方法は未確認。

### 方法C: 矩形 + テキストで代替

SAP アイコンなしでも、矩形・エッジ・テキストラベルで構成要素を表現できる。
`docs/sample.png` のような構造はこの方法でも再現可能。

---

## ツールのパフォーマンス比較

| ツール | 速度 | 理由 |
|---|---|---|
| `add-rectangle` / `add-edge` | **速い** | 1要素を直接APIで追加する |
| `import-diagram` | **遅い・タイムアウトする場合あり** | XMLを丸ごと Chrome 拡張経由で送信しdraw.ioにパース・処理させる |

**結論**: `import-diagram` は使わず、`add-rectangle` / `add-edge` で組み立てる方針が安定する。

---

## SAP スタイルの直接指定（有効な方法）

SAP 公式の `.drawio` ファイルを参照してスタイル文字列を取り出し、`add-rectangle` の `style` パラメータに直接指定することで SAP 風の図形を描ける。

**BTP Subaccount コンテナ**:
```
rounded=1;whiteSpace=wrap;html=1;strokeColor=#0070F2;fillColor=#EBF8FF;
arcSize=32;absoluteArcSize=1;strokeWidth=1.5;verticalAlign=top;
fontSize=14;fontStyle=1;fontColor=#002A86;
```

**SAP ロゴ + テキスト**:
```
image;image=img/lib/sap/SAP_Logo.svg;editableCssRules=.*;part=1;
verticalLabelPosition=middle;labelBackgroundColor=none;verticalAlign=middle;
imageAspect=0;labelPosition=right;align=left;fontSize=16;fontStyle=1;
spacingLeft=0;fontColor=#002A86;spacing=0;
```
※ `parent_id` での親子関係はMCPでは機能しない。絶対座標でコンテナに重ねて配置する。

### 方法D: `shape=mxgraph.sap.icon` + `SAPIcon=` で指定（最もシンプル）

draw.io で SAP ライブラリを有効化してシェイプを配置し、右クリック →「スタイルの編集」でスタイル文字列を確認すると、以下の形式で記述されている：

```
shape=mxgraph.sap.icon;...;aspect=fixed;SAPIcon=SAP_Build_Work_Zone_-_Standard_Edition
```

base64 SVG の埋め込みは不要。`SAPIcon=` にアイコン名を指定するだけでよい。

`add-rectangle` の `style` パラメータにそのまま渡せる。確認済みのアイコン名：
- `SAP_Build_Work_Zone_-_Standard_Edition`
- `SAP_Cloud_Identity_Service`
- `SAP_Task_Center`

---

## 未解決課題

- [ ] `import-diagram` が遅い・タイムアウトする根本原因の特定
- [ ] `parent_id` が機能しない問題（子要素として配置できない）
- [ ] draw.io のサイドバー SAP ライブラリからシェイプを選択した際の stencil 名を CDP で取得する方法
