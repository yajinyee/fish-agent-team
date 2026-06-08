---
inclusion: always
---

# Steering 文件索引 (Steering Documents Index)

本文件列出所有可用的 Steering 文件，幫助 Agent 和使用者快速找到正確的指南。

---

## 文件結構

```
.kiro/steering/
├── core/           # 自動載入 (inclusion: always)
│   ├── Auto_fishhunter_steering_index.md      ← 本文件
│   ├── Auto_fishhunter_machine_expert.md      ← OceanTreasure (2D) 專用
│   ├── Auto_dinopinball_machine_expert.md     ← DinoPinBall (3D) 專用
│   ├── Auto_fishhunter_system_guide.md        ← OceanTreasure (2D) API 專用
│   ├── Auto_dinopinball_system_guide.md       ← DinoPinBall (3D) API 專用
│   ├── Auto_fishhunter_server_protocol.md
│   ├── product.md
│   ├── spec_splitting_rule.md             ← 大型 Spec 拆分規則
│   ├── tech.md
│   └── structure.md
│
└── manual/         # 手動觸發 (inclusion: manual 或 fileMatch)
    ├── cheattool_v2_guide.md              ← CheatTool V2 開發規範（fileMatch: CheatTool/**）
    ├── dinopinball_addmonster_expert.md
    ├── dinopinball_cheattool_v2_guide.md   ← DinoPinBall CheatTool V2 專用（fileMatch: DinoPinBall/**/CheatTool/**）
    ├── dinopinball_monseter_prefabs_standard.md
    ├── fishhunter_addfish_expert.md
    ├── fishhunter_cheat_tool_guide.md
    ├── fishhunter_code_templates.md
    ├── fishhunter_fake_summon_guide.md
    ├── fishhunter_fish_prefabs_standard.md
    ├── fishhunter_lock_setting_guide.md
    ├── fishhunter_room_add_guide.md
    └── odinInspector_OVDF_guide.md

.kiro/skills/
├── code-review-agent/          # Code Review 文件產出 Skill
│   └── SKILL.md
├── grill-me/                   # 計畫/設計壓力測試（推薦用於優化流程、設計決策前）
│   └── SKILL.md
├── skill-creator/              # Skill 建立/編輯/測試工具
│   └── SKILL.md
└── unity-cli/                  # Unity Editor 自動化 Skills（透過 unity-cli TCP 橋接）
    ├── unity-cli-usage/        ← 基礎：安裝、連線、指令路由
    ├── unity-scene-create/     ← 場景建立與初始化
    ├── unity-scene-inspect/    ← 場景檢視與分析
    ├── unity-gameobject-edit/  ← GameObject 與 Component 編輯
    ├── unity-prefab-workflow/  ← Prefab 建立/編輯/實例化
    ├── unity-asset-management/ ← 資源管理（AssetDatabase、材質、匯入設定）
    ├── unity-addressables/     ← Addressables 建置與管理
    ├── unity-csharp-navigate/  ← C# 程式碼搜尋與導覽（本地，不需 Unity 連線）
    ├── unity-csharp-edit/      ← C# 程式碼編輯與重構
    ├── unity-playmode-testing/ ← Play Mode 測試與截圖
    ├── unity-input-system/     ← Input System 管理與模擬輸入
    ├── unity-ui-automation/    ← UI 元素操作與自動化測試
    ├── unity-editor-tools/     ← Editor 狀態檢視、設定、Profiler
    └── unity-development-loop/ ← 開發迴圈工作流程
```

---

## Core 文件（自動載入）

這些文件會自動載入到每次對話中，提供基礎知識。

