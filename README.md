# AI SOC Analyst Agent

**A Domain AI Agent for Next-Gen SOC** — *Smarter Triage. Safer Enterprise.*

本專題提出 AI SOC Analyst Agent，結合機器學習（GBDT + SHAP）、資安領域知識（MITRE ATT&CK、NIST SP 800-61 Rev. 3）與 LLM 推理（ReAct），對資安告警提供有依據、可解釋的分級（TP / BP / FP）。證據不足、資產關鍵性高或判斷不確定時，系統會主動將告警升級給人類分析師。

![AI SOC Analyst Agent 資安專題總覽](myplan/2.AI%20SOC%20Analyst%20Agent%20資安專題總覽.png)

## 核心文件

| 文件 | 內容 |
|---|---|
| [docs/project-core.md](docs/project-core.md) | 一頁整理：Abstract A–F、5 個核心思維（M1–M5）、3 個取捨（T1–T3）、10 個深度問題 |
| [docs/proposal.md](docs/proposal.md) | 完整 Proposal；Introduction 依 A2Z G–L、Related Work 依 M–O 重構 |
| [DESIGN.md](DESIGN.md) | 系統設計文件（v0.8），已轉換為下方的 OpenSpec 變更 |

## OpenSpec 工作流程（Spec-Driven Development）

本專題以 [OpenSpec](https://github.com/Fission-AI/OpenSpec) 管理規格：先寫規格（WHAT），再寫設計（HOW）與任務，最後實作並封存。

```
openspec/
├── config.yaml                          # 專案脈絡與語言設定
├── specs/                               # 封存後的主規格（目前為空）
└── changes/build-soc-triage-agent/      # 由 DESIGN.md 轉換的變更
    ├── proposal.md                      # Why / What Changes / 7 個 Capabilities
    ├── specs/<capability>/spec.md       # Requirement + Scenario（WHEN / THEN）
    ├── design.md                        # 架構、流程圖、Algorithm 1–2、決策與風險
    └── tasks.md                         # W1–W6 實作任務（含驗證方式）
```

| Capability | 對應 DESIGN.md | 核心思維 |
|---|---|---|
| `triage-request` | ② Input、FR1、FR6 | — |
| `evidence-gathering` | ③④ Planner / Retrieval | M1 |
| `triage-classification` | ⑤ Analytics | M2、M4 |
| `attack-mapping` | ⑥ Domain Knowledge、R1–R5 | M2、M4 |
| `triage-decision` | ⑦ Evidence Fusion & Escalation | M3、M5 |
| `triage-report` | ⑧ Output & Feedback | M5 |
| `triage-evaluation` | §10 Evaluation Plan | T1–T3 |

常用指令（需先安裝 Node.js 與 `npm install -g @fission-ai/openspec`）：

```bash
openspec list                                       # 列出進行中的變更與任務進度
openspec show build-soc-triage-agent                # 檢視變更內容
openspec validate build-soc-triage-agent --strict   # 驗證規格格式
openspec archive build-soc-triage-agent             # 任務完成後封存並合併到主規格
```

## 原始規劃文件

[`myplan/`](myplan/) 資料夾：

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
