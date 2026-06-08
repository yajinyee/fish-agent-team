---
inclusion: manual
---

# DinoPinBall 新增怪物 Skill (Add Monster Species)

當使用者說「新增怪物」、「加入新恐龍」、「建立 DinoPinBall 魚種」時，啟動此 Skill 執行完整的怪物開發流程。
本文件適用於 DinoPinBall / DinoGame (3D 魚機) 開發。
OceanTreasure (2D 魚機) 請參閱 `fishhunter_addfish_expert.md`。

## 語言規範

- 所有對話使用**繁體中文**
- 程式碼中的專有名詞保持英文

---

## 第零步：選擇參考範本

當使用者說要新增怪物時，**先列出以下參考範本選項**讓使用者選擇，再進入資訊收集：

| 選項 | 分類 | 封包路徑 | 參考怪物 | 複雜度 | 說明 |
|------|------|----------|----------|--------|------|
| A | 一般怪物 | W2 | Robin_Goblin (哥布林) | ⭐ | 最簡單，僅 WeaponSystem w2 捕獲，無特殊演出 |
| B | 翻倍怪物 | W2 + F7 | Juras_GoldenMammoth (黃金猛瑪象) | ⭐⭐ | w2 捕獲 + f7 封包更新倍數，捕獲後翻倍演出 |
| C | 技能怪物（獨立） | SkillSystem (自訂 sk_xxx) | Juras_MechanicGoblin (機甲哥布林) | ⭐⭐⭐ | 獨立 SkillXxxObject，多種技能模式（HitFloor/Missile/Laser） |
| D | 技能怪物（通用） | SkillSystem (sk_skill_start) | SkillCommonObject 通用模組 | ⭐⭐⭐ | 標準化流程 sk_skill_start → sk_bomb_fish → sk_end |
| E | BOSS (W2) | W2 | Stoneage_Tyrannosaurus (霸王龍) | ⭐⭐⭐ | BOSS 等級，走 WeaponSystem，有進度條 |
| F | 空殼模板 | 未定 | 未定 | ⭐ | 僅建立基礎骨架（繼承 PathAnimal + 覆寫 Init），提供後續開發模板 |
| G | BOSS (Skill) | SkillSystem (自訂 sk_xxx) | Stoneage_BossAriken (阿力肯) | ⭐⭐⭐⭐ | BOSS 等級，走 SkillSystem，完整技能+演出+砲台養成 |
| H | BOSS (特殊機制) | W2 + F8 | Dino_SlimeKing (史萊姆王) | ⭐⭐⭐⭐ | BOSS 等級，f8 計量條 + 分裂機制，特殊生成邏輯 |

使用者選擇後，Agent 會：
1. 讀取參考怪物的 SO 設定作為範本
2. 建立空資料夾結構 + 複製參考怪物的主 Prefab
3. SO 的 `assetbundleName` 和 `prefabName` **直接指向新怪物自己的資源**（不借用參考怪物）

### 資源處理機制（Agent 自動執行）

Agent 在第五步會自動執行以下操作：
1. 建立怪物資源資料夾（依廳館建立對應的子資料夾結構）
2. **僅複製參考怪物的主 Prefab**（遊戲用的那個，如 `Robin_Goblin.prefab`、`Stoneage_Dilophosaurus.prefab`），並改名為新怪物名稱
3. 子資料夾（Animation/Animator/Material/Model/Texture）保持空白，等美術團隊填入資源
4. SO 的 `assetbundleName` 和 `prefabName` 直接設為新怪物自己的值

> ⚠️ **不要複製整個參考怪物資料夾的所有資源**（FBX、材質、貼圖等）。這些由美術團隊提供，Agent 只需建立空的資料夾結構 + 複製主 Prefab。

### 怪物 Prefab 資源資料夾結構

怪物資源統一放在 `Assets/FishtHunter_Res/CrazyDino/Bundle/Monster/` 下。
**新增怪物一律使用以下統一結構**，不加廳館前綴、不放入內層子資料夾：

```
Bundle/Monster/{MonsterName}/
├── Animator/          # 動畫控制器與動畫 Clip（空，美術填入）
├── Material/          # 材質（空，美術填入）
├── Model/             # FBX 模型（空，美術填入）
├── Texture/           # 貼圖（空，美術填入）
└── {MonsterName}.prefab  # 主 prefab（遊戲用，SO 的 prefabName 指向此）
```

> ⚠️ **舊有怪物**仍保留原本的廳館前綴結構（如 `Juras_xxx/`、`RobinMonster/Robin_xxx/`），但新增怪物不再沿用。

### 舊有怪物資源資料夾位置對照（僅供參考）

| 廳館 | 舊有資料夾路徑 | 結構 | 舊有主 Prefab 命名 |
|------|-----------|------|---------------|
| Dino / StoneAge | `Bundle/Monster/{MonsterName}/` | 標準（含子資料夾） | `Stoneage_{MonsterName}.prefab` |
| Juras | `Bundle/Monster/Juras_{MonsterName}/` | 標準（含子資料夾） | `Juras_{MonsterName}.prefab` |
| Robin | `Bundle/Monster/RobinMonster/Robin_{MonsterName}/` | 扁平（無子資料夾） | `Robin_{MonsterName}.prefab` |

---

## 第一步：收集必要資訊

向使用者詢問以下資訊（若未提供）：

| 參數 | 說明 | 範例 |
|-----|------|------|
| **monsterName** | 怪物名稱 (UPPER_SNAKE_CASE) | `FLAME_DRAGON` |
| **chineseName** | 中文名稱 | 火焰龍 |
| **serverIndex** | Server ID（**必填，由 Server 端決定**） | 9102 或 39026 |
| **monsterSource** | 所屬廳館 | Dino / Juras / Robin / StoneAge |
| **monsterCategory** | 怪物分類（見下方說明） | 一般怪物 / 翻倍怪物 / 技能怪物 |
| **minOdds** | 最小倍率 | 50 |
| **maxOdds** | 最大倍率 | 200 |
| **isBoss** | 是否為 BOSS | 是/否 |
| **activeType** | 生成方式 | NormalSpawn / GolemSpawn / StaticSpawn |
| **hasFeature** | 是否有特殊演出 | 是/否 |
| **assetbundleName** | AssetBundle 名稱（全小寫） | `flamedragon` |
| **prefabName** | Prefab 名稱 | `FlameDragon` |
| **serverName** | Server 代號（由 monsterName 推導） | `FLAME_DRAGON` |

### 怪物分類說明

| 分類 | 封包路徑 | 處理系統 | 捕獲封包 | 範例 |
|------|----------|----------|----------|------|
| **一般怪物** | W2 | WeaponSystem | w2 | 蛋蛋龍、鳥龍、哥布林 |
| **翻倍怪物 (WiningMonster)** | W2 + F7 | WeaponSystem | w2 + f7 | 黃金猛瑪象 |
| **技能怪物（獨立）** | SkillSystem | SkillSystem | 自訂 sk_xxx 指令 | 機甲哥布林（3 種技能模式） |
| **技能怪物（通用）** | SkillSystem | SkillCommonObject | sk_skill_start → sk_bomb_fish → sk_end | 通用模組 |
| **BOSS (W2)** | W2 | WeaponSystem | w2 | 霸王龍 |
| **空殼模板** | 未定 | 未定 | 未定 | 僅建立基礎骨架 |
| **BOSS (Skill)** | SkillSystem | SkillSystem | 自訂 sk_xxx | 阿力肯 |
| **BOSS (特殊)** | W2 + F8 | WeaponSystem + 特殊 | w2 + f8 計量條 | 史萊姆王（分裂機制） |

### 自動推導規則

- **serverName**: 與 monsterName 相同（UPPER_SNAKE_CASE）
- **assetbundleName**: monsterName 轉為全小寫無底線，**不加廳館前綴**
  - `FLAME_DRAGON` → `flamedragon`
  - `CIRCE_WITCH` → `circewitch`
