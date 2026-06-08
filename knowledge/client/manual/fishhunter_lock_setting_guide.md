---
inclusion: manual
---

# 智能鎖定面板新增魚種指南 (Smart Lock Setting Panel Guide)

本指南說明如何在 OceanTreasure (2D 海王) 的智能鎖定面板中新增一個魚種。
當使用者說「新增智能鎖定魚種」、「加入鎖定面板」、「智能鎖定新增」時，請依照本指南逐步完成。

---

## 1. 系統概述

智能鎖定面板 (`FH_LockSettingPanel`) 是 VIP 廳館的功能，允許玩家自訂要鎖定的魚種優先順序。
系統由三層組件組成：

| 組件 | 類別 | 職責 |
|------|------|------|
| 主控制器 | `FH_LockSettingPanel` | 面板邏輯、魚種列表管理、鎖定目標搜尋 |
| 資料結構 | `FishHunterLockSettingDataSO` | ScriptableObject，定義單一魚種的圖示與類型 |
| UI 項目 | `FH_FishItemCtrl` | 個別魚種的 UI 表現（圖示、倍率、選取狀態） |

### 運作流程

```
FH_LockSettingPanel.Init()
  ↓ 遍歷 lockFishSettingDataList
  ↓ 每個 SO → Instantiate fishPrefab → FH_FishItemCtrl.Init(SO)
  ↓ 顯示圖示 + 倍率範圍（OddsMin ~ OddsMax）
  ↓
玩家選取魚種 → BtnLockSettingActivate()
  ↓ 收集選取的 fishType → 按倍率排序 → lockSettingFishLists
  ↓
WeaponLocked.UpdateLockTarget()
  ↓ 呼叫 FH_LockSettingPanel.GetLockSettingFish()
  ↓ 依 lockSettingFishLists 順序搜尋場上可鎖定的魚
  ↓ 回傳第一個符合的 Fish2D
```

---

## 2. 快速參考：必要資訊

新增魚種前，需確認以下資訊：

| 項目 | 說明 | 範例 | 必須確認 |
|------|------|------|----------|
| enumFishType | 魚種在 `Fish2DEnum.cs` 中的列舉值 | `enumType_ThreePigsBoss` | ✅ 必確認 |
| type 編號 | 魚種的 index 數值 | `258` | ✅ 必確認 |
| 魚種中文名稱 | 用於識別 | `金屋藏寶` | ✅ 必確認 |
| 圖示 Sprite | 面板中顯示的圖片 | `Gaming_ThreePigsBoss.png` | ✅ 必確認 |
| isGem | 是否顯示寶石圖示（隱藏倍率） | `false` | ⚠️ 預設 false |
| Fish2D.FishType[] 倍率 | MinOdds / MaxOdds | `200 / 13000` | 🔄 自動讀取 |

### Agent 詢問範本

```
請確認以下資訊：
1. 魚種的 enumFishType 或 type 編號：
2. 魚種中文名稱：
3. 圖示 Sprite 檔案路徑（或由我搜尋）：
4. 是否顯示寶石圖示？（預設顯示倍率數字）

以下我會自動處理：
- 從 Fish2D.FishType[] 讀取倍率範圍
- 建立 ScriptableObject 資產
- 加入 Prefab 的 lockFishSettingDataList（最前面）
```

---

## 3. 關鍵檔案路徑

```
Assets/FishHunter_Script/ArkGame/Scripts/
├── ScriptableObject/FishHunterLockSettingDataSO.cs    ← SO 定義
├── Game/GameSystem/FH_LockSettingPanel.cs             ← 主控制器
├── Game/GameSystem/FH_FishItemCtrl.cs                 ← UI 項目控制器
├── Game/WeaponSystem/WeaponLocked.cs                  ← 鎖定邏輯整合
└── FishAI/Fish2DEnum.cs                               ← enumFishType + FishType[]

Assets/FishtHunter_Res/FishGame/
├── FishHunterLockSetting/                             ← SO 資產存放位置
│   ├── FishHunterLockSettingData_258.asset            ← 命名格式：_type編號
│   └── FishHunterLockSettingData_258.asset.meta
└── Prefab_common/
    └── FH_LockSetting_UIPanel.prefab                  ← 面板 Prefab（含 lockFishSettingDataList）
```

