# Spec Delta：triage-request

## Purpose

定義分析師可以向 Agent 提出的分級請求型態，以及系統如何驗證輸入、在格式錯誤時回應，確保後續流程只處理有效且一致的請求。

## ADDED Requirements

### Requirement: 接受三種請求型態
系統 SHALL 接受三種分級請求：單一 Incident ID、一批告警，或一個自然語言調查問題，並將其轉為統一的分級請求物件。

#### Scenario: 以 Incident ID 提出請求
- **WHEN** 分析師輸入一個存在的 Incident ID（以 `OrgId` + `IncidentId` 識別）
- **THEN** 系統建立一個型態為 incident 的分級請求，並進入證據蒐集

#### Scenario: 以告警批次提出請求
- **WHEN** 分析師提交一批告警
- **THEN** 系統建立一個型態為 alert batch 的分級請求，並保留每筆告警的識別碼

#### Scenario: 以自然語言問題提出請求
- **WHEN** 分析師輸入「這台主機在告警前後還有哪些可疑行為？」這類調查問題
- **THEN** 系統建立一個型態為 question 的分級請求，並保留原始問題文字

### Requirement: 拒絕無效輸入
系統 SHALL 在輸入缺少必要欄位或格式錯誤時，回傳說明缺少什麼的錯誤，且 MUST NOT 進入證據蒐集或分級。

#### Scenario: 缺少識別碼
- **WHEN** 請求既沒有 Incident ID、沒有告警，也沒有問題文字
- **THEN** 系統回傳錯誤 "missing or malformed input"，並要求分析師補充

#### Scenario: 找不到事件
- **WHEN** 分析師輸入的 Incident ID 在資料來源中不存在
- **THEN** 系統回傳找不到事件的錯誤，而不是產生空的分級結果
