# ICEIB 2027 — An Explainable Freeway Congestion Prediction Agent

> **Paper Title**: *An Explainable Freeway Congestion Prediction Agent Grounded in Real-World ETC Data and Official Traffic Regulations*  
> **中文題目**：可解釋的國道壅塞預測 Agent（聚焦台灣國道 5 號主線）  
> **Implementation & Experiments Repository**: <https://github.com/nns2qgdq/T1-highway-congestion-agent>

本儲存庫用於管理 ICEIB 2027 會議論文之 LaTeX 原始碼與參考資料。

---

## 研究簡介

本研究旨在將國道車速預測轉化為具明確依據的行前資訊，回應「預測準確度未必等同於出行建議有效性」的落差。為降低大型語言模型（LLM）產生數值幻覺與遺漏限制的風險，系統採用 **Tool-First** 設計，由程式端負責資料清理、車速預測、條件式機率校準、管制公告檢索與出發時機規則計算，LLM 僅負責自然語言意圖解析與說明生成，並透過確定性數字核對（`check_numbers`）、必要內容檢查（`required_missing`）與固定模板回退機制確保回答可追溯。

- **資料來源**：交通部高速公路局 TDCS M05A 門架 5 分鐘車速與流量資料，以及即時 `ETagPairLive.xml` / `LiveEvents.xml`（透過 `-3` 分鐘對齊規則驗證 3,418 筆 100% 一致）。
- **研究範圍**：國道 5 號主線 10 個區間（南向 5、北向 5），預測未來 15 / 30 / 60 分鐘車速與壅塞風險（自訂門檻 $< 40\text{ km/h}$）。
- **主要實測結論**（2024 訓練、2025 全年測試、2026 年 311 天嚴格樣本外評估）：
  - **車速誤差（MAE）**：上班日（$-0.36$ 至 $-0.75\text{ km/h}$）與一般休息日（$-0.31$ 至 $-0.70\text{ km/h}$）穩定優於持續預測（Persistence）；但連假改善幅度小且跨年度不穩定（2026 年 $+60$ 分鐘為 $-0.04\text{ km/h}$，95% CI 含 0）。
  - **事件層級預警**：於固定警報量（0.25 / 0.50 / 1.00 次/區間/日）下，模型預警能力相較持續預測無明顯優勢；另訓之 Onset 模型未達事前約定上線標準，故未接進 Agent。
  - **機率校準與延遲穩健性**：H1 2025 擬合之條件式 Isotonic 校準於 H2 2025 達標（高風險區預測與實際差距皆 $\le 0.10$）；模擬 0–15 分鐘資料延遲下，GBM 仍優於零延遲持續預測。
  - **Agent 端到端驗證**：在 72 題凍結與即時測試案例（每題 3 次）中，拒答／追問、區間時間解析、數字可追溯性、公告引用、規則一致與連假提醒均達 **100%**（Batch 1 中 14.3% 之 LLM 草稿由核對機制攔截並安全退回固定模板）。

---

## 目錄結構

```text
.
├── ICEIB_paper/
│   └── main.tex                                # IEEEtran 會議論文完整 LaTeX 原始碼（內嵌 TikZ 向量圖）
├── myRef/
│   ├── Abstract.md                             # 中英文摘要與關鍵字定稿
│   ├── A2Z outline example.pdf                 # A–Z 論文寫作架構 SOP
│   ├── 期末專題提案作業規範.pdf                  # 專題規範與評量指標
│   ├── 國道預測 Agent.pdf                       # 期中報告 A–Z 對照分析
│   ├── 可解釋國道壅塞預測 Agent 資訊圖表.png      # 系統架構與評估摘要圖表
│   └── github-url.txt                          # 實作與實驗專案連結
├── .gitignore                                  # 排除 T1-highway-congestion-agent-main/、.DS_Store 與 LaTeX 暫存檔
└── README.md
```

> 註：實作程式碼、訓練模型、40 項單元測試、原始實驗結果（`results/*.json`）與 SHA-256 凍結評估案例（`eval/`）皆獨立託管於 [T1-highway-congestion-agent](https://github.com/nns2qgdq/T1-highway-congestion-agent)，本儲存庫透過 `.gitignore` 排除本地副本 `T1-highway-congestion-agent-main/`。

---

## 論文內容與架構（`ICEIB_paper/main.tex`）

採用標準 `\documentclass[conference]{IEEEtran}` 格式，純 ASCII 編碼且無外部圖檔依賴：

- **Section I — Introduction**：研究動機（2026 春節交通量 $1.3\times$ 平日）、問題陳述、研究目標與三大貢獻。
- **Section II — Related Work**：涵蓋傳統統計/ML、深度時空模型、LLM 交通 Agent 與高公局實務對照（**Table I**）及研究缺口。
- **Section III — Proposed Scheme**：五模組三層架構圖（**Fig. 1**，TikZ）、三階段驗證與模板回退流程圖（**Fig. 2**，TikZ）、數學模型（**Eq. 1–7**）與帶行號演算法（**Algorithm 1**）。
- **Section IV — Experiments and Discussion**：實驗環境、資料集切分、評估指標、車速 MAE（**Table II**）、固定警報量事件偵測（**Table III**）、機率校準與延遲分析（**Table IV**）、72 題 Agent 評估（**Table V**）與六項研究限制。
- **Section V — Conclusion & References**：總結、未來工作與 15 篇已查證參考文獻。

---

## 編譯方式

### 1. Overleaf（推薦）
- 將 `ICEIB_paper/main.tex` 上傳至 Overleaf 專案根目錄（或透過 MCP 同步），選擇 **`pdfLaTeX`** 編譯器即可直接編譯出 PDF（已於 Overleaf TeX Live 2026 驗證 `0 errors`）。

### 2. 本機編譯（macOS / Linux）
若本機已安裝 TeX Live / MacTeX 或 Tectonic：

```bash
cd ICEIB_paper

# 使用 latexmk + pdflatex
latexmk -pdf main.tex

# 或使用 Tectonic
tectonic main.tex
```

---

## 資料來源與授權標示

本論文使用之實測資料源自以下公開資料集（依各來源條款標示）：
- **TDCS M05A 及即時車速／事件檔**：交通部高速公路局「交通資料庫」（政府資料開放授權條款第 1 版）。
- **中華民國政府行政機關辦公日曆表**：政府資料開放平臺，經 [ruyut/TaiwanCalendar](https://github.com/ruyut/TaiwanCalendar) 整理（CC BY 4.0）。
