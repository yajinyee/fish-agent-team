---
inclusion: manual
---

# 魚機新增廳館指南 (FishHunter Room Addition Guide)

本指南詳細說明如何在 FishHunter 專案中新增一個新的魚機廳館。
當使用者說「新增 XXX 館」時，請依照本指南的 Checklist 逐步完成所有必要修改。

---

## 快速參考：必要資訊

新增廳館前，需確認以下資訊：

| 項目 | 說明 | 範例 (上古神獸館) | 必須詢問 |
|------|------|------------------|----------|
| 廳館英文代號 | 小寫，用於程式碼 | `monstrum` | ✅ 必問 |
| 廳館中文名稱 | 用於 UI 顯示 | `上古神獸` | ✅ 必問 |
| 遊戲類型 | 2D海王 或 3D恐龍 | `2D海王` | ✅ 必問 |
| RoomType 數值 | Server GameID | `569` | ⚠️ 未指定則自動遞增 |
| GameLevel 數值 | 內部 ID | `24` | ⚠️ 未指定則自動遞增 |
| 等級限制欄位名 | ArgsCtrl 欄位名 | `fish_monstrum_lv_limit` | 🔄 自動生成 |
| Server 參數 Key | Server JSON key | `fish_monstrum_lv` | 🔄 自動生成 |
| 預設等級限制 | 預設值 | `10` | 🔄 預設 10 |

### 遊戲類型說明

| 類型 | 說明 | 場景 | 清單 |
|------|------|------|------|
| **2D海王 (VIP 系列)** | 傳統 2D 捕魚遊戲 | `FishHunter_Init` | `FishhunterVipTypeRoomList` |
| **3D恐龍 (Dino 系列)** | 3D 恐龍主題遊戲 | `DinoPinBallInit` | `FishhunterDinoTypeRoomList` |

### Agent 詢問範本

當使用者說「新增 XXX 館」時，Agent 應詢問：

```
請確認以下資訊：
1. 廳館英文代號（小寫）：例如 zombie、dragon
2. 廳館中文名稱：例如 殭屍館、神龍館
3. 遊戲類型：2D海王 或 3D恐龍？
4. 是否需要測試廳館 JSON？（Server 尚未實作時，可注入測試資料讓大廳顯示新廳館）

以下若未指定，我會自動處理：
- RoomType 數值：將使用下一個可用數值 (目前最大: XXX，建議: XXX)
- GameLevel 數值：將使用下一個可用數值 (目前最大: XX，建議: XX)
```

### 測試資料注入完成後提示

若使用者選擇需要測試廳館 JSON，Agent 在完成注入後**必須**提示：

```
✅ 測試資料注入完成！

已在 FishLobby2024.cs 中加入測試資料注入程式碼：
- 方法：InjectTestRoomData_XXX()
- 條件：#if DEBUG_LOG
- 排序：新廳館會顯示在大廳第一個位置

⚠️ 重要提醒：
測試完成後，請告訴我「刪除測試程式碼」或「清理 XXX 館測試資料」，
我會協助你移除測試程式碼，避免影響正式環境。
```

### ⚠️ 重複檢查機制

在開始修改程式碼前，Agent 必須先檢查是否有重複或相似的廳館：

#### 檢查步驟

1. **搜尋 RoomType 枚舉**：確認英文代號是否已存在
   ```
   搜尋：FishHunterRoomInfo.RoomType 枚舉中是否有相同或相似名稱
   ```

2. **搜尋 GameLevel 枚舉**：確認是否已存在
   ```
   搜尋：GameLogic_Lobby.GameLevel 枚舉中是否有相同或相似名稱
   ```

3. **相似名稱判斷**：若發現以下情況，必須提示使用者
   - 完全相同的名稱（如 `zombie` 已存在）
   - 名稱加數字（如 `zombie` 存在，使用者要新增 `zombie2`）
   - 名稱相似（如 `mech` 和 `mecha` 都存在）

#### 提示範本

**情況 1：完全重複**
```
⚠️ 發現重複：廳館 "zombie" 已存在於 RoomType 枚舉中！

現有廳館資訊：
- RoomType: zombie = 271
- GameLevel: 可能對應其他名稱

建議下一步：
1. 若要修改現有廳館，請說「修改 zombie 館」
2. 若要新增不同廳館，請使用其他英文代號（如 zombie_new、undead）
```

**情況 2：相似名稱（數字後綴）**
```
⚠️ 發現相似廳館：
- 現有：zombie (RoomType = 271)
- 現有：zombie2 (RoomType = 576)
- 您要新增：zombie3

這是預期的系列廳館嗎？
1. 是，繼續新增 zombie3
2. 否，我要使用其他名稱
```

**情況 3：名稱相似（可能混淆）**
```
⚠️ 發現相似名稱的廳館：
- 現有：mech (機甲館, RoomType = 526)
- 現有：mecha (機甲武士館, RoomType = 562)
- 您要新增：mechanic

這些名稱很接近，可能造成混淆。建議：
1. 確認這是不同的廳館，繼續新增
2. 使用更明確區分的名稱（如 robot、cyborg）
```

## Checklist：新增廳館步驟

### ✅ 第一階段：類型定義（必要）

#### 1.1 新增 RoomType 枚舉

**檔案**: `FishHunterLobby/Scripts/FishHunterLobbyMain.cs`

**位置**: `FishHunterRoomInfo.RoomType` 枚舉內

**注意**: RoomType 數值對應 Server 的 GameID

```csharp
public enum RoomType // GameID對照 https://docs.google.com/spreadsheets/d/...
{
    // ... 現有廳館 ...
    monstrum = 569, //魚機 上古神獸館
    // ✅ 新增：
    newroom = XXX, //魚機 新廳館名稱
}
```

#### 1.2 新增 GameLevel 枚舉

