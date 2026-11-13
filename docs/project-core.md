# Project Core：A–F、5 Cores、3 Trade-offs、10 Details

> 本文件把專題的核心整理成一頁，作為 OpenSpec 規格與 Introduction / Related Work 的共同依據。
> 文獻編號（例如 [3]）對應 [`proposal.md`](proposal.md) §6 References；完整論述見 `proposal.md` §1、§4。

**一句話定位**：AI SOC Analyst Agent 結合 ML 分級、MITRE ATT&CK 領域知識與 LLM 推理，對資安事件提供**有依據、可解釋、知道何時該交給人**的分級（TP / BP / FP）與處置建議。

---

## 1. Abstract A–F

| 字母 | 一句話 | 關鍵依據 |
|---|---|---|
| **A. Attention Getter** | 攻擊越來越多、偵測越來越慢，偵測與回應速度直接決定損失。 | 國安局：2025 年日均 263 萬次侵擾 [23]；IBM 2026：外洩平均成本 499 萬美元、247 天 [14] |
| **B. But** | SOC 被誤報淹沒；ML 不懂攻擊鏈，LLM Agent 漏判多且會過早結案，自動化又讓人自滿。 | LLM 推理策略漏判 ≥ 40.4% [3]；benchmark 最佳 reward 0.606 [4]；自動化自滿 [12] |
| **C. Cure** | 提出 AI SOC Analyst Agent：ML + ATT&CK 知識 + LLM 推理，並有明確的升級機制。 | — |
| **D. Development** | ReAct LLM 規劃與蒐證 → GBDT 預測 TP/BP/FP + SHAP → RAG 檢索 ATT&CK 與 NIST SP 800-61 Rev. 3 → 證據融合與升級（Algorithm 1、2）。 | [1][6][13] |
| **E. Experiments** | GUIDE（約 100 萬起真實事件）與 ExCyTIn-Bench；比較多數類別、Detector 誤報率規則、純 GBDT、純 LLM；E1–E5 + 消融。 | [1][4] |
| **F. Findings（預期）** | TP 漏判率低於純 GBDT / 純 LLM；量化「漏判率–升級率」取捨；ATT&CK 檢索提升對應正確性與可解釋性。 | 尚未實驗，期末報告呈現 |

---

## 2. 5 Cores：五個核心思維模式

每個思維模式都必須落到一個可驗證的設計決策（對應的 OpenSpec 規格見最右欄）。

| # | 思維模式 | 一句話意義 | 設計決策 | OpenSpec 規格 |
|---|---|---|---|---|
| **M1** | **Assume Breach** | 「查無紀錄」不等於「沒有攻擊」。 | 查無紀錄時擴大查詢；用完預算仍不足就升級；`IsSufficient` 由規則判斷而非 LLM。 | `evidence-gathering` |
| **M2** | **事件與攻擊鏈脈絡** | 單一告警意義有限，要看它在攻擊鏈的位置。 | 以事件為分析單位；Tactic 覆蓋與攻擊鏈階段作為特徵與高風險規則 R1–R5。 | `attack-mapping`、`triage-classification` |
| **M3** | **錯誤成本不對稱** | 漏判比誤報昂貴，但誤報太多也會害人漏看。 | TP 漏判率為首要指標；高風險時偏向 TP；同時報告升級率。 | `triage-decision` |
| **M4** | **看 TTPs 而非 IOCs** | IP、hash 容易換，攻擊手法難換（Pyramid of Pain）。 | 以 ATT&CK 技術為知識單位；特徵避免依賴 IP、hash。 | `attack-mapping`、`triage-classification` |
| **M5** | **自動化要可驗證** | 分析師要能檢查 AI 的依據，否則只會更自滿。 | 每個結論附證據鏈與來源；衝突明確標示；「需人工複核」狀態。 | `triage-report`、`triage-decision` |

---

## 3. 3 Trade-offs：三個專業爭論與本專題立場

