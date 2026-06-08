# 專案結構 (Project Structure)

## 根目錄組織

```
FishHunter_EmptyProject/
├── Assets/
│   ├── Common/              # 共用工具與核心系統
│   ├── External_Script/     # 第三方與共用遊戲腳本
│   ├── FishHunter_Script/   # 主要遊戲邏輯 (Git Submodule)
│   ├── FishtHunter_Res/     # 遊戲資源與素材 (Git Submodule)
│   ├── Plugins/             # 第三方插件
│   ├── Editor/              # Unity Editor 擴充工具
│   ├── Scenes/              # Unity 場景
│   └── Shader/              # 自訂 Shader
└── .kiro/                   # Kiro AI 助手設定
```

## 重要目錄說明

### Assets/Common/
跨遊戲模式共用的工具：
- `FishHunter_AgentEventManager.cs`：中央事件系統，使用 Enum 事件類型，支援跨模組通訊
- `GoKit/`：舊版 Tweening 函式庫
- `Tools/`：資源管理、Soft Masking、工具集

### Assets/External_Script/
可重用的遊戲系統與第三方整合：
- `AudioManager.cs`、`AudioCtrlManager.cs`：音效管理（搭配 MasterAudio）
- `BaseFunc.cs`：工具函式（加密、檔案 I/O、字串處理、時間格式化）
- `ISingleton.cs`：泛型單例基底類別（`ArkGame` namespace，執行緒安全 lock 實作）
- `PrefabSingleton.cs`：MonoBehaviour 單例（從 Resources 載入 Prefab）
- `NGUI/`：舊版 UI 系統（逐步遷移至 Unity UI）
- `ArkSDK/`：平台 SDK 整合
- `Item/`：道具與背包系統
- `Buy/`：內購系統
- `MiniGame/`：小遊戲框架

### Assets/FishHunter_Script/ (Git Submodule)
主要遊戲實作，依遊戲模式組織：
- `FishHunter_Agent.cs`：**橋接類別**，連接 ArkGame、OceanTreasure、DinoPinBall，透過 `FishHunter_AgentEventManager` 路由事件
- `ArkGame/`：**OceanTreasure (2D 魚機)**，Namespace: `ArkGame`
  - `Scripts/GameClient.cs`：2D 魚機系統入口（單例，強耦合，具名 getter）
  - `Scripts/Game/`：核心系統 (GameSystem, PlayerSystem, FishSystem, WeaponSystem, SkillSystem)
  - `Scripts/FishAI/`：魚種行為、AI 控制器、FishMaintainer (Object Pool)
  - `Scripts/Common/`：ArkGame 共用工具、BaseSystem
- `DinoGame/DinoPinBall/`：**DinoPinBall (3D 魚機)**，Namespace: `DinoPinBall`
  - `GameClient.cs`：3D 魚機系統入口（單例，鬆耦合，泛型 `GetSystem<T>()`）
  - `BaseSystem.cs`：3D 魚機 BaseSystem（依賴注入模式）
  - `DinoPinBallM.cs`：場景入口 MonoBehaviour（`Table` 欄位為 `GameLevel` enum）
  - `Game/`：遊戲系統 (MonsterSystem, SkillSystem, WeaponSystem, EquipmentSystem)
  - `Game/Common/EventManager/`：事件管理器（`Dino_EventManager` + `EventManager`）
  - `Game/Common/TableManager/`：廳館資源管理（AssetBundle、多語系、參數）
  - `Game/TableModules/`：**廳館模組化體系**（ITableModule、BaseTableModule、各廳館 Module）
  - `Game/GameSystem/GameManager.cs`：遊戲管理器（透過 `TableModuleRegistry` 委派初始化）
  - `Award/`：獎勵與獎品系統