**檔案**: `FishHunter/FishHunter_Script/Lobby/GameLogic_Lobby.cs`

**位置**: `GameLevel` 枚舉內（在 `OceanTreasure` namespace 的 `GameLogic_Lobby` 類別中）

```csharp
public enum GameLevel
{
    // ... 現有廳館 ...
    monstrum = 24, //怪獸拳願
    // ✅ 新增：
    newroom = XX,
}
```

---

### ✅ 第二階段：VIP 系列清單（若為 VIP 系列廳館）

#### 2.1 新增 FishhunterVipTypeRoomList

**檔案**: `Script/Ctrl/DragonCtrl.cs`
**類別**: `FishHunterData`
**位置**: `FishhunterVipTypeRoomList` 清單欄位

在 `"ember"` 後新增：

```csharp
public List<string> FishhunterVipTypeRoomList = new List<string>
{
    "vip", "myth", "mech", "luck", "wealth",
    "mecha",       // 機甲館
    "dice",        // 骰子館
    "vegas",       // 拉斯維加斯館
    "xmas",        // 聖誕館
    "buddhaex",    // 彌勒佛EX館
    "circus",      // 馬戲團館
    "beast",       // 動物拳願
    "monstrum",    // 怪獸拳願
    "sharkbar",    // 酒保鯊魚
    "o4max",       // 海王4D max
    "fest",        // 特殊節慶館
    "sonata",      // 古典音樂館
    "ember",       // 異世界：古の焔、醒めし刻
    // ✅ 新增：
    "newroom",     // 新廳館
};
```

#### 2.2 新增 CheckIsVipSeriesLevel（若為 VIP 系列）

**檔案**: `FishHunter/FishHunter_Script/Lobby/GameLogic_Lobby.cs`
**類別**: `GameLogic_Lobby`（namespace: `OceanTreasure`）
**方法**: `CheckIsVipSeriesLevel(GameLevel _level)`

在 `GameLevel.ember` 判斷後新增：

```csharp
public bool CheckIsVipSeriesLevel(GameLevel _level)
{
    bool isVipSeries = false;
    isVipSeries = _level == GameLevel.luck
        || _level == GameLevel.myth
        || _level == GameLevel.mech
        // ... 現有判斷 ...
        || _level == GameLevel.monstrum
        // ✅ 新增：
        || _level == GameLevel.newroom;

    return isVipSeries;
}
```

---

### ✅ 第三階段：等級限制設定（必要）

#### 3.1 新增等級限制欄位

**檔案**: `Script/Ctrl/DragonCtrl.cs`
**類別**: `ArgsCtrl`
**位置**: 類別欄位區（在 `fish_monstrum_lv_limit` 附近）

```csharp
public int fish_ember_lv_limit = 10;
public int fish_zombie2_lv_limit = 10;
public int fish_monstrum_lv_limit = 10;
// ✅ 新增：
public int fish_newroom_lv_limit = 10;
```

#### 3.2 解析 Server 參數

**檔案**: `Script/Ctrl/DragonCtrl.cs`
**類別**: `ArgsCtrl`
**方法**: `SetArgsNew(JSON _JsData)`

在 `fish_monstrum_lv` 解析附近新增：

```csharp
if (_JsData.fields.ContainsKey("fish_monstrum_lv"))
    fish_monstrum_lv_limit = _JsData.ToInt("fish_monstrum_lv");
// ✅ 新增：
if (_JsData.fields.ContainsKey("fish_newroom_lv"))
    fish_newroom_lv_limit = _JsData.ToInt("fish_newroom_lv");
```

---

### ✅ 第四階段：進入邏輯（必要）

#### 4.1 新增 Join 方法

**檔案**: `Script/Ctrl/DragonCtrl.cs`
**類別**: `ArgsCtrl`
**位置**: 在 `JoinMonstrum()` 方法附近新增

```csharp
public void JoinNewRoom()
{
#if DEBUG_LOG
    Debug.Log("[tonyleo] Join NewRoom");
#endif
    DragonCtrl.sFishHunterData.sChooseRoom = "newroom";
    if (CheckCanJoinFishRoom(FishHunterRoomInfo.RoomType.newroom))
        LoadSceneManager.Instance.LoadScene("FishHunter_Init", false); // VIP 系列
        // 或 LoadSceneManager.Instance.LoadScene("DinoPinBallInit", false); // Dino 系列
    else
        PopCantJoinFish();
}
```

#### 4.2 修改 CheckCanJoinRoom（若有特殊等級判斷）

**檔案**: `Script/Ctrl/DragonCtrl.cs`
**類別**: `ArgsCtrl`
**方法**: `CheckCanJoinRoom(FishHunterRoomInfo.RoomType type)`

**注意**: 大多數 VIP 系列廳館使用 `default` 分支，不需額外新增 case

```csharp
switch (type)
{
    // ... 現有 case ...
    case FishHunterRoomInfo.RoomType.myth:
    case FishHunterRoomInfo.RoomType.mech:
    case FishHunterRoomInfo.RoomType.luck:
    case FishHunterRoomInfo.RoomType.dice:
    case FishHunterRoomInfo.RoomType.sharkbar:
    case FishHunterRoomInfo.RoomType.monstrum:
    // ✅ 若需特殊判斷才新增，否則走 default：
    // case FishHunterRoomInfo.RoomType.newroom:
    default:
        return (isOpenAll || isVipEnough) && isLvEnough;
}
```

#### 4.3 修改 JoinFishHunterRoom（通用進入方法）

**檔案**: `Script/Ctrl/DragonCtrl.cs`
**類別**: `ArgsCtrl`
**方法**: `JoinFishHunterRoom(FishHunterRoomInfo.RoomType roomType)`

**注意**: 此方法已使用 `isFishhunterVipRoom` 和 `isDinoRoom` 判斷，若已正確設定 VipTypeRoomList 或 DinoTypeRoomList，則不需修改此方法

