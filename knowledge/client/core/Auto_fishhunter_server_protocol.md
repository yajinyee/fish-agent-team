# FishHunter Server 通訊協定指南 (Server Protocol Guide)

本文件詳細說明 FishHunter 專案中 Client 與 Server 之間的通訊協定。
專案包含兩個獨立的遊戲模式，各有獨立的 BaseSystem 實作：

- **OceanTreasure (海王/ArkGame)**: 主要捕魚模式
- **DinoPinBall (恐龍/DinoGame)**: 恐龍主題彈珠台模式

---

## 1. 架構差異總覽

### 1.1 兩套 BaseSystem 對照

| 面向 | OceanTreasure (海王) | DinoPinBall (恐龍) |
|------|---------------------|-------------------|
| **Namespace** | `ArkGame` | `DinoPinBall` |
| **BaseSystem 位置** | `ArkGame/Scripts/Common/BaseSystem.cs` | `DinoGame/DinoPinBall/BaseSystem.cs` |
| **GameClient 位置** | `ArkGame/Scripts/GameClient.cs` | `DinoGame/DinoPinBall/GameClient.cs` |
| **建構子參數** | `base(gameClient, "name")` | `base(gameDataManager, eventManager, cmdSender, "name")` |
| **系統存取方式** | `getXXXSystem()` 具名方法 | `GetSystem<T>()` 泛型方法 |
| **魚/怪物系統** | `FishSystem` | `MonsterSystem` |
| **事件系統** | `FishHunter_EventManager` | `Dino_EventManager` |
| **onMessage 方法** | `onMessage(JSON json)` | `OnMessage(JSON json)` |

### 1.2 封包格式（兩者相同）

```json
{
    "sys": "fish",      // 系統名稱
    "cmd": "f1",        // 指令代碼
    "data": { ... }     // 資料內容
}
```

---

# Part A: OceanTreasure (海王/ArkGame) 通訊協定

## A.1 BaseSystem 核心機制

```csharp
// 位置：ArkGame/Scripts/Common/BaseSystem.cs
public abstract class BaseSystem
{
    private Dictionary<string, Callback> m_SocketCallback;  // 指令回呼字典
    public string m_Name;                                    // 系統名稱
    
    // 建構子：向 GameClient 註冊系統
    protected BaseSystem(GameClient gameClient, string name);
    
    // 註冊指令處理函式
    public void Register(string cmd, Callback callback);
    
    // 接收 Server 封包並路由到對應的 callback
    public void onMessage(JSON json);
    
    // 發送封包給 Server
    public bool Send(string cmd, JSON data);
}
```

## A.2 系統指令總覽

### A.2.1 GameSystem (game) - 遊戲流程系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `join_table` | S→C | 加入房間回應 | `game_id`, `seat`, `table_name`, `player_id` |
| `leave_table` | S→C | 玩家離開房間 | `seat` |
| `init_game` | C→S | 請求遊戲初始化 | `game_id` |
| `init_game` | S→C | 遊戲初始化資料 | `jp`, `rank_enable`, `jp_enable`, `disable_skill_gem`, `unload_enable`, `kingData` |
| `update_game` | C↔S | 更新遊戲狀態 | `game_id` |
| `marquee` | S→C | 跑馬燈訊息 | 訊息內容 |
| `test` | C→S | 測試指令（加錢） | `coin`, `gem` |
| `pause_game` | C→S | 延長 Socket 等待時間 | `minutes` |
| `hurray_sale` | S→C | 成就禮包通知 | `type` |
| `rookie_finish` | S→C | 新手房完成通知 | - |
| `random_drop` | S→C | 隨機掉落物 | `seat`, 掉落資料 |
| `super_lucky_farm_drop` | S→C | 幸運農場掉落 | `data`, `fish_type_list` |
| `super_lucky_farm_accelerate` | C↔S | 幸運農場加速 | `event_id`, `token_type`, `token_count`, `bean_uuid` |

