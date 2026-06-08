---
inclusion: manual
---

# 新增魚種 Skill (Add Fish Species)

當使用者說「新增魚種」、「加入新魚」或「建立魚種」時，啟動此 Skill 執行完整的魚種開發流程。

## 語言規範

- 所有對話使用**繁體中文**
- 程式碼中的專有名詞保持英文

---

## 第零步：選擇參考範本

當使用者說要新增魚種時，**先列出以下參考範本選項**讓使用者選擇，再進入資訊收集：

| 選項 | 分類 | 封包路徑 | 處理系統 | 複雜度 | 說明 |
|------|------|----------|----------|--------|------|
| A | 一般魚種 | W2 | WeaponSystem | ⭐ | 死了直接給錢，無特殊演出 |
| B | DoubleFish（翻倍魚） | W2 | WeaponSystem | ⭐⭐ | 拉到砲台數錢，翻倍資訊在 w2 的 extra_dict（欄位 29） |
| C | 特殊狀態魚（f7） | W2 + F7 | WeaponSystem + FishSystem | ⭐⭐ | f7 封包即時更新魚種狀態（倍率、階段等），捕獲走 W2 |
| D | 技能魚種（傳統） | SkillSystem (自訂 sk_xxx) | SkillXxxObject | ⭐⭐⭐ | 獨立技能物件，自訂指令格式 |
| E | 技能魚種（通用） | SkillSystem (sk_skill_start) | SkillCommonObject | ⭐⭐⭐ | 標準化流程 sk_skill_start → sk_bomb_fish → sk_end |
| F | 空殼模板 | 未定 | 未定 | ⭐ | 僅建立基礎骨架（繼承 Fish2D 或 Fish3D + 覆寫 Init），提供後續開發模板 |

### 魚種分類對照表

| 選項 | FishRules 註冊 | Debug 功能 | 封包類型 | 範例 |
|------|---------------|-----------|---------|------|
| A | 不需要 | W2 + 假召喚 | W2 | 迦納魚、金蝙蝠 |
| B | `IsDoubleFish()` | W2 + 假召喚 | W2（翻倍資訊在 extra_dict） | 殺人鯨、聚寶盆、玉如意、月兔、吸血鬼公爵 |
| C | 不需要 | W2 + F7 + 假召喚 | W2 + F7 | 猴爺、虎爺、武士魚、小丑魚、彌勒佛、龍舞極、酒霸狂鯊、貓魚、泡泡魚、聖誕老人 |
| D | 視情況 | 自訂 sk_xxx | SkillSystem | 電磁蟹、鑽頭蟹、烈焰風暴、轉輪蟹、招財貓、黑龍 |
| E | 視情況 | sk_skill_start | SkillCommonObject | 邱比特2024、朱雀、朱雀覺醒、健美兔 |

使用者選擇後，Agent 會：
1. 根據分類決定需要修改的檔案範圍
2. 選擇對應的 Debug 範本（A 一般 / B DoubleFish / C 特殊狀態 f7 / D/E 技能）
3. 決定是否需要 FishRules 註冊、FishSystem f7 處理、SkillSystem 整合等
4. **Prefab 架構一律使用標準架構**（定義於 `fishhunter_fish_prefabs_standard.md`），不需要額外判斷或詢問，除非使用者明確表示不需要

---

## 第一步：收集必要資訊

向使用者詢問以下資訊（若未提供）：

| 參數 | 說明 | 範例 |
|-----|------|------|
| **fishName** | 魚種名稱 (PascalCase) | `SalmonFish`, `GoldenDragon` |
| **serverIndex** | Server ID（**必填，由 Server 端決定**） | 259 |
| **fishCategory** | 魚種分類（見下方說明） | 一般魚種 / DoubleFish / 技能魚種 |
| **minOdds** | 最小倍率 | 50 |
| **maxOdds** | 最大倍率 | 200 |
| **hasFeature** | 是否有特殊演出 | 是 |
| **serverName** | Server 名稱 (由 fishName 轉換) | `SALMON_FISH` |

### 魚種分類說明

| 分類 | 說明 | 處理方式 | 範例 |
|------|------|----------|------|
| **一般魚種** | 死了直接給錢 | WeaponSystem W2 封包 | 迦納魚、金蝙蝠 |
| **DoubleFish** | 拉到砲台數錢 | W2 + F7 狀態更新 | 殺人鯨、聚寶盆、玉如意 |
| **技能魚種** | 有特殊技能玩法 | SkillSystem 自訂指令 | 霸王蟹、電磁蟹 |

