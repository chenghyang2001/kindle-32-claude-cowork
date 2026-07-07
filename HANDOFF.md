# 交接文件 — Kindle #32《Claude Cowork》NLM Pipeline

> 這份文件給「在 `C:\Users\user\workspace\kindle-32-claude-cowork` 目錄下開的新 Claude Code session」閱讀。
> 舊 pipeline 是在 `Downloads\AutoRead-GoogleBook` 目錄做的，2026-07-08 整包搬到此獨立 repo。
> **不要再回 AutoRead-GoogleBook 目錄動這本書的東西。**

---

## 一、這是什麼書

- 書名：《Claude Cowork：31個超好用範例實作》
- 作者：胡嘉璽　出版：深智數位　ISBN 978-626-7889-35-0
- 來源：Google Play Books 連續捲動書，362 頁
- NotebookLM notebook ID：**`7cc07ede-1d49-45e7-8377-615646d660cd`**

---

## 二、目前狀態（✅ 已完成 / ⏳ 待辦）

### ✅ 已完成並上線（Session 274）

| 產出 | 狀態 | 連結 |
| ------ | ------ | ------ |
| 402 張截圖 + 全書 PDF（62MB） | ✅ | `.nlm-pipeline/claude-cowork.pdf` |
| 14 章分章 PDF（NLM 獨立 source） | ✅ | `.nlm-pipeline/chapters/ch00~ch13.pdf` |
| **14 章語音 → VPS m3u** | ✅ HTTP 200 | <http://187.127.109.145/kindle-32.m3u> |
| **簡報 viewer（194 PNG）** | ✅ HTTP 200 | <https://chenghyang2001.github.io/kindle-32-slides/> |
| **併入 mermaid-viewer 第 18 tab** | ✅ HTTP 200 | <https://chenghyang2001.github.io/mermaid-viewer/> |
| NLM artifact 全繁中章節命名 | ✅ | audio 15/15、slide 14/14、video 15/15 completed |
| 影片 artifact ID 對照表 | ✅ | `.nlm-pipeline/chapter_video_tasks.tsv`（14 支 video ID↔繁中章名） |
| **15 張互動練習網頁**（導論+13章+加碼） | ✅ HTTP 200 | <https://chenghyang2001.github.io/kindle-32-practice/> |
| **練習併入 mermaid-viewer 第 19 tab** | ✅ | 「Kindle 32｜Cowork 練習」（綠色 tab） |
| **練習頁字體放大/縮小鈕**（14→15 頁，跨章記憶） | ✅ | 每頁右上 A−/重設/A+ |

> 2026-07-08 新增（**15 張互動練習全數完成**）：`code/` 目錄放導論 ch00 + 全 13 章（ch01~ch13）+ 1 張加碼綜合（`bonus/` Code vs Cowork 任務分流），每頁 10 題即時對錯＋解釋＋計分＋接回 Cowork demo、右上角字體大小鈕（localStorage 跨章記憶）。13 章由 12 個並行 subagent 依各章 PDF 產出、ch00 與 bonus 另補。總覽入口 `code/index.html`（順序 ch00→ch13→加碼）。已發布到 public repo `chenghyang2001/kindle-32-practice` 的 GitHub Pages，並嵌進 mermaid-viewer 第 19 tab。

### ⏳ 可選待辦（本 repo 開新 session 主要要接的事）

1. **🎬 14 支章節影片對外部署**（進行中，已下載未部署）
   - 現況（2026-07-08）：@小雲 已在 **VPS 下載 14 支 mp4**（`/home/claude/kindle-32-videos/ch00~ch13.mp4`，720p、約 9 分/支、共 547MB），但使用者當場喊停，**viewer 網頁與對外部署尚未做**。檔案原地保留在 VPS。
   - 影片 artifact ID 已落檔於 `.nlm-pipeline/chapter_video_tasks.tsv`（不必再重撈）。
   - 續做：做 video viewer 網頁 → 併進 mermaid-viewer（比照練習頁走 GitHub Pages；**勿嵌 VPS HTTP，會被 HTTPS mermaid-viewer 擋成 mixed content**）。
   - ⚠️ 決策紀錄：影片 host 選 **GitHub Pages**（單支最大 45.9MB < 100MB 上限），不是 VPS HTTP——因 VPS 是裸 IP 無 HTTPS，嵌不進 mermaid-viewer。
2. **📊 第13章簡報補生**（次要，可略）
   - 現況：slide tasks 只到第12章（`chapter_slide_tasks.tsv` 缺 ch13）；當初第13章 slide 被 Google 限流。
   - 簡報 viewer 目前 = 第0~12章 + 全書導覽 = 14 張，缺第13章。
   - 要補：用 ch13 source（見下表）重新 `generate slide-deck`，完成後下載併進 slide viewer。

---

## 三、🔴 鐵律（做這本書必守）

1. **所有 VPS 工作一律派 @小雲（冷靜的遠端小雲）在 VPS 上做。**
   - 不要「本機下載 mp4 再 scp 上 VPS」。要在 VPS 上直接下載、直接部署。
   - 每一個要在 VPS 執行的命令都交給 @小雲。
2. **章節 audio/video 必須用「每章獨立 source」**（NLM 鐵律，混合大 source 會被 reject）。source ID 見下表。
3. **NLM 操作預設主 Claude 直做**（`python -m notebooklm` CLI 或 `mcp__notebooklm__*`），只有「批次 > 15 分鐘」或「VPS 端」才派 sub-agent。
4. **語言碼是 `zh_Hant`**（底線，不是 `zh-TW`）；audio 下載出來是 **.m4a**；download 用 positional path 不是 `--output-dir`。
5. **先自己驗證再回報**（curl HTTP 200 / artifact list status），不要叫使用者幫忙確認。