### A.2.2 FishSystem (fish) - 魚種系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `f0` | C→S | 魚離場請求 | `1` (fish_id) |
| `f1` | S→C | 生成魚種 | `1`(id), `2`(type), `3`(x), `4`(y), `5`(o), `6`(feature), `8`(size), `17`(path), `23`(extra_data) |
| `f2` | S→C | 魚事件（烈焰風暴/黑龍出場） | `1`(id), `24`(army_name), `25`(army_id), `26`(delay) |
| `f3` | S→C | 場景切換 | `14`(scene) |
| `f4` | S→C | 魚腳本初始化 | `13`(current), `14`(scene), `21`(bg_sec), `11`(fish_group) |
| `f5` | C→S | 心跳函式（維持出魚） | - |
| `f6` | S→C | 魚潮/特殊魚進場宣告 | `2`(type) |
| `f7` | S→C | 特殊魚種資料更新 | `fish_type`, `fish_id`, 各魚種專屬資料 |
| `f8` | C↔S | 場景額外資料（黃金拉霸蟹） | 場景 odds 表 |
| `script_name` | S→C | 腳本名稱 | `name` |

#### f1 魚種資料欄位對照

| 代碼 | 欄位名 | 說明 |
|------|--------|------|
| `1` | id | 魚的唯一 SID |
| `2` | type | 魚種類型 (enumFishType) |
| `3` | x | X 座標 (0-100 百分比) |
| `4` | y | Y 座標 (0-100 百分比) |
| `5` | o | 朝向角度 |
| `6` | feature | 特殊魚代碼 |
| `7` | position | 特殊魚在串魚中的位置 |
| `8` | size | 串魚數量 |
| `9` | frame | 生成時間戳（用於同步） |
| `17` | path | 路徑魚路徑名稱 |
| `18` | teleport | 是否為召喚魚 (1=是) |
| `19` | extra_type | 額外類型（復活節魚、俄羅斯娃娃） |
| `20` | min_bet | 最低押注限制 |
| `23` | extra_data | 額外資料（泡泡魚道具等） |
| `28` | min_vip | 最低 VIP 限制 |

### A.2.3 WeaponSystem (weapon) - 武器系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `w1` | C↔S | 一般射擊 | `1`(id), `2`(seat), `4`(x), `5`(y), `9`(bet) |
| `w2` | C↔S | 子彈命中（捕獲結果） | `1`(id), `2`(seat), `7`(fish_id_list), `8`(fish_dead_dict), `9`(bet), `10`(odds), `11`(coin) |
| `w3` | C↔S | 鎖定射擊 | `1`(id), `2`(seat), `6`(fish_id), `9`(bet), `46`(fish_extradata) |
| `w4` | C↔S | 狂暴射擊 | `1`(id), `2`(seat), `6`(fish_id), `9`(bet), `46`(fish_extradata) |
| `w5` | C↔S | 電擊射擊 | `1`(id), `2`(seat), `7`(fish_id_list), `9`(bet), `46`(fish_extradata) |
| `w6` | C↔S | 廣播 Auto 狀態 | `2`(seat), `25`(player_auto) |
| `w7` | C↔S | 雷鳴射擊 | `1`(id), `2`(seat), `7`(fish_id_list), `27`(bonus_fish_id_list), `9`(bet), `46`(fish_extradata) |
| `w17` | S→C | 魚技能返還初始化 | - |
| `w18` | S→C | 魚技能返還獎勵 | - |
| `w31` | C↔S | 雙重射擊 | 同 w1，`1`(id) 為陣列 |
| `w32` | C↔S | 雙重命中 | 同 w2，`1`(id) 為陣列 |
| `w33` | C↔S | 雙重鎖定射擊 | 同 w3，`1`(id) 為陣列 |

#### w2 捕獲結果重要欄位