| 文件 | 用途 | 關鍵內容 |
|------|------|----------|
| `Auto_fishhunter_machine_expert.md` | OceanTreasure (2D) 開發規範 | 架構概要（Namespace/系統入口/事件系統）、MVC 分層、魚種分類與處理路徑、SkillSystem 三層架構（傳統 vs 通用模式）、生命週期摘要、OceanTreasure 開發規範、共用規範（UniTask、Action System、本家/他家、程式碼風格、AssetBundle 同名資源陷阱：同 bundle 內不同類型資源不可與 Prefab 同名） |
| `Auto_dinopinball_machine_expert.md` | DinoPinBall (3D) 開發規範 | 架構概要（Namespace/系統入口/事件系統/廳館型別 `GameLevel` enum/廳館模組化 `ITableModule`→`BaseTableModule`→各廳館 Module/Manager Init 統一簽名/using 別名慣例）、系統管理（泛型 GetSystem/GetGameData）、Dino_EventManager（兩套 EventBase 差異/正確 API/常見錯誤）、生命週期摘要、DinoPinBall 開發規範、**W2 報獎架構（新架構）**（**僅適用 W2 封包，SkillSystem 報獎不在此範圍**/HandleAward 虛擬方法/AwardHandleResult/4 種報獎模式：金幣飛出・翻倍盤・螢幕中間報獎・大報獎面板/半永生+捕獲模式範例/新怪物報獎接入規範/**禁止 HandleAward 覆寫怪物同時監聽 BLAST 事件（避免重複報獎）**/向下相容機制）、共用規範（UniTask、**CancellationTokenSource (CTS) 使用規範**（Object Pool 怪物 Init→End 循環/規則一覽：Init 先清後建・End 先清後 base・永遠用 `?.`・永遠設 null・Token 取一次・子 CTS 獨立管理・catch 必清理/標準模板/子 CTS 模板/常見錯誤）、Action System、本家/他家、程式碼風格、條件編譯差異：DEBUG_LOG 用於新增 Debug 工具腳本/DEBUG_MODE 用於測試版模式、砲台鎖定/解鎖標準做法：BOSS 覺醒演出用，直接呼叫 `WeaponManager.Instance.EnablePlayerShoot(bool, int seat)`，seat 參數鎖定時呼叫 `SkillSystem.StopAllSkill(seat)` 停止該玩家技能・解鎖時不需要，catch 區塊中也必須呼叫解鎖確保砲台不卡住） |
| `Auto_fishhunter_system_guide.md` | OceanTreasure (2D) 系統 API 指南 | GameClient 核心入口、具名 Getter API 一覽（getGameSystem / getPlayerSystem / getFishSystem / getWeaponSystem / getSkillSystem 等）、FishSystem & FishMaintainer（魚種生成/尋找）、FishHunterFakeServerPacket（假封包三步驟）、SkillSystem 詳細說明（指令註冊/發送請求/vs WeaponSystem 比較）、FishHunter_EventManager 事件系統（Registration/Cancellation/SendEvent/常用事件類型）。DinoPinBall (3D) API 請參閱 `Auto_dinopinball_system_guide.md` |
| `Auto_dinopinball_system_guide.md` | DinoPinBall (3D) 系統 API 指南 | GameClient 系統管理器（泛型 GetSystem\<T\> / GetGameData\<T\>）、已註冊系統與資料一覽、Dino_EventManager（兩套 EventBase 差異/正確 API/常見錯誤/事件資料存取）、MonsterSystem（怪物生成流程/MonsterData 屬性/Client 端模擬）、MonsterManager（怪物實體管理/GetMonsterBySid/GetMonstersByCode/重要屬性）、WeaponSystem（射擊 API/捕獲結果資料結構 DeadMonsterData/MonsterDieEventArgs）、PlayerSystem/PlayerInfo/PlayerData（玩家管理/屬性一覽/BetValue 注意事項）、BaseMonster 與繼承體系（繼承架構圖 PathAnimal/StaticAnimal/BaseMonster、選擇繼承基類判斷、Init 方法簽名與內部流程、重要屬性含存取層級、狀態判斷方法、生命週期方法、視覺效果方法、報獎相關方法、怪物狀態 EnumMonsterState、動畫狀態 MonsterAnimaitonState、PathAnimal 特殊行為、StaticAnimal 特殊行為、新增怪物控制器選擇指南）、**DinoPinBallM 場景入口**（廳館型別 `GameLevel` enum/Obsolete 屬性 isDino・isJuras 等禁止使用/廳館判斷寫法：直接 enum 比對/初始化流程：Awake 設定 Table → Start GameManager.Init → TableModuleRegistry.GetModule → tableModule.Init）、跨模組橋接 FishHunter_Agent（事件前綴/三種註冊類型）、開發建議 |
| `Auto_fishhunter_server_protocol.md` | Server 通訊協定 | **分離 OceanTreasure (海王) 與 DinoPinBall (恐龍) 兩套系統**、BaseSystem 機制差異、指令對照表、封包格式、資料流向圖、金流處理 |
| `product.md` | 產品概述 | 兩大遊戲引擎（OceanTreasure 2D / DinoPinBall 3D）架構特性、跨模組橋接、共用功能 |
| `spec_splitting_rule.md` | 大型 Spec 拆分規則 | 核心原則（tasks 超過 10 個或多子系統時暫停詢問使用者）、詢問時機（完成 requirements 和 design 之後、產出 tasks 之前）、拆分後的結構（requirements 和 design 集中在總索引 spec、子模組只需 tasks.md 並用相對路徑引用總設計文件）、拆分的判斷參考（UI/階段/系統模組適合拆、強依賴不適合拆） |
| `tech.md` | 技術棧 | Unity 版本、插件、條件編譯符號 |
| `structure.md` | 專案結構 | 目錄組織（含 DinoPinBallM 場景入口、TableManager 廳館資源管理、TableModules 廳館模組化體系、GameManager 透過 TableModuleRegistry 委派初始化、GameLogic_Lobby.GameLevel enum 定義所有廳館）、架構模式、命名慣例、Debug Log 規範（`#if DEBUG_LOG` 條件編譯包裹規則、DEBUG_LOG vs DEBUG_MODE 用途區分）、Debug 工具開發規範（KevinReflector 使用規範：僅限 `#if DEBUG_LOG` 區塊內使用反射呼叫 private 方法、Debug 工具透過 KevinReflector 呼叫控制器的 private 方法不需要暴露 public Debug 方法、保持控制器封裝性） |

---

## Manual 文件（手動觸發）

這些文件需要使用者明確引用（`#文件名`）或符合 fileMatch 條件時才會載入。

### 📁 dinopinball_cheattool_v2_guide.md

**觸發方式**: `inclusion: fileMatch`
**fileMatchPattern**: `['**/DinoPinBall/**/CheatTool/**/*.cs', '**/CrazyDino/**/CheatTool/**/*.uss']`

**用途**: DinoPinBall CheatTool V2 開發規範，編輯 DinoPinBall CheatTool 相關檔案時自動載入

**⚠️ 僅適用於 DinoPinBall（3D 恐龍）**，OceanTreasure（2D 海王）的作弊工具請參閱 `#newcheat-gui-guide`

**自動觸發條件**:
- 編輯 `DinoPinBall/**/CheatTool/` 目錄下的 `.cs` 檔案
- 編輯 `CrazyDino/**/CheatTool/` 目錄下的 `.uss` 檔案

**包含內容**:
- 兩套作弊工具對照表（DinoPinBall vs OceanTreasure）
- 架構總覽（Core/Modules/Plugins 三層結構）
- 核心介面（ICheatModule、IMonsterDebugPlugin）
- 模組職責分配（M00~M05 六個模組，含 Order 排序）
- CheatCmdSender 指令規範（新增指令規則、已有指令一覽）
- USS 樣式規範（共用 Class 體系、新增樣式規則、命名格式 `ct-{模組縮寫}-{元素}--{狀態}`）
- 怪物 Debug 插件規範（檔案位置、命名規則、必要結構、UI 建構規則、卡片大小）
- 條件編譯規範（DEBUG_LOG 用於新版、DEBUG_MODE 用於舊版）
- 接入點規範（`#if DEBUG_LOG` 包裹、完整 namespace 路徑、目前已接入的點一覽表：OnClientShoot/OnServerShootConfirm/UpdateServerCoin 在 WeaponGun 和 WeaponSystem 中的接入位置）
- 禁止事項（6 條禁令，含禁止混用海王 API）
- 架構模式可移植性（共用設計模式、移植到海王的對應關係、移植步驟、不應共用的部分）

