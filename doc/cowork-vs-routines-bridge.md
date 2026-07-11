# Claude Code CLI ↔ Claude Cowork：能不能互相驅動？（研究筆記）

> 研究日期：2026-07-11
> 問題起源：能不能在終端機下 `claude cowork /schedule` 之類的指令，讓 Claude Code CLI 去命令「活在桌面 App / claude.ai 裡的 Cowork」幫忙跑任務？

---

## 0. 一句話結論

- **CLI 不能直接驅動 Cowork 桌面 agent**——這是官方已知、尚未實作的功能請求（GitHub Issue #25791），兩者目前「完全隔離」。
- **但你真正想要的「從 CLI/API 觸發一個雲端跑、關機也跑的 agent 任務」是做得到的**——用官方的 **Claude Code Routines + API trigger**（`/fire` 端點）。
- 差別的本質不在「能力」，而在「agent 跑在哪」：官方讓**雲端 agent（Routines）有 API**，讓**桌面 agent（Cowork 本尊）沒有**。

---

## 1. 為什麼會有這個疑問

Claude 現在有多個「會自己動手」的 runtime，長得很像但各跑各的：

| Runtime | 身分 | 跑在哪 | 存取本機檔案 |
| --- | --- | --- | --- |
| **Claude Code CLI** | 終端機的 `claude` 指令 | 你的機器 | ✅ 直接讀寫本機 repo、跑 shell |
| **Claude Cowork** | 桌面 App / claude.ai 裡的「Cowork」介面 | 桌面版可本機或雲、Cowork 排程可雲端常駐 | ✅（在桌面情境）整理檔案/照片/下載夾 |
| **Claude Code Routines** | 雲端版 Claude Code（claude.ai/code） | **Anthropic 雲端**，關機也跑 | ❌ 本機檔案碰不到，但可 clone GitHub repo、用 connector |

疑問就是：這幾個能不能互相下指令？特別是「CLI → Cowork」。

---

## 2. 直接答案：CLI → Cowork 目前無官方橋接

**GitHub Issue #25791「Claude Code ↔ Cowork Bridge: Programmatically trigger Cowork tasks from Claude Code CLI」**（anthropics/claude-code）就是問一模一樣的事：

- **狀態**：Open，純功能請求，無人認領，Anthropic 未公開回應。
- **明講**：Claude Code 與 Cowork「completely isolated」，沒有程式化觸發 Cowork 任務的方法。
- **目前做不到的事**：
  - 從 Claude Code 跑 Cowork 的 UI 測試流程
  - 在兩個工具之間串接任務
  - 在 CI/CD 裡自動化 Cowork 任務
  - 用 Claude Code 的 MCP 去驅動 Cowork
- **提案中的四種方案（都還沒實作）**：
  1. 一個 CLI 命令，如 `claude cowork "organize files in ~/Downloads"`
  2. 本機 API / IPC bridge（Cowork 開一個 localhost 端點或 Unix socket）
  3. 共用 MCP context，兩邊互傳任務
  4. 幫 Cowork 做一個 MCP server，讓 Claude Code 把 Cowork 當工具用（`cowork_run_task` / `cowork_get_result`）
- **現有 workaround**：無。只能手動切換 + 複製貼上 context。

> 換句話說：`claude cowork /schedule` 這種指令**不存在**，`claude` CLI 的子命令裡也沒有 `cowork`。

---

## 3. 真正能做的方法：Claude Code Routines + API trigger

如果目標是「**從 CLI 或任何能發 HTTP 的地方，觸發一個雲端跑的 Claude 任務**」——這正是 Cowork 排程的核心價值——官方有現成而且穩定的路徑：**Routines**。

### 3.1 Routines 是什麼

- 一個 routine = 存起來的 Claude Code 設定：**一段 prompt + 一或多個 GitHub repo + 一組 connector**，打包成可以無人值守自動跑。
- **跑在 Anthropic 雲端**，所以你筆電關了它照跑。
- 每個 routine 可掛一或多個 **trigger**：
  - **Schedule**：每小時 / 每天 / 平日 / 每週 / 或未來某個時間點跑一次
  - **API**：發一個帶 bearer token 的 HTTP POST 就觸發
  - **GitHub**：repo 事件（PR、release）自動觸發