| 代碼 | 欄位名 | 說明 |
|------|--------|------|
| `3` | credits | 玩家當前金幣 |
| `8` | fish_dead_dict | 死亡魚資料字典 |
| `10` | odds | 倍率 |
| `11` | coin | 贏得金幣 |
| `12` | gem | 贏得寶石 |
| `13` | item_dict | 掉落道具 |
| `29` | extra_dict | 額外資料（武器能量等） |
| `39` | showAwardType | 報獎類型 |
| `40` | bonus_item | 掉落物列表 |
| `42` | client_show | 演出資料 |
| `43` | drop_reward_item | 掉落物列表（新協定） |

### A.2.4 PlayerSystem (player) - 玩家系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `init_player` | C→S | 請求玩家初始化 | `name`, `exdata`, `lang`, `channel` |
| `init_player` | S→C | 玩家初始化資料 | `weapon_skin`, `bet_value`, `name`, `id`, `coin`, `gem`, `lv`, `vip_lv`, `country`, `bet_list`, `mission_detail` |
| `init_account` | S→C | 帳號初始化 | - |
| `update_player` | C→S | 請求更新玩家 | `game_id` |
| `update_player` | S→C | 所有玩家資料 | 各玩家的 `seat`, `weapon_skin`, `bet_value`, `name`, `coin`, `gem`, `skillData` |
| `update_bet` | C→S | 切換押注 | `seat`, `value`, `type` |
| `update_bet` | S→C | 押注更新結果 | `seat`, `bet_value`, `jp_value` |
| `reflash` | C→S | 同步金幣（購買/領獎後） | `type`, `coin`, `gem`, `product_id`, `price` |
| `reflash` | S→C | 金幣同步結果 | `coin`, `gem`, `vip_lv`, `change_coin`, `change_gem` |
| `get_weapon_skins` | C↔S | 取得武器外觀 | - |
| `change_weapon_skin` | C↔S | 更換武器外觀 | - |
| `refresh_reward_item_data` | C→S | 請求存入置獎 DB | - |
| `fishing_boat_mission_complete` | S→C | 捕魚趣任務完成 | - |
| `reload_fishing_boat` | C→S | 捕魚趣領獎 | - |

### A.2.5 SkillSystem (skill) - 技能系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `sk_icons` | C↔S | 取得/更新技能道具 | 道具列表 |
| `skc_icons` | S→C | 武器卡數量 | - |
| `exsummon_cards_init` | S→C | 召王卡初始化 | - |
| `play_bonus_init` | C↔S | PlayBonus 初始化 | - |
| `play_bonus_use` | C↔S | 使用 PlayBonus | - |

#### 技能魚種專用指令（動態註冊）

| 指令前綴 | 說明 | 範例 |
|----------|------|------|
| `sk_electric` | 電磁蟹 | `sk_electric`, `sk_electric_use`, `sk_electric_hit` |
| `sk_drill` | 鑽頭蟹 | `sk_drill`, `sk_drill_hit` |
| `sk_firestorm` | 烈焰風暴 | `sk_firestorm_start`, `sk_firestorm_hit`, `sk_firestorm_end` |
| `sk_wheel` | 轉輪蟹 | `sk_wheel`, `sk_wheel_result` |
| `sk_luckycat` | 招財貓 | `sk_luckycat`, `sk_luckycat_hit` |
| `sk_blackdragon` | 黑龍 | `sk_blackdragon`, `sk_blackdragon_hit` |
| `sk_bison` | 野牛 | `sk_bison`, `sk_bison_hit` |
| `sk_samurai` | 武士魚 | `sk_samurai`, `sk_samurai_hit` |
| `sk_fullmetal` | 鳳雷魚 | `sk_fullmetal`, `sk_fullmetal_hit` |
| `sk_wukong` | 悟空魚 | `sk_wukong`, `sk_wukong_hit` |
| `sk_thunderdragon` | 雷霆帝龍 | `sk_thunderdragon`, `sk_thunderdragon_hit` |
| `sk_infinitywheel` | 無限轉輪 | `sk_infinitywheel`, `sk_infinitywheel_result` |
| `sk_machinecrab` | 機械霸王蟹 | `sk_machinecrab_laser`, `sk_machinecrab_shotgun`, `sk_machinecrab_explosion` |
| `sk_kuromu` | 庫洛姆 | `sk_kuromu`, `sk_kuromu_hit` |