- **prefabName**: monsterName 轉為 PascalCase，**不加廳館前綴**
  - `FLAME_DRAGON` → `FlameDragon`
  - `CIRCE_WITCH` → `CirceWitch`

### Server ID 編號規則

DinoPinBall 的 `MonsterType` 是離散值 enum，**不需要像 OceanTreasure 那樣處理跳號佔位**。

| 廳館 | ID 範圍 | 範例 |
|------|---------|------|
| Dino (恐龍) | 9001 ~ 9xxx | 蛋蛋龍=9001, 霸王龍=9016 |
| Juras (獵龍2) | 39000 ~ 39xxx | 迅猛龍=39003, 岩晶龍=39022 |
| Robin (勇者) | 9096 ~ 9xxx | 俠客鰻=9096 |
| 特殊 | 其他 | 幽靈魚=216 |

---

## 第二步：修改 MonsterEnum.cs

在 `MonsterType` 列舉的 `MAX` 之前新增怪物類型。

檔案位置：`Assets/FishHunter_Script/DinoGame/DinoPinBall/Game/MonsterSystem/Enum/MonsterEnum.cs`

```csharp
/// <summary>
/// {中文名稱}
/// </summary>
{MONSTER_NAME} = {serverIndex},

MAX,
```

### 範例

```csharp
/// <summary>
/// 火焰龍
/// </summary>
FLAME_DRAGON = 39026,

MAX,
```

---

## 第三步：建立 MonsterParameterDataSO（ScriptableObject）

DinoPinBall 的怪物參數資料主要從 Excel 表單（`MONSTER.xlsx`）讀取，但可透過 ScriptableObject 補充或覆寫。
當 Excel 表單尚未更新時，使用 SO 是最快的方式讓 Client 端認識新怪物。

> ⚡ **此步驟由 Agent 自動完成**：Agent 會直接建立 `.asset` 和 `.asset.meta` 檔案，不需要手動操作 Unity Editor。
> 建立完成後，回到 Unity Editor 等待自動刷新（或手動 Reimport）即可。
> **注意**：SO 建立後，需在 `MonsterManager` Inspector 上點擊「掃描怪物 SO 列表」按鈕，讓 `m_MonsterSOList` 自動收集新增的 SO。

### 3.1 建立 SO 資產檔（Agent 自動執行）

#### 檔案路徑規則

SO 資產依廳館分類存放：

| 廳館 | 存放路徑 |
|------|----------|
| Dino (恐龍) | `Assets/FishtHunter_Res/CrazyDino/Bundle/ScriptableObject/MonsterParameter/StoneAge/` |
| Dino3 (恐龍3廳) | `Assets/FishtHunter_Res/CrazyDino/Bundle/ScriptableObject/MonsterParameter/Dino3/` |
| Juras (獵龍2) | `Assets/FishtHunter_Res/CrazyDino/Bundle/ScriptableObject/MonsterParameter/StoneAge/` |
| Robin (天羽) | `Assets/FishtHunter_Res/CrazyDino/Bundle/ScriptableObject/MonsterParameter/Robin/` |
| StoneAge (史前) | `Assets/FishtHunter_Res/CrazyDino/Bundle/ScriptableObject/MonsterParameter/StoneAge/` |

#### 命名規則

- Dino 廳：`Dino_{PascalCaseName}.asset`（如 `Dino_IcerBerus.asset`）
- Dino3 廳：`Dino3{PascalCaseName}.asset`（如 `Dino3Velociraptor_3.asset`）
- Juras 廳：`Juras_{PascalCaseName}.asset`（如 `Juras_GoldenMammoth.asset`）
- Robin 廳：`Robin_{PascalCaseName}.asset`（如 `Robin_Goblin.asset`）
- StoneAge 廳：`Stoneage_{PascalCaseName}.asset`（如 `Stoneage_Ankylosaurus.asset`）

#### .asset 檔案範本（Unity YAML 格式）

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
  m_Script: {fileID: 11500000, guid: 08cf39210e132d049bd479a74c366f0e, type: 3}
  m_Name: {AssetFileName不含副檔名}
  m_EditorClassIdentifier: 
  monsterType: {serverIndex}
  chineseName: "{中文名稱}"
  assetbundleName: {assetbundleName全小寫}
  prefabName: {prefabName}
  serverName: {MONSTER_NAME}
  isBoss: {0或1}
  maxSpeed: {依參考怪物設定}
  maxForce: {依參考怪物設定}
  minOdds: {minOdds}
  maxOdds: {maxOdds}
  monsterGroup: 0
  winingOddsList: {見下方說明}
  colliderRadius: 0
  repelDistance: 1
  baseDieStayTime: 1
  awardTextScale: 1
  activeType: {0=NormalSpawn, 1=GolemSpawn, 2=StaticSpawn}
  monsterSource: 0
  subType: 0
  SpecialDeclare: {fileID: 0}
```

#### winingOddsList 格式

- 一般怪物（無翻倍）：留空
  ```yaml
  winingOddsList: 
  ```
- 翻倍怪物：使用 YAML 陣列格式
  ```yaml
  winingOddsList:
  - 2
  - 3
  - 5
  - 8
  ```

#### maxSpeed / maxForce 說明

這兩個值**依參考怪物的 SO 設定而定**，不是固定值。Agent 在第零步讀取參考怪物 SO 後，直接沿用其 `maxSpeed` 和 `maxForce` 值。

| 參考怪物 | maxSpeed | maxForce |
|----------|----------|----------|
| Robin_Goblin (哥布林) | 7 | 1 |
| Juras_GoldenMammoth (黃金猛瑪象) | 依 SO | 依 SO |
| Juras_CrystalDragon (岩晶龍) | 依 SO | 依 SO |

#### monsterSource 說明

`monsterSource` 統一設為 `0`（All），表示怪物可跨廳出現。不需要依廳館設定不同值。

#### .asset.meta 檔案範本

```yaml
fileFormatVersion: 2
guid: {生成的GUID}
NativeFormatImporter:
  externalObjects: {}
  mainObjectFileID: 11400000
  userData: 
  assetBundleName: 
  assetBundleVariant: 