```csharp
public void JoinFishHunterRoom(FishHunterRoomInfo.RoomType roomType)
{
    DragonCtrl.sFishHunterData.sChooseRoom = roomType.ToString();
    if (CheckCanJoinFishRoom(roomType))
    {
        if (DragonCtrl.sFishHunterData.isFishhunterVipRoom)
            LoadSceneManager.Instance.LoadScene("FishHunter_Init", false);
        else if (DragonCtrl.sFishHunterData.isDinoRoom)
            LoadSceneManager.Instance.LoadScene("DinoPinBallInit", false);
    }
    else
        PopCantJoinFish();
}
```

---

### ✅ 第五階段：外部跳轉支援（必要）

#### 5.1 GoActionCtrl 跳轉

**檔案**: `Script/Misc/GoActionCtrl.cs`
**類別**: `GoActionCtrl`
**方法**: `DoAction()`

在 `case "FISH_MACHINE_MONSTRUM":` 附近新增：

```csharp
case "FISH_MACHINE_MONSTRUM":
    ArgsCtrl.Instance.JoinMonstrum();
    isSuccess = true;
    break;
// ✅ 新增：
case "FISH_MACHINE_NEWROOM":
    ArgsCtrl.Instance.JoinNewRoom();
    isSuccess = true;
    break;
```

#### 5.2 GoToManager 跳轉

**檔案**: `Script/EveneSystem/GoTo/GoToManager.cs`
**類別**: `GoToManager`
**方法**: `GotoFishRoom(string fishRoom)`

在 `case "monstrum":` 附近新增：

```csharp
case "monstrum":
    ArgsCtrl.Instance.JoinMonstrum();
    break;
// ✅ 新增：
case "newroom":
    ArgsCtrl.Instance.JoinNewRoom();
    break;
```

#### 5.3 EventSystemDefine 跳轉

**檔案**: `Script/EveneSystem/Common/EventSystemDefine.cs`
**類別**: `EventSystemDefine`
**方法**: `DoGoAction(GoActionData _Data)` 內的 `case "fish":` 區塊

在 `_Data.sType.Equals("monstrum")` 判斷後新增：

```csharp
else if (_Data.sType.Equals("monstrum"))
{
    ArgsCtrl.Instance.JoinMonstrum();
}
// ✅ 新增：
else if (_Data.sType.Equals("newroom"))
{
    ArgsCtrl.Instance.JoinNewRoom();
}
```

#### 5.4 GameLogic_Lobby 換廳跳轉

**檔案**: `FishHunter/FishHunter_Script/Lobby/GameLogic_Lobby.cs`
**類別**: `GameLogic_Lobby`
**方法**: `ChangeRoom()` 或包含 `case GameLevel.monstrum:` 的 switch 區塊

在 `case GameLevel.monstrum:` 附近新增：

```csharp
case GameLevel.monstrum:
    ArgsCtrl.Instance.JoinMonstrum();
    break;
// ✅ 新增：
case GameLevel.newroom:
    ArgsCtrl.Instance.JoinNewRoom();
    break;
```

---

### ✅ 第六階段：進場背景設定（必要）

#### 6.1 EnterFishHunterCheck 背景設定

**檔案**: `FishHunterLobby/Scripts/EnterFishHunterCheck.cs`
**類別**: `EnterFishHunterCheck`
**方法**: `InitializeBackgroundSettings()`

在 `case "monstrum":` 附近新增：

```csharp
case "monstrum":
    var monstrum = GetRandomVariant("monstrum");
    if (monstrum != null) monstrum.ActivateRoomObject();
    break;
// ✅ 新增：
case "newroom":
    var newroom = GetRandomVariant("newroom");
    if (newroom != null) newroom.ActivateRoomObject();
    break;
```

#### 6.2 FishHunter_Init 場景設定（Unity Editor 操作）

**檔案**: `FishHunter_Init.unity` 場景

**操作**: 
1. 在場景中找到 `EnterFishHunterCheck` 組件
2. 在 `m_RoomActivationList` 清單中新增項目
3. 設定 `RoomType` 為 `"newroom"`
4. 設定對應的背景 GameObject

---

### ✅ 第七階段：鎖定設定面板（必要）

#### 7.1 FH_LockSettingPanel 判斷

**檔案**: `FishHunter/FishHunter_Script/ArkGame/Scripts/Game/GameSystem/FH_LockSettingPanel.cs`
**類別**: `FH_LockSettingPanel`
**方法**: 包含 `case GameLogic_Lobby.GameLevel.monstrum:` 的方法

在 `case GameLogic_Lobby.GameLevel.monstrum:` 附近新增：

```csharp
case GameLogic_Lobby.GameLevel.dice:
case GameLogic_Lobby.GameLevel.sharkbar:
case GameLogic_Lobby.GameLevel.monstrum:
// ✅ 新增：
case GameLogic_Lobby.GameLevel.newroom:
    lockSettingBtnTF.localScale = Vector3.one;
    break;
```

---

### ✅ 第八階段：大廳等級鎖定判斷（必要）

#### 8.1 FishLobbyMenu2024 等級檢查

**檔案**: `FishHunter/NewFishHunterLobby/FishLobbyMenu2024.cs`
**類別**: `FishLobbyMenu2024`
**方法**: 包含 `m_RoomType == FishHunterRoomInfo.RoomType.monstrum` 判斷的方法

在 `RoomType.monstrum` 判斷附近新增：

```csharp
else if(_RoomItem.RoomInfo.m_RoomType == FishHunterRoomInfo.RoomType.monstrum)
{
    if (UserData.Instance.m_iLv < ArgsCtrl.Instance.fish_monstrum_lv_limit)
        bActiveLock = true;
}
// ✅ 新增：
else if(_RoomItem.RoomInfo.m_RoomType == FishHunterRoomInfo.RoomType.newroom)
{
    if (UserData.Instance.m_iLv < ArgsCtrl.Instance.fish_newroom_lv_limit)
        bActiveLock = true;
}
```