#### 通用技能模組指令 (SkillCommonObject)

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `sk_skill_start` | S→C | 技能開始 | `fish_id`, `fish_type`, `player_seat`, `skill_data` |
| `sk_create_army` | S→C | 生成魚潮 | `fish_id`, `fish_type`, `army_id`, `army` |
| `sk_bomb_fish` | C↔S | 炸魚請求/結果 | `fish_id`, `fish_type`, `hit_list`, `this_win`, `fish_bombed_list` |
| `sk_end` | C↔S | 技能結束 | `fish_id`, `fish_type`, `total_win`, `total_odds` |

### A.2.6 JackpotSystem (jackpot) - Jackpot 系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `get_jackpot_info` | C→S | 請求 JP 資訊 | - |
| `get_jackpot_info` | S→C | JP 資訊回應 | `jp`, `bet_gate`, `vip_level`, `max_gate`, `gate_value` |
| `get_jackpot_history` | C→S | 請求 JP 歷史 | - |
| `get_jackpot_history` | S→C | JP 歷史回應 | `history` (陣列) |
| `win_jackpot` | S→C | 中 JP 通知 | `win`, `jp`, `seat`, `win_info` |

### A.2.7 MissionSystem (mission) - 任務系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `mission_update` | S→C | 任務進度更新 | 任務資料 |
| `mission_rewards` | C→S | 請求領獎 | - |
| `mission_rewards` | S→C | 領獎資料 | 獎勵資料 |
| `mission_rewards_finish` | C→S | 確認領獎 | - |
| `mission_rewards_finish` | S→C | 領獎完成 | - |
| `test_mission` | C→S | 測試任務（DEBUG） | `mission_id`, `degree_id` |

### A.2.8 SocialSystem (social) - 社交系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `get_emojis` | C→S | 請求表情包 | - |
| `get_emojis` | S→C | 表情包資料 | 表情包列表 |
| `emoji` | C→S | 發送表情 | `pack`, `id` |
| `emoji` | S→C | 接收表情 | `seat`, `pack`, `id` |
| `get_businesscard_info` | C→S | 請求名片框資訊 | - |
| `get_businesscard_info` | S→C | 名片框資訊 | `businesscard` (陣列) |
| `change_businesscard` | C→S | 更換名片框 | `businesscard` |
| `change_businesscard` | S→C | 更換結果 | `status`, `seat`, `businesscard`, `effect_time` |
| `get_businesscard_effect` | C→S | 請求名片框效益 | - |
| `get_businesscard_effect` | S→C | 名片框效益 | `status`, `reward`, `effect_time` |

### A.2.9 DailyRankSystem (rank) - 每日排行系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `now` | C→S | 請求當前排行 | `s` (起始), `l` (筆數) |
| `now` | S→C | 當前排行資料 | `my_rank`, `total`, `leader` |
| `last` | C→S | 請求上次排行 | `s`, `l` |
| `last` | S→C | 上次排行資料 | `my_rank`, `total`, `leader` |
| `settle` | S→C | 活動結算通知 | - |
| `info` | C→S | 請求排行榜資訊 | - |
| `info` | S→C | 排行榜資訊 | `display_num`, `title`, `rule`, `session_start_time`, `session_end_time`, `remaining_time` |

### A.2.10 其他系統

#### ItemSystem (item) - 道具系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `use` | C→S | 使用道具 | `seat`, `id`, `name`, `type`, `value` |
| `use` | S→C | 使用結果 | - |
| `gain` | S→C | 獲得道具 | `seat`, `item` |

