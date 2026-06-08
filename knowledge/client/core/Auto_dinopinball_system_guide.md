# DinoPinBall (3D 魚機) 系統 API 指南

本指南說明 DinoPinBall / DinoGame (3D 魚機) 的核心系統 API。
OceanTreasure (2D 魚機) 請參閱 `Auto_fishhunter_system_guide.md`。

---

## 1. GameClient (系統管理器)

- **Namespace**: `DinoPinBall`
- **如何取得**: `DinoPinBall.GameClient.Instance`（延遲初始化）
- **BaseSystem 建構子**: `base(gameDataManager, eventManager, cmdSender, "name")`
- **OnMessage 方法**: `OnMessage(JSON json)`（大寫 O）
- **實作介面**: `IGameClient`, `ISystemManager`, `IGameDataManager`, `IEventManager`

### 系統管理 API

| 方法 | 說明 |
|------|------|
| `GetSystem<T>()` | 取得指定系統（泛型） |
| `AddSystem<T>()` | 新增系統 |
| `RemoveSystem<T>()` | 移除系統 |

### 資料管理 API

| 方法 | 說明 |
|------|------|
| `GetGameData<T>()` | 取得資料（如 `GetGameData<PlayerInfo>()`） |
| `AddGameData<T>()` | 新增資料 |
| `RemoveGameData<T>()` | 移除資料 |

### 便捷方法（相容舊版）
- `getGameSystem()`: 等同 `GetSystem<GameSystem>()`
- `getSkillSystem()`: 等同 `GetSystem<SkillSystem>()`
- `getGameInfo()`: 取得 `GameInfo`

### 已註冊的系統
`WeaponSystem`, `PlayerSystem`, `GameSystem`, `MonsterSystem`, `SkillSystem`, `TableSystem`, `DP_JackpotSystem`, `DP_DailyRankSystem`, `SocialSystem`, `MissionSystem`, `EquipmentSystem`
（`DEBUG_MODE` 下額外加入 `DebugSystem`）

### 已註冊的資料
`GameInfo`, `PlayerInfo`

### ⚠️ IEventManager 介面
`GameClient` 實作了 `IEventManager`，但 `SendEvent`/`Registration`/`Cancellation` 皆拋出 `NotImplementedException`。
實際使用 `Dino_EventManager`（靜態方法）。

---

## 2. Dino_EventManager (事件中心)

MonoBehaviour，提供靜態方法。

### ⚠️ 兩套 EventBase 類別

| 類別 | 位置 | 用途 |
|------|------|------|
| `Dino_EventManager.EventBase` | 巢狀於 `Dino_EventManager` 內 | `Dino_EventManager` 靜態方法使用 |
| `DinoPinBall.EventBase` | namespace 層級 `EventManager.cs` | `IEventManager` 介面使用（未實作） |

### 正確 API

```csharp
// ✅ 註冊
Dino_EventManager.Registration(Enum e, Action<Dino_EventManager.EventBase> action);

// ✅ 取消
Dino_EventManager.Cancellation(Enum e, Action<Dino_EventManager.EventBase> action);

// ✅ 發送（帶參數）
Dino_EventManager.SendEvent(Enum eid, params object[] args);

// ✅ 發送（直接傳 EventBase）
Dino_EventManager.SendEventByEventBase(Enum eid, Dino_EventManager.EventBase eventBase);
```

### ❌ 常見錯誤

```csharp
Dino_EventManager.Register(...);    // ❌ 不存在
Dino_EventManager.Unregister(...);  // ❌ 不存在
Action<DinoEventData> callback;     // ❌ 需完整路徑
// ✅ 正確：Action<Dino_EventManager.EventBase> callback;
```

### 事件資料存取

```csharp
void OnMonsterActive(Dino_EventManager.EventBase e) {
    var data = (e as Dino_EventManager.DinoEventData);
    if (data != null) {
        var monsterData = (MonsterData)data.args[0];
    }
}

Dino_EventManager.Registration(Dino_EventType.MonsterSystemEvent.MONSTER_ACTIVE, OnMonsterActive);
Dino_EventManager.Cancellation(Dino_EventType.MonsterSystemEvent.MONSTER_ACTIVE, OnMonsterActive);
```

---

## 3. MonsterSystem (怪物管理)

對應 ArkGame 的 `FishSystem`，系統名稱為 `"fish"`，管理怪物生成與同步。

