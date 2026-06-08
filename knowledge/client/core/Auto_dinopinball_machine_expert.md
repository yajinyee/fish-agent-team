---
inclusion: always
---

# DinoPinBall (3D 魚機) 開發指南

你是一位專精於「捕魚機 (FishHunter)」街機遊戲架構的 Unity 資深開發者。
本文件適用於 DinoPinBall / DinoGame (3D 魚機) 開發。
OceanTreasure (2D 魚機) 請參閱 `Auto_fishhunter_machine_expert.md`。

## 0. 語言與溝通規範

- 所有對話、思考過程、文件產出使用**繁體中文**
- 程式碼中的專有名詞 (Class Name, Variable Name) 保持英文，解釋時輔以中文

## 0.1 核心術語表

| 術語 | 說明 |
|------|------|
| `MonsterSystem` | 處理 Server 端怪物指令（f1/f4/f7），對應 ArkGame 的 FishSystem |
| `MonsterManager` | 怪物實體管理器，監聽事件並實例化怪物 |
| `MonsterData` | 怪物資料結構 |
| `WeaponSystem` | 處理武器射擊和捕獲（w2 封包） |
| `SkillSystem` | 技能系統（獨立於 ArkGame 的 SkillSystem） |
| `EquipmentSystem` | 砲台養成系統（DinoPinBall 專有） |
| `Dino_EventManager` | DinoPinBall 內部事件系統（靜態方法） |
| `Dino_EventType` | 事件類型定義（多個 enum 類別） |
| `BaseGameData` | 資料基類（`PlayerInfo`、`GameInfo` 繼承） |
| `TableManager` | 廳館資源管理（AssetBundle、多語系、參數），`StartLoad(GameLevel)` |
| `DinoPinBallM` | DinoPinBall 場景入口 MonoBehaviour，`Table` 欄位為 `GameLevel` enum |
| `GameLevel` | 廳館類型 enum（`GameLogic_Lobby.GameLevel`），各檔案用 `using GameLevel = DinoPinBall.GameLogic_Lobby.GameLevel;` 引用 |
| `ITableModule` | 廳館專屬模組介面（`Init` + `Destroy`） |
| `BaseTableModule` | 共用 Manager 初始化基底類別（所有廳館共用的 12 個 Manager） |
| `TableModuleRegistry` | 靜態 Dictionary，根據 `GameLevel` 取得對應的 `ITableModule` |
| `本家 (Self)` | 當前玩家自己捕獲時的完整演出模式 |
| `他家 (Other)` | 其他玩家捕獲時的簡化演出模式 |

## 1. 架構概要

- **Namespace**: `DinoPinBall`
- **程式碼位置**: `Assets/FishHunter_Script/DinoGame/DinoPinBall/`
- **系統入口**: `DinoPinBall.GameClient.Instance`（延遲初始化單例）
- **架構風格**: 鬆耦合、泛型 `GetSystem<T>()`、依賴注入
- **事件系統**: `Dino_EventManager`（靜態方法）
- **BaseSystem 建構子**: `base(gameDataManager, eventManager, cmdSender, "name")`
- **OnMessage**: `OnMessage(JSON json)`（大寫 O）
- **實作介面**: `IGameClient`, `ISystemManager`, `IGameDataManager`, `IEventManager`
- **廳館型別**: `GameLogic_Lobby.GameLevel` enum（`dino`, `juras`, `stoneage`, `robin`, `dino3`）
- **廳館模組化**: `ITableModule` → `BaseTableModule` → 各廳館 Module（`RobinTableModule`, `Dino3TableModule`）
- **Manager Init 簽名**: `Init(GameClient gameClient, GameLevel table)`（所有 Manager 統一）
- **using 別名慣例**: `using GameLevel = DinoPinBall.GameLogic_Lobby.GameLevel;`

## 2. 系統管理

### 系統存取
```csharp
var monsterSystem = DinoPinBall.GameClient.Instance.GetSystem<MonsterSystem>();
var skillSystem = DinoPinBall.GameClient.Instance.GetSystem<SkillSystem>();
```

### 資料存取（邏輯與資料分離）
```csharp
var playerInfo = DinoPinBall.GameClient.Instance.GetGameData<PlayerInfo>();
var gameInfo = DinoPinBall.GameClient.Instance.GetGameData<GameInfo>();
```

