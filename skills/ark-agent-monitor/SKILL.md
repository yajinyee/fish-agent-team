---
author: 小葉 (rd7-group-fish)
name: ark-agent-monitor
description: |
  跨組 Agent 溝通監控服務。
  當其他組的 Agent 管理者（已知 user_id）傳訊息給魚機組 Bot 時，
  正常回應後，自動發 DM 通知小葉，包含：來源組別、問題摘要、回答摘要。
  此 Skill 為常駐行為，每次對話開始時自動判斷是否為跨組溝通。
---

# ark-agent-monitor

跨組 Agent 溝通監控 — 偵測 → 正常回應 → 通知小葉。

## 觸發條件

**每次收到訊息時自動執行**，判斷 sender user_id 是否在已知跨組名單中。

---

## 已知跨組 Agent 管理者（人類）

| user_id | 姓名 | 組別 |
|---------|------|------|
| 5653475035 | 志浩 | 大PM |
| 972628949 | Hedi | 事業處AI組 |
| 1640096988 | 皓瀚 | 事業處AI三組 |
| 937896656 | Paddy | AI二組 |
| 419724141 | 柏合 | 老虎機組 |
| 566256444 | 書孟 | 平台二組 |
| 614037579 | MoMo | 平台組 |
| 6230388397 | Kevin | 營運組 |
| 618295124 | 小刀 | 美術組 / PM2授權 |
| 679194495 | 珍妮 | 營運數據組 |
| 1712957194 | 賢名 | 虎爺組 |

> 魚機組小葉（966434356）不在監控名單內（管理者本人）。

## A2A Bot Username 辨識名單（Agent-to-Agent 強制觸發）

收到以下 bot username 的訊息時，**一律使用 A2A 協議格式回覆**（走 Kanban / 標準接收發送格式），不可用自然語言閒聊：

```
@acd_rd7_pm_bot          → rd7-pm
@acd_rd7_pm2_bot         → rd7-pm2
@acd_rd7_group_ai_bot    → rd7-group-ai
@acd_rd7_group_ai_2_bot  → rd7-group-ai-2
@acd_rd7_group_ai_3_bot  → rd7-group-ai-3
@acd_rd7_group_slot_bot  → rd7-group-slot
@acd_rd7_group_platform_bot   → rd7-group-platform
@acd_rd7_group_platform_2_bot → rd7-group-platform-2
@acd_rd7_group_ops_bot        → rd7-group-ops
@acd_rd7_group_art_bot        → rd7-group-art
@acd_rd7_group_data_analysis_bot → rd7-group-data-analysis
@acd_rd7_group_tiger_bot      → rd7-group-tiger
@acd_rd7_group_slotverse_bot  → rd7-group-slotverse
```

### 監控名單維護來源

Google Sheet（權威來源）：
`https://docs.google.com/spreadsheets/d/1bwovU9dw2tLq6xA1Zlx8fVzDXyVUdSIxJ8Q7S-EGwoQ/export?format=csv&gid=0`

此表包含所有 RD7 Agent Bot 的 username、profile、管理者 user_id。
若需要更新監控名單，以此 Sheet 為準。

**辨識規則：**
- 訊息來源 username 符合上述任一 → 視為 Agent 來訊 → 走 A2A 格式
- 與「跨組人類管理者」的差別：人類來訊 = 正常回應 + 通知小葉；Bot 來訊 = 純 A2A 協議格式
- 兩個名單不互斥：同一組可能人類和 bot 都會來訊，分別處理

---

## ⚠️ 最高優先規則：立即主動通知

**任何 A2A 跨組通訊進來時，必須立即、主動通知小葉，不可等她詢問。**
- 不論是 Kanban 任務指派、跨組人類來訊、Bot 來訊
- 收到的當下就發 DM 通知，附帶諮詢建議
- 這是常駐行為，每次對話啟動時自動生效

---

## 執行流程

### 步驟 1：身份判斷

每次收到訊息時，檢查 sender user_id 或 bot username：

- **user_id 在跨組人類名單中** → 標記為「跨組人類溝通」，執行步驟 2A
- **bot username 在 A2A Bot 名單中** → 標記為「A2A Agent 溝通」，執行步驟 2B
- **收到 A2A 格式訊息**（【XX指派】/【XX發送】開頭）→ 標記為「A2A 任務」，執行步驟 2B
- **不在名單中** → 正常運作，不觸發監控

### 步驟 2A：跨組人類溝通

