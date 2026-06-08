# RD7 工作日誌整合系統 — 設計文件

> 版本：v2.0
> 日期：2026-06-06
> 作者：主 PM Agent
> 狀態：已部署（Phase 5 完成，Phase 6 驗證待週一）
> 審核：盧又豪（部長）、Paddy（AI二組）

---

## 1. 系統概述

### 1.1 設計目標

將 RD7 部門（11 組 + 1 主控）的工作紀錄從「自由文本」升級為「結構化追問 → 歸檔管理 → 自動蒸餾 → 知識入庫」的知識管理流水線。

### 1.2 核心設計原則

1. **日誌與知識分離**：worklog/ 是管理區，knowledge/ 是知識區，中間靠蒸餾橋接
2. **全員追問**：不分職級，所有成員的日報都經過 5 維度追問
3. **非侵入式**：成員只需回覆訊息，系統自動歸檔
4. **可溯源**：raw/ 保留原始彙整，wiki/ 存精煉結果，兩者可對照
5. **漸進過渡**：舊系統（daily-reports/）保留唯讀，新系統平行運作

### 1.3 涵蓋範圍

| 項目 | 涵蓋 |
|------|------|
| 每日工作日誌收集 | ✅ |
| 5 維度追問 + 結構化存檔 | ✅ |
| 16:00 自動提醒 | ✅ |
| 週五蒸餾（daily → raw → wiki） | ✅ |
| 跨組聯邦搜尋 | ✅（rd7-pm federated-wiki-search） |
| 歷史日報遷移 | ❌（舊資料保留原位，不遷移） |

---

## 2. 系統架構

### 2.1 整體拓撲

```
                    ┌──────────────────────────────────────────────┐
                    │           default (主 PM Agent)               │
                    │  - federated-wiki-search (跨組查詢)           │
                    │  - 部長透過此 Agent 取得全部門狀態             │
                    └────────────────────┬─────────────────────────┘
                                         │ hermes -p <profile> chat -z
                    ┌────────────────────┼─────────────────────────┐
                    ▼                    ▼                          ▼
        ┌───────────────┐    ┌───────────────┐         ┌───────────────┐
        │ rd7-group-ai  │    │ rd7-group-ai-2│   ...   │ rd7-group-tiger│
        │               │    │               │         │               │
        │ Skills:       │    │ Skills:       │         │ Skills:       │
        │ - ark-exec-   │    │ - ark-exec-   │         │ - ark-exec-   │
        │   assistant   │    │   assistant   │         │   assistant   │
        │ - ark-wiki-   │    │ - ark-wiki-   │         │ - ark-wiki-   │
        │   engine      │    │   engine      │         │   engine      │
        └───────┬───────┘    └───────┬───────┘         └───────┬───────┘
                │                    │                          │
                ▼                    ▼                          ▼
        workspace/worklog/   workspace/worklog/         workspace/worklog/
        workspace/knowledge/ workspace/knowledge/       workspace/knowledge/
```

### 2.2 組件清單

| 組件 | 類型 | 部署位置 | 用途 |
|------|------|----------|------|
| ark-executive-assistant | Skill | 全部 12 profiles | 追問 + 存檔引擎 |
| ark-wiki-engine | Skill | 全部 12 profiles | Wiki 知識庫管理 |
| federated-wiki-search | Skill | rd7-pm | 跨組聯邦搜尋 |
| worklog-reminder-* | Cron (×11) | 各組 profile | 16:00 提醒 |
| weekly-distill-* | Cron (×11) | 各組 profile | 週五 21:00 蒸餾 |
| SOUL.md 規則 | Config | 全部 12 profiles | 行為指引 |

### 2.3 Profile 列表與對應關係