---

## 4. Checklist：新增魚種步驟

### ✅ 步驟 1：確認前置條件

#### 1.1 確認 enumFishType 已定義

**檔案**: `Assets/FishHunter_Script/ArkGame/Scripts/FishAI/Fish2DEnum.cs`

搜尋目標魚種的列舉值，確認存在且 index 正確。

```csharp
public enum enumFishType
{
    // ...
    enumType_ThreePigsBoss,   // 258 - 金屋藏寶
    // ...
}
```

#### 1.2 確認 Fish2D.FishType[] 有倍率資料

**檔案**: 同上，`FishType` 靜態陣列

確認對應 index 的 `MinOdds` / `MaxOdds` 已設定（面板會顯示 `"200-13,000"` 格式）。

```csharp
// FishType[258]
new Fish2D._fishType("金屋藏寶", Fish2D.enumFishType.enumType_ThreePigsBoss, ..., 200, 13000, ...)
```

#### 1.3 確認圖示 Sprite 存在

圖示統一使用 `Lock_Setting_Atlas_01.png` 圖集中對應魚種編號的 Sprite。

**圖集位置**: `Assets/FishtHunter_Res/FishGame/Games/FishUI2/Lock_Setting_Atlas_01.png`

**命名規則**: `Lock_Setting_{type}`（如 `Lock_Setting_258`）

**取得 fileID 方式**：
```powershell
# 從 Atlas meta 檔案中搜尋對應 Sprite 的 fileID
Select-String -Path "Assets/FishtHunter_Res/FishGame/Games/FishUI2/Lock_Setting_Atlas_01.png.meta" -Pattern "Lock_Setting_{type}" -Context 0,0
```

在 `nameFileIdTable` 區段中找到對應的 fileID（負數），例如：
```yaml
Lock_Setting_258: -8126478980449582511
```

---

### ✅ 步驟 2：建立 FishHunterLockSettingDataSO 資產

#### 2.1 取得必要 GUID 和 fileID

```powershell
# 生成新 SO 的 GUID
[System.Guid]::NewGuid().ToString("N")

# 取得 Atlas 圖片的 GUID（固定值）
# Lock_Setting_Atlas_01.png GUID: 744a00ab6f7f8124ca69ed038a5c1ea4

# 取得對應 Sprite 的 fileID（從 Atlas meta 的 nameFileIdTable）
Select-String -Path "Assets/FishtHunter_Res/FishGame/Games/FishUI2/Lock_Setting_Atlas_01.png.meta" -Pattern "Lock_Setting_{type}:"
# 輸出範例：Lock_Setting_258: -8126478980449582511
```

#### 2.2 建立 .asset 檔案

**路徑**: `Assets/FishtHunter_Res/FishGame/FishHunterLockSetting/FishHunterLockSettingData_{type}.asset`

**YAML 格式**：

```yaml
%YAML 1.1
%TAG !u! tag:unity3d.com,2011:
--- !u!114 &11400000
MonoBehaviour:
  m_ObjectHideFlags: 0
  m_CorrespondingSourceObject: {fileID: 0}
  m_PrefabInstance: {fileID: 0}
  m_PrefabAsset: {fileID: 0}
  m_GameObject: {fileID: 0}
  m_Enabled: 1
  m_EditorHideFlags: 0
  m_Script: {fileID: 11500000, guid: c0ee5d2cec57bac4c9751306482fabf2, type: 3}
  m_Name: FishHunterLockSettingData_{type}
  m_EditorClassIdentifier: 
  fishType: {type}
  fish2D: {fileID: 0}
  iconSprite: {fileID: {Sprite的fileID}, guid: 744a00ab6f7f8124ca69ed038a5c1ea4, type: 3}
  isGem: 0
```

