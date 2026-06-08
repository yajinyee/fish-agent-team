---
inclusion: manual
---
# FishHunterCheatTool 作弊測試系統說明

本文件說明 FishHunter 專案中的作弊測試系統 `FishHunterCheatTool`，包含假召魚功能與其他測試工具。

---

## 1. 概述

`FishHunterCheatTool` 是開發階段使用的測試工具，提供以下功能：
- **假召魚 (FakeSummon)**：在 Client 端模擬 Server 生魚封包
- **真召魚 (SpecialSummon)**：透過 Server 生成指定魚種
- **武器切換**：快速切換砲台類型
- **魚種過濾**：阻擋特定魚種生成
- **技能測試**：測試各種技能魚種的演出
- **金流測試**：測試金幣增減

### 啟用條件

```csharp
#if DEBUG_MODE && !REMOVE_FH_TEST
    // 作弊工具啟用
#else
    Destroy(gameObject); // 非 DEBUG 模式下銷毀
#endif
```

---

## 2. 假召魚 (FakeSummon) 系統

### 2.1 與真召魚的差異

| 面向 | 假召魚 (FakeSummon) | 真召魚 (SpecialSummon) |
|-----|---------------------|------------------------|
| **觸發方式** | Client 端模擬封包 | Server 端生成 |
| **網路需求** | 不需要連線 | 需要連線到 Server |
| **SID 來源** | Client 端自動遞增 (100000+) | Server 分配 |
| **適用場景** | 離線測試、快速驗證 | 完整流程測試 |
| **封包路徑** | 直接注入 FishSystem | 經由 GameSystem 發送請求 |

### 2.2 假召魚 Entry Point

```csharp
// 位置：FishHunterCheatTool.cs

public void FakeSummon()
{
    // 1. 從 InputField 取得魚種 Index
    int fishTypeIndex = fakeSummonInput.text.ToInt32();
    Fish2D.enumFishType fishType = (Fish2D.enumFishType)fishTypeIndex;
    
    // 2. 建立假魚封包
    JSON fishData = FishHunterFakeServerPacket.GetFakeFishPacket(
        fishType,
        fakeSummonX: fakeSummonX,  // 生成 X 座標
        fakeSummonY: fakeSummonY,  // 生成 Y 座標
        fakeSummonO: fakeSummonO   // 生成角度
    );
    
    // 3. 包裝成生魚封包
    JSON spawnData = FishHunterFakeServerPacket.GetFakeSpawnFishPacket(fishData);
    
    // 4. 包裝成系統指令封包
    JSON sysData = FishHunterFakeServerPacket.GetFakeSystemCmdPacket("fish", "f1", spawnData);
    
    // 5. 直接注入 FishSystem
    GameClient.Instance.getFishSystem().onMessage(sysData);
}
```

### 2.3 假召魚流程圖

```
┌─────────────────────────────────────────────────────────────────┐
│                    假召魚 (FakeSummon) 流程                      │
└─────────────────────────────────────────────────────────────────┘

[使用者操作]
    │
    ▼
┌─────────────────────────────────────┐
│ 1. 輸入魚種 Index                    │
│    fakeSummonInput.text             │
│    (例：199 = 彌勒佛)                │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 2. FakeSummon() 被呼叫               │
│    - 解析魚種類型                    │
│    - 設定座標 (X, Y, O)              │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 3. GetFakeFishPacket()              │
│    - 產生假 SID (100000+)            │
│    - 建立魚種資料 JSON               │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 4. GetFakeSpawnFishPacket()         │
│    - 包裝成魚群資料                  │
│    - 設定 script_type               │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 5. GetFakeSystemCmdPacket()         │
│    - sys = "fish"                   │
│    - cmd = "f1"                     │
│    - data = spawnData               │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 6. FishSystem.onMessage()           │
│    - 直接注入封包                    │
│    - 觸發正常生魚流程                │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 7. FishMaintainer 生成魚種           │
│    - 從 Object Pool 取得魚           │
│    - 初始化並開始游動                │
└─────────────────────────────────────┘
```

---

## 3. FishHunterFakeServerPacket 封包工具

### 3.1 封包結構

```csharp
// 系統指令封包結構
{
    "sys": "fish",      // 系統名稱 (fish/weapon/...)
    "cmd": "f1",        // 指令類型 (f1=生魚)
    "data": { ... }     // 資料內容
}
```

### 3.2 主要 API

| 方法 | 用途 | 參數 |
|-----|------|------|
| `GetFakeSystemCmdPacket()` | 建立系統指令封包 | sysStr, cmd, data |
| `GetFakeFishPacket()` | 建立單隻魚資料 | fishType, x, y, o, extraType |
| `GetFakeSpawnFishPacket()` | 包裝生魚封包 | fishData |
| `GetFakeCpatureFishDataPacket()` | 建立捕獲封包 | fishType, seat, bet, odds... |