#### TableSystem (table) - 桌次系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `shutdown` | S→C | 伺服器關閉通知 | `min` (剩餘分鐘) |

#### TournamentRankSystem (tournament) - 錦標賽系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `info` | C↔S | 錦標賽資訊 | - |

#### RookieMissionSystem (rookie) - 新手任務系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `init` | C↔S | 新手任務初始化 | - |
| `update` | S→C | 任務進度更新 | - |
| `complete` | S→C | 任務完成 | - |

#### TokenSystem (collect) - Token 代幣系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `init` | C→S | 初始化 Token 系統 | - |
| `collect` | C↔S | 收集 Token | - |

---

# Part B: DinoPinBall (恐龍/DinoGame) 通訊協定

## B.1 BaseSystem 核心機制

```csharp
// 位置：DinoGame/DinoPinBall/BaseSystem.cs
public abstract class BaseSystem : IBaseSystemBehaviour
{
    private readonly Dictionary<string, Callback> SocketCallback;
    private readonly string name;
    protected IGameDataManager gameDataManager;
    protected IEventManager eventManager;
    public ICmdSender cmdSender;
    
    // 建構子：依賴注入模式
    protected BaseSystem(IGameDataManager gameDataManager, IEventManager eventManager, 
                         ICmdSender cmdSender, string name);
    
    // 註冊指令處理函式
    public void Register(string cmd, Callback callback);
    
    // 接收 Server 封包（注意：方法名為 OnMessage，首字母大寫）
    public void OnMessage(JSON json);
    
    // 發送封包給 Server
    public bool Send(string cmd, JSON data);
    
    // 生命週期方法
    public virtual void Init();
    public virtual void Destroy();
    public virtual void Update();
}
```

### B.1.1 介面定義

```csharp
public interface ISystemManager {
    void AddSystem<T>() where T : BaseSystem;
    void RemoveSystem<T>() where T : BaseSystem;
    T GetSystem<T>() where T : BaseSystem;
}

public interface ICmdSender {
    bool Send(string cmd, JSON data, string sys);
}

public interface IGameDataManager {
    void AddGameData<T>() where T : BaseGameData;
    void RemoveGameData<T>() where T : BaseGameData;
    T GetGameData<T>() where T : BaseGameData;
}
```

## B.2 系統指令總覽

### B.2.1 GameSystem (game) - 遊戲流程系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `join_table` | S→C | 加入房間回應 | `game_id`, `seat` |
| `leave_table` | S→C | 玩家離開房間 | `seat` |
| `init_game` | C→S | 請求遊戲初始化 | `game_id` |
| `init_game` | S→C | 遊戲初始化資料 | `Scene`, `jp`, `jp_enable` |
| `marquee` | S→C | 跑馬燈訊息 | 訊息內容 |
| `pause_game` | C→S | 延長 Socket 等待時間 | `minutes` |
| `hurray_sale` | S→C | 成就禮包通知 | - |
| `random_drop` | S→C | 隨機掉落物 | `seat`, 掉落資料 |

**與海王差異**：
- 無 `update_game`、`test`、`rookie_finish`、`super_lucky_farm_*` 指令
- `init_game` 回應使用 `Scene` 而非 `scene`（首字母大寫）

### B.2.2 MonsterSystem (fish) - 怪物系統

**注意**：DinoPinBall 使用 `MonsterSystem` 而非 `FishSystem`，但系統名稱仍為 `"fish"`。

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `f0` | C→S | 怪物離場請求 | `1` (monster_id) |
| `f1` | S→C | 生成怪物 | `1`(id), `2`(type), `3`(x), `4`(y), `5`(o), `6`(feature), `8`(size), `17`(path), `23`(extra_data) |
| `f2` | S→C | 怪物事件 | `1`(id), 事件資料 |
| `f3` | S→C | 場景切換 | `14`(scene) |
| `f4` | S→C | 怪物腳本初始化 | `13`(current), `14`(scene), `11`(monster_group) |
| `f5` | C→S | 心跳函式（維持出怪） | - |
| `f6` | S→C | 怪物潮/特殊怪進場宣告 | `2`(type) |
| `f7` | S→C | 特殊怪物資料更新 | 怪物專屬資料（如黃金猛瑪象倍數） |
| `f8` | S→C | 史萊姆王計量條 | 進度資料 |