### 生成流程
1. `MonsterSystem` 接收 Server `f1` / `f4` 指令
2. 解析資料為 `MonsterData`
3. 發送 `Dino_EventType.MonsterSystemEvent.MONSTER_ACTIVE` 事件
4. `MonsterManager` 監聽並實例化怪物

### MonsterData 屬性（注意大寫開頭）

| 屬性 | 類型 | 說明 |
|------|------|------|
| `ID` | `int` | 怪物唯一 SID |
| `MonsterType` | `int` | 怪物種類代號 |
| `X` / `Y` / `Z` | `float` | 座標 |
| `Orientation` | `float` | 朝向角度 |
| `MonsterSize` | `int` | 生成數量 |
| `Spacing` | `float` | 間距 |
| `Path` | `string` | 路徑名稱 |
| `SyncTime` | `float` | 同步時間 |
| `IsSummon` | `bool` | 是否為召喚怪 |
| `Extra_data` | `JSON` | 額外資料 |
| `FakeCmd` | `int` | 假封包標記 |
| `Sub_Type` | `int` | 子類型 |
| `Limit_Vip` | `int` | VIP 限制 |

### Client 端模擬生成
```csharp
MonsterData data = new MonsterData();
data.ID = 1001;
data.MonsterType = 9112;
data.Path = "LARGE_1";
Dino_EventManager.SendEvent(Dino_EventType.MonsterSystemEvent.MONSTER_ACTIVE, data);
```

---

## 4. MonsterManager (怪物實體管理)

- **如何取得**: `MonsterManager.Instance`（`SingletonService` 單例）
- **對應 ArkGame**: `FishMaintainer`

### 主要 API

| 方法 | 回傳類型 | 說明 |
|------|----------|------|
| `GetMonsterBySid(int sid)` | `BaseMonster` | 依 SID 尋找場上怪物 |
| `GetMonstersBySource(MonsterSource)` | `List<BaseMonster>` | 依來源取得怪物列表 |
| `GetMonstersByCode(string serverName)` | `List<BaseMonster>` | 依 Server 代號取得怪物列表 |
| `KillMonster(int sid, ...)` | `void` | 擊殺怪物 |

### 重要屬性

| 屬性 | 類型 | 說明 |
|------|------|------|
| `MonsterDictinary` | `Dictionary<int, BaseMonster>` | SID → 怪物實體字典 |
| `MonsterParameterDataDict` | `Dictionary<int, MonsterParameterData>` | Type → 怪物參數字典 |
| `LockableList` | `List<BaseMonster>` | 可鎖定怪物列表 |

---

## 5. WeaponSystem (武器系統)

- **系統名稱**: `"weapon"`
- **建構子**: `WeaponSystem(GameClient gameClient) : base(gameClient, gameClient, gameClient, "weapon")`

### 主要射擊 API

| 方法 | 說明 |
|------|------|
| `Shoot(seat, x, y, id, bulletType, ...)` | 一般射擊 (w1) |
| `LockShoot(seat, dino_id, id, ...)` | 鎖定射擊 (w3) |
| `ViolentShoot(seat, dino_id, bulletIDs, bet, ...)` | 狂暴射擊 (w4) |
| `ThunderShoot(seat, dino_id, bulletIDs, bet, ...)` | 電擊射擊 (w5) |
| `ThunderTwoShoot(seat, dino_id, bulletIDs, bet, bonusDinoIDs, ...)` | 雷鳴射擊 (w7) |
| `Blast(seat, bulletID, bulletType, monsterIDs)` | 命中請求 (w2) |
| `ReturnShoot(seatID, bulletIDs, pos)` | 返還射擊 (w8) |

### 捕獲結果資料結構

