# Spec Delta：attack-mapping

## Purpose

定義系統如何把事件對應到 MITRE ATT&CK 技術、戰術與攻擊鏈階段，以高風險攻擊鏈規則標記危險事件，並依 NIST SP 800-61 Rev. 3 產生處置建議。

## ADDED Requirements

### Requirement: 對應 ATT&CK 技術與戰術
系統 SHALL 將事件對應到 ATT&CK Technique 與 Tactic，並推估攻擊鏈已推進到的最遠階段（FR3）。

#### Scenario: 事件含已標註技術
- **WHEN** 事件的告警含有 T1078 等技術代碼
- **THEN** 結果列出對應的 Technique、所屬 Tactic 與攻擊鏈階段

### Requirement: 檢索知識需引用來源
系統 SHALL 從 ATT&CK 知識庫與處置 playbook 檢索相關內容，且每段檢索內容 MUST 附上來源（例如技術 ID）。

#### Scenario: 檢索技術說明
- **WHEN** 事件對應到 Credential Access 相關技術
- **THEN** 系統取回該技術的描述、偵測方式與緩解措施，並標示技術 ID

### Requirement: 高風險攻擊鏈規則
系統 SHALL 在同一事件內符合任一規則時標記 high_risk：R1 出現 Exfiltration 或 Impact 戰術，或類別為 Ransomware；R2 同時出現 Credential Access 與 Lateral Movement；R3 Initial Access 與 Execution、Persistence、Privilege Escalation 任一同時出現；R4 出現 T1070；R5 涵蓋 4 個以上不同戰術。

#### Scenario: 憑證加橫向移動
- **WHEN** 事件同時對應到 Credential Access 與 Lateral Movement
- **THEN** 系統觸發 R2 並標記 high_risk

#### Scenario: 不要求先後順序
- **WHEN** 事件中 Persistence 的時間戳記早於 Initial Access
- **THEN** 系統仍觸發 R3，因為時間戳記可能有雜訊

#### Scenario: 清除痕跡
- **WHEN** 事件出現 T1070（Indicator Removal）
- **THEN** 系統觸發 R4 並標記 high_risk

### Requirement: 規則不單獨決定分級
高風險規則 MUST 只用於標記風險與偏向升級，MUST NOT 單獨決定最終分級。

#### Scenario: 觸發規則但 TP 機率很低
- **WHEN** 事件觸發 R5，但 P(TP) 低於安全門檻
- **THEN** 最終分級不因規則而改變，但事件仍帶有 high_risk 旗標

### Requirement: 處置建議
系統 SHALL 依最終分級與攻擊鏈階段，從依 NIST SP 800-61 Rev. 3 與 D3FEND 整理的 playbook 產生處置建議（FR4）。

#### Scenario: 帳號遭盜用
- **WHEN** 事件判為 TP，且包含有效帳號登入與資料外傳
- **THEN** 建議包含停用帳號與工作階段、保留證據、檢查同來源的其他帳號
