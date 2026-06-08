---
inclusion: manual
---

# DinoPinBall (恐龍系列) 怪物 Prefab 結構規範 (Monster Prefab Standard)

當使用者說「檢查恐龍 Prefab」、「驗證恐龍怪物結構」、「製作恐龍 Prefab」、「設定恐龍物件」、
「DinoPinBall Prefab 規範」時，啟動此規範。
本文件**僅適用於 DinoPinBall / DinoGame (3D 恐龍魚機)**。
OceanTreasure / ArkGame (2D 海王魚機) 的魚種 Prefab 規範請參閱未來建立的 `fishhunter_fish_prefabs_standard.md`。

本文件定義 DinoPinBall 怪物 Prefab 的標準結構、必要 Component、SerializeField 欄位規範，
以及使用 unity-cli 進行製作與檢查的標準操作流程。

> 本規範基於實際使用 unity-cli 分析以下怪物 Prefab 歸納而成：
> - **一般怪物**：Stoneage_Dilophosaurus（雙冠龍，PathAnimal）
> - **翻倍怪物**：Juras_Golden_Mammoth（黃金猛瑪象，JurasGoldenMammothCtrl）
> - **BOSS 怪物**：Stoneage_Tyrannosaurus（霸王龍，PathAnimal）
>
> **與海王系列的關鍵差異**：
> - 恐龍系列使用 3D 模型（FBX + SkinnedMeshRenderer），海王系列使用 2D Spine 骨骼動畫
> - 恐龍系列怪物基類為 `BaseMonster` → `PathAnimal` / `StaticAnimal`，海王系列為 `Fish2D`
> - 恐龍系列移動系統為 `ArkSteerBehavior` + `PathController`，海王系列為 `FishMaintainer` 管理
> - 恐龍系列事件系統為 `Dino_EventManager`，海王系列為 `FishHunter_EventManager`

---

## 1. Prefab Hierarchy 標準結構

所有怪物 Prefab 都必須遵循以下子物件層級：

```
{PrefabName} (Root)                    ← Tag: Monster, Layer: FISH_ZORDER
├── Shape/                             ← 模型容器（空 Transform）
│   └── {ModelName}/                   ← 3D 模型根節點（Animator）
│       ├── Bip001                     ← 骨骼根節點（Transform）
│       └── {MeshName}                 ← 網格（SkinnedMeshRenderer）
├── Shadow/                            ← 影子容器（空 Transform）
│   └── Obj                            ← 影子圖片（SpriteRenderer）
├── Colliders/                         ← 碰撞器容器（⚠️ Tag 必須設為 `Monster`）
│   ├── Collider_0                     ← 碰撞器（SphereCollider 或 BoxCollider，isTrigger=true）
│   └── [Collider_1..N]               ← BOSS/大型怪物可有多個碰撞器
├── Effects/                           ← 特效容器（空 Transform）
│   ├── Freeze (inactive)              ← 冰凍效果（MeshFilter + MeshRenderer）
│   └── ThunderTwoTarget (inactive)    ← 雷鳴目標點（空 Transform）
└── [UI/] (選用)                       ← 翻倍怪物/BOSS 等需要 UI 時才有
    └── Canvas/                        ← World Space Canvas
        └── {UIElement}                ← UI 元素（Image、Text 等）
```

### 1.1 Hierarchy 檢查清單