---

### ✅ 第九階段：多語系設定（Unity Editor 操作）

#### 9.1 FH_MarqueeMultiLanguage 設定

**檔案**: `FishHunter/FishHunter_Script/FishGame/Script/TableManager/FH_MarqueeMultiLanguage.asset`

**操作**: 在 Unity Editor 中開啟，於 `roomNames` 清單新增：

```yaml
- roomType: newroom
  cht: 新廳館名稱
  chs: 新厅馆名称
  en: New Room
  # 其他語系...
```

---

### ✅ 第十階段：大廳 UI Prefab（分步驟執行）

本階段分為兩個步驟，需要使用者在中間刷新 Unity Editor：

---

#### 10.1 步驟一：複製 Room Prefab 並生成新 GUID

**來源檔案**：
- `FishHunterLobby/U_Prefab/Room_Monstrum.prefab`
- `FishHunterLobby/U_Prefab/Room_Monstrum.prefab.meta`

**Agent 操作步驟**：
1. 使用 PowerShell 生成標準 UUID v4
2. 複製 `Room_Monstrum.prefab` 為 `Room_XXX.prefab`（XXX = 新廳館英文代號，首字母大寫）
3. 複製 `Room_Monstrum.prefab.meta` 為 `Room_XXX.prefab.meta`
4. 替換 `.meta` 檔案中的 `guid` 值為新生成的 GUID

**⚠️ GUID 生成規範（強制）**

Agent **必須**執行以下 PowerShell 命令生成 GUID：

```powershell
[System.Guid]::NewGuid().ToString("N")
```

**輸出範例**：
```
85737ba950764d5d8144437e7ad09650
```

**❌ 禁止的做法**：
- 手動編造 GUID（如 `a1b2c3d4e5f6...`）
- 使用簡單隨機數拼湊
- 複製其他檔案的 GUID

**原因**：手動編造的 GUID 不符合 RFC 4122 UUID v4 標準，碰撞機率高，可能導致 Unity 資源引用錯誤。

**新 .meta 檔案範例**：
```yaml
fileFormatVersion: 2
guid: 85737ba950764d5d8144437e7ad09650
PrefabImporter:
  externalObjects: {}
  userData: 
  assetBundleName: 
  assetBundleVariant: 
```

詳細規範請參閱：[捕魚機開發指南 - 5.4 Unity GUID 生成規範](fish-machine-expert-zh.md)

**步驟一完成後，Agent 必須提示使用者**：

```
✅ Room Prefab 複製完成！

已建立：
- Room_XXX.prefab（複製自 Room_Monstrum）
- Room_XXX.prefab.meta（新 GUID: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx）

⚠️ 請執行以下操作後回覆我：
1. 切換到 Unity Editor
2. 在 Project 視窗中，對 FishHunterLobby/U_Prefab 資料夾按右鍵
3. 選擇「Reimport」或按 Ctrl+R 刷新
4. 確認 Room_XXX.prefab 已正確顯示在資料夾中
5. 回覆「已刷新」或「OK」，我將繼續下一步

（若 Prefab 未顯示或有錯誤，請告訴我）
```

**⚠️ Agent 必須等待使用者確認後，才能執行步驟二**

---

#### 10.2 步驟二：更新 Canvas_FishArea.prefab（等待使用者確認後執行）

**前置條件**：使用者已確認 Unity Editor 刷新完成

**檔案**: `Assets/Res/Bundle/FishHunterLobby/FishChangeScene/Resources/Canvas_FishArea.prefab`

**操作**：直接修改 Prefab 的 YAML 檔案，在 `roomUnits:` 陣列末尾（`m_RoomItem: []` 之前）新增項目

**⚠️ 重要：YAML 格式說明**

Canvas_FishArea.prefab 中的 `roomUnits` 陣列格式如下：

```yaml
  roomUnits:
  - type: 576
    roomObj: {fileID: 1628898869790806, guid: 4d372451a9bfa204eaaf17c25b4bfd19, type: 3}
  - type: 560
    roomObj: {fileID: 1438299436756400, guid: 52250ea6d594796438348fbeddfa68e9, type: 3}
  - type: 547
    roomObj: {fileID: 1438299436756400, guid: 69870d133f5806144b9bb90f231c7bae, type: 3}
  - type: 577
    roomObj: {fileID: 1438299436756400, guid: 96b15e4160f945fbade63309ea1e8fc1, type: 3}
  m_RoomItem: []
```

**Agent 修改步驟**：

1. **搜尋定位**：使用 grepSearch 搜尋 `m_RoomItem: \[\]` 找到插入位置
2. **字串替換**：在 `m_RoomItem: []` 之前插入新的 roomUnit 項目

**strReplace 範例**：

```
oldStr:
  - type: 547
    roomObj: {fileID: 1438299436756400, guid: 69870d133f5806144b9bb90f231c7bae, type: 3}
  m_RoomItem: []

newStr:
  - type: 547
    roomObj: {fileID: 1438299436756400, guid: 69870d133f5806144b9bb90f231c7bae, type: 3}
  - type: 577
    roomObj: {fileID: 1438299436756400, guid: 96b15e4160f945fbade63309ea1e8fc1, type: 3}
  m_RoomItem: []
```

**欄位說明**：

| 欄位 | 值 | 說明 |
|------|-----|------|
| `type` | `577` | 新廳館的 RoomType 數值（對應 `FishHunterRoomInfo.RoomType` 枚舉） |
| `fileID` | `1438299436756400` | Prefab 內部根 GameObject 的 ID（複製的 Prefab 保持相同） |
| `guid` | 步驟一生成的 GUID | 識別 Room_XXX.prefab 檔案 |
| `type: 3` | `3` | Unity 資源類型（3 = Prefab） |

