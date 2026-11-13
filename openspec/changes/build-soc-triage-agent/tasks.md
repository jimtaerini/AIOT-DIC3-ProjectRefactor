# Tasks

> 依 6 週開發時程排列（W1–W6）。每項任務都寫出驗證方式；規格見 `specs/`，做法見 `design.md`。

## 1. 資料與前處理（W1）

- [x] 1.1 下載 GUIDE 並以 `experiments/e0_profile_guide.py` 完成資料檢查，驗證：`experiments/results/guide_train_profile.json` 存在且事件數為 707,108
- [ ] 1.2 確認「無此實體」的固定 ID 與同事件內標籤一致性，並更新 DESIGN.md §5 的待確認項目，驗證：兩項不再標示「（待確認）」
- [ ] 1.3 以 DuckDB 將 Evidence → Alert → Incident 聚合成事件層級資料，驗證：每個 `OrgId` + `IncidentId` 只有一列
- [ ] 1.4 依事件、組織、時間三種方式切分並固定 seed，驗證：切分測試確認任一事件鍵只出現在一個集合（`triage-evaluation`）

## 2. 請求與證據蒐集（W1–W2）

- [ ] 2.1 實作三種請求型態與輸入驗證，驗證：單元測試涵蓋 `triage-request` 的五個情境
- [ ] 2.2 實作事件證據庫，每筆證據附來源參照，驗證：測試確認所有證據都有資料表、欄位、列 ID
- [ ] 2.3 實作唯讀 log 查詢工具，驗證：含 `DELETE`、`UPDATE` 的查詢被拒絕
- [ ] 2.4 實作 `IsSufficient(E)` 三項規則與 GUIDE 版本的第 3 項，驗證：單元測試涵蓋 `evidence-gathering` 的足夠性與空查詢情境

## 3. 分級模型（W1–W2）

- [ ] 3.1 實作事件層級特徵（含 Tactic 覆蓋數、攻擊鏈最遠階段，排除 IP 與 hash），驗證：特徵清單測試不含 IP、hash 原始值
- [ ] 3.2 訓練 GBDT 三類分級並加 class weight，驗證：在驗證集輸出三類機率且總和為 1
- [ ] 3.3 加入 SHAP 前幾名特徵與 k 個相似事件檢索，驗證：每次分級結果都含特徵貢獻與相似事件
- [ ] 3.4 執行 E1 的多數類別、Detector 誤報率規則、純 GBDT baseline，驗證：產生 E1 baseline 結果表

## 4. 領域知識（W2–W3）

- [ ] 4.1 鎖定 ATT&CK STIX 版本，切塊存入 Chroma，驗證：以技術 ID 查詢可取回描述、偵測與緩解
- [ ] 4.2 實作 ATT&CK 對應與攻擊鏈階段推估，驗證：E2 在有 `MitreTechniques` 的告警上產生 Technique / Tactic Precision、Recall
- [ ] 4.3 實作高風險規則 R1–R5，驗證：單元測試涵蓋每條規則，且 R3 不要求先後順序
- [ ] 4.4 依 NIST SP 800-61 Rev. 3 與 D3FEND 整理處置 playbook，驗證：帳號遭盜用情境產生停用帳號、保留證據等建議

## 5. Agent 與決策（W3–W4）

- [ ] 5.1 實作 Planner 的 ReAct 迴圈、決策點與工具呼叫上限 B，驗證：用完預算時加上 evidence-insufficient 旗標並繼續分析
- [ ] 5.2 實作 `OneHotWithConfidence` 與 `FuseEvidence`（Algorithm 2），驗證：範例 (0.46, 0.45, 0.09) + TP 中信心得到 (0.58, 0.30, 0.12)
- [ ] 5.3 實作升級條件與升級原因（Algorithm 1 第 18–21 行），驗證：單元測試涵蓋 `triage-decision` 的所有情境
- [ ] 5.4 在驗證集上調整 w_ml、w_llm、τ、τ_safety，驗證：調參紀錄只引用驗證集

## 6. 報告與回饋（W4）

- [ ] 6.1 產生 JSON 與 Markdown 報告，含 token、延遲、工具呼叫次數，驗證：兩種格式內容一致的測試通過
- [ ] 6.2 實作分析師回饋寫入相似事件歷史庫與訓練資料，驗證：修正後的標籤出現在下一次相似事件檢索

## 7. 實驗與交付（W5–W6）

- [ ] 7.1 完成 E1（含純 LLM baseline）並繪製漏判率–升級率曲線，驗證：結果含曲線圖與多次執行的平均、標準差
- [ ] 7.2 完成 E5 成本統計，驗證：報告每事件 token、延遲、工具呼叫次數
- [ ] 7.3 建置 ExCyTIn-Bench 並完成 E3（第 5 週無法運作則改用 BOTS v3），驗證：產生 benchmark reward
- [ ] 7.4 抽樣 50 個事件完成 E4 報告品質評分，驗證：評分表含正確性、證據完整性、可執行性
- [ ] 7.5 完成消融實驗（移除 ⑤、⑥、升級規則），驗證：結果以 E1 指標呈現
- [ ] 7.6 建置 Streamlit Dashboard，驗證：可顯示事件佇列、報告與 SHAP 圖
- [ ] 7.7 更新 README、DESIGN.md 的 How to Run 與 Results，驗證：新成員照 README 可重跑 E1

## Workflow follow-up

- 全部任務完成且通過審查後，執行 `openspec archive build-soc-triage-agent`，把 specs 合併到 `openspec/specs/`。
- 封存後以 `openspec validate --specs --strict` 確認主規格有效。