| 項目 | 必要性 | 檢查方式 (unity-cli) |
|------|--------|---------------------|
| Root Tag = `Monster` | ✅ 必要 | `get_gameobject_details` → `tag` |
| Root Layer = `FISH_ZORDER` (18) | ✅ 必要 | `get_gameobject_details` → `layer` |
| Colliders/ Tag = `Monster` | ✅ 必要 | 子彈 `OnTriggerEnter` 用 `CompareTag("Monster")` 判斷命中，Collider 所在物件的 Tag 必須是 `Monster` |
| Colliders/ isTrigger = true | ✅ 必要 | 子彈碰撞偵測用 `OnTriggerEnter`，Collider 必須設為 Trigger |
| Root Layer = `FISH_ZORDER` | ✅ 必要 | `get_gameobject_details` → `layer` |
| Shape/ 子物件存在 | ✅ 必要 | `get_gameobject_details` → children 含 `Shape` |
| Shadow/ 子物件存在 | ✅ 必要 | `get_gameobject_details` → children 含 `Shadow` |
| Shadow/Obj 有 SpriteRenderer | ✅ 必要 | `get_component_values` componentType=`SpriteRenderer` |
| Colliders/ 子物件存在 | ✅ 必要 | `get_gameobject_details` → children 含 `Colliders` |
| Colliders/ 下至少一個碰撞器 | ✅ 必要 | Collider_0 有 SphereCollider 或 BoxCollider |
| Effects/ 子物件存在 | ✅ 必要 | `get_gameobject_details` → children 含 `Effects` |
| Effects/Freeze 存在且 inactive | ✅ 必要 | `isActive: false` |
| Effects/ThunderTwoTarget 存在且 inactive | ✅ 必要 | `isActive: false` |
| UI/ 子物件（翻倍/BOSS 用） | ⚪ 選用 | 依怪物類型決定 |

### 1.2 怪物類型與 Hierarchy 差異

| 怪物類型 | Colliders 數量 | UI/ 子物件 | Effects/ 額外子物件 |
|----------|---------------|-----------|-------------------|
| 一般怪物 (A) | 1 個 | 無 | 無 |
| 翻倍怪物 (B) | 1 個 | 有（倍率 UI） | 有（GoldState 粒子等） |
| BOSS (E/G/H) | 多個（2~4） | 有（血條/進度條） | 有（各種特效） |
| 技能怪物 (C/D) | 1 個 | 視情況 | 視情況 |


---

## 2. Root 物件必要 Component 組成

所有怪物 Prefab 的 Root 物件上必須掛載以下 6 個 Component（順序固定）：

| # | Component | 類型 | 必要性 | 功能說明 |
|---|-----------|------|--------|---------|
| 1 | **Transform** | Unity 內建 | ✅ 必要 | 位置/旋轉/縮放 |
| 2 | **ArkSteerBehavior** | 自訂腳本 | ✅ 必要 | 轉向行為系統（移動引擎核心） |
| 3 | **SteerForTether** | 自訂腳本 | ✅ 必要 | 繫繩行為（限制移動範圍） |
| 4 | **怪物控制器腳本** | 自訂腳本 | ✅ 必要 | 繼承 PathAnimal / StaticAnimal / BaseMonster |
| 5 | **PathController** | 自訂腳本 | ✅ 必要 | 路徑控制器（沿路徑移動） |
| 6 | **Animator** | Unity 內建 | ⚪ 選用 | 根物件動畫控制器（部分怪物不需要，動畫由 Shape 子物件的 Animator 處理） |

### 2.1 怪物控制器腳本選擇

| 怪物類型 | 控制器腳本 | 說明 |
|----------|-----------|------|
| 一般怪物（無自訂行為） | `PathAnimal` | 直接使用基類，不需自訂控制器 |
| 翻倍怪物 | 自訂 Ctrl（繼承 `PathAnimal`） | 如 `JurasGoldenMammothCtrl` |
| BOSS（走 WeaponSystem） | `PathAnimal` 或自訂 Ctrl | 如霸王龍直接用 `PathAnimal` |
| BOSS（走 SkillSystem） | 自訂 Ctrl（繼承 `PathAnimal`） | 如 `BossGoblinRobot` |
| BOSS（特殊機制） | 自訂 Ctrl（繼承 `PathAnimal`） | 如 `BossSlimePathAnimal` |
| 靜態怪物 | 自訂 Ctrl（繼承 `StaticAnimal`） | 如 `JurasTreasureSakuraCtrl` |

---

## 3. SerializeField 欄位規範

以下欄位在 unity-cli `get_component_values` 回傳中以 `[SF]` 前綴標記。