```

#### 關鍵常數

- **MonsterParameterDataSO script GUID**: `08cf39210e132d049bd479a74c366f0e`
- **GUID 生成方式**: `[System.Guid]::NewGuid().ToString("N")`（禁止手動編造）

#### monsterSource 數值對照

`monsterSource` 統一設為 `0`（All），不需要依廳館設定不同值。

| 廳館 | enum 值 | 數值 | 備註 |
|------|---------|------|------|
| All | `MonsterSource.All` | 0 | **統一使用此值** |
| Dino | `MonsterSource.Dino` | 2 | 僅供參考，新怪物不使用 |
| Juras | `MonsterSource.Juras` | 5 | 僅供參考，新怪物不使用 |
| Robin | `MonsterSource.Robin` | 6 | 僅供參考，新怪物不使用 |

#### activeType 數值對照

| 生成方式 | enum 值 | 數值 |
|----------|---------|------|
| NormalSpawn | `ActiveType.NormalSpawn` | 0 |
| GolemSpawn | `ActiveType.GolemSpawn` | 1 |
| StaticSpawn | `ActiveType.StaticSpawn` | 2 |

### 3.2 中文名稱 Unicode 編碼

Unity YAML 中的中文字串會自動以 Unicode 轉義序列儲存（如 `"\u54E5\u5E03\u6797"`）。
Agent 建立 `.asset` 時，可以直接使用中文字串（如 `"哥布林"`），Unity 讀取時會自動處理。
若要與現有檔案格式完全一致，可使用 Unicode 轉義。

### 3.3 點擊「掃描怪物 SO 列表」按鈕（⚠️ 需手動操作）

SO 建立後，需在 `MonsterManager` Inspector 上點擊「掃描怪物 SO 列表」按鈕，讓 `m_MonsterSOList` 自動收集 `MonsterParameter/` 資料夾下所有 `MonsterParameterDataSO`。

`MonsterManager` 直接持有 `List<MonsterParameterDataSO> m_MonsterSOList`，不再透過 `MonsterInfo` 中間層。

**操作步驟**：
1. 回到 Unity Editor，等待自動刷新（或右鍵 Reimport）
2. 在場景中找到 `MonsterManager` 物件（或開啟其所在 Prefab）
3. 在 Inspector 中找到「怪物 SO 列表」區塊
4. 點擊「掃描怪物 SO 列表」按鈕
5. 確認列表中出現新增的 SO
6. 儲存場景 / Prefab

### 3.4 載入機制說明

`MonsterManager.InitMonsterParameterData()` 的載入順序：
1. 先從 Excel 表單（`MONSTER.xlsx`）讀取所有怪物資料
2. 再遍歷 `m_MonsterSOList`（由「掃描怪物 SO 列表」按鈕自動收集）
   - 若 `monsterType` 已存在於字典 → **覆寫** `SpecialDeclare` 和 `IsBoss`
   - 若 `monsterType` 不存在 → **新增**整筆資料（呼叫 `so.ToRuntimeData()`）

---

## 第四步：新增 Dino_EventType 事件類型（若有特殊演出）

檔案位置：`Assets/FishHunter_Script/DinoGame/DinoPinBall/Game/Common/EventManager/Dino_EventType.cs`

一般怪物不需要新增事件類型，捕獲流程走 `WeaponSystemEvent.BLAST` 通用事件。

若怪物有 BOSS 進度條或特殊演出，在對應的 enum 中新增：

```csharp
public enum BossProgressBar
{
    SLIME_EVENT,
    LICH_EVENT,
    {MONSTER_NAME}_EVENT,  // 新增
}
```

---

## 第五步：建立怪物資源資料夾

### 5.1 建立 Prefab 資源資料夾（Agent 自動執行）

**所有廳館統一使用以下結構**，直接放在 `Bundle/Monster/` 下，不加廳館前綴、不放入內層：

```
Bundle/Monster/{MonsterName}/
├── Animation/         # 空（美術填入）
├── Animator/          # 空（美術填入）
├── Material/          # 空（美術填入）
├── Model/             # 空（美術填入）
├── Texture/           # 空（美術填入）
└── (主 Prefab 由美術建立，或從參考怪物複製)
```

### ⚠️ AssetBundle 同名資源禁止

**同一個 AssetBundle 內，不同類型的資源檔案不可與 Prefab 同名。**
例如 `Monster/FlameDragon/` 下同時存在 `FlameDragon.prefab` 和 `FlameDragon.png`，
會導致 `AssetBundleManager.LoadAssetAsync` 載入到錯誤的資源（`GetAsset<GameObject>()` 回傳 `null`）。
確保 Prefab 名稱在 bundle 內唯一，非 Prefab 資源需改名避免衝突。

### ⚠️ 資料夾已存在時必須檢查 AssetBundle 設定

**若 Prefab 資源資料夾已由美術或其他流程預先建立**，Agent 不可跳過，必須讀取該資料夾的 `.meta` 檔案，確認 `assetBundleName` 已正確設定為 `dinopinball/{assetbundlename小寫}`。若為空或不正確，必須主動修正。

Agent 建立資料夾 `.meta`（含 AssetBundle 設定）：

```yaml
# Bundle/Monster/{MonsterName}.meta
fileFormatVersion: 2
guid: {生成的GUID}
folderAsset: yes
DefaultImporter:
  userData: 
  assetBundleName: dinopinball/{assetbundlename小寫}
  assetBundleVariant: 
```

子資料夾 `.meta`（不設 AssetBundle）：

```yaml
# Bundle/Monster/{MonsterName}/Animation.meta 等
fileFormatVersion: 2
guid: {生成的GUID}
folderAsset: yes
DefaultImporter:
  externalObjects: {}
  userData: 
  assetBundleName: 
  assetBundleVariant: 
```

### 5.2 複製參考怪物主 Prefab（Agent 自動執行）

Agent **僅複製主 Prefab**（遊戲用的那個），並改名為新怪物名稱，**不加廳館前綴**：

| 要複製的主 Prefab（參考怪物） | 複製後改名為 |
|-------------------------------|-------------|
| 參考怪物的主 Prefab（如 `Robin_Goblin.prefab`） | `{MonsterName}.prefab`（如 `CirceWitch.prefab`） |

> ⚠️ **不要複製 FBX、材質、貼圖等美術資源**。這些由美術團隊提供。
> ⚠️ Prefab 是 Unity 二進位資產，複製後內部 GUID 引用仍指向原始資源，需在 Unity Editor 中重新指定。

### 5.4 使用 unity-cli 檢查並補齊 Prefab Component（⚠️ 必要流程）

**當 Prefab 已存在時**（美術已建立或從參考怪物複製），Agent **必須**使用 unity-cli 檢查 Prefab 內容，並依據 `#dinopinball_monseter_prefabs_standard` 補齊缺少的 Component 和子物件。

#### 檢查流程

```
1. open_prefab → 進入 Prefab 編輯模式
2. get_hierarchy → 檢查 Hierarchy 結構（Shape/Shadow/Colliders/Effects）
3. get_component_values → 檢查 Root 上的 Component
4. 比對標準 → 補齊缺少的項目
5. save_prefab → 儲存
```

#### 檢查清單與自動修正

| 檢查項目 | unity-cli 指令 | 缺少時自動修正 |
|----------|---------------|---------------|
| Root Tag = `Monster` | `modify_gameobject` 檢查 `tag` | `modify_gameobject --tag Monster` |
| Root Layer = `FISH_ZORDER` (18) | `modify_gameobject` 檢查 `layer` | `modify_gameobject --layer 18` |
| ArkSteerBehavior | `get_component_values` | `add_component` |
| SteerForTether | `get_component_values` | `add_component` |
| 怪物控制器（PathAnimal 或自訂） | `get_component_values` | `add_component` |
| PathController | `get_component_values` | `add_component` |
| Animator | `get_component_values` | ⚪ 選用，不一定需要加（部分怪物動畫由 Shape 子物件的 Animator 處理） |
| Colliders/ 子物件 | `get_hierarchy` 檢查 | `create_gameobject` + `modify_gameobject --tag Monster` |
| Colliders/ SphereCollider isTrigger | `get_component_values` | `add_component --properties isTrigger:true` |
| Effects/Freeze (inactive) | `get_hierarchy` 檢查 | `create_gameobject` + `modify_gameobject --active false` |
| Effects/ThunderTwoTarget | `get_hierarchy` 檢查 | `create_gameobject` |

#### unity-cli 標準操作順序

```bash
# 1. 開啟 Prefab
unity-cli raw open_prefab --params-file params.json
# params: {"prefabPath":"Assets/.../Monster.prefab"}

# 2. 檢查 Hierarchy
unity-cli raw get_hierarchy --params-file params.json
# params: {"maxDepth":3}

# 3. 設定 Root Tag/Layer
unity-cli raw modify_gameobject --params-file params.json
# params: {"path":"/MonsterName","tag":"Monster","layer":18}

# 4. 加 Component（依序）
unity-cli raw add_component --params-file params.json
# params: {"gameObjectPath":"/MonsterName","componentType":"ArkSteerBehavior"}
# params: {"gameObjectPath":"/MonsterName","componentType":"SteerForTether"}
# params: {"gameObjectPath":"/MonsterName","componentType":"DinoPinBall.PathAnimal"}
# params: {"gameObjectPath":"/MonsterName","componentType":"DinoPinBall.PathController"}

# 5. 建 Colliders（Tag=Monster, SphereCollider isTrigger=true）
unity-cli raw create_gameobject --params-file params.json
# params: {"name":"Colliders","parentPath":"/MonsterName"}
unity-cli raw modify_gameobject --params-file params.json
# params: {"path":"/MonsterName/Colliders","tag":"Monster"}
unity-cli raw add_component --params-file params.json
# params: {"gameObjectPath":"/MonsterName/Colliders","componentType":"SphereCollider","properties":{"isTrigger":true}}

# 6. 建 Effects（Freeze inactive + ThunderTwoTarget）
unity-cli raw create_gameobject --params-file params.json
# params: {"name":"Effects","parentPath":"/MonsterName"}
unity-cli raw create_gameobject --params-file params.json
# params: {"name":"Freeze","parentPath":"/MonsterName/Effects"}
unity-cli raw modify_gameobject --params-file params.json
# params: {"path":"/MonsterName/Effects/Freeze","active":false}
unity-cli raw create_gameobject --params-file params.json
# params: {"name":"ThunderTwoTarget","parentPath":"/MonsterName/Effects"}

# 7. 儲存
unity-cli raw save_prefab --params-file params.json
# params: {}
```

