---
inclusion: fileMatch
fileMatchPattern: ['**/DinoPinBall/**/CheatTool/**/*.cs', '**/CrazyDino/**/CheatTool/**/*.uss']
---

# DinoPinBall CheatTool V2 開發規範

> ⚠️ 本文件僅適用於 **DinoPinBall（3D 恐龍魚機）** 的 CheatTool V2。
> OceanTreasure（2D 海王魚機）的作弊工具請參閱 `#newcheat-gui-guide`（NewCheatGUI，基於 UI Toolkit，位於 `FishGame/Script/CheatCodeGUI/NewCheatGUI/`）。

本文件在編輯 DinoPinBall CheatTool 相關檔案時自動載入，提供架構規範與開發指引。

---

## 兩套作弊工具對照

| 面向 | DinoPinBall CheatTool V2（本文件） | OceanTreasure NewCheatGUI |
|------|------|------|
| 適用遊戲 | DinoPinBall（3D 恐龍） | OceanTreasure / FishGame（2D 海王） |
| 程式碼位置 | `DinoGame/DinoPinBall/Game/CheatTool/` | `FishGame/Script/CheatCodeGUI/NewCheatGUI/` |
| USS 位置 | `FishtHunter_Res/CrazyDino/Bundle/Games/CheatTool/` | `FishtHunter_Res/FishGame/CheatTool/UIToolkit/USS/` |
| 條件編譯 | `#if DEBUG_LOG` | `#if DEBUG_MODE && !REMOVE_FH_TEST` |
| 指令發送 | `CheatCmdSender`（靜態） | `GameClient.Instance.getGameSystem().SendFishTest()` |
| 怪物系統 | `MonsterSystem` / `MonsterManager` | `FishSystem` / `FishMaintainer` |
| 事件系統 | `Dino_EventManager` | `FishHunter_EventManager` |
| Namespace | `DinoPinBall.CheatTool` | `ArkGame.CheatTool` |

---

## 架構總覽

```
Assets/FishHunter_Script/DinoGame/DinoPinBall/Game/CheatTool/
├── Core/           # 核心框架（不常改動）
│   ├── ICheatModule.cs
│   ├── CheatToolPanel.cs
│   ├── CheatToolToggleButton.cs
│   └── CheatCmdSender.cs
├── Modules/        # 功能模組（ICheatModule 實作）
│   ├── StatusDashboardModule.cs   M00
│   ├── MonsterSpawnModule.cs      M01
│   ├── GameControlModule.cs       M02
│   ├── SkillWeaponModule.cs       M03
│   ├── SystemToolModule.cs        M04
│   └── MonsterDebugModule.cs      M05
└── Plugins/        # 怪物 Debug 插件（IMonsterDebugPlugin 實作）
    └── BossLichDebugPlugin.cs
```

USS 樣式表：`Assets/FishtHunter_Res/CrazyDino/Bundle/Games/CheatTool/`

---

## 核心介面

### ICheatModule

```csharp
public interface ICheatModule
{
    string TabName { get; }           // Sidebar 頁籤名稱
    int Order { get; }                // 排序（數字越小越前）
    VisualElement CreateUI();         // 建構 UI（Lazy，切換時才呼叫）
    void OnDestroy();                 // 清理
    void OnUpdate() { }              // 每幀更新（可選）
}
```

### IMonsterDebugPlugin

```csharp
public interface IMonsterDebugPlugin
{
    string MonsterName { get; }       // 卡片標題
    VisualElement CreateUI();         // 卡片內容
    void OnDestroy();                 // 清理
    void OnUpdate() { }              // 每幀更新（可選）
}
```

---

## 模組職責分配

| 編號 | 模組 | Order | 職責 |
|------|------|-------|------|
| M00 | StatusDashboardModule | -1 | 即時監控 + 控制中心（金流/FPS/封包/瞬獄殺/場景/語言/Inspector） |
| M01 | MonsterSpawnModule | 0 | 怪物生成 + 跑馬燈 + 過濾黑名單 |
| M02 | GameControlModule | 1 | 補錢 + 技能道具數量 |
| M03 | SkillWeaponModule | 2 | 武器能量 + 技能快捷（Placeholder） |
| M04 | SystemToolModule | 3 | 系統事件 + 主題資訊（Placeholder） |
| M05 | MonsterDebugModule | 4 | 通用怪物 Debug 框架（插件式） |

---

## CheatCmdSender 指令規範

所有 Server 指令集中在 `CheatCmdSender.cs`（靜態類別）。

### 新增指令的規則

1. 方法必須是 `public static`
2. 方法內建構 JSON → 呼叫 `Send("test", data)`
3. 結尾呼叫 `Log(...)` 記錄
4. 跑馬燈等不走 Server 的指令直接用 `Dino_EventManager.SendEvent`