### 3.1 通用必要欄位（所有怪物，繼承自 BaseMonster/PathAnimal）

#### 物件引用欄位

| 欄位名 | 類型 | 必要性 | 引用目標 | 說明 |
|--------|------|--------|---------|------|
| `m_shape` | `GameObject` | ✅ 必要 | `/{Root}/Shape` | 模型容器引用 |
| `m_monsterAnimator` | `Animator` | ✅ 必要 | Root 或 Shape 子物件上的 Animator | 整體動畫控制器（Root 無 Animator 時指向 Shape 下的 Animator） |
| `m_runAnimator` | `Animator` | ✅ 必要 | 模型上的 Animator | 跑步/移動動畫控制器 |
| `monsterController` | `BaseMonsterController` | ✅ 必要 | Root 上的 PathController | 移動控制器引用 |
| `m_lightingMaterial` | `Material` | ✅ 必要 | Lightning 材質檔案 | 閃電效果材質 |

#### 數值參數欄位

| 欄位名 | 類型 | 預設值 | 說明 |
|--------|------|--------|------|
| `m_crossSize` | `float` | 0.012 | 準心大小（一般怪物），BOSS 可較大（0.018） |
| `m_elecWaveSize` | `float` | 0.7 | 電擊波紋大小（一般怪物），BOSS 可較大（1.0） |
| `hit_ShakePower` | `float` | 0.14 | 受擊震動力度 |
| `m_deadDespawnDelayTime` | `float` | 5.0 | 死亡後回收延遲（秒） |
| `m_deadScaleTweenrDuration` | `float` | 0.3 | 死亡縮放動畫時長 |
| `m_delayDeadScaleTweenTime` | `float` | 0.8 | 延遲死亡縮放時間 |
| `overrideTurnTime` | `float` | 0.25 | 轉向時間覆寫 |
| `m_onHitEffectTime` | `float` | 0.0 | 受擊特效時間 |
| `m_onHitSound` | `string` | "" | 受擊音效名稱 |

#### 布林旗標欄位

| 欄位名 | 類型 | 預設值 | 說明 | 何時設為 true |
|--------|------|--------|------|-------------|
| `m_IsFeatureMonster` | `bool` | false | 是否為特殊魚種 | 翻倍怪物、BOSS |
| `m_IsEternalLife` | `bool` | false | 是否為永生魚種 | 永生怪物 |
| `m_IsImmuneToSpecialWeapon` | `bool` | false | 特殊武器免疫 | 翻倍怪物、部分 BOSS |
| `m_IsRunAnimatorParalysis` | `bool` | false | 是否執行麻痺動畫 | 有麻痺效果的怪物 |
| `m_isHaveDeadAnimation` | `bool` | false | 是否有死亡動畫 | 有自訂死亡動畫的怪物 |
| `isEventDino` | `bool` | false | 是否為活動怪物 | 活動限定怪物 |

#### 報獎類型欄位

| 欄位名 | 類型 | 說明 |
|--------|------|------|
| `m_specialAwardType` | `SpecialAwardType` | 特殊報獎類型，依怪物設定 |

**常用 SpecialAwardType 值**：
- `StoneAgeAwardSmall` — 石器時代小報獎（一般小怪）
- `StoneAgeAwardMedium` — 石器時代中報獎
- `StoneAgeAwardBig` — 石器時代大報獎（大型怪物）
- `JurasShowDoubleAward` — 侏羅紀翻倍報獎
- `ShowDeclareBoardAward` — 報獎面板
- `None` — 無特殊報獎

#### 可選引用欄位（部分怪物可為 null/unassigned）

| 欄位名 | 類型 | 說明 |
|--------|------|------|
| `m_alphaMaterial` | `Material` | 透明材質（特殊武器用） |
| `m_material` | `Material` | 主材質引用 |
| `m_materialOriginTexture` | `Texture` | 原始貼圖 |
| `m_onHitTexture` | `Texture` | 受擊貼圖 |
| `m_onKillTexture` | `Texture` | 死亡貼圖 |
| `m_onHitEffectGO` | `GameObject` | 受擊特效物件 |
| `m_hitTweenTargetTf` | `Transform` | 受擊動畫目標 |