| # | Profile | 組名 | 組長 | Bot |
|---|---------|------|------|-----|
| 1 | default | 主控（部長） | 盧又豪 | — |
| 2 | rd7-pm | 大PM組 | 志浩 + 小刀 | @acd_rd7_pm_bot |
| 3 | rd7-group-ai | 事業處AI組 | Hedi | @acd_rd7_group_ai_bot |
| 4 | rd7-group-ai-2 | AI二組 | Paddy | @acd_rd7_group_ai_2_bot |
| 5 | rd7-group-slot | 老虎機組 | 柏合 | @acd_rd7_group_slot_bot |
| 6 | rd7-group-platform | 平台組 | 書孟 | @acd_rd7_group_platform_bot |
| 7 | rd7-group-fish | 魚機組 | 小葉 | @acd_rd7_group_fish_bot |
| 8 | rd7-group-ops | 營運組 | Kevin | @acd_rd7_group_ops_bot |
| 9 | rd7-group-art | 美術組 | 小刀 | @acd_rd7_group_art_bot |
| 10 | rd7-group-data-analysis | 營運數據組 | 珍妮 | @acd_rd7_group_data_analysis_bot |
| 11 | rd7-group-h5-game | H5遊戲組 | 繼崴 | @acd_rd7_group_h5_game_bot |
| 12 | rd7-group-tiger | 虎爺組 | 賢名 | @acd_rd7_group_tiger_bot |

---

## 3. 目錄結構規格

### 3.1 每個 Profile 的 workspace 結構

```
~/.hermes/profiles/<profile>/workspace/
├── worklog/                              ← 工作紀錄區（wiki 外部）
│   ├── daily/
│   │   └── YYYY-MM-DD/
│   │       └── <人名>.md                 ← 每人每天一檔（追問後存入）
│   └── weekly/
│       └── YYYY-Www/
│           └── <人名>.md                 ← 每人每週一檔（週報心得）
│
├── knowledge/                            ← Wiki 知識庫
│   ├── raw/
│   │   ├── worklog-YYYY-Www.md          ← 週蒸餾原始彙整（唯讀）
│   │   └── telegram-channel-directory.md ← 通訊錄（共用參考）
│   ├── wiki/
│   │   ├── overview.md
│   │   └── weekly-digest/
│   │       └── YYYY-Www.md              ← 蒸餾精煉摘要
│   ├── schema.md
│   ├── index.md
│   └── log.md
│
└── daily-reports/                        ← [DEPRECATED] 舊系統，唯讀保留
```

### 3.2 路徑約定

| 用途 | 路徑模板 | 範例 |
|------|----------|------|
| 每日日誌 | `workspace/worklog/daily/YYYY-MM-DD/<人名>.md` | `workspace/worklog/daily/2026-06-06/Paddy.md` |
| 每週週報 | `workspace/worklog/weekly/YYYY-Www/<人名>.md` | `workspace/worklog/weekly/2026-W23/Paddy.md` |
| 蒸餾原始 | `workspace/knowledge/raw/worklog-YYYY-Www.md` | `workspace/knowledge/raw/worklog-2026-W23.md` |
| 蒸餾精煉 | `workspace/knowledge/wiki/weekly-digest/YYYY-Www.md` | `workspace/knowledge/wiki/weekly-digest/2026-W23.md` |

---

## 4. 資料流設計

### 4.1 每日日報流程

```
成員傳送工作紀錄（Telegram 私訊 or 群組）
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  ark-executive-assistant 觸發                            │
│                                                         │
│  Step 1: 簡短確認（≤15 字）                              │
│  Step 2: 5 維度評分                                     │
│           事件背景 /20 · 決策理由 /20 · 關鍵人物 /20     │
│           後續行動 /20 · 預期結果 /20                    │
│  Step 3: < 60 分 → 追問最關鍵 1 維度                    │
│  Step 4: 回答後重新評分（最多 3 輪）                     │
│  Step 5: ≥ 60 分 or 3 輪到 → 產出結構化摘要              │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│  write_file → workspace/worklog/daily/YYYY-MM-DD/<人名>.md │
│  （同日多筆 append，不覆蓋）                               │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
              回覆：✅ 已記錄（{分數}/100）
```

### 4.2 16:00 提醒流程

```
Cron Scheduler（0 16 * * 1-5）
    │
    ├─ 有群組的 Profile → deliver: telegram:<group_id>
    │   - rd7-group-ai → -5274923032
    │   - rd7-group-ai-2 → -1003992378190:1
    │   - rd7-group-fish → -1002861533983
    │   - rd7-group-art → -1003749269810
    │   - rd7-group-data-analysis → -5039625888
    │
    └─ 無群組的 Profile → deliver: telegram:<leader_id>
        - rd7-group-slot → 419724141（柏合）
        - rd7-group-platform → 566256444（書孟）
        - rd7-group-ops → 6230388397（Kevin）
        - rd7-group-h5-game → 5983638682（繼崴）
        - rd7-group-tiger → 1712957194（賢名）
        - rd7-pm → 5653475035（志浩）
```