**⚠️ 注意事項**：
- YAML 格式對縮排敏感，必須保持一致的空格縮排
- `roomObj` 必須在同一行，格式為 `{fileID: xxx, guid: xxx, type: 3}`
- 新項目必須插入在 `m_RoomItem: []` 之前

**步驟二完成後，Agent 必須提示使用者**：

```
✅ Canvas_FishArea.prefab 更新完成！

已將 Room_XXX 加入 roomUnits 清單：
- RoomType: XXX (數值)
- GUID: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

⚠️ 請在 Unity Editor 中完成以下步驟：
1. 重新整理 Project 視窗（右鍵 > Reimport 或 Ctrl+R）
2. 開啟 Room_XXX.prefab 修改素材和設定（Icon、背景等）
3. 開啟 Canvas_FishArea.prefab 確認 Room Units 正確顯示新廳館

其他需要在 Unity Editor 完成的項目：
- FH_MarqueeMultiLanguage.asset：新增多語系設定
- FishHunter_Init.unity：新增背景物件到 m_RoomActivationList
```

---

#### 10.3 流程總覽

```
┌─────────────────────────────────────────────────────────────┐
│                    第十階段執行流程                           │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  步驟一：複製 Prefab                                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 1. Agent 生成 GUID（PowerShell）                     │   │
│  │ 2. Agent 複製 Room_Monstrum.prefab → Room_XXX.prefab │   │
│  │ 3. Agent 建立 .meta 檔案（使用新 GUID）               │   │
│  │ 4. Agent 提示使用者刷新 Unity Editor                  │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                  │
│  ⏸️ 等待使用者確認：「已刷新」或「OK」                        │
│                          ↓                                  │
│  步驟二：更新 Canvas_FishArea.prefab                         │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 1. Agent 搜尋 Canvas_FishArea.prefab 中的            │   │
│  │    m_RoomItem: [] 位置                               │   │
│  │ 2. Agent 使用 strReplace 在其前方插入新 roomUnit     │   │
│  │ 3. Agent 提示使用者完成後續 Unity Editor 操作         │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

#### 10.4 Agent 執行範例

**步驟一：生成 GUID 並複製 Prefab**

```powershell
# 1. 生成 GUID
[System.Guid]::NewGuid().ToString("N")
# 輸出：96b15e4160f945fbade63309ea1e8fc1

# 2. 複製 Prefab
Copy-Item "Assets/FishHunterLobby/U_Prefab/Room_Monstrum.prefab" "Assets/FishHunterLobby/U_Prefab/Room_Gundam.prefab"

# 3. 建立 .meta 檔案（使用 fsWrite）
```

**步驟二：修改 Canvas_FishArea.prefab**

首先搜尋定位：
```
grepSearch: "m_RoomItem: \[\]" in Canvas_FishArea.prefab
```

然後使用 strReplace：
```
oldStr:
  - type: 547
    roomObj: {fileID: 1438299436756400, guid: 69870d133f5806144b9bb90f231c7bae, type: 3}
  m_RoomItem: []

newStr:
  - type: 547
    roomObj: {fileID: 1438299436756400, guid: 69870d133f5806144b9bb90f231c7bae, type: 3}
  - type: 577
    roomObj: {fileID: 1438299436756400, guid: 96b15e4160f945fbade63309ea1e8fc1, type: 3}
  m_RoomItem: []
```

---

#### 10.5 Agent 對話範例

**Agent（步驟一完成後）**：
```
✅ Room Prefab 複製完成！

已建立：
- Room_Gundam.prefab（複製自 Room_Monstrum）
- Room_Gundam.prefab.meta（新 GUID: 96b15e4160f945fbade63309ea1e8fc1）

⚠️ 請執行以下操作後回覆我：
1. 切換到 Unity Editor
2. 在 Project 視窗中，對 FishHunterLobby/U_Prefab 資料夾按右鍵
3. 選擇「Reimport」或按 Ctrl+R 刷新
4. 確認 Room_Gundam.prefab 已正確顯示在資料夾中
5. 回覆「已刷新」或「OK」，我將繼續下一步
```

**使用者**：
```
OK
```

**Agent（步驟二執行後）**：
```
✅ Canvas_FishArea.prefab 更新完成！

已將 Room_Gundam 加入 roomUnits 清單：
- RoomType: gundam (577)
- GUID: 96b15e4160f945fbade63309ea1e8fc1

⚠️ 請在 Unity Editor 中完成以下步驟：
1. 重新整理 Project 視窗
2. 開啟 Room_Gundam.prefab 修改素材和設定
3. 確認 Canvas_FishArea.prefab 中的 Room Units 正確顯示