- `DinoGame/Lobby/`：恐龍大廳（`GameLogic_Lobby.GameLevel` enum 定義所有廳館）
- `FishGame/`：經典捕魚變體（共用 ArkGame 架構）
- `ThreePig/`：Token 代幣模式
- `Lobby/`：大廳與 Meta 系統
- `FH_Common/`：跨遊戲共用程式碼（FH_SpineActionSystem、FH_TimelineActionSystem 等）
- `CustomCameraModule/`：攝影機控制系統
- `PerformanceCtrl/`：效能優化

### Assets/FishtHunter_Res/ (Git Submodule)
遊戲素材與資源：
- `ArkGame/`：ArkGame 素材（Prefab、Sprite、動畫）
- `CrazyDino/`：DinoPinBall 素材
- `FishGame/`：經典 FishGame 素材
- `Common/`：共用素材（UI、音效、特效）
- `TableExcel/`：遊戲資料表

### Assets/Plugins/
第三方插件：
- `Spine/`：Spine 動畫 Runtime
- `MasterAudio/`：音效管理
- `Sirenix/`：Odin Inspector
- `Demigiant/`：DOTween
- `UniRx/`、`UniTask.dll`：Reactive 與 Async 函式庫
- `Effekseer/`：VFX 系統

## 架構模式

### 事件系統（三層架構）

專案存在三套獨立的事件系統，各有不同用途：

#### 1. `FishHunter_AgentEventManager`（全域跨模組橋接）
- 位置：`Assets/Common/FishHunter_AgentEventManager.cs`
- 用途：跨遊戲模式（ArkGame ↔ OceanTreasure ↔ DinoPinBall）的事件橋接
- API：`Registration(Enum, Action<EventBase>)` / `Cancellation(Enum, Action<EventBase>)`
- 事件類型前綴：`AG_`（ArkGame）、`OT_`（OceanTreasure）、`DP_`（DinoPinBall）
- 支援無回傳值事件（`AgentEvent`）與有回傳值事件（`ValueEvent`）
- `FishHunter_Agent.cs` 負責在 `Awake()` 中註冊所有事件處理函式

#### 2. `Dino_EventManager`（DinoPinBall 內部事件）
- 位置：`DinoGame/DinoPinBall/Game/Common/EventManager/Dino_EventManager.cs`
- 用途：DinoPinBall 模組內部系統間通訊
- API：`Registration(Enum, Action<EventBase>)` / `Cancellation(Enum, Action<EventBase>)`
- **注意**：此處的 `EventBase` 是 `Dino_EventManager.EventBase`（巢狀類別），非 namespace 層級的 `DinoPinBall.EventBase`
- 事件資料類別：`Dino_EventManager.DinoEventData`（繼承自 `Dino_EventManager.EventBase`）

#### 3. `IEventManager` 介面（DinoPinBall GameClient 實作）
- 位置：`DinoGame/DinoPinBall/Game/Common/EventManager/EventManager.cs`
- 用途：定義 `DinoPinBall.EventBase`（namespace 層級）與泛型 `EventData<T>` 系列
- **注意**：`DinoPinBall.EventBase` 與 `Dino_EventManager.EventBase` 是**不同的類別**
- `GameClient` 實作了 `IEventManager` 介面，但目前 `SendEvent`/`Registration`/`Cancellation` 方法皆拋出 `NotImplementedException`

### 單例模式（多種實作共存）

專案中存在多種單例模式，使用時需注意區分：

| 類別 | Namespace | 類型 | 說明 |
|------|-----------|------|------|
| `ISingleton<T>` | `ArkGame` | 泛型 class（非 interface） | 執行緒安全 `lock` 實作，`where T : class, new()` |
| `PrefabSingleton<T>` | 全域 | MonoBehaviour | 從 `Resources` 載入 Prefab，含 `Initialize()` 虛擬方法 |
| `ArkSingletonBehavior<T>` | `ArkSDK` | MonoBehaviour | ArkSDK 平台用 |
| `SingletonService` | `DinoPinBall` | 靜態 Dictionary | 服務定位器模式 |
| `Singleton<T>` | `RD2Tools` | 泛型 class | 簡易單例 |