### 3.3 魚種資料欄位

```csharp
// FishData 欄位對應
data[fishData.id] = fishFakeId;        // 魚 SID
data[fishData.type] = (int)fishType;   // 魚種類型
data[fishData.x] = fakeSummonX;        // X 座標
data[fishData.y] = fakeSummonY;        // Y 座標
data[fishData.o] = fakeSummonO;        // 角度
data[fishData.feature] = 0;            // 特徵
data[fishData.position] = 0;           // 位置
data[fishData.size] = 1;               // 大小
data[fishData.frame] = 128;            // 幀數
data[fishData.min_bet] = 0;            // 最小押注
data[fishData.spacing] = null;         // 間距
data[fishData.path] = null;            // 路徑
data[fishData.min_vip] = 0;            // 最小 VIP
data[fishData.extra_type] = extraType; // 額外類型
```

---

## 4. 真召魚 (SpecialSummon) 系統

### 4.1 Entry Point

```csharp
// 透過 Server 生成魚種
public void SpecialSummon(string SpecialFishName)
{
    lastSummon = SpecialFishName;
    for (int i = 0; i < SpecialSummonNumber; i++)
    {
#if DEBUG_MODE
        // 發送測試生魚請求到 Server
        ArkGame.GameClient.Instance.getGameSystem().SendTestCreateFish(SpecialFishName);
#endif
    }
}
```

### 4.2 手動輸入召喚

```csharp
// 從 InputField 輸入魚種名稱
public void SpecialSummon_Manual()
{
#if DEBUG_MODE
    ArkGame.GameClient.Instance.getGameSystem().SendTestCreateFish(SpecialSummon_Input.text);
    lastSummon = SpecialSummon_Input.text;
#endif
}
```

---

## 5. FishTestData 魚種測試資料

### 5.1 資料結構

```csharp
class FishTestData
{
    public string showName;        // 顯示名稱（中文）
    public string serverName;      // Server 名稱（大寫底線格式）
    public int serverIndex;        // Server Index（= enumFishType 值）
    public bool isSpawnSummonBtn;  // 是否顯示召喚按鈕
    public Toggle spawmBlockToggle;// 阻擋 Toggle 參考
}
```

### 5.2 命名規則

| 欄位 | 格式 | 範例 |
|-----|------|------|
| `showName` | 繁體中文 | "彌勒佛"、"招財貓" |
| `serverName` | UPPER_SNAKE_CASE | "BUDDHA_FISH"、"LUCKY_CAT" |
| `serverIndex` | enumFishType 數值 | 199、114 |

### 5.3 常用魚種對照表

| showName | serverName | serverIndex | 說明 |
|----------|------------|-------------|------|
| 彌勒佛 | BUDDHA_FISH | 199 | 技能魚種 |
| 彌勒佛EX | BUDDHA_FISH_EX | 243 | 技能魚種 |
| 招財貓 | LUCKY_CAT | 114 | 技能魚種 |
| 龍舞極 | DOUBLE_DRAGON | 244 | 技能魚種 |
| 無限轉輪 | INFINITY_WHEEL | 222 | 技能魚種 |
| 雷霆帝龍 | THUNDER_DRAGON_EX | 215 | 技能魚種 |
| 黃金拉霸蟹 | GOLDEN_SLOT_CRAB | 229 | 技能魚種 |
| 寶鎚貓 | MALLET_MEOW | 213 | 技能魚種 |
| 狂暴火龍 | FIRE_DRAGON | 19 | 一般魚種 |
| 炸彈蟹 | BOMB_CRAB | 21 | 特殊蟹 |
| 電磁蟹 | LASER_CRAB | 22 | 特殊蟹 |
| 鑽頭蟹 | DRILL_CRAB | 23 | 特殊蟹 |
| 霸王蟹 | KING_SPIDER_CRAB | 24 | 技能魚種 |
| 轉輪蟹 | WHEEL_CRAB | 85 | 技能魚種 |
| 骰子魚 | DICE_FISH | 195 | 技能魚種 |
| 銅骰子 | COPPER_DICE_FISH | 247 | 技能魚種 |
| 銀骰子 | SILVER_DICE_FISH | 248 | 技能魚種 |
| 金骰子 | GOLDEN_DICE_FISH | 249 | 技能魚種 |
| 彩骰子 | MULTICOLOR_DICE_FISH | 250 | 技能魚種 |

---

## 6. 其他測試功能

### 6.1 魚種過濾 (SpawnBlock)