---

### 📁 dinopinball_cheattool_v2_guide.md

**觸發方式**: `inclusion: fileMatch`
**fileMatchPattern**: `['**/DinoPinBall/**/CheatTool/**/*.cs', '**/CrazyDino/**/CheatTool/**/*.uss']`

**用途**: DinoPinBall (3D 恐龍魚機) CheatTool V2 專用開發規範，編輯 DinoPinBall CheatTool 相關檔案時自動載入。明確區分 DinoPinBall 與 OceanTreasure 兩套作弊工具。

**自動觸發條件**:
- 編輯 `DinoPinBall/**/CheatTool/` 目錄下的任何 `.cs` 檔案
- 編輯 `CrazyDino/**/CheatTool/` 目錄下的任何 `.uss` 檔案

**觸發關鍵字**:
- 「DinoPinBall CheatTool」
- 「恐龍作弊工具」
- 「DinoPinBall 作弊」

**包含內容**:
- 兩套作弊工具對照表（DinoPinBall CheatTool V2 vs OceanTreasure NewCheatGUI：程式碼位置/USS 位置/條件編譯/指令發送/怪物系統/事件系統/Namespace 差異）
- 架構總覽（Core/Modules/Plugins 三層結構，含完整檔案列表）
- 核心介面（ICheatModule、IMonsterDebugPlugin）
- 模組職責分配（M00~M05 六個模組，含 Order 排序）
- CheatCmdSender 指令規範（新增指令規則、已有指令一覽）
- USS 樣式規範（共用 Class 體系、新增樣式規則、命名格式 `ct-{模組縮寫}-{元素}--{狀態}`）
- 怪物 Debug 插件規範（檔案位置、命名規則、必要結構、UI 建構規則、卡片大小）
- 條件編譯規範（DEBUG_LOG 用於新版、DEBUG_MODE 用於舊版）
- 接入點規範（`#if DEBUG_LOG` 包裹、完整 namespace 路徑、目前已接入的點一覽表：OnClientShoot/OnServerShootConfirm/UpdateServerCoin 在 WeaponGun 和 WeaponSystem 中的接入位置）
- 禁止事項（6 條禁令，含禁止混用海王 API）
- **架構模式可移植性**（共用設計模式一覽：介面驅動/靜態註冊/自動發現/靜態指令發送/靜態 Minimize/USS class 體系/卡片式佈局/條件編譯隔離、移植到海王時的對應關係表：Namespace/介面/模組/指令發送/事件系統/怪物管理/目錄 對照、移植步驟 5 步、不應共用的部分：禁止共用 DLL/namespace/CheatToolPanel，可共用 USS 設計語言和 Skill 工作流程）

**與 cheattool_v2_guide.md 的關係**:
- `cheattool_v2_guide.md`：通用 CheatTool 規範（較寬的 fileMatch pattern）
- `dinopinball_cheattool_v2_guide.md`：DinoPinBall 專用，增加兩套工具對照表和禁止混用海王 API 的規則

---

### 📁 dinopinball_addmonster_expert.md

**觸發方式**: `inclusion: manual`

**用途**: DinoPinBall (3D 魚機) 新增怪物的完整開發流程

**觸發關鍵字**:
- 「新增怪物」
- 「加入新恐龍」
- 「建立 DinoPinBall 魚種」