### 系統架構
- 遊戲邏輯組織為「System」（GameSystem、PlayerSystem、WeaponSystem、SkillSystem 等）
- 每個 System 管理特定領域
- **OceanTreasure (2D)**：透過 `ArkGame.GameClient` 的具名 getter 方法存取（如 `getGameSystem()`），強耦合架構
- **DinoPinBall (3D)**：透過 `DinoPinBall.GameClient.GetSystem<T>()` 泛型方法存取，鬆耦合 + 依賴注入架構

### 資源管理
- 基於 AssetBundle 的資源載入
- `BundleCtrl`、`AtlasLoader` 用於執行時期資源載入
- Addressables 系統用於資源參照

## 命名慣例

### 檔案
- C# 腳本：PascalCase（如 `AudioManager.cs`、`FishCtrl.cs`）
- 前綴表示用途：
  - `FH_`：FishHunter 專用
  - `UI_`：UI 元件
  - `Skill`：技能系統元件
  - `Gun`：武器系統元件
  - `Dino_`：DinoPinBall 專用

### 程式碼
- 類別：PascalCase
- 方法：PascalCase
- Private 欄位：camelCase，不加前綴（不用 `_` 或 `m_`）
  - ✅ `serializedField private float colorTweenDuration;`
  - ✅ `private int currentPhase;`
  - ❌ `private int _currentPhase;`
  - ❌ `private int m_currentPhase;`
  - 注意：舊程式碼可能使用 `_` 或 `m_` 前綴，不需要主動重構，但新增程式碼一律使用 camelCase 無前綴

### 註解
- 新舊程式碼混合使用中英文註解
- 舊版/外部程式碼以中文註解為主
- 新程式碼建議使用英文註解

## Debug Log 規範

所有 `Debug.Log` / `Debug.LogWarning` / `Debug.LogError` 呼叫**必須**使用 `#if DEBUG_LOG` 條件編譯包裹，禁止裸寫。

```csharp
// ✅ 正確
#if DEBUG_LOG
Debug.Log($"[MySystem] value: {value}");
#endif

// ✅ 正確（多行）
#if DEBUG_LOG
Debug.LogWarning($"[MySystem] 未知的類型: {type}");
Debug.Log($"[MySystem] 額外資訊: {data}");
#endif

// ❌ 錯誤：裸寫 Debug.Log
Debug.Log("something happened");

// ❌ 錯誤：使用 DEBUG_MODE 包裹 Debug.Log（DEBUG_MODE 用於測試版模式功能，不用於日誌）
#if DEBUG_MODE
Debug.Log("this is wrong");
#endif
```

**規則摘要**：
- `DEBUG_LOG`：用於所有 `Debug.Log` / `Debug.LogWarning` / `Debug.LogError` 輸出
- `DEBUG_MODE`：用於測試版模式功能（CheatTool、DebugSystem、假付費等），**不用於**日誌輸出
- 新增的 Debug 工具腳本統一使用 `#if DEBUG_LOG`

## Debug 工具開發規範

### KevinReflector 使用規範

`KevinReflector.TryInvokeMethod()` 可透過反射呼叫 private 方法，**僅限 Unity Editor 開發環境使用**（`#if DEBUG_LOG` 包裹的 Debug 工具中）。

```csharp
// ✅ 正確：在 #if DEBUG_LOG 的 Debug 工具中使用 KevinReflector 呼叫 private 方法
#if DEBUG_LOG
KevinReflector.TryInvokeMethod(ctrl, "SetColorPhase", new object[] { phase }, out var _);
#endif
```

**規則**：
- `KevinReflector` 只能在 `#if DEBUG_LOG` 區塊內使用
- Debug 工具透過 `KevinReflector` 呼叫控制器的 private 方法，**不需要在控制器上暴露 public Debug 方法**
- 保持控制器的封裝性：所有內部方法維持 private，Debug 工具用反射存取
