---
inclusion: manual
---

# OceanTreasure (海王系列) 魚種 Prefab 結構規範 (Fish Prefab Standard)

當使用者說「檢查海王 Prefab」、「驗證魚種結構」、「製作魚種 Prefab」、「設定魚種物件」、
「OceanTreasure Prefab 規範」時，啟動此規範。
本文件**僅適用於 OceanTreasure / ArkGame (2D 海王魚機)**。
DinoPinBall / DinoGame (3D 恐龍魚機) 的怪物 Prefab 規範請參閱 `dinopinball_monseter_prefabs_standard.md`。

本文件定義 OceanTreasure 魚種 Prefab 的標準結構、必要 Component、子物件規範，
以及使用 unity-cli 進行製作與檢查的標準操作流程。

---

## ⚠️ 標準架構強制規則（Agent 必讀）

> **所有新增的海王魚種 Prefab 一律使用本文件定義的標準架構，無例外。**

### Agent 執行規則

1. **新增魚種時不需要判斷「要用哪種架構」**，直接套用本文件的標準架構
2. **Agent 應主動建議使用標準架構**：在新增魚種流程中，向使用者說明將使用標準架構，並簡述標準架構的 Hierarchy 結構（Root → Shape → freeze/shadow/maskroot/collider/Paralysis）
3. **除非使用者明確表示不需要標準架構**（例如「這隻魚不用標準架構」、「用特殊結構」），否則一律套用
4. **舊有魚種不主動重構**：已存在的魚種 Prefab 維持原樣，不需要也不應該主動改為標準架構。僅在使用者明確要求時才進行舊魚種的架構調整

### Agent 建議話術範例

在新增魚種流程中，Agent 應主動告知：

> 「新增的魚種將使用標準 Prefab 架構：Root（Fish2D + 物理 + 路徑）→ {FishName}Shape（Handler 群）→ freeze / shadow / maskroot/main / collider / Paralysis。如果有特殊需求需要調整結構，請告訴我。」

### 適用範圍

| 情境 | 是否套用標準架構 |
|------|----------------|
| 新增任何類型的魚種（A~F） | ✅ 一律套用 |
| 複製參考魚種建立新魚種 | ✅ 確認符合標準架構 |
| 檢查/驗證既有魚種 | ⚠️ 僅報告差異，不主動重構 |
| 使用者明確說不用標準架構 | ❌ 依使用者指示 |

---

> 本規範基於實際使用 unity-cli 分析以下魚種 Prefab 歸納而成：
> - **一般魚種**：GoldenBat（金蝙蝠，Fish2D，Sprite 動畫）
> - **DoubleFish 翻倍魚**：KillerWhale（殺人鯨，Fish2D + FishDialogue + Thunder2BonusChoose，Spine 動畫）
>
> **與恐龍系列的關鍵差異**：
> - 海王系列使用 2D 渲染（Spine `SkeletonAnimation` 或 Sprite + `Animator`），恐龍系列使用 3D 模型（FBX + SkinnedMeshRenderer）
> - 海王系列魚種基類為 `Fish2D`，恐龍系列為 `BaseMonster` → `PathAnimal` / `StaticAnimal`
> - 海王系列移動系統為 `FishRigidBody2D` + `SteerForOceanKing2D` + `PathBehavior`，恐龍系列為 `ArkSteerBehavior` + `PathController`
> - 海王系列碰撞使用 `RadiusChecker`（自訂半徑檢測），恐龍系列使用 Unity `SphereCollider` / `BoxCollider`
> - 海王系列事件系統為 `FishHunter_EventManager`，恐龍系列為 `Dino_EventManager`
> - 海王系列 Tag = `Fish`，恐龍系列 Tag = `Monster`

---

## 1. Prefab Hierarchy 標準結構

所有魚種 Prefab 都遵循以下子物件層級：