### 已有指令一覽

| 方法 | 用途 |
|------|------|
| `AddCoin(int)` | 補錢 |
| `SecKill(int state)` | 瞬獄殺（0=正常/1=善良/2=必殺） |
| `GetJackpot()` | 下一發 JP 必中 |
| `ChangeScene()` | Server 端切換腳本 |
| `ScriptTime()` | 查詢腳本時間 |
| `SetSkillCount(string skillNo, int count)` | 設定技能道具數量（skillNo 為 int 值字串如 "30001"） |
| `SetWeaponEnergy(int id, double value)` | 設定能量武器數值 |
| `CreateDino(string serverName, int subType)` | Server 端生成怪物 |
| `TestMarquee(string serverName)` | 怪物跑馬燈（NORMAL_WIN） |
| `TestMarqueeCustom(string json)` | 自訂 JSON 跑馬燈 |
| `BossTest(JSON data)` | BOSS 通用 test 指令 |
| `RockcrystalTest(int)` | 岩晶龍 |
| `GoblinTest(int)` | 機甲哥布林 |
| `TreasureturtleTest(int)` | 財寶樹龜 |
| `IcerBerusTest(string)` | 冰原狼 |
| `UpdateServerCoin(double, int)` | 被動接收 Server Coin |

---

## USS 樣式規範

### 共用 Class 體系（定義在 StatusDashboard.uss）

所有模組統一使用以下 class，禁止在模組內用 inline style 重新定義已有的樣式：

| Class | 用途 | 視覺 |
|-------|------|------|
| `ct-dash-card` | 一般卡片 | 深色半透明背景、12px 圓角 |
| `ct-dash-card--important` | 重要卡片 | 稍亮背景、更大 padding |
| `ct-dash-row` | 水平列容器 | flex-direction: row, center |
| `ct-dash-control-row` | 控制列 | 深色背景 + 藍色微邊框 + 8px 圓角 |
| `ct-dash-control-title` | 控制列標題 | 白色粗體 24px |
| `ct-dash-control-status` | 狀態文字 | 金黃粗體 28px |
| `ct-dash-control-btn` | 操作按鈕 | 藍色背景 44px 高 |
| `ct-dash-scene-dropdown` | Dropdown | 深藍底 + 藍邊 + 金黃文字 |
| `ct-dash-fps-btn` | 數值按鈕 | 綠色邊框透明底 |
| `ct-dash-fps-btn--active` | 選中狀態 | 綠色填充 + 粗邊框 |
| `ct-dash-section-header` | 區段標題容器 | flex-direction: row |
| `ct-dash-section-title` | 大標題 | 白色粗體 22px |
| `ct-dash-section-subtitle` | 副標題 badge | 藍色半透明背景 14px |
| `ct-dash-label` | 卡片標題 | 白色 20px 置中 |
| `ct-dash-value` | 卡片數值 | 綠色粗體 36px 置中 |
| `ct-dash-subtitle` | 卡片副標題 | 灰色 16px 置中 |

### 新增樣式的規則

1. 模組專屬樣式放在對應的 `.uss` 檔案（如 `MonsterSpawnBoard.uss`）
2. 通用樣式加到 `StatusDashboard.uss`
3. 禁止在 C# 中用 inline style 覆蓋已有 USS class 的屬性（除非是動態值如顏色切換）
4. 新 class 命名格式：`ct-{模組縮寫}-{元素}--{狀態}`

---

## 怪物 Debug 插件規範

### 檔案位置

所有插件統一放在 `CheatTool/Plugins/` 目錄。

### 命名規則

- 檔案名：`{怪物PascalCase}DebugPlugin.cs`（如 `BossLichDebugPlugin.cs`）
- 類別名：`{怪物PascalCase}DebugPlugin`
- Namespace：`DinoPinBall.CheatTool.Plugins`

### 必要結構

```csharp
#if DEBUG_LOG
using DinoPinBall.CheatTool.Modules;
// ...

namespace DinoPinBall.CheatTool.Plugins
{
    public class XxxDebugPlugin : IMonsterDebugPlugin
    {
        public string MonsterName => "中文名";

        [RuntimeInitializeOnLoadMethod(RuntimeInitializeLoadType.BeforeSceneLoad)]
        private static void AutoRegister()
        {
            MonsterDebugModule.AddPlugin(new XxxDebugPlugin());
        }

        public VisualElement CreateUI() { /* ... */ }
        public void OnDestroy() { }
    }
}
#endif
```

### UI 建構規則

1. Dropdown 列使用 `ct-dash-control-row` + `ct-dash-scene-dropdown`
2. 按鈕使用 `ct-dash-control-btn`
3. 發送後呼叫 `MonsterDebugModule.MinimizePanel()`
4. 插件 UI 的 root 設定 `alignItems = Align.Stretch`