**欄位說明**：

| 欄位 | 值 | 說明 |
|------|-----|------|
| `m_Script` guid | `c0ee5d2cec57bac4c9751306482fabf2` | FishHunterLockSettingDataSO.cs 的 GUID（固定值） |
| `m_Name` | `FishHunterLockSettingData_{type}` | 資產名稱 |
| `fishType` | `{type}` | enumFishType 的 index 數值 |
| `fish2D` | `{fileID: 0}` | 可設為 null（面板不使用此欄位） |
| `iconSprite` fileID | 從 Atlas meta 取得的負數 ID | `Lock_Setting_{type}` 對應的 fileID |
| `iconSprite` guid | `744a00ab6f7f8124ca69ed038a5c1ea4` | Lock_Setting_Atlas_01.png 的 GUID（固定值） |
| `isGem` | `0` 或 `1` | 0=顯示倍率文字，1=顯示寶石圖示 |

#### 2.3 建立 .asset.meta 檔案

```yaml
fileFormatVersion: 2
guid: {新生成的GUID}
NativeFormatImporter:
  externalObjects: {}
  mainObjectFileID: 11400000
  userData: 
  assetBundleName: 
  assetBundleVariant: 
```

---

### ✅ 步驟 3：加入 Prefab 的 lockFishSettingDataList

#### 3.1 修改 Prefab 檔案

**檔案**: `Assets/FishtHunter_Res/FishGame/Prefab_common/FH_LockSetting_UIPanel.prefab`

**操作**: 在 `lockFishSettingDataList:` 的第一個項目前插入新 SO

**⚠️ 重要：新魚種必須插入到列表最前面**（增加對玩家的曝光）

使用 strReplace：

```
oldStr:
  lockFishSettingDataList:
  - fishHunterLockSettingDataSO: {fileID: 11400000, guid: {原第一個SO的GUID}, type: 2}
    isOpen: 1

newStr:
  lockFishSettingDataList:
  - fishHunterLockSettingDataSO: {fileID: 11400000, guid: {新SO的GUID}, type: 2}
    isOpen: 1
  - fishHunterLockSettingDataSO: {fileID: 11400000, guid: {原第一個SO的GUID}, type: 2}
    isOpen: 1
```

**YAML 格式說明**：

| 欄位 | 值 | 說明 |
|------|-----|------|
| `fileID` | `11400000` | ScriptableObject 的標準 fileID（固定值） |
| `guid` | SO 的 GUID | 步驟 2.3 中 .meta 檔案的 guid |
| `type` | `2` | Unity 資源類型（2 = ScriptableObject） |
| `isOpen` | `1` | 是否在面板中顯示（1=顯示，0=隱藏） |

---

### ✅ 步驟 4：刷新 AssetDatabase

使用 unity-cli 刷新：

```powershell
$json = '{"action":"refresh"}'; [System.IO.File]::WriteAllText("params.json", $json, [System.Text.UTF8Encoding]::new($false)); & "$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe" raw manage_asset_database --params-file params.json
```

---

### ✅ 步驟 5：驗證

#### 5.1 打開 Prefab 確認列表數量

```powershell
$json = '{"prefabPath":"Assets/FishtHunter_Res/FishGame/Prefab_common/FH_LockSetting_UIPanel.prefab"}'; [System.IO.File]::WriteAllText("params.json", $json, [System.Text.UTF8Encoding]::new($false)); & "$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe" raw open_prefab --params-file params.json
```

#### 5.2 確認 SO 被正確引用

```powershell
Select-String -Path "Assets/FishtHunter_Res/FishGame/Prefab_common/FH_LockSetting_UIPanel.prefab" -Pattern "{新SO的GUID}"
```

#### 5.3 Play Mode 測試

進入 VIP 廳館 → 點擊智能鎖定按鈕 → 確認新魚種出現在列表最前面，圖示和倍率正確。

---

## 5. FishHunterLockSettingDataSO 結構