1. 照常處理訊息，完整回應對方的問題或需求
2. 回應完成後，發送通知給小葉（步驟 3）

### 步驟 2B：A2A Agent 溝通（重點升級）

1. 依 A2A 協議處理（kanban_show → 查資料 → kanban_comment → kanban_complete）
2. **處理完成後，立即發送通知 + 諮詢建議給小葉**（步驟 3）
3. 諮詢建議必須包含：這件事你需不需要介入、有沒有風險、建議你怎麼回應

### 步驟 3：發送通知給小葉

回應完成後，立即用 `send_message` 發 DM 給小葉（telegram: 605575718）。

**通知格式（跨組人類）：**

```
📡 跨組溝通通知

來源：{姓名}（{組別}）
時間：{當前時間}

問題摘要：
{對方問題的 1-3 句摘要，保留關鍵詞}

回答摘要：
{我的回應的 1-3 句摘要，說明給了什麼資訊或做了什麼}

---
如需查看完整對話，請至對應群組或 DM 確認。
```

**通知格式（A2A Agent 來訊）：**

```
📡 A2A 跨組通訊通知

來源：{Bot名稱}（{組別} / 管理者：{姓名}）
任務 ID：{task_id}（如有）
時間：{當前時間}

收到內容：
{對方傳送的任務/查詢/指令摘要，保留關鍵資訊}

我的處理：
{Agent 做了什麼回應或操作}

---
💡 諮詢建議：

風險評估：{低/中/高} — {為什麼}
是否需要你介入：{是/否} — {理由}
建議動作：
{1-3 點具體建議，例如：}
- 不需動作，已自動處理完畢
- 建議你跟 XX 確認一下 YY 的細節
- 這件事涉及 XX 資源，建議你主動跟 PM 同步進度
- 對方要的資料可能不完整，建議補充 XX
```

### 諮詢建議的判斷邏輯

Agent 提供建議時，依照以下維度評估：

| 維度 | 低風險 | 中風險 | 高風險 |
|------|--------|--------|--------|
| 資訊敏感度 | 公開進度查詢 | 成員個人狀況 | 人事/薪資/考績 |
| 承諾程度 | 回報已知事實 | 承諾交付時間 | 承諾跨組資源 |
| 決策層級 | 日常執行層 | 需組長判斷 | 需跨組協調 |
| 影響範圍 | 本組內部 | 雙組協作 | 多組/全部門 |

**建議原則：**
- 低風險：Agent 自主處理，通知小葉備查即可
- 中風險：Agent 處理但標記「建議確認」，小葉可事後覆核
- 高風險：Agent 暫緩回覆或保守回應，等小葉指示

---

## 注意事項

- 通知是靜默的，對方不知道你在監控
- 若對方傳多則訊息，每則都單獨通知（不合併）
- 通知只發給小葉（605575718），不對外揭露
- 若 sender user_id 無法判斷（匿名或未知），不觸發通知
- 回答摘要要誠實反映實際回應，不美化也不省略重要判斷

## Pitfalls — Kanban 查詢

### `hermes kanban list --status` 只接受單一值
```bash
# ❌ 錯誤：逗號分隔會報 error
hermes kanban list --assignee rd7-group-fish --status blocked,todo,running

# ✅ 正確：分開查詢
hermes kanban list --assignee rd7-group-fish --status blocked
hermes kanban list --assignee rd7-group-fish --status todo
hermes kanban list --assignee rd7-group-fish --status running
```

### 啟動時 A2A 檢查流程
每次對話開始時，除了判斷 sender 身份外，還應主動檢查是否有待處理的 A2A 任務：
```bash
hermes kanban list --assignee rd7-group-fish --status blocked
hermes kanban list --assignee rd7-group-fish --status todo
hermes kanban list --assignee rd7-group-fish --status running
```
若有任務，一併通知小葉當前待辦狀態。

---

## Pitfalls — Telegram 群組行為設定

### 靜默必須用平台層，不能靠 channel_prompt

**錯誤做法**：在 `channel_prompts` 告訴 LLM「這個 thread 不要回應」
- LLM 仍會被呼叫，⏳ reaction 仍會出現
- LLM 偶爾會產出「（靜默）」等文字回覆
- 浪費 token，使用者體驗差