### 3.2 Runtime 屬性（不需在 Inspector 設定，由程式碼控制）

| 屬性名 | 類型 | 說明 |
|--------|------|------|
| `Sid` | `int` | 怪物唯一 SID（Server 分配） |
| `MonsterType` | `int` | 怪物類型代號 |
| `MonsterSubType` | `int` | 子類型 |
| `State` | `EnumMonsterState` | 當前狀態（MonsterIdle/Moving/Geting/Leaving/Feature） |
| `Lockable` | `bool` | 是否可鎖定 |
| `IsKill` | `bool` | 是否已被擊殺 |
| `IsTransparent` | `bool` | 是否透明 |
| `ExtraData` | `JSON` | f1 封包的額外資料 |
| `VfxTargetObj` | `GameObject` | 雷鳴目標物件（自動指向 Effects/ThunderTwoTarget） |


### 3.3 翻倍怪物額外欄位（選項 B）

翻倍怪物的自訂控制器（如 `JurasGoldenMammothCtrl`）除了通用欄位外，還需要以下額外欄位：

| 欄位名 | 類型 | 必要性 | 說明 |
|--------|------|--------|------|
| `arkSteerBehavior` | `ArkSteerBehavior` | ✅ | Root 上的 ArkSteerBehavior 引用 |
| `OddsRoot` | `Transform` | ✅ | 倍率 UI 根節點 |
| `OddsText` | `UGUI_ImageText` | ✅ | 倍率文字元件 |
| `OddsTextHeight` | `int[]` | ✅ | 倍率文字高度陣列（依位數） |
| `canvasGroup` | `CanvasGroup` | ✅ | UI 透明度控制 |
| `uICanvas` | `Canvas` | ✅ | UI Canvas 引用 |
| `goldRoot` | `Transform` | ⚪ 視情況 | 黃金特效根節點 |
| `goldAnchorPos` | `Transform` | ⚪ 視情況 | 黃金特效錨點 |
| `goldAnimator` | `Animator` | ⚪ 視情況 | 黃金特效動畫 |
| `circleObject` | `Transform` | ⚪ 視情況 | 圓形特效 |
| `mammothAnimator` | `Animator` | ⚪ 視情況 | 專用動畫控制器 |
| `m_ChangeOddsSpeed` | `float` | ✅ | 倍率變化速度 |
| `moveSpeed` | `float` | ✅ | 移動速度 |
| `radius` | `float` | ✅ | 半徑 |

**翻倍怪物必須設定的旗標**：
- `m_IsFeatureMonster` = `true`
- `m_IsImmuneToSpecialWeapon` = `true`

---

## 4. ArkSteerBehavior 移動參數規範

ArkSteerBehavior 控制怪物的移動行為，以下是關鍵參數：

| 參數 | 類型 | 預設值 | 說明 | 調整建議 |
|------|------|--------|------|---------|
| `MaxSpeed` | `float` | 5.7 | 最大移動速度 | 小怪 4~6，大怪 3~5，BOSS 2~4 |
| `MaxForce` | `float` | 6.0 | 最大轉向力 | 與 MaxSpeed 同比例調整 |
| `TurnTime` | `float` | 0.25 | 轉向時間 | 越大轉向越慢 |
| `Mass` | `float` | 1.0 | 質量 | 通常不需修改 |
| `ArrivalRadius` | `float` | 0.25 | 到達半徑 | 通常不需修改 |
| `AllowedMovementAxes` | `Vector3` | (1,0,1) | 允許移動軸 | XZ 平面移動，Y 軸鎖定 |
| `_accelerationRate` | `float` | 1.0 | 加速率 | 通常不需修改 |
| `_decelerationRate` | `float` | 0.5 | 減速率 | 通常不需修改 |

