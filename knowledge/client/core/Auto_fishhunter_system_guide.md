# OceanTreasure (2D 魚機) 系統 API 指南

本指南說明 OceanTreasure / ArkGame (2D 魚機) 的核心系統 API。
DinoPinBall (3D 魚機) 請參閱 `Auto_dinopinball_system_guide.md`。

---

## 1. GameClient (核心入口)

- **Namespace**: `ArkGame`
- **如何取得**: `ArkGame.GameClient.Instance`（延遲初始化，首次存取時建立並呼叫 `Init_GameClient()`）
- **BaseSystem 建構子**: `base(gameClient, "name")`
- **onMessage 方法**: `onMessage(JSON json)`（小寫 o）

### 主要 API（具名 Getter）

| 方法 | 回傳類型 | 說明 |
|------|----------|------|
| `getGameSystem()` | `GameSystem` | 遊戲流程管理 |
| `getPlayerSystem()` | `PlayerSystem` | 玩家資料管理 |
| `getFishSystem()` | `FishSystem` | 魚種系統 |
| `getWeaponSystem()` | `WeaponSystem` | 武器系統 |
| `getSkillSystem()` | `SkillSystem` | 技能系統 |
| `getItemSystem()` | `ItemSystem` | 道具系統 |
| `getJackpotSystem()` | `JackpotSystem` | Jackpot 系統 |
| `getDailyRankSystem()` | `DailyRankSystem` | 每日排行 |
| `getTableSystem()` | `TableSystem` | 桌次系統 |
| `getTournamentRankSystem()` | `TournamentRankSystem` | 錦標賽排行 |
| `getMissionSystem()` | `MissionSystem` | 任務系統 |
| `getSocialSystem()` | `SocialSystem` | 社交系統 |
| `GetPartySystem()` | `PartySystem` | 派對系統 |
| `GetTokenSystem()` | `TokenSystem` | Token 代幣系統 |
| `GetCollectMissionSystem()` | `CollectMissionSystem` | 收集任務 |
| `GetRookieMissionSystem()` | `RookieMissionSystem` | 新手任務 |
| `GetMissionPanelSystem()` | `MissionPanelSystem` | 任務面板 |

### 泛用系統註冊
- `Register(string name, BaseSystem system)`: 以名稱註冊系統（用於 Socket 訊息路由）
- `getSystem(string name)`: 以名稱取得系統

### 其他重要屬性
- `MainPlayerSeat`: 本家玩家座位（`int`）
- `getIsConnect`: Socket 連線狀態（`bool`）

---

## 2. GameSystem (遊戲流程)

- `GetGame()`: 取得 `Game` 資料物件（`TableID`, `SeatID` 等）
- `getMainPlayerSeat()`: 取得本家玩家座位 ID
- `OnGameInitAction`: 遊戲初始化事件 (Action)
- `sendPauseGameRequest()`: 發送暫停遊戲請求
- `AddCoinToList(double creditDiff)` / `RemoveCoinFromList(double coin)`

---

## 3. PlayerSystem (玩家管理)

- `GetPlayer(int seatId)`: 取得指定座位的玩家資料
- `GetLocalPlayer()`: 取得本地玩家
- `OnUpdateScoreAction`: 分數更新事件
- `OnReFlashEvent`: 刷新事件（`Action<double, double, int, double, double>`）
- `ReflashCoin(ReflashType type, double coin, double gem)`: 刷新金幣

---

## 4. FishSystem & FishMaintainer (魚種管理)

- `FishSystem`: 處理 Server 端的魚訊 (f1, f4, f7 cmd)、同步時間
- `FishMaintainer`: 處理 Client 端的魚體生成、游動、Object Pool

### 生成魚（Client 端測試）
```csharp
FishMaintainer.Instance.SpawnFish(
    Fish2D.enumFishType.enumType_Nemo,
    Fish2D.enumFishSubType.SpecialFish_None,
    (obj) => { obj.SetActive(true); }
);
```

### 尋找場景中的魚
```csharp
var fish = FishMaintainer.Instance?.GetFishByType(Fish2D.enumFishType.enumType_GoldenBat);
var fish = FishMaintainer.Instance?.GetFishBySID(fishSid);
```