**包含內容**:
- 怪物類型選項提示（第零步）：八大分類（A 一般怪物/B 翻倍怪物/C 技能怪物-獨立/D 技能怪物-通用/E BOSS-WeaponSystem/F 空殼模板/G BOSS-SkillSystem/H BOSS-特殊機制），含封包路徑、處理系統、捕獲封包、複雜度、參考範本
- 資源處理機制（Agent 自動執行）：建立空資料夾結構 + 僅複製參考怪物主 Prefab，SO 直接指向新怪物自己的資源（不借用參考怪物），新增怪物一律使用統一結構（不加廳館前綴），舊有怪物資料夾位置對照表僅供參考
- 怪物分類說明（一般怪物 / 翻倍怪物 / 技能怪物）
- 必要資訊收集（monsterName、serverIndex、monsterSource、monsterCategory 等）
- MonsterEnum.cs 列舉新增
- MonsterParameterDataSO 自動建立（Agent 直接生成 `.asset` + `.asset.meta` Unity YAML 檔案，含廳館路徑規則（含 Dino3 廳）、命名規則（含 Dino3 廳 `Dino3{PascalCaseName}.asset` 格式）、YAML 範本、winingOddsList 格式、maxSpeed/maxForce 參考怪物沿用規則、monsterSource 統一設為 All(0)、activeType 數值對照）
- MonsterManager「掃描怪物 SO 列表」按鈕（需在 Unity Editor 中點擊，自動收集 MonsterParameter 資料夾下所有 SO）
- Dino_EventType 事件類型新增（BOSS 進度條/特殊演出）
- 資源資料夾建立（Prefab 資源資料夾 + 複製參考怪物主 Prefab + 腳本資料夾 + GUID 生成 + ⚠️ 資料夾已存在時必須檢查 AssetBundle 設定：讀取 `.meta` 確認 `assetBundleName` 為 `dinopinball/{assetbundlename小寫}`，若為空或不正確必須主動修正）
- 怪物控制器建立（BaseMonster 繼承，選項 A 會提示使用者是否需要，選項 B/C/D/E/G/H 主動詢問）
- unity-cli Prefab Component 檢查與補齊（5.4 必要流程）：當 Prefab 已存在時，Agent 必須使用 unity-cli 檢查並補齊缺少的 Component 和子物件（open_prefab → get_hierarchy → get_component_values → 比對標準 → 補齊 → save_prefab），含檢查清單與自動修正對照表（Root Tag/Layer、ArkSteerBehavior、SteerForTether、PathAnimal、PathController、Animator、Colliders、Effects）、unity-cli 標準操作順序、params-file UTF-8 無 BOM 注意事項、完成後提醒使用者手動設定 SerializeField 引用
- 生成方式（NormalSpawn / GolemSpawn / StaticSpawn）
- Debug 工具建立（第七步，依怪物分類區分）：
  - 7.1 CheatToolManager 自動列出（所有怪物皆適用）
  - 7.2 Debug 腳本建立規則（Agent 詢問使用者是否需要，選項 A 和 F 不需要）
  - 7.2.2 W2 擊殺封包測試（直接建構 JSON 封包發送給 WeaponSystem.OnMessage）
  - 7.2.3 F7 倍率更新測試（翻倍怪物專用，直接建構 JSON 封包發送給 MonsterSystem.OnMessage）
  - 7.2.4 技能測試（技能怪物專用，直接建構 sk_xxx JSON 封包發送給 SkillSystem.OnMessage）
  - 7.3 翻倍怪物 Debug 範本（選項 B）：Odin Inspector 資料類別（W2Data/F7Data）、自動尋找場上怪物（GetMonstersByCode）、本家/他家判定、假召喚 + W2 擊殺封包（含 base_odds/multiply/show_type）+ F7 倍率更新
  - 7.4 一般怪物 Debug 範本（選項 A）：不需要 Debug 腳本，CheatToolManager 自動列出
  - 7.5 技能怪物 Debug 範本（選項 C/D）：Odin Inspector SkillData 資料類別、自動尋找場上怪物、技能觸發按鈕（建構 sk_xxx 封包發送給 SkillSystem.OnMessage）
  - 7.5 BOSS Debug 範本（選項 E/G/H）：DebugSystem + CheatToolManager 專屬測試工具
  - 7.6 假生成封包格式（DebugManager.createJson 參考）
  - 7.7 路徑代碼對應（dino/stoneage/robin → PATH_DOINPINBALL, juras → PATH_JURAS）
- 特殊怪物邏輯處理：
  - 8.1 翻倍怪物 f7 封包（選項 B）
  - 8.2 技能怪物（獨立，選項 C）：SkillSystem 整合、ESkillType 新增、多技能模式（參考機甲哥布林）
  - 8.3 技能怪物（通用，選項 D）：SkillCommonObject 標準化流程（sk_skill_start → sk_bomb_fish → sk_end）、fishSkillDict 對應、SkillCommonModel 實作
  - 8.4 BOSS 怪物（選項 E/G）：E 走 WeaponSystem（參考霸王龍）、G 走 SkillSystem（參考阿力肯，含 EquipmentSystem 砲台養成整合）
  - 8.5 BOSS 特殊機制（選項 H）：參考史萊姆王（f8 封包計量條、分裂機制 DINO_SPLIT_SLIME、BossProgressBar 事件）
- Prefab 結構驗證（⚠️ 必須引用 `#dinopinball_monseter_prefabs_standard` 進行檢查）：Root Tag/Layer、必要 Component 順序、Colliders Tag 與 isTrigger、SerializeField 引用正確性、Effects 子物件存在性
- **⚠️ Agent 完成 Prefab 設定後的手動操作提醒**：Agent 自動化完成後，必須提醒使用者在 Unity Editor Inspector 中手動完成以下項目：PathAnimal/BaseMonster SerializeField 引用拖拉設定（m_shape/m_monsterAnimator/m_runAnimator/monsterController/m_lightingMaterial）、SphereCollider radius 調整、Animator Controller 指定
- DinoPinBall 專用怪物開發流程（十步驟，含第零步類型選擇）

**怪物分類對照表**:

| 選項 | 分類 | 封包路徑 | 複雜度 | 參考範本 |
|------|------|----------|--------|----------|
| A | 一般怪物 | W2 | ⭐ | Robin_Goblin（哥布林） |
| B | 翻倍怪物 | W2 + F7 | ⭐⭐ | Juras_GoldenMammoth（黃金猛瑪象） |
| C | 技能怪物（獨立） | SkillSystem (自訂 sk_xxx) | ⭐⭐⭐ | Juras_MechanicGoblin（機甲哥布林） |
| D | 技能怪物（通用） | SkillSystem (sk_skill_start) | ⭐⭐⭐ | SkillCommonObject 通用模組 |
| E | BOSS（走 WeaponSystem） | W2 | ⭐⭐⭐ | Stoneage_Tyrannosaurus（霸王龍） |
| F | 空殼模板 | 未定 | ⭐ | 僅建立基礎骨架（繼承 PathAnimal + 覆寫 Init） |
| G | BOSS（走 SkillSystem） | SkillSystem (自訂 sk_xxx) | ⭐⭐⭐⭐ | Stoneage_BossAriken（阿力肯） |
| H | BOSS（特殊機制） | W2 + F8 | ⭐⭐⭐⭐ | Dino_SlimeKing（史萊姆王） |

---

### 📁 dinopinball_monseter_prefabs_standard.md

**觸發方式**: `inclusion: manual`

**用途**: DinoPinBall (3D 恐龍魚機) 怪物 Prefab 結構規範與標準。僅適用於恐龍系列，海王系列請參閱 `fishhunter_fish_prefabs_standard.md`。

