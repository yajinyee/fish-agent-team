---
inclusion: manual
---

# OVDF (Odin Validator Diff Format) 套用指南

## 什麼是 OVDF

OVDF 是 Odin Inspector Design Tool 產生的私有差異格式文件（無公開文件），描述對 C# 類別中欄位或方法的 Attribute 變更。
使用者會貼上 OVDF 內容，Agent 需要解讀並自動套用到對應的 .cs 原始碼中。

## OVDF 格式規範

### Header（檔頭）

```
OVDF v1.1
{Namespace}.{ClassName}, {AssemblyName}
MetaGuid:{guid}
```

- 第一行：版本號
- 第二行：完整類別名稱（含 Namespace）和 Assembly 名稱
- 第三行：Unity meta GUID（用於定位檔案，通常不需要）

### 目標宣告

每個目標以 `# targetName` 開頭：
- `# fieldName` — 欄位
- `# MethodName()` — 方法（帶括號）
- `# $RandomId` — 匿名群組（`$` 開頭，Visual Designer 自動生成的 ID）

#### 匿名群組（`$` 開頭）

當目標名稱以 `$` 開頭時，代表 Visual Designer 新建了一個佈局群組（如 `HorizontalGroup`、`VerticalGroup`）。
這個 ID 本身沒有意義，只是用來讓其他欄位的 Position 引用。

處理方式：
1. 讀取該匿名群組的 `+ [Attribute]`，確認群組類型（如 `HorizontalGroupAttribute`）
2. 讀取該匿名群組的 `Position`，確認它歸屬於哪個父群組
3. 為這個匿名群組取一個有意義的巢狀路徑名稱（格式：`"父群組/子群組名"`）
4. 找到所有 Position 引用此匿名 ID 的欄位，將它們的群組 Attribute 改為這個巢狀路徑

範例：
```
# $JWDoamfCpho284o5F9QoQa
Position: "假生成封包設定":2
+ [Sirenix.OdinInspector.HorizontalGroupAttribute]

# customPath
Position: $JWDoamfCpho284o5F9QoQa:0

# useCustomPath
Position: $JWDoamfCpho284o5F9QoQa:1
```

解讀：
- 新建一個 `[HorizontalGroup]`，歸屬於 `"假生成封包設定"`，排在第 2 位
- `customPath` 放在這個群組的第 0 位（左邊）
- `useCustomPath` 放在這個群組的第 1 位（右邊）

轉換結果：
- 匿名 ID 轉為巢狀路徑：`"假生成封包設定/CustomPathGroup"`（根據包含的欄位語意命名）
- `customPath` 加上 `[HorizontalGroup("假生成封包設定/CustomPathGroup")]`
- `useCustomPath` 加上 `[HorizontalGroup("假生成封包設定/CustomPathGroup")]`
- 兩者原本的 `[BoxGroup("假生成封包設定")]` 被 `[HorizontalGroup]` 取代（巢狀群組會自動歸屬父群組）
- 程式碼中的欄位順序按 Position 數字排列（:0 在前，:1 在後）

### Position（歸屬群組）

```
Position: "GroupName":N
```

- `GroupName`：該目標應歸屬的 `[BoxGroup]` / `[TabGroup]` / `[FoldoutGroup]` 名稱
- `N`：在群組中的排序位置（僅供參考）
- **必須驗證**：程式碼中的群組名稱是否與 OVDF 一致，不一致則同步修正

### 變更指令

| 前綴 | 意義 | 說明 |
|------|------|------|
| `+ [Attribute]` | 新增 Attribute | 在目標上加入新的 Attribute |
| `- [Attribute]` | 移除 Attribute | 從目標上移除指定的 Attribute |
| `* [Attribute]` | 修改 Attribute | 修改現有 Attribute 的參數 |

### Attribute 參數格式

```
+ [Sirenix.OdinInspector.EnableIfAttribute]
Condition = "useCustomPath"
```

- Attribute 名稱使用完整命名空間路徑
- 參數以 `Key = Value` 格式列在下一行
- 多個參數各佔一行
- 帶有 `HasDefined` 前綴的參數是內部標記，表示該參數已被明確設定

## 套用規則

### 1. 定位檔案和目標

根據 Header 中的 `{Namespace}.{ClassName}` 找到對應的 .cs 檔案，然後找到目標欄位或方法。

### 2. Position 檢查（群組歸屬驗證）

