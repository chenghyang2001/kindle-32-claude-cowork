# Kindle #32 —《Claude Cowork：31個超好用範例實作》NLM Pipeline

> Repo：<https://github.com/chenghyang2001/kindle-32-claude-cowork>

胡嘉璽 著，深智數位（ISBN 978-626-7889-35-0），Google Play Books 共 362 頁。

從 GPB 連續捲動抓圖 → 合併 PDF → 依章節切分 → NotebookLM 每章生成語音／影片／簡報 → VPS m3u + GitHub Pages 部署的完整資料與工具。

## 目錄結構

```
.nlm-pipeline/
├── grab_test.py              # GPB PageDown 抓圖工具（SHA256 唯一性驗證，防 focus-trap）
├── claude-cowork.pdf         # 全書合併 PDF（402 頁 / 62MB）
├── chapters/                 # ch00.pdf ~ ch13.pdf（14 章，各為 NLM 獨立 source）
├── chapter_sources.tsv       # 14 章 NotebookLM source ID 對照
├── chapter_audio_tasks.tsv   # 14 章語音 artifact ID 對照
├── chapter_slide_tasks.tsv   # 簡報 task ID 對照
├── screenshots/              # 402 張原始截圖（.gitignore，可重抓）
└── .poll_*.py                # NLM 任務輪詢輔助腳本
```

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

```bash
cd .nlm-pipeline
PYTHONUTF8=1 python grab_test.py <張數> [倒數秒=5] [間隔秒=1.2]
```

倒數期間請主動點擊 Google Play Books 視窗（避免 PyAutoGUI focus-trap 抓到同一頁）。