其他需要在 Unity Editor 完成的項目：
- FH_MarqueeMultiLanguage.asset：新增多語系設定
- FishHunter_Init.unity：新增背景物件到 m_RoomActivationList
```

---

### ✅ 第十一階段：測試工具（建議）

#### 11.1 SimpleTutorial 測試清單

**檔案**: `Script/Tutorial/SimpleTutorial.cs`
**類別**: `SimpleTutorial`
**位置**: `roomList` 字典欄位

在 `上古神獸` 項目後新增：

```csharp
Dictionary<string, FishHunterRoomInfo.RoomType> roomList = new Dictionary<string, FishHunterRoomInfo.RoomType>
{
    { "神話館", FishHunterRoomInfo.RoomType.myth },
    // ... 現有項目 ...
    { "上古神獸", FishHunterRoomInfo.RoomType.monstrum },
    // ✅ 新增：
    { "新廳館", FishHunterRoomInfo.RoomType.newroom },
};
```

---

## 完整修改清單總覽

| 階段 | 檔案 | 修改類型 | 必要性 |
|------|------|----------|--------|
| 1.1 | FishHunterLobbyMain.cs | 新增 RoomType 枚舉值 | ✅ 必要 |
| 1.2 | GameLogic_Lobby.cs | 新增 GameLevel 枚舉值 | ✅ 必要 |
| 2.1 | DragonCtrl.cs | 新增 VipTypeRoomList 項目 | ⚠️ 2D海王 |
| 2.2 | GameLogic_Lobby.cs | 修改 CheckIsVipSeriesLevel | ⚠️ 2D海王 |
| 3.1 | DragonCtrl.cs (ArgsCtrl) | 新增等級限制欄位 | ✅ 必要 |
| 3.2 | DragonCtrl.cs (SetArgsNew) | 新增 Server 參數解析 | ✅ 必要 |
| 4.1 | DragonCtrl.cs | 新增 JoinXXX 方法 | ✅ 必要 |
| 4.2 | DragonCtrl.cs | 修改 CheckCanJoinRoom | ⚠️ 特殊判斷 |
| 5.1 | GoActionCtrl.cs | 新增 switch case | ✅ 必要 |
| 5.2 | GoToManager.cs | 新增 switch case | ✅ 必要 |
| 5.3 | EventSystemDefine.cs | 新增 else if 判斷 | ✅ 必要 |
| 5.4 | GameLogic_Lobby.cs | 新增換廳 switch case | ✅ 必要 |
| 6.1 | EnterFishHunterCheck.cs | 新增背景 switch case | ✅ 必要 |
| 6.2 | FishHunter_Init.unity | Unity Editor 編輯 | ✅ 必要 |
| 7.1 | FH_LockSettingPanel.cs | 新增 switch case | ✅ 必要 |
| 8.1 | FishLobbyMenu2024.cs | 新增等級檢查 else if | ✅ 必要 |
| 9.1 | FH_MarqueeMultiLanguage.asset | Unity Editor 編輯 | ✅ 必要 |
| 10.1 | Room_XXX.prefab + .meta | 複製並生成新 GUID | ✅ 可自動（步驟一） |
| 10.2 | Canvas_FishArea.prefab | 修改 YAML roomUnits | ✅ 可自動（步驟二，需等待使用者刷新） |
| 11.1 | SimpleTutorial.cs | 新增字典項目 | 📝 建議 |

---

## 注意事項

### ⚠️ 不要自動生成的項目

以下項目需要美術資源或 Unity Editor 操作，Agent 不應自動生成：

1. **FH_MarqueeMultiLanguage.asset** - 需要 Unity Editor 編輯
2. **FishHunter_Init.unity 場景** - 需要 Unity Editor 編輯
3. **AssetBundle 設定** - 需要 Unity Editor 操作
4. **背景圖片、Icon 素材** - 需要美術製作
5. **Room Prefab 內部素材修改** - 需要 Unity Editor 編輯

### ✅ Agent 可自動完成的項目

1. 所有 `.cs` 檔案的程式碼修改
2. 枚舉值新增
3. 方法新增
4. switch case / else if 新增
5. 欄位新增
6. 清單項目新增
7. **Room Prefab 複製**（步驟一：複製 .prefab 和 .meta，生成新 GUID）
8. **Canvas_FishArea.prefab 的 roomUnits 陣列**（步驟二：等待使用者刷新 Unity Editor 後執行）

### ⚠️ 第十階段分步驟執行說明

第十階段需要分兩步驟執行，中間需要使用者刷新 Unity Editor：

| 步驟 | 操作 | 等待使用者 |
|------|------|-----------|
| 步驟一 | 複製 Room Prefab + 生成新 GUID | ✅ 需要等待使用者刷新 Unity Editor |
| 步驟二 | 更新 Canvas_FishArea.prefab | ❌ 完成後提示使用者 |

**原因**：Unity 需要重新載入 Prefab 檔案才能正確識別新的 GUID，若在刷新前修改 Canvas_FishArea.prefab，可能導致引用錯誤。

### ⚠️ 重要差異：2D海王 vs 3D恐龍

| 項目 | 2D海王 (VIP 系列) | 3D恐龍 (Dino 系列) |
|------|-------------------|-------------------|
| 場景 | `FishHunter_Init` | `DinoPinBallInit` |
| 清單 | `FishhunterVipTypeRoomList` | `FishhunterDinoTypeRoomList` |
| 背景設定 | `EnterFishHunterCheck.cs` | `EnterFishHunterCheckDino.cs` |
| CheckIsVipSeriesLevel | ✅ 需要新增 | ❌ 不需要 |

---

## 使用範例

當使用者說：「新增殭屍館」

Agent 應：
1. 詢問確認：
   - 廳館英文代號？
   - 廳館中文名稱？
   - 遊戲類型：2D海王 或 3D恐龍？
   - 提示 RoomType/GameLevel 數值（若未指定會自動遞增）
2. 依照 Checklist 逐步修改所有 `.cs` 檔案
3. **第十階段步驟一**：複製 Prefab 後，提示使用者刷新 Unity Editor
4. **等待使用者確認**後，執行**第十階段步驟二**
5. 列出需要手動在 Unity Editor 完成的項目

### 對話範例

**範例 1：正常新增**
```
使用者：新增殭屍館

Agent：請確認以下資訊：
1. 廳館英文代號（小寫）：
2. 廳館中文名稱：
3. 遊戲類型：2D海王 或 3D恐龍？

以下若未指定，我會自動處理：
- RoomType 數值：將使用下一個可用數值 (目前最大: 576，建議: 577)
- GameLevel 數值：將使用下一個可用數值 (目前最大: 33，建議: 34)

使用者：英文 undead，中文 亡靈館，2D海王

