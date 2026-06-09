Telegram reactions: ⏳ on processing start → ✅ success / 👎 fail / clear on NOOP. NOOP = ProcessingOutcome.NOOP in base.py when delivery_attempted=False. 靜默 thread channel_prompt 必須說「完全不發送任何文字」。
§
群組 chat_id 對照表已存於 workspace/knowledge/group_directory.md。Bot 已在所有 25 個群組設定靜默觀察 channel_prompt（只有被 @ 或 reply 才回應）。魚機組主群 -1002861533983 另有 Grill-me 日報追問機制。
§
Client 研發知識庫已存於 workspace/knowledge/client/（core/ 10 個 + manual/ 11 個 .md 檔），內容涵蓋魚機與彈珠台的系統導覽、機台知識、Server 協定、新增魚種/房間流程、Prefab 規範、作弊工具等。
§
已安裝的 Skills（rd7-group-fish profile）：
- ark-executive-assistant — 工作日誌追問 + 日結提醒 + 週報彙整
- ark-wiki-engine — Wiki 知識庫引擎規格
- ark-weekly-report-reviewer — 週報 7 維度點評（部長校準版）
- ark-management-weekly-report — 管理週報生成（陳總視角）
- help-menu — /help 指令，顯示所有功能清單

Wiki 知識庫已初始化：workspace/knowledge/wiki/（schema.md v3.0、index.md、log.md、wiki/overview.md）
小葉歷史週報 16 筆已匯入：wiki/raw/weekly_reports/（3月〜9月 2025 + 2026年）
§
跨組 Agent 監控名單（user_id → 姓名/組別）：
5653475035 志浩/大PM、972628949 Hedi/AI組、937896656 Paddy/AI二組、419724141 柏合/老虎機組、566256444 書孟/平台組、614037579 MoMo/平台組、6230388397 Kevin/營運組、618295124 小刀/美術組、679194495 珍妮/營運數據組、1712957194 賢名/虎爺組。
完整目錄：workspace/knowledge/agent_directory.md
監控行為：跨組 user_id 來訊 → 正常回應 → DM 通知小葉（605575718），格式：來源/問題摘要/回答摘要。
§
魚機組成員名字：小崴（非「小葳」）。小崴是 AI 移植製程的主要執行+培育對象，與小G搭配。首批 AI 移植標的：魚機廳館＝富貴乾坤館、類魚機廳館＝史前紀元館（皆小崴+小G負責，與 Paddy AI小組合作）。
§
GitHub repo: github.com/yajinyee/fish-agent-team (main branch)。workspace = git repo root。觸發 commit+push 條件：功能更新、知識庫新增/修改、日報、管理日誌變更。Wiki 在 wiki/ 目錄下。README.md 為專案總覽。
§
魚機組專案日報是整組一起寫的，不分人。存檔時使用「團隊日報.md」（路徑：workspace/daily-reports/YYYY-MM-DD/團隊日報.md）。
§
A2A 通訊鐵律：所有跨組資料收集（收日報、查進度、問某人、協查）一律走 Kanban 流程（kanban create → notify-subscribe → 等 dispatcher）。禁止直接讀其他組 workspace 或用 hermes chat 繞過。notifier-profile 用自己的 profile 名稱（不可用 default），chat-id 填發起人 user_id。
§
核心互動三原則：(1) 對 13 個 RD7 bot（@acd_rd7_pm_bot、pm2、ai、ai_2、ai_3、slot、platform、platform_2、ops、art、data_analysis、tiger、slotverse）一律走 A2A 協議格式。(2) 自然語言問答以陳總人格高度調整論述，優先參考本地知識庫，可上網但須標明來源。(3) 真實性最高原則，禁止虛假，不確定就說不確定。