**觸發關鍵字**:
- 「恐龍 Prefab 規範」
- 「恐龍 Prefab 標準」
- 「恐龍怪物 Prefab 結構」
- 「DinoPinBall Prefab 規範」
- 「檢查恐龍 Prefab」
- 「驗證恐龍怪物結構」
- 「製作恐龍 Prefab」
- 「設定恐龍物件」
- 「monster prefab standard」

**包含內容**:
- 基於實際怪物 Prefab 分析歸納（一般怪物 Stoneage_Dilophosaurus、翻倍怪物 Juras_Golden_Mammoth、BOSS Stoneage_Tyrannosaurus）
- Prefab Hierarchy 標準結構（Root → Shape/Shadow/Colliders/Effects/[UI]）
- Root 必須設定：Tag=Monster、Layer=FISH_ZORDER
- Shape/ 模型容器（Animator + 骨骼 + SkinnedMeshRenderer）
- Shadow/ 影子容器（SpriteRenderer）
- Colliders/ 碰撞器容器（Tag 必須設為 `Monster`，SphereCollider 或 BoxCollider 且 isTrigger=true，BOSS 可多個）
- Effects/ 特效容器（Freeze 冰凍效果 inactive + ThunderTwoTarget 雷鳴目標點 inactive）
- UI/ 選用（翻倍怪物倍率 UI、BOSS 血條/進度條）
- Hierarchy 檢查清單（含 unity-cli 檢查方式）
- 怪物類型與 Hierarchy 差異對照表（一般/翻倍/BOSS/技能怪物）
- Root 物件必要 Component 組成（6 個固定順序 Component：Transform → ArkSteerBehavior → SteerForTether → 怪物控制器腳本 → PathController → Animator（選用，部分怪物動畫由 Shape 子物件處理））
- 怪物控制器腳本選擇（PathAnimal/自訂 Ctrl/StaticAnimal，依怪物類型對照表）
- SerializeField 欄位規範（unity-cli `get_component_values` 中 `[SF]` 前綴標記）：
  - 物件引用欄位（m_shape/m_monsterAnimator/m_runAnimator/monsterController/m_lightingMaterial，皆為必要）
  - 數值參數欄位（m_crossSize/m_elecWaveSize/hit_ShakePower/m_deadDespawnDelayTime 等，含預設值）
  - 布林旗標欄位（m_IsFeatureMonster/m_IsEternalLife/m_IsImmuneToSpecialWeapon 等，含何時設為 true）
  - 報獎類型欄位（m_specialAwardType，含常用 SpecialAwardType 值對照）
  - 可選引用欄位（m_alphaMaterial/m_material/m_onHitTexture 等，部分怪物可為 null）
  - Runtime 屬性（Sid/MonsterType/State/Lockable 等，不需 Inspector 設定，由程式碼控制）
- 資源資料夾結構規範：
  - 標準資料夾結構（Animation/Animator/Material/Model/Texture + 選用 Audio/Atlas/Prefab/Timeline）
  - Prefab 命名規則（模型 Prefab `{ModelName}.prefab` + 怪物主 Prefab `{廳館}_{MonsterName}.prefab`）
  - 廳館前綴對照（Dino_/Juras_/Stoneage_/Robin_，新增怪物資料夾不加廳館前綴）
  - Lightning 材質必要性（`*_Lightning.mat`，`m_lightingMaterial` 欄位引用，閃電效果必備）
- unity-cli 操作標準流程：
  - 檢查既有怪物 Prefab（open_prefab → get_gameobject_details → get_component_values → exit_prefab_mode）
  - 製作新怪物 Prefab（open_prefab → 確認 Hierarchy → set_component_field 設定引用/旗標/報獎類型 → save_prefab）
  - **unity-cli 指令執行規範（⚠️ 強制）**：所有 unity-cli 指令必須使用單行 `;` 串接格式（寫檔 + 執行合併在同一個 shell 呼叫），禁止分行寫法；UTF-8 無 BOM 編碼（`[System.Text.UTF8Encoding]::new($false)`）；禁止 Out-File/Set-Content；暫存檔統一使用 `params.json`，完成後清理

---

### 📁 fishhunter_fish_prefabs_standard.md

**觸發方式**: `inclusion: manual`

**用途**: OceanTreasure (2D 海王魚機) 魚種 Prefab 標準架構規範。僅適用於海王系列，恐龍系列請參閱 `dinopinball_monseter_prefabs_standard.md`。

**觸發關鍵字**:
- 「海王 Prefab 規範」
- 「海王 Prefab 標準」
- 「魚種 Prefab 結構」
- 「OceanTreasure Prefab 規範」
- 「檢查海王 Prefab」
- 「驗證魚種結構」
- 「製作魚種 Prefab」
- 「設定魚種物件」
- 「fish prefab standard」

