# 單機 H5 魚機製作指南：借鑑 LMAO MOBA 的 AI Agent 開發法

## 1. 目標

本文件整理一套可複製的 AI 協作開發方法，用來製作一款「單機 H5 魚機」。

核心不是直接模仿 MOBA，而是學習其開發流程：

> 把 Claude / AI 當成一支小型開發團隊，而不是單純的程式助手。

本專案目標是製作一款可在瀏覽器執行的單機 H5 魚機 MVP，包含：

- Canvas 遊戲畫面
- 魚群生成與移動
- 玩家砲台
- 子彈射擊
- 命中判定
- 魚死亡與掉落分數
- 基礎倍率與金幣系統
- 簡單特效
- 可擴充的魚種、砲台、技能與活動架構

---

## 2. 不建議一開始做的事

第一版不要做：

- 多人連線
- 真實金流
- 完整後台
- 複雜活動系統
- 付費數值
- 商業級防作弊
- 大量美術資源
- 完整魚機產品化

第一版重點是：

> 先做出一個可以玩、可以驗證節奏、可以持續擴充的單機版本。

---

## 3. 推薦技術棧

### 前端

```text
Vite
React
TypeScript
Canvas 2D
```

### 狀態管理

MVP 可先不用 Redux / Zustand。

建議先用：

```text
GameState class
GameLoop class
Entity Manager
```

等遊戲變複雜後再拆。

### 美術

第一版可以使用：

```text
SVG 由 AI 生成
↓
轉成 Canvas 繪製邏輯
↓
用程式控制動畫
```

或更簡化：

```text
Canvas procedural shapes
```

也就是先用幾何圖形畫魚、砲台、子彈與特效，不依賴外部美術。

---

## 4. 核心方法：把 AI 當成開發團隊

不要只用一個 Prompt 叫 AI「幫我做魚機」。

應該拆成多個角色：

```text
主控 Agent / PM Agent
├── Game Designer Agent
├── Fish Designer Agent
├── Economy Designer Agent
├── Canvas Engineer Agent
├── Animation Agent
├── QA Agent
├── Refactor Agent
└── Optimization Agent
```

每個 Agent 只負責一件事。

---

## 5. AI Agent 分工

### 5.1 PM Agent

負責：

- 定義 MVP 範圍
- 拆任務
- 排優先級
- 控制不要過度開發
- 整合其他 Agent 的輸出

Prompt 範例：

```text
You are the PM of a single-player H5 fish shooting game.

Goal:
Build a playable MVP in React + TypeScript + Canvas.

Constraints:
- Browser only
- No backend
- No real money
- No external art dependency
- Must be playable with mouse click / tap

Output:
1. MVP feature list
2. Development milestones
3. File structure
4. Task breakdown for engineering agents
```

---

### 5.2 Game Designer Agent

負責：

- 遊戲節奏
- 玩法規則
- 魚種定位
- 砲台設計
- 關卡節奏

Prompt 範例：

```text
You are a game designer for a single-player H5 fish shooting game.

Design the core gameplay loop.

Requirements:
- Player controls a cannon at the bottom of the screen
- Fish swim across the screen
- Player shoots bullets to catch fish
- Fish have HP, score value, speed, and spawn weight
- Game should feel satisfying within 60 seconds

Output:
- Core loop
- Fish types
- Cannon behavior
- Reward rules
- Difficulty curve
```

---

### 5.3 Fish Designer Agent

負責：

- 魚種設計
- HP
- 分數
- 速度
- 出現權重
- 稀有度
- 行為模式

Prompt 範例：

```text
You are a fish enemy designer.

Create 8 fish types for a single-player fish shooting H5 game.

Each fish needs:
- id
- name
- rarity
- hp
- score
- speed
- spawnWeight
- movementPattern
- visualDescription

Return as JSON.
```

輸出格式範例：

```json
[
  {
    "id": "small_blue_fish",
    "name": "小藍魚",
    "rarity": "common",
    "hp": 1,
    "score": 2,
    "speed": 90,
    "spawnWeight": 40,
    "movementPattern": "straight",
    "visualDescription": "small blue oval fish with simple tail"
  }
]
```

---