#### ⚠️ unity-cli 指令執行規範（強制）

**所有 unity-cli 指令必須使用單行 `;` 串接格式**，禁止分行寫法：

```powershell
# ✅ 正確：單行串接
$json = '{"prefabPath":"Assets/.../Monster.prefab"}'; [System.IO.File]::WriteAllText("params.json", $json, [System.Text.UTF8Encoding]::new($false)); & "$env:USERPROFILE\.unity\tools\unity-cli\win-x64\unity-cli.exe" raw open_prefab --params-file params.json

# ❌ 禁止：分行寫法（會被拆成多次執行）、Out-File / Set-Content（會加 BOM）
```

詳細規範請參閱 `dinopinball_monseter_prefabs_standard.md` 的「7.3 unity-cli 指令執行規範」。

#### 完成後提醒使用者

Agent 完成 unity-cli 自動設定後，**必須提醒使用者**手動完成以下項目：

1. **PathAnimal / BaseMonster SerializeField 引用**（Inspector 拖拉）：
   - `m_shape` → Shape 子物件
   - `m_monsterAnimator` → Root Animator
   - `m_runAnimator` → Shape 下的 Animator
   - `monsterController` → Root 的 PathController
   - `m_lightingMaterial` → Lightning 材質
2. **SphereCollider radius** → 調整到適合怪物模型大小
3. **Animator Controller** → Root Animator 指定 AnimatorController 資源

### 5.3 建立腳本資料夾

路徑：`Assets/FishHunter_Script/DinoGame/DinoPinBall/Game/MonsterSystem/{MonsterName}/`

```yaml
# MonsterSystem/{MonsterName}.meta
fileFormatVersion: 2
guid: {生成的GUID}
folderAsset: yes
DefaultImporter:
  externalObjects: {}
  userData: 
  assetBundleName: 
  assetBundleVariant: 
```

### GUID 生成方式

```powershell
[System.Guid]::NewGuid().ToString("N")
```

禁止手動編造 GUID。

---

## 第六步：建立怪物控制器（視需求而定）

一般怪物（小魚）**不需要**建立控制器，使用預設的 `BaseMonster` 即可。
只有具備特殊功能的怪物才需要自訂控制器。

> 💡 **跳過判斷**：若使用者在第零步選擇了 A（一般怪物），預設跳過本步驟，需提示使用者是否需要。選項 F（空殼模板）**必定建立控制器**（繼承 `PathAnimal`）。僅在使用者選擇 B/C/D/E 或主動要求時才詢問。

Agent 應在此步驟**詢問使用者**是否需要：
1. 怪物控制器（繼承 `BaseMonster`，處理特殊行為邏輯）
2. 演出控制器（`FeatureCtrl`，處理捕獲後的特殊演出）
3. 技能整合（`SkillSystem`，處理 sk_xxx 指令）

若使用者確認需要，才建立對應檔案。

### 控制器檔案位置

`Assets/FishHunter_Script/DinoGame/DinoPinBall/Game/MonsterSystem/{MonsterName}/{MonsterName}Controller.cs`

繼承 `BaseMonster`（位於 `Game/MonsterSystem/Common/BaseMonster.cs`）。

### 怪物控制器範本（繼承 BaseMonster）

```csharp
using UnityEngine;

namespace DinoPinBall
{
    /// <summary>
    /// {中文名稱} 怪物控制器
    /// </summary>
    public class {MonsterName}Controller : BaseMonster
    {
        protected override void OnInit()
        {
            base.OnInit();
#if DEBUG_LOG
            Debug.Log($"[{MonsterName}] OnInit SID={sid}");
#endif
        }

        protected override void OnDead()
        {
            base.OnDead();
#if DEBUG_LOG
            Debug.Log($"[{MonsterName}] OnDead SID={sid}");
#endif
        }
    }
}
```

### 空殼模板控制器範本（選項 F，繼承 PathAnimal）

選項 F 的控制器**必定建立**，繼承 `PathAnimal`，提供路徑移動怪物的基礎骨架。
封包路徑、處理系統、捕獲封包皆為「未定」，由後續開發者依需求補充。

```csharp
using UnityEngine;

namespace DinoPinBall
{
    /// <summary>
    /// {中文名稱} 怪物控制器（空殼模板）
    /// 繼承 PathAnimal，沿路徑移動。
    /// TODO: 依需求補充特殊行為邏輯、封包處理、演出控制。
    /// </summary>
    public class {MonsterName}Ctrl : PathAnimal
    {
        /// <summary>
        /// 初始化（PathAnimal 會在此設定路徑並呼叫 base.Init）
        /// </summary>
        public override void Init(
            int monsterType, float o, int sid, string path,
            float syncTime, int fakeCmd, JSON extraData = null,
            int subType = 0, int limitVip = 0)
        {
            base.Init(monsterType, o, sid, path, syncTime, fakeCmd, extraData, subType, limitVip);

#if DEBUG_LOG
            Debug.Log($"[{MonsterName}] Init SID={sid}, Type={monsterType}, Path={path}");
#endif

            // TODO: 自訂初始化邏輯（解析 extraData、註冊事件等）
        }

        /// <summary>
        /// 自訂初始化（PathAnimal 在 Init 成功後呼叫）
        /// </summary>
        protected override void CustomInit(int monsterType)
        {
            base.CustomInit(monsterType);

            // TODO: 初始化特殊元件（UI、特效、動畫等）
        }

        /// <summary>
        /// 怪物被擊殺時
        /// </summary>
        public override void Kill(int seat, bool isGetPhysics = false,
            float stayTime = 0, float additionalStayTime = 0, int winType = 0)
        {
            base.Kill(seat, isGetPhysics, stayTime, additionalStayTime, winType);

#if DEBUG_LOG
            Debug.Log($"[{MonsterName}] Kill SID={Sid}, Seat={seat}");
#endif

            // TODO: 自訂死亡邏輯（特殊演出、事件發送等）
        }

        private void OnDestroy()
        {
            // TODO: 清理資源（取消事件註冊、釋放引用等）
        }
    }
}
```

---

## 第七步：建立 Debug 工具

> ⚡ **規則**：當怪物需要建立 CODE 腳本（選項 B/C/D/E/G/H），Agent 應**詢問使用者是否需要建立 Debug 測試工具**。
> 選項 A（一般怪物）和選項 F（空殼模板）不需要 Debug 腳本，因為 `CheatToolManager` 會自動列出。

### 7.1 自動測試（CheatToolManager 內建，所有怪物皆適用）

只要怪物資料正確註冊到 `MonsterParameterDataDict`，
`CheatToolManager` 的「恐龍生成」面板就會**自動列出新怪物**，
不需要像 OceanTreasure 那樣手動新增 `FishTestData`。

```
CheatToolManager.OnCreateDinoButtonPress()
  → DebugManager.TestCreateDinoOnlyClient(monsterType, count, fakeKill)
    → MonsterSystem.onCreateMonsterForOnlyClient(json)
```

### 7.2 建立 Debug 腳本（詢問使用者是否需要）

Agent 應詢問：「是否需要建立 Debug 測試工具？」

