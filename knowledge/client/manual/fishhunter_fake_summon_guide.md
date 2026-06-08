---
inclusion: manual
---
# 假召魚 (Fake Summon Fish) 系統說明

本文件說明 FishHunter 專案中「假召魚」機制的 Entry Point 與完整流程。

---

## 1. 概述

「假召魚」是一種不經由 Server 生成指令（f1）產生的魚種，主要用於：
- **PlayBonus 系統**：玩家使用武器卡時，在 UI 層生成假魚進行演出
- **測試用途**：開發階段快速測試魚種行為

假召魚的特點：
- 不參與正常的魚場邏輯（不游動、不被一般子彈擊中）
- 使用負數或特殊 SID 避免與真實魚衝突
- 生成在 UI Layer，與遊戲場景分離
- 由 `FishMaintainer.fakeFishList` 統一管理

---

## 2. Entry Point（入口點）

### 2.1 主要入口：`FishMaintainer.SummonFakeFish()`

```csharp
// 位置：Assets/FishHunter_Script/ArkGame/Scripts/FishAI/FishMaintainer.cs

public void SummonFakeFish(
    int type,                           // 魚種類型 (enumFishType)
    int fakeFishSid,                    // 假魚 SID（通常為負數）
    Vector3 position,                   // 生成位置
    Action<Fish2D> SpawnFinishCB = null,// 生成完成回呼
    string sortingLayerName = "Fish",   // Sorting Layer 名稱
    int sortingOrder = -1,              // Sorting Order
    LayerMask layerMask = default,      // Unity Layer
    bool isUILayer = false              // 是否為 UI 層
)
```

### 2.2 使用範例

```csharp
// PlayBonus 系統中的使用方式
int fakeFishSid = -data.playerID + fishType; // 避免與真實魚 SID 衝突
Vector3 spawnPos = new Vector3(20000, 20000, 0); // UI 層位置

FishMaintainer.Instance.SummonFakeFish(
    fishType,
    fakeFishSid,
    spawnPos,
    (fish) => SpawnFishFinishCallback(fish, data),
    "UI",           // Sorting Layer
    595,            // Sorting Order
    LayerMask.NameToLayer("UI"),
    true            // isUILayer
);
```

---

## 3. 完整流程圖

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           假召魚完整流程                                      │
└─────────────────────────────────────────────────────────────────────────────┘

[玩家操作]
    │
    ▼
┌─────────────────────────────────────┐
│ 1. FH_PlayBonusManager              │
│    OnClickChooseSubIcon()           │
│    - 玩家選擇武器卡                   │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 2. SendChoosePlayBonusCMD()         │
│    - 發送 play_bonus_use 請求        │
│    - 透過 EventSystemGDK 路由        │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 3. SkillSystem.SendUsePlayBonus()   │
│    - Send("play_bonus_use", data)   │
│    - 發送至 Server                   │
└─────────────────────────────────────┘
    │
    ▼ (Server 回應)
┌─────────────────────────────────────┐
│ 4. SkillSystem.OnGetChoosePlayBonusCB()│
│    - 接收 Server 回應                 │
│    - 觸發 Event_ChoosePlayBonusCB    │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 5. FH_PlayBonusManager              │
│    OnGetChoosePlayBonusCB()         │
│    - 解析 PlayBonusUseResultData    │
│    - 扣除玩家金幣                    │
│    - 鎖定玩家射擊                    │
└─────────────────────────────────────┘
    │
    ├─────────────────┬─────────────────┐
    │ (本家玩家)       │ (他家玩家)       │
    ▼                 ▼                 │
┌─────────────────┐ ┌─────────────────┐ │
│ StartPlayBonus  │ │ 直接生魚         │ │
│ Coming()        │ │ SpawnFishByType │ │
│ - 顯示前置演出   │ │                 │ │
└─────────────────┘ └─────────────────┘ │
    │                 │                 │
    ▼                 │                 │
┌─────────────────┐   │                 │
│ PlayBonusComing │   │                 │
│ Ended()         │   │                 │
│ - 前置演出結束   │   │                 │
└─────────────────┘   │                 │
    │                 │                 │
    └────────┬────────┘                 │
             ▼                          │