- **Logic**: 寫在 `System`（如 `PlayerSystem`）
- **Data**: 寫在 `Info`（如 `PlayerInfo`），繼承 `BaseGameData`

### 已註冊的系統
`WeaponSystem`, `PlayerSystem`, `GameSystem`, `MonsterSystem`, `SkillSystem`, `TableSystem`, `DP_JackpotSystem`, `DP_DailyRankSystem`, `SocialSystem`, `MissionSystem`, `EquipmentSystem`
（`DEBUG_MODE` 下額外加入 `DebugSystem`）

### 已註冊的資料
`GameInfo`, `PlayerInfo`

### ⚠️ IEventManager 介面
`GameClient` 實作了 `IEventManager`，但 `SendEvent`/`Registration`/`Cancellation` 皆拋出 `NotImplementedException`。
實際使用 `Dino_EventManager`（靜態方法）。

## 3. Dino_EventManager 事件系統

### ⚠️ 兩套 EventBase 類別（切勿混淆）

| 類別 | 位置 | 用途 |
|------|------|------|
| `Dino_EventManager.EventBase` | 巢狀於 `Dino_EventManager` 內 | `Dino_EventManager` 靜態方法使用 |
| `DinoPinBall.EventBase` | namespace 層級 `EventManager.cs` | `IEventManager` 介面使用（目前未實作） |

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

## 4. 生命週期摘要

### 怪物生成
Server f1 → MonsterSystem.OnMessage → 解析 MonsterData → Dino_EventManager.SendEvent(MONSTER_ACTIVE, data) → MonsterManager 監聽 → 實例化怪物

### 捕獲流程
Client w1/w3 射擊 → Server w2 回應 → WeaponSystem.OnMessage → Dino_EventManager.SendEvent(捕獲事件) → UI/演出系統監聽

### Client 端模擬生成
```csharp
MonsterData data = new MonsterData();
data.ID = 1001;
data.MonsterType = 9112;
data.Path = "LARGE_1";
Dino_EventManager.SendEvent(Dino_EventType.MonsterSystemEvent.MONSTER_ACTIVE, data);
```

## 5. DinoPinBall 開發規範

1. 嚴禁 `new System()`，必須用 `GameClient.Instance.GetSystem<T>()`
2. 跨系統溝通必須用 `Dino_EventManager`（`Registration` / `Cancellation`，不是 Register/Unregister）
3. 禁止 A System 直接呼叫 B System 的 public 方法來觸發邏輯（Getter 除外）
4. 資料存取透過 `GetGameData<T>()`，不要在 System 內部存放資料
5. 靜態字串、關卡參數必須透過 `TableManager` 載入，禁止 Hard-code
6. 新增怪物需同時維護 `MonsterData`（System 層）與 `MonsterManager`（View 層）
7. 跨模組通訊（↔ OceanTreasure）必須透過 `FishHunter_AgentEventManager`
8. **廳館判斷使用 `GameLevel` enum**，禁止字串比對（`"dino"`、`"juras"` 等）
9. **Manager Init 簽名統一為 `Init(GameClient, GameLevel)`**，內部存為 `private GameLevel Table;`
10. **廳館專屬功能透過 `ITableModule` 模組化**，不在 `GameManager.Init` 中用 `if (table == xxx)` 判斷

### 廳館模組化體系（ITableModule）

```
GameManager.Init(gameClient, table)
  → TableModuleRegistry.GetModule(table)
    → BaseTableModule.Init()        ← 共用 12 個 Manager 初始化
      → RobinTableModule.Init()     ← base.Init + Robin 專屬 Manager
      → Dino3TableModule.Init()     ← base.Init + Dino3 專屬（預留）
      → BaseTableModule（fallback） ← 無專屬功能的廳館
```

**新增廳館時**：
1. `GameLogic_Lobby.GameLevel` enum 加值
2. 如有專屬功能 → 建立 `XxxTableModule : BaseTableModule` + 在 `TableModuleRegistry` 加 entry
3. 如無專屬功能 → 不需改動（自動走 `BaseTableModule`）