若使用者確認需要，建立檔案：`Assets/FishHunter_Script/DinoGame/DinoPinBall/Game/MonsterSystem/{MonsterName}/{MonsterName}Debug.cs`

#### 7.2.1 尋找場上怪物

**統一使用 `FindAnyObjectByType<{MonsterName}Ctrl>()` 取得場上怪物的 SID**：

```csharp
var monster = FindAnyObjectByType<{MonsterName}Ctrl>();
if (monster == null) { Debug.LogError("找不到怪物"); return; }
var sid = monster.Sid;
```

#### 7.2.2 假召喚方式

**統一使用 `CheatToolManager.debugCreateDino()` 加入怪物到魚場**：

```csharp
cheatToolManager.debugCreateDino(sDinoID); // sDinoID = "MONSTER_NAME"
```

#### 7.2.2 W2 擊殺封包測試（參考海王 Debug 方式）

直接建構 W2 JSON 封包，發送給 `WeaponSystem.OnMessage()`：

```csharp
[Button("W2-{中文名稱}擊殺封包", ButtonSizes.Medium)]
private void TestW2Kill()
{
    var monster = MonsterManager.Instance.GetMonsterBySid(Convert.ToInt32(iSid));
    if (monster == null)
    {
        Debug.LogError("[{MonsterName}Debug] 場景中找不到怪物，請先召喚");
        return;
    }

    var playerInfo = GameClient.Instance.GetGameData<PlayerInfo>();
    var player = playerInfo.GetPlayer(testSeat);
    if (player == null) return;

    double totalWins = player.Bet * testOdds;

    string jsonString = $@"{{
        ""cmd"": ""w2"",
        ""data"": {{
            ""1"": ""0_9"",
            ""2"": ""{player.Seat}"",
            ""3"": ""{player.Coin}"",
            ""8"": {{
                ""{iSid}"": {{
                    ""10"": ""{testOdds}"",
                    ""11"": ""{totalWins}"",
                    ""12"": 0,
                    ""13"": {{}},
                    ""14"": 0,
                    ""23"": {{}}
                }}
            }},
            ""9"": ""{player.Bet}"",
            ""10"": ""{testOdds}"",
            ""15"": null,
            ""24"": false,
            ""26"": ""{player.ID}"",
            ""28"": 0
        }}
    }}";

    var json = JSON.Parse(jsonString);
    GameClient.Instance.GetSystem<WeaponSystem>().OnMessage(json);
}
```

#### 7.2.3 F7 倍率更新測試（翻倍怪物專用，參考海王 Debug 方式）

直接建構 F7 JSON 封包，發送給 `MonsterSystem.OnMessage()`：

```csharp
[Button("F7-倍率更新", ButtonSizes.Medium)]
private void TestF7OddsUpdate()
{
    string jsonString = $@"{{
        ""cmd"": ""f7"",
        ""data"": {{
            ""fish_id"": ""{iSid}"",
            ""fish_type"": ""{(int)MonsterType.{MONSTER_NAME}}"",
            ""odds"": ""{testOdds}"",
            ""level"": ""{testLevel}""
        }}
    }}";

    var json = JSON.Parse(jsonString);
    GameClient.Instance.GetSystem<MonsterSystem>().OnMessage(json);
}
```

#### 7.2.4 技能測試（技能怪物專用，參考海王 Debug 方式）

直接建構 sk_xxx JSON 封包，發送給 `SkillSystem.OnMessage()`：

```csharp
[Button("觸發技能", ButtonSizes.Medium)]
private void TestSkillTrigger()
{
    // 依技能類型建構對應的 sk_xxx 封包
    string jsonString = $@"{{
        ""cmd"": ""sk_{skill_name}"",
        ""data"": {{
            ""fish_id"": ""{iSid}"",
            ""fish_type"": ""{(int)MonsterType.{MONSTER_NAME}}"",
            ""player_seat"": ""{testSeat}""
        }}
    }}";

    var json = JSON.Parse(jsonString);
    GameClient.Instance.GetSystem<SkillSystem>().OnMessage(json);
}
```

### 7.3 翻倍怪物 Debug 範本（選項 B）

```csharp
using Sirenix.OdinInspector;
using System;
using UnityEngine;

namespace DinoPinBall
{
    /// <summary>
    /// {中文名稱} Debug 工具
    /// 翻倍怪物（W2 捕獲 + F7 倍率更新）
    /// </summary>
    public class {MonsterName}Debug : MonoBehaviour
    {
#if DEBUG_LOG

        [System.Serializable]
        public class {MonsterName}Data
        {
            [BoxGroup("基本設定"), LabelText("基底倍率 (BaseOdds)")]
            public int baseOdds = 100;

            [BoxGroup("基本設定"), LabelText("乘倍 (Multiply)")]
            public int multiply = 2;

            [BoxGroup("基本設定"), LabelText("總倍率 (TotalOdds)")]
            public double totalOdds => baseOdds * multiply;

            [BoxGroup("基本設定"), LabelText("測試座位 (Seat)"), Range(0, 2)]
            public int testSeat = 0;

            [BoxGroup("基本設定"), LabelText("是否為本家")]
            public bool isMainPlayer = true;

            [BoxGroup("演出設定"), LabelText("演出類型")]
            public int showType = 0;

            [BoxGroup("F7 設定"), LabelText("F7 倍率 (Odds)")]
            public int f7Odds = 200;

            [BoxGroup("F7 設定"), LabelText("F7 等級 (Level)"), Range(0, 4)]
            public int f7Level = 0;
        }

        [FoldoutGroup("{中文名稱}")]
        [Title("CheatToolManager")]
        [SerializeField] private CheatToolManager cheatToolManager;

        private string sDinoID = "{MONSTER_NAME}";

        [FoldoutGroup("{中文名稱}")]
        [Button("加到魚場", ButtonSizes.Medium)]
        private void Add{MonsterName}()
        {
            cheatToolManager.debugCreateDino(sDinoID);
        }

        [FoldoutGroup("{中文名稱}")]
        [Title("{中文名稱} Debug 工具", "翻倍怪物 - W2 捕獲 + F7 倍率更新")]
        [Button("W2-{中文名稱}擊殺封包", buttonSize: ButtonSizes.Large, ButtonStyle.Box)]
        private void TEST_{MONSTER_NAME}_HIT({MonsterName}Data data)
        {
            var monster = FindAnyObjectByType<{MonsterName}Ctrl>();
            if (monster == null)
            {
                Debug.LogError("[{MonsterName}Debug] 場景中找不到{中文名稱}，請先召喚");
                return;
            }

            var sid = monster.Sid;
            var playerInfo = GameClient.Instance.GetGameData<PlayerInfo>();
            var player = data.isMainPlayer
                ? playerInfo.MainPlayer
                : playerInfo.GetPlayer(data.testSeat);

            if (player == null)
            {
                Debug.LogError($"[{MonsterName}Debug] 找不到座位 {data.testSeat} 的玩家");
                return;
            }

            var bet = player.Bet;
            var totalWins = bet * data.totalOdds;

            string jsonString = $@"{{
                ""cmd"": ""w2"",
                ""data"": {{
                    ""1"": ""0_9"",
                    ""2"": ""{player.Seat}"",
                    ""3"": ""{player.Coin}"",
                    ""8"": {{
                        ""{sid}"": {{
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
                    ""26"": ""{player.ID}"",
                    ""28"": 0,
                    ""29"": {{
                        ""base_odds"": ""{data.baseOdds}"",
                        ""multiply"": ""{data.multiply}"",
                        ""show_type"": ""{data.showType}""
                    }}
                }}
            }}";

            var json = JSON.Parse(jsonString);
            GameClient.Instance.GetSystem<WeaponSystem>().OnMessage(json);
            Debug.Log($"[{MonsterName}Debug] W2 擊殺: SID={sid}, TotalOdds={data.totalOdds}, Win={totalWins}");
        }

        [FoldoutGroup("{中文名稱}")]
        [Button("F7-{中文名稱}倍率更新", buttonSize: ButtonSizes.Large, ButtonStyle.Box)]
        private void TEST_{MONSTER_NAME}_F7({MonsterName}Data data)
        {
            var monster = FindAnyObjectByType<{MonsterName}Ctrl>();
            if (monster == null)
            {
                Debug.LogError("[{MonsterName}Debug] 場景中找不到{中文名稱}");
                return;
            }

            var sid = monster.Sid;

            string jsonString = $@"{{
                ""cmd"": ""f7"",
                ""data"": {{
                    ""fish_id"": ""{sid}"",
                    ""fish_type"": ""{(int)MonsterType.{MONSTER_NAME}}"",
                    ""odds"": ""{data.f7Odds}"",
                    ""level"": ""{data.f7Level}""
                }}
            }}";

            var json = JSON.Parse(jsonString);
            GameClient.Instance.GetSystem<MonsterSystem>().OnMessage(json);
            Debug.Log($"[{MonsterName}Debug] F7 倍率更新: SID={sid}, Odds={data.f7Odds}, Level={data.f7Level}");
        }

#endif
    }
}
```