```csharp
// Assets/FishHunter_Script/ArkGame/Scripts/ScriptableObject/FishHunterLockSettingDataSO.cs

[CreateAssetMenu(fileName = "FishHunterLockSettingData", menuName = "VIP館/鎖定魚種", order = 1)]
public class FishHunterLockSettingDataSO : ScriptableObject
{
    [SerializeField] private enumFishType fishType;      // 魚種類型
    public enumFishType FishType { get; }

    [SerializeField] private Fish2D fish2D;              // Fish2D 參考（面板未使用）
    public Fish2D Fish2D { get; }

    [SerializeField] private Sprite iconSprite;          // 面板圖示
    public Sprite IconSprite { get; }

    [SerializeField] private bool isGem;                 // 是否顯示寶石圖示
    public bool IsGem { get; }
}
```

---

## 6. 鎖定目標搜尋邏輯

`GetLockSettingFish()` 的搜尋邏輯：

```csharp
public Fish2D GetLockSettingFish()
{
    Bounds Bound = BoundCalculater.WorldBounds;

    // 依 lockSettingFishLists 順序（倍率高→低）搜尋
    foreach (int fishTypeID in lockSettingFishLists)
    {
        enumFishType lockSettingFishType = (enumFishType)fishTypeID;
        foreach (var pair in FishMaintainer.Instance.FishServerIDs)
        {
            Fish2D fish = pair.Value;
            RadiusChecker _TargetCollider = fish.GetLockableCollider();
            if (_TargetCollider != null &&
                BoundCalculater.Contains2D(Bound, _TargetCollider.transform.position) &&
                fish.IsTargetable())
            {
                if (lockSettingFishType == fish.type)
                    return fish; // 找到目標魚
            }
        }
    }
    return null; // 未找到目標魚
}
```

**搜尋條件**：
1. 魚在畫面範圍內（`BoundCalculater.Contains2D`）
2. 魚可被鎖定（`IsTargetable()` = true）
3. 魚的 type 在玩家選取的列表中

---

## 7. 特殊分類處理

### 7.1 星座魚

若新魚種屬於星座魚，需額外在 `FH_LockSettingPanel` Inspector 的 `constellationFishList` 中加入：

```csharp
[Header("星座魚List"), SerializeField]
List<enumFishType> constellationFishList = new List<enumFishType>();
```

同時確認 `FishRules.IsConstellationFish()` 回傳 `true`。

### 7.2 50 倍以下魚種

不需額外處理。`BtnLockSettingActivate()` 會自動根據 `Fish2D.FishType[i].MaxOdds <= setMaxOdds` 判斷。

---

## 8. ClickLog 數據影響

新增魚種後 `fishItemCtrlList` 長度 +1，ClickLog 字串會多一位。

**重要**：程式碼中有 `Reverse()` 處理：
```csharp
sendLockFishList.Reverse(); // 反轉字串
```

因此新魚種（列表最前面）的 0/1 會出現在 ClickLog 字串的**末尾**，不影響既有數據分析。

但仍建議通知數據分析團隊字串長度變更。

---

## 9. 注意事項

### ⚠️ IsTargetable() 確認

BOSS 魚種或技能魚種在特殊狀態（演出中、覺醒中）時 `IsTargetable()` 可能回傳 `false`。
確認新魚種在正常游動狀態下此方法回傳 `true`，否則智能鎖定無法鎖定該魚。

### ⚠️ 廳館限制

智能鎖定面板僅在 VIP 廳館開放（`EnableSmartLock` 方法中判斷）：
- ✅ 開放：myth、mech、luck 等 VIP 系列
- ❌ 不開放：normal、tutorial、rookie

### ⚠️ 武器限制

需要在 `ArgsCtrl.Instance.isEnableSmartLockWeapon` 清單中的武器才能使用智能鎖定。

### ⚠️ SO 的 fish2D 欄位

`FishHunterLockSettingDataSO` 的 `fish2D` 欄位在 `FH_FishItemCtrl.Init()` 中**未被使用**，
可安全設為 `{fileID: 0}`（null）。面板只使用 `fishType`、`iconSprite`、`isGem` 三個欄位。