### 5.4 Economy Designer Agent

負責：

- 子彈成本
- 魚分數
- 倍率
- 金幣回收率
- 遊戲節奏

單機版不碰真實金流，只做模擬經濟。

Prompt 範例：

```text
You are an economy designer for a casual fish shooting game.

Design a simple coin economy for a single-player prototype.

Constraints:
- No real money
- Player starts with 1000 coins
- Each shot costs coins
- Catching fish gives coins
- Game should last at least 3 minutes before the player runs out of coins

Output:
- Cannon levels
- Bullet cost
- Fish rewards
- Expected return range
- Tuning notes
```

---

### 5.5 Canvas Engineer Agent

負責：

- 遊戲主迴圈
- Canvas render
- Entity update
- 碰撞判定
- 輸入控制

Prompt 範例：

```text
You are a senior TypeScript Canvas game engineer.

Build the core engine for a single-player H5 fish shooting game.

Tech:
- React
- TypeScript
- Canvas 2D
- No external game engine

Need:
- Game loop
- Entity system
- Fish spawning
- Bullet shooting
- Collision detection
- Score and coin state

Output:
- File structure
- TypeScript interfaces
- Core implementation code
```

---

### 5.6 Animation Agent

負責：

- 魚游動
- 尾巴擺動
- 命中特效
- 死亡特效
- 金幣飛行
- 砲台旋轉

Prompt 範例：

```text
You are an animation designer for a Canvas 2D H5 game.

Design procedural animations for:
- Fish swimming
- Fish hit reaction
- Fish death
- Bullet trail
- Coin collection
- Cannon recoil

Constraints:
- Canvas 2D only
- No image assets
- Use simple math and shapes

Output:
- Animation rules
- Timing
- Easing suggestions
- TypeScript-friendly implementation notes
```

---

### 5.7 QA Agent

負責：

- 找 Bug
- 找體驗問題
- 找數值問題
- 找效能問題

Prompt 範例：

```text
You are a QA tester for a single-player H5 fish shooting game.

Review the current implementation concept.

Find issues in:
- Gameplay
- Controls
- Economy
- Collision
- Performance
- Edge cases

Output:
- Critical bugs
- Medium issues
- Minor polish suggestions
- Reproduction steps if applicable
```

---

### 5.8 Optimization Agent

負責：

- 減少 Canvas 繪製成本
- 控制 entity 數量
- Object pooling
- 碰撞優化
- 手機效能

Prompt 範例：

```text
You are a performance engineer for mobile H5 Canvas games.

Optimize a fish shooting game with many fish and bullets.

Focus on:
- Canvas draw calls
- Collision checks
- Object pooling
- Garbage collection
- Mobile browser performance

Output:
- Bottlenecks
- Optimization strategy
- Code-level recommendations
```

---

## 6. 建議檔案結構

```text
src/
├── App.tsx
├── main.tsx
├── game/
│   ├── Game.ts
│   ├── GameLoop.ts
│   ├── GameState.ts
│   ├── config.ts
│   ├── types.ts
│   ├── entities/
│   │   ├── Entity.ts
│   │   ├── Fish.ts
│   │   ├── Bullet.ts
│   │   ├── Cannon.ts
│   │   └── Effect.ts
│   ├── systems/
│   │   ├── SpawnSystem.ts
│   │   ├── CollisionSystem.ts
│   │   ├── RenderSystem.ts
│   │   ├── InputSystem.ts
│   │   └── EconomySystem.ts
│   ├── data/
│   │   ├── fishData.ts
│   │   ├── cannonData.ts
│   │   └── economyData.ts
│   └── utils/
│       ├── math.ts
│       ├── random.ts
│       └── collision.ts
└── styles.css
```

---

## 7. MVP 功能清單

### 必做

- Canvas 畫面
- 砲台固定在畫面底部
- 滑鼠點擊 / 手機點擊射擊
- 子彈朝點擊方向移動
- 魚從左右兩側生成
- 魚移動穿越畫面
- 子彈碰到魚扣血
- 魚死亡給分
- 每次射擊扣金幣
- 畫面顯示金幣、分數、砲台倍率

### 第二階段