### 7.4 一般怪物 Debug 範本（選項 A）

選項 A 的怪物不需要 Debug 腳本，`CheatToolManager` 會自動列出。

### 7.5 技能怪物 Debug 範本（選項 C/D）

在翻倍怪物範本基礎上，替換 F7 為技能觸發按鈕，直接建構 sk_xxx JSON 封包發送給 `SkillSystem.OnMessage()`：

```csharp
[System.Serializable]
public class {MonsterName}Data
{
    [BoxGroup("基本設定"), LabelText("倍率 (Odds)")]
    public int odds = 100;

    [BoxGroup("基本設定"), LabelText("測試座位 (Seat)"), Range(0, 2)]
    public int testSeat = 0;

    [BoxGroup("基本設定"), LabelText("是否為本家")]
    public bool isMainPlayer = true;

    [BoxGroup("技能設定"), LabelText("技能名稱")]
    public string skillName = "sk_{skill_name}";
}

[FoldoutGroup("{中文名稱}")]
[Button("觸發技能", buttonSize: ButtonSizes.Large, ButtonStyle.Box)]
private void TEST_{MONSTER_NAME}_SKILL({MonsterName}Data data)
{
    var monster = FindAnyObjectByType<{MonsterName}Ctrl>();
    if (monster == null)
    {
        Debug.LogError("[{MonsterName}Debug] 場景中找不到{中文名稱}");
        return;
    }

    var sid = monster.Sid;

    string jsonString = $@"{{
        ""cmd"": ""{data.skillName}"",
        ""data"": {{
            ""fish_id"": ""{sid}"",
            ""fish_type"": ""{(int)MonsterType.{MONSTER_NAME}}"",
            ""player_seat"": ""{data.testSeat}""
        }}
    }}";

    var json = JSON.Parse(jsonString);
    GameClient.Instance.GetSystem<SkillSystem>().OnMessage(json);
    Debug.Log($"[{MonsterName}Debug] 觸發技能: {data.skillName}, SID={sid}");
}
```

### 7.5 BOSS Debug 範本（選項 E/G/H）

BOSS 怪物除了上述功能外，還需在以下兩處新增專屬測試工具：

1. `DebugSystem.cs` — 新增發送測試指令的方法
2. `CheatToolManager.cs` — 新增 UI 按鈕與呼叫邏輯

參考現有 BOSS 測試工具：
- 岩晶龍：`DebugSystem.RockcrystalTest()` + `CheatToolManager` 三狀態切換
- 機甲哥布林：`DebugSystem.GoblinTest()` + `CheatToolManager` 技能類型選擇
- 史萊姆王：`DebugSystem.BossSlimeTest()`
- 阿力肯：`DebugSystem.ArikenTest()`

### 7.6 假生成封包格式（參考）

`DebugManager.createJson()` 建立的假封包結構：

```csharp
jsonArrayData["1"] = _serverId;   // 怪物 SID（自動遞增）
jsonArrayData["2"] = _id;         // MonsterType（如 9112）
jsonArrayData["3"] = 77;          // X 座標
jsonArrayData["4"] = 50;          // Y 座標
jsonArrayData["5"] = 54;          // 朝向角度
jsonArrayData["6"] = 0;           // Feature
jsonArrayData["8"] = 1;           // Size
jsonArrayData["17"] = _pathCode;  // 路徑代碼
jsonArrayData["4444"] = fakeKill; // 假封包標記（0=可擊殺, 1=不可擊殺）
```

### 7.7 路徑代碼對應

| 廳館 | 路徑代碼 |
|------|----------|
| dino / stoneage / robin | `PATH_DOINPINBALL` |
| juras | `PATH_JURAS` |

---

## 第八步：處理特殊怪物邏輯（視分類而定）

### 8.1 翻倍怪物 (WiningMonster)（選項 B）

在 SO 中設定 `winingOddsList` 陣列（如 `[2, 3, 5, 8]`），
`MonsterParameterData.IsWiningMonster` 會自動設為 `true`。

需額外處理：
- `MonsterSystem` 的 `f7` 封包接收（`onReceiveMonsterFeatureEvent`，目前用於更新黃金猛瑪象倍數）
- 翻倍演出邏輯（`BaseMonster` 已內建 `WiningOddsList` 判定）

### 8.2 技能怪物（獨立，選項 C）

需在 `SkillSystem.SetSkillObjects()` 中新增技能物件實例化：

```csharp
private void SetSkillObjects()
{
    // ... 現有技能物件
    m_{monsterName} = new Skill{MonsterName}Object(this);  // 新增
}
```

並建立對應的技能類別：
- `Skill{MonsterName}Object` — 技能指令處理器（繼承自對應基類）
- 在建構子中向 `SkillSystem` 註冊 Server 指令

在 `ESkillType` 列舉中新增技能類型：

```csharp
public enum ESkillType : int
{
    // ... 現有項目
    {MonsterName} = {skillTypeId},  // 新增
}
```

參考機甲哥布林的多技能模式：一隻怪物可有多個 `ESkillType`（如 `Goblin_HitFloor = 6000007`、`Goblin_Missile = 6000008`、`Goblin_Laser = 6000009`），對應多個 `SkillXxxObject`。

### 8.3 技能怪物（通用，選項 D）

使用 `SkillCommonObject`（`m_commonObject`）通用模組，標準化流程：
- `sk_skill_start` → `sk_bomb_fish` → `sk_end`
- 不需建立獨立的 `SkillXxxObject`
- 需在 `SkillCommonObject` 的 fishSkillDict 新增 monsterType → ESkillType 對應
- 建立 `SkillXxx : SkillCommonModel` 實作 `SkillStart` / `BombFish` / `SkillEnd`

### 8.4 BOSS 怪物（選項 E/G）

- SO 中設定 `isBoss = true`
- `MonsterManager.SpawnMonster()` 會自動將其加入 `m_AllMonsterKingList`
- 若有進度條，需在 `Dino_EventType.BossProgressBar` 新增事件
- 選項 E（BOSS W2）：捕獲走 WeaponSystem，參考霸王龍
- 選項 G（BOSS Skill）：捕獲走 SkillSystem，參考阿力肯（含砲台養成 `EquipmentSystem` 整合）

### 8.5 BOSS 特殊機制（選項 H）

參考史萊姆王（`DINO_SLIME_KING`）：
- 使用 `f8` 封包（`SlimeKingProgressBar`）接收計量條進度
- 有分裂機制（`DINO_SPLIT_SLIME`）
- 需在 `MonsterSystem` 中處理 `f8` 封包的特殊邏輯
- 需在 `Dino_EventType.BossProgressBar` 新增對應事件

---

## 第九步：建立演出控制器（若 hasFeature = true）

演出控制器遵循共用規範，使用 UniTask 非同步流程。

### 演出控制器範本

