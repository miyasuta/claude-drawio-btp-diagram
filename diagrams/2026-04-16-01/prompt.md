以下のランドスケープをダイアグラム化してください。
不明点は確認してください。

**BTP 内**:
- Subaccount (BTP 内)
  - SAP Build Work Zone
  - UI5 Application
  - Cloud Foundry
    - CAP
  - SAP HANA Cloud
- SAP Cloud Identity Services (BTP 内、Subaccount 外)

**BTP 外**:
- End User (非SAP)
- Application Clients: Mobile or Desktop (非SAP)

**関係**:
- End User → Application Clients → Build Work Zone 
- UI5 Application → Cloud Foundry (CAP)
- Cloud Foundry (CAP) → SAP HANA Cloud
- Subaccount ↔ Cloud Identity Services (trust)