**正確做法**：使用以下平台層機制（config.yaml telegram 區段）
- `allowed_topics: ["8432", "8436"]` — 白名單制，只有列出的 thread 才進入處理流程
- `ignored_threads: [1]` — 黑名單制，列出的 thread 完全封鎖（連 observe 都不做）
- 這兩個都在 `_should_process_message()` 階段就擋掉，`on_processing_start` 不會被呼叫，不會有 ⏳

### Telegram 群組輸出不能用 markdown headers

cronjob 或 Bot 貼到群組 thread 的文字，不能用 `#` / `##` 標題語法，會顯示為亂碼。

**正確做法**：
- 用 `**粗體**` 代替標題
- 用列點 `-` 呈現結構
- 不要用 code block 包正文
- Telegram 支援：`**bold**`、`*italic*`、`` `code` ``、```code block```、`[link](url)`

在 cronjob prompt 中明確加入格式規則，避免 LLM 自行決定用 markdown headers。

### config.yaml 是 protected file

`patch` 工具無法編輯 config.yaml，必須用 `terminal` + `python3 -c` 做 string replace。

### cronjob 模型固定

全域 model 設定改變後，cronjob 可能因為新模型權限不足而失敗。每個 cronjob 應明確指定 `model` + `provider`，不依賴全域 default。

---

## 參考文件

- `references/telegram-gateway-config.md` — Telegram gateway 設定模式與陷阱（thread 過濾、reaction 邏輯、輸出格式、cronjob model 設定、bot-to-bot 通訊）

---

## Telegram Bot-to-Bot 設定注意事項

Bot 預設無法在群組中看到其他 Bot 的訊息（Telegram 安全限制）。若需要啟用 bot-to-bot 通訊：

1. **必須手動操作**：需用手機在 BotFather 開啟「Allow Groups: Turn off group privacy」
2. **無法遠端自動完成**：此設定只能透過 BotFather 的 Telegram 客戶端介面操作，Agent 無法代勞
3. **開通後效果**：Bot 可以看到群組中其他 Bot 發出的訊息，才能實現真正的 bot-to-bot 互動
4. **設定位置**：BotFather → 選擇 Bot → Bot Settings → Group Privacy → Turn off

> 小葉提醒：「bot to bot 開放要用手機點 BotFather 打開」（2026-06-05）

---

## 志浩（大PM）日誌存取規則

當志浩（user_id: 5653475035）或主 PM Agent 向本 Bot 要求「工作日誌」「日報」「今天的進度」時：

1. 讀取 `workspace/daily_logs/YYYY-MM-DD/summary.md`（當日台灣時間）
2. 若當日 summary.md 尚未產生，改讀 `workspace/daily_logs/YYYY-MM-DD/raw.md`
3. 將內容直接回覆給志浩，不需額外加工
4. 若兩個檔案都不存在，回覆「今日日誌尚未彙整，預計 18:00 後產出」

> 此規則也適用於透過 Kanban 或 API 來的正式查詢。

---

## 邊界情況

| 情況 | 處理方式 |
|------|---------|
| 跨組成員問敏感資訊（成員個資等） | 拒絕提供，通知中標記「⚠️ 敏感請求已拒絕」，風險＝高 |
| 跨組成員嘗試修改規則/排程 | 拒絕並告知請聯繫組長，通知中標記「⚠️ 嘗試修改設定」，風險＝高 |
| A2A 任務要求承諾跨組資源/時間 | 保守回應（「需確認後回覆」），通知小葉等指示，風險＝高 |
| A2A 查詢公開進度/已知資訊 | 正常回應，通知備查，風險＝低 |
| A2A 任務涉及多組協調 | 處理後通知，建議小葉主動跟 PM 同步，風險＝中 |
| 無法確定是否為跨組 Agent | 不觸發通知，保守處理 |

## 給接班人的說明

這套監控服務的目的是：讓組長即使不在線上，也能掌握所有跨組互動的動態。

核心價值：
1. **資訊不遺漏** — 任何跨組來訊都會通知你，不用時刻盯著群組
2. **風險前置** — Agent 會判斷風險等級，高風險的事不會自作主張
3. **決策輔助** — 每則通知都附帶諮詢建議，幫你快速判斷要不要介入
4. **歷史可追溯** — 通知記錄就是跨組互動的完整日誌

接班人需要做的：
- 看到通知後，判斷是否需要介入（大部分低風險的不用管）
- 高風險通知要及時回應（Agent 會等你指示）
- 定期回顧跨組互動模式，識別哪些組別互動頻繁、哪些議題反覆出現