```
{FishName} (Root)                          ← Tag: Fish, Layer: FISH_ZORDER
└── Shape/                                 ← 外觀容器（Handler 腳本群，命名不限）
    ├── freeze (inactive)                  ← 冰凍效果
    │   ├── freezeSprite                   ← 冰凍圖片（SpriteRenderer）
    │   └── Freeze_Effect_{S|L}            ← 冰凍粒子（PerformanceChecker）
    ├── shadow                             ← 影子（SpriteFollower + SpriteRenderer）
    ├── maskroot/                           ← 遮罩根節點
    │   └── main                           ← 主要視覺（見 1.1 渲染方式）
    ├── collider/                           ← 碰撞器容器
    │   ├── collider                       ← 碰撞檢測（RadiusChecker）
    │   └── [collider (1)]                 ← 大型魚種可有多個碰撞器
    └── Paralysis (inactive)               ← 麻痺效果
        └── Effect_Paralysis               ← 麻痺動畫（Animator）
```

### 1.1 main 節點的兩種渲染方式

| 渲染方式 | Component 組成 | 適用場景 | 範例 |
|----------|---------------|---------|------|
| **Sprite 動畫** | `SpriteRenderer` + `Animator` + `SpriteFlicker` + `FishAnimationEventListener` | 簡單魚種、序列幀動畫 | GoldenBat（金蝙蝠） |
| **Spine 骨骼動畫** | `MeshFilter` + `MeshRenderer` + `SkeletonAnimation` + `SpriteFlicker` | 複雜魚種、骨骼動畫 | KillerWhale（殺人鯨） |

### 1.2 Hierarchy 檢查清單

| 項目 | 必要性 | 檢查方式 (unity-cli) |
|------|--------|---------------------|
| Root Tag = `Fish` | ✅ 必要 | `get_gameobject_details` → `tag` |
| Root Layer = `FISH_ZORDER` | ✅ 必要 | `get_gameobject_details` → `layer` |
| Shape 子物件存在（命名不限） | ✅ 必要 | `get_gameobject_details` → children |
| freeze/ 存在且 **inactive** | ✅ 必要 | `isActive: false` |
| freeze/freezeSprite 有 SpriteRenderer | ✅ 必要 | 冰凍效果圖片 |
| shadow 存在（含 SpriteFollower + SpriteRenderer） | ✅ 必要 | 影子跟隨 |
| maskroot/ 存在 | ✅ 必要 | 遮罩根節點 |
| maskroot/main 存在（含渲染 Component） | ✅ 必要 | 主要視覺 |
| collider/ 存在 | ✅ 必要 | 碰撞容器 |
| collider/ 下至少一個 RadiusChecker | ✅ 必要 | 碰撞檢測 |
| Paralysis/ 存在且 **inactive** | ✅ 必要 | 麻痺效果 |

### 1.3 Fish2D SerializeField 引用檢查（重點）

> **Shape 的命名不重要**，重點是 Fish2D（或其子類）的 `[SerializeField]` 引用是否正確拉到 Spine / Sprite 的顯示 Root。

使用 `get_component_values` 檢查 Fish2D 子類的 `[SF]` 前綴欄位，確認以下引用：

| SerializeField 欄位 | 應引用目標 | 說明 |
|---------------------|-----------|------|
| `[SF]fishSpine`（Spine 魚種） | Spine 顯示 Root 的 `SkeletonAnimation` | 魚種的主要 Spine 動畫元件，必須指向 maskroot/main 下的 Spine 節點 |
| `[SF]m_ColliderHandler` | Shape 上的 `FishColliderHandler` | 碰撞器管理 Handler |
| 其他子類自訂 `[SF]` 欄位 | 依魚種類型而定 | 如 `[SF]uiRoot`、`[SF]uiBoard`、`[SF]upgradeEffect` 等 |

**檢查原則**：
1. **Spine 魚種**：`[SF]fishSpine` 必須指向有 `SkeletonAnimation` 的 GameObject（通常是 maskroot/main 或其子物件）
2. **Sprite 魚種**：對應的 Animator / SpriteRenderer 引用必須指向 maskroot/main
3. **所有 `[SF]` 欄位不應為 null**（除非該欄位確實是選用的）
4. **有 error 訊息的欄位**（如 `"has not been assigned"`）代表 Inspector 中未拉引用，需要確認是否為問題

### 1.4 與恐龍系列 Hierarchy 對照