### 自動推導規則

- **serverName**: 將 fishName 轉換為 UPPER_SNAKE_CASE
  - `SalmonFish` → `SALMON_FISH`
  - `BuddhaFishEx` → `BUDDHA_FISH_EX`

### ⚠️ 重要：Server ID 對應規則

**`enumFishType`、`FishType[]`、`FishKind[]` 三個陣列的索引必須與 Server ID 完全對應！**

1. **Server ID 決定陣列位置**：魚種在陣列中的位置由 Server ID 決定，不是按新增順序
2. **跳號處理**：如果 Server ID 有跳號（保留號），必須使用預設值佔位：
   - `enumFishType`: 使用 `enumType_Reserved{數字},  // {數字} - 跳號保留佔位{數字}`
   - `FishType[]`: 使用 `new Fish2D._fishType("跳號保留佔位{數字}", Fish2D.enumFishType.enumType_Reserved{數字}, ...)`
   - `FishKind[]`: 使用 `new Fish2D._fishKind("跳號保留佔位{數字}", ...)`
   
   **重要**：`FishType[]` 中的 `enumFishType` 必須使用對應的 `enumType_Reserved{數字}`，不要使用 `enumType_Gurnard`

3. **範例**：若要新增 Server ID 259 的魚，但目前陣列只到 253，則需要先補上 254~258 的佔位項目

```csharp
// 假設目前最後一個是 253 (豬小弟)
enumType_PracticalPig,        // 253 - 豬小弟
enumType_Reserved254,         // 254 - 跳號保留佔位254
enumType_Reserved255,         // 255 - 跳號保留佔位255
enumType_Reserved256,         // 256 - 跳號保留佔位256
enumType_Reserved257,         // 257 - 跳號保留佔位257
enumType_Reserved258,         // 258 - 跳號保留佔位258
enumType_GoldenBat,           // 259 - 金蝙蝠
```

---

## 第二步：修改 Fish2DEnum.cs

### 2.1 確認目前陣列最大索引

查看 `enumFishType` 枚舉中 `enumLineOff_Leave` 之前的最後一個項目索引。

### 2.2 處理跳號（若有）

如果 Server ID 大於目前最大索引 + 1，需要先補上跳號佔位：

```csharp
enumType_Reserved{數字},         // {數字} - 跳號保留佔位{數字}
```

### 2.3 新增魚種枚舉

在 `enumFishType` 枚舉的 `enumLineOff_Leave` 之前新增：

```csharp
enumType_{FishName},              // {serverIndex} - {中文名稱}
```

---

## 第三步：修改 Fish2D.cs

### 3.1 處理跳號（若有）

如果 Server ID 有跳號，需要先在 `FishType[]` 和 `FishKind[]` 陣列中補上佔位項目。

**FishType[] 佔位格式**：
```csharp
new Fish2D._fishType("跳號保留佔位{數字}", Fish2D.enumFishType.enumType_Reserved{數字}, 1.0f, (int)Fish2D.enumFishLayer.Gurnard, 500, 0, 3, 1.0f, 5.0f, 1.0f, 30),
```

**重要**：`enumFishType` 必須使用對應的 `enumType_Reserved{數字}`，確保陣列索引與枚舉值一致。

**FishKind[] 佔位格式**：
```csharp
new Fish2D._fishKind("跳號保留佔位{數字}", 2, 2, 63, 42, 300, 2.0f, 30, 115, 72, true),
```

### 3.2 新增 _fishType

在 `FishType` 陣列中新增（位置必須對應 Server ID）：

```csharp
new Fish2D._fishType("{中文名稱}", Fish2D.enumFishType.enumType_{FishName}, 1.0f, (int)Fish2D.enumFishLayer.EventFish, {speed}, {steer}, 6, 0.0f, 0.0f, 0.0f, 20),
```

### 3.3 新增 _fishKind

在 `FishKind` 陣列中新增（位置必須對應 Server ID）：

```csharp
new Fish2D._fishKind("{中文名稱}", {minOdds}, {maxOdds}, {width}, {length}, 300, 2.0f, 30, 115, 72, true),
```

### 3.4 更新陣列大小

確保 `FishType[]` 和 `FishKind[]` 的陣列大小足夠容納新的 Server ID：