### 4.3 週五蒸餾流程

```
Cron Scheduler（0 21 * * 5）
    │
    ▼
┌─────────────────────────────────────────────────────────┐
│  Step 1: 讀取 workspace/worklog/daily/ 本週所有 .md      │
│          （週一 ~ 週五的所有人所有條目）                    │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│  Step 2: 彙整成原始週報                                  │
│          → workspace/knowledge/raw/worklog-YYYY-Www.md   │
│          （寫入後唯讀，不可再修改）                        │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│  Step 3: LLM 精煉摘要                                   │
│          提取：重點成果、風險、決策、下週計畫              │
│          → workspace/knowledge/wiki/weekly-digest/YYYY-Www.md │
└────────────────────────────┬────────────────────────────┘
                             │
                             ▼
              deliver: local（不通知人員）
```

### 4.4 跨組查詢流程（部長端）

```
部長（default Agent）問：「AI組這週做了什麼？」
    │
    ▼
主 PM Agent 判斷路由 → rd7-group-ai
    │
    ▼
hermes -p rd7-group-ai chat -z '查詢本週 weekly-digest'
    │
    ▼
rd7-group-ai Agent 讀取 knowledge/wiki/weekly-digest/最新.md
    │
    ▼
回報結果 → 主 PM Agent 彙整 → 回覆部長
```

---

## 5. 資料格式規格

### 5.1 每日日誌 Frontmatter

```yaml
---
title: "工作日誌 YYYY-MM-DD"
author: <人名>
type: worklog
tags: [worklog, daily]
created: YYYY-MM-DD
updated: YYYY-MM-DD
completeness: <分數>/100
---
```

### 5.2 每日日誌 Body

```markdown
## HH:MM — {事件標題}

- **背景**：{一句話}
- **決策**：{做了什麼決定，為什麼}
- **關鍵人**：{誰，什麼角色}
- **下一步**：{具體行動 + 時程}
- **預期結果**：{成功標準}
- **完整度**：{分數}/100

---
```

### 5.3 週報心得 Frontmatter

```yaml
---
title: "週報 YYYY-Www"
author: <人名>
type: worklog-weekly
tags: [worklog, weekly, reflection]
created: YYYY-MM-DD
updated: YYYY-MM-DD
completeness: <分數>/100
---
```

### 5.4 蒸餾精煉 Frontmatter

```yaml
---
title: "組級週摘要 YYYY-Www"
type: synthesis
tags: [weekly-digest, worklog]
sources: [raw/worklog-YYYY-Www.md]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: mature
---
```

---

## 6. Cron 排程清單

### 6.1 每日提醒（worklog-reminder-*）

| Profile | Job ID | Schedule | Deliver | 目標說明 |
|---------|--------|----------|---------|----------|
| rd7-group-ai | c944282b4fcf | 0 16 * * 1-5 | telegram:-5274923032 | Agent資訊同步群組 |
| rd7-group-ai-2 | e744b85c6f90 | 0 16 * * 1-5 | telegram:-1003992378190:1 | RD7 Team Agent topic 1 |
| rd7-group-slot | a71020fde391 | 0 16 * * 1-5 | telegram:419724141 | 柏合 DM |
| rd7-group-platform | fbbea0ddb941 | 0 16 * * 1-5 | telegram:566256444 | 書孟 DM |
| rd7-group-fish | 7b184b8ad9fc | 0 16 * * 1-5 | telegram:-1002861533983 | 魚機組群組 |
| rd7-group-ops | eda58b40c886 | 0 16 * * 1-5 | telegram:6230388397 | Kevin DM |
| rd7-group-art | 0db1e2a2b70e | 0 16 * * 1-5 | telegram:-1003749269810 | RD7美術群組 |
| rd7-group-data-analysis | 50655bfce766 | 0 16 * * 1-5 | telegram:-5039625888 | BOT-group |
| rd7-group-h5-game | 9de824390cc5 | 0 16 * * 1-5 | telegram:5983638682 | 繼崴 DM |
| rd7-group-tiger | 4b6eae6b0051 | 0 16 * * 1-5 | telegram:1712957194 | 賢名 DM |
| rd7-pm | 937b1eef76da | 0 16 * * 1-5 | telegram:5653475035 | 志浩 DM |