#### DeadMonsterData

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Type` | `int` | 怪物類型 |
| `Odds` | `int` | 倍率 |
| `Wins` | `double` | 贏得金額 |
| `Sid` | `int` | 怪物 SID |
| `SubType` | `int` | 子類型 |
| `Gem` | `float` | 寶珠 |
| `Ticket` | `int` | 票券 |
| `ShowAwardsType` | `int` | 報獎類型 |
| `Pos` | `Vector3` | 怪物位置 |
| `ClientShow` | `JSON` | 演出資料 |

#### MonsterDieEventArgs

| 屬性 | 類型 | 說明 |
|------|------|------|
| `PlayerSeat` | `int` | 玩家座位 |
| `PlayerID` | `int` | 玩家 ID |
| `Bet` | `int` | 押注量 |
| `BulletID` | `string` | 子彈 ID |
| `MonsterWin` | `double` | 怪物贏分 |
| `CreditType` | `CreditType` | 金流類型 |
| `DieMonsterDataList` | `List<DeadMonsterData>` | 死亡怪物列表 |
| `IsCoinAddToUI` | `bool` | 是否加錢到 UI |
| `IsFeatureKill` | `bool` | 是否為特殊擊殺 |

---

## 6. PlayerSystem / PlayerInfo / PlayerData (玩家管理)

### PlayerInfo（繼承 `BaseGameData`）

```csharp
var playerInfo = GameClient.Instance.GetGameData<PlayerInfo>();
```

| 方法/屬性 | 類型 | 說明 |
|-----------|------|------|
| `MainPlayer` | `PlayerData` | 本家玩家資料 |
| `GetPlayer(int seat)` | `PlayerData` | 依座位取得玩家 |
| `AddPlayer(PlayerData)` | `void` | 新增玩家 |
| `RemovePlayer(int seat)` | `void` | 移除玩家 |

### PlayerData 屬性（注意大寫開頭）

| 屬性 | 類型 | 說明 |
|------|------|------|
| `Name` | `string` | 玩家名稱 |
| `Seat` | `int` | 座位 |
| `Weapon` | `int` | 武器 |
| `VipLevel` | `int` | VIP 等級 |
| `BetValue` | `int` | 押注量（⚠️ 不是 `Bet`） |
| `Coin` | `double` | 金幣 |
| `Credit` | `double` | 信用額度 |
| `Gem` | `double` | 寶珠 |
| `Country` | `string` | 國籍 |
| `ID` | `int` | 玩家 ID |
| `Level` | `int` | 等級 |
| `Ticket` | `int` | 票券 |
| `Main` | `bool` | 是否為本家 |

---

## 7. BaseMonster (怪物基類) 與繼承體系

### 繼承架構圖

```
BaseMonster (MonoBehaviour)
├── PathAnimal                    ← 路徑移動怪物（最常用）
│   ├── JurasGoldenMammothCtrl   ← 黃金猛瑪象（翻倍怪物）
│   ├── MoistureDuckCtrl         ← 潤水鴨（翻倍怪物）
│   ├── BossRockcrystalDragon    ← 岩晶龍（BOSS）
│   ├── BossGoblinRobot          ← 機甲哥布林（BOSS 技能）
│   ├── BossSlimePathAnimal      ← 史萊姆王（BOSS 特殊）
│   ├── BossLichPathAnimal       ← 巫妖王（BOSS）
│   ├── BossTianYuPathAnimalCtrl ← 天羽 BOSS
│   ├── RobinEternalAnimal       ← 天羽覺醒怪物
│   ├── HalloweenGhost3DCtrl    ← 3D 幽靈魚
│   ├── JurasTreasureTurtleCtrl  ← 財寶樹龜
│   ├── JurasBrontoCtrl          ← 雷龍
│   ├── JurasFrilledLizardCtrl   ← 閃電傘蜥蜴
│   ├── JurasStoneoCtrl          ← 岩石龍
│   ├── JurasPteroCtrl           ← 翼龍
│   ├── ChargeTriceratopsCtrl    ← 衝鋒三角龍
│   ├── CollectEventMonsterCtrl  ← 收集活動怪物
│   └── VolcanoTurtleCtrl        ← 火山龜
├── StaticAnimal                  ← 靜態生成怪物（不走路徑）
│   ├── JurasTreasureSakuraCtrl  ← 櫻花樹
│   └── GardenEelCtrl            ← 花園鰻
```

### 選擇繼承基類的判斷

| 基類 | 適用場景 | 生成方式 (activeType) |
|------|----------|----------------------|
| `PathAnimal` | 沿路徑移動的怪物（大多數怪物） | NormalSpawn (0) |
| `StaticAnimal` | 固定位置的怪物（不需路徑） | StaticSpawn (2) |
| `BaseMonster` | 自訂移動邏輯的怪物 | 視需求 |

### Init 方法簽名

```csharp
// PathAnimal / BaseMonster 使用
public virtual void Init(
    int monsterType, float o, int sid, string path, 
    float syncTime, int fakeCmd, JSON extraData = null, 
    int subType = 0, int limitVip = 0)