```csharp
static public Fish2D._fishType[] FishType = new Fish2D._fishType[{新大小}]
static public Fish2D._fishKind[] FishKind = new Fish2D._fishKind[{新大小}]
```

---

## 第四步：修改 FishRules.cs

新增魚種時，需要根據魚種特性在 `FishRules.cs` 的多個判斷方法中註冊。以下為完整的判斷方法清單：

### 4.1 必須註冊的方法（所有特殊魚種）

| 方法 | 用途 | 何時需要 |
|------|------|----------|
| `IsSpecialHide(Fish2D)` | 武器卡、烈焰風暴、鑽頭、電磁蟹、機械霸王蟹攻擊時進入透明 | **所有特殊魚種**（一般小魚不需要） |
| `IsAssignationHide(Fish2D)` | 召喚卡無法召喚此魚種 | 不可被召喚的魚種（BOSS、技能魚等） |
| `FishComingStay(Fish2D)` | 不會被魚潮來襲沖走 | 所有特殊魚種（BOSS、活動魚、技能魚等） |
| `IsCustomCaptureFish(enumFishType)` | 有自訂捕獲演出（w2 路由到 ShowAwardsManager 的特殊分支） | 有特殊演出的魚種（FeatureCtrl） |

### 4.2 依魚種特性選擇註冊的方法

| 方法 | 用途 | 何時需要 |
|------|------|----------|
| `IsDoubleFish(enumFishType)` | 翻倍魚判定（拉到砲台數錢） | DoubleFish 分類 |
| `IsEternalFish(enumFishType)` | 永生魚（不會自然死亡） | 永生魚種（星座魚、三隻小豬等） |
| `IsHaveMinBet(Fish2D)` | 有最小押注限制 | 有押注門檻的魚種 |
| `IsHaveMinVIP(Fish2D)` | 有最小 VIP 限制 | 有 VIP 門檻的魚種 |

### 4.3 電擊/雷鳴相關排除清單

| 方法 | 用途 | 何時需要 |
|------|------|----------|
| 電擊排除清單（`IsSpecialHide` 後面的方法） | 電擊不能打的魚 | 所有特殊魚種 |
| `ThunderTwoExceptBonusFish(enumFishType)` | 雷鳴二不能被選為 bonus 的魚種 | 所有特殊魚種 |

### 4.4 其他清單

| 方法 | 用途 | 何時需要 |
|------|------|----------|
| 最後的特殊魚總清單（檔案末尾） | 特殊魚種總清單（用於 UI 顯示等） | 所有特殊魚種 |

### 4.5 DoubleFish 專用

若魚種為 **DoubleFish**（拉到砲台數錢），需在 `FishRules.cs` 的 `IsDoubleFish()` 方法中註冊：

```csharp
public static bool IsDoubleFish(Fish2D.enumFishType fishType)
{
    switch (fishType)
    {
        // ... 現有項目
        case Fish2D.enumFishType.enumType_{FishName}:
            return true;
        default:
            return false;
    }
}
```

### 4.6 快速判斷表

根據魚種分類，以下為需要註冊的方法對照：

| 分類 | IsSpecialHide | IsAssignationHide | FishComingStay | IsCustomCaptureFish | 電擊排除 | 雷鳴二排除 | 特殊魚總清單 |
|------|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| A 一般魚種 | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ | ✅ |
| B DoubleFish | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ | ✅ |
| C 特殊狀態魚 | ✅ | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| D 技能魚種（傳統） | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| E 技能魚種（通用） | ✅ | ❌ | ✅ | ❌ | ✅ | ✅ | ✅ |
| BOSS 魚種 | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## 第五步：修改 FishHunter_EventType.cs

在 `EventType` 枚舉中新增：

```csharp
//對應{中文名稱}相關功能
{FishName}_Capture,         //{中文名稱}捕獲事件
{FishName}_SpawnFeatureObj, //生成{中文名稱}Feature物件
// 報獎使用通用事件 Common_ShowDeclareBoard
```

---

## 第六步：修改 FishHunterCheatTool.cs

在 `FishTestDataList` 中新增：

```csharp
new FishTestData("{中文名稱}", "{SERVER_NAME}", {serverIndex}, true),
```

---

## 第七步：建立魚種資源資料夾

