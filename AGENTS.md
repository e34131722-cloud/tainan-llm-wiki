# AGENTS.md - TAINAN VIBE｜台南年輕人旅遊 LLM Wiki

## 1. 專案目的與架構理念
本專案是一個結合 **LLM Wiki 知識庫架構** 的「台南年輕人旅遊指南」靜態網站。
我們不直接手寫零散網頁，而是透過 **「原始素材（raw）→ AI 編譯知識庫（wiki）→ 靜態網頁呈現（pages）」** 的三層流程，讓 AI 能持續吸收新的景點筆記、自動建立條目間的交叉關聯，並同步更新到 GitHub Pages 網站上。

## 2. 目錄結構與檔案職責
- `/AGENTS.md`：本專案最高開發與知識庫維護規則書。
- `/raw/`：**原始資料層（唯讀）**。存放未經整理的旅遊筆記、社群推薦清單（如 `2026-11-tainan-notes.md`）。AI 絕對不可修改或刪除 `/raw/` 內的任何檔案。
- `/wiki/`：**LLM Wiki 知識層**。由 AI 負責建立、重構與交叉連結的 Markdown 知識庫：
  - `/wiki/index.md`：全站知識條目總目錄（按主題分類列出所有條目與一句話摘要）。
  - `/wiki/log.md`：Wiki 編修日誌（以時間倒序記錄每次新增或修改的條目）。
  - `/wiki/routes/<slug>.md`：各旅遊路線的標準化知識條目（檔名採 kebab-case）。
- `/index.html` & `/pages/<slug>.html`：**網頁呈現層**。將 `/wiki/` 的知識內容轉換為視覺化網頁，方便一般讀者瀏覽。
- `/assets/css/style.css`：全站唯一共用樣式表，禁止在 HTML 內寫大量行內樣式。

## 3. Wiki 條目 Markdown 格式規範（`/wiki/routes/*.md`）
所有 Wiki 條目必須包含以下 YAML Frontmatter 與固定段落：
1. **YAML Frontmatter**：包含 `id`（如 `WIKI-001`）、`title`、`category`（`day` 或 `night`）、`updated`（YYYY-MM-DD）、`source`（對應的 `/raw/` 檔名）、`related`（關聯條目 slug）。
2. **固定內容段落**：`## 核心亮點摘要`、`## 精選停留點`、`## 順遊時間軸`、`## 關聯知識條目`。

## 4. 網頁呈現層設計與結構規範（`/pages/*.html`）
- **視覺風格**：「台南老宅日光與微醺夜色」，使用 `--bg-color: #FAF7F2`（溫潤米白）、`--primary-color: #C84B31`（古蹟磚紅）、`--secondary-color: #2D4263`（夜色靛藍）。
- **分頁固定 7 大區塊**：
  1. 頂部導覽列（`<nav class="navbar">`）
  2. Wiki 元資料資訊列（`<div class="wiki-meta-bar">`）：顯示條目編號（`WIKI-XXX`）、資料來源檔名、最後編譯日期。
  3. 頁面標頭（`<header class="page-header">`）：分類徽章、主標題、導讀與基礎資訊列。
  4. 行前亮點摘要框（`<section class="vibe-summary">`）：3 點條列摘要。
  5. 推薦地點卡片列表（`<section class="spot-list">`）：至少 3 張 `<article class="spot-card">`。
  6. 順遊時間軸建議表（`<table class="schedule-table">`）。
  7. **Wiki 關聯知識節點（`<section class="related-wiki">`）**：提供可點擊跳轉至其他關聯路線分頁的連結卡片。

## 5. AI 執行任務的標準工作流（SOP）
當使用者提供新的原始筆記（或要求新增旅遊主題）時，AI 必須依序執行以下兩階段任務：

### 階段 A：Wiki 知識入庫（Ingest）
1. 在 `/wiki/routes/` 建立新的 `.md` 條目（依序編號 `WIKI-003`...），提煉 `/raw/` 中的重點並補齊時間軸。
2. 更新 `/wiki/index.md`，將新條目加入對應分類目錄中。
3. 在 `/wiki/log.md` 最上方新增一筆今日的編修紀錄（格式：`- [YYYY-MM-DD] 新增條目：<標題>（來源：<raw檔名>）`）。
4. 檢查既有條目，在雙方的「關聯知識條目」中互相建立連結。

### 階段 B：網頁呈現同步（Publish）
1. 依據剛建立的 `/wiki/routes/<slug>.md`，在 `/pages/<slug>.html` 產出符合第 4 點 7 大區塊結構的網頁（引入 `../assets/css/style.css`）。
2. 開啟 `/index.html`：
   - 在 `<div class="guide-grid">` 新增該路線的篩選卡片（`<a class="guide-card" data-category="...">`）。
   - 在 `<ul class="wiki-log-list">` 最上方同步新增一筆更新日誌，確保網頁首頁與 `/wiki/log.md` 保持一致。