> **注意**：新增怪物時，`MaxSpeed` 和 `MaxForce` 應參考同類型參考怪物的 MonsterParameterDataSO 設定值。
> 若 SO 中有指定 `maxSpeed` / `maxForce`，Runtime 會覆寫 Prefab 上的值。

---

## 5. 怪物類型與旗標設定對照表

| 怪物類型 | m_IsFeatureMonster | m_IsEternalLife | m_IsImmuneToSpecialWeapon | m_specialAwardType |
|----------|-------------------|-----------------|--------------------------|-------------------|
| 一般怪物 (A) | `false` | `false` | `false` | `StoneAgeAwardSmall` 等 |
| 翻倍怪物 (B) | `true` | `false` | `true` | `None` 或自訂 |
| 技能怪物 (C/D) | `true` | 視情況 | 視情況 | 視情況 |
| BOSS-WeaponSystem (E) | `true` | `false` | 視情況 | 自訂 |
| BOSS-SkillSystem (G) | `true` | `false` | 視情況 | 自訂 |
| BOSS-特殊機制 (H) | `true` | `false` | 視情況 | 自訂 |
| 空殼模板 (F) | `false` | `false` | `false` | `None` |


---

## 6. 資源資料夾結構規範

### 6.1 標準資料夾結構

```
Assets/FishtHunter_Res/CrazyDino/Bundle/Monster/{MonsterFolder}/
├── Animation/                    ← 動畫 Clip (.anim)
│   └── Monster_{Name}_Walk.anim
├── Animator/                     ← 動畫控制器 (.controller)
│   └── Monster_{Name}_Animator.controller
├── Material/                     ← 材質 (.mat)
│   ├── Monster_{Name}_01.mat           ← 主材質
│   ├── Monster_{Name}_01_New.mat       ← 新版材質（如有）
│   ├── Monster_{Name}_01_Lightning.mat ← 閃電材質 ✅ 必要
│   ├── Monster_{Name}_02.mat           ← 第二材質（如有）
│   └── Monster_{Name}_02_Lightning.mat ← 第二閃電材質（如有）
├── Model/                        ← 3D 模型 (.FBX)
│   └── Monster_{Name}.FBX
├── Texture/                      ← 貼圖 (.png)
│   ├── Monster_{Name}_01.png
│   └── Monster_{Name}_02.png
├── [Audio/]                      ← 音效（選用）
├── [Atlas/]                      ← 圖集（選用，翻倍怪物 UI 用）
├── [Prefab/]                     ← 子 Prefab（選用，BOSS 演出用）
├── [Timeline/]                   ← Timeline（選用，BOSS 運鏡用）
├── {ModelName}.prefab            ← 模型 Prefab（純模型，無遊戲邏輯）
└── {廳館}_{MonsterName}.prefab   ← 怪物主 Prefab ✅ 必要（掛載遊戲腳本）
```

### 6.2 Prefab 命名規則

| Prefab 類型 | 命名格式 | 範例 |
|------------|---------|------|
| 模型 Prefab | `{ModelName}.prefab` | `Dilophosaurus.prefab` |
| 怪物主 Prefab | `{廳館}_{MonsterName}.prefab` | `Stoneage_Dilophosaurus.prefab` |

**廳館前綴對照**：
- `Dino_` / 無前綴 — Dino 廳（舊版）
- `Juras_` — Juras 廳
- `Stoneage_` — Stoneage 廳
- `Robin_` — Robin 廳

> **注意**：新增怪物一律使用統一結構（不加廳館前綴於資料夾名稱），
> 廳館前綴僅用於 Prefab 檔名。

### 6.3 Lightning 材質必要性

每個怪物**必須**有對應的 Lightning 材質（`*_Lightning.mat`），
此材質被 `m_lightingMaterial` 欄位引用，用於閃電效果。
若缺少此材質，閃電武器命中時將無法顯示效果。

---

## 7. unity-cli 操作標準流程

### 7.1 檢查既有怪物 Prefab