```csharp
// 阻擋所有魚種
public void SpawnBlock_ToggleAll()

// 阻擋特殊魚種
public void SpawnBlock_ToggleSpecialFish()

// 儲存/讀取自訂阻擋設定
public void SaveCustomBlockFishSetting()
public void LoadCustomBlockFishSetting()
```

### 6.2 武器切換

```csharp
// 切換砲台
public void SetWeapon(int _WeaponNo)

// 設定武器卡
public void SetWeaponCard_Electric()  // 電磁卡
public void SetWeaponCard_Drill()     // 電鑽卡
```

### 6.3 技能測試

```csharp
// 無限轉輪測試
public void SendInfinityWheelTest()

// 健美兔測試
public void SendMuscleRabbitTest()

// 馬戲團老虎測試
public void SendCircusTigerTest()

// 骰子魚測試
public void SendDiceFishAwardTest()

// 黃金拉霸蟹測試
public void FakePacket_GoldenSlotCrab_Slot()
public void FakePacket_GoldenSlotCrab_Scene()

// 龍舞極測試
public void OnSend_DoubleDragon_Test()
```

### 6.4 快捷鍵

| 功能 | 方法 |
|-----|------|
| 換房 | `TriggerChangeRoomHotKey()` |
| 重新召喚 | `TriggerReSummonFishHotKey()` |
| 召喚魚 | `TriggerSummonFishHotKey()` |
| 換武器 | `TriggerWeaponChangeHotKey()` |
| 設定召喚 | `TriggerSetSummonHotKey()` |
| 武士魚 Buffer | `TriggerSamuraiFishBufferHotKey()` |

---

## 7. UI 結構

### 7.1 主要 Panel

| Panel | 用途 |
|-------|------|
| `MainPanelAry` | 主功能面板陣列 |
| `SpecialPanelAry` | 特殊功能面板陣列 |
| `SpecialSummonBtnPanel` | 召喚魚按鈕面板 |
| `SetSummonBtnPanel` | 設定召喚面板 |
| `SetWeaponChangeBtnPanel` | 武器切換面板 |
| `SpawnBlockTogglePanel` | 魚種阻擋面板 |

### 7.2 快捷 Toggle

| Toggle | 功能 |
|--------|------|
| `SummonFish_Toggle` | 召喚魚面板開關 |
| `SetSummon_Toggle` | 設定召喚開關 |
| `WeaponChange_Toggle` | 武器切換開關 |
| `ReSummonFish_Toggle` | 重新召喚開關 |
| `ChangeRoom_Toggle` | 換房開關 |

---

## 8. 新增魚種測試資料

若要新增魚種到作弊工具，需修改 `FishTestDataList`：

```csharp
// FishHunterCheatTool.cs
List<FishTestData> FishTestDataList = new List<FishTestData>
{
    // ... 現有資料
    
    // 新增魚種
    new FishTestData(
        "顯示名稱",      // showName（中文）
        "SERVER_NAME",   // serverName（大寫底線格式）
        251,             // serverIndex（= enumFishType 值）
        true             // isSpawnSummonBtn（是否顯示按鈕）
    ),
};
```

### 8.1 SubType 魚種

若魚種有 SubType 屬性，需加入 `SubTypeFishTest` 枚舉：

```csharp
private enum SubTypeFishTest
{
    DICE_FISH,
    THUNDER_DRAGON_EX,
    VERMILION_BIRD,
    GOLDEN_SLOT_CRAB,
    GOLD_TREE,
    // 新增有 SubType 的魚種
    NEW_FISH_WITH_SUBTYPE,
}
```

---

## 9. 注意事項

1. **DEBUG_MODE 限定**：作弊工具僅在 `DEBUG_MODE` 下啟用
2. **假召魚 SID**：從 100000 開始遞增，避免與 Server 分配的 SID 衝突
3. **Server 名稱格式**：必須使用 UPPER_SNAKE_CASE 格式
4. **Server Index**：必須與 `enumFishType` 枚舉值一致
5. **特殊魚種處理**：蒐集任務魚等需要額外設定 `extra_data`

---

## 10. 檔案位置

| 檔案 | 位置 |
|-----|------|
| `FishHunterCheatTool.cs` | `FishGame/Script/CheatCodeGUI/` |
| `FishHunterFakeServerPacket.cs` | `FishGame/Script/CheatCodeGUI/` |

---

## 11. Server Index 參考

Server 的魚種 Index 資料可參考：
- Google Sheets: `https://docs.google.com/spreadsheets/d/11pBr9UNix_C3ogrT8cVyBkQ-Dpc5PFm1_zLnK0b37JY/edit?gid=2047043804#gid=2047043804`
- 或直接查看 `Fish2DEnum.cs` 中 `enumFishType` 的定義順序