### 6.2 週五蒸餾（weekly-distill-*）

| Profile | Job ID | Schedule | Deliver | Toolsets |
|---------|--------|----------|---------|----------|
| rd7-group-ai | 825842ecd8da | 0 21 * * 5 | local | file, terminal |
| rd7-group-ai-2 | e3c46115ac4e | 0 21 * * 5 | local | file, terminal |
| rd7-group-slot | 8af884fecf7c | 0 21 * * 5 | local | file, terminal |
| rd7-group-platform | 4d389c33b830 | 0 21 * * 5 | local | file, terminal |
| rd7-group-fish | c6c5277a8991 | 0 21 * * 5 | local | file, terminal |
| rd7-group-ops | 08d43884a17a | 0 21 * * 5 | local | file, terminal |
| rd7-group-art | 53ee552b1d8f | 0 21 * * 5 | local | file, terminal |
| rd7-group-data-analysis | 812adfcb4139 | 0 21 * * 5 | local | file, terminal |
| rd7-group-h5-game | a0155b00475d | 0 21 * * 5 | local | file, terminal |
| rd7-group-tiger | 52f4c9495c33 | 0 21 * * 5 | local | file, terminal |
| rd7-pm | 6f46bb8a2c48 | 0 21 * * 5 | local | file, terminal |

---

## 7. 5 維度追問機制

### 7.1 評分維度

| 維度 | 滿分 | 評估重點 |
|------|------|----------|
| 事件背景 | 20 | 起因、脈絡、為何現在發生 |
| 決策理由 | 20 | 判斷依據、考慮過的選項 |
| 關鍵人物 | 20 | 涉及誰、各自角色與立場 |
| 後續行動 | 20 | 具體下一步、時程 |
| 預期結果 | 20 | 期望產出、成功標準 |

### 7.2 追問規則

- 門檻：60 分
- 最大追問輪數：3 輪
- 每輪只追問 1 個最關鍵的缺漏維度
- 成員拒絕追問（「先這樣」「不想補了」）→ 立即停止
- 未達門檻但停止 → 存檔標記「待補充」

### 7.3 追問對應表

| 缺漏維度 | 追問方向 |
|----------|----------|
| 事件背景 | 「這件事的起因是什麼？為什麼現在發生？」 |
| 決策理由 | 「做這個決定的依據是什麼？有其他選項嗎？」 |
| 關鍵人物 | 「涉及哪些關鍵人？各自立場是什麼？」 |
| 後續行動 | 「接下來具體做什麼？時程是？」 |
| 預期結果 | 「期望帶來什麼結果？怎麼判斷成功？」 |

---

## 8. 投遞策略設計

### 8.1 決策邏輯

```
IF profile 有 type=group 的 channel
    → deliver 到群組（全組可見，促進透明度）
ELSE
    → deliver 到組長私訊（由組長轉達或自行回報）
```

### 8.2 群組 vs 私訊分類

**群組投遞（5 組）：**
- rd7-group-ai：Agent資訊同步群組 (-5274923032)
- rd7-group-ai-2：RD7 Team Agent topic 1 (-1003992378190:1)
- rd7-group-fish：魚機組 (-1002861533983)
- rd7-group-art：RD7美術 (-1003749269810)
- rd7-group-data-analysis：BOT-group (-5039625888)

**私訊投遞（6 組）：**
- rd7-group-slot → 柏合 (419724141)
- rd7-group-platform → 書孟 (566256444)
- rd7-group-ops → Kevin (6230388397)
- rd7-group-h5-game → 繼崴 (5983638682)
- rd7-group-tiger → 賢名 (1712957194)
- rd7-pm → 志浩 (5653475035)

### 8.3 蒸餾投遞

- deliver: `local`（純內部作業，結果存檔，不額外通知人員）
- 部長需要時透過 default Agent 主動查詢

---

## 9. 過渡期設計

### 9.1 新舊系統並存

