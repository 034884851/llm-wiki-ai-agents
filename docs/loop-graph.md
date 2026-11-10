# 條件檢查迴圈與 Graph 拓撲架構圖 (Loop Graph Specification)

本文件詳細記錄國立臺灣大學 NTU COOL「生成式AI：文字與圖像生成的原理與實務」第十週作業（W10）所實作之 **帶條件檢查與局部重試迴圈的 Graph Pipeline**。

---

## 一、系統拓撲流程圖 (Mermaid Architecture)

```mermaid
flowchart TD
    Start([開始 Start]) --> N1[Node 1: Fetch 原文獲取]
    N1 --> N2[Node 2: Ingest 知識結構化]
    N2 --> C2{Gate C2: 格式與結構檢核}
    
    C2 -- 未通過 (重試) --> N2
    C2 -- 通過 --> N3[Node 3: Markdown Lint 語法清理]
    N3 --> N4[Node 4: Wiki Query 概念檢索]
    N4 --> C3{Gate C3: 覆蓋度檢核 (12概念/6來源)}
    
    C3 -- 缺失詞條 --> N2
    C3 -- 通過 --> N5[Node 5: Outline 投影片大綱生成]
    N5 --> C4{Gate C4: 頁數與篇章節奏檢核}
    
    C4 -- 節奏不均 --> N5
    C4 -- 通過 --> N6[Node 6: Draft 投影片內文撰寫]
    N6 --> N7[Node 7: Fact-Check 事實與引註錨點校驗]
    N7 --> C5{Gate C5: 幻覺率與錨點自洽檢核}
    
    C5 -- 發現幻覺/無效錨點 --> N6
    C5 -- 通過 --> N8[Node 8: Render 自洽 HTML 簡報渲染]
    N8 --> N9[Node 9: HTML Lint & Validate DOM/CSS 測試]
    N9 --> C6{Gate C6: 語意標籤與無死鏈驗證}
    
    C6 -- DOM結構錯誤 --> N8
    C6 -- 驗證通過 --> End([交付 Complete])
```

---

## 二、節點職責說明 (Nodes Description)

| 節點名稱 | 階段職責 | 輸入 | 輸出 | 核心依賴模型/規則 |
| :--- | :--- | :--- | :--- | :--- |
| **Node 1: Fetch** | 下載並解析 6 篇權威文獻原文 (PDF / Markdown / HTML) | 論文 URL 與文獻清單 | Raw 文獻文本 | 外部網路抓取模組 |
| **Node 2: Ingest** | 依據 Karpathy LLM Wiki 結構，切分段落並注入唯一錨點標籤 (`^p-*`) | Raw 文獻 | 結構化 Markdown 詞條庫 | 正規表示式與段落切分器 |
| **Node 3: Lint** | 移除控制字元、修正標點符號與雙向鏈接格式 | 詞條庫初稿 | 規範化 Wiki 詞條 | Markdown AST 解析器 |
| **Node 4: Query** | 抽取 12 大核心概念與關聯錨點，建立教學檢索向量 | Wiki 知識庫 | 檢索上下文 (Context) | BM25 + 語意倒排索引 |
| **Node 5: Outline** | 規劃 15 頁自洽教學簡報架構（從直觀比喻到底層協定） | 檢索上下文與課程大綱 | 15 頁大綱 JSON | 大語言模型 (Generator) |
| **Node 6: Draft** | 逐頁填寫講義內文、比喻、ASCII 架構圖與 Wiki 雙向證據抽屜 | 投影片大綱 | 投影片結構化數據 | 大語言模型 (Generator) |
| **Node 7: Fact-Check** | 逐一對照 Wiki 段落錨點，執行嚴格事實校驗（反幻覺） | 投影片初稿與 Wiki 錨點 | 事實評估報告 | Evaluator-Optimizer 評估器 |
| **Node 8: Render** | 將結構化數據渲染為單一檔案自洽 HTML5 簡報（內嵌 CSS/JS） | 投影片審定稿與樣式模板 | `index.html` 簡報檔案 | Jinja2 / 純字串模板引擎 |
| **Node 9: Validate** | 執行 Headless 渲染、DOM 完整性檢測與死鏈自動化測試 | 渲染後 HTML | 測試報告與日誌 | 自動化驗證套件 |

---

## 三、五大條件檢查閘門 (Quality Gates C2 ~ C6)

1. **Gate C2 (Ingest Format Gate)**：
   - 檢核段落是否皆具備 `^p-[topic]-[index]` 格式之唯一錨點。
   - 檢核 6 篇來源標籤（`[src-01]` ～ `[src-06]`）是否齊全。
2. **Gate C3 (Coverage Gate)**：
   - 檢核 12 大關鍵概念（Workflow、Agent、ReAct、Tool Use、Memory、Planning、MCP 等）是否均有對應知識庫條目。
   - 若有缺漏，觸發 Feedback Loop 回退至 Ingest 階段重新補充。
3. **Gate C4 (Rhythm Gate)**：
   - 檢核投影片頁數是否嚴格符合 15 頁。
   - 檢核「概念篇」、「架構篇」、「協定篇」與「案例篇」比例是否平衡。
4. **Gate C5 (Fact-Check & Anti-Hallucination Gate)**：
   - **核心反幻覺機制**：每一頁簡報所引用的文字論述，必須在 LLM Wiki 知識庫中找到精確的 `^p-*` 錨點與來源文獻。
   - 若 Evaluator 偵測到無法追溯的數值或虛構功能，判定不通過，強制局部重新生成該頁。
5. **Gate C6 (DOM & Dead Link Gate)**：
   - 檢核全域鍵盤監聽（`←` / `→` / `Space`）翻頁功能是否正常。
   - 檢核全螢幕切換按鈕、進度條刻度與外部超連結是否具備 `rel="noopener noreferrer"`。