- 適用方案：Pro / Max / Team / Enterprise，且開啟「Claude Code on the web」。
- 管理入口：[claude.ai/code/routines](https://claude.ai/code/routines)，或 CLI 的 `/schedule`。
- **web / Desktop / CLI 三個介面寫的是同一個雲端帳號**，任一邊建的 routine 另外兩邊都看得到。

### 3.2 用 CLI 建 routine：`/schedule`

在任何 session 裡打：

```text
/schedule daily PR review at 9am           # 週期性
/schedule tomorrow at 9am, summarize yesterday's merged PRs   # 一次性
/schedule clean up feature flag in one week
```

- `/schedule` 只能建**排程觸發**版；要加 **API / GitHub trigger** 得去網頁編輯。
- 其他子指令：`/schedule list`（列出）、`/schedule update`（改，含設 cron）、`/schedule run`（立刻跑一次）。
- **最小間隔 1 小時**（更頻繁的 cron 會被拒）。

### 3.3 加 API trigger（只能在網頁做）

1. 到 [claude.ai/code/routines](https://claude.ai/code/routines) → 點 routine → 鉛筆圖示「Edit routine」。
2. 捲到「Select a trigger」→「Add another trigger」→ 選「API」。
3. Modal 會顯示這個 routine 的 URL + 範例 curl；按「Generate token」複製 token（`sk-ant-oat01-...`，**只顯示一次**，之後拿不回來）。
4. 每個 routine 有自己的 token，只能觸發那一個 routine；重新產生就把舊的撤銷。

### 3.4 觸發：`/fire` 端點完整規格

```bash
curl -X POST https://api.anthropic.com/v1/claude_code/routines/$ROUTINE_ID/fire \
  -H "Authorization: Bearer $ROUTINE_TOKEN" \
  -H "anthropic-beta: experimental-cc-routine-2026-04-01" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{"text": "這次要處理的內容/context"}'
```

**Headers**

| Header | 必要 | 說明 |
| --- | --- | --- |
| `Authorization` | ✅ | `Bearer <token>`，per-routine token，前綴 `sk-ant-oat01-` |
| `anthropic-beta` | ✅ | 必含 `experimental-cc-routine-2026-04-01`，少了回 400 |
| `anthropic-version` | ✅ | 例如 `2023-06-01` |
| `Content-Type` | 有 body 時 | `application/json` |

**Path 參數**

- `routine_id`：雖然叫 routine_id，實際值前綴是 **`trig_`**（不是 `routine_`），加 API trigger 時 modal 給的 URL 裡就有。

**Request body**

- `text`（選填）：這次跑的初始 context（alert 內文、失敗 log、git diff…）。**純文字、不解析**；傳 JSON 也只會被當字串。上限 65,536 字元。body 可省略。

**成功回應（200）**

```json
{
  "type": "routine_fire",
  "claude_code_session_id": "session_01HJKLMNOPQRSTUVWXYZ",
  "claude_code_session_url": "https://claude.ai/code/session_01HJKLMNOPQRSTUVWXYZ"
}
```

- 打完**立刻回**，不 stream 輸出、不等 session 跑完。開那個 URL 可在瀏覽器看它跑、審改動、繼續對話。

**錯誤**

| HTTP | type | 原因 |
| --- | --- | --- |
| 400 | invalid_request_error | 缺/錯 `anthropic-beta`、text 超長、routine 被暫停 |
| 401 | authentication_error | 沒 token 或 token 不匹配這個 routine |
| 403 | permission_error | 帳號/組織無此端點權限 |
| 404 | not_found_error | routine 不存在 |
| 429 | rate_limit_error | 到達每日 run 上限或用量上限（附 `Retry-After`） |
| 500 / 503 | api_error / overloaded_error | 伺服器錯誤 / 過載（此端點過載回 503，非 529） |

**其他重點**

- **無 idempotency key**：webhook 重試會建多個 session。
- **不在官方 SDK 裡**：token 模型跟 API key 不同，典型呼叫方（CI、alert webhook）直接發 HTTP。
- token 只能在網頁的 API trigger 設定裡產生/撤銷，**沒有 token 管理的公開 API**。

---

## 4. 對照表：Cowork 桌面 App agent vs Claude Code Routines

| | Cowork 桌面 App agent | Claude Code Routines |
| --- | --- | --- |
| API 觸發 | ❌ 無（Issue #25791） | ✅ 有 `/fire` 端點 |
| 在哪跑 | 桌面版排程（Local=本機 / Remote=雲） | 雲端，關機也跑 |
| CLI 建立 | —（桌面 App 內操作） | ✅ `/schedule` 直接建排程版 |
| 三介面同步 | web / Desktop / CLI 寫同一個雲端帳號，互通 | 同左 |
| 本機檔案 | ✅（桌面情境可整理本機檔） | ❌ 碰不到本機，只能 clone repo + connector |
| 觸發方式 | 桌面 App 手動 / 排程 | schedule / **API** / GitHub 三種可組合 |

> **關鍵**：`/fire` 打的是 **Routines（雲端 Claude Code session）**，**不是** Cowork 那個整理檔案的桌面 agent。要驅動「Cowork 本尊」目前仍無 API——但 Routines 幾乎能覆蓋你想自動化的同類工作（讀 connector、寫報告、跑 shell、動 repo）。

---

## 5. 「網路上有人做到嗎？」——三條實證路徑

1. **官方 Routines API（首選）**：已有第三方教學示範 schedule / API / GitHub 三種 trigger 的組合用法。這是「用 CLI/HTTP 觸發雲端 agent」的正解。
2. **開源重寫版 Cowork**：Composio 的 `open-claude-cowork`（GitHub），用 Claude Agent SDK + Composio Tool Router，號稱 500+ SaaS 整合。因為是自己的，**可完全用 API 驅動**——這是「繞過官方隔離」最徹底的做法。
3. **Claude Agent SDK / headless（`claude -p`）**：自建 Cowork-like agent，程式化完全掌控（Python / TS / CLI）。你要的排程 + 自動化可以自己拼。

---

## 6. 對「楊政憲」這台機器的實務注記

- **成本**：routine token 是 `sk-ant-oat01-`（綁 claude.ai 訂閱的 per-routine OAuth token，**不是** API key），用量算 **Max 訂閱**、不扣 API credits。符合你的 cost-rules（禁止全域設 `ANTHROPIC_API_KEY`）。
- **`/schedule` 觸發條件**：CLI 必須用 claude.ai 訂閱登入，且 shell/settings **不能設** `ANTHROPIC_API_KEY` / `ANTHROPIC_AUTH_TOKEN` / `apiKeyHelper`（會蓋掉 claude.ai 登入，導致 `/schedule` 消失）。你本來就沒設，剛好符合。CLI 需 ≥ v2.1.81。
- **connector 來源**：routine 用的是 **claude.ai 帳號上的 connector**；你用 `claude mcp add` 加在本機 CLI 的 MCP server **不會**出現在 routine 裡。要用得在 [claude.ai/customize/connectors](https://claude.ai/customize/connectors) 加，或寫進 repo 的 `.mcp.json`。
- **網路**：routine 在雲端環境跑，預設 Trusted 白名單；要打你自己的服務（NUC/VPS 內網）得改環境的 Network access，且內網 IP 雲端根本連不到——這類任務仍該留給 **NUC/VPS cron 跑 `claude -p`**。
- **定位建議**：
  - 需要**碰本機檔 / 內網 DB / 桌面自動化** → 留在本機 CLI 或 NUC/VPS cron。
  - 需要**雲端常駐、關機也跑、可被外部系統 HTTP 觸發**（如接 alert、CI、webhook）→ 用 Routines API trigger。
  - 需要**桌面整理檔案那種 Cowork 體驗** → 目前只能進桌面 App 手動用，無法程式化。

---

## 7. 限制與注意（截至 2026-07）

- `/fire` 端點掛在 **experimental beta header**（`experimental-cc-routine-2026-04-01`），Routines 整體仍是 **research preview**，request/response、限額、token 語意可能變。破壞性改動走新 dated beta header，舊的兩版會保留一段過渡期。
- **每日 run 上限**（依方案不同），用完回 429；訂閱用量與互動 session 共用額度。一次性 run 不計入每日 routine 上限。
- routine 在雲端**無權限確認提示**：它能跑 shell、用你所有勾選的 connector（含寫入）而不再問你——所以 connector 與 repo push 權限要縮到最小。
- routine 的動作以「你」的身分出現：commit / PR 掛你的 GitHub 帳號，Slack / Linear 等用你連結的帳號。

---

## 8. 來源

- GitHub Issue #25791 — Claude Code ↔ Cowork Bridge：<https://github.com/anthropics/claude-code/issues/25791>
- Trigger a routine via API（Claude Platform Docs）：<https://platform.claude.com/docs/en/api/claude-code/routines-fire>
- Automate work with routines（Claude Code Docs）：<https://code.claude.com/docs/en/routines>
- Get started with Claude Cowork（Help Center）：<https://support.claude.com/en/articles/13345190-get-started-with-claude-cowork>
- Schedule recurring tasks in Claude Cowork：<https://support.claude.com/en/articles/13854387-schedule-recurring-tasks-in-claude-cowork>
- Run Claude Code programmatically（headless）：<https://code.claude.com/docs/en/headless>
- open-claude-cowork（Composio，開源）：<https://github.com/composiohq/open-claude-cowork>