| 系統 | 狀態 | 寫入 | 讀取 |
|------|------|------|------|
| workspace/daily-reports/（舊） | DEPRECATED | ❌ 停止寫入 | ✅ 唯讀保留 |
| workspace/worklog/（新） | ACTIVE | ✅ 所有新日報 | ✅ |
| workspace/knowledge/（知識庫） | ACTIVE | ✅ 僅蒸餾結果 | ✅ |

### 9.2 過渡期行為

1. SOUL.md 中舊規則標記為 `[DEPRECATED]`
2. 成員傳送舊格式日報仍接受，但 Agent 會引導新格式
3. 舊 daily-reports/ 資料不刪除、不遷移
4. 切換完成標準：連續一週全組使用新流程

---

## 10. 安全與隱私

### 10.1 存取控制

| 資料 | 可讀取者 |
|------|----------|
| 個人日誌（worklog/daily/） | 本人 + 本組 Bot + 主 PM Agent |
| 組級週摘要（weekly-digest/） | 全部門（透過聯邦搜尋） |
| 原始彙整（raw/worklog-*） | 本組 Bot + 主 PM Agent |
| 通訊錄 | 全部 Agent（參考用） |

### 10.2 資料保護原則

- worklog/ 不對外暴露（無 HTTP 存取）
- 蒸餾過程去除個人敏感資訊（由 LLM 判斷）
- Cron 產出的 raw/ 寫入後唯讀（不可修改）
- 組長 DM 投遞確保提醒不洩漏到非相關群組

---

## 11. 錯誤處理

### 11.1 常見失敗場景

| 場景 | 處理方式 |
|------|----------|
| 16:00 提醒發送失敗 | Cron 記錄 last_delivery_error，下次重試 |
| 蒸餾時 worklog/daily/ 為空 | 寫空白模板，標註「本週無日誌紀錄」 |
| 成員回覆格式無法解析 | 視為自由文本，直接進追問流程 |
| 目錄不存在 | Agent 自動 mkdir -p |
| deliver 目標無效 | 檢查 channel_directory.json 更新 |

### 11.2 監控指標（未來擴充）

- 每日日誌收集率（有日誌人數 / 總人數）
- 平均完整度分數
- 蒸餾成功率
- 提醒投遞成功率

---

## 12. 擴充規劃

### 12.1 短期（1-2 週）

- Phase 6 驗證通過後正式啟用
- 根據實際使用調整追問問題
- 觀察成員接受度，必要時放寬門檻

### 12.2 中期（1-2 月）

- 加入日報收集率統計 cron（每日統計哪些人未回報）
- weekly-digest 自動通知部長（改 deliver 為 telegram:605575718）
- 跨組 weekly-digest 彙整成部門月報

### 12.3 長期（3+ 月）

- 基於累積的 weekly-digest 建立團隊知識圖譜
- 風險模式辨識（連續低分、連續未回報）
- 自動標記高價值決策進入永久知識庫

---

## 13. 相關文件索引

| 文件 | 路徑 | 說明 |
|------|------|------|
| 通訊錄 | workspace/knowledge/raw/telegram-channel-directory.md | 群組 & 私訊 ID + 通報流程 |
| 本設計文件 | workspace/knowledge/raw/worklog-system-design.md | 本文件 |
| ark-executive-assistant | ~/.hermes/skills/ark-executive-assistant/SKILL.md | 追問引擎 Skill |
| ark-wiki-engine | ~/.hermes/skills/ark-wiki-engine/SKILL.md | Wiki 引擎 Skill |
| federated-wiki-search | profiles/rd7-pm/skills/federated-wiki-search/SKILL.md | 跨組搜尋 Skill |
| 執行計畫（原始） | ~/work-logs/worklog-integration-plan.md | Phase 1-6 步驟 |

---

## 14. 變更紀錄

| 日期 | 版本 | 變更內容 |
|------|------|----------|
| 2026-06-01 | v1.0 | 初始計畫（worklog-integration-plan.md） |
| 2026-06-06 | v2.0 | 正式設計文件：完成 Phase 1-5 部署後整理 |
| — | — | 蒸餾時間從 17:30 改為 21:00 |
| — | — | 提醒投遞改為群組優先（有群組發群組，無群組發組長） |
