# Spec Delta：triage-classification

## Purpose

定義系統如何依事件特徵產生 TP / BP / FP 三類機率，並以特徵貢獻與相似歷史事件說明判斷依據，讓分級不只依賴 LLM。

## ADDED Requirements

### Requirement: 輸出三類機率
系統 SHALL 對每個事件輸出 P(TP)、P(BP)、P(FP) 三個機率，且三者總和為 1。分級機率 MUST 來自以分析師標註事件訓練的模型，而不是由 LLM 單獨產生。

#### Scenario: 對事件分級
- **WHEN** 證據蒐集完成
- **THEN** 系統輸出三類機率，例如 (0.46, 0.45, 0.09)

### Requirement: 以事件為分析單位
系統 SHALL 以事件為單位擷取特徵，把證據與告警聚合成事件層級，並包含 ATT&CK 戰術覆蓋數與攻擊鏈最遠階段（M2）。

#### Scenario: 多筆告警屬於同一事件
- **WHEN** 一個事件有 6 筆告警、20 筆證據
- **THEN** 系統只產生一組事件層級的特徵與一組機率

### Requirement: 避免依賴易替換指標
系統 MUST NOT 以 IP 位址、檔案 hash 等容易替換的指標作為分級特徵（M4）。

#### Scenario: 檢查特徵清單
- **WHEN** 檢視模型使用的特徵
- **THEN** 清單中不含 IP 位址或檔案 hash 的原始值

### Requirement: 特徵貢獻解釋
系統 SHALL 對每次分級列出影響最大的特徵及其貢獻方向，作為分析師可讀的證據。

#### Scenario: 產生解釋
- **WHEN** 系統完成一次分級
- **THEN** 結果包含前幾名特徵，例如「Tactic 覆蓋數」與「Detector 歷史誤報率」及其貢獻

### Requirement: 相似歷史事件
系統 SHALL 為每個事件找出 k 個最相似的歷史事件及其標籤，並納入分析師修正後的標籤。

#### Scenario: 找到相似事件
- **WHEN** 系統完成一次分級
- **THEN** 結果列出 k 個相似事件的識別碼與標籤

#### Scenario: 分析師曾修正相似事件
- **WHEN** 某個相似事件的標籤曾被分析師修正
- **THEN** 系統顯示修正後的標籤
