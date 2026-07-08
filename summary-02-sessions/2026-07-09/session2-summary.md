# Session 2 Summary — 2026-07-09

## 主題

把 kindle-32-claude-cowork 專案推上 GitHub、清除書籍 PDF 的版控歷史、修 README，並延伸做出「Claude Code vs Claude Cowork 硬邊界」分析 → NotebookLM 三摘要；另設定停車費繳費行事曆提醒。

## 完成事項

### Git / GitHub

- 專案本來就是 git repo（main、3 commit）但**無 remote**；用 `gh repo create chenghyang2001/kindle-32-claude-cowork --public --source=. --remote=origin --push` 一步建立 public repo 並推送。
- 發現 `.nlm-pipeline/` 含整本書全文 PDF（62MB）+ 各章 PDF，public repo 會外流書籍原文。
- 先 `git rm -r --cached .nlm-pipeline/` + 擴大 `.gitignore`（`.nlm-pipeline/screenshots/` → 整個 `.nlm-pipeline/`），移出版控但保留本機檔案（23 檔）。
- 使用者要求徹底清除歷史：`pip install git-filter-repo`（v2.47.0）→ `git filter-repo --path .nlm-pipeline/ --invert-paths --force` → 重接 origin → force push。
- **force push 被權限層擋 3 次**，最後由使用者自己用 `!` 前綴執行成功。
- 以「從遠端 fresh clone」驗證：`.git` 93MB → **172KB**，0 個 commit 引用 `.nlm-pipeline`，最大 blob 僅 22KB HTML，PDF 全數消失。
- 備份 bundle（68MB）用完已依使用者指示刪除。

### README

- 補上 repo 連結、把「目錄結構」段從描述 `.nlm-pipeline/` 改為實際 repo 內容（`code/`、`HANDOFF.md`、`summary-02-sessions/`），並標註 `.nlm-pipeline/` 為本機專屬不發佈。

### 分析 + NotebookLM

- 調出既有 `code/bonus/index.html`（第14張加碼練習「Code vs Cowork 任務分流」）。
- 針對使用者更尖銳的提問（「哪些工作**一定**只能用 Cowork、Code 根本做不到」）產出誠實分析：對已搭好 MCP+VPS+cron+gws 的重度 CLI 使用者，硬邊界不在「做什麼」而在「在哪跑/用什麼裝置/給誰用」。
- 把分析寫成來源檔 → 建新 NotebookLM notebook → 觸發語音/簡報/影片三摘要（背景下載中）。

### Google Calendar

- 帳號只有主行事曆（無獨立工作曆）；在主行事曆建**循環繳費提醒**：正好停·南崗停車位 NT$1200，每 2 個月，到期日 9/8 09:00，3 個 email 提醒落在 9/1、9/2、9/3（到期前 7/6/5 天）。event id `e17oamqk805pvgs4tp1371q7hg`。

## 關鍵技術筆記

- `git rm --cached` 只改新 commit，舊 blob 仍在歷史；徹底清除要 `git filter-repo` + force push，且必須用**遠端 fresh clone** 驗證（本機乾淨 ≠ 遠端乾淨）。
- filter-repo 會自動移除 origin remote（安全設計），重寫後要手動 `git remote add` + 重設上游。
- 本環境**權限層會攔截 `git push --force`**，即使對話中同意也擋；需使用者用 `!` 前綴自行執行。
- notebooklm CLI v0.4.1 指令：`create` / `source add <file>` / `generate audio|slide-deck|video --language zh_Hant` / `artifact wait <id>` / `download <type> -a <id> <path>`（語言碼 `zh_Hant`）。
- Google Calendar email 提醒是「事件前 N 分鐘」相對值，無法指定絕對日；用到期日 + 7/6/5 天（10080/8640/7200 分）達成指定的三天提醒。

## 產出檔案

| 檔案 | 說明 |
| --- | --- |
| GitHub repo（public） | <https://github.com/chenghyang2001/kindle-32-claude-cowork> |
| `.gitignore`（已 commit `19961c3` 之前數個 commit） | 忽略整個 `.nlm-pipeline/` |
| `README.md` | 補 repo 連結 + 目錄結構改為實際內容 |
| `scratchpad/claude-code-vs-cowork.md` | NLM 來源檔（Code vs Cowork 硬邊界分析） |
| NotebookLM notebook `9258a069-6b24-4c1f-a5f5-7bb8c850c79a` | 三摘要來源 |
| `scratchpad/cowork-artifacts/`（背景下載中） | cowork-audio.m4a / slide-deck.pptx / video.mp4 |
| Calendar event `e17oamqk805pvgs4tp1371q7hg` | 停車費循環提醒 |

## HANDOFF（下次 session 優先處理）

### 立即行動

- [ ] （可選）把 NLM 三摘要對外部署：影片上 VPS/Drive、簡報用 gws 轉 Google Slides、語音掛進播放清單。
- [ ] （可選）詢問使用者是否要把 `scratchpad/claude-code-vs-cowork.md` 分析納入 repo（例如更新 `code/bonus/index.html` 的「決定性差異」為兩層：真正只有 Cowork / Code 也能做只是要設定）。

### 進行中（需接續）

- **NLM 三摘要已生成並下載完成**（背景 task `bvhlnb24e` exit 0，共 70MB）：notebook `9258a069-6b24-4c1f-a5f5-7bb8c850c79a`
  - Audio `a7c8a3f5...` →「Claude Code 與 Cowork 的硬邊界」→ `scratchpad/cowork-artifacts/cowork-audio.m4a`（39M）
  - Slide Deck `f1507ca6...` →「Claude Code vs Claude Cowork」→ `cowork-slide-deck.pptx`（4M）
  - Video `cc339128...` →「Claude Code 對決 Cowork：硬邊界」→ `cowork-video.mp4`（27M）
  - ⚠️ 檔案在 scratchpad（session 專屬 temp）；若要保留須移出或部署。重新下載：`notebooklm download <audio|slide-deck|video> -n 9258a069... -a <id> <路徑>`。來源檔 `scratchpad/claude-code-vs-cowork.md` 同為 temp。

### 注意事項

- 本環境 `git push --force` 會被權限層攔，需使用者用 `!` 自行執行。
- GitHub Pages 未啟用：使用者要求「以 `code/` 為站台」但中途喊停（Pages 分支部署只支援 `/` 或 `/docs`，要 code/ 需 Actions workflow 或改名 docs/）。若日後續做，走 workflow `.yml` 會觸發三 agent 鐵律。
- 停車費提醒是「固定每 2 個月的 8 號」；實際繳費日漂移時需調整循環事件起算日。
