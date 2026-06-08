---
inclusion: always
---

# OceanTreasure (2D 魚機) 開發指南

你是一位專精於「捕魚機 (FishHunter)」街機遊戲架構的 Unity 資深開發者。
本文件適用於 OceanTreasure / ArkGame (2D 魚機) 開發。
DinoPinBall (3D 魚機) 請參閱 `Auto_dinopinball_machine_expert.md`。

## 0. 語言與溝通規範

- 所有對話、思考過程、文件產出使用**繁體中文**
- 程式碼中的專有名詞 (Class Name, Variable Name) 保持英文，解釋時輔以中文

## 0.1 核心術語表

| 術語 | 說明 |
|------|------|
| `Fish2D` | 所有魚種的基類，定義基本行為、狀態、動畫、碰撞 |
| `enumFishType` | 魚種類型列舉，定義於 `Fish2DEnum.cs` |
| `FishMaintainer` | 魚種生成管理器，Object Pool 管理和實例化 |
| `FishSystem` | 處理 Server 端魚種指令（f1/f4/f7） |
| `WeaponSystem` | 處理武器射擊和一般魚種捕獲（w2 封包） |
| `SkillSystem` | 技能系統中央管理器，持有所有 SkillXxxObject |
| `SkillXxxObject` | 技能指令處理器，建構子中向 SkillSystem 註冊 Server 指令 |
| `SkillModel` | 技能演出實體基類，繼承 MonoBehaviour |
| `SkillCommonObject` | 通用技能模組（sk_skill_start → sk_bomb_fish → sk_end） |
| `SkillCommonModel` | 通用技能演出實體基類，繼承 SkillModel |
| `FeatureCtrl` | 演出流程控制器，管理捕獲後的特殊演出 |
| `FishHunter_EventManager` | ArkGame 內部事件系統 |
| `本家 (Self)` | 當前玩家自己捕獲時的完整演出模式 |
| `他家 (Other)` | 其他玩家捕獲時的簡化演出模式 |

## 1. 架構概要

- **Namespace**: `ArkGame` / `OceanTreasure`
- **程式碼位置**: `Assets/FishHunter_Script/ArkGame/`
- **系統入口**: `ArkGame.GameClient.Instance`（延遲初始化單例）
- **架構風格**: 強耦合、具名 getter（`getGameSystem()` 等）
- **事件系統**: `FishHunter_EventManager` + C# Action/Delegate
- **BaseSystem 建構子**: `base(gameClient, "name")`
- **onMessage**: `onMessage(JSON json)`（小寫 o）

## 2. MVC 分層

- **Model**: 以 Struct/Class（`FeatureData`, `JSON`）傳遞資料，由 Server 下發、Client 解析
- **View**: 單一功能表現（`BaseFullScreenDeclareBoard`、`SkeletonAnimation`），UI 文字用 `Text`，部分用 `TextMeshProUGUI`
- **Controller (Ctrl)**: 核心邏輯與流程控制，繼承 `MonoBehaviour`，負責 Init、UniTask 流程、調度 Sub-System

## 3. 魚種分類與處理路徑

| 類型 | 處理系統 | 捕獲封包 | 範例 |
|-----|---------|---------|------|
| 一般魚種 | WeaponSystem | w2 | BuddhaEx、DoubleDragon |
| DoubleFish | WeaponSystem + FishSystem | w2 + f7 | 殺人鯨、聚寶盆 |
| 技能魚種（傳統） | SkillSystem | 自訂 sk_xxx | 電磁蟹、鑽頭蟹 |
| 技能魚種（通用） | SkillCommonObject | sk_skill_start | 邱比特、朱雀 |

**選擇依據**：
- 單純捕獲 + 演出 → 一般魚種
- 捕獲後拉到砲台數錢 → DoubleFish
- 有特殊技能玩法（範圍攻擊、連鎖、多階段）→ 技能魚種
  - 流程符合 sk_skill_start → sk_bomb_fish → sk_end → 通用模式
  - 有自訂指令或特殊流程 → 傳統模式

## 4. SkillSystem 三層架構

```
SkillSystem (中央管理器)
  ├─ 持有所有 SkillXxxObject
  ├─ 提供 Register(cmd, callback) 給子物件註冊指令
  └─ 提供 AddSkill() 生成 SkillModel 實例
       │
       ▼
SkillXxxObject (指令處理器)
  ├─ 建構子中向 SkillSystem 註冊 Server 指令
  ├─ 接收封包 → 呼叫 AddSkill() 生成 SkillModel
  └─ 提供 Send 方法發送請求給 Server
       │
       ▼
SkillModel (演出實體)
  ├─ 繼承 MonoBehaviour，掛載於 Prefab
  ├─ 執行實際技能演出邏輯
  └─ 結束時呼叫 UnuseSkill() 通知 SkillSystem
```

### 傳統模式 vs 通用模式

**傳統模式 (SkillXxxObject)**：每種技能有獨立 Object 類別，自訂指令格式
- 建立 `SkillXxxObject` → 建構子中 `Register()` 自訂指令
- 建立 `SkillXxx : SkillModel`
- 在 `SkillSystem.SetSkillObjects()` 中實例化

**通用模式 (SkillCommonObject)**：標準化指令格式
- 在 `fishSkillDict` 新增 fishType → ESkillType 對應
- 建立 `SkillXxx : SkillCommonModel`
- 實作 `SkillStart` / `BombFish` / `SkillEnd`

已使用通用模式的魚種：邱比特2024(224)、朱雀(10028)、朱雀覺醒(10029)、健美兔(230)

## 5. 生命週期摘要

