# 《生成式AI：文字與圖像生成的原理與實務》第十週作業繳交報告

> **課程編號**：國立臺灣大學 NTU COOL #63218 / #62123  
> **作業主題**：第十週作業：把你的工作拆成 AI 能做的流程 (AI Agents LLM Wiki & Teaching Deck)  
> **學生姓名**：劉正祥  
> **學號**：11341461  
> **提交日期**：2026 年 11 月 10 日  

---

## 📌 一、作業提交連結 (Submission URLs)

- **GitHub 專案儲存庫 (Repository)**：  
  👉 [https://github.com/034884851/llm-wiki-ai-agents](https://github.com/034884851/llm-wiki-ai-agents)

- **線上成果展示入口 (GitHub Pages 即開即看，免下載)**：  
  - 📖 **AI Agents LLM Wiki 結構化知識庫瀏覽器**：  
    👉 [https://034884851.github.io/llm-wiki-ai-agents/](https://034884851.github.io/llm-wiki-ai-agents/)
  - 🖥️ **15 頁自洽教學簡報 (Teaching Slide Deck)**：  
    👉 [https://034884851.github.io/llm-wiki-ai-agents/deck/](https://034884851.github.io/llm-wiki-ai-agents/deck/)

---

## 🎯 二、專案架構與成果總覽 (Project Overview)

本專案完全對齊第十週「把你的工作拆成 AI 能做的流程」之教學核心，落實了 Andrej Karpathy 提出的 **LLM Wiki「你負責讀，AI 負責寫（You Read, AI Writes）」** 範式，並建構了一套帶有條件檢查閘門與局部重試迴圈的自動化生成管線：

### 1. 三層結構 AI Agents LLM Wiki
- **攝取 6 份核心權威文獻**：
  1. Anthropic (2024)：《Building Effective Agents》
  2. Lilian Weng / OpenAI (2023)：《LLM Powered Autonomous Agents》
  3. Yao et al. / Princeton & Google (2022)：《ReAct: Synergizing Reasoning and Acting in Language Models》
  4. OpenAI (2025)：《A Practical Guide to Building Agents》
  5. Hugging Face (2025)：《Agents Course — Unit 1: What are Agents?》
  6. Anthropic / MCP Open Source (2024)：《Model Context Protocol (MCP) — Introduction》
- **12 大核心概念與 80 個錨點引用**：完整覆蓋 Agent vs Workflow、ReAct 思考-行動迴圈、Planning 分解、Memory 機制、Tool Use 協議、MCP Client-Server 架構等關鍵面向。

### 2. 帶條件檢查迴圈的 Graph Pipeline (9 Nodes, 5 Gates)
- **九大節點**：`Fetch` → `Ingest` → `Lint` → `Query` → `Outline` → `Draft` → `Fact-Check` → `Render` → `Validate`
- **五大門檻閘門 (C2 ~ C6)**：落實 **Evaluator-Optimizer** 評估優化迴圈，當某頁面事實檢驗未達標或出現幻覺時，系統精確針對單一節點執行局部重試，而非笨拙地全域重寫。
- **左移優化 (Shift-Left Validation)**：對照真實運行日誌，v2 相比 v1 減少了 39.5% 執行時間，將事實匹配率提升至 100%，徹底消除終端渲染故障。

### 3. 15 頁繁體中文自洽教學 HTML 簡報
- **純靜態零外部依賴**：100% 原生 HTML5 + CSS + JavaScript，支援離線預覽與 GitHub Pages 即時託管。
- **全方位互動功能**：全螢幕控制器、進度條、鍵盤翻頁（`←` / `→`）、字級微調、主題色切換，以及內建可展開之「Wiki 知識庫段落證據抽屜」。

---

## 📂 三、檔案結構索引 (File Structure)

```text
llm-wiki-ai-agents/
├── README.md                      # GitHub 專案首頁與完整操作導覽
├── index.html                     # AI Agents 結構化知識庫瀏覽器 (GitHub Pages 根目錄)
├── deck/
│   └── index.html                 # 15 頁教學簡報投影入口
├── docs/
│   ├── loop-graph.md              # 系統條件檢查迴圈與 Graph 拓撲架構圖
│   └── reflection.md              # v1 vs v2 運行日誌與左移優化反思報告
└── submission.md                  # NTU COOL 作業完整繳交報告
```

---

## 💡 四、總結與心得

本作業跳脫傳統「單向一次性生成（One-shot Generation）」的侷限，將生成式 AI 提升為具備「狀態維持」、「多步驟規劃」與「自我檢核迴圈」的 Agentic 系統。透過 GitHub 進行版本管理與 GitHub Pages 託管，完整呈現了從事實攝取到高質感互動教學的端到端實踐。