| 功能 | 海王系列 | 恐龍系列 |
|------|---------|---------|
| 外觀容器 | Shape（命名不限，含 Handler 腳本） | `Shape/`（空 Transform） |
| 模型/動畫 | `maskroot/main`（Spine 或 Sprite） | `Shape/{ModelName}/`（3D FBX） |
| 影子 | `shadow`（在 Shape 內，SpriteFollower） | `Shadow/Obj`（獨立子物件） |
| 碰撞 | `collider/`（RadiusChecker） | `Colliders/`（SphereCollider/BoxCollider） |
| 冰凍效果 | `freeze/`（在 Shape 內） | `Effects/Freeze`（獨立子物件） |
| 麻痺效果 | `Paralysis/`（在 Shape 內） | 無獨立節點（由 BaseMonster 控制） |
| 雷鳴目標 | 無獨立節點 | `Effects/ThunderTwoTarget` |


---

## 2. Root 物件必要 Component 組成

### 2.1 所有魚種共通 Component（Root 上）

| # | Component | 類型 | 功能說明 |
|---|-----------|------|---------|
| 1 | **Transform** | Unity 內建 | 位置/旋轉/縮放 |
| 2 | **Fish2D** | 自訂腳本 | 魚種基類，定義類型、狀態、行為 |
| 3 | **FishRigidBody2D** | 自訂腳本 | 2D 物理模擬（質量、速度、慣性） |
| 4 | **SteerForOceanKing2D** | 自訂腳本 | 轉向行為（海王專用） |
| 5 | **PathBehavior** | 自訂腳本 | 路徑行為控制 |

### 2.2 依魚種類型額外 Component（Root 上）

| 魚種類型 | 額外 Component | 說明 |
|----------|---------------|------|
| DoubleFish 翻倍魚 | `FishDialogue` | 翻倍魚對話/演出控制 |
| DoubleFish 翻倍魚 | `Thunder2BonusChoose` | 雷鳴二獎勵選擇 |
| 技能魚種 | 無額外（技能邏輯在 SkillXxxObject 中） | — |
| 一般魚種 | 無額外 | — |

### 2.3 Shape 子物件上的 Handler Component

`{FishName}Shape`（或其他命名的 Shape）子物件上掛載以下 Handler 腳本（所有魚種共通）：

| Component | 功能 |
|-----------|------|
| `SpriteLayerHandler` | Sprite 圖層排序管理 |
| `FishAnimatorHandler` | 動畫狀態管理 |
| `FishMaskStatusHandler` | 遮罩狀態管理 |
| `FishColliderHandler` | 碰撞器管理 |

---

## 3. Fish2D 關鍵屬性

Fish2D 是所有海王魚種的基類，以下是 unity-cli 可見的關鍵屬性：

### 3.1 類型與狀態屬性（Runtime 設定）

| 屬性名 | 類型 | 說明 |
|--------|------|------|
| `type` | `enumFishType` | 魚種類型（如 `enumType_GoldenBat`） |
| `kind` | `enumFishType` | 魚種種類（通常與 type 相同） |
| `state` | `enumFishState` | 當前狀態（`enumFishIdle` / `enumFishSwim` 等） |
| `SubType` | `enumFishSubType` | 子類型（`SpecialFish_None` 為預設） |
| `group` | `int` | 魚群編號 |
| `rate` | `float` | 速率 |
| `tick` | `float` | 計時器 |

### 3.2 物理屬性（FishRigidBody2D）

| 屬性名 | 類型 | 說明 |
|--------|------|------|
| `fMass` | `float` | 質量 |
| `fSpeed` | `float` | 速度 |
| `fOrientation` | `float` | 朝向角度 |
| `fInertia` | `float` | 慣性 |
| `vPosition` | `Vector3` | 位置 |
| `vVelocity` | `Vector3` | 速度向量 |
| `vShiftPosition` | `Vector3` | 偏移位置（螢幕座標偏移） |
| `SeparationDis` | `float` | 分離距離 |
| `SteerForce` | `float` | 轉向力 |
| `ThrustForce` | `float` | 推進力 |

---