```csharp
using System;
using System.Threading;
using Cysharp.Threading.Tasks;
using UnityEngine;
using static DinoPinBall.Dino_EventManager;

namespace DinoPinBall
{
    /// <summary>
    /// {中文名稱} 演出控制器
    /// </summary>
    public class {MonsterName}FeatureCtrl : MonoBehaviour
    {
        private CancellationTokenSource _cts;

        [SerializeField] private float _selfScale = 1.0f;
        [SerializeField] private float _otherScale = 0.6f;

        public void Init(bool isPlayerSelf)
        {
            // Init 先清後建
            _cts?.Cancel();
            _cts?.Dispose();
            _cts = new CancellationTokenSource();
            RunFeatureProcess(isPlayerSelf).Forget();
        }

        private async UniTaskVoid RunFeatureProcess(bool isPlayerSelf)
        {
            try
            {
                var token = _cts.Token;

                // 鎖定砲台（本家）
                if (isPlayerSelf)
                {
                    Dino_EventManager.SendEvent(
                        Dino_EventType.eEventWeapon.LockPlayerShoot, true);
                }

                // 設定縮放
                float scale = isPlayerSelf ? _selfScale : _otherScale;
                transform.localScale = Vector3.one * scale;

                await PlayOpening(token);
                await PlayFeatureLoop(token);

                if (isPlayerSelf)
                    await ShowResult(token);

                OnFeatureEnd(isPlayerSelf);
            }
            catch (OperationCanceledException)
            {
                // 取消時也要清理中間狀態（砲台可能已鎖定）
                OnFeatureEnd(isPlayerSelf);
            }
            catch (Exception e)
            {
                Debug.LogError($"[{MonsterName}Feature] Error: {e}");
                OnFeatureEnd(isPlayerSelf);
            }
        }

        private async UniTask PlayOpening(CancellationToken token)
        {
            // TODO: 開場演出
            await UniTask.Delay(500, cancellationToken: token);
        }

        private async UniTask PlayFeatureLoop(CancellationToken token)
        {
            // TODO: 主要演出循環
            await UniTask.Delay(1000, cancellationToken: token);
        }

        private async UniTask ShowResult(CancellationToken token)
        {
            // TODO: 結算面板（僅本家）
            await UniTask.Delay(500, cancellationToken: token);
        }

        /// <summary>
        /// 演出結束，解鎖砲台與按鈕
        /// </summary>
        private void OnFeatureEnd(bool isPlayerSelf)
        {
            if (isPlayerSelf)
            {
                Dino_EventManager.SendEvent(
                    Dino_EventType.eEventWeapon.LockPlayerShoot, false);
            }
        }

        private void OnDestroy()
        {
            _cts?.Cancel();
            _cts?.Dispose();
            _cts = null;
        }
    }
}
```

---

## 驗證檢查清單

### 必要項目

- [ ] `MonsterEnum.cs` — 已新增 `MonsterType` 列舉值
- [ ] `MonsterParameterDataSO` — 已建立 `.asset` + `.asset.meta`（Agent 自動完成）
- [ ] `MonsterManager` — 已點擊「掃描怪物 SO 列表」按鈕更新 `m_MonsterSOList`（⚠️ 需手動在 Unity Editor 操作）
- [ ] Prefab 資源資料夾已建立（`Bundle/Monster/` 下，含子資料夾結構與 AssetBundle 設定）
- [ ] 腳本資料夾已建立（含 `.meta`）
- [ ] `CheatToolManager` 恐龍生成面板可看到新怪物（自動）

### Prefab 結構驗證（⚠️ 必須引用 `#dinopinball_monseter_prefabs_standard` 進行檢查）

Agent 在設定 Prefab Component 時，**必須先讀取 `dinopinball_monseter_prefabs_standard.md`**，依據其中的 Hierarchy 檢查清單逐項驗證：

- [ ] Root Tag = `Monster`、Layer = `FISH_ZORDER` (18)
- [ ] Root 必要 Component：Transform → ArkSteerBehavior → SteerForTether → 怪物控制器腳本 → PathController（Animator 選用）
- [ ] Colliders/ Tag = `Monster`（子彈 `OnTriggerEnter` 用 `CompareTag("Monster")` 判斷命中）
- [ ] Colliders/ Collider `isTrigger = true`（子彈碰撞偵測用 `OnTriggerEnter`）
- [ ] SerializeField 引用正確（`m_shape`、`m_monsterAnimator`、`monsterController`、`m_lightingMaterial`）
- [ ] Effects/ 子物件存在（Freeze inactive、ThunderTwoTarget）

### ⚠️ Agent 完成 Prefab 設定後，必須提醒使用者手動完成以下項目

Agent 透過 unity-cli 可自動完成 Component 新增、子物件建立、Tag/Layer 設定，但以下項目**需要使用者在 Unity Editor Inspector 中手動操作**：

1. **PathAnimal / BaseMonster SerializeField 引用**（拖拉設定）：
   - `m_shape` → Shape 子物件（模型容器）
   - `m_monsterAnimator` → Root 上的 Animator
   - `m_runAnimator` → Shape 子物件下的 Animator
   - `monsterController` → Root 上的 PathController
   - `m_lightingMaterial` → Lightning 材質（`*_Lightning.mat`）
2. **SphereCollider radius** → 調整到適合怪物模型大小
3. **Animator Controller** → Root Animator 指定對應的 AnimatorController 資源

### Debug 工具（選項 B/C/D/E/G/H，詢問使用者是否需要）

- [ ] `{MonsterName}Debug.cs` — 已建立 Debug 工具腳本
- [ ] 假召喚功能正常（透過 `CheatToolManager.debugCreateDino`）
- [ ] （翻倍怪物）W2 擊殺封包測試功能正常（直接建構 JSON 發送給 `WeaponSystem.OnMessage`）
- [ ] （翻倍怪物）F7 倍率更新測試功能正常（直接建構 JSON 發送給 `MonsterSystem.OnMessage`）
- [ ] （技能怪物）sk_xxx 技能觸發測試功能正常（直接建構 JSON 發送給 `SkillSystem.OnMessage`）
- [ ] （BOSS）`DebugSystem` / `CheatToolManager` 已新增專屬測試按鈕

### 翻倍怪物專用（選項 B）

- [ ] SO 中 `winingOddsList` 已設定
- [ ] `MonsterSystem` 已處理 f7 封包（`onReceiveMonsterFeatureEvent`）

### 技能怪物專用（選項 C/D）

- [ ] `ESkillType` 已新增技能類型
- [ ] 選項 C：`SkillSystem.SetSkillObjects()` 已實例化技能物件，`Skill{MonsterName}Object` 已建立並註冊指令
- [ ] 選項 D：`SkillCommonObject` 的 fishSkillDict 已新增對應，`SkillXxx : SkillCommonModel` 已建立

### 空殼模板專用（選項 F）

- [ ] 怪物控制器已建立（繼承 `PathAnimal`，覆寫 `Init` / `CustomInit` / `Kill`）

### BOSS 專用（選項 E/G/H）

- [ ] SO 中 `isBoss = true`
- [ ] `Dino_EventType.BossProgressBar` 已新增事件（若有進度條）
- [ ] 選項 H：`MonsterSystem` 已處理 f8 封包特殊邏輯

### 選用項目

- [ ] `Dino_EventType.cs` — 已新增事件類型（若有特殊演出）
- [ ] 怪物控制器（繼承 `BaseMonster`）
- [ ] 演出控制器（UniTask 非同步流程）

---

## 與 OceanTreasure 新增魚種的關鍵差異

