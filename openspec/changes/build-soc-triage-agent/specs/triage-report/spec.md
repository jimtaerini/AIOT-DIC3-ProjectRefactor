# Spec Delta：triage-report

## Purpose

定義系統交給分析師的事件報告內容、每個事件的成本紀錄，以及分析師修正如何回到系統，讓每個結論都可被驗證而不是照單全收。

## ADDED Requirements

### Requirement: 結構化事件報告
系統 SHALL 對每個事件同時輸出 JSON 與 Markdown 報告，內容包含：分級、信心、狀態、證據鏈與來源、特徵貢獻、相似事件、ATT&CK 對應、處置建議、旗標與升級原因（FR7）。

#### Scenario: 產生報告
- **WHEN** 一個事件完成分級
- **THEN** 系統產生一份 JSON 與一份 Markdown，兩者內容一致

#### Scenario: 輸入無效
- **WHEN** 請求在驗證階段被拒絕
- **THEN** 系統只輸出錯誤報告，不包含分級欄位

### Requirement: 結論可追溯
報告中的每個結論 MUST 能追溯到至少一筆證據來源或知識來源（NFR1、M5）。

#### Scenario: 檢查 ATT&CK 對應
- **WHEN** 分析師查看報告中的某個 ATT&CK 技術
- **THEN** 報告指出該對應來自哪筆告警或證據

### Requirement: 記錄成本
系統 SHALL 為每個事件記錄 LLM token 用量、延遲與工具呼叫次數（NFR4）。

#### Scenario: 完成一個事件
- **WHEN** 事件報告產生
- **THEN** 報告包含 token 數、延遲秒數與工具呼叫次數

### Requirement: 分析師回饋
系統 SHALL 接受分析師對分級的修正，並立即將修正後的標籤加入相似事件的歷史庫；修正也 SHALL 被保存，供之後重新調整門檻與重新訓練模型。

#### Scenario: 分析師修正分級
- **WHEN** 分析師把一個 BP 事件改為 TP
- **THEN** 之後的相似事件檢索顯示 TP，且該修正被保存為訓練資料
