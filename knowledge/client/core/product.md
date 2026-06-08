# 產品概述

FishHunter (捕魚機) 是一款基於 Unity 的街機風格捕魚遊戲，包含兩種獨立的遊戲引擎：

## 兩大遊戲引擎

### OceanTreasure — 2D 魚機（海王）
- 代號：ArkGame
- 類型：2D 經典捕魚
- 程式碼位置：`Assets/FishHunter_Script/ArkGame/`
- 資源位置：`Assets/FishtHunter_Res/ArkGame/`
- Namespace：`ArkGame`、`OceanTreasure`
- 系統入口：`ArkGame.GameClient.Instance`
- 架構特性：強耦合、具名 getter、Action/Delegate 事件
- 魚種基類：`Fish2D`，透過 `FishMaintainer` 管理 Object Pool
- 技能系統：SkillSystem 三層架構（SkillXxxObject → SkillModel）

### DinoPinBall — 3D 魚機（恐龍 / Juras）
- 代號：DinoGame
- 類型：3D 恐龍主題彈珠台
- 程式碼位置：`Assets/FishHunter_Script/DinoGame/DinoPinBall/`
- 資源位置：`Assets/FishtHunter_Res/CrazyDino/`
- Namespace：`DinoPinBall`
- 系統入口：`DinoPinBall.GameClient.Instance`
- 架構特性：鬆耦合、泛型 `GetSystem<T>()`、依賴注入、`Dino_EventManager` 事件驅動
- 怪物基類：透過 `MonsterSystem` + `MonsterManager` 管理
- 技能系統：SkillSystem（與 ArkGame 獨立）

## 其他遊戲模式
- FishGame：經典捕魚變體（共用 ArkGame 架構）
- ThreePig：Token 代幣系統模式

## 跨模組橋接
- `FishHunter_Agent.cs`：橋接 ArkGame ↔ OceanTreasure ↔ DinoPinBall
- `FishHunter_AgentEventManager`：跨模組事件路由（`AG_` / `OT_` / `DP_` 前綴）

## 共用功能
- 多人即時捕魚玩法，多種武器系統與特殊技能
- 任務系統、錦標賽、社交系統、Jackpot、公會功能
- 內購系統 (IAP)、道具、Buff、武器卡牌
- MasterAudio 音效管理 (BGM + 音效)
- Spine 骨骼動畫整合
- 本家/他家演出分離邏輯
- FH_Common 共用程式碼（FH_SpineActionSystem、FH_TimelineActionSystem 等）
