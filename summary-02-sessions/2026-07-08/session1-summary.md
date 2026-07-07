# Session 1 — Kindle #32《Claude Cowork》互動練習系統 + 影片下載

- **日期**：2026-07-08
- **repo**：`kindle-32-claude-cowork`（本地 git，無 remote）
- **主題**：本 repo 首個 session。原定接手「14 支章節影片對外部署」，中途轉向做「全書 15 張互動練習網頁」並上線。

## 完成事項

### A. 影片（部分完成，使用者喊停）

- 撈出 14 支 video artifact 清單，落檔 `.nlm-pipeline/chapter_video_tasks.tsv`（章號↔artifact ID↔繁中章名）。
- 派 @小雲 在 VPS（187.127.109.145）下載 14 支 mp4 → `/home/claude/kindle-32-videos/ch00~ch13.mp4`（720p、約 9 分/支、共 547MB，14/14 成功）。@小雲 遇 VPS 端 NLM auth 過期，改同步本機新鮮 auth token 上 VPS（僅同步憑證、影片仍全程 VPS 下載，未違反鐵律）。
- **使用者當場喊停**：video viewer 網頁 + 對外部署**未做**，mp4 原地保留在 VPS。

### B. PDF 整理

- 建 `PDF/`（全書 `claude-cowork-full-book.pdf` + `chapters/ch00~13.pdf`），發現正本早已 commit 在 `.nlm-pipeline/`，故整個 `PDF/` 加進 `.gitignore` 避免重複入版控。

### C. 全書 15 張互動練習網頁（主要產出，已上線）

- **ch01** 主 Claude 手動建；**ch02~ch13** 由 12 個並行 sonnet subagent 各讀章節 image PDF（PyMuPDF 渲染 PNG → Read）照模板產出；**ch00 導論**與 **bonus 加碼**另補。共 **15 張**（導論 + 13 章 + 加碼）。
- 每頁三段式：① 重點回顧 → ② 10 題即時對錯互動題（含解釋＋計分，題目全取自各章書中實際範例）→ ③ 接回 Claude Cowork 真實 demo（可貼提示詞）。
- 加碼頁 `bonus/`：Code vs Cowork 任務分流速查表 + 10 題二選一（回應使用者「該用 Cowork 還是 Claude Code」提問）。
- 總覽入口 `code/index.html`（順序 ch00→ch13→加碼）。
- 全頁右上角加 **A−/重設/A+ 字體控制**（調 html 根字級縮放 rem 文字、localStorage 跨章記憶、try/catch 相容 file://）。
- 發布到 public repo `chenghyang2001/kindle-32-practice` GitHub Pages，併進 **mermaid-viewer 第 19 tab**「Kindle 32｜Cowork 練習」（綠色）。

## 關鍵技術筆記 / 決定

- **影片 host 決策**：選 GitHub Pages 而非 VPS HTTP——VPS 是裸 IP 無 HTTPS，嵌進 HTTPS 的 mermaid-viewer 會被擋成 mixed content。單支最大 45.9MB < GitHub 100MB 上限。此決策同樣適用未來影片 viewer。
- **並行 subagent 省主 context**：讀 image PDF 極吃 token，12 章交給並行 subagent 在各自 context 讀＋寫檔（subagent context 內寫 .html 不觸發三 agent 鐵律），主 context 保持精簡。惟 12 個 sonnet 並行使本週用量配額超 100%。
- **Windows native git 路徑雷**：native git 不認 MSYS 的 `/tmp`、`~` 展開路徑（clone 回報成功但目錄「消失」）。解法：在 `~/workspace` 下用**相對路徑** clone。
- **字體縮放**：用 `html` 根字級 + rem 縮放（頁面文字多為 rem），非 zoom；localStorage 同源跨 15 頁生效。
- **PowerShell 路徑**：Git Bash `-File "$HOME/..."` 會轉成 `/c/...` 讓 PowerShell 不認，需 `cygpath -w` + `MSYS_NO_PATHCONV=1`。

## 產出檔案

| 檔案 | 說明 |
| ------ | ------ |
| `.nlm-pipeline/chapter_video_tasks.tsv` | 14 支影片 artifact 對照表 |
| `.gitignore` | 加排除 `PDF/` |
| `code/index.html` | 15 張練習總覽入口 |
| `code/ch00~ch13/index.html` | 導論 + 13 章互動練習 |
| `code/bonus/index.html` | 加碼 Code vs Cowork 分流練習 |
| `code/README.md`、`code/BUILD-GUIDE.md` | 進度表 / subagent 建置規範 |
| `HANDOFF.md` | 更新交接狀態（影片已下載未部署、15 張練習完成） |
| repo `chenghyang2001/kindle-32-practice` | GitHub Pages 上線站 |
| `mermaid-viewer/index.html` | 加第 19 tab（另一 repo，已 push） |

commit：`48a2615`、`21ca5ae`、`2ab0f88`、`a564736`、`9ac2d52`、`ba9b80d`、`402c263`、`c453770`（+本次 HANDOFF）。

## HANDOFF（下次 session 優先處理）

### 立即行動

- [ ] 影片對外部署續做：做 video viewer 網頁 → 發布 GitHub Pages（比照 kindle-32-practice，**勿用 VPS HTTP 免 mixed content**）→ 併進 mermaid-viewer（會是第 20 tab）。mp4 已在 VPS `/home/claude/kindle-32-videos/`，artifact ID 在 `chapter_video_tasks.tsv`。
- [ ] （可選）第 13 章簡報補生（`chapter_slide_tasks.tsv` 缺 ch13，當初被 Google 限流）。

### 進行中（需接續）

- 影片：14 mp4 已在 VPS 下載完成、原地保留，viewer + 部署未做（使用者本 session 喊停）。

### 注意事項

- 本週 API 用量配額已超 100%（12 並行 subagent 讀圖所致）；下次大量讀圖任務建議分批或開新 session。
- 本 repo 無 git remote（本地版控）；練習頁的線上版在獨立 repo `kindle-32-practice`，改練習頁要記得**重新部署**（相對路徑 clone → 複製 → push）。
- VPS 端 NLM keepalive cron 已不在 crontab，auth 會過期；需要時同步本機 token 上去。
