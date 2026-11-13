# Data

原始資料檔案體積大，**不納入版本控制**（見 `.gitignore`）。請依下列方式下載並放到對應目錄。

| 資料集 | 放置位置 | 來源 | 授權 |
|---|---|---|---|
| Microsoft GUIDE | `data/archive/` | Kaggle：*Microsoft Security Incident Prediction*；論文 [arXiv:2407.09017](https://arxiv.org/abs/2407.09017) | CDLA-2.0（依原始論文） |
| ExCyTIn-Bench | 依官方 repo 放在 `secgym/database/` | 程式：[microsoft/SecRL](https://github.com/microsoft/SecRL)；資料：Hugging Face `anandmudgerikar/excytin-bench`；論文 [arXiv:2507.14201](https://arxiv.org/abs/2507.14201) | 程式為 MIT |
| MITRE ATT&CK（STIX 2.1） | `data/raw/attack/` | MITRE ATT&CK 官方 STIX 資料 | MITRE Terms of Use |
| Splunk BOTS v3（選用） | `data/raw/bots/` | [Splunk BOTS v3](https://www.splunk.com/en_us/blog/security/botsv3-dataset-released.html) | CC0 |

## GUIDE 資料說明（依原始論文 arXiv:2407.09017 附錄 B、C）

- **檔案**：train（70%）與 test（30%）兩個 CSV，皆放在 Kaggle。實際檔名與大小見下方「目前的實際放置位置」。
- **欄位**：共 45 欄（已實際檢查確認），包含特徵、標籤與識別碼。
  - 識別碼：`Id`、`OrgId`、`IncidentId`、`AlertId`、`Timestamp`
  - 告警：`DetectorId`、`AlertTitle`、`Category`、`MitreTechniques`
  - 標籤：`IncidentGrade`（TP / BP / FP）、`ActionGrouped`、`ActionGranular`
  - 證據實體：`EntityType`、`EvidenceRole`、`Roles`、`DeviceId`、`DeviceName`、`Sha256`、`IpAddress`、`Url`、`AccountSid`、`AccountUpn`、`AccountObjectId`、`AccountName`、`NetworkMessageId`、`EmailClusterId`、`RegistryKey`、`RegistryValueName`、`RegistryValueData`、`ApplicationId`、`ApplicationName`、`OAuthApplicationId`、`ThreatFamily`、`FileName`、`FolderPath`、`ResourceIdName`、`ResourceType`
  - 環境：`OSFamily`、`OSVersion`、`AntispamDirection`、`SuspicionLevel`、`LastVerdict`、`CountryCode`、`State`、`City`
- **匿名化**：敏感值先以 SHA1 雜湊，再替換成隨機產生的 ID；時間戳記加入雜訊。論文沒有列出哪些欄位被匿名化，實際檢查結果見下方待確認事項與 `DESIGN.md` §5。

## 待確認事項

- [x] GUIDE 是否包含 `MitreTechniques`、`IncidentGrade` 等欄位（論文 Table 4 確認有）
- [x] GUIDE 各欄位實際的匿名化程度：`AlertTitle` 與實體欄位為數字 ID；`Category`、`MitreTechniques` 等為可讀文字（詳見 `DESIGN.md` §5）
- [x] GUIDE 檔案大小：`GUIDE_Train.csv` 2.4 GB、`GUIDE_Test.csv` 1.1 GB
- [x] ExCyTIn-Bench 官方 repo 與授權：[microsoft/SecRL](https://github.com/microsoft/SecRL)，MIT；需要 Docker 與 MySQL，8 個事件的資料庫約佔 10 GB
- [ ] 實體欄位中代表「無此實體」的固定 ID
- [ ] 同一事件內各列的 `IncidentGrade` 是否一致
- [ ] 鎖定使用的 ATT&CK 版本

## 目前的實際放置位置

Kaggle 下載的檔案目前放在 `data/archive/`（已加入 `.gitignore`）：

| 檔案 | 大小 | 內容 |
|---|---|---|
| `GUIDE_Train.csv` | 2.4 GB | 訓練集 |
| `GUIDE_Test.csv` | 1.1 GB | 測試集，多一欄 `Usage`（Public / Private） |
| `GUIDE_Test_Queue_Rankings.csv` | 136 KB | 9,980 起測試事件、499 個組織的佇列優先順序（1–20），來自 AIP 論文 [arXiv:2607.16963](https://arxiv.org/abs/2607.16963) |

訓練集有 2.4 GB，`experiments/e0_profile_guide.py` 以每次 100 萬列分批讀取，一般筆電的記憶體即可處理。