> **📐 標準架構提醒**：新增的魚種 Prefab **一律使用標準架構**（定義於 `fishhunter_fish_prefabs_standard.md`）。
> Agent 應在此步驟主動向使用者說明：
>
> 「新增的魚種將使用標準 Prefab 架構：Root（Fish2D + 物理 + 路徑）→ {FishName}Shape（Handler 群）→ freeze / shadow / maskroot/main / collider / Paralysis。如果有特殊需求需要調整結構，請告訴我。」
>
> 除非使用者明確表示不需要，否則直接套用標準架構，不需要額外詢問。

### 7.1 建立腳本資料夾

建立資料夾路徑：`Assets/FishHunter_Script/ArkGame/Scripts/FishAI/{FishName}/`

使用 PowerShell 生成標準 UUID v4：

```powershell
[System.Guid]::NewGuid().ToString("N")
```

建立 `.meta` 檔案：`Assets/FishHunter_Script/ArkGame/Scripts/FishAI/{FishName}.meta`

```yaml
fileFormatVersion: 2
guid: {生成的GUID}
folderAsset: yes
DefaultImporter:
  externalObjects: {}
  userData: 
  assetBundleName: 
  assetBundleVariant: 
```

### 7.2 建立資源資料夾（含 AssetBundle 設定）

建立資料夾路徑：`Assets/FishtHunter_Res/ArkGame/Fishes/{FishName}/`

建立 `.meta` 檔案：`Assets/FishtHunter_Res/ArkGame/Fishes/{FishName}.meta`

```yaml
fileFormatVersion: 2
guid: {生成的GUID}
folderAsset: yes
DefaultImporter:
  userData: 
  assetBundleName: {fishname小寫}
  assetBundleVariant: 
```

### ⚠️ AssetBundle 命名規範

| 項目 | 規則 | 範例 |
|------|------|------|
| **assetBundleName** | 魚種名稱轉為**全小寫** | `GoldenBat` → `goldenbat` |
| **格式** | 無底線、無空格、純小寫英文 | `goldendragon`, `salmonfish` |

### ⚠️ AssetBundle 同名資源禁止

**同一個 AssetBundle 內，不同類型的資源檔案不可與 Prefab 同名。**

例如以下結構會導致 `LoadFishBundle` 載入失敗（`obj = null`）：
```
Fishes/TreasureBowl2/
├── spine/TreasureBowl2.png    ← 與 Prefab 同名！
└── TreasureBowl2.prefab
```

`AssetBundleManager.LoadAssetAsync` 以 assetName 搜尋時會找到 `.png` 而非 `.prefab`，
導致 `request.GetAsset<GameObject>()` 回傳 `null`。

**解決方式**：確保 Spine 圖檔（atlas png）不與 Prefab 同名，例如改名為 `TreasureBowl2_atlas.png`。

---

## 第八步：建立 Debug 工具

建立檔案：`Assets/FishHunter_Script/ArkGame/Scripts/FishAI/{FishName}/{FishName}Debug.cs`

### 8.1 尋找魚種方式

**統一使用 `FishMaintainer.Instance?.GetFishByType()` 尋找魚種**，不要自訂 Find 方法：

```csharp
var fish = FishMaintainer.Instance?.GetFishByType(Fish2D.enumFishType.enumType_{FishName});
```

### 8.2 假召喚方式

**使用 `FishHunterFakeServerPacket` 三步驟**：

```csharp
// 1. 建立魚種資料
JSON fishData = FishHunterFakeServerPacket.GetFakeFishPacket(
    Fish2D.enumFishType.enumType_{FishName},
    fakeSummonX: x,
    fakeSummonY: y,
    fakeSummonO: orientation
);

// 2. 包裝成生成封包
JSON spawnData = FishHunterFakeServerPacket.GetFakeSpawnFishPacket(fishData);

// 3. 包裝成系統指令並發送
JSON sysData = FishHunterFakeServerPacket.GetFakeSystemCmdPacket("fish", "f1", spawnData);
GameClient.Instance.getFishSystem().onMessage(sysData);
```

### 8.3 一般魚種 Debug 範本