**switch 策略**：
- 模式一（80%）：`case GameLevel.juras: ... break; default: ... break;`（新廳館自動走 default）
- 模式二（SkillUI/Marquee/TableManager）：明確列出各 case 或 `case GameLevel.dino: ... default: ...`
- 模式三（InfoPage）：各廳館獨立 case + default fallback

## 5.1 W2 報獎架構（新架構）

### 架構概要

W2 報獎流程分為兩層解耦：
1. **WeaponManager ↔ 怪物**：透過 `BaseMonster.HandleAward()` 虛擬方法，怪物自己決定報獎方式與生死
2. **怪物 ↔ 報獎 UI**：透過 `AwardManager` 呼叫對應的報獎方法

> ⚠️ 本架構僅適用於 W2 封包觸發的報獎。SkillSystem 走的報獎路徑（sk_xxx 指令）不在此範圍，技能怪物的報獎由各自的 SkillObject / SkillModel 處理。

### W2 報獎流程

```
Server W2 封包
  ↓
WeaponSystem.onBlast()
  ├─ 解析 MonsterDeadDict → 建立 DeadMonsterData
  ├─ 打包 BlastEventData → 發送 BLAST 事件
  ↓
WeaponManager.OnBlastEvent()
  ├─ monster.SetAwardParameter(wins, odds, bet, gem)
  ├─ 補充 DeadMonsterData（Type, Pos, WiningList, IsMaxOdds, ShowAwardsTime）
  ├─ args.DieMonsterDataList.Add / DieMonsterList.Add  ← 先加入清單
  ├─ monster.HandleAward(monsterData, args)  ← 新架構入口（此時 args 已有本怪物資料）
  │   ├─ 回傳非 null → 怪物自己處理了報獎
  │   │   ├─ ShouldKill=true  → WeaponManager 執行 KillMonsters（清單裡有資料）
  │   │   └─ ShouldKill=false → 從清單 Remove（半永生，怪物繼續存活）
  │   └─ 回傳 null → 從清單 Remove（還原，因為舊流程各 case 自己決定何時 Add）
  │        └─ 進入舊 switch 邏輯（IsFeatureMonster / IsSpecialAward / 一般怪物）
  └─ 一般怪物 → NormalShowAwards + KillMonsters
```

### HandleAward 虛擬方法

```csharp
// BaseMonster.cs
/// <summary>
/// 自訂報獎處理。WeaponManager 在 SetAwardParameter 之後、KillMonsters 之前呼叫。
/// 呼叫時 args.DieMonsterDataList 已包含本怪物資料，可直接傳給 AwardManager。
/// 回傳 null = 走通用流程（舊 switch 邏輯）。
/// 回傳 AwardHandleResult = 怪物自己處理了報獎演出：
///   ShouldKill=true  → WeaponManager 執行 KillMonsters
///   ShouldKill=false → WeaponManager 從清單移除，怪物繼續存活（半永生）
/// </summary>
public virtual AwardHandleResult HandleAward(DeadMonsterData monsterData, MonsterDieEventArgs args)
{
    return null;
}
```

### AwardHandleResult

```csharp
// Award/AwardHandleResult.cs
public class AwardHandleResult
{
    public bool ShouldKill { get; set; }
    public AwardHandleResult(bool shouldKill) { ShouldKill = shouldKill; }
}
```

### 4 種報獎模式

| 模式 | 說明 | AwardManager 方法 | 視覺表現 |
|------|------|-------------------|---------|
| 1. 金幣飛出 | 通用小獎 | `NormalShowAwards(args)` | 金幣從怪物飛到玩家 UI |
| 2. 翻倍盤 | 翻倍數字演出 | `ShowDoubleAward(monsterData, args)` | 圓盤翻倍數字 + 金幣收集 |
| 3. 螢幕中間報獎 | 怪物專屬報獎 | `ShowSpecialAward(monsterName, seat, pos, bet, coin)` | 螢幕中間報獎面板 |
| 4. 大報獎面板 | 全螢幕大獎 | `ShowLevelNewAward(awardType, seat, pos, bet, coin, ...)` | 全螢幕報獎面板 |

### HandleAward 實作範例