- 多種魚
- 魚群路徑
- Boss 魚
- 技能砲
- 冰凍 / 鎖定 / 炸彈
- 金幣飛行特效
- 魚死亡動畫
- 手機版 UI

### 第三階段

- 關卡模式
- 任務系統
- 活動玩法
- 圖鑑
- 離線收益模擬
- 數值調參工具

---

## 8. 開發順序

### Milestone 1：空 Canvas 與 Game Loop

目標：

```text
畫面能穩定刷新
可以顯示 FPS
可以 resize
```

---

### Milestone 2：砲台與射擊

目標：

```text
玩家點擊畫面
砲台轉向
產生子彈
子彈往目標方向飛行
```

---

### Milestone 3：魚生成與移動

目標：

```text
魚從左右兩側出現
魚會移動
離開畫面後自動移除
```

---

### Milestone 4：碰撞與死亡

目標：

```text
子彈碰到魚
魚扣血
HP 歸零死亡
玩家獲得分數
```

---

### Milestone 5：經濟系統

目標：

```text
射擊扣金幣
捕獲魚加金幣
不同魚有不同獎勵
```

---

### Milestone 6：體驗優化

目標：

```text
命中特效
死亡特效
砲台後座力
魚游動動畫
金幣飛行
```

---

## 9. 單機魚機的核心資料結構

### FishConfig

```ts
export interface FishConfig {
  id: string;
  name: string;
  hp: number;
  reward: number;
  speed: number;
  radius: number;
  spawnWeight: number;
  color: string;
  movementPattern: 'straight' | 'wave' | 'curve';
}
```

### CannonConfig

```ts
export interface CannonConfig {
  level: number;
  bulletCost: number;
  damage: number;
  fireRate: number;
  bulletSpeed: number;
}
```

### GameState

```ts
export interface GameState {
  coins: number;
  score: number;
  cannonLevel: number;
  fishCaught: number;
  shotsFired: number;
  elapsedTime: number;
}
```

---

## 10. 基礎數值範例

### 魚種

```ts
export const fishData: FishConfig[] = [
  {
    id: 'small_fish',
    name: '小魚',
    hp: 1,
    reward: 2,
    speed: 90,
    radius: 16,
    spawnWeight: 45,
    color: '#4aa3ff',
    movementPattern: 'straight',
  },
  {
    id: 'medium_fish',
    name: '中魚',
    hp: 3,
    reward: 8,
    speed: 70,
    radius: 24,
    spawnWeight: 30,
    color: '#44cc88',
    movementPattern: 'wave',
  },
  {
    id: 'big_fish',
    name: '大魚',
    hp: 8,
    reward: 25,
    speed: 45,
    radius: 36,
    spawnWeight: 15,
    color: '#ffbb33',
    movementPattern: 'curve',
  },
  {
    id: 'boss_fish',
    name: 'Boss 魚',
    hp: 30,
    reward: 120,
    speed: 28,
    radius: 56,
    spawnWeight: 3,
    color: '#ff5555',
    movementPattern: 'wave',
  },
];
```

### 砲台

```ts
export const cannonData: CannonConfig[] = [
  {
    level: 1,
    bulletCost: 1,
    damage: 1,
    fireRate: 250,
    bulletSpeed: 520,
  },
  {
    level: 2,
    bulletCost: 2,
    damage: 2,
    fireRate: 230,
    bulletSpeed: 560,
  },
  {
    level: 3,
    bulletCost: 5,
    damage: 5,
    fireRate: 210,
    bulletSpeed: 600,
  },
];
```

---

## 11. AI 開發工作流

每一輪開發用這個節奏：

```text
1. PM Agent 定義目標
2. Engineer Agent 產生程式
3. QA Agent 找問題
4. Refactor Agent 整理架構
5. Optimization Agent 檢查效能
6. PM Agent 決定下一輪
```

不要一次叫 AI 做完整遊戲。

應該分段下指令：

```text
第一輪：只做 Canvas + Game Loop
第二輪：加入砲台與射擊
第三輪：加入魚
第四輪：加入碰撞
第五輪：加入金幣系統
第六輪：加入特效
```

---

## 12. 可直接使用的總 Prompt