// StaticAnimal 使用
public virtual void StaticMonsterInit(
    int monsterType, float o, int sid, float syncTime)
```

### Init 內部流程

1. 設定基本屬性（`monsterType`、`sid`、`orientation`、`fakeCmd`、`limitVip`、`extraData`）
2. 檢查 `m_KillBeforeInitList`（初始化前已被擊殺 → 直接 `KillImmediately`）
3. `InitParameter()` — 從 `MonsterParameterDataDict` 讀取怪物參數
4. `InitAwardParameter()` — 初始化報獎相關變數
5. 重設材質、顏色、透明度
6. 初始化 `monsterController`（`PathController` / `ArkSteerBehavior`）
7. 設定動畫狀態為 `MOVE`
8. 設定狀態為 `MonsterMoving`
9. 觸發 `DeclareManager.MonsterDeclare()` 來襲宣告

### 重要屬性

| 屬性 | 類型 | 存取 | 說明 |
|------|------|------|------|
| `Sid` | `int` | get | 怪物唯一 SID（Server 分配） |
| `MonsterType` | `int` | get | 怪物類型代號（如 9112） |
| `MonsterSubType` | `int` | get | 子類型 |
| `State` | `EnumMonsterState` | get/set | 當前狀態 |
| `Lockable` | `bool` | get/set | 是否可鎖定（set 時 DEBUG_MODE 下會變色） |
| `IsTransparent` | `bool` | get | 是否透明 |
| `IsKill` | `bool` | get/set | 是否已被擊殺 |
| `ParameterData` | `MonsterParameterData` | get | 怪物參數資料 |
| `MonsterController` | `BaseMonsterController` | get | 移動控制器 |
| `ExtraData` | `JSON` | get/set | f1 封包的額外資料 |
| `LimitVip` | `int` | get/set | VIP 限制 |
| `m_IsFeatureMonster` | `bool` | SerializeField | 是否為特殊魚種（不走通用 WeaponManager 流程） |
| `m_IsEternalLife` | `bool` | SerializeField | 是否為永生魚種 |
| `m_IsImmuneToSpecialWeapon` | `bool` | SerializeField | 使用特殊武器時是否免疫（變透明） |
| `m_specialAwardType` | `SpecialAwardType` | SerializeField | 特殊報獎型別 |

### 狀態判斷方法

| 方法 | 回傳 | 說明 |
|------|------|------|
| `IsAttackable()` | `bool` | 是否可攻擊（Moving 狀態 + 非透明） |
| `IsAttackable(int seat)` | `bool` | 指定座位是否可攻擊（含 VIP 限制檢查） |
| `IsDead()` | `bool` | 是否已死亡（Getting 狀態） |
| `IsFeatureMonster()` | `bool` | 是否為特殊魚種 |
| `IsImmuneToSpecialWeapon()` | `bool` | 是否免疫特殊武器 |
| `IsEternelLife()` | `bool` | 是否為永生魚種 |
| `IsSpecialAward()` | `bool` | 是否有特殊報獎 |
| `IsMaxOdds()` | `bool` | 是否達到最大倍率 |

### 生命週期方法

| 方法 | 說明 |
|------|------|
| `Kill(seat, isGetPhysics, stayTime, additionalStayTime, winType)` | 擊殺怪物（觸發死亡動畫 + 報獎計時） |
| `ActDead(killPlayerSeatID, isGetPhysics, additionalStayTime, winType)` | 執行死亡演出（換貼圖、播動畫、延遲回收） |
| `KillImmediately()` | 立即銷毀（跳過動畫） |
| `ForceDead()` | 強制死亡（用於清場） |
| `QuickLeave()` | 快速離場 |
| `ChangeToFeatureState()` | 切換到特殊狀態（`MonsterFeature`，停止移動） |
| `MonsterOutBorder()` | 怪物離開邊界時回收 |
| `Despawn()` | 回收到 Object Pool（`SpawnPool`） |
| `End()` | 內部結束方法（清理 + Despawn） |

### 視覺效果方法

| 方法 | 說明 |
|------|------|
| `SetTransparent(bool)` | 設定透明（特殊武器使用時） |
| `SetAnimationState(MonsterAnimaitonState)` | 設定動畫狀態（MOVE/IDEL/HIT/CATCH） |
| `SetAnimationSpeed(float)` | 設定動畫速度 |
| `DeadAnimation()` | 播放死亡動畫 |
| `Freeze(bool, float)` | 冰凍效果（顯示冰凍物件 + 停止移動） |
| `Paralysis(bool, float)` | 麻痺效果（停止移動一段時間） |
| `ToggleLightingEffect(seat, enable, hueOffset)` | 切換閃電材質效果 |
| `ChangeColor(Color)` / `ResetOriginColor()` | 變色 / 重設顏色 |

### 報獎相關方法

| 方法 | 說明 |
|------|------|
| `SetAwardParameter(win, odds, bet, winGem, additionalTime)` | 設定報獎參數（由 WeaponSystem 呼叫） |
| `GetShowAwardTime()` | 取得報獎演出時間 |
| `GetAwardWiningWinList()` | 取得翻倍 Win 陣列（翻倍怪物用） |

### 怪物狀態 (EnumMonsterState)

| 狀態 | 值 | 說明 |
|------|-----|------|
| `MonsterIdle` | 0 | 未使用 |
| `MonsterMoving` | 1 | 移動中（可攻擊） |
| `MonsterGeting` | 2 | 被捕獲中（死亡） |
| `MonsterLeaving` | 3 | 離場中 |
| `MonsterFeature` | 4 | 特殊狀態中（演出中，不可攻擊） |

### 動畫狀態 (MonsterAnimaitonState)

| 狀態 | 值 | 說明 |
|------|-----|------|
| `MOVE` | 0 | 移動 |
| `IDEL` | 1 | 待機 |
| `HIT` | 3 | 打擊 |
| `CATCH` | 9 | 死亡 |

### PathAnimal 特殊行為

`PathAnimal` 在 `Init` 中會：
1. 取得 `PathController`（`monsterController as PathController`）
2. 呼叫 `pathController.SetPath(path)` 設定路徑
3. 若路徑不存在 → 直接 `Despawn()` 回收
4. 若路徑存在 → 呼叫 `base.Init()` + `CustomInit(monsterType)`

### StaticAnimal 特殊行為

`StaticAnimal` 使用 `StaticMonsterInit` 而非 `Init`：
- 不需要 `path` 參數
- 固定朝向 180 度
- 有自訂的 `OnHit` 和 `ActDead` 覆寫（支援受擊動畫、死亡貼圖切換）

### 新增怪物控制器的選擇指南

| 怪物類型 | 建議繼承 | 原因 |
|----------|----------|------|
| 一般怪物（沿路徑移動） | 不需要控制器，使用預設 `BaseMonster` | `MonsterManager` 會自動處理 |
| 翻倍怪物（需要倍率 UI） | `PathAnimal` 或 `BaseMonster` | 需要自訂 Init 處理 extraData 中的 odds/level |
| BOSS（沿路徑 + 特殊行為） | `PathAnimal` | 需要覆寫 Init、處理事件、自訂死亡演出 |
| 靜態怪物（固定位置） | `StaticAnimal` | 不需路徑，使用 `StaticMonsterInit` |
| 覺醒怪物（路徑 + 變身） | `RobinEternalAnimal` | 已有覺醒機制 |

---

## 8. TableManager (廳館資源管理)

- **如何取得**: `TableManager.Instance`（MonoBehaviour 單例）
- `GetStaticString_CHT(string key)`: 多語系字串
- `GetStageMonster(uint key)`: 關卡怪物設定
- `StartLoad(string tableName)`: 載入廳館資源

---

## 9. DinoPinBallM (場景入口)

- **類型**: `MonoBehaviour`，場景中的主要入口
- **單例**: `DinoPinBallM.Instance`
- **廳館型別**: `public GameLevel Table`（`GameLogic_Lobby.GameLevel` enum）
- **廳館判斷**: `DinoPinBallM.Instance.Table == GameLevel.juras`（直接用 enum 比對）
- **⚠️ Obsolete 屬性**：`isDino` / `isJuras` / `isStoneage` / `isRobin` 已標記 `[Obsolete]`，新程式碼禁止使用

### 廳館判斷寫法

```csharp
using GameLevel = DinoPinBall.GameLogic_Lobby.GameLevel;