## 4. 魚種類型與 Prefab 差異對照表

| 魚種類型 | Root 額外 Component | main 渲染方式 | collider 數量 | 範例 |
|----------|-------------------|--------------|--------------|------|
| 一般魚種 (A) | 無 | Sprite 或 Spine | 1 個 | GoldenBat |
| DoubleFish (B) | `FishDialogue` + `Thunder2BonusChoose` | Spine | 1~2 個 | KillerWhale |
| 特殊狀態魚 (C) | 無 | Spine | 1 個 | SamuraiFish |
| 技能魚種-傳統 (D) | 無 | Spine | 1 個 | FireStorm |
| 技能魚種-通用 (E) | 無 | Spine | 1 個 | VermilionBird |
| 空殼模板 (F) | 無 | 視情況 | 1 個 | — |

---

## 5. 資源資料夾結構規範

### 5.1 標準資料夾結構

海王系列的資料夾結構較扁平，所有資源放在同一層：

```
Assets/FishtHunter_Res/ArkGame/Fishes/{FishName}/
├── {FishName}.prefab                    ← 魚種 Prefab ✅ 必要
├── {FishName}Animator.controller        ← 動畫控制器（Sprite 動畫用）
├── {FishName}Swim.anim                  ← 游泳動畫 Clip
├── {FishName}_Atlas.png                 ← 圖集貼圖
├── {FishName}_Atlas.tpsheet             ← 圖集設定
├── [Spine 資源]                         ← Spine 魚種用
│   ├── {FishName}_SkeletonData.asset
│   ├── {FishName}_Atlas.atlas.txt
│   └── {FishName}_Atlas.png
├── [Script/]                            ← 自訂控制器腳本（選用）
├── [Feature/]                           ← 演出 Prefab（選用）
└── [Audio/]                             ← 音效（選用）
```

### 5.2 Prefab 命名規則

| 項目 | 命名格式 | 範例 |
|------|---------|------|
| 魚種 Prefab | `{FishName}.prefab` | `GoldenBat.prefab` |
| 動畫控制器 | `{FishName}Animator.controller` | `GoldenBatAnimator.controller` |
| 游泳動畫 | `{FishName}Swim.anim` | `GoldenBatSwim.anim` |
| 圖集 | `{FishName}_Atlas.png` | `GoldenBat_Atlas.png` |

> **注意**：海王系列不像恐龍系列有廳館前綴，Prefab 直接以魚種名稱命名。

### 5.3 AssetBundle 同名資源陷阱

**同一個 AssetBundle 內，不同類型的資源檔案不可與 Prefab 同名**。
若 Spine 資源的圖檔與 Prefab 同名，需將圖檔改名（如加 `_Atlas` 後綴）或放到不同 bundle。

---

## 6. unity-cli 操作標準流程

### 6.1 檢查既有魚種 Prefab

```bash
# 步驟 1: 打開 Prefab
unity-cli raw open_prefab --params-file {"prefabPath":"Assets/FishtHunter_Res/ArkGame/Fishes/{FishName}/{FishName}.prefab"}

# 步驟 2: 查看 Hierarchy 結構
unity-cli raw get_gameobject_details --params-file {"gameObjectName":"{FishName}","includeChildren":true}
# → 確認：Tag=Fish, Layer=FISH_ZORDER
# → 確認：有 Shape 子物件（命名不限）
# → 確認：Shape 內含 freeze(inactive), shadow, maskroot/main, collider, Paralysis(inactive)

# 步驟 3: 檢視 Fish2D SerializeField 引用（重點）
unity-cli raw get_component_values --params-file {"gameObjectName":"{FishName}","componentType":"{FishCtrlName}"}
# → 檢查所有 [SF] 前綴欄位
# → 確認 [SF]fishSpine 指向正確的 Spine 顯示 Root（SkeletonAnimation 所在的 GameObject）
# → 確認 [SF]m_ColliderHandler 不為 null
# → 確認沒有 "has not been assigned" 錯誤訊息

# 步驟 4: 檢視 Shape Handler
unity-cli raw get_component_values --params-file {"gameObjectName":"{ShapeName}","componentType":"FishColliderHandler"}
# → 確認 Handler 腳本存在

# 步驟 5: 退出（不儲存）
unity-cli raw exit_prefab_mode --params-file {}
```