| # | 取捨 | 一端 | 另一端 | 本專題立場 | 如何量化 |
|---|---|---|---|---|---|
| **T1** | **自動化 ↔ 人機協作** | 全自動 AI SOC：告警量超過人力，只有自動化跟得上 [14][1][7] | AI 只輔助：LLM 漏判 ≥ 40.4% [3]、自動化造成自滿 [12] | ML 處理大量高信心事件；高風險或不確定就升級給人 | **TP 漏判率 vs. 人工升級率**取捨曲線（E1） |
| **T2** | **規則 ↔ 機器學習** | 規則透明可稽核、可對應 ATT&CK [10][11][19] | ML 能學到人寫不出的模式，已大規模部署 [1][7] | 混合式：GBDT 給機率，Kill Chain 規則作為硬性約束，LLM 整合與解釋 | 消融實驗：移除 ⑤、⑥ 或升級規則 |
| **T3** | **公開資料 ↔ 真實 SOC** | 公開資料是可重現研究的基礎 [1][4] | 標註錯誤、取樣偏差、資料洩漏會高估結果 [16][17] | GUIDE（真實標籤）+ ExCyTIn（完整 log）交叉驗證；依事件與組織切分 | 依事件 / 組織 / 時間三種切分比較 |

---

## 4. 10 Details：十個深度理解問題

> 用來分辨「真正理解」與「死記硬背」。每題的重點答案見 `proposal.md` §4 Q3。

| # | 類型 | 問題 | 答題重點（一句話） | 對應 |
|---|---|---|---|---|
| 1 | Why | 某 Detector 90% 告警是 FP，為什麼不直接關掉？ | 會失去偵測覆蓋；先分清 FP 還是 BP，再決定調校、白名單或改事件層級關聯。 | T2 |
| 2 | What-if | 攻擊者知道 Agent「查無紀錄就結案」會怎麼做？ | 刪 log（T1070）；log 中斷本身就是訊號，應升級而非結案。 | M1、R4 |
| 3 | Trade-off | 全部升級漏判率就是 0，為什麼不好？ | 人力有限，升級太多會造成告警疲勞；依漏判率–升級率曲線選門檻 τ。 | M3、T1 |
| 4 | Failure Case | 高信心判為 BP 的事件其實是 Living off the Land 攻擊，為什麼失敗？ | 合法工具的行為與管理員相同；差別在意圖與上下文，需攻擊鏈脈絡與基線行為。 | M2 |
| 5 | Evidence | 怎麼證明 Agent 真的對 SOC 有幫助？ | 除了分數，要看漏判率、升級率、跨資料集、報告品質，並檢查是否造成自滿。 | T3、M5 |
| 6 | Comparison | TP、BP、FP 差在哪？為什麼要分 BP 與 FP？ | FP 要改規則，BP 要建例外；多數「誤報」其實是 BP [24]。 | — |
| 7 | Design Decision | 為什麼分級主力是 GBDT 而不是 LLM？ | GUIDE 欄位匿名化，LLM 讀不出語意；GBDT 穩定、可校準、可用 SHAP 解釋。 | T2 |
| 8 | Real-world Scenario | 凌晨海外登入 → 建郵件轉寄規則 → 大量下載 SharePoint，怎麼處置？ | Initial Access → Persistence/Collection → Exfiltration，觸發 R1，判高風險 TP 並升級；停用帳號、移除規則、保留證據。 | M2、R1 |
| 9 | Why | ATT&CK 為什麼分 Tactic 與 Technique？ | Tactic 是目的、反映攻擊階段（判嚴重度）；Technique 是方法（用於偵測與防禦對應）。 | M2、M4 |
| 10 | Evidence / Design | GUIDE 隨機切 train/test 會出什麼問題？ | 同事件、同組織資料同時出現在兩邊造成洩漏；應依事件、組織或時間切分。 | T3 |