#### f1 怪物資料欄位對照

| 代碼 | 欄位名 | 說明 |
|------|--------|------|
| `1` | ID | 怪物的唯一 SID |
| `2` | Type | 怪物種類 |
| `3` | X | X 座標 |
| `4` | Y | Y 座標 |
| `5` | O | 朝向角度 |
| `6` | Feature | 特殊類型 |
| `8` | Size | 生成數量 |
| `17` | path | 路徑名稱 |
| `18` | teleport | 是否為召喚怪 |
| `19` | sub_type | 子類型 |
| `20` | min_bet | 最低押注限制 |
| `22` | Spacing | 間距 |
| `23` | extra_data | 恐龍特有資訊 |
| `28` | limit_vip | VIP 限制 |
| `29` | millisecond | 毫秒 |

**與海王 FishSystem 差異**：
- `f8` 用途不同：海王用於黃金拉霸蟹場景資料，恐龍用於史萊姆王計量條
- 新增 `22`(Spacing)、`29`(millisecond) 欄位

### B.2.3 WeaponSystem (weapon) - 武器系統

| 指令 | 方向 | 說明 | 主要參數 |
|------|------|------|----------|
| `w1` | C↔S | 一般射擊 | `1`(id), `2`(seat), `4`(x), `5`(y), `20`(bulletType) |
| `w2` | C↔S | 子彈命中（捕獲結果） | `1`(id), `2`(seat), `7`(monster_id_list), `8`(monster_dead_dict), `9`(bet), `10`(odds), `11`(coin) |
| `w3` | C↔S | 鎖定射擊 | `1`(id), `2`(seat), `6`(monster_id), `20`(bulletType) |
| `w4` | C↔S | 狂暴射擊 | `1`(id), `2`(seat), `6`(monster_id), `9`(bet) |
| `w5` | C↔S | 電擊射擊 | `1`(id), `2`(seat), `7`(monster_id_list) |
| `w6` | C↔S | 廣播 Auto 狀態 | `2`(seat), `25`(player_auto) |
| `w7` | C↔S | 雷鳴射擊 | `1`(id), `2`(seat), `7`(monster_id_list), `27`(bonus_monster_id_list) |
| `w8` | C↔S | 返還射擊（子彈回收） | `1`(id), `2`(seat) |
| `w9` | S→C | ExtraBet 設定初始化 | `31`(vip_limit), `32`(multiple), `33`(cd), `34`(vip_lv_lock), `35`(status) |
| `w10` | C↔S | 迴力鏢射擊 | `1`(id), `2`(seat), `6`(monster_id), `9`(bet) |
| `w11` | C↔S | 迴力鏢命中 | `1`(id), `2`(seat), `7`(monster_id_list), `20`(bulletType) |
| `w12` | C↔S | 迴力鏢開關 | `36`(extraBet_type) |
| `w24` | S→C | 能量武器數值（定時傳送） | `energy_info` |

#### w2 捕獲結果重要欄位