```text
I want to build a single-player H5 fish shooting game.

Tech stack:
- Vite
- React
- TypeScript
- Canvas 2D
- No backend
- No external game engine
- No external art assets for MVP

Game requirements:
- Cannon at bottom center
- Player clicks or taps to shoot
- Bullets fly toward click position
- Fish spawn from left and right
- Fish have HP, reward, speed, radius, and movement pattern
- Bullet collision damages fish
- Dead fish grant coins and score
- Every shot costs coins
- Show coins, score, cannon level, and FPS

Architecture requirements:
- Clean file structure
- Entity-based design
- Separate systems for rendering, spawning, collision, input, and economy
- Easy to add new fish types
- Easy to add new cannon levels
- Mobile browser friendly

Please build Milestone 1 first only:
- React component with Canvas
- Game loop
- Resize handling
- FPS display
- Clean TypeScript structure

Do not implement fish or bullets yet.
```

---

## 13. 下一輪 Prompt 範例

### 加入砲台與射擊

```text
Now implement Milestone 2.

Add:
- Cannon at bottom center
- Cannon rotates toward mouse / touch position
- Click or tap fires a bullet
- Bullet moves toward target direction
- Bullets are removed when off screen

Keep architecture clean.
Do not add fish yet.
```

### 加入魚

```text
Now implement Milestone 3.

Add:
- Fish entity
- Fish config data
- Spawn system
- Fish spawn from left and right edges
- Fish move across the screen
- Remove fish when off screen

Do not add collision yet.
```

### 加入碰撞

```text
Now implement Milestone 4.

Add:
- Bullet-fish collision
- Fish HP
- Bullet damage
- Fish death
- Score increment
- Simple hit effect

Keep the collision system separate.
```

### 加入經濟

```text
Now implement Milestone 5.

Add:
- Player coins
- Shooting costs coins
- Fish death rewards coins
- Cannon levels
- UI for coins, score, cannon level
- Prevent shooting if coins are insufficient
```

---

## 14. 製作時的注意事項

### 14.1 不要太早追求美術

第一版用簡單圖形即可。

魚可以先畫成：

```text
橢圓身體 + 三角尾巴 + 小眼睛
```

等玩法成立後，再讓 AI 生成 SVG 或圖片風格。

---

### 14.2 不要太早做複雜數值

第一版只需要確認：

```text
射擊是否爽
魚是否好打
金幣是否有波動
玩家是否想繼續玩
```

---

### 14.3 Canvas 效能原則

- 每幀只畫必要物件
- 離開畫面的魚與子彈要移除
- 不要每幀建立大量新物件
- 特效生命週期要短
- 手機上 entity 數量要有限制

建議上限：

```text
魚：30 ~ 50 隻
子彈：50 ~ 100 顆
特效：30 個以內
```

---

## 15. 單機 H5 魚機 MVP 驗收標準

完成後應該能做到：

- 打開網頁即可玩
- 手機與桌機都可操作
- 玩家能射擊
- 魚會出現並移動
- 子彈能命中魚
- 魚會死亡
- 金幣會增減
- 分數會增加
- 遊戲至少能連續玩 3 分鐘不壞掉
- 程式架構能繼續擴充

---

## 16. 後續可擴充方向

### 玩法

- 鎖定魚
- 自動射擊
- 冰凍技能
- 炸彈技能
- 狂暴模式
- Boss 波次
- Jackpot 假模擬

### 活動

- 每日任務
- 捕魚圖鑑
- 限時 Boss
- 累積射擊獎勵
- 連續捕獲獎勵

### 產品驗證

- 測試魚種節奏
- 測試砲台倍率
- 測試回收率感受
- 測試玩家停留時間
- 測試不同魚群密度

---

## 17. 結論

LMAO MOBA 案例最值得借鑑的不是「一天做出大型遊戲」，而是：

```text
把 AI 拆成多個專業角色
讓每個角色負責清楚的小任務
用短週期反覆開發、測試、修正
```

用在單機 H5 魚機上，最適合的策略是：

```text
先做可玩的 Canvas MVP
再補數值
再補美術
再補特效
最後才補活動與產品化系統
```

第一版的成功標準不是完整，而是：

> 可以玩、可以改、可以驗證。