### 卡片大小

由 `MonsterDebugModule.WrapPluginCard()` 統一控制：
- `width: 48%`（每列 2 張）
- `min-height: 280px`
- 自動換行

---

## 條件編譯規範

| 符號 | 用途 |
|------|------|
| `DEBUG_LOG` | 所有 DinoPinBall CheatTool V2 程式碼（新版統一使用） |
| `DEBUG_MODE` | 舊版 CheatToolManager / DebugManager（不在新版使用） |

---

## 接入點規範

當新功能需要從遊戲系統接入 DinoPinBall CheatTool 時：

```csharp
#if DEBUG_LOG
CheatTool.Modules.StatusDashboardModule.OnClientShoot();
#endif
```

- 永遠用 `#if DEBUG_LOG` 包裹
- 使用完整 namespace 路徑（`CheatTool.Modules.XXX`）
- 放在既有 `#if DEBUG_MODE` 區塊旁邊（不要混在一起）

---

## 禁止事項

1. ❌ 在新版程式碼中引用 `DebugManager` / `DebugSystem` / `CheatToolManager`
2. ❌ 使用 `#if DEBUG_MODE` 包裹新版程式碼
3. ❌ 在 Plugin 中直接持有 `CheatToolPanel` 引用（用 `MonsterDebugModule.MinimizePanel()` 代替）
4. ❌ 在模組中 hard-code 怪物專屬邏輯（應拆成 Plugin）
5. ❌ 用 inline style 覆蓋已有 USS class 的固定屬性
6. ❌ 混用海王（OceanTreasure）的 CheatTool API（如 `FishHunterCheatTool`、`SendFishTest`）

---

## 架構模式可移植性

本架構設計為可移植模式，OceanTreasure（海王）可採用相同的設計模式建立自己的作弊工具。

### 共用的設計模式（兩套系統通用）

| 模式 | 說明 |
|------|------|
| 介面驅動 | `ICheatModule`（模組）+ `IMonsterDebugPlugin`（插件）定義契約 |
| 靜態註冊 | `AddPlugin()` 靜態方法，允許在框架建立前註冊 |
| 自動發現 | `[RuntimeInitializeOnLoadMethod]` 零改動框架 |
| 靜態指令發送 | 集中式 `CmdSender` 靜態類別，不需要持有引用 |
| 靜態 Minimize | 插件不需要知道 Panel 存在 |
| USS class 體系 | 統一視覺風格，模組/插件共用 |
| 卡片式佈局 | 框架統一包裝大小，插件只管內容 |
| 條件編譯隔離 | 整個工具在 Release 建置時完全移除 |

### 移植到海王時的對應關係

| DinoPinBall（恐龍） | OceanTreasure（海王） |
|---|---|
| `DinoPinBall.CheatTool` namespace | `ArkGame.CheatTool` namespace |
| `DinoPinBall.CheatTool.Modules` | `ArkGame.CheatTool.Modules` |
| `DinoPinBall.CheatTool.Plugins` | `ArkGame.CheatTool.Plugins` |
| `IMonsterDebugPlugin` | `IFishDebugPlugin` |
| `MonsterDebugModule` | `FishDebugModule` |
| `CheatCmdSender.BossTest(JSON)` | `FishCheatCmdSender.SkillTest(JSON)` |
| `Dino_EventManager.SendEvent` | `FishHunter_EventManager.SendEvent` |
| `MonsterManager.Instance` | `FishMaintainer.Instance` |
| `#if DEBUG_LOG` | `#if DEBUG_LOG`（統一） |
| `CheatTool/Plugins/` 目錄 | `NewCheatGUI/Plugins/` 目錄 |

### 移植步驟

1. 在 `NewCheatGUI/` 下建立 `IFishDebugPlugin` 介面（複製 `IMonsterDebugPlugin` 改名）
2. 建立 `FishDebugPanelView`（對應 `MonsterDebugModule`，負責卡片網格 + 插件包裝）
3. 建立 `FishCheatCmdSender`（對應 `CheatCmdSender`，走 `getGameSystem().SendFishTest()`）
4. 各魚種 Debug 工具建立獨立 Plugin 檔案，用 `[RuntimeInitializeOnLoadMethod]` 自動註冊
5. USS 樣式可共用 class 命名規則（`ct-` 前綴），但放在海王自己的 USS 檔案中

### 不應共用的部分

- ❌ 不要讓兩套系統引用同一個 DLL 或共用 namespace
- ❌ 不要在恐龍的 Plugin 中引用海王的 API，反之亦然
- ❌ 不要共用 `CheatToolPanel`（各自有自己的主控制器）
- ✅ 可以共用 USS 設計語言（class 命名規則、配色方案）
- ✅ 可以共用 Skill 工作流程（同一個 Skill 根據目標系統走不同路徑）