```csharp
using ArkGame;
using OceanTreasure;
using Sirenix.OdinInspector;
using UnityEngine;

/// <summary>
/// {中文名稱} Debug 工具
/// 一般魚種（死了直接給錢）
/// </summary>
public class {FishName}Debug : MonoBehaviour
{
#if DEBUG_LOG
    [System.Serializable]
    public class {FishName}Data
    {
        [BoxGroup("基本設定"), LabelText("倍率 (Odds)")]
        public int odds = 100;

        [BoxGroup("基本設定"), LabelText("測試座位 (Seat)"), Range(0, 3)]
        public int testSeat = 0;

        [BoxGroup("基本設定"), LabelText("是否為本家")]
        public bool isMainPlayer = true;
    }

    [Title("{中文名稱} Debug 工具", "一般魚種 - 死了直接給錢")]
    [Button("W2-{中文名稱}擊殺封包", buttonSize: ButtonSizes.Large, ButtonStyle.Box)]
    private void TEST_{FISH_NAME}_HIT({FishName}Data data)
    {
        var fish = FishMaintainer.Instance?.GetFishByType(Fish2D.enumFishType.enumType_{FishName});

        if (fish == null)
        {
            Debug.LogError("[{FishName}Debug] 場景中找不到{中文名稱}，請先召喚魚種");
            return;
        }

        var fishId = fish.sid;
        var playerSystem = GameClient.Instance.getPlayerSystem();
        var player = data.isMainPlayer
            ? playerSystem.getMainPlayer()
            : playerSystem.getPlayer(data.testSeat);

        if (player == null)
        {
            Debug.LogError($"[{FishName}Debug] 找不到座位 {data.testSeat} 的玩家");
            return;
        }

        var bet = player.bet_value;
        var totalWins = bet * data.odds;

        string jsonString = $@"{{
            ""cmd"": ""w2"",
            ""data"": {{
                ""1"": ""0_9"",
                ""2"": ""{player.seat}"",
                ""3"": ""{player.credit}"",
                ""8"": {{
                    ""{fishId}"": {{
                        ""10"": ""{data.odds}"",
                        ""11"": ""{totalWins}"",
                        ""12"": 0,
                        ""13"": {{}},
                        ""14"": 0,
                        ""23"": {{}}
                    }}
                }},
                ""9"": ""{bet}"",
                ""10"": ""{data.odds}"",
                ""15"": null,
                ""24"": false,
                ""26"": ""{player.id}"",
                ""28"": 0
            }}
        }}";

        var json = JSON.Parse(jsonString);
        GameClient.Instance.getWeaponSystem().onMessage(json);

        Debug.Log($"[{FishName}Debug] 發送擊殺封包: FishID={fishId}, Odds={data.odds}, TotalWin={totalWins}");
    }

    [Button("假召喚{中文名稱}", buttonSize: ButtonSizes.Medium)]
    private void SpawnFake{FishName}(float x = 0, float y = 0, float orientation = 90)
    {
        JSON fishData = FishHunterFakeServerPacket.GetFakeFishPacket(
            Fish2D.enumFishType.enumType_{FishName},
            fakeSummonX: x,
            fakeSummonY: y,
            fakeSummonO: orientation
        );

        JSON spawnData = FishHunterFakeServerPacket.GetFakeSpawnFishPacket(fishData);
        JSON sysData = FishHunterFakeServerPacket.GetFakeSystemCmdPacket("fish", "f1", spawnData);

        GameClient.Instance.getFishSystem().onMessage(sysData);

        Debug.Log($"[{FishName}Debug] 假召喚{中文名稱}: X={x}, Y={y}, O={orientation}");
    }
#endif
}
```

### 8.4 DoubleFish Debug 範本

DoubleFish 需要額外的 F7 狀態更新功能：