┌─────────────────────────────────────┐ │
│ 6. SpawnFishByType()                │ │
│    - 播放召喚特效                    │ │
│    - 計算假魚 SID                    │ │
│    - 呼叫 SummonFakeFish()          │ │
└─────────────────────────────────────┘ │
    │                                   │
    ▼                                   │
┌─────────────────────────────────────┐ │
│ 7. FishMaintainer.SummonFakeFish()  │◄┘
│    [ENTRY POINT]                    │
│    - SpawnFish() 從 Object Pool 取魚 │
│    - 初始化魚種屬性                  │
│    - 設定為不可攻擊狀態              │
│    - 加入 fakeFishList              │
│    - 呼叫 TeleportFish()            │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 8. TeleportFish() (假魚版本)         │
│    - 呼叫 Fish2D.SummonIn()         │
│    - isFake = true                  │
│    - 播放縮放動畫                    │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 9. Fish2D.SummonIn()                │
│    - DOScale 動畫                   │
│    - isFake 時不設定 Force          │
│    - 觸發 SpawnFinishCB             │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 10. SpawnFishFinishCallback()       │
│     - 根據魚種類型客製化初始化        │
│     - 例：BuddhaCtrl.buddhaFishOddsInit()│
│     - 呼叫 KillFishBySID()          │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 11. KillFishBySID()                 │
│     - 根據魚種類型發送對應事件        │
│     - Buddha: Buddha_Capture        │
│     - MalletMeow: SkillMalletMeow   │
│     - InfinityWheel: SkillInfinityWheel│
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│ 12. 魚種 Feature 演出               │
│     - 各魚種獨立處理演出流程          │
│     - 演出結束後 Despawn 假魚        │
│     - 解鎖玩家射擊                   │
└─────────────────────────────────────┘
```

---

## 4. 關鍵類別與檔案

| 類別/檔案 | 位置 | 職責 |
|----------|------|------|
| `FH_PlayBonusManager` | `FH_PlayBonus/FH_PlayBonusManager.cs` | PlayBonus UI 管理、流程控制 |
| `FishMaintainer` | `ArkGame/Scripts/FishAI/FishMaintainer.cs` | 魚種生成、Object Pool、假魚管理 |
| `Fish2D` | `ArkGame/Scripts/FishAI/Fish2D.cs` | 魚種基底類別、SummonIn 動畫 |
| `SkillSystem` | `ArkGame/Scripts/Game/SkillSystem/SkillSystem.cs` | Server 通訊、play_bonus_use 指令 |
| `EventSystemGDK` | `External_Script/EventSystemGDK.cs` | 跨模組事件橋接 |
| `PlayBonusUseResultData` | `FH_PlayBonus/FH_PlayBonusData.cs` | Server 回應資料結構 |

---

## 5. 假魚與真魚的差異

| 面向 | 真魚 | 假魚 |
|-----|------|------|
| **生成來源** | Server f1 指令 | Client 端直接呼叫 |
| **SID** | 正數（Server 分配） | 負數或特殊值 |
| **游動行為** | 正常游動 | 靜止不動 (`isPause = true`) |
| **可攻擊性** | 可被子彈擊中 | 無敵狀態 (`isImmortal = true`) |
| **Layer** | Fish Layer | UI Layer |
| **管理清單** | `FishMaintainer.FishList` | `FishMaintainer.fakeFishList` |
| **Force** | 正常速度 | 0（不移動） |

---

## 6. SummonFakeFish 內部實作

```csharp
public void SummonFakeFish(int type, int fakeFishSid, Vector3 position, 
    Action<Fish2D> SpawnFinishCB = null, string sortingLayerName = "Fish", 
    int sortingOrder = -1, LayerMask layerMask = default, bool isUILayer = false)
{
    enumFishType initType = (enumFishType)type;
    
    // 1. 從 Object Pool 取得魚物件
    SpawnFish(initType, 0, delegate (GameObject _FishObj)
    {
        Fish2D _Fish = _FishObj.GetComponent<Fish2D>();
        
        // 2. 初始化魚種
        _Fish.Init((enumFishType)type, position, 0, 0);
        
        // 3. 設定假魚特殊狀態
        _Fish.FishStopMoving();           // 停止移動
        _Fish.isImmortal = true;          // 無敵
        _Fish.BackSwimCount = 0;          // 不回游
        _Fish.sid = fakeFishSid;          // 設定假 SID
        _Fish.isPause = true;             // 暫停狀態
        _Fish.SetForce(0);                // 不施加力
        _Fish.state = Fish2D.enumFishState.enumFishIdle;
        
        // 4. 設定渲染層級
        _Fish.SetSpriteSortLayerName(sortingLayerName);
        _Fish.SetSpriteSortLayerOrderBase(sortingOrder);
        _Fish.transform.SetLayerRecursively(layerMask);
        _Fish.m_SteerForOceanKing2D.isAction = false;
        
        // 5. 設定位置
        _Fish.craft.vPosition = position;
        _Fish.transform.position = position;
        
        // 6. 加入假魚清單
        fakeFishList.Add(_Fish);
        
        // 7. 執行傳送動畫
        if (_Fish != null)
        {
            TeleportFish(_Fish, position, false, 0, true, SpawnFinishCB);
        }
    });
}
```

---

## 7. 假魚的生命週期

```
┌──────────────┐
│   建立階段    │
├──────────────┤
│ SpawnFish()  │ ← 從 Object Pool 取得
│      ↓       │
│ Init()       │ ← 初始化魚種屬性
│      ↓       │
│ 設定假魚狀態  │ ← isImmortal, isPause, Force=0
│      ↓       │
│ fakeFishList │ ← 加入管理清單
│      ↓       │
│ TeleportFish │ ← 播放召喚動畫
│      ↓       │
│ SummonIn()   │ ← DOScale 縮放動畫
│      ↓       │
│ Callback     │ ← SpawnFinishCB 觸發
└──────────────┘
        │
        ▼