### 6.2 辨識渲染方式

```bash
# 檢查 main 節點的 Component
unity-cli raw get_gameobject_details --params-file {"gameObjectName":"main"}
# → 若有 SkeletonAnimation → Spine 骨骼動畫
# → 若有 SpriteRenderer + Animator → Sprite 序列幀動畫
```

### 6.3 unity-cli 指令執行規範（⚠️ 強制）

**所有 unity-cli 指令必須使用單行 `;` 串接格式**，禁止分行寫法：

```powershell
# ✅ 正確：單行串接，一次 tool call 完成
$json = '{"prefabPath":"Assets/.../Fish.prefab"}'; [System.IO.File]::WriteAllText("params.json", $json, [System.Text.UTF8Encoding]::new($false)); & "$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe" raw open_prefab --params-file params.json

# ❌ 禁止：分行寫法（會被拆成多次執行）、Out-File / Set-Content（會加 BOM）
```

詳細規範請參閱 `dinopinball_monseter_prefabs_standard.md` 的「7.3 unity-cli 指令執行規範」。


---

## 7. 驗證檢查清單 (Validation Checklist)

### 7.1 結構驗證

- [ ] Root Tag = `Fish`
- [ ] Root Layer = `FISH_ZORDER`
- [ ] Root 至少有 5 個 Component（Transform, Fish2D 子類, FishRigidBody2D, SteerForOceanKing2D, PathBehavior）
- [ ] Shape 子物件存在且 active（命名不限）
- [ ] Shape 上有 4 個 Handler（SpriteLayerHandler, FishAnimatorHandler, FishMaskStatusHandler, FishColliderHandler）
- [ ] freeze/ 存在且 **inactive**
- [ ] freeze/freezeSprite 有 SpriteRenderer
- [ ] shadow 存在且有 SpriteFollower + SpriteRenderer
- [ ] maskroot/ 存在且 active
- [ ] maskroot/main 存在且有渲染 Component（Spine 或 Sprite）
- [ ] main 上有 SpriteFlicker
- [ ] collider/ 存在且 active
- [ ] collider/ 下至少一個 RadiusChecker
- [ ] Paralysis/ 存在且 **inactive**
- [ ] Paralysis/Effect_Paralysis 有 Animator

### 7.2 Fish2D SerializeField 引用驗證（重點）

> 這是最關鍵的檢查項目。Fish2D 的 `[SF]` 引用必須正確指向 Spine / Sprite 的顯示 Root。

**Spine 魚種**：
- [ ] `[SF]fishSpine` 指向有 `SkeletonAnimation` 的 GameObject（Spine 顯示 Root）
- [ ] `[SF]fishSpine` 的 `gameObject` 名稱與 Hierarchy 中的 Spine 節點一致
- [ ] `[SF]m_ColliderHandler` 不為 null（指向 Shape 上的 FishColliderHandler）

**Sprite 魚種**：
- [ ] 對應的 Animator / SpriteRenderer 引用指向 maskroot/main
- [ ] `[SF]m_ColliderHandler` 不為 null

**通用**：
- [ ] 所有 `[SF]` 欄位無 `"has not been assigned"` 錯誤
- [ ] 子類自訂的 `[SF]` 欄位（如 `[SF]uiRoot`、`[SF]uiBoard`）引用正確

### 7.3 渲染方式驗證

**Sprite 動畫模式**：
- [ ] main 有 SpriteRenderer
- [ ] main 有 Animator
- [ ] main 有 FishAnimationEventListener

**Spine 骨骼動畫模式**：
- [ ] main 有 MeshFilter
- [ ] main 有 MeshRenderer
- [ ] main 有 SkeletonAnimation

### 7.4 DoubleFish 翻倍魚額外驗證

- [ ] Root 有 `FishDialogue` Component
- [ ] Root 有 `Thunder2BonusChoose` Component

---

## 8. 常見問題與排查

### 8.1 Spine vs Sprite 判斷

