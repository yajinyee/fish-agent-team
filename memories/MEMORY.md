Telegram reactions: ⏳→✅/👎/clear(NOOP). 靜默 channel_prompt 說「完全不發送任何文字」。流程鐵律：「收到」≠任務完成，只做 comment+通知發起人，維持 blocked 直到交付物提交才 complete。
§
群組 chat_id 對照表已存於 workspace/knowledge/group_directory.md。Bot 已在所有 25 個群組設定靜默觀察 channel_prompt（只有被 @ 或 reply 才回應）。魚機組主群 -1002861533983 另有 Grill-me 日報追問機制。
§
Client 研發知識庫已存於 workspace/knowledge/client/（core/ 10 個 + manual/ 11 個 .md 檔），內容涵蓋魚機與彈珠台的系統導覽、機台知識、Server 協定、新增魚種/房間流程、Prefab 規範、作弊工具等。
§
知識庫：Wiki workspace/knowledge/wiki/、週報 wiki/raw/weekly_reports/。咖啡會排班：workspace/knowledge/coffee-schedule.md（每6h同步），提醒週日+週二17:00發telegram:-1003903467272:170。問排班讀此檔。咖啡會topic互動規則：平時靜默，只有小葉@才動；被@要題材時從管理日誌、早會報告、日報整理多個主題選項有條理列出讓她挑。
§
跨組 Agent 監控（升級版，2026-06-09）：
人類名單：5653475035志浩/大PM、972628949Hedi/AI組、1640096988皓瀚/AI三組、937896656Paddy/AI二組、419724141柏合/老虎機組、566256444書孟/平台二組、614037579MoMo/平台組、6230388397Kevin/營運組、618295124小刀/美術組+PM2、679194495珍妮/營運數據組、1712957194賢名/虎爺組。
Bot 來訊也監控（A2A格式），處理後通知小葉（605575718）附諮詢建議（風險評估+是否需介入+建議動作）。
權威來源 Google Sheet: 1bwovU9dw2tLq6xA1Zlx8fVzDXyVUdSIxJ8Q7S-EGwoQ
§
魚機組成員名字：小崴（非「小葳」）。小崴是 AI 移植製程的主要執行+培育對象，與小G搭配。首批 AI 移植標的：魚機廳館＝富貴乾坤館、類魚機廳館＝史前紀元館（皆小崴+小G負責，與 Paddy AI小組合作）。
§
GitHub repo: github.com/yajinyee/fish-agent-team (main branch)。workspace = git repo root。觸發 commit+push 條件：功能更新、知識庫新增/修改、日報、管理日誌變更。Wiki 在 wiki/ 目錄下。README.md 為專案總覽。
§
魚機組專案日報是整組一起寫的，不分人。存檔時使用「團隊日報.md」（路徑：workspace/daily-reports/YYYY-MM-DD/團隊日報.md）。
§
A2A 鐵律：跨組一律走 Kanban（create→notify-subscribe→等 dispatcher）。notifier-profile 用自己 profile，chat-id 填發起人 user_id。例外：單向回報可直接 hermes -p default/rd7-pm2 chat -q。回覆 PM 格式：【魚機組回覆—主題】+編號要點。
§
核心互動三原則：(1) 對 13 個 RD7 bot（@acd_rd7_pm_bot、pm2、ai、ai_2、ai_3、slot、platform、platform_2、ops、art、data_analysis、tiger、slotverse）一律走 A2A 協議格式。(2) 自然語言問答以陳總人格高度調整論述，優先參考本地知識庫，可上網但須標明來源。(3) 真實性最高原則，禁止虛假，不確定就說不確定。
§
代理人對上溝通模式（2026-06-09 確立）：小葉給草稿要點 → Agent 用三層結構（事實+意義+行動）升級論述 → 送出 → 記錄反饋回報。Skill: ark-pm-communication。知識頁: wiki/knowhow/agent-as-proxy-communication.md。
§
「部長」= 志浩（大PM，user_id 5653475035，profile: default）。小葉提到「部長」時指的是志浩。
§
/id 指令：收到 /id 時，回覆對方的 Telegram Chat ID 和名稱。格式：「你的 Chat ID: {user_id}\n名稱: {顯示名稱}」。user_id 從 session context 的 User 欄位取得。