// ✅ 正確：直接用 enum 比對
if (DinoPinBallM.Instance.Table == GameLevel.juras) { ... }
if (DinoPinBallM.Instance.Table != GameLevel.juras) { ... }

// ❌ 錯誤：使用已過時的 bool 屬性
if (DinoPinBallM.Instance.isJuras) { ... }
if (DinoPinBallM.Instance.isDino || DinoPinBallM.Instance.isStoneage || ...) { ... }
```

### 初始化流程
```
Awake → 設定 FPS(30) / fixedDeltaTime(0.0333f) / Table = GameLogic_Lobby.Instance.gameLevel
Start → GameManager.Init(gameClient, table)
     → TableModuleRegistry.GetModule(table) → tableModule.Init(gameClient, table)
       → BaseTableModule.Init（共用 Manager）
       → XxxTableModule.Init（廳館專屬）
     → TableManager.StartLoad(table) → EnterGame
     → Socket 連線 → join_table → init_game → init_player → update_player
     → f4 怪物同步 → Loading 頁關閉
```

---

## 10. Debug 系統

### 條件編譯符號差異

| 符號 | 用途 | 說明 |
|------|------|------|
| `DEBUG_MODE` | 測試版模式 | 啟用 CheatTool、DebugSystem、假付費等 |
| `DEBUG_LOG` | Debug 日誌 | 包裹 `Debug.Log` 輸出，**新增程式碼統一使用此符號** |

⚠️ 新增的 Debug 工具腳本統一使用 `#if DEBUG_LOG`，不使用 `#if DEBUG_MODE`。

