# RD7 Telegram 通訊錄（群組 & 私訊 ID）

> 資料來源：各 Profile channel_directory.json
> 更新日期：2026-06-06

---

## 通報流程說明

### 每日工作日誌提醒（週一至週五 16:00）

```
┌─────────────┐     16:00 cron      ┌──────────────────┐
│  Scheduler  │ ──────────────────▶  │  各組 Agent Bot  │
└─────────────┘                      └────────┬─────────┘
                                              │
                              ┌────────────────┼────────────────┐
                              ▼                ▼                ▼
                      【有群組的組】     【無群組的組】
                       發到群組            發到組長私訊
                              │                │
                              ▼                ▼
                      組員在群組/私訊中直接回覆日誌
                              │                │
                              ▼                ▼
                      ┌──────────────────────────────┐
                      │  Agent 收到回覆後自動：        │
                      │  1. 五維度追問（補全資訊）      │
                      │  2. 歸檔至 workspace/worklog/ │
                      │     daily/YYYY-MM-DD/<人名>.md │
                      └──────────────────────────────┘
```

### 週五蒸餾流程（每週五 21:00）

```
┌─────────────────────────────────────────────────────────────┐
│  workspace/worklog/daily/                                    │
│  （本週一～週五所有日誌）                                      │
└──────────────────────────┬──────────────────────────────────┘
                           │ Agent 讀取 + LLM 彙整
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  workspace/knowledge/raw/worklog-YYYY-Www.md                 │
│  （原始彙整，唯讀保存）                                        │
└──────────────────────────┬──────────────────────────────────┘
                           │ Agent 精煉摘要
                           ▼
┌─────────────────────────────────────────────────────────────┐
│  workspace/knowledge/wiki/weekly-digest/YYYY-Www.md          │
│  （精煉版：重點成果、風險、決策、下週計畫）                      │
└─────────────────────────────────────────────────────────────┘
```

### 投遞規則

1. **有群組 → 發群組**：讓全組可見，促進透明度
2. **無群組 → 發組長私訊**：由組長轉達或自行回報
3. **蒸餾結果 → local**：內部作業，不額外通知人員
4. **跨組協調 → Kanban 任務**：主 PM Agent 建立任務，透過 notify-subscribe 通知部長

### 資料流向總覽

```
組員回覆日誌
    ↓
各組 Agent 追問 + 歸檔（workspace/worklog/daily/）
    ↓ 週五 21:00
蒸餾：raw/worklog-YYYY-Www.md（原始）
    ↓
蒸餾：wiki/weekly-digest/YYYY-Www.md（精煉）
    ↓ 主 PM Agent 可跨組查詢
部長透過 default Agent 取得全部門週報
```

---

## 群組清單

| Profile | 群組名稱 | Chat ID | Thread ID | 備註 |
|---------|----------|---------|-----------|------|
| default | Agent 高峰論壇 | -5213753834 | — | |
| rd7-group-ai | Agent資訊同步群組 | -5274923032 | — | |
| rd7-group-ai-2 | RD7 Team Agent | -1003992378190 | 1 | topic 1 |
| rd7-group-ai-2 | RD7 Team Agent | -1003992378190 | 5 | topic 5 |
| rd7-group-ai-2 | Kevin | -4842109351 | — | |
| rd7-group-ai-2 | Kevin | -1003594282389 | 1 | topic 1 |
| rd7-group-ai-2 | Kevin | -1003594282389 | 2 | topic 2 |
| rd7-group-ai-2 | Kevin | -1003594282389 | 23 | topic 23 |
| rd7-group-ai-2 | Agent 高峰論壇 | -5213753834 | — | |
| rd7-group-fish | 魚機組 | -1002861533983 | — | 主群 |
| rd7-group-fish | 魚機組 | -1002861533983 | 1 | topic 1 |
| rd7-group-fish | 魚機組 | -1002861533983 | 8432 | topic 8432 |
| rd7-group-fish | 魚機組 | -1002861533983 | 8436 | topic 8436 |
| rd7-group-fish | 魚機組 | -1002861533983 | 8441 | topic 8441 |
| rd7-group-art | RD7美術 | -1003749269810 | 1 | topic 1 |
| rd7-group-art | RD7美術 | -1003749269810 | 50 | topic 50 |
| rd7-group-art | RD7美術 | -1003749269810 | 228 | topic 228 |
| rd7-group-data-analysis | BOT-group | -5039625888 | — | |