| 代碼 | 欄位名 | 說明 |
|------|--------|------|
| `3` | Credits | 玩家當前金幣 |
| `8` | MonsterDeadDict | 死亡怪物資料字典 |
| `10` | Odds | 倍率 |
| `11` | Coin | 贏得金幣 |
| `12` | Gem | 贏得寶石 |
| `13` | ItemDict | 掉落道具 |
| `14` | AddTicket | 增加票券 |
| `23` | Token_dict | Token 字典 |
| `24` | have_reward_item | 是否有獎勵道具 |
| `26` | player_id | 玩家 ID |
| `28` | bonus_fish_effect | 獎勵魚特效類型 |
| `29` | extra_data | 額外資料（含 award_type） |
| `39` | showAwardType | 報獎類型 |
| `42` | client_show | 演出資料（幽靈魚等） |
| `43` | drop_reward_item | 掉落物列表（新協定） |
| `44` | client_shoot_extra_data | 特殊子彈參數（阿力肯炮） |

**與海王 WeaponSystem 差異**：
- 新增 `w8` (返還射擊)、`w9` (ExtraBet 設定)、`w10-w12` (迴力鏢)、`w24` (能量武器)
- 無 `w17`、`w18`、`w31-w33` 指令
- 新增 `44` (client_shoot_extra_data) 欄位

### B.2.4 其他系統

DinoPinBall 的其他系統（PlayerSystem、SkillSystem、TableSystem、DP_JackpotSystem、DP_DailyRankSystem、SocialSystem、MissionSystem、EquipmentSystem）指令格式與海王類似，主要差異在於：

1. **系統存取方式**：使用 `GameClient.Instance.GetSystem<T>()` 泛型方法
2. **事件發送**：使用 `Dino_EventManager.SendEvent()` 而非 `FishHunter_EventManager`
3. **資料存取**：使用 `gameDataManager.GetGameData<T>()` 取得資料

---

# Part C: 指令對照表

## C.1 共用指令（兩者相同）

| 系統 | 指令 | 說明 |
|------|------|------|
| game | `join_table`, `leave_table`, `init_game`, `marquee`, `pause_game`, `hurray_sale`, `random_drop` | 遊戲流程 |
| fish | `f0`-`f7` | 魚/怪物基本操作 |
| weapon | `w1`-`w7` | 基本武器操作 |
| player | `init_player`, `update_player`, `update_bet`, `reflash` | 玩家操作 |

## C.2 海王專屬指令

| 系統 | 指令 | 說明 |
|------|------|------|
| game | `update_game`, `test`, `rookie_finish`, `super_lucky_farm_*` | 海王特有功能 |
| fish | `f8` (場景額外資料), `script_name` | 黃金拉霸蟹、腳本名稱 |
| weapon | `w17`, `w18`, `w31-w33` | 魚技能返還、雙重射擊 |
| skill | `sk_*` 系列 | 技能魚種專用指令 |

## C.3 恐龍專屬指令

| 系統 | 指令 | 說明 |
|------|------|------|
| fish | `f8` (史萊姆王計量條) | 史萊姆王進度 |
| weapon | `w8` (返還射擊) | 子彈回收 |
| weapon | `w9` (ExtraBet 設定) | ExtraBet 初始化 |
| weapon | `w10-w12` (迴力鏢) | 迴力鏢系統 |
| weapon | `w24` (能量武器) | 能量武器數值 |

---

# Part D: 資料流向圖

## D.1 進入遊戲流程（兩者相同）

```
Client                                  Server
   │                                      │
   │──── join_table (game) ──────────────>│
   │<─── join_table response ─────────────│
   │                                      │
   │──── init_game (game) ───────────────>│
   │<─── init_game response ──────────────│
   │                                      │
   │──── init_player (player) ───────────>│
   │<─── init_player response ────────────│
   │                                      │
   │──── update_player (player) ─────────>│
   │<─── update_player response ──────────│
   │                                      │
   │──── f4 (fish) ──────────────────────>│  (腳本請求)
   │<─── f4 response (場景同步) ──────────│
   │                                      │
   │<─── f1 (持續出魚/怪) ────────────────│
   │                                      │
```

## D.2 射擊與捕獲流程（兩者相同）