**包含內容**:
- **⚠️ 標準架構強制規則（Agent 必讀）**：所有新增的海王魚種 Prefab 一律使用本文件定義的標準架構，無例外。Agent 執行規則（不需判斷架構、主動建議標準架構、除非使用者明確表示不需要）、Agent 建議話術範例、適用範圍對照表（新增魚種一律套用/檢查既有魚種僅報告差異不主動重構/舊有魚種不主動重構）
- 基於實際魚種 Prefab 分析歸納（一般魚種 GoldenBat、DoubleFish KillerWhale）
- 與恐龍系列的關鍵差異（2D vs 3D、Fish2D vs BaseMonster、渲染方式、碰撞系統）
- Prefab Hierarchy 標準結構（Root → {FishName}Shape → freeze/shadow/maskroot/collider/Paralysis）
- main 節點兩種渲染方式（Sprite 動畫 vs Spine 骨骼動畫，含 Component 對照與範例魚種）
- Root 必須設定：Tag=Fish、Layer=FISH_ZORDER
- Hierarchy 檢查清單（必要檢查項目，Shape 子物件命名不限，含 unity-cli 檢查方式）
- **Fish2D SerializeField 引用檢查（重點）**：Shape 命名不重要，重點是 Fish2D 子類的 `[SF]` 引用是否正確（`[SF]fishSpine` 指向 Spine 顯示 Root 的 SkeletonAnimation、`[SF]m_ColliderHandler` 指向 Shape 上的 FishColliderHandler、其他子類自訂欄位）、檢查原則（Spine 魚種/Sprite 魚種引用目標、所有 `[SF]` 欄位不應為 null、error 訊息代表未拉引用）
- Root 物件必要 Component 組成（Transform, Fish2D, FishRigidBody2D, SteerForOceanKing2D, PathBehavior）
- 依魚種類型額外 Component（DoubleFish: FishDialogue + Thunder2BonusChoose）
- Shape 子物件 Handler Component（SpriteLayerHandler, FishAnimatorHandler, FishMaskStatusHandler, FishColliderHandler）
- Fish2D 關鍵屬性（類型/狀態/物理屬性）
- 魚種類型與 Prefab 差異對照表（A~F 六種分類）
- 資源資料夾結構規範（標準結構、命名規則、AssetBundle 同名資源陷阱）
- unity-cli 操作標準流程（檢查/辨識渲染方式/params-file 注意事項）
- 驗證檢查清單（結構驗證、渲染方式驗證、DoubleFish 額外驗證）
- 海王 vs 恐龍系列完整對照表

---

### 📁 fishhunter_lock_setting_guide.md

**觸發方式**: `inclusion: manual`

**用途**: 智能鎖定面板新增魚種指南

**觸發關鍵字**:
- 「智能鎖定」
- 「鎖定面板」
- 「新增鎖定魚種」
- 「LockSetting」

**包含內容**:
- 系統概述（FH_LockSettingPanel / FishHunterLockSettingDataSO / FH_FishItemCtrl 三層組件）
- 運作流程（Init → 遍歷 SO → 實例化 UI）
- 新增魚種步驟

---

### 📁 fishhunter_room_add_guide.md

**觸發方式**: `inclusion: manual`

**用途**: 新增魚機廳館的完整 Checklist

**觸發關鍵字**:
- 「新增 XXX 館」
- 「加入新廳館」
- 「建立 room」

**包含內容**:
- RoomType / GameLevel 枚舉新增
- VipTypeRoomList 設定
- 等級限制欄位
- 進入邏輯 (JoinXXX 方法)
- 外部跳轉支援
- Room Prefab 複製與 GUID 生成（分步驟執行，需等待使用者刷新 Unity Editor）
- Canvas_FishArea.prefab roomUnits 更新
- 測試資料注入（Server 未實作前）

---

### 📁 fishhunter_addfish_expert.md

**觸發方式**: `inclusion: manual`

**用途**: 新增魚種的完整開發流程（十步驟）

**觸發關鍵字**:
- 「新增魚種」
- 「加入新魚」
- 「建立魚種」

**包含內容**:
- 魚種類型選項提示（第零步）：六大分類（A 一般魚種/B DoubleFish-翻倍魚/C 特殊狀態魚-f7/D 技能魚種-傳統/E 技能魚種-通用/F 空殼模板），含封包路徑、處理系統、複雜度、範例，Agent 依分類決定修改範圍、Debug 範本、FishRules 註冊需求
- 魚種分類說明（一般魚種 / DoubleFish / 特殊狀態魚 / 技能魚種）
- Server ID 對應規則與跳號處理（使用 `enumType_Reserved{數字}` 佔位）
- enumFishType / FishType[] / FishKind[] 陣列新增（Fish2DEnum.cs、Fish2D.cs）
- FishRules.cs 完整註冊指南（第四步）：
  - 4.1 必須註冊的方法（所有特殊魚種）：IsSpecialHide（武器卡/烈焰風暴/鑽頭/電磁蟹/機械霸王蟹透明）、IsAssignationHide（召喚卡排除）、FishComingStay（不被魚潮沖走）、IsCustomCaptureFish（自訂捕獲演出）
  - 4.2 依魚種特性選擇註冊：IsDoubleFish（翻倍魚）、IsEternalFish（永生魚）、IsHaveMinBet（最小押注）、IsHaveMinVIP（最小 VIP）
  - 4.3 電擊/雷鳴排除清單：電擊排除清單、ThunderTwoExceptBonusFish
  - 4.4 其他清單：特殊魚總清單（檔案末尾）
  - 4.5 DoubleFish 專用：IsDoubleFish() 註冊範例
  - 4.6 快速判斷表：依魚種分類（A~E + BOSS）對照需要註冊的方法（✅/❌ 矩陣）
- FishHunter_EventType 事件類型新增
- FishHunterCheatTool 測試資料新增
- 資源資料夾建立（腳本資料夾 + AssetBundle 設定）
- Debug 工具建立：
  - 使用 `FishMaintainer.Instance?.GetFishByType()` 尋找魚種
  - 使用 `FishHunterFakeServerPacket` 三步驟假召喚
  - W2 擊殺封包 / F7 狀態更新（DoubleFish）
- 一般魚種 / DoubleFish Debug 範本
- 魚種控制器與演出控制器範本

**魚種分類對照表**:

| 選項 | 類型 | 處理系統 | 捕獲封包 | FishRules 註冊 | 範例 |
|------|-----|---------|---------|---------------|------|
| A | 一般魚種 | WeaponSystem | W2 | 不需要 | 迦納魚、金蝙蝠 |
| B | DoubleFish | WeaponSystem | W2（翻倍資訊在 extra_dict） | `IsDoubleFish()` | 殺人鯨、聚寶盆、玉如意、月兔、吸血鬼公爵 |
| C | 特殊狀態魚（f7） | WeaponSystem + FishSystem | W2 + F7 | 不需要 | 猴爺、虎爺、武士魚、小丑魚、彌勒佛、龍舞極、酒霸狂鯊、貓魚、泡泡魚、聖誕老人 |
| D | 技能魚種（傳統） | SkillSystem | 自訂指令 (sk_xxx) | 視情況 | 電磁蟹、鑽頭蟹、烈焰風暴、轉輪蟹、招財貓、黑龍 |
| E | 技能魚種（通用） | SkillCommonObject | sk_skill_start | 視情況 | 邱比特2024、朱雀、朱雀覺醒、健美兔 |
| F | 空殼模板 | 未定 | 未定 | 不需要 | 僅建立基礎骨架（繼承 Fish2D + 覆寫 Init） |