```bash
# 步驟 1: 打開 Prefab
unity-cli raw open_prefab --params-file {"prefabPath":"Assets/.../MonsterName.prefab"}

# 步驟 2: 查看 Hierarchy 結構
unity-cli raw get_gameobject_details --params-file {"gameObjectName":"MonsterName","includeChildren":true}
# → 確認：Tag=Monster, Layer=FISH_ZORDER
# → 確認：子物件包含 Shape, Shadow, Colliders, Effects
# → 確認：Effects/Freeze 和 ThunderTwoTarget 為 inactive

# 步驟 3: 檢視怪物控制器欄位
unity-cli raw get_component_values --params-file {"gameObjectName":"MonsterName","componentType":"控制器類名"}
# → 確認所有 [SF] 必要欄位已正確設定
# → 確認物件引用（m_shape, m_monsterAnimator, m_runAnimator, monsterController, m_lightingMaterial）非 null
# → 確認布林旗標符合怪物類型

# 步驟 4: 檢視移動參數
unity-cli raw get_component_values --params-file {"gameObjectName":"MonsterName","componentType":"ArkSteerBehavior"}
# → 確認 MaxSpeed, MaxForce 合理

# 步驟 5: 退出（不儲存）
unity-cli raw exit_prefab_mode --params-file {}
```

### 7.2 製作新怪物 Prefab（從參考怪物複製後）

```bash
# 步驟 1: 打開複製後的 Prefab
unity-cli raw open_prefab --params-file {"prefabPath":"Assets/.../NewMonster.prefab"}

# 步驟 2: 確認 Hierarchy 結構完整
unity-cli raw get_gameobject_details --params-file {"gameObjectName":"NewMonster","includeChildren":true}

# 步驟 3: 設定怪物控制器的物件引用
# 3a. 設定 m_lightingMaterial（指向新怪物自己的 Lightning 材質）
unity-cli raw set_component_field --params-file {
  "gameObjectPath":"/NewMonster",
  "componentType":"PathAnimal",
  "fieldPath":"m_lightingMaterial",
  "value":"Assets/.../Material/Monster_NewMonster_01_Lightning.mat",
  "valueType":"objectReference"
}

# 3b. 設定 m_shape（指向 Shape 子物件）
unity-cli raw set_component_field --params-file {
  "gameObjectPath":"/NewMonster",
  "componentType":"PathAnimal",
  "fieldPath":"m_shape",
  "value":"/NewMonster/Shape",
  "valueType":"objectReference"
}

# 3c. 設定其他必要引用...（m_monsterAnimator, m_runAnimator, monsterController）

# 步驟 4: 設定布林旗標（依怪物類型）
unity-cli raw set_component_field --params-file {
  "gameObjectPath":"/NewMonster",
  "componentType":"PathAnimal",
  "fieldPath":"m_IsFeatureMonster",
  "value":false
}

# 步驟 5: 設定報獎類型
unity-cli raw set_component_field --params-file {
  "gameObjectPath":"/NewMonster",
  "componentType":"PathAnimal",
  "fieldPath":"m_specialAwardType",
  "value":"StoneAgeAwardSmall"
}

# 步驟 6: 儲存並退出
unity-cli raw save_prefab --params-file {}
unity-cli raw exit_prefab_mode --params-file {}
```

### 7.3 unity-cli 指令執行規範（⚠️ 強制）

**所有 unity-cli 指令必須使用單行 `;` 串接格式**，將寫檔和執行合併在同一個 shell 呼叫中，禁止分行寫法。

#### ✅ 正確寫法（單行串接，一次 tool call 完成）

```powershell
$json = '{"prefabPath":"Assets/.../Monster.prefab"}'; [System.IO.File]::WriteAllText("params.json", $json, [System.Text.UTF8Encoding]::new($false)); & "$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe" raw open_prefab --params-file params.json
```

```powershell
$json = '{"gameObjectName":"MonsterName","includeChildren":true}'; [System.IO.File]::WriteAllText("params.json", $json, [System.Text.UTF8Encoding]::new($false)); & "$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe" raw get_gameobject_details --params-file params.json
```

