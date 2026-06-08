---
title: "Wiki Schema v3.0"
type: system
tags: [schema, system]
created: 2026-06-05
updated: 2026-06-05
status: mature
---

# Wiki Schema v3.0

## 目錄結構規則

```
knowledge/wiki/
├── raw/          → 唯讀原始資料（LLM 只讀不改）
├── wiki/         → 結構化知識（LLM 維護）
│   ├── overview.md
│   └── {category}/
│       └── {page}.md
├── schema.md     → 本規則定義
├── index.md      → 索引目錄
└── log.md        → 操作日誌（append-only）
```

## 頁面 Frontmatter 規則

```yaml
---
title: "頁面標題"
type: concept | entity | source | synthesis | comparison | overview | system | worklog | worklog-summary
tags: [tag1, tag2]
sources: [raw/來源檔案]
related: [相關頁面檔名]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: seedling | developing | mature
---
```

## 必要欄位

| 欄位 | 必要 | 說明 |
|------|------|------|
| title | ✅ | 頁面標題（繁體中文） |
| type | ✅ | 頁面類型（見上方合法值） |
| tags | ✅ | 分類標籤（陣列） |
| sources | 建議 | 來源 raw 檔案 |
| related | 建議 | 相關頁面（用於圖譜） |
| created | ✅ | 建立日期 YYYY-MM-DD |
| updated | ✅ | 最後更新日期 YYYY-MM-DD |
| status | 建議 | seedling / developing / mature |

## 雙向連結

使用 `[[頁面檔名]]`（不含 .md、不含路徑）建立雙向連結。

## 特殊標記

- 矛盾：`> ⚠️ **矛盾**：來源 A 說 X，來源 B 說 Y，待釐清。`
- 不確定：`(?)` 標記
- 禁止自行解決矛盾，只能標記

## Worklog 特殊規則

- 日誌存至 `wiki/worklog/YYYY-MM-DD.md`，同日 append
- 週報存至 `wiki/worklog/weekly-YYYY-Www.md`
- log.md 為 append-only，禁止刪除舊記錄