```csharp
using ArkGame;
using OceanTreasure;
using Sirenix.OdinInspector;
using UnityEngine;

/// <summary>
/// {中文名稱} Debug 工具
/// DoubleFish（拉到砲台數錢）
/// </summary>
public class {FishName}Debug : MonoBehaviour
{
#if DEBUG_LOG
    [System.Serializable]
    public class {FishName}Data
    {
        [BoxGroup("基本設定"), LabelText("基底倍率 (BaseOdds)")]
        public int baseOdds = 100;

        [BoxGroup("基本設定"), LabelText("乘倍 (Multiply)")]
        public int multiply = 2;

        [BoxGroup("基本設定"), LabelText("總倍率 (TotalOdds)")]
        public double totalOdds => baseOdds * multiply;

        [BoxGroup("基本設定"), LabelText("測試座位 (Seat)"), Range(0, 3)]
        public int testSeat = 0;

        [BoxGroup("基本設定"), LabelText("是否為本家")]
        public bool isMainPlayer = true;

        [BoxGroup("演出設定"), LabelText("演出類型")]
        public int showType = 0;
    }

    [Title("{中文名稱} Debug 工具", "DoubleFish - 拉到砲台數錢")]
    [Button("W2-{中文名稱}擊殺封包", buttonSize: ButtonSizes.Large, ButtonStyle.Box)]
    private void TEST_{FISH_NAME}_HIT({FishName}Data data)
    {
        var fish = FishMaintainer.Instance?.GetFishByType(Fish2D.enumFishType.enumType_{FishName});

        if (fish == null)
        {
            Debug.LogError("[{FishName}Debug] 場景中找不到{中文名稱}，請先召喚魚種");
            return;
        }

        var fishId = fish.sid;
        var playerSystem = GameClient.Instance.getPlayerSystem();
        var player = data.isMainPlayer
            ? playerSystem.getMainPlayer()
            : playerSystem.getPlayer(data.testSeat);

        if (player == null)
        {
            Debug.LogError($"[{FishName}Debug] 找不到座位 {data.testSeat} 的玩家");
            return;
        }

        var bet = player.bet_value;
        var totalWins = bet * data.totalOdds;

        string jsonString = $@"{{
            ""cmd"": ""w2"",
            ""data"": {{
                ""1"": ""0_9"",
                ""2"": ""{player.seat}"",
                ""3"": ""{player.credit}"",
                ""8"": {{
                    ""{fishId}"": {{
                        ""10"": ""{data.totalOdds}"",
                        ""11"": ""{totalWins}"",
                        ""12"": 0,
                        ""13"": {{}},
                        ""14"": 0,
                        ""23"": {{}}
                    }}
                }},
                ""9"": ""{bet}"",
                ""10"": ""{data.totalOdds}"",
                ""15"": null,
                ""24"": false,
                ""26"": ""{player.id}"",
                ""28"": 0,
                ""29"": {{
                    ""base_odds"": ""{data.baseOdds}"",
                    ""multiply"": ""{data.multiply}"",
                    ""show_type"": ""{data.showType}""
                }}
            }}
        }}";

        var json = JSON.Parse(jsonString);
        GameClient.Instance.getWeaponSystem().onMessage(json);

        Debug.Log($"[{FishName}Debug] 發送擊殺封包: FishID={fishId}, TotalOdds={data.totalOdds}, TotalWin={totalWins}");
    }

    [Button("F7-{中文名稱}狀態更新", buttonSize: ButtonSizes.Large, ButtonStyle.Box)]
    private void TEST_{FISH_NAME}_UPDATE(int odds = 100, int multiply = 2)
    {
        var fish = FishMaintainer.Instance?.GetFishByType(Fish2D.enumFishType.enumType_{FishName});

        if (fish == null)
        {
            Debug.LogError("[{FishName}Debug] 場景中找不到{中文名稱}");
            return;
        }

        var fishId = fish.sid;

        string jsonString = $@"{{
            ""cmd"": ""f7"",
            ""data"": {{
                ""fish_id"": ""{fishId}"",
                ""odds"": ""{odds}"",
                ""fish_type"": ""{(int)Fish2D.enumFishType.enumType_{FishName}}"",
                ""multiply"": ""{multiply}""
            }}
        }}";

        var json = JSON.Parse(jsonString);
        GameClient.Instance.getFishSystem().onMessage(json);

        Debug.Log($"[{FishName}Debug] 發送狀態更新 F7: Odds={odds}, Multiply={multiply}");
    }

    [Button("假召喚{中文名稱}", buttonSize: ButtonSizes.Medium)]
    private void SpawnFake{FishName}(float x = 0, float y = 0, float orientation = 90)
    {
        JSON fishData = FishHunterFakeServerPacket.GetFakeFishPacket(
            Fish2D.enumFishType.enumType_{FishName},
            fakeSummonX: x,
            fakeSummonY: y,
            fakeSummonO: orientation
        );

        JSON spawnData = FishHunterFakeServerPacket.GetFakeSpawnFishPacket(fishData);
        JSON sysData = FishHunterFakeServerPacket.GetFakeSystemCmdPacket("fish", "f1", spawnData);

        GameClient.Instance.getFishSystem().onMessage(sysData);

        Debug.Log($"[{FishName}Debug] 假召喚{中文名稱}: X={x}, Y={y}, O={orientation}");
    }
#endif
}
```

---

## 第九步：建立魚種控制器（若需要）