---

## 私訊清單（DM）

| Profile | 姓名 | Telegram ID | 角色 |
|---------|------|-------------|------|
| default | 盧又豪 | 605575718 | 部長 |
| default | Paddy #2141 | 937896656 | AI二組組長 |
| default | Kevin(繼崴) | 5983638682 | H5遊戲組 |
| rd7-pm | 盧又豪 | 605575718 | 部長 |
| rd7-pm | 志浩 | 5653475035 | PM組長 |
| rd7-pm | 小刀 黃 | 618295124 | PM / 美術組長 |
| rd7-group-ai | 盧又豪 | 605575718 | 部長 |
| rd7-group-ai | Kevin(繼崴) | 5983638682 | |
| rd7-group-ai | Leo Lin | 568160123 | |
| rd7-group-ai | Hedi Ho | 972628949 | AI組組長 |
| rd7-group-ai | Paul Yao | 1640096988 | |
| rd7-group-ai-2 | 盧又豪 | 605575718 | 部長 |
| rd7-group-ai-2 | Paddy #2141 | 937896656 | AI二組組長 |
| rd7-group-ai-2 | Leo Lin | 568160123 | |
| rd7-group-ai-2 | Kevin(繼崴) | 5983638682 | |
| rd7-group-slot | 盧又豪 | 605575718 | 部長 |
| rd7-group-slot | Bowen Su | 419724141 | 老虎機組組長（柏合） |
| rd7-group-platform | 盧又豪 | 605575718 | 部長 |
| rd7-group-platform | 陳 摸摸 | 614037579 | MoMo |
| rd7-group-platform | BookDream(書孟) | 566256444 | 平台組組長 |
| rd7-group-fish | 盧又豪 | 605575718 | 部長 |
| rd7-group-fish | 小葉 | 966434356 | 魚機組組長 |
| rd7-group-ops | 盧又豪 | 605575718 | 部長 |
| rd7-group-ops | Kevin🌀 Game🔥 AI Agent🤖 | 6230388397 | 營運組組長 |
| rd7-group-art | 盧又豪 | 605575718 | 部長 |
| rd7-group-art | 小刀 黃 | 618295124 | 美術組組長 |
| rd7-group-data-analysis | 盧又豪 | 605575718 | 部長 |
| rd7-group-data-analysis | Fany 珍妮 | 679194495 | 營運數據組組長 |
| rd7-group-h5-game | 盧又豪 | 605575718 | 部長 |
| rd7-group-h5-game | Kevin(繼崴) | 5983638682 | H5遊戲組 |
| rd7-group-h5-game | 李 智傑 | 1646708824 | |
| rd7-group-tiger | 盧又豪 | 605575718 | 部長 |
| rd7-group-tiger | 賢名 潘 | 1712957194 | 虎爺組組長 |

---

## 提醒 Cron 投遞目標摘要

| Profile | 投遞方式 | 目標 ID |
|---------|----------|---------|
| rd7-group-ai | 群組 | telegram:-5274923032 |
| rd7-group-ai-2 | 群組 topic 1 | telegram:-1003992378190:1 |
| rd7-group-fish | 群組 | telegram:-1002861533983 |
| rd7-group-art | 群組 topic 1 | telegram:-1003749269810 |
| rd7-group-data-analysis | 群組 | telegram:-5039625888 |
| rd7-group-slot | 組長私訊 | telegram:419724141 |
| rd7-group-platform | 組長私訊 | telegram:566256444 |
| rd7-group-ops | 組長私訊 | telegram:6230388397 |
| rd7-group-h5-game | 組長私訊 | telegram:5983638682 |
| rd7-group-tiger | 組長私訊 | telegram:1712957194 |
| rd7-pm | 組長私訊 | telegram:5653475035 |

---

## 無群組的 Profile（僅私訊）

- rd7-pm
- rd7-group-slot
- rd7-group-platform
- rd7-group-ops
- rd7-group-h5-game
- rd7-group-tiger
