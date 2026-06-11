# 🐟 魚機組 Fish Agent Team

魚機組團隊工作管理系統，由 Hermes Agent 驅動。

## 📂 專案結構

```
workspace/
├── README.md              ← 本檔案
├── wiki/                  ← GitHub Wiki 鏡像
│   ├── Home.md
│   ├── daily-reports/     ← 每日工作日報
│   ├── management-log/    ← 管理日誌
│   └── knowledge/         ← 知識庫索引
├── daily-reports/         ← 原始工作日報存檔
│   ├── 2026-06-10/        ← 仲仁、奕德、奕証、YYN、李文勛、智傑、琮偉、小葉
│   └── 2026-06-11/        ← 小葉、summary
├── management-log/        ← 管理日誌
│   ├── 2026-06-08/        ← 小葉
│   ├── 2026-06-09/        ← 小葉
│   ├── 2026-06-10/        ← 小葉
│   └── 2026-06-11/        ← 小葉（Slotverse 交接、研五 Boss 模型、AI 五大方向）
├── knowledge/             ← 知識庫
│   ├── client/            ← Client 研發知識（10 核心 + 11 手冊）
│   ├── wiki/              ← 結構化知識頁面（knowhow、翻譯）
│   ├── raw/               ← 原始資料（唯讀）
│   ├── group_directory.md ← 群組目錄
│   ├── agent_directory.md ← Agent 目錄
│   └── h5_fish_game_workflow.md ← H5 魚機工作流程
├── contacts/              ← 通訊錄（symlink）
└── a2a-audit/             ← A2A 跨組通訊紀錄
```

## 🤖 Agent 功能

- 團隊工作日報收集與歸檔（5 維度追問）
- 管理日誌 CRUD + 陳總視角整理
- Wiki 知識庫管理（Client、Knowhow、翻譯）
- 跨組 Kanban 任務協作（A2A Protocol）
- 管理週報 / 咖啡會報告生成
- 跨組 Agent 通訊監控

## 👥 團隊成員

小葉（PM Lead）、仲仁、奕德、奕証、YYN、李文勛、智傑、琮偉、羽農、宗承

## 📋 更新紀錄

| 日期 | 更新內容 |
|------|----------|
| 2026-06-11 | 新增管理日誌（Slotverse 交接、研五 Boss 模型、部門 AI 五大方向）；同步 Wiki；新增知識庫索引 |
| 2026-06-10 | 新增 8 人日報（仲仁、奕德、奕証、YYN、李文勛、智傑、琮偉、小葉）；管理日誌 |
| 2026-06-09 | 管理日誌（Boss 預測模型、代理人溝通模式） |
| 2026-06-08 | 初始化 repo，建立基本結構，Wiki 初始化 |

---
*Maintained by 魚機組 Hermes Agent (rd7-group-fish)*
