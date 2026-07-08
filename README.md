# Kindle #32 —《Claude Cowork：31個超好用範例實作》NLM Pipeline

> Repo：<https://github.com/chenghyang2001/kindle-32-claude-cowork>

胡嘉璽 著，深智數位（ISBN 978-626-7889-35-0），Google Play Books 共 362 頁。

從 GPB 連續捲動抓圖 → 合併 PDF → 依章節切分 → NotebookLM 每章生成語音／影片／簡報 → VPS m3u + GitHub Pages 部署的完整資料與工具。

## 目錄結構

```
code/                         # 各章互動練習頁（部署至 GitHub Pages）
├── index.html                # 練習總覽入口
├── ch00/ ~ ch13/index.html   # 14 章 + bonus 練習頁
├── BUILD-GUIDE.md            # 練習頁建置說明
└── README.md
HANDOFF.md                    # 跨 session 交接狀態
summary-02-sessions/          # 各 session 收工 summary
```

> **本機專屬（不隨 repo 發佈）**：NLM pipeline 的中間產物放在 `.nlm-pipeline/`（已列入 `.gitignore`，僅存在於本機工作目錄），內含全書合併 PDF、`chapters/` 各章 PDF、`grab_test.py` 抓圖工具、`chapter_*.tsv` 對照表與 `.poll_*.py` 輪詢腳本。這些皆為可重建產物，且含書籍原文，故不進版控。

## 產出（已上線，Session 274 完成）

| 產出 | 連結 |
| ------ | ------ |
| 語音 m3u 播放清單 | <http://187.127.109.145/kindle-32.m3u> |
| 簡報 viewer | <https://chenghyang2001.github.io/kindle-32-slides/> |
| 彙整（mermaid-viewer 第 18 tab） | <https://chenghyang2001.github.io/mermaid-viewer/> |
| NotebookLM notebook | `7cc07ede-1d49-45e7-8377-615646d660cd` |

- 語音 15/15、簡報 14/14、影片 15/15 於 NLM 皆 completed，章節名全繁中化（第00章～第13章）。
- **可選待辦**：14 支章節影片已於 NLM 內生成＋命名，尚未下載 mp4 對外部署。

## 抓圖工具用法

> `grab_test.py` 位於本機 `.nlm-pipeline/`（不隨 repo 發佈,見上方說明）。

```bash
cd .nlm-pipeline
PYTHONUTF8=1 python grab_test.py <張數> [倒數秒=5] [間隔秒=1.2]
```

倒數期間請主動點擊 Google Play Books 視窗（避免 PyAutoGUI focus-trap 抓到同一頁）。