**必須檢查**：程式碼中目標的群組 Attribute 名稱是否與 OVDF 的 Position GroupName 一致。
- 一致 → 正常套用
- 不一致 → **同時修正群組 Attribute 的名稱**

### 3. Attribute 名稱簡化

OVDF 使用完整路徑，套用到程式碼時：
- 如果檔案已有 `using Sirenix.OdinInspector;`，移除命名空間前綴
- 移除 `Attribute` 後綴（`EnableIfAttribute` → `EnableIf`）

### 4. 操作對應

| OVDF 操作 | 程式碼動作 |
|-----------|-----------|
| `+ [XxxAttribute]` | 新增 `[Xxx]` |
| `- [XxxAttribute]` | 移除 `[Xxx]` |
| `* [XxxAttribute]` | 修改現有 `[Xxx]` 的參數 |
| `+` 加 `-` 同類型 | 替換（移除舊的，加入新的） |

### 5. 參數轉換規則

#### 簡單 Attribute（單一主參數）

| OVDF 參數 | C# 寫法 |
|-----------|---------|
| `Condition = "xxx"` | 建構子第一個參數：`[EnableIf("xxx")]` |
| `Text = "xxx"` | 建構子第一個參數：`[LabelText("xxx")]` |
| `MemberName = "xxx"` | 建構子第一個參數：`[ShowIf("xxx")]` |

#### 複雜 Attribute（多參數，如 ButtonAttribute）

ButtonAttribute 的 OVDF 參數對應（從 Odin XML 文件驗證）：

**重要規則**：OVDF 中 Field 用原始名稱，Property 用 backing field 名稱（小寫開頭）。轉換到 C# 命名參數時，Property 需要用大寫開頭的 Property 名稱。

| OVDF 參數 | C# 成員類型 | C# 寫法 | 說明 |
|-----------|------------|---------|------|
| `Name = "xxx"` | Field | 建構子第一個參數 `"xxx"` | 按鈕顯示文字 |
| `Style = CompactBox` | Field | 建構子第二個參數 `ButtonStyle.CompactBox` 或命名參數 | 按鈕樣式 |
| `Expanded = False` | Field | 命名參數 `Expanded = false` | 是否展開參數 |
| `Icon = SendCheckFill` | Field | 命名參數 `Icon = SdfIconType.SendCheckFill` | SDF 圖示 |
| `DisplayParameters = True` | Field | 命名參數 `DisplayParameters = true` | 是否顯示參數 |
| `DirtyOnClick = True` | Field | 命名參數 `DirtyOnClick = true` | 點擊是否標記 dirty |
| `buttonHeight = 50` | Property (backing) | 命名參數 `ButtonHeight = 50` | 按鈕高度（px） |
| `buttonIconAlignment = LeftOfText` | Property (backing) | 命名參數 `IconAlignment = IconAlignment.LeftOfText` | 圖示對齊 |
| `buttonAlignment = 0` | Property (backing) | 命名參數 `ButtonAlignment = 0f` | 按鈕對齊（0~1） |
| `stretch = True` | Property (backing) | 命名參數 `Stretch = true` | 是否填滿寬度 |
| `HasDefined*` | 內部標記 | **忽略，不轉換** | 表示該參數已被明確設定 |

#### ButtonAttribute 建構子（從 Odin XML 文件驗證）

```csharp
// 無參數
[Button]

// 只有大小
[Button(ButtonSizes.Large)]
[Button(50)]  // 像素高度

// 只有名稱
[Button("名稱")]

// 名稱 + 大小
[Button("名稱", ButtonSizes.Large)]
[Button("名稱", 50)]

// 只有樣式
[Button(ButtonStyle.Box)]

// 大小 + 樣式
[Button(ButtonSizes.Large, ButtonStyle.Box)]
[Button(50, ButtonStyle.Box)]

// 名稱 + 樣式
[Button("名稱", ButtonStyle.CompactBox)]

// 名稱 + 大小 + 樣式
[Button("名稱", ButtonSizes.Large, ButtonStyle.Box)]
[Button("名稱", 50, ButtonStyle.Box)]

// 圖示
[Button(SdfIconType.SendCheckFill)]
[Button(SdfIconType.SendCheckFill, IconAlignment.LeftOfText)]
[Button(SdfIconType.SendCheckFill, "名稱")]
```

