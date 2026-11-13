# Spec Delta：triage-decision

## Purpose

定義系統如何融合 ML 機率、LLM 判斷與攻擊鏈規則產生最終分級，以及何時必須把事件升級給人工複核，以實作錯誤成本不對稱與人機協作。

## ADDED Requirements

### Requirement: 融合 ML 與 LLM 判斷
系統 SHALL 將 ML 三類機率與 LLM 的判斷加權融合，取分數最高的類別為分級、最高分數為信心。LLM 信心 MUST 只以「低／中／高」三級回報，再轉為機率向量。

#### Scenario: 加權融合
- **WHEN** ML 輸出 (0.46, 0.45, 0.09)，LLM 判斷為 TP、信心「中」，權重各 0.5
- **THEN** 融合結果為 (0.58, 0.30, 0.12)，分級 TP、信心 0.58

### Requirement: 偵測結論衝突
系統 SHALL 在 ML 最高機率類別與 LLM 判斷不同時，加上 conflict 旗標並說明雙方的結論。

#### Scenario: ML 判 BP、LLM 判 TP
- **WHEN** ML 最高機率為 BP，LLM 判斷為 TP
- **THEN** 結果帶有 conflict 旗標，並列出雙方結論與理由

### Requirement: 高風險時偏向 TP
系統 SHALL 在事件帶有 high_risk 旗標、融合分級不是 TP、且 P(TP) 不低於安全門檻 τ_safety 時，將分級改為 TP（M3、NFR2）。

#### Scenario: 高風險事件改判
- **WHEN** 事件觸發 R1，融合分級為 BP，且 P(TP) ≥ τ_safety
- **THEN** 最終分級改為 TP

### Requirement: 升級給人工複核
系統 MUST 在下列任一條件成立時，把狀態設為 NEEDS_HUMAN_REVIEW：信心低於門檻 τ、帶有 conflict 旗標、帶有 high_risk 旗標、或證據不足。四項都不成立時才可設為 AUTO_TRIAGED（FR5）。

#### Scenario: 信心不足
- **WHEN** 融合信心為 0.58，低於門檻 τ
- **THEN** 狀態為 NEEDS_HUMAN_REVIEW

#### Scenario: 證據不足
- **WHEN** 蒐證用完預算 B 仍帶有 evidence-insufficient 旗標
- **THEN** 狀態為 NEEDS_HUMAN_REVIEW，即使信心高於門檻

#### Scenario: 全部條件通過
- **WHEN** 信心 ≥ τ、無 conflict、無 high_risk、證據足夠
- **THEN** 狀態為 AUTO_TRIAGED

### Requirement: 升級需說明原因
系統 SHALL 對每個 NEEDS_HUMAN_REVIEW 的事件列出觸發升級的所有條件（M5）。

#### Scenario: 多個條件同時觸發
- **WHEN** 事件同時信心不足、觸發 R2、且缺少登入紀錄
- **THEN** 升級原因列出這三項

### Requirement: 門檻只在驗證集調整
權重 w_ml、w_llm 與門檻 τ、τ_safety MUST 只在驗證集上調整，MUST NOT 使用測試集。

#### Scenario: 調整門檻
- **WHEN** 團隊調整 τ
- **THEN** 調整紀錄只引用驗證集的結果