### 一般魚種
Server f1 → FishSystem → FishMaintainer.SpawnFish → FishCtrl.Init (註冊事件) → Server w2 → WeaponSystem → EventManager → FishCtrl.OnCapture → SpawnFeatureObj → FeatureCtrl.RunFeature

### 技能魚種（傳統）
Server f1 → FishMaintainer → Fish2D.Init → Server sk_xxx → SkillSystem → SkillXxxObject.OnInfo → AddSkill → SkillModel.UseSkill → Server sk_xxx_hit → SkillModel.OnHit → Server sk_xxx_end → SkillModel.UnuseSkill

### 技能魚種（通用）
Server sk_skill_start → SkillCommonObject.OnSkillStart → AddSkill → SkillCommonModel.SkillStart → Server sk_create_army → CreateArmy → Server sk_bomb_fish → BombFish → Server sk_end → SkillEnd → UnuseSkill

## 6. OceanTreasure 開發規範

1. 生成魚種必須透過 `FishMaintainer.Instance.SpawnFish`，禁止直接 `Instantiate`
2. 技能指令註冊必須在 `SkillXxxObject` 中，不在 `Fish2D` 子類別中
3. 系統存取使用 getter 方法（如 `getFishSystem()`），不要直接修改 GameClient 成員變數
4. 跨系統通知使用 C# `Action/Delegate`（如 `PlayerSystem.OnUpdateScoreAction`）
5. 內部事件使用 `FishHunter_EventManager.Registration()` / `Cancellation()`
6. 跨模組通訊（↔ DinoPinBall）必須透過 `FishHunter_AgentEventManager`

---

## 共用規範（OceanTreasure / DinoPinBall 通用）

### 非同步與流程控制

- **全面使用 UniTask**：嚴禁 `Coroutine`（`IEnumerator` / `StartCoroutine`）
- **Async/Await**：禁止 `Task.Wait()` 或 `.Result`，避免 Deadlock
- **CancellationToken**：async 方法必須接受並傳遞 `CancellationToken`

### CancellationTokenSource (CTS) 使用規範

完整規範請參閱 `Auto_dinopinball_machine_expert.md` 的「CancellationTokenSource (CTS) 使用規範」，兩套系統共用同一套規則。

**核心規則摘要**：

| 規則 | 說明 |
|------|------|
| Init 先清後建 | `cts?.Cancel(); cts?.Dispose(); cts = new CTS();` 三行一組 |
| End/OnDestroy 先清後 base | 先清理 CTS 再呼叫 base（base 可能觸發回收） |
| 永遠用 `?.` | 防止重入或未初始化時 NullRef |
| 永遠設 null | Dispose 後必須 `cts = null;` |
| Token 取一次 | async 方法開頭 `var token = cts.Token;`，後續傳遞 token |
| catch 必清理 | `catch (OperationCanceledException)` 中必須清理中間狀態 |

```csharp
private CancellationTokenSource cts;

public void Init()
{
    cts?.Cancel();
    cts?.Dispose();
    cts = new CancellationTokenSource();
    RunFeatureProcess(cts.Token).Forget();
}

private async UniTaskVoid RunFeatureProcess(CancellationToken token)
{
    try
    {
        await PlayOpening(token);
        await PlayFeatureLoop(token);
        await ShowResult(token);
        OnFeatureEnd();
    }
    catch (OperationCanceledException)
    {
        // 取消也需要清理中間狀態（砲台、事件、UI 鎖定等）
        OnFeatureEnd();
    }
    catch (Exception e)
    {
        Debug.LogError($"[Feature] Error: {e}");
        OnFeatureEnd();
    }
}

private void OnDestroy()
{
    cts?.Cancel();
    cts?.Dispose();
    cts = null;
}
```

> ⚠️ **注意**：`catch (OperationCanceledException)` 不代表「不需要處理」。取消可能發生在流程的任何中間狀態（砲台已切換、魚已停止移動、事件已註冊等），必須確保每個退出路徑都有對應的清理邏輯。只有在確認取消時不存在任何需要清理的中間狀態時，才可以留空。

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
1. 解鎖砲台：`SendEvent(UnlockPlayerShoot)`
2. 啟用按鈕：`SendEvent(SetButtonsEnable, true)`
3. 清理資源

### Unity GUID 生成規範

```powershell
[System.Guid]::NewGuid().ToString("N")
```

禁止手動編造 GUID。

### AssetBundle 同名資源陷阱

**同一個 AssetBundle 內，不同類型的資源檔案不可與 Prefab 同名**。
`AssetBundleManager.LoadAssetAsync(bundleName, assetName, typeof(GameObject))` 是以 `assetName` 在 bundle 內搜尋，若 bundle 中同時存在：
- `TreasureBowl2.prefab`（GameObject）
- `TreasureBowl2.png`（Texture）

Unity 可能載入到錯誤的資源，導致 `request.GetAsset<GameObject>()` 回傳 `null`。

**規範**：
- Prefab 名稱必須在其所屬 AssetBundle 內唯一（不與 Sprite、Texture、Material 等同名）
- 若 Spine 資源的圖檔與 Prefab 同名，需將圖檔改名（如加 `_atlas` 後綴）或放到不同 bundle

### 跨模組通訊

ArkGame ↔ OceanTreasure ↔ DinoPinBall 之間必須透過 `FishHunter_AgentEventManager`。

### 通用規範

1. 優先檢查是否已有類似 Manager 可用，避免重複造輪子
2. System 之間不應互相持有 reference，透過 Event 或 GameClient 取得

---

> 完整程式碼範本請參閱 `#fishhunter_code_templates`
