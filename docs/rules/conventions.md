# ダイアグラム設計規約

ダイアグラムの構成要素をどう組み合わせるか、どう配置するか、ラベルや言語をどうするかといった**設計上の規約**を定義する。Mermaid（論理構造）の段階から適用する。

draw.io の見た目（色・サイズ・座標の px 値）は [`../styles/`](../styles/) を参照。

---

## 1. 言語

特に指示がない限り、ダイアグラム上の要素名は**全て英語**とする。

---

## 2. 矢印ラベル

矢印にラベルは**原則つけない**。以下のように特別な意味を持つ関係のみラベルを設定する:

| ラベル | 用途 |
|---|---|
| `trust` | Subaccount ↔ Cloud Identity Services 間の信頼関係 |
| `authenticate` | 認証フロー |
| `delegates auth` | 認証委譲（CIS ↔ 3rd-party IdP） |

---

## 3. データフローは一直線に並べる

データフローやシーケンスで連続する要素は、**横方向でも縦方向でも一直線に揃える**。折れ線や段差を入れると視覚的な流れが断たれる。

- **横並びの例**: `Application Clients` → `SAP Build Work Zone` → `SAP Task Center` は同じ高さに揃える
- **縦並びの例**: 上から下に流れるシーケンスは同じ列に揃える

> draw.io での実装詳細（直線 vs orthogonal の使い分け、中心座標の px 揃え）は [`../styles/layout.md`](../styles/layout.md) を参照。

---

## 4. End User → Application Clients は常にセットで配置する

End User の右隣に **Application Clients**（Generic Icon）を配置し、BTP 内サービスへの入口とする。

- End User → Application Clients → BTP 内サービス（Build Work Zone 等）
- **認証線は Application Clients から出す**（End User からではない）

---

## 5. SAP Cloud Identity Services は Subaccount の下に配置する

SAP Cloud Identity Services (CIS) は BTP コンテナ内・Subaccount 外の要素。配置は **Subaccount の直下（垂直下方）** に固定する。

- Subaccount ↔ CIS の trust 線は短い垂直線になる
- 左右や上に置くと他の要素と交差しやすく、trust 線の意味が読み取りにくくなる

> draw.io での実装詳細（px サイズ・余白・BTP コンテナ高さの計算）は [`../styles/layout.md`](../styles/layout.md) を参照。

---

## 6. サービスボックスからの矢印は1本にまとめる

サービスボックス（Build Work Zone 等）の中に複数のコンポーネント（アプリ、Inbox 等）がある場合、**個々のコンポーネントから外部サービスへ矢印を引かず、外側のボックスから 1 本の矢印**として出す。

- **理由**: 複数の矢印が1つのアイコンに集中すると見た目が煩雑になる
- Mermaid でも `UI5 App → SPA` ではなく `BWZ → SPA` とする

---

## 7. サービスボックス内のアプリケーション配置

Build Work Zone 等のサービスボックス内にアプリケーションを配置する場合、**内側にコンテナ（箱）を追加**し、その中に Generic Icon プレースホルダ + テキストラベルを横並びで配置する。

```
┌─ BWZ ─────────────────────────┐
│ 🔵 SAP Build Work Zone        │
│ ┌───────────────────────────┐ │
│ │  [icon]      [icon]       │ │
│ │  App Name    My Inbox     │ │
│ └───────────────────────────┘ │
└───────────────────────────────┘
```

- 内側コンテナは青枠・白地・角丸（サービスボックスと同様のスタイル）
- 各アプリは Generic Icon プレースホルダ（40×40）+ 下部テキストラベルで構成
- アプリ同士は横並びに等間隔で配置

> draw.io での実装詳細（px サイズ・スタイル）は [`../styles/elements.md`](../styles/elements.md) を参照。

---

## 8. オンプレミス／Private Cloud 接続パターン

BTP Subaccount からオンプレミスまたは Private Cloud システムへ接続する場合、以下の **3 サービスを 1 セットで Subaccount 内に配置する**:

```
Destination Service → Connectivity Service → Cloud Connector
```

- **Destination Service**: 接続先の設定管理
- **Connectivity Service**: オンプレミスへのトンネル管理
- **Cloud Connector**: 接続先システムの箱の中に配置する（BTP 外）

Cloud Connector は BTP 側のコンポーネントだが、図上は**接続先システムの箱の中**に描く（物理的にオンプレミス環境にインストールされるため）。
