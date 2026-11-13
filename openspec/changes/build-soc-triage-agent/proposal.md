# Proposal：建置 AI SOC Analyst Agent

## Why

SOC 分析師被大量誤報與良性告警淹沒，真實攻擊因此被延誤或誤判。現有 ML 分級缺少攻擊鏈脈絡，LLM Agent 在證據不足時漏判至少 40.4% 的攻擊告警 [3]，且多數系統沒有「何時交給人」的設計，容易加劇自動化自滿 [12]。本變更建立一個對事件提供有依據、可解釋、會主動升級的分級 Agent（背景與研究缺口見 `docs/proposal.md` §2–§3，核心思維見 `docs/project-core.md`）。

## What Changes

- 新增事件分級請求的輸入與驗證：接受 Incident ID、告警批次或自然語言調查問題。
- 新增證據蒐集迴圈：依 Assume Breach（M1）擴大查詢，以規則判斷證據是否足夠，並受工具呼叫預算限制。
- 新增 TP / BP / FP 分級：GBDT 機率、SHAP 解釋與相似歷史事件。
- 新增 ATT&CK 對應：技術與戰術對應、攻擊鏈階段、高風險規則 R1–R5、NIST SP 800-61 Rev. 3 處置 playbook。
- 新增證據融合與升級決策：整合 ML、LLM 與規則，在信心不足、結論衝突、高風險或證據不足時標記「需人工複核」。
- 新增結構化事件報告（JSON + Markdown）與分析師回饋。
- 新增評估流程：E1–E5 與消融實驗，同時報告 TP 漏判率與人工升級率。

## Capabilities

### New Capabilities

- `triage-request`：分級請求的輸入型態、格式驗證與錯誤回應（DESIGN.md ②、FR1、FR6）。
- `evidence-gathering`：證據蒐集迴圈、證據足夠性判斷、工具預算與來源追溯（DESIGN.md ③④、M1、NFR1、NFR4）。
- `triage-classification`：TP / BP / FP 機率、SHAP 特徵解釋與相似事件（DESIGN.md ⑤、FR1、FR2）。
- `attack-mapping`：ATT&CK 技術與戰術對應、攻擊鏈階段、高風險規則與處置建議（DESIGN.md ⑥、FR3、FR4、M2、M4）。
- `triage-decision`：證據融合、衝突偵測與升級給人工的條件（DESIGN.md ⑦、FR5、NFR2、M3）。
- `triage-report`：事件報告內容、成本紀錄與分析師回饋（DESIGN.md ⑧、FR7、NFR1、NFR4、M5）。
- `triage-evaluation`：資料切分、baseline、評估指標與實驗可重現性（DESIGN.md §10、NFR3、T3）。

### Modified Capabilities

（無，本專題尚無既有規格。）

## Impact

- **程式**：新增 `src/soc_agent/`（schemas、agent、data、tools、analytics、knowledge、decision、report）與 `experiments/e1`–`e5`（對應 DESIGN.md §11）。
- **資料**：Microsoft GUIDE（CDLA-2.0）、ExCyTIn-Bench（需 Docker + MySQL，約 10 GB）、MITRE ATT&CK STIX 2.1。
- **相依套件**：pandas、DuckDB、LightGBM / XGBoost、SHAP、Chroma、mitreattack-python、LLM API、Streamlit。
- **成本**：LLM 實驗花費上限約 550 美元；以工具呼叫上限 B 與 token 預算控制。
- **文件**：DESIGN.md 改為指向本變更的 specs 與 design；README 加入 OpenSpec 工作流程說明。
