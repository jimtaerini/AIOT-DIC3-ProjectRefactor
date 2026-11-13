# AI SOC Analyst Agent

**A Domain AI Agent for Next-Gen SOC** — *Smarter Triage. Safer Enterprise.*

本專題提出 AI SOC Analyst Agent，結合機器學習（GBDT + SHAP）、資安領域知識（MITRE ATT&CK、NIST SP 800-61 Rev. 3）與 LLM 推理（ReAct），對資安告警提供有依據、可解釋的分級（TP / BP / FP）。證據不足、資產關鍵性高或判斷不確定時，系統會主動將告警升級給人類分析師。

![AI SOC Analyst Agent 資安專題總覽](myplan/2.AI%20SOC%20Analyst%20Agent%20資安專題總覽.png)

## 專題文件

所有文件都放在 [`myplan/`](myplan/) 資料夾：

| 文件 | 內容 |
|---|---|
| [1. AI SOC 代理分析](myplan/1.AI%20SOC%20代理分析.md) | 依《期末專題提案作業規範》與 A2Z 26 字母 SOP 對照解析專題架構 |
| [2. AI SOC Analyst Agent 資安專題總覽](myplan/2.AI%20SOC%20Analyst%20Agent%20資安專題總覽.md) | 專題總覽圖（[PNG](myplan/2.AI%20SOC%20Analyst%20Agent%20資安專題總覽.png)）與其文字版（架構、流程、演算法、評估） |
| [3. AI SOC 代理摘要](myplan/3.AI%20SOC%20代理摘要.md) | A–F 六段式摘要（約 500 字）與學術定位 |

## 系統架構（8 大模組）

1. User/Scenario
2. Query/Input
3. Planner & Reasoning Agent
4. Data Acquisition/Retrieval
5. Analytics Engine
6. Domain Knowledge Integration
7. Decision/Recommendation
8. Output & Interaction

## 實驗規劃

- **資料集**：Microsoft GUIDE、ExCyTIn-Bench
- **基線**：多數類別、歷史誤報率規則、純 GBDT、純 LLM
- **評估指標**：Macro-F1、TP 漏判率、人工升級率