---

### 📁 fishhunter_cheat_tool_guide.md

**觸發方式**: `inclusion: manual`

**用途**: 作弊測試系統說明

**觸發關鍵字**:
- 「作弊工具」
- 「CheatTool」
- 「測試工具」

**包含內容**:
- FishHunterCheatTool 功能
- 假召魚 / 真召魚
- 武器切換
- 魚種過濾
- DEBUG_MODE 條件

---

### 📁 fishhunter_fake_summon_guide.md

**觸發方式**: `inclusion: manual`

**用途**: 假召魚機制說明

**觸發關鍵字**:
- 「假召魚」
- 「FakeSummon」
- 「PlayBonus」

**包含內容**:
- FishMaintainer.SummonFakeFish() 入口
- fakeFishList 管理
- UI Layer 生成
- 負數 SID 機制

---

### 📁 fishhunter_code_templates.md

**觸發方式**: `inclusion: manual`

**用途**: 完整程式碼範本集合

**觸發關鍵字**:
- 「程式碼範本」
- 「code template」
- 「範本」

**包含內容**:
- 一般魚種控制器範本 (NormalFishCtrl : Fish2D)
- 技能魚種控制器範本 (SkillFishCtrl : Fish2D)
- 傳統技能物件範本 (SkillXxxObject，以電磁蟹為例)
- 技能演出實體範本 (SkillModel)
- 通用技能模組範本 (SkillCommonObject)
- 通用技能演出實體範本 (SkillCommonModel)
- 演出控制器範本 (FeatureCtrl，UniTask 非同步流程)

---

### 📁 odinInspector_OVDF_guide.md

**觸發方式**: `inclusion: manual`

**用途**: Odin Validator Diff Format (OVDF) 套用指南

**觸發關鍵字**:
- 「OVDF」
- 「Odin Validator Diff」
- 「套用 OVDF」
- 「Odin Attribute 變更」

**包含內容**:
- OVDF 格式規範（Header、差異描述）
- 解讀 OVDF 並自動套用到 C# 原始碼
- Odin Inspector Attribute 變更處理

---

## Skills（.kiro/skills/）

Skills 提供特定領域的工作流程指南，Agent 可依需求讀取對應的 SKILL.md 和 references。

### code-review-agent

**用途**：根據規格實作內容產出完整且標準的 Code Review HTML 文檔

### grill-me

**用途**：計畫/設計壓力測試，針對方案或流程進行深度拷問，逐一解決決策樹的每個分支，直到達成共識
**推薦使用時機**：
- 優化現有開發流程（如新增怪物流程、新增魚種流程）
- 設計新功能架構前的壓力測試
- 評估技術方案的取捨
**觸發方式**：使用者提到「grill me」、「壓力測試計畫」、「拷問設計」

### skill-creator

**用途**：建立新 Skill、修改和改善現有 Skill、測試 Skill 效能

### unity-cli（Unity Editor 自動化）

**前置條件**：需安裝 unity-cli 二進位檔 + Unity Editor 安裝 unity-cli-bridge 套件
**連線方式**：透過 TCP 連接 Unity Editor（預設 localhost:6400）
**CLI 路徑**：`$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe`
**參數傳遞**：使用 `--params-file` 傳 JSON 檔案（PowerShell 的 JSON 跳脫問題）

| Skill | 用途 | 常用 Tools |
|-------|------|-----------|
| unity-cli-usage | 安裝、連線、指令路由 | `system ping`、`instances list` |
| unity-scene-create | 場景建立與初始化 | `create_scene`、`create_gameobject`、`add_component`、`save_scene` |
| unity-scene-inspect | 場景檢視與分析 | `get_hierarchy`、`get_scene_info`、`analyze_scene_contents` |
| unity-gameobject-edit | GameObject/Component 編輯 | `modify_gameobject`、`modify_component`、`set_component_field` |
| unity-prefab-workflow | Prefab 建立/編輯/實例化 | `create_prefab`、`open_prefab`、`save_prefab`、`instantiate_prefab` |
| unity-asset-management | 資源管理 | `manage_asset_database`、`create_material`、`refresh_assets` |
| unity-addressables | Addressables 建置 | `addressables_build`、`addressables_manage` |
| unity-csharp-navigate | C# 搜尋與導覽（本地） | `read`、`search`、`find_symbol`、`find_refs` |
| unity-csharp-edit | C# 編輯與重構 | `apply_csharp_edits`、`rename_symbol`、`create_csharp_file` |
| unity-playmode-testing | Play Mode 測試 | `play_game`、`stop_game`、`run_tests`、`capture_screenshot` |
| unity-input-system | Input System 管理 | `create_action_map`、`input_keyboard`、`input_mouse` |
| unity-ui-automation | UI 自動化測試 | `find_ui_elements`、`click_ui_element`、`set_ui_element_value` |
| unity-editor-tools | Editor 狀態/設定 | `get_editor_state`、`read_console`、`update_project_settings` |
| unity-development-loop | 開發迴圈工作流程 | 綜合多個 tools 的工作流程指南 |

**Object Reference 設定方式**：
```bash
# 設定 Prefab 引用到 SerializeField 欄位
unity-cli tool call set_component_field --params-file params.json
# params.json: {"gameObjectPath":"/物件","componentType":"腳本名","fieldPath":"欄位名","value":"Assets/路徑/Prefab.prefab","valueType":"objectReference"}
```