---

## 10. 完整修改清單總覽

| 步驟 | 檔案 | 修改類型 | 必要性 |
|------|------|----------|--------|
| 1.1 | Fish2DEnum.cs | 確認 enumFishType 存在 | ✅ 前置確認 |
| 1.2 | Fish2DEnum.cs | 確認 FishType[] 倍率資料 | ✅ 前置確認 |
| 1.3 | 魚種資源資料夾 | 確認圖示 Sprite 存在 | ✅ 前置確認 |
| 2.1 | — | 生成 GUID、取得圖示 GUID | ✅ 必要 |
| 2.2 | FishHunterLockSettingData_{type}.asset | 建立 SO 資產 | ✅ 必要 |
| 2.3 | FishHunterLockSettingData_{type}.asset.meta | 建立 meta 檔案 | ✅ 必要 |
| 3.1 | FH_LockSetting_UIPanel.prefab | 插入 lockFishSettingDataList 最前面 | ✅ 必要 |
| 4 | — | 刷新 AssetDatabase | ✅ 必要 |
| 5 | — | 驗證 | ✅ 必要 |

---

## 11. Agent 執行範例

### 完整對話範例

```
使用者：幫我把金屋藏寶加入智能鎖定面板

Agent：讓我確認一下資訊：
- enumFishType: enumType_ThreePigsBoss (index 258)
- 倍率：200 ~ 13,000
- 圖示：搜尋中...

找到圖示：Gaming_ThreePigsBoss.png
是否使用此圖示？是否顯示寶石圖示？（預設顯示倍率文字 "200-13,000"）

使用者：用那張圖，顯示倍率就好

Agent：好的，開始執行：
1. ✅ 建立 FishHunterLockSettingData_258.asset
2. ✅ 建立 FishHunterLockSettingData_258.asset.meta
3. ✅ 修改 FH_LockSetting_UIPanel.prefab（插入列表最前面）
4. ✅ 刷新 AssetDatabase

完成！請在 Play Mode 中進入 VIP 廳館測試智能鎖定面板。
```

---

## 12. iconSprite 圖示來源

所有智能鎖定面板的魚種圖示統一來自 **Lock_Setting_Atlas_01.png** 圖集。

**圖集路徑**: `Assets/FishtHunter_Res/FishGame/Games/FishUI2/Lock_Setting_Atlas_01.png`
**圖集 GUID**: `744a00ab6f7f8124ca69ed038a5c1ea4`（固定值）

### Sprite 命名規則

```
Lock_Setting_{type編號}
```

範例：`Lock_Setting_258`、`Lock_Setting_244`、`Lock_Setting_260`

### 取得 fileID 方式

```powershell
# 搜尋 nameFileIdTable 中對應的 fileID
Select-String -Path "Assets/FishtHunter_Res/FishGame/Games/FishUI2/Lock_Setting_Atlas_01.png.meta" -Pattern "Lock_Setting_{type}:"
```

輸出範例：
```
Lock_Setting_258: -8126478980449582511
```

### SO 中的 iconSprite 格式

```yaml
iconSprite: {fileID: -8126478980449582511, guid: 744a00ab6f7f8124ca69ed038a5c1ea4, type: 3}
```

| 欄位 | 值 | 說明 |
|------|-----|------|
| `fileID` | 負數（從 nameFileIdTable 取得） | Atlas 中特定 Sprite 的 ID |
| `guid` | `744a00ab6f7f8124ca69ed038a5c1ea4` | Lock_Setting_Atlas_01.png 的 GUID（固定） |
| `type` | `3` | Unity 資源類型（3 = Prefab/Asset） |

### ⚠️ 注意

若新魚種的圖示尚未加入 Atlas（`Lock_Setting_{type}` 不存在於 meta 中），
需先請美術將圖示加入 `Lock_Setting_Atlas_01.png` 圖集，重新打包後才能取得 fileID。
