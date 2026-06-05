# taiwan-legal-plugin

[English](README.en.md) · **繁體中文**

> 讓 Claude 直接在 Cowork 或 Claude Code 裡查台灣公開法律資料庫——裁判書與法規。

`taiwan-legal-plugin` 是 [Claude Code](https://claude.com/claude-code) 的 plugin marketplace，把台灣三個公開法律資料來源接上 Anthropic 的 [Claude for Legal](https://github.com/anthropics/claude-for-legal) 生態：

- **司法院裁判書系統**（judgment.judicial.gov.tw）— 全文搜尋與裁判全文取得
- **全國法規資料庫**（law.moj.gov.tw）— 11,700+ 部法規條文
- **憲法法庭裁判 / 大法官解釋**（cons.judicial.gov.tw，預計 v0.2 開放）

底層由開源 [`mcp-taiwan-legal-db`](https://github.com/lawchat-oss/mcp-taiwan-legal-db) MCP server 提供 **8 個工具**。本 v0.1 只把其中兩類包成 skill：裁判書搜尋／法規查詢。**憲法法庭釋字／憲判字與引用關係圖譜的工具雖在底層 MCP server 中就緒，v0.1 尚未暴露為 skill——預計 v0.2 開放。**

## 安裝

**前置**：安裝 [uv](https://docs.astral.sh/uv/)（單一執行檔，跨平台；已安裝可略過）：

```
curl -LsSf https://astral.sh/uv/install.sh | sh           # macOS / Linux
powershell -c "irm https://astral.sh/uv/install.ps1|iex"  # Windows
```

在 Claude Code 內：

```
/plugin marketplace add github:lawchat-oss/taiwan-legal-plugin
/plugin install taiwan-legal@taiwan-legal-plugin
```

安裝後重啟 Claude Code。底層 [`mcp-taiwan-legal-db`](https://github.com/lawchat-oss/mcp-taiwan-legal-db) 會在**首次查詢時由 `uvx` 自動從 PyPI 取得並執行**；第一筆需要瀏覽器的裁判書查詢會**自動安裝 Chromium**（約 150MB，僅一次）。無需手動 `pip install` 或 `playwright install`。

第一次使用建議先跑：

```
/taiwan-legal:cold-start-interview
```

設定法院層級、日期範圍、引用格式等預設值。

## 提供的 skills（v0.1）

| Skill | 用途 |
|---|---|
| `/taiwan-legal:cold-start-interview` | 一次性設定研究預設值（法院、日期範圍、引用格式） |
| `/taiwan-legal:judgment-search` | 搜尋裁判書 / 以字號或 URL 取得單一裁判全文 |
| `/taiwan-legal:statute-lookup` | 法規查詢（依名稱、條號、關鍵字） |

v0.2 規劃：`taiwan-interpretation-lookup`（大法官解釋 / 憲判字 + 引用關係圖譜）。

## 定位

這是台灣公開法律資料的 access layer，把原始來源以 MCP 形式接到 Claude Code。資料由司法院、法務部依其開放資料政策維護；本 plugin **不修改原始內容**。為提升效能與支援離線查詢，底層 MCP server 包含少量公開資料的本地快取（如司法院釋字理由書），皆直接取自原始來源（cons.judicial.gov.tw、judgment.judicial.gov.tw、law.moj.gov.tw）並於回傳結果中標註出處 URL。原始資料依《著作權法》§9(1)(1) 不受著作權保護（屬公文／法令）；本 repo 對該等資料的結構化整理以 CC0 1.0 釋出（見 [`mcp-taiwan-legal-db`](https://github.com/lawchat-oss/mcp-taiwan-legal-db) 的 `DATA_LICENSE`）。

## 設計準則

- **誠實引用**：每一筆查詢結果都附上原始來源 URL 與字號／條號，AI 不應對引文做未經詢問的改寫。
- **不取代律師**：所有 skill 都在輸出尾端標註「不構成法律意見」，把實際法律決定交回給有執照的律師。
- **資料層／存取層分離**：資料屬於原始發布單位；我們做的是工具與整合。

## 授權

本 repo 程式碼以 MIT License 釋出。資料來源由各原始發布單位（司法院、法務部）依其開放資料政策提供。

---

Maintained by [lawchat-oss](https://github.com/lawchat-oss). Contributions welcome.