Agent：好的，我將新增以下廳館：
- 英文代號：undead
- 中文名稱：亡靈館
- 類型：2D海王 (VIP 系列)
- RoomType：577
- GameLevel：34

開始修改程式碼...

[完成所有程式碼修改後]

✅ 程式碼修改完成！

已自動建立：
- Room_Undead.prefab（複製自 Room_Monstrum）
- Room_Undead.prefab.meta（新 GUID: a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6）
- 已更新 Canvas_FishArea.prefab 的 Room Units 清單

⚠️ 請在 Unity Editor 中完成以下步驟：
1. 重新整理 Project 視窗（右鍵 > Reimport）
2. 開啟 Room_Undead.prefab 修改素材和設定
3. 確認 Canvas_FishArea.prefab 中的 Room Units 正確顯示

其他需要在 Unity Editor 完成的項目：
- FH_MarqueeMultiLanguage.asset：新增多語系設定
- FishHunter_Init.unity：新增背景物件到 m_RoomActivationList
```

**範例 2：發現重複**
```
使用者：新增殭屍館，英文 zombie

Agent：⚠️ 發現重複：廳館 "zombie" 已存在於 RoomType 枚舉中！

現有廳館資訊：
- RoomType: zombie = 271

建議下一步：
1. 若要修改現有廳館，請說「修改 zombie 館」
2. 若要新增不同廳館，請使用其他英文代號（如 zombie3、undead、ghoul）

使用者：那用 zombie3

Agent：⚠️ 發現相似廳館：
- 現有：zombie (RoomType = 271)
- 現有：zombie2 (RoomType = 576)
- 您要新增：zombie3

這是預期的系列廳館嗎？請確認繼續。

使用者：是的，繼續

Agent：好的，我將新增 zombie3 廳館...
```

---

## 相關檔案路徑快速參考

```
FishHunterLobby/Scripts/
├── FishHunterLobbyMain.cs      # RoomType 枚舉
├── EnterFishHunterCheck.cs     # 進場背景設定 (2D海王)
├── EnterFishHunterCheckDino.cs # 進場背景設定 (3D恐龍)
└── FishHunterRoomMenuItem.cs   # Room Prefab 組件

FishHunterLobby/U_Prefab/
├── Room_Monstrum.prefab        # 預設複製來源
└── Room_Monstrum.prefab.meta   # GUID: ed77d16b8941f7245bb70e4a26bedf26

FishHunter/FishHunter_Script/
├── Lobby/GameLogic_Lobby.cs    # GameLevel 枚舉、CheckIsVipSeriesLevel
└── ArkGame/Scripts/Game/GameSystem/FH_LockSettingPanel.cs

FishHunter/NewFishHunterLobby/
└── FishLobbyMenu2024.cs        # 大廳等級鎖定判斷、RoomUnit 結構定義

Res/Bundle/FishHunterLobby/FishChangeScene/Resources/
└── Canvas_FishArea.prefab      # Room Units 清單（FishLobbyMenu2024 組件）

Script/Ctrl/
└── DragonCtrl.cs               # ArgsCtrl、JoinXXX、CheckCanJoinRoom、VipTypeRoomList

Script/Misc/
└── GoActionCtrl.cs             # 外部跳轉

Script/EveneSystem/
├── GoTo/GoToManager.cs         # 跳轉管理
└── Common/EventSystemDefine.cs # 事件跳轉

Script/Tutorial/
└── SimpleTutorial.cs           # 測試工具
```

---

## ✅ 第十二階段：測試資料注入（Server 未實作前）

當 Server 尚未實作新廳館的 API 時，可以在 Client 端注入測試資料，讓大廳 UI 顯示新廳館以便測試。

### 12.1 測試資料注入位置

**檔案**: `FishHunter/NewFishHunterLobby/FishLobby2024.cs`
**方法**: `ParserRoomData(JSON data, bool _bInitSet = false)`

### 12.2 注入程式碼範本

在 `ParserRoomData` 方法開頭加入：

```csharp
private void ParserRoomData(JSON data, bool _bInitSet = false)
{
    if (data == null)
    {
        Debug.LogError("[FishHunterLobby] SetRoomData --> data = null!!");
    }

#if DEBUG_LOG
    // ========== 測試資料注入：newroom 廳館（比照 monstrum 設定，排序第一）==========
    data = InjectTestRoomData_NewRoom(data);
#endif

    Debug.LogError("[FishHunterLobby] ParserRoomData data " + data.serialized);
    // ... 原本的程式碼 ...
}
```

### 12.3 注入方法範本（比照 monstrum）

在 `FishLobby2024.cs` 檔案末尾（class 結束前）加入：

```csharp
#if DEBUG_LOG
    /// <summary>
    /// 注入 newroom 測試廳館資料（比照 monstrum 設定，排序第一）
    /// Server 尚未實作前，用於 Client 端測試
    /// </summary>
    private JSON InjectTestRoomData_NewRoom(JSON data)
    {
        if (data == null) return data;

        // 檢查是否已存在，避免重複注入
        if (data.ContainsKey("room_sort_list"))
        {
            List<string> existingSortList = data.ToList<string>("room_sort_list");
            if (existingSortList.Contains("newroom"))
            {
                Debug.Log("[TestData] newroom already exists, skip injection");
                return data;
            }
        }

        // 建立 bet_info（比照 monstrum）
        List<JSON> betInfoList = new List<JSON>();
        int[] levels = { 80, 90, 100, 125 };
        int[] bets = { 200000, 500000, 1000000, 2000000 };
        int[] vipLevels = { 2, 2, 2, 3 };
        for (int i = 0; i < 4; i++)
        {
            JSON betInfo = new JSON();
            betInfo["limited_level"] = levels[i];
            betInfo["bet_value"] = bets[i];
            betInfo["limited_vip_level"] = vipLevels[i];
            betInfo["jp_limited_vip_level"] = -1;
            betInfoList.Add(betInfo);
        }

        // 建立 card_drop_rate_up_hint
        JSON cardDropHint = new JSON();
        cardDropHint["threshold"] = 0;
        cardDropHint["enable"] = false;

        // 建立 collect_token_info（比照 monstrum）
        JSON collectTokenInfo = new JSON();
        collectTokenInfo["start_time"] = 1771948800;
        collectTokenInfo["enable"] = true;
        collectTokenInfo["event_type"] = "ThePigHouse";
        collectTokenInfo["end_time"] = 1772121540;

        // 建立 newroom 的 game_list 項目（比照 monstrum）
        JSON newroomGame = new JSON();
        newroomGame["category"] = "slot game";
        newroomGame["avalible"] = true;
        newroomGame["card_drop_rate_up_hint"] = cardDropHint;
        newroomGame["room_name"] = "newroom";
        newroomGame["coin_limit"] = new int[] { -1, -1 };
        newroomGame["vip_need"] = 1;
        newroomGame["bet_info"] = betInfoList;
        newroomGame["collect_token_info"] = collectTokenInfo;
        newroomGame["is_open_all"] = false;
        newroomGame["game_id"] = "igsOceanKing2";

        // 插入到 game_list 第一個位置
        if (data.ContainsKey("game_list"))
        {
            List<JSON> gameList = data.ToList<JSON>("game_list");
            gameList.Insert(0, newroomGame);
            data["game_list"] = gameList;
        }

        // 插入到 room_sort_list 第一個位置
        if (data.ContainsKey("room_sort_list"))
        {
            List<string> sortList = data.ToList<string>("room_sort_list");
            sortList.Insert(0, "newroom");
            data["room_sort_list"] = sortList;
        }

        Debug.Log("[TestData] Injected newroom room data at first position (monstrum settings)");

        return data;
    }