┌──────────────┐
│   演出階段    │
├──────────────┤
│ Feature 演出 │ ← 各魚種獨立處理
│ 報獎動畫     │
│ 金幣飛入     │
└──────────────┘
        │
        ▼
┌──────────────┐
│   銷毀階段    │
├──────────────┤
│ Despawn()    │ ← 魚種銷毀
│      ↓       │
│ fakeFishList │ ← 從清單移除
│   .Remove()  │
│      ↓       │
│ Object Pool  │ ← 回收至池中
└──────────────┘
```

---

## 8. 支援的假召魚種

目前 PlayBonus 系統支援以下魚種的假召：

| 魚種 | enumFishType | 處理方式 |
|-----|--------------|---------|
| 彌勒佛 | `enumType_Buddha` | `Buddha_Capture` 事件 |
| 招財貓 | `enumType_MalletMeow` | `SkillMalletMeow.OnInfoReceive()` |
| 無限轉輪 | `enumType_InfinityWheel` | `SkillInfinityWheel.OnInfoReceive()` |

---

## 9. 擴充新的假召魚種

若要新增支援假召的魚種，需修改以下位置：

### 9.1 SpawnFishFinishCallback

```csharp
// FH_PlayBonusManager.cs
private void SpawnFishFinishCallback(Fish2D fish, PlayBonusUseResultData data)
{
    switch (fish.type)
    {
        case Fish2D.enumFishType.enumType_Buddha:
            // 現有處理...
            break;
        
        // 新增魚種
        case Fish2D.enumFishType.enumType_NewFish:
            // 客製化初始化
            break;
    }
    KillFishBySID(fish, data);
}
```

### 9.2 KillFishBySID

```csharp
// FH_PlayBonusManager.cs
private async void KillFishBySID(Fish2D fish, PlayBonusUseResultData data)
{
    switch (fish.type)
    {
        case Fish2D.enumFishType.enumType_Buddha:
            // 現有處理...
            break;
        
        // 新增魚種
        case Fish2D.enumFishType.enumType_NewFish:
            // 發送對應事件或呼叫 Skill 方法
            FishHunter_EventManager.SendEvent(
                FishHunter_EventType.EventType.NewFish_Capture,
                fish.sid,
                // ... 其他參數
            );
            break;
    }
}
```

---

## 10. 注意事項

1. **SID 衝突**：假魚 SID 必須使用負數或特殊偏移，避免與真實魚衝突
2. **Layer 設定**：假魚通常在 UI Layer，確保不會被遊戲邏輯影響
3. **清單管理**：假魚 Despawn 時會自動從 `fakeFishList` 移除
4. **演出結束**：確保演出結束後解鎖玩家射擊（`UnlockPlayerShoot`）
5. **本家/他家**：他家玩家的假魚生成在螢幕外（`200000, 200000`），僅供流程判斷使用