| 面向 | OceanTreasure (2D) | DinoPinBall (3D) |
|------|-------------------|-----------------|
| 怪物資料來源 | 程式碼陣列 (`FishType[]`, `FishKind[]`) | Excel 表單 + ScriptableObject |
| 類型定義 | 連續索引 enum（需跳號佔位） | 離散值 enum（不需佔位） |
| 測試工具 | 手動新增 `FishTestData` | 自動從 `MonsterParameterDataDict` 讀取 |
| 假生成方式 | `FishHunterFakeServerPacket` 三步驟 | `DebugManager.TestCreateDinoOnlyClient()` |
| 系統存取 | 具名 getter (`getFishSystem()`) | 泛型 (`GetSystem<MonsterSystem>()`) |
| 事件系統 | `FishHunter_EventManager` | `Dino_EventManager` |
| 怪物基類 | `Fish2D` | `BaseMonster` |
| 怪物管理器 | `FishMaintainer` | `MonsterManager` |

---

## 快速參考

### 關鍵檔案位置

| 檔案 | 用途 |
|------|------|
| `DinoGame/DinoPinBall/Game/MonsterSystem/Enum/MonsterEnum.cs` | `MonsterType` 列舉定義 |
| `DinoGame/DinoPinBall/Game/MonsterSystem/MonsterParameterDataSO.cs` | SO 類別定義（含 `ToRuntimeData()` 轉換方法） |
| `DinoGame/DinoPinBall/Game/MonsterSystem/MonsterManager.cs` | 怪物實體管理器（直接持有 `m_MonsterSOList`，含「掃描怪物 SO 列表」按鈕） |
| `DinoGame/DinoPinBall/Game/MonsterSystem/MonsterSystem.cs` | Server 指令處理 + `MonsterParameterData` |
| `DinoGame/DinoPinBall/Game/MonsterSystem/Common/BaseMonster.cs` | 怪物基類 |
| `DinoGame/DinoPinBall/Game/Common/EventManager/Dino_EventType.cs` | 事件類型定義 |
| `DinoGame/DinoPinBall/Game/SkillSystem/SkillSystem.cs` | 技能系統 + `ESkillType` |
| `DinoGame/DinoPinBall/Game/DebugSystem/DebugManager.cs` | Debug 假生成 |
| `DinoGame/DinoPinBall/Game/DebugSystem/CheatToolManager.cs` | 測試工具 UI |
| `DinoGame/DinoPinBall/Game/DebugSystem/DebugSystem.cs` | Debug 指令發送 |

### 關鍵 API

| 功能 | API |
|------|-----|
| 取得怪物系統 | `GameClient.Instance.GetSystem<MonsterSystem>()` |
| 取得怪物管理器 | `MonsterManager.Instance` |
| 依 SID 尋找怪物 | `MonsterManager.Instance.GetMonsterBySid(sid)` |
| 依來源取得怪物列表 | `MonsterManager.Instance.GetMonstersBySource(MonsterSource)` |
| 依代號取得怪物列表 | `MonsterManager.Instance.GetMonstersByCode(serverName)` |
| 生成怪物 | `MonsterManager.Instance.SpawnMonster(monsterType, callback)` |
| Client 假生成 | `DebugManager.TestCreateDinoOnlyClient(monsterType, count, fakeKill)` |
| 發送事件 | `Dino_EventManager.SendEvent(enum, args)` |
| 註冊事件 | `Dino_EventManager.Registration(enum, callback)` |

---

## 使用範例

### 範例 1：新增一般怪物

使用者：「新增一隻火焰龍(39026)，Juras 廳，一般怪物，倍率 50~200」

執行：
1. `MonsterEnum.cs` 新增 `FLAME_DRAGON = 39026`
2. Agent 自動建立 `Juras_FlameDragon.asset` + `.asset.meta`（monsterType=39026, serverName="FLAME_DRAGON", minOdds=50, maxOdds=200, monsterSource=0）
   - 路徑：`Assets/FishtHunter_Res/CrazyDino/Bundle/ScriptableObject/MonsterParameter/StoneAge/`
3. 提醒使用者在 Unity Editor 中點擊 `MonsterManager` Inspector 的「掃描怪物 SO 列表」按鈕
4. 建立 Prefab 資源資料夾 `Assets/FishtHunter_Res/CrazyDino/Bundle/Monster/FlameDragon/`（含空子資料夾 + 複製參考怪物主 Prefab 改名為 `FlameDragon.prefab`）
5. 建立腳本資料夾 `Assets/FishHunter_Script/DinoGame/DinoPinBall/Game/MonsterSystem/FlameDragon/`
6. 第六步跳過（一般怪物不需控制器）
7. CheatToolManager 自動可見

### 範例 2：新增翻倍怪物

使用者：「新增黃金鳳凰(9103)，Dino 廳，翻倍怪物，倍率 100~500，翻倍陣列 [2,3,5,8]」

執行：
1. `MonsterEnum.cs` 新增 `GOLDEN_PHOENIX = 9103`
2. Agent 自動建立 `Dino_GoldenPhoenix.asset` + `.asset.meta`（winingOddsList=[2,3,5,8], monsterSource=0）
   - 路徑：`Assets/FishtHunter_Res/CrazyDino/Bundle/ScriptableObject/MonsterParameter/StoneAge/`
3. 提醒使用者在 Unity Editor 中點擊 `MonsterManager` Inspector 的「掃描怪物 SO 列表」按鈕
4. 建立 Prefab 資源資料夾 `Assets/FishtHunter_Res/CrazyDino/Bundle/Monster/GoldenPhoenix/`
5. 建立資源/腳本資料夾
6. 處理 f7 封包邏輯（若有特殊狀態更新）

### 範例 3：新增 BOSS 技能怪物

使用者：「新增冰霜巨龍(39027)，Juras 廳，BOSS，走 SkillSystem」

執行：
1. `MonsterEnum.cs` 新增 `FROST_DRAGON = 39027`
2. Agent 自動建立 `Juras_FrostDragon.asset` + `.asset.meta`（isBoss=1, monsterSource=0）
   - 路徑：`Assets/FishtHunter_Res/CrazyDino/Bundle/ScriptableObject/MonsterParameter/StoneAge/`
3. 提醒使用者在 Unity Editor 中點擊 `MonsterManager` Inspector 的「掃描怪物 SO 列表」按鈕
4. `ESkillType` 新增 `FrostDragon = {id}`
5. 建立 `SkillFrostDragonObject` 並在 `SetSkillObjects()` 實例化
6. `Dino_EventType.BossProgressBar` 新增 `FROST_DRAGON_EVENT`（若有進度條）
7. 建立 Prefab 資源資料夾 `Assets/FishtHunter_Res/CrazyDino/Bundle/Monster/FrostDragon/`
8. 建立腳本資料夾
9. 建立演出控制器
10. `DebugSystem` / `CheatToolManager` 新增專屬測試按鈕

### 範例 4：新增空殼模板

使用者：「新增喀耳刻女巫(39028)，Juras 廳，空殼模板，倍率 100~300」

執行：
1. `MonsterEnum.cs` 新增 `CIRCE_WITCH = 39028`
2. Agent 自動建立 `Juras_CirceWitch.asset` + `.asset.meta`（monsterType=39028, serverName="CIRCE_WITCH", minOdds=100, maxOdds=300, monsterSource=0）
   - 路徑：`Assets/FishtHunter_Res/CrazyDino/Bundle/ScriptableObject/MonsterParameter/StoneAge/`
3. 提醒使用者在 Unity Editor 中點擊 `MonsterManager` Inspector 的「掃描怪物 SO 列表」按鈕
4. 建立 Prefab 資源資料夾 `Assets/FishtHunter_Res/CrazyDino/Bundle/Monster/CirceWitch/`（含空子資料夾 + 複製參考怪物主 Prefab 改名為 `CirceWitch.prefab`）
5. 建立腳本資料夾 `Assets/FishHunter_Script/DinoGame/DinoPinBall/Game/MonsterSystem/CirceWitch/`
6. 建立空殼控制器 `CirceWitchCtrl.cs`（繼承 `PathAnimal`，覆寫 `Init` / `CustomInit` / `Kill`，含 TODO 標記）
7. 第七步跳過（空殼模板不需 Debug 腳本）
8. CheatToolManager 自動可見
