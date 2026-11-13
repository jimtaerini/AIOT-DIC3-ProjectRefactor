# Spec Delta：evidence-gathering

## Purpose

定義 Agent 如何依 Assume Breach 原則蒐集事件證據、判斷證據是否足夠，以及在預算內停止，避免 LLM 在「查無紀錄」時過早結案。

## ADDED Requirements

### Requirement: 依輸入型態選擇資料來源
系統 SHALL 依請求型態選擇資料來源：incident 與 alert batch 從事件證據庫取得證據；question 透過唯讀 log 查詢取得證據。

#### Scenario: 事件請求使用證據庫
- **WHEN** 請求型態為 incident
- **THEN** 系統從事件證據庫取得該事件的所有告警與證據

#### Scenario: 調查問題使用 log 查詢
- **WHEN** 請求型態為 question
- **THEN** 系統產生並執行 log 查詢來回答問題

### Requirement: log 查詢必須唯讀
系統 MUST 只執行唯讀查詢，MUST NOT 修改、刪除或新增任何 log 資料。

#### Scenario: 查詢含寫入語句
- **WHEN** 產生的查詢包含 `SELECT` 以外的語句（例如 `DELETE`、`UPDATE`）
- **THEN** 系統拒絕執行該查詢，並將其記錄為失敗的工具呼叫

### Requirement: 證據足夠性以規則判斷
系統 SHALL 以規則判斷證據是否足夠，MUST NOT 交由 LLM 自行決定。三項全部成立才算足夠：每筆告警都有對應證據；主要實體（帳號、主機）都至少有一筆上下文紀錄；任何空查詢都已確認同時間窗 log 未中斷。

#### Scenario: 證據齊全
- **WHEN** 三項條件全部成立
- **THEN** 系統停止蒐證並進入分級

#### Scenario: 缺少帳號上下文
- **WHEN** 事件涉及的某個帳號沒有任何登入、程序或網路紀錄
- **THEN** 系統判定證據不足，並在下一輪查詢該帳號的登入紀錄

#### Scenario: 調查問題沒有告警清單
- **WHEN** 請求型態為 question
- **THEN** 第一項條件改為「問題涉及的每個實體都已查詢過」

### Requirement: 查無紀錄不得視為無害
系統 MUST 把空查詢結果視為需要確認的訊號；在無法確認 log 未中斷前，MUST NOT 將其解讀為「沒有可疑活動」（M1 Assume Breach）。

#### Scenario: 查詢回傳空結果
- **WHEN** 某次查詢回傳零筆紀錄
- **THEN** 系統先查詢同一資料來源在該時間窗內是否有其他紀錄

#### Scenario: log 已中斷
- **WHEN** 同一資料來源在該時間窗內沒有任何紀錄
- **THEN** 系統將證據標記為不足，而不是判定為無可疑活動

### Requirement: 依攻擊鏈擴大查詢範圍
系統 SHALL 在已對應到橫向移動等擴散型技術時，把相關主機納入下一輪查詢。

#### Scenario: 出現 Lateral Movement
- **WHEN** 目前證據已對應到 Lateral Movement 戰術
- **THEN** 下一輪查詢包含橫向移動的目標主機

### Requirement: 工具呼叫預算
系統 SHALL 對每個事件設定工具呼叫上限 B；用完預算仍不足時，MUST 標記「證據不足」並繼續分析，而不是無限查詢。

#### Scenario: 用完預算
- **WHEN** 工具呼叫次數達到 B 且證據仍不足
- **THEN** 系統停止蒐證、在結果加上 evidence-insufficient 旗標，並繼續分級

### Requirement: 證據附來源
系統 MUST 為每一筆證據附上來源（資料表、欄位、列 ID），讓結論可以追溯（NFR1）。

#### Scenario: 產生證據
- **WHEN** 任一工具回傳證據
- **THEN** 每筆證據都帶有可定位的來源參照