```powershell
$json = '{"gameObjectPath":"/Monster","componentType":"PathAnimal","fieldPath":"m_IsEternalLife","value":true}'; [System.IO.File]::WriteAllText("params.json", $json, [System.Text.UTF8Encoding]::new($false)); & "$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe" raw set_component_field --params-file params.json
```

```powershell
$json = '{}'; [System.IO.File]::WriteAllText("params.json", $json, [System.Text.UTF8Encoding]::new($false)); & "$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe" raw save_prefab --params-file params.json
```

#### ❌ 禁止寫法

```powershell
# ❌ 分行寫法 — 會被拆成多次執行，浪費 tool call
$json = '{"prefabPath":"Assets/.../Monster.prefab"}'
[System.IO.File]::WriteAllText("params.json", $json, [System.Text.UTF8Encoding]::new($false))
& "$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe" raw open_prefab --params-file params.json

# ❌ 使用 Out-File / Set-Content — 會加 BOM 導致 JSON 解析失敗
$json | Out-File params.json
$json | Set-Content params.json
```

#### 規則摘要

| 規則 | 說明 |
|------|------|
| 單行 `;` 串接 | 寫檔 + 執行必須在同一行，用 `;` 分隔 |
| UTF-8 無 BOM | 使用 `[System.Text.UTF8Encoding]::new($false)` |
| 禁止 Out-File / Set-Content | 會加 BOM 或改編碼 |
| 暫存檔名 | 統一使用 `params.json` |
| 完成後清理 | 所有 unity-cli 操作結束後刪除 `params.json` |


---

## 8. 驗證檢查清單 (Validation Checklist)

使用 unity-cli 檢查怪物 Prefab 時，依序驗證以下項目：

### 8.1 結構驗證

- [ ] Root Tag = `Monster`
- [ ] Root Layer = `FISH_ZORDER`
- [ ] Root 有 5 個必要 Component（Transform, ArkSteerBehavior, SteerForTether, 怪物控制器, PathController）+ 選用 Animator
- [ ] Shape/ 子物件存在且 active
- [ ] Shape/ 下有模型子物件（含 Animator + SkinnedMeshRenderer）
- [ ] Shadow/ 子物件存在且 active
- [ ] Shadow/Obj 有 SpriteRenderer
- [ ] Colliders/ 子物件存在且 active
- [ ] Colliders/ 下至少有一個碰撞器（SphereCollider 或 BoxCollider）
- [ ] Effects/ 子物件存在且 active
- [ ] Effects/Freeze 存在且 **inactive**
- [ ] Effects/ThunderTwoTarget 存在且 **inactive**

### 8.2 必要引用驗證

- [ ] `[SF]m_shape` → 指向 `/{Root}/Shape`（非 null）
- [ ] `[SF]m_monsterAnimator` → 指向 Root 上的 Animator（非 null）
- [ ] `[SF]m_runAnimator` → 指向模型上的 Animator（非 null）
- [ ] `[SF]monsterController` → 指向 Root 上的 PathController（非 null）
- [ ] `[SF]m_lightingMaterial` → 指向 Lightning 材質檔案（非 null）
- [ ] `VfxTargetObj` → 指向 Effects/ThunderTwoTarget（非 null）

### 8.3 參數合理性驗證

- [ ] `m_crossSize` > 0（一般 0.012，BOSS 0.018）
- [ ] `m_elecWaveSize` > 0（一般 0.7，BOSS 1.0）
- [ ] `hit_ShakePower` > 0（通常 0.14）
- [ ] `m_deadDespawnDelayTime` > 0（通常 5.0）
- [ ] ArkSteerBehavior.`MaxSpeed` > 0
- [ ] ArkSteerBehavior.`MaxForce` > 0
- [ ] ArkSteerBehavior.`AllowedMovementAxes` = (1, 0, 1)

### 8.4 怪物類型一致性驗證