使用 `get_gameobject_details` 查看 `maskroot/main` 的 components：
- 有 `SkeletonAnimation` → **Spine 骨骼動畫**
- 有 `SpriteRenderer` + `Animator` → **Sprite 序列幀動畫**

### 8.2 海王系列只有一個 Prefab

與恐龍系列不同（恐龍有模型 Prefab + 怪物主 Prefab），
海王系列每個魚種只有一個 `{FishName}.prefab`，直接包含所有遊戲邏輯和視覺。

### 8.3 碰撞器差異

海王系列使用自訂的 `RadiusChecker`（基於距離檢測），
不是 Unity 內建的 Collider Component。
在 unity-cli 中查看時，碰撞器節點的 Component 是 `RadiusChecker` 而非 `SphereCollider`。

### 8.4 影子跟隨機制

海王系列的 shadow 使用 `SpriteFollower` 自動跟隨魚種移動，
而恐龍系列的 Shadow/Obj 是靜態 SpriteRenderer（由父物件 Transform 帶動）。

### 8.5 冰凍效果大小

freeze/ 下的粒子效果有兩種尺寸：
- `Freeze_Effect_S` — 小型魚種用
- `Freeze_Effect_L` — 大型魚種用

---

## 9. 參考魚種速查表

| 魚種類型 | 參考魚種 | Prefab 路徑 | 渲染方式 | 額外 Component |
|----------|---------|------------|---------|---------------|
| 一般魚種 (A) | 金蝙蝠 | `.../Fishes/GoldenBat/GoldenBat.prefab` | Sprite | 無 |
| DoubleFish (B) | 殺人鯨 | `.../Fishes/KillerWhale/KillerWhale.prefab` | Spine | FishDialogue, Thunder2BonusChoose |
| 特殊狀態魚 (C) | 武士魚 | `.../Fishes/SamuraiFish/SamuraiFish.prefab` | Spine | 無 |
| 技能魚種-傳統 (D) | 烈焰風暴 | `.../Fishes/FireStorm/FireStorm.prefab` | Spine | 無 |
| 技能魚種-通用 (E) | 朱雀 | `.../Fishes/VermilionBird/VermilionBird.prefab` | Spine | 無 |

---

## 10. 海王 vs 恐龍系列完整對照表

| 面向 | 海王系列 (OceanTreasure) | 恐龍系列 (DinoPinBall) |
|------|------------------------|----------------------|
| **維度** | 2D | 3D |
| **Tag** | `Fish` | `Monster` |
| **基類** | `Fish2D` | `BaseMonster` → `PathAnimal` / `StaticAnimal` |
| **渲染** | Spine `SkeletonAnimation` 或 Sprite | FBX + `SkinnedMeshRenderer` |
| **移動系統** | `FishRigidBody2D` + `SteerForOceanKing2D` + `PathBehavior` | `ArkSteerBehavior` + `SteerForTether` + `PathController` |
| **碰撞** | `RadiusChecker`（自訂） | `SphereCollider` / `BoxCollider`（Unity 內建） |
| **事件系統** | `FishHunter_EventManager` | `Dino_EventManager` |
| **Hierarchy 根** | Shape（命名不限）包含所有子節點 | Shape/Shadow/Colliders/Effects 平行子物件 |
| **影子** | `SpriteFollower`（自動跟隨） | 靜態 SpriteRenderer（父物件帶動） |
| **冰凍效果** | `freeze/`（在 Shape 內） | `Effects/Freeze`（獨立子物件） |
| **Prefab 數量** | 1 個（`{FishName}.prefab`） | 2 個（模型 Prefab + 怪物主 Prefab） |
| **資料夾結構** | 扁平（所有資源同一層） | 分層（Animation/Material/Model/Texture） |
| **命名** | 無廳館前綴 | 有廳館前綴（`Stoneage_` / `Juras_` / `Robin_`） |
| **Handler 腳本** | Shape 上掛 4 個 Handler | 無 Handler（邏輯在 BaseMonster 內） |
| **規範文件** | 本文件 | `dinopinball_monseter_prefabs_standard.md` |

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