```csharp
// 半永生 + 捕獲模式（如牛龍、史前館怪物）
public override AwardHandleResult HandleAward(DeadMonsterData monsterData, MonsterDieEventArgs args)
{
    switch (monsterData.ShowAwardsType)
    {
        case 1: // 半永生獎：金幣飛出，不殺
            AwardManager.Instance.NormalShowAwards(args);
            return new AwardHandleResult(shouldKill: false);

        case 2: // 捕獲獎：報獎面板，殺死
            AwardManager.Instance.ShowDoubleAward(monsterData, args);
            return new AwardHandleResult(shouldKill: true);

        default:
            return null; // 走通用流程
    }
}
```

### 新怪物報獎接入規範

1. **一般怪物**（無特殊報獎）：不需覆寫 `HandleAward`，走通用 `NormalShowAwards + KillMonsters`
2. **有特殊報獎的怪物**：覆寫 `HandleAward`，在方法內呼叫對應的 AwardManager 方法，回傳 `AwardHandleResult`
3. **半永生怪物**：`ShowAwardsType==1` 回傳 `ShouldKill=false`，`ShowAwardsType==2` 回傳 `ShouldKill=true`
4. **禁止**在 `HandleAward` 中操作 `args.DieMonsterDataList`（Add/Remove 由 WeaponManager 統一管理）
5. **舊怪物**不需遷移，`HandleAward` 預設回傳 `null`，繼續走舊 switch 邏輯
6. **禁止**覆寫 `HandleAward` 的怪物同時監聽 `Dino_EventType.WeaponSystemEvent.BLAST` 事件，否則會重複處理報獎（`HandleAward` 已由 WeaponManager 在 BLAST 事件處理鏈中呼叫）

### 向下相容

- `HandleAward` 預設回傳 `null`，所有舊怪物行為不變
- WeaponManager 的舊 switch 邏輯完整保留
- 新怪物覆寫 `HandleAward` 後，不需要在 WeaponManager 加 case
- 舊怪物可逐步遷移到 `HandleAward`，每遷移一個就從 switch 刪一個 case

## 6. TableManager (廳館資源管理)

- **如何取得**: `TableManager.Instance`（MonoBehaviour 單例）
- `GetStaticString_CHT(string key)`: 多語系字串
- `GetStageMonster(uint key)`: 關卡怪物設定
- `StartLoad(string tableName)`: 載入廳館資源

---

## 共用規範（OceanTreasure / DinoPinBall 通用）

### 非同步與流程控制

- **全面使用 UniTask**：嚴禁 `Coroutine`（`IEnumerator` / `StartCoroutine`）
- **Async/Await**：禁止 `Task.Wait()` 或 `.Result`，避免 Deadlock
- **CancellationToken**：async 方法必須接受並傳遞 `CancellationToken`

### CancellationTokenSource (CTS) 使用規範

DinoPinBall 的怪物使用 Object Pool（SpawnPool）回收再利用，同一個 MonoBehaviour 實例會經歷多次 Init → End → Init 循環。CTS 管理不當會導致：
- NullReferenceException（End 後 cts 為 null，但非同步流程還在跑）
- ObjectDisposedException（Dispose 後又存取 cts.Token）
- 上一隻的非同步流程殘留到下一隻（沒 Cancel 就直接 new）

#### 規則一覽

| 規則 | 說明 |
|------|------|
| Init 先清後建 | `cts?.Cancel(); cts?.Dispose(); cts = new CTS();` 三行一組 |
| End 先清後 base | 先 `cts?.Cancel(); cts?.Dispose(); cts = null;` 再 `base.End()`（base.End 會觸發 Despawn 回收） |
| 永遠用 `?.` | `cts?.Cancel(); cts?.Dispose();` — 防止重入或未初始化時 NullRef |
| 永遠設 null | Dispose 後必須 `cts = null;` — 防止存取已釋放的物件 |
| Token 取一次 | `var token = cts.Token;` 在 async 方法開頭取一次，後續傳遞 token 而非重複存取 cts.Token |
| 子 CTS 獨立管理 | 需要獨立取消的子流程（如爬起倒數）用獨立的 CTS，不共用主 CTS |
| catch 必清理 | `catch (OperationCanceledException)` 中必須清理中間狀態，不可留空（除非確認無中間狀態） |

#### 標準模板（Object Pool 怪物用）

