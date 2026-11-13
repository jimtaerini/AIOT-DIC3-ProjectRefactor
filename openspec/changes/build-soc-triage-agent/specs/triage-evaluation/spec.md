# Spec Delta：triage-evaluation

## Purpose

定義專題如何以可重現、避免資料洩漏的方式評估 Agent，並同時呈現 TP 漏判率與人工升級率的取捨，而不是只報告單一準確率。

## ADDED Requirements

### Requirement: 依事件切分資料
評估 MUST 以事件（`OrgId` + `IncidentId`）為單位切分訓練、驗證與測試集，同一事件的告警 MUST NOT 同時出現在不同集合。

#### Scenario: 檢查切分
- **WHEN** 產生資料切分
- **THEN** 任一事件鍵只出現在一個集合

### Requirement: 泛化切分
評估 SHALL 另外提供依組織與依時間切分的結果，以檢驗對新客戶與新時段的泛化能力（T3）。

#### Scenario: 依組織切分
- **WHEN** 執行組織切分評估
- **THEN** 測試集中的組織不出現在訓練集

### Requirement: 與 baseline 比較
分級評估 SHALL 至少與四個 baseline 比較：多數類別、Detector 歷史誤報率規則、只用 ML 模型、只用 LLM（無知識庫）。

#### Scenario: 執行 E1
- **WHEN** 執行分級準確度實驗
- **THEN** 結果表同時列出 Agent 與四個 baseline

### Requirement: 同時報告漏判率與升級率
評估 MUST 同時報告 Macro-F1、各類 Precision / Recall、TP 漏判率與人工升級率，並以不同門檻 τ 繪出漏判率–升級率曲線（M3、T1）。

#### Scenario: 呈現取捨
- **WHEN** 產生 E1 結果
- **THEN** 結果包含漏判率–升級率曲線，而不只是單一 F1

### Requirement: 消融實驗
評估 SHALL 分別移除 ML 分級、領域知識或升級規則，以量化各模組的貢獻（T2）。

#### Scenario: 移除領域知識
- **WHEN** 執行移除領域知識的消融實驗
- **THEN** 結果以與 E1 相同的指標呈現

### Requirement: 可重現
所有前處理、訓練與評估 MUST 以腳本執行並固定 random seed，結果 SHALL 報告多次執行的平均與標準差（NFR3）。

#### Scenario: 重新執行實驗
- **WHEN** 以相同 seed 重新執行同一實驗腳本
- **THEN** 得到相同的資料切分與模型結果