#endif
```

### 12.4 測試資料設定說明（比照 monstrum）

| 欄位 | 值 | 說明 |
|------|-----|------|
| `vip_need` | `1` | 需要 VIP 1 |
| `is_open_all` | `false` | 非全開放 |
| `limited_level` | `80, 90, 100, 125` | 各 bet 等級限制 |
| `bet_value` | `200000, 500000, 1000000, 2000000` | 各 bet 金額 |
| `limited_vip_level` | `2, 2, 2, 3` | 各 bet VIP 限制 |
| `jp_limited_vip_level` | `-1` | Jackpot VIP 限制（-1 = 無限制） |
| `collect_token_info.enable` | `true` | 啟用代幣收集 |

### 12.5 JSON 資料結構參考

Server 回傳的 JSON 資料結構中，新廳館需要加入以下位置：

| 位置 | 說明 | 是否需要注入 |
|------|------|-------------|
| `game_list` | 遊戲清單（含 bet_info、collect_token_info） | ✅ 需要 |
| `room_list` | 房間清單 | ❌ 不需要（monstrum 也沒有在 room_list） |
| `room_sort_list` | 房間排序清單 | ✅ 需要（插入第一個位置方便測試） |

---

## ⚠️ 測試完成後清理流程（重要）

### 必須刪除的測試程式碼

當 Server 實作完成後，**必須**刪除以下測試程式碼：

#### 1. 刪除注入呼叫

**檔案**: `FishHunter/NewFishHunterLobby/FishLobby2024.cs`
**位置**: `ParserRoomData` 方法開頭

刪除以下區塊：
```csharp
#if DEBUG_LOG
    // ========== 測試資料注入：newroom 廳館（比照 monstrum 設定，排序第一）==========
    data = InjectTestRoomData_NewRoom(data);
#endif
```

#### 2. 刪除注入方法

**檔案**: `FishHunter/NewFishHunterLobby/FishLobby2024.cs`
**位置**: 檔案末尾

刪除整個 `#if DEBUG_LOG ... #endif` 區塊，包含 `InjectTestRoomData_NewRoom` 方法。

### 清理 Checklist

| 步驟 | 檔案 | 操作 | 狀態 |
|------|------|------|------|
| 1 | FishLobby2024.cs | 刪除 `ParserRoomData` 中的注入呼叫 | ⬜ |
| 2 | FishLobby2024.cs | 刪除 `InjectTestRoomData_NewRoom` 方法 | ⬜ |
| 3 | Unity Editor | 確認大廳正常顯示 Server 回傳的廳館資料 | ⬜ |

### Agent 提示範本

當使用者說「Server 已實作完成」或「清理測試程式碼」時，Agent 應：

```
⚠️ 測試程式碼清理流程

請確認以下項目已刪除：

1. FishLobby2024.cs - ParserRoomData 方法中的注入呼叫：
   - 搜尋：InjectTestRoomData_
   - 刪除整個 #if DEBUG_LOG ... #endif 區塊

2. FishLobby2024.cs - 注入方法：
   - 搜尋：InjectTestRoomData_NewRoom
   - 刪除整個方法（包含 #if DEBUG_LOG ... #endif）

3. 驗證：
   - 在 Unity Editor 中執行遊戲
   - 確認大廳正常顯示 Server 回傳的廳館資料
   - 確認新廳館可正常進入

是否需要我幫你執行清理？
```

---

## 測試資料注入 vs 正式資料對照

| 項目 | 測試資料注入 | 正式 Server 資料 |
|------|-------------|-----------------|
| 資料來源 | Client 端 `InjectTestRoomData_XXX` | Server `ENTER_FH_LOBBY` 回傳 |
| 啟用條件 | `#if DEBUG_LOG` | 無條件 |
| 排序位置 | 強制第一個（方便測試） | 依 Server `room_sort_list` |
| bet_info | 固定值（比照 monstrum） | Server 動態設定 |
| collect_token_info | 固定值 | Server 動態設定 |
| 生命週期 | 測試期間 | 永久 |
