# 技術棧 (Technology Stack)

## 引擎與平台
- Unity 6000.3.9f1
- 目標平台：Android（支援 iOS）
- C# 9.0 / .NET 4.7.1
- 建置系統：Unity Build Pipeline + AssetBundle

## 核心函式庫與框架

### 主要依賴
- **Spine-Unity**: 2D 骨骼動畫系統，用於角色與魚種動畫
- **DOTween (Demigiant)**: Tweening 動畫函式庫
- **UniTask**: Unity 非同步 async/await 支援（`Plugins/UniTask.dll`）
- **UniRx**: Unity Reactive Extensions
- **MasterAudio**: 進階音效管理系統
- **Odin Inspector (Sirenix)**: 強化 Unity Inspector 與序列化（含 Odin Validator）

### 其他插件
- **Effekseer**: VFX 特效系統
- **JSON.cs** (`External_Script/ArkSDK/lib/JSON.cs`): 自訂 JSON 序列化
- **SQLite4Unity3d**: 本地資料庫（`Plugins/SQLite4Unity3d.dll`）
- **WebSocket**: 即時網路通訊
- **TextMesh Pro**: 進階文字渲染
- **GoKit**: Tweening 函式庫（舊版，逐步被 DOTween 取代）
- **FastScriptReload**: 熱重載插件，加速開發迭代
- **PoolManager (PathologicalGames)**: Object Pool 管理
- **Coffee UIEffect**: UI 特效擴充（UIEffect for UGUI / TextMeshPro）
- **CameraFilterPack**: 攝影機後處理濾鏡（模糊、故障、碎裂等效果）
- **KM_HighResScreenshot**: 高解析度截圖工具（Editor 用）

## 條件編譯符號 (Project Defines)

### 目前啟用（Assembly-CSharp.csproj）

| 符號 | 用途 |
|------|------|
| `DEBUG_LOG` | Debug 日誌輸出（`#if DEBUG_LOG` 包裹 `Debug.Log`） |
| `DEBUG_MODE` | 測試版模式（假付費、測試 Server、Debug UI、CheatTool 等） |
| `HTTP_WEBREQUEST` | 網路請求方式切換 |
| `FH_EMPTY_PROJECT` | 空專案模板模式（停用 FishHunter_Agent 註冊） |
| `ODIN_INSPECTOR` | 啟用 Odin Inspector 功能 |
| `ODIN_VALIDATOR` | 啟用 Odin Validator 功能 |

## 常用指令

### 測試
- Play Mode：Unity Editor Play 按鈕
- 裝置測試：透過 ADB 建置並部署至 Android 裝置