```
Client                                  Server
   │                                      │
   │──── w1/w3/w4/w5 (射擊) ─────────────>│
   │<─── w1/w3/w4/w5 response (扣錢) ─────│
   │                                      │
   │──── w2 (命中) ──────────────────────>│
   │<─── w2 response (捕獲結果) ──────────│
   │                                      │
```

## D.3 恐龍專屬：子彈返還流程

```
Client                                  Server
   │                                      │
   │──── w8 (返還射擊) ──────────────────>│
   │<─── w8 response (返還金額) ──────────│
   │                                      │
```

## D.4 恐龍專屬：能量武器流程

```
Client                                  Server
   │                                      │
   │<─── w24 (能量數值定時推送) ──────────│
   │                                      │
```

---

# Part E: 開發注意事項

## E.1 指令註冊規範

### 海王 (OceanTreasure)
1. 所有指令在 `Init()` 中透過 `Register(cmd, callback)` 註冊
2. 技能魚種指令在 `SkillXxxObject` 建構子中註冊
3. 使用 `FishHunter_EventManager` 發送事件

### 恐龍 (DinoPinBall)
1. 所有指令在 `Init()` 中透過 `Register(cmd, callback)` 註冊
2. 使用 `Dino_EventManager.SendEvent()` 發送事件
3. 資料透過 `gameDataManager.GetGameData<T>()` 存取

## E.2 資料欄位編碼

- 兩者的 FishSystem/MonsterSystem 都使用數字字串作為 key（如 `"1"`, `"2"`）
- 其他系統使用語意化 key（如 `"seat"`, `"coin"`）

## E.3 金流處理（兩者相同）

1. 射擊時先進行「預扣」（Client 端扣除 UI 顯示）
2. 收到 Server 回應後進行「預扣結算」
3. 捕獲時先進行「預加」（記錄待加金額）
4. 演出結束後進行「預加結算」（更新 UI）

## E.4 同步機制

- `f5` 心跳函式維持 Server 出魚/怪
- 進入遊戲時透過 `f4` 同步場上現有魚/怪

---

# Part F: 快速參考卡

```
┌─────────────────────────────────────────────────────────────────────────┐
│                 OceanTreasure (海王) 指令快速參考                         │
├─────────────────────────────────────────────────────────────────────────┤
│ 【遊戲流程】game                                                         │
│   join_table → init_game → init_player → update_player                 │
├─────────────────────────────────────────────────────────────────────────┤
│ 【魚種系統】fish                                                         │
│   f1(生成) f4(同步) f5(心跳) f7(特殊魚更新) f8(場景資料)                 │
├─────────────────────────────────────────────────────────────────────────┤
│ 【武器系統】weapon                                                       │
│   w1(射擊) w2(命中) w3(鎖定) w4(狂暴) w5(電擊) w7(雷鳴)                 │
│   w31-w33(雙重射擊)                                                     │
├─────────────────────────────────────────────────────────────────────────┤
│ 【技能系統】skill                                                        │
│   sk_icons(道具) sk_xxx(各技能魚專用指令)                                │
│   通用模組: sk_skill_start → sk_bomb_fish → sk_end                      │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                 DinoPinBall (恐龍) 指令快速參考                           │
├─────────────────────────────────────────────────────────────────────────┤
│ 【遊戲流程】game                                                         │
│   join_table → init_game → init_player → update_player                 │
├─────────────────────────────────────────────────────────────────────────┤
│ 【怪物系統】fish (MonsterSystem)                                         │
│   f1(生成) f4(同步) f5(心跳) f7(特殊怪更新) f8(史萊姆王計量條)           │
├─────────────────────────────────────────────────────────────────────────┤
│ 【武器系統】weapon                                                       │
│   w1(射擊) w2(命中) w3(鎖定) w4(狂暴) w5(電擊) w7(雷鳴)                 │
│   w8(返還) w9(ExtraBet) w10-w12(迴力鏢) w24(能量武器)                   │
└─────────────────────────────────────────────────────────────────────────┘
```