### DebugManager API

| 方法 | 說明 |
|------|------|
| `TestCreateDinoOnlyClient(int id, int value, int fakeKill)` | Client 端假生成怪物 |
| `TestCreateDino(string ID, int value, int subType)` | 透過 Server 生成怪物 |

### CheatToolManager API

| 方法 | 說明 |
|------|------|
| `debugCreateDino(string sDinoID)` | 依 serverName 假生成怪物（如 `"MOISTURE_DUCK"`） |

---

## 11. 跨模組橋接 — FishHunter_Agent

`FishHunter_Agent` 連接 ArkGame、OceanTreasure、DinoPinBall 三個模組。

- **位置**: `Assets/FishHunter_Script/FishHunter_Agent.cs`
- **類型**: `MonoBehaviour`，`DontDestroyOnLoad`
- **條件編譯**: `FH_EMPTY_PROJECT` 下所有事件註冊被跳過

### 事件前綴對應

| 前綴 | 模組 | 範例 |
|------|------|------|
| `AG_` | ArkGame | `AG_AgentEvent.GameClient_getGameSystem_sendPauseGameRequest` |
| `OT_` | OceanTreasure | `OT_AgentEvent.PlayerUIManager_ReFlashData` |
| `DP_` | DinoPinBall | `DP_AgentEvent.GameUIManager_ReFlashData` |
| 無前綴 | 跨模組共用 | `AgentEvent.EntityLibrary_TableManager_Reset` |

### 三種註冊類型
1. `RegisterInsExistFunc`: 確認 Manager/Singleton 是否存在
2. `Registration`: 無回傳值事件
3. `RegisterReturnValueFunc`: 有回傳值事件

### 使用範例
```csharp
bool exists = FishHunter_AgentEventManager.IsInsExist(AG_InsEnum.GameClient);
FishHunter_AgentEventManager.SendEvent(DP_AgentEvent.GameUIManager_ReFlashData);
int seat = (int)FishHunter_AgentEventManager.getReturnValue(AG_ValueEvent.GameClient_getGameSystem_getMainPlayerSeat);
```

---

## 12. 開發建議

1. 修改 `MonsterSystem.cs` 或 `MonsterSystemDataCmd.cs`（若有新指令）
2. 確保 `Dino_EventManager` 有對應事件定義
3. 在 `MonsterManager` 中處理 `MONSTER_ACTIVE` 事件
4. 跨模組功能需在 `FishHunter_AgentEnum` 新增事件列舉，並在 `FishHunter_Agent.cs` 註冊處理函式
5. **新怪物報獎**：覆寫 `BaseMonster.HandleAward()` 處理自訂報獎，不需修改 `WeaponManager` 的 switch（詳見 `Auto_dinopinball_machine_expert.md` 5.1 節）