### 程式碼位置
- `FishSystem`: `ArkGame/Scripts/Game/FishSystem/FishSystem.cs`
- `FishMaintainer`: `ArkGame/Scripts/FishAI/FishMaintainer.cs`
---

## 5. FishHunterFakeServerPacket (假封包工具)

```csharp
// 三步驟假召喚
JSON fishData = FishHunterFakeServerPacket.GetFakeFishPacket(
    Fish2D.enumFishType.enumType_GoldenBat, fakeSummonX: 0, fakeSummonY: 0, fakeSummonO: 90);
JSON spawnData = FishHunterFakeServerPacket.GetFakeSpawnFishPacket(fishData);
JSON sysData = FishHunterFakeServerPacket.GetFakeSystemCmdPacket("fish", "f1", spawnData);
GameClient.Instance.getFishSystem().onMessage(sysData);
```

位置: `FishGame/Script/CheatCodeGUI/FishHunterFakeServerPacket.cs`

---

## 6. SkillSystem 詳細說明

### 取得
```csharp
var skillSystem = GameClient.Instance.getSkillSystem();
```

### 指令註冊
```csharp
skillSystem.Register("sk_myfish_init", OnSkillInit);
skillSystem.Register("sk_myfish_hit", OnSkillHit);
skillSystem.Register("sk_myfish_end", OnSkillEnd);
```

### 發送請求
```csharp
skillSystem.SendHit(ESkillType skillType, int fishId, string bulletId = "");
skillSystem.SendRangeFishes(ESkillType skillType, int skillFishId, int[] fishIds);
skillSystem.SendSpecialFishDead(ESkillType skillType, int fishId);
```

### SkillSystem vs WeaponSystem

| 面向 | WeaponSystem (一般魚種) | SkillSystem (技能魚種) |
|-----|------------------------|----------------------|
| 觸發方式 | 被動接收 w2 封包 | 主動發送 sk_xxx 請求 |
| 指令格式 | 固定 w2 格式 | 自訂 sk_xxx 格式 |
| 適用場景 | 單純捕獲 + 演出 | 範圍攻擊、連鎖、多階段 |

---

## 7. FishHunter_EventManager 事件系統

```csharp
// 註冊
FishHunter_EventManager.Registration(FishHunter_EventType.EventType.Fish_Capture, OnFishCapture);

// 取消
FishHunter_EventManager.Cancellation(FishHunter_EventType.EventType.Fish_Capture, OnFishCapture);

// 發送
FishHunter_EventManager.SendEvent(FishHunter_EventType.EventType.LockPlayerShoot, true);

// 回呼
private void OnFishCapture(FishHunter_EventManager.EventBase e) {
    var data = e as FishHunter_EventManager.FishEventData;
    if (data == null) return;
    int fishId = (int)data.args[0];
    double totalWin = (double)data.args[1];
}
```

### 常用事件類型
- `LockPlayerShoot` / `UnlockPlayerShoot`: 砲台控制
- `SetButtonsEnable`: 按鈕啟用狀態
- `Fish_Capture`: 魚種捕獲
- `Fish_OddsUpdate`: 倍率更新
- `Fish_SpawnFeatureObj`: 生成演出物件
- `ShowDeclareBoard`: 報獎面板
- `PlaySound`: 音效

---

## 8. 跨模組橋接 — FishHunter_Agent

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
FishHunter_AgentEventManager.SendEvent(AG_AgentEvent.GameClient_getGameSystem_sendPauseGameRequest);
int seat = (int)FishHunter_AgentEventManager.getReturnValue(AG_ValueEvent.GameClient_getGameSystem_getMainPlayerSeat);
```

---

## 9. 開發建議

1. 修改 `FishSystem.cs` 的 `FishSystemCmd`（若有新指令）
2. 修改 `FishMaintainer.cs` 註冊新魚種 Prefab
3. 使用 `FishMaintainer.Instance.SpawnFish` 測試
4. 跨模組功能需在 `FishHunter_AgentEnum` 新增事件列舉，並在 `FishHunter_Agent.cs` 註冊處理函式