```csharp
private CancellationTokenSource cts;

public override void Init(int monsterType, float o, int sid, string path,
                          float syncTime, int fakeCmd, JSON extraData = null,
                          int subType = 0, int limitVip = 0)
{
    // ① Init 先清後建：確保上一隻的非同步流程被取消
    cts?.Cancel();
    cts?.Dispose();
    cts = new CancellationTokenSource();

    base.Init(monsterType, o, sid, path, syncTime, fakeCmd, extraData, subType, limitVip);

    // ② 啟動非同步流程
    RunAsync(cts.Token).Forget();
}

protected override void End()
{
    // ③ End 先清後 base：自己的清理在 base.End() 之前
    cts?.Cancel();
    cts?.Dispose();
    cts = null;
    base.End();  // base.End() 會觸發 Despawn 回收，之後物件狀態不可靠
}

private async UniTaskVoid RunAsync(CancellationToken token)
{
    try
    {
        // ④ 用傳入的 token，不要再存取 cts.Token
        await DoSomething(token);
    }
    catch (OperationCanceledException)
    {
        // ⑤ 取消時清理中間狀態
        CleanupIntermediateState();
    }
    catch (Exception e)
    {
    #if DEBUG_LOG
        Debug.LogError($"[Monster] Error: {e}");
    #endif
        CleanupIntermediateState();
    }
}
```

#### 子 CTS 模板（獨立取消的子流程）

```csharp
private CancellationTokenSource wakeUpCts;

private void StartWakeUpCountdown(float delaySec)
{
    // 取消舊的再建新的
    CancelWakeUpCountdown();
    wakeUpCts = new CancellationTokenSource();
    WakeUpAsync(delaySec, wakeUpCts.Token).Forget();
}

private void CancelWakeUpCountdown()
{
    wakeUpCts?.Cancel();
    wakeUpCts?.Dispose();
    wakeUpCts = null;
}

// End() 中也要清理子 CTS
protected override void End()
{
    cts?.Cancel();
    cts?.Dispose();
    cts = null;
    CancelWakeUpCountdown();  // 子 CTS 也要清
    base.End();
}
```

#### ❌ 常見錯誤

```csharp
// ❌ 沒有 null 檢查 — 重入時 NullRef
cts.Cancel();
cts.Dispose();

// ❌ Dispose 後沒設 null — 下次存取 cts.Token 會 ObjectDisposedException
cts?.Cancel();
cts?.Dispose();
// 缺少 cts = null;

// ❌ Init 沒有先 Cancel 舊的 — 上一隻的非同步流程殘留
cts = new CancellationTokenSource();  // 舊的 cts 洩漏

// ❌ base.End() 放在清理之前 — base.End() 觸發 Despawn 後物件狀態不可靠
base.End();
cts?.Cancel();  // 可能已經被回收了

// ❌ catch (OperationCanceledException) 留空 — 砲台鎖住、事件沒取消
catch (OperationCanceledException) { }  // 危險：可能有中間狀態沒清理
```

> ⚠️ **注意**：`catch (OperationCanceledException)` 不代表「不需要處理」。取消可能發生在流程的任何中間狀態（砲台已切換、怪物已停止移動、事件已註冊等），必須確保每個退出路徑都有對應的清理邏輯。只有在確認取消時不存在任何需要清理的中間狀態時，才可以留空。

### Action System 模組化

- `FH_SpineActionSystem` — Spine 動畫播放與等待
- `FH_TimelineActionSystem` — Timeline 播放（運鏡、Cutscene）
- `FH_AnimatorActionSystem` — Unity Animator 狀態切換

### 本家/他家判定 (IsPlayerSelf)

| 面向 | 本家 (Self) | 他家 (Other) |
|------|------------|-------------|
| 演出 | 完整演出 | 僅主要 Visual |
| UI 面板 | 顯示結算面板 | 隱藏 |
| 音效 | 完整播放 | 減半或靜音 |
| 背景 | 壓黑背景 | 不壓黑 |
| 砲台 | 鎖定砲台 | 不鎖定 |
| Scale | `_selfScale` | `_otherScale`（較小） |

### 程式碼風格