建立檔案：`Assets/FishHunter_Script/ArkGame/Scripts/FishAI/{FishName}/{FishName}Ctrl.cs`

（控制器範本請參考 Auto_fishhunter_machine_expert.md 中的魚種控制器範本）

---

## 第十步：建立演出控制器（若 hasFeature = true）

建立檔案：`Assets/FishHunter_Script/ArkGame/Scripts/FishAI/{FishName}/{FishName}FeatureCtrl.cs`

（演出控制器範本請參考 Auto_fishhunter_machine_expert.md 中的演出控制器範本）

---

## 驗證檢查清單

完成後，確認以下項目：

### 必要項目

- [ ] `Fish2DEnum.cs` - 已新增 `enumType_{FishName}`（含跳號佔位）
- [ ] `Fish2D.cs` - 已新增 `_fishType` 定義（含跳號佔位）
- [ ] `Fish2D.cs` - 已新增 `_fishKind` 定義（含跳號佔位）
- [ ] `Fish2D.cs` - 陣列大小已更新
- [ ] `FishHunterCheatTool.cs` - 已新增測試資料
- [ ] 腳本資料夾 + `.meta` 檔案已建立
- [ ] 資源資料夾 + `.meta` 檔案已建立（含 AssetBundle 設定）

### DoubleFish 專用

- [ ] `FishRules.cs` - 已在 `IsDoubleFish()` 中註冊

### FishRules 透明化與排除清單

- [ ] `FishRules.cs` - 已在 `IsSpecialHide()` 中註冊（武器卡/烈焰風暴/鑽頭/電磁蟹/機械霸王蟹透明）
- [ ] `FishRules.cs` - 已在 `IsAssignationHide()` 中註冊（召喚卡排除，若適用）
- [ ] `FishRules.cs` - 已在 `FishComingStay()` 中註冊（不被魚潮沖走）
- [ ] `FishRules.cs` - 已在 `IsCustomCaptureFish()` 中註冊（若有自訂捕獲演出）
- [ ] `FishRules.cs` - 已在電擊排除清單中註冊
- [ ] `FishRules.cs` - 已在 `ThunderTwoExceptBonusFish()` 中註冊
- [ ] `FishRules.cs` - 已在特殊魚總清單（檔案末尾）中註冊

### Debug 工具

- [ ] `{FishName}Debug.cs` - 已建立 Debug 工具
- [ ] W2 擊殺封包功能正常
- [ ] 假召喚功能正常
- [ ] （DoubleFish）F7 狀態更新功能正常

### 選用項目

- [ ] `FishHunter_EventType.cs` - 已新增事件類型（若有特殊演出）
- [ ] `{FishName}Ctrl.cs` - 已建立魚種控制器（若有特殊邏輯）
- [ ] `{FishName}FeatureCtrl.cs` - 已建立演出控制器（若有特殊演出）

---

## 快速參考

### 魚種分類對照表

| 分類 | FishRules 註冊 | Debug 功能 | 封包類型 |
|------|---------------|-----------|---------|
| 一般魚種 | 不需要 | W2 + 假召喚 | W2 |
| DoubleFish | `IsDoubleFish()` | W2 + F7 + 假召喚 | W2 + F7 |
| 技能魚種 | 視情況 | 自訂 sk_xxx | SkillSystem |

### 關鍵 API

| 功能 | API |
|------|-----|
| 尋找魚種 | `FishMaintainer.Instance?.GetFishByType(enumFishType)` |
| 假召喚 | `FishHunterFakeServerPacket.GetFakeFishPacket()` → `GetFakeSpawnFishPacket()` → `GetFakeSystemCmdPacket()` |
| 發送 W2 | `GameClient.Instance.getWeaponSystem().onMessage(json)` |
| 發送 F7 | `GameClient.Instance.getFishSystem().onMessage(json)` |

---

## 使用範例

使用者：「我要新增三隻魚：金蝙蝠(259)是一般魚、聚寶盆(260)和玉如意(261)是DoubleFish」

執行：
1. 確認目前陣列最大索引（假設 253）
2. 補上 254~258 的跳號佔位
3. 新增三隻魚的枚舉、FishType、FishKind
4. 聚寶盆和玉如意需在 `FishRules.IsDoubleFish()` 註冊
5. 建立三個 Debug 工具（金蝙蝠用一般範本，另兩個用 DoubleFish 範本）
6. 更新 FishHunterCheatTool 測試資料