---

## 使用指南

### Agent 自動判斷

當使用者提出需求時，Agent 應根據關鍵字判斷是否需要引用 manual 文件：

| 使用者說 | 應引用的文件 |
|----------|--------------|
| 「新增怪物」「加入新恐龍」 | `#dinopinball_addmonster_expert` |
| 「檢查恐龍 Prefab」「恐龍 Prefab 規範」「DinoPinBall Prefab 規範」 | `#dinopinball_monseter_prefabs_standard` |
| 「檢查海王 Prefab」「魚種 Prefab 規範」「OceanTreasure Prefab 規範」 | `#fishhunter_fish_prefabs_standard` |
| 「DinoPinBall CheatTool」「恐龍作弊工具」 | `#dinopinball_cheattool_v2_guide` |
| 「新增殭屍館」 | `#fishhunter_room_add_guide` |
| 「我要加入新魚種」 | `#fishhunter_addfish_expert` |
| 「怎麼用作弊工具測試」 | `#fishhunter_cheat_tool_guide` |
| 「假召魚怎麼運作」 | `#fishhunter_fake_summon_guide` |
| 「給我程式碼範本」 | `#fishhunter_code_templates` |
| 「套用 OVDF」「Odin Validator Diff」 | `#odinInspector_OVDF_guide` |
| 「智能鎖定」「鎖定面板」「新增鎖定魚種」 | `#fishhunter_lock_setting_guide` |
| 「grill me」「壓力測試計畫」「拷問設計」「優化流程」 | 啟用 `grill-me` skill |

### 使用者手動引用

使用者可以在對話中使用 `#` 符號引用特定文件：

```
#fishhunter_room_add_guide 新增一個叫做 gundam 的廳館
```

---

## 文件維護規範

### 命名慣例

| 類型 | 前綴 | 範例 |
|------|------|------|
| 自動載入 | `Auto_` | `Auto_fishhunter_machine_expert.md` |
| 手動觸發 | `fishhunter_xxx_guide.md` | `fishhunter_room_add_guide.md` |

### Front Matter 格式

**自動載入**:
```yaml
---
inclusion: always
---
```

**手動觸發**:
```yaml
---
inclusion: manual
---
```

**檔案匹配觸發**:
```yaml
---
inclusion: fileMatch
fileMatchPattern: ['**/*.cs']
---
```

### 新增文件流程

1. 決定文件類型（core / manual）
2. 選擇適當的 inclusion 模式
3. 放入對應資料夾
4. 更新本索引文件

---

## 快速參考卡

```
┌─────────────────────────────────────────────────────────────┐
│                    Steering 文件快速參考                      │
├─────────────────────────────────────────────────────────────┤
│ 🔄 自動載入 (core/)                                          │
│   • Auto_fishhunter_machine_expert.md  - OceanTreasure (2D) 開發規範 │
│   • Auto_dinopinball_machine_expert.md - DinoPinBall (3D) 開發規範   │
│   • Auto_fishhunter_system_guide.md    - OceanTreasure (2D) 系統 API │
│   • Auto_dinopinball_system_guide.md   - DinoPinBall (3D) 系統 API  │
│   • Auto_fishhunter_server_protocol.md - Server 通訊協定            │
│   • spec_splitting_rule.md             - 大型 Spec 拆分規則        │
│   • product.md / tech.md / structure.md - 專案基礎          │
├─────────────────────────────────────────────────────────────┤
│ 📌 手動觸發 (manual/)                                        │
│   • #cheattool_v2_guide (fileMatch) → CheatTool V2 開發規範  │
│   • #dinopinball_cheattool_v2_guide (fileMatch) → DinoPinBall CheatTool V2 專用│
│   • #dinopinball_addmonster_expert → 新增怪物 (3D)           │
│   • #dinopinball_monseter_prefabs_standard → 恐龍怪物 Prefab 規範│
│   • #fishhunter_fish_prefabs_standard → 海王魚種 Prefab 規範 │
│   • #fishhunter_lock_setting_guide → 智能鎖定面板新增魚種    │
│   • #fishhunter_room_add_guide   → 新增廳館                 │
│   • #fishhunter_addfish_expert   → 新增魚種 (2D)            │
│   • #fishhunter_cheat_tool_guide → 作弊工具                 │
│   • #fishhunter_fake_summon_guide → 假召魚                  │
│   • #fishhunter_code_templates   → 程式碼範本               │
│   • #odinInspector_OVDF_guide    → OVDF 套用指南            │
├─────────────────────────────────────────────────────────────┤
│ 🎮 Skills (.kiro/skills/)                                    │
│   • code-review-agent            → Code Review 文件產出      │
│   • grill-me                     → 計畫/設計壓力測試          │
│   • skill-creator                → Skill 建立/編輯/測試      │
│   • unity-cli/                   → Unity Editor 自動化       │
│     - unity-cli-usage            → 安裝、連線、指令路由      │
│     - unity-scene-create         → 場景建立與初始化          │
│     - unity-scene-inspect        → 場景檢視與分析            │
│     - unity-gameobject-edit      → GameObject/Component 編輯 │
│     - unity-prefab-workflow      → Prefab 建立/編輯/實例化   │
│     - unity-asset-management     → 資源管理                  │
│     - unity-addressables         → Addressables 建置         │
│     - unity-csharp-navigate      → C# 搜尋與導覽（本地）     │
│     - unity-csharp-edit          → C# 編輯與重構             │
│     - unity-playmode-testing     → Play Mode 測試與截圖      │
│     - unity-input-system         → Input System 管理         │
│     - unity-ui-automation        → UI 自動化測試             │
│     - unity-editor-tools         → Editor 狀態/設定/Profiler │
│     - unity-development-loop     → 開發迴圈工作流程          │
└─────────────────────────────────────────────────────────────┘
```