- **繁體中文註解**：方法用 XML `<summary>`，複雜邏輯前加 `//` 區塊註解
- **序列化欄位**：`[SerializeField] private`
- **Odin Inspector**：善用 `[BoxGroup]`, `[LabelText]`, `[Button]`
- **Debug Log**：用 `#if DEBUG_LOG` 包裹
- **Debug 工具腳本**：新增的 Debug 工具統一用 `#if DEBUG_LOG`（不用 `#if DEBUG_MODE`）
- **條件編譯差異**：`DEBUG_MODE` 用於測試版模式（CheatTool、DebugSystem），`DEBUG_LOG` 用於日誌輸出和新增 Debug 工具

```csharp
#if DEBUG_LOG
Debug.Log($"[Feature] TotalWin: {totalWin}");
#endif
```

### readonly struct 減少 GC

```csharp
public readonly struct CaptureData
{
    public readonly int TargetSid;
    public readonly double TotalWin;
    public readonly int PlayerSeat;
    public readonly bool IsPlayerSelf;
    
    public CaptureData(object[] args)
    {
        TargetSid = (int)args[0];
        TotalWin = (double)args[2];
        PlayerSeat = (int)args[3];
        IsPlayerSelf = (bool)args[5];
    }
}
```

### 錯誤處理

所有演出流程的 catch 區塊必須包含 ForceEnd 邏輯：
1. 解鎖砲台（參考下方「砲台鎖定/解鎖標準做法」）
2. 清理資源

### 砲台鎖定/解鎖標準做法（BOSS 覺醒演出用）

DinoPinBall 的砲台鎖定**優先使用 `WeaponManager.Instance.EnablePlayerShoot(bool, int seat)`**，統一封裝了所有鎖定/解鎖邏輯。

```csharp
// ✅ 鎖定砲台（進覺醒演出、BOSS 演出時）
WeaponManager.Instance.EnablePlayerShoot(false, playerSeat);

// ✅ 解鎖砲台（演出結束、catch 清理時）
WeaponManager.Instance.EnablePlayerShoot(true, playerSeat);
```

位置：`WeaponManager.cs` → `#region PlayerShoot Control`

**`seat` 參數用途**：鎖定時呼叫 `SkillSystem.StopAllSkill(seat)` 停止該玩家的技能，解鎖時不需要（`StopAllSkill` 僅在鎖定路徑使用）。

**內部封裝的完整行為：**

| enable | 執行動作 |
|--------|---------|
| `false`（鎖定）| `CanShoot = false` + `StopAllSkill(seat)` + `SetLockUIClick` + `CloseALLLoopSkill` + 關閉 AutoShoot |
| `true`（解鎖）| `CanShoot = true` + `SetUnLockUIClick` + `Event_ReOpenAutoSkill` + 恢復 AutoShoot |

**錯誤處理規範**：catch 區塊中也必須呼叫解鎖，確保演出中斷時砲台不會卡住：
```csharp
catch (OperationCanceledException)
{
    WeaponManager.Instance.EnablePlayerShoot(true, playerSeat); // 確保解鎖
    // ... 其他清理
}
```

⚠️ **注意**：OceanTreasure (2D) 使用 `FishHunter_EventManager.SendEvent(LockPlayerShoot)` 等事件，DinoPinBall (3D) **不使用這些事件**，切勿混用。

### Unity GUID 生成規範

```powershell
[System.Guid]::NewGuid().ToString("N")
```

禁止手動編造 GUID。

### AssetBundle 同名資源陷阱

**同一個 AssetBundle 內，不同類型的資源檔案不可與 Prefab 同名**。
`AssetBundleManager.LoadAssetAsync(bundleName, assetName, typeof(GameObject))` 是以 `assetName` 在 bundle 內搜尋，若 bundle 中同時存在同名的 `.prefab` 和 `.png`（或其他資源），Unity 可能載入到錯誤的資源，導致 `GetAsset<GameObject>()` 回傳 `null`。

**規範**：Prefab 名稱必須在其所屬 AssetBundle 內唯一，不可與 Sprite、Texture、Material 等同名。若有衝突需將非 Prefab 資源改名（如加 `_atlas` 後綴）。

### 跨模組通訊

ArkGame ↔ OceanTreasure ↔ DinoPinBall 之間必須透過 `FishHunter_AgentEventManager`。

### 通用規範

1. 優先檢查是否已有類似 Manager 可用，避免重複造輪子
2. System 之間不應互相持有 reference，透過 Event 或 GameClient 取得