---

## 四、關鍵 ID 對照表（真實值，已驗證）

### NotebookLM source ID（每章獨立 source）

| 章 | source ID | 標題 |
| ---- | ----------- | ------ |
| 0 | `62d5bca2-13a5-4c42-ab53-677e69474f1b` | 第0章 AI Agent 的時代 |
| 1 | `08e8fecb-b396-4b18-b80a-5eb7cf830b3e` | 第1章 認識 Claude Cowork |
| 2 | `9ea3a4ce-889d-4605-a872-03ce0aa00ffd` | 第2章 設定你的 Cowork |
| 3 | `38cf2ae6-fa37-4a91-883c-f3b67ece6979` | 第3章 Cowork 的日常操作 |
| 4 | `2398b890-7ac2-4d2c-85af-0ec93f05a2b4` | 第4章 生活管理實戰 |
| 5 | `067dd261-4b06-4d69-a2cc-d8344885e630` | 第5章 行程與通訊實戰 |
| 6 | `a117c771-a780-47c2-9fe7-97a647889420` | 第6章 研究與學習實戰 |
| 7 | `b2c2dcb5-1409-42f1-85c2-1bb0210954be` | 第7章 辦公文件製作實戰 |
| 8 | `4f041ced-e19e-459b-b2b5-cdb4ef6395e3` | 第8章 資料分析實戰 |
| 9 | `a3152fa1-7eca-4d10-851c-7e5b6ab342f3` | 第9章 設計與創意實戰 |
| 10 | `a09a16f5-30df-42b3-91cd-1aea65354df4` | 第10章 工程與開發實戰 |
| 11 | `9c2a2adf-a198-40d6-a856-18d3d36e7cd8` | 第11章 工作效率實戰 |
| 12 | `bb57f13d-6d81-446c-931c-5182e0f7fe30` | 第12章 排程與自動化 |
| 13 | `3543fd12-3057-47cc-a551-0d766090c1c2` | 第13章 進階自訂與品牌設計 |

> 完整對照另存於 `.nlm-pipeline/chapter_sources.tsv`（source）、`chapter_audio_tasks.tsv`（audio artifact）、`chapter_slide_tasks.tsv`（slide task，僅到第12章）、`chapter_video_tasks.tsv`（video artifact，14 章全）。
> ✅ 2026-07-08：影片 artifact ID 已落檔於 `chapter_video_tasks.tsv`，不必再重撈。

---

## 五、目錄結構

```
kindle-32-claude-cowork/           ← 本 repo 根目錄（已 git init，branch main；無 remote）
├── HANDOFF.md                     ← 本文件
├── README.md                      ← 書籍與 pipeline 說明
├── .gitignore                     ← 排除 screenshots/、PDF/、Python 快取、垃圾檔
├── code/                          ← 15 張互動練習網頁（導論+13章+加碼，2026-07-08 新增）
│   ├── index.html                 ← 練習總覽入口（13 章卡片，點了開對應練習）
│   ├── ch00/ … ch13/index.html    ← 導論+各章互動練習（單檔自足、離線可開）
│   ├── bonus/index.html           ← 加碼：Code vs Cowork 任務分流
│   ├── README.md / BUILD-GUIDE.md ← 章節進度表 / subagent 建置規範
│   └── （已發布至 GitHub Pages repo chenghyang2001/kindle-32-practice）
├── PDF/                           ← 全書＋分章 PDF 便利複本（.gitignore；正本在 .nlm-pipeline/）
└── .nlm-pipeline/
    ├── grab_test.py               ← GPB PageDown 抓圖工具（SHA256 唯一性驗證）
    ├── .poll_*.py                 ← NLM 任務輪詢輔助腳本
    ├── claude-cowork.pdf          ← 全書 PDF（402 頁 / 62MB）
    ├── chapters/ch00~ch13.pdf     ← 14 章分章 PDF
    ├── chapter_sources.tsv        ← source ID 對照
    ├── chapter_audio_tasks.tsv    ← audio artifact ID 對照
    ├── chapter_slide_tasks.tsv    ← slide task ID 對照（僅到 ch12）
    ├── chapter_video_tasks.tsv    ← video artifact ID 對照（14 章全，2026-07-08）
    └── screenshots/               ← 402 張原始截圖（.gitignore，可重抓）
```

---

## 六、VPS 部署現況（給 @小雲 接手參考）

- VPS：Hostinger `187.127.109.145`
- 語音 m3u 已在：`http://187.127.109.145/kindle-32.m3u`
- 簡報 viewer GitHub Pages repo：`chenghyang2001/kindle-32-slides`
- ⚠️ **VPS 沒有 LibreOffice** → PPTX→PNG 不通；S274 改用「NLM 匯 PDF + pymupdf 渲染 PNG」繞道。影片部署若需縮圖同理。
- nginx：站台在 `sites-enabled/hookhub`（default_server），不是 demo17.conf。

---

## 七、下一步一句話總結

> 影片 14 支 mp4 **已在 VPS 下載完成**（`/home/claude/kindle-32-videos/`），剩下的是：
> **做 video viewer 網頁 → 發布到 GitHub Pages（比照 kindle-32-practice / kindle-32-slides，勿用 VPS HTTP 以免 mixed content）→ 併進 mermaid-viewer（會是第 20 tab）。**
> 参考已完成的練習頁部署流程（本檔第二節 ✅ 表 + `code/`）。