- [ ] `m_IsFeatureMonster` 符合怪物類型（翻倍/BOSS = true，一般 = false）
- [ ] `m_IsImmuneToSpecialWeapon` 符合怪物類型（翻倍 = true）
- [ ] `m_specialAwardType` 已設定且非預設值（除非刻意為 None）

### 8.5 翻倍怪物額外驗證（選項 B）

- [ ] 有 UI/ 子物件
- [ ] `[SF]OddsRoot` 非 null
- [ ] `[SF]OddsText` 非 null
- [ ] `[SF]canvasGroup` 非 null
- [ ] `[SF]uICanvas` 非 null
- [ ] `[SF]arkSteerBehavior` 指向 Root 上的 ArkSteerBehavior

---

## 9. 常見問題與排查

### 9.1 get_component_values 回傳 error 的欄位

```json
"[SF]m_alphaMaterial": {
  "error": "Failed to serialize: The variable m_alphaMaterial of PathAnimal has not been assigned."
}
```

**含義**：該 SerializeField 欄位在 Inspector 中未指定（null）。
**是否正常**：
- `m_alphaMaterial`、`m_material`、`m_onHitTexture`、`m_onKillTexture`、`m_onHitEffectGO`、`m_hitTweenTargetTf` → **正常**，這些是可選欄位
- `m_shape`、`m_monsterAnimator`、`m_runAnimator`、`monsterController`、`m_lightingMaterial` → **異常**，這些是必要欄位，必須指定

### 9.2 怪物控制器類名不確定

使用 `get_gameobject_details` 查看 Root 的 components 列表，
排除 Transform、ArkSteerBehavior、SteerForTether、PathController、Animator 後，
剩下的就是怪物控制器腳本。

### 9.3 Prefab 有兩個（模型 Prefab vs 怪物主 Prefab）

- **模型 Prefab**（如 `Dilophosaurus.prefab`）：純 3D 模型，無遊戲邏輯腳本
- **怪物主 Prefab**（如 `Stoneage_Dilophosaurus.prefab`）：掛載遊戲腳本，是實際使用的 Prefab

操作時應打開**怪物主 Prefab**（帶廳館前綴的那個）。

---

## 10. 參考怪物速查表

| 怪物類型 | 參考怪物 | Prefab 路徑 | 控制器腳本 |
|----------|---------|------------|-----------|
| 一般怪物 (A) | 雙冠龍 | `.../Dilophosaurus/Stoneage_Dilophosaurus.prefab` | `PathAnimal` |
| 翻倍怪物 (B) | 黃金猛瑪象 | `.../Juras_Golden_Mammoth/Juras_Golden_Mammoth.prefab` | `JurasGoldenMammothCtrl` |
| BOSS-WeaponSystem (E) | 霸王龍 | `.../Tyrannosaurus/Stoneage_Tyrannosaurus.prefab` | `PathAnimal` |
| BOSS-SkillSystem (G) | 阿力肯 | `.../BossAriken/...` | 自訂 Ctrl |
| BOSS-特殊機制 (H) | 史萊姆王 | `.../BossSlime/...` | `BossSlimePathAnimal` |

---

## 附錄 A：unity-cli 指令速查

| 操作 | 指令 | 關鍵參數 |
|------|------|---------|
| 打開 Prefab | `raw open_prefab` | `prefabPath` |
| 查看 Hierarchy | `raw get_gameobject_details` | `gameObjectName`, `includeChildren` |
| 查看 Component 欄位 | `raw get_component_values` | `gameObjectName`, `componentType` |
| 設定欄位值 | `raw set_component_field` | `gameObjectPath`, `componentType`, `fieldPath`, `value` |
| 設定物件引用 | `raw set_component_field` | 同上 + `valueType: "objectReference"` |
| 儲存 Prefab | `raw save_prefab` | `{}` |
| 退出 Prefab 模式 | `raw exit_prefab_mode` | `{}` |
| 尋找物件 | `raw find_gameobject` | `name` |
| 場景分析 | `raw analyze_scene_contents` | `includeInactive` |