套用時根據 OVDF 參數的組合選擇最適合的建構子，其餘用命名參數。

### 6. Enum 值轉換

OVDF 中的 enum 值不帶類型前綴，轉換時需要加上：

| OVDF 值 | C# 值 |
|---------|-------|
| `CompactBox` | `ButtonStyle.CompactBox` |
| `Box` | `ButtonStyle.Box` |
| `FitContent` | `ButtonStyle.FitContent` |
| `SendCheckFill` | `SdfIconType.SendCheckFill` |
| `LeftOfText` | `IconAlignment.LeftOfText` |
| `RightOfText` | `IconAlignment.RightOfText` |
| `Large` | `ButtonSizes.Large` |
| `Medium` | `ButtonSizes.Medium` |

## 範例

### 範例 1：欄位 Attribute 替換

**OVDF 輸入**：
```
# customPath
+ [Sirenix.OdinInspector.EnableIfAttribute]
Condition = "useCustomPath"
- [Sirenix.OdinInspector.ShowIfAttribute]
```

**套用結果**：
- 移除 `[ShowIf("useCustomPath")]`
- 新增 `[EnableIf("useCustomPath")]`

### 範例 2：方法 Button Attribute 修改

**OVDF 輸入**：
```
# SpawnMonster()
Position: "假生成封包設定":5
* [Sirenix.OdinInspector.ButtonAttribute]
Name = "生成指定怪物"
Style = CompactBox
Icon = SendCheckFill
buttonIconAlignment = LeftOfText
HasDefinedButtonIconAlignment = True
buttonHeight = 50
HasDefinedButtonHeight = True
Stretch = True
HasDefinedStretch = True
```

**套用結果**：
1. 驗證 `[BoxGroup]` 名稱是否為 `"假生成封包設定"`，不一致則修正
2. 修改 `[Button]`：
```csharp
// 修改前
[Button("生成怪物", ButtonSizes.Large)]

// 修改後
[Button("生成指定怪物", ButtonStyle.CompactBox, Icon = SdfIconType.SendCheckFill, IconAlignment = IconAlignment.LeftOfText, ButtonHeight = 50, Stretch = true)]
```

### 範例 3：匿名群組（HorizontalGroup 佈局）

**OVDF 輸入**：
```
# $JWDoamfCpho284o5F9QoQa
Position: "假生成封包設定":2
+ [Sirenix.OdinInspector.HorizontalGroupAttribute]

# fakeCmd
Position: "假生成封包設定":4

# useCustomPath
Position: $JWDoamfCpho284o5F9QoQa:1

# customPath
Position: $JWDoamfCpho284o5F9QoQa:0
```

**解讀步驟**：
1. `$JWDoamfCpho284o5F9QoQa` 是匿名群組 → 新建 `[HorizontalGroup]`，歸屬 `"假生成封包設定"`
2. `customPath` 歸屬匿名群組 Position :0（左邊）
3. `useCustomPath` 歸屬匿名群組 Position :1（右邊）
4. `fakeCmd` 維持在 `"假生成封包設定":4`（只是排序確認，無 Attribute 變更）

**套用結果**：
```csharp
// 修改前
[BoxGroup("假生成封包設定")] [LabelText("使用自定義路徑")] [SerializeField]
private bool useCustomPath;

[BoxGroup("假生成封包設定")] [LabelText("自訂路徑名稱")] [EnableIf("useCustomPath")] [SerializeField]
private string customPath = "";

// 修改後（customPath 排前面 :0，useCustomPath 排後面 :1）
[HorizontalGroup("假生成封包設定/CustomPathGroup")]
[LabelText("自訂路徑名稱")] [EnableIf("useCustomPath")] [SerializeField]
private string customPath = "";

[HorizontalGroup("假生成封包設定/CustomPathGroup")]
[LabelText("使用自定義路徑")] [SerializeField]
private bool useCustomPath;
```

## 注意事項

- 保持原始程式碼的縮排和格式風格
- 如果欄位有多個 Attribute 在同一行，替換時保持同樣的排列方式
- 如果欄位的 Attribute 分多行，替換時保持多行格式
- 不要改動 Attribute 以外的任何程式碼（欄位類型、名稱、預設值等）
- `HasDefined` 前綴的參數一律忽略，不轉換到 C# 程式碼
- 套用後必須執行 `getDiagnostics` 驗證編譯是否通過
