# 章節互動練習 — 建置規範（給 subagent 照做）

你要為《Claude Cowork：31 個超好用範例實作》的**某一章**產出一個互動練習網頁
`code/chNN/index.html`。請嚴格照本規範，產出風格要跟**第 1 章範本**一致。

## 0. 必讀範本

先讀 `code/ch01/index.html`（第 1 章成品）當作**版型範本**：

- **CSS 幾乎原樣沿用**（同一套深色主題、`:root` 變數、卡片/題目/回饋/結果樣式）。
- **三段式結構原樣保留**：① 重點回顧 → ② 動手練習（互動題）→ ③ 接回真實 demo。
- 只換「內容」與「題庫」，不要自創另一種排版。

## 1. 讀你的章節（圖片 PDF）

你的章節 PDF 在 `PDF/chapters/chNN.pdf`（頁面是截圖，需渲染成 PNG 再用 Read 讀）。渲染指令（Windows Python + PyMuPDF）：

```bash
cd ~/workspace/kindle-32-claude-cowork
OUT="$(cygpath -w "$(mktemp -d)/chNN")"
PYTHONUTF8=1 python -c "
import fitz, os
out=r'''$OUT'''; os.makedirs(out, exist_ok=True)
doc=fitz.open('PDF/chapters/chNN.pdf')
for i in range(doc.page_count):
    doc[i].get_pixmap(dpi=110).save(os.path.join(out,f'p{i:02d}.png'))
print('rendered',doc.page_count,'->',out)
"
```

然後用 Read 讀那些 PNG。**不必讀完每一頁**——讀前 8～12 頁（章名頁、各小節標題、書中列出的**範例任務**與**適合場景**）足以設計練習。目標是抓出：

- 這章在教什麼（主題、幾個小節）
- 書中設計的**具體範例任務 / 提示詞**（練習題就從這裡取材，不要瞎編）
- 有沒有「適合 / 不適合」「步驟順序」「該用哪個功能」這類可出題的判準

## 2. 設計「② 動手練習」的題型（挑最貼合這章的）

一律做成**即時對錯 + 解釋 + 計分**的互動題（沿用 ch01 的 JS 邏輯骨架）。依章節內容選最合適的題型：

| 章節性質 | 建議題型 |
| --- | --- |
| 概念 / 判斷（哪個模式、哪個功能） | **情境分類題**（點選 → 即時對錯，如 ch01） |
| 實戰任務（生活/行程/研究/文件/資料/設計/開發…） | **「這個需求該怎麼委派」**：給情境，選「最好的提示詞寫法」或「Cowork 會產出什麼」或「該接哪個資料來源(Drive/Calendar/Gmail)」 |
| 流程 / 設定 | **步驟排序**或**設定項配對**（選正確的下一步 / 正確設定值） |

至少 **8 題**，每題附一句「為什麼」解釋（根據書中內容）。題目與正解**必須源自該章實際內容**。

## 3. 「③ 接回真實 demo」

給 1 個「打開 Claude Desktop → Cowork 實際做一次」的建議任務，用該章書中最有代表性的範例，附可直接貼上的提示詞（放 `<code>`）。

## 4. 硬性要求

- `lang="zh-TW"`、繁體中文、單一 `index.html`、**內含 CSS/JS 無外部相依**（離線可開）。
- 沿用 ch01 的 `<title>` 格式：`第N章 <章名> — 互動練習`。
- 完成前**自我驗證**：抽出 `<script>` 內容跑 `node --check` 確認 JS 語法無誤。
- 只動 `code/chNN/index.html`（自己那一章），不要碰別章、不要碰別的檔。

## 5. 回報（給主 Claude）

簡短回報：章號、章名、你判定的主題、選用題型、題目數、node --check 結果（PASS/FAIL）、任何卡點。
