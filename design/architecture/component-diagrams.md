---
title: 《暗室》组件图 (Component Diagrams · C4 Model)
doc_id: DESIGN-anzhong-architecture-04
parent: design/architecture/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 尚书省
---

# 《暗室》组件图 (Component Diagrams · C4 Model)

> **一句话定位：** 3 层 C4 模型（Context L1 / Container L2 / Component L3）的 Mermaid 图，覆盖玩家、系统边界、客户端/服务端容器、14 模块组件视图。

## 目的 (Purpose)

本文档是《暗室》**架构可视化 (Architecture Visualization)** 的**唯一权威基线**。它向：

- **架构师** — 提供标准化的 C4 模型图（Context / Container / Component）做架构治理
- **新加入工程师** — 5 分钟看懂系统边界、容器划分、模块依赖
- **技术评审委员会** — 用于评审架构决策（ADR）的合理性
- **跨团队沟通** — 与 美术 / 策划 / QA / 运维 共享同一套架构图
- **文档化资产** — Wiki / Confluence / GitHub README 直接嵌入 Mermaid 图

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的系统边界、容器划分、模块视图——**第一次**用 C4 模型（Simon Brown 提出的 4 层架构可视化方法）的 L1 / L2 / L3 三层 Mermaid 图统一描述，作为架构治理、ADR 评审、跨团队沟通的"图谱基线"。

## 范围 (Scope)

### 包含

- **C4 Level 1（System Context）** — 系统与外部用户/系统的关系
- **C4 Level 2（Container）** — 客户端 / 服务端 / 数据库 / CDN 的容器视图
- **C4 Level 3（Component）** — 14 模块的组件视图 + 依赖关系
- **扩展图** — 玩家操作时序图 + 房间通关时序图 + 存档写入时序图
- **Mermaid 图 ≥ 6 张**（L1 × 1 + L2 × 2 + L3 × 2 + Sequence × 3）

### 不包含

- C4 Level 4（Code）— 类级别的 UML 图，实施时由 IDE 自动生成
- 部署图 → 见 `system-overview.md` §4
- 数据流图 → 见 `data-flow.md`

## 一句话描述 (One-liner)

> **"L1 系统边界 + L2 容器视图 + L3 模块视图，6+ Mermaid 图覆盖《暗室》架构全景。"**

## 1. C4 模型简介 (C4 Model Introduction)

> C4 模型由 Simon Brown 提出，用 4 层（Context / Container / Component / Code）从粗到细描述软件架构。本文档覆盖前三层。

| 层级 | 名称 | 视角 | 目标读者 |
|:----:|------|------|---------|
| **L1** | System Context | 系统整体 + 外部用户/系统 | 非技术利益相关者 |
| **L2** | Container | 进程/容器边界 | 技术团队 + DevOps |
| **L3** | Component | 模块/子系统视图 | 软件架构师 + 工程师 |
| **L4** | Code | 类级别 UML | 实施工程师（IDE 自动生成） |

## 2. C4 Level 1 — System Context（系统上下文）

> L1 视角：玩家如何与《暗室》系统交互，以及系统与哪些外部系统通信。

```mermaid
flowchart TB
    Player([玩家<br/>25-40 岁<br/>解谜爱好者])
    
    subgraph SystemBoundary["《暗室》系统边界"]
        AnzhongGame["《暗室》游戏<br/>无战斗 2D 房间解谜<br/>3 章节 19 房间<br/>v1.0 客户端 + v2.0+ 服务端"]
    end
    
    Steam["Steam<br/>PC/Mac 商店 + 成就 + 云存档"]
    ItchIO["Itch.io<br/>试玩版 1-1~1-5"]
    AppleAppStore["Apple App Store<br/>iOS v2.0+"]
    GooglePlay["Google Play<br/>Android v2.0+"]
    SonyPSN["PlayStation Network<br/>PS5 v2.0+"]
    XboxLive["Xbox Live<br/>Xbox Series v2.0+"]
    NintendoeShop["Nintendo eShop<br/>Switch v1.1+"]
    AnzhongCDN["CloudFront CDN<br/>静态资源 + DLC<br/>v2.0+"]
    AnalyticsDB[("PostgreSQL 16<br/>遥测 + 排行榜<br/>v2.0+")]
    
    Player -->|"购买 + 下载 + 玩"| AnzhongGame
    Player -->|"购买 + 下载 + 玩"| Steam
    Player -->|"试玩"| ItchIO
    Player -->|"购买 + 下载 + 玩 v2.0+"| AppleAppStore
    Player -->|"购买 + 下载 + 玩 v2.0+"| GooglePlay
    Player -->|"购买 + 下载 + 玩 v2.0+"| SonyPSN
    Player -->|"购买 + 下载 + 玩 v1.1+"| NintendoeShop
    
    AnzhongGame -.->|"下载静态资源"| AnzhongCDN
    AnzhongGame -.->|"上报遥测 v2.0+"| AnalyticsDB
    AnzhongGame -.->|"同步云存档 v1.0+"| Steam
```

**L1 关键实体：**

- **玩家** — 25-40 岁解谜爱好者，是系统的唯一用户
- **《暗室》系统** — 单一系统边界，覆盖客户端 + 服务端 + 数据库 + CDN
- **7 平台商店** — Steam / Itch.io / Apple App Store / Google Play / PSN / Xbox Live / Nintendo eShop
- **CloudFront CDN** — v2.0+ 静态资源分发
- **PostgreSQL 16** — v2.0+ 遥测 + 排行榜数据存储

**L1 设计决策：**

- ✅ **v1.0 无后端** — 系统边界仅含客户端进程
- ✅ **v2.0+ 渐进式后端化** — 服务端 + 数据库 + CDN 引入不破坏 v1.0 离线可玩
- ✅ **平台商店作为外部系统** — 不属于本系统边界，通过 SDK 集成

## 3. C4 Level 2 — Container（容器视图）

> L2 视角：系统由哪些容器（进程/服务）组成，以及它们之间如何通信。

### 3.1 v1.0 容器视图（单机客户端）

```mermaid
flowchart TB
    subgraph ClientContainer["Unity Client Container (v1.0)"]
        UnityGame["Unity Game Process<br/>Anzhong.exe / .app<br/>Unity 2022 LTS + IL2CPP<br/>14 模块 + 事件总线"]
        LocalSave[("Local Save<br/>savegame.json<br/>AES-256 加密<br/>本地磁盘")]
        LocalSaveBak[("Backup Save<br/>savegame.json.bak<br/>上一版本")]
    end
    
    subgraph ExternalSystems["外部系统 (External)"]
        SteamCloud["Steam Cloud<br/>云存档 v1.0 可选"]
        SteamSDK["Steamworks SDK<br/>成就 + DLC"]
    end
    
    subgraph AssetPipelines["资源管线"]
        Addressables["Addressables<br/>资源分组加载"]
        StreamingAssets["StreamingAssets<br/>本地资源"]
    end
    
    Player([玩家]) -->|"启动 + 输入"| UnityGame
    UnityGame <-->|"读写 ≤ 50ms"| LocalSave
    UnityGame -.->|"备份 fallback"| LocalSaveBak
    UnityGame -.->|"云同步 (可选)"| SteamCloud
    UnityGame -->|"成就/DLC 接口"| SteamSDK
    UnityGame -->|"资源加载"| Addressables
    Addressables -.->|"fallback"| StreamingAssets
```

**v1.0 容器清单：**

| 容器 | 进程类型 | 持久化 | 网络 |
|------|---------|--------|------|
| Unity Game Process | 客户端进程 | savegame.json (本地) | ⚠️ Steam Cloud (可选) |
| Local Save | 本地文件 | savegame.json | ❌ |
| Backup Save | 本地文件 | savegame.json.bak | ❌ |
| Steam Cloud | 外部云服务 | Steam 用户存档 | ✅ |
| Steamworks SDK | 外部库 | — | ✅ |
| Addressables | 客户端模块 | — | ❌ |

### 3.2 v2.0+ 容器视图（客户端 + 服务端 + 数据库 + CDN）

```mermaid
flowchart TB
    subgraph ClientContainers["客户端容器 (Unity)"]
        UnityClient["Unity Client Container<br/>Unity 2022 LTS + IL2CPP<br/>14 模块 + AnzhongApiClient"]
        LocalCache["Local Cache<br/>v2.0+ 离线模式缓存"]
    end
    
    subgraph EdgeLayer["边缘层"]
        APIGateway["API Gateway Container<br/>HTTPS 终止 + JWT 鉴权<br/>限流 60/1000/10"]
        CloudFrontCDN["CloudFront CDN Container<br/>静态资源 + DLC + 补丁"]
    end
    
    subgraph ServerContainers["服务端容器 (ASP.NET Core 8)"]
        SessionSvc["Session Service<br/>:5001<br/>E01-E02"]
        RoomSvc["Room Service<br/>:5002<br/>E03-E05"]
        SlotSvc["Slot Service<br/>:5003<br/>E06"]
        SaveSvc["Save Service<br/>:5004<br/>E08-E09 / E15"]
        ProgressSvc["Progress Service<br/>:5005<br/>E11 / E16"]
        LeaderboardSvc["Leaderboard Service<br/>:5006<br/>E10"]
        FeedbackSvc["Feedback Service<br/>:5007<br/>E12"]
        SettingsSvc["Settings Service<br/>:5008<br/>E13-E14"]
        HintSvc["Hint Service<br/>:5009<br/>E17"]
        ChapterSvc["Chapter Service<br/>:5010<br/>E18"]
    end
    
    subgraph DataContainers["数据层容器"]
        PostgreSQLPrimary[("PostgreSQL 16 Primary<br/>业务数据")]
        PostgreSQLReplica[("PostgreSQL 16 Replica x2<br/>只读副本")]
        RedisCache["Redis 7 Sentinel<br/>会话 + 排行榜"]
    end
    
    Player([玩家]) -->|"REST API"| UnityClient
    UnityClient -->|"HTTPS REST/JSON"| APIGateway
    UnityClient -.->|"Addressables"| CloudFrontCDN
    UnityClient <-->|"本地缓存"| LocalCache
    
    APIGateway --> SessionSvc
    APIGateway --> RoomSvc
    APIGateway --> SlotSvc
    APIGateway --> SaveSvc
    APIGateway --> ProgressSvc
    APIGateway --> LeaderboardSvc
    APIGateway --> FeedbackSvc
    APIGateway --> SettingsSvc
    APIGateway --> HintSvc
    APIGateway --> ChapterSvc
    
    SessionSvc --> PostgreSQLPrimary
    RoomSvc --> PostgreSQLPrimary
    SlotSvc --> PostgreSQLPrimary
    SaveSvc --> PostgreSQLPrimary
    ProgressSvc --> PostgreSQLPrimary
    LeaderboardSvc --> PostgreSQLPrimary
    FeedbackSvc --> PostgreSQLPrimary
    SettingsSvc --> PostgreSQLPrimary
    HintSvc --> PostgreSQLPrimary
    ChapterSvc --> PostgreSQLPrimary
    
    SessionSvc -.->|"缓存"| RedisCache
    RoomSvc -.->|"缓存"| RedisCache
    SaveSvc -.->|"会话锁"| RedisCache
    LeaderboardSvc -.->|"排行榜"| RedisCache
    
    PostgreSQLPrimary -->|"主从同步"| PostgreSQLReplica
```

**v2.0+ 容器清单：**

| 容器 | 技术栈 | 数量 | 职责 |
|------|-------|:----:|------|
| Unity Client | Unity 2022 LTS | 1+ (玩家设备) | 客户端游戏运行时 |
| API Gateway | NGINX / AWS ALB | 2+ (HA) | HTTPS + 鉴权 + 限流 |
| Session Service | ASP.NET Core 8 | 2+ (HA) | E01-E02 会话管理 |
| Room Service | ASP.NET Core 8 | 2+ (HA) | E03-E05 房间流转 |
| Slot Service | ASP.NET Core 8 | 2+ (HA) | E06 槽位切换事件 |
| Save Service | ASP.NET Core 8 | 2+ (HA) | E08-E09 / E15 存档 |
| Progress Service | ASP.NET Core 8 | 2+ (HA) | E11 / E16 进度上报 |
| Leaderboard Service | ASP.NET Core 8 | 2+ (HA) | E10 排行榜 |
| Feedback Service | ASP.NET Core 8 | 1 (低流量) | E12 玩家反馈 |
| Settings Service | ASP.NET Core 8 | 1 (低流量) | E13-E14 设置 |
| Hint Service | ASP.NET Core 8 | 1 (低流量) | E17 Hint 触发 |
| Chapter Service | ASP.NET Core 8 | 1 (低流量) | E18 章节列表 |
| PostgreSQL Primary | PostgreSQL 16 | 1 | 主库（写入） |
| PostgreSQL Replica | PostgreSQL 16 | 2 | 只读副本 |
| Redis Sentinel | Redis 7 | 3 (Sentinel) | 会话缓存 + 排行榜 |
| CloudFront CDN | AWS CloudFront | 全球边缘 | 静态资源分发 |
| Local Cache | SQLite / 文件 | 1 (玩家设备) | 离线模式缓存 |

**L2 设计决策：**

- ✅ **微服务架构** — 10 个微服务，按端点分组（与 design/api/ 端点对应）
- ✅ **PostgreSQL 主从** — 1 Primary + 2 Replica，Multi-AZ
- ✅ **Redis Sentinel 3 节点** — 会话缓存（TTL 24h）+ 排行榜（Sorted Set）
- ✅ **API Gateway** — 集中限流 + 鉴权 + HTTPS 终止
- ✅ **离线优先** — Local Cache 支持弱网/断网降级

## 4. C4 Level 3 — Component（组件视图）

> L3 视角：客户端 14 模块的组件视图 + 依赖关系。

### 4.1 客户端组件视图（v1.0 + v2.0 共用）

```mermaid
flowchart TB
    subgraph InputLayer["输入层"]
        M08["M08 Input<br/>键盘 + 手柄<br/>300/500ms 冷却"]
    end
    
    subgraph CoreLayer["核心层"]
        M01["M01 Core<br/>12 态状态机<br/>事件总线<br/>服务注册中心"]
    end
    
    subgraph LogicLayer["逻辑层"]
        M02["M02 SwitchSlot<br/>4 槽位类型<br/>5 状态机"]
        M03["M03 Room<br/>19 房间管理<br/>通关判定"]
        M04["M04 Player<br/>移动 + 碰撞<br/>trigger 检测"]
        M09["M09 HintSystem<br/>渐进式 Hint<br/>卡点识别"]
    end
    
    subgraph PresentationLayer["表现层"]
        M05["M05 UI<br/>HUD + 4 菜单<br/>4 组件状态"]
        M06["M06 Audio<br/>9 类音频<br/>动态混音"]
        M11["M11 Localization<br/>v1.0 中英<br/>v1.1 5 语种"]
    end
    
    subgraph PersistenceLayer["持久层"]
        M07["M07 SaveSystem<br/>JSON + 备份<br/>GDPR API"]
        M10["M10 Telemetry<br/>4 指标聚合<br/>v2.0+ 上报"]
    end
    
    subgraph FoundationLayer["基础层"]
        M12["M12 Settings<br/>音频 + 无障碍<br/>难度选项"]
        M13["M13 AssetPipeline<br/>Addressables<br/>Tilemap"]
    end
    
    subgraph APILayer["API 层 (v2.0+)"]
        ApiClient["AnzhongApiClient<br/>REST SDK<br/>JWT 鉴权"]
    end
    
    M08 -->|"原始输入"| M01
    M01 -->|"状态变更事件"| M02
    M01 -->|"房间状态"| M03
    M01 -->|"玩家事件"| M04
    M01 -->|"UI 状态"| M05
    M01 -->|"音频事件"| M06
    M01 -->|"存档事件"| M07
    M01 -->|"Hint 触发"| M09
    M01 -->|"遥测事件"| M10
    M05 -->|"设置持久化"| M12
    M05 -->|"本地化字符串"| M11
    M06 -->|"音量读取"| M12
    M02 -->|"槽位实例化"| M03
    M04 -->|"位置上报"| M03
    M03 -->|"通关存档"| M07
    M09 -->|"Hint 显示"| M05
    M10 -->|"指标持久化"| M07
    M13 -->|"预制件加载"| M02
    M13 -->|"Tilemap"| M03
    M13 -->|"UI 资源"| M05
    M13 -->|"音频加载"| M06
    ApiClient -.->|"REST 调用 v2.0+"| M01
```

**L3 组件依赖统计：**

| 组件 | 上游依赖 | 下游被依赖 | 层级 |
|------|---------|----------|------|
| M01 Core | 0 | 11 | Core |
| M02 SwitchSlot | 1 (M01) | 2 (M03/M09) | Logic |
| M03 Room | 3 (M01/M02/M04) | 5 (M07/M09/M10/M13) | Logic |
| M04 Player | 2 (M01/M08) | 1 (M03) | Logic |
| M05 UI | 4 (M01/M09/M11/M12) | 0 | Presentation |
| M06 Audio | 2 (M01/M12) | 0 | Presentation |
| M07 SaveSystem | 4 (M01/M03/M10/M12) | 0 | Persistence |
| M08 Input | 0 | 2 (M01/M04) | Input |
| M09 HintSystem | 4 (M01/M02/M03) | 1 (M05) | Logic |
| M10 Telemetry | 3 (M01/M03/M07) | 0 | Persistence |
| M11 Localization | 0 | 2 (M05/M06) | Presentation |
| M12 Settings | 0 | 4 (M05/M06/M07) | Foundation |
| M13 AssetPipeline | 0 | 4 (M02/M03/M05/M06) | Foundation |
| ApiClient | 0 | 1 (M01) | API |

### 4.2 服务端组件视图（v2.0+）

```mermaid
flowchart LR
    subgraph Gateway["API Gateway"]
        Auth["JWT Auth Middleware"]
        RateLimit["Rate Limiter<br/>60/1000/10"]
        HTTPS["HTTPS Terminator"]
    end
    
    subgraph Services["10 Microservices"]
        SessionSvc["Session Service"]
        RoomSvc["Room Service"]
        SlotSvc["Slot Service"]
        SaveSvc["Save Service"]
        ProgressSvc["Progress Service"]
        LeaderboardSvc["Leaderboard Service"]
        FeedbackSvc["Feedback Service"]
        SettingsSvc["Settings Service"]
        HintSvc["Hint Service"]
        ChapterSvc["Chapter Service"]
    end
    
    subgraph Persistence["持久化"]
        Repos["Repository Layer<br/>EF Core + Dapper"]
        Migrations["DB Migrations<br/>Flyway"]
        CacheLayer["Cache Layer<br/>StackExchange.Redis"]
    end
    
    HTTPS --> Auth --> RateLimit --> Services
    Services --> Repos
    Services --> CacheLayer
    Repos --> Migrations
```

**服务端组件清单：**

| 组件 | 技术 | 数量 | 职责 |
|------|------|:----:|------|
| JWT Auth Middleware | ASP.NET Core Middleware | 1 | Token 验证 |
| Rate Limiter | AspNetCoreRateLimit | 1 | 限流 60/1000/10 |
| HTTPS Terminator | Kestrel + NGINX | 2 | TLS 1.3 |
| 10 Microservices | ASP.NET Core 8 Minimal API | 2+/服务 | 业务逻辑 |
| Repository Layer | EF Core 8 + Dapper | 10 | 数据访问 |
| DB Migrations | Flyway 9 | 1 | schema 演进 |
| Cache Layer | StackExchange.Redis | 1 | Redis 客户端 |

## 5. 序列图 (Sequence Diagrams)

> 关键场景的时序图，展示组件间调用顺序。

### 5.1 玩家按 E 切换槽位时序图

```mermaid
sequenceDiagram
    actor Player as 玩家
    participant Input as M08 Input
    participant Core as M01 Core
    participant SwitchSlot as M02 SwitchSlot
    participant Room as M03 Room
    participant UI as M05 UI
    participant Audio as M06 Audio
    
    Player->>Input: 按 E 键
    Input->>Input: 检查 300ms 冷却
    alt 冷却未过
        Input-->>Player: 忽略输入
    else 冷却已过
        Input->>Core: 发布 OnSwitchForward 事件
        Core->>SwitchSlot: 调用 TrySwitchForward()
        SwitchSlot->>SwitchSlot: 检查状态 (Hover/Locked)
        alt Locked
            SwitchSlot-->>Core: 拒绝切换
            Core->>UI: 显示"槽位未解锁"提示
        else Hover
            SwitchSlot->>SwitchSlot: currentIndex++ (循环)
            SwitchSlot->>SwitchSlot: 启动 200ms 切换动画
            SwitchSlot->>Core: 发布 OnSlotSwitch 事件
            Core->>Room: 通知拓扑变化
            Room->>Room: 更新 Tilemap + Collider
            Core->>Audio: 播放切换音 (-12dB)
            Core->>UI: 更新槽位提示 (选项 2/3)
            Core->>Core: 检查路径连通性
            alt 出口连通
                Core->>UI: 显示"走到出口"提示
            end
        end
    end
```

### 5.2 房间通关时序图

```mermaid
sequenceDiagram
    actor Player as 玩家
    participant PlayerCtrl as M04 Player
    participant Room as M03 Room
    participant Core as M01 Core
    participant Save as M07 SaveSystem
    participant UI as M05 UI
    participant Audio as M06 Audio
    
    Player->>PlayerCtrl: WASD 移动到出口
    PlayerCtrl->>Room: 通知玩家位置更新
    Room->>Room: WinCondition.Check(pos)
    alt 路径不连通
        Room->>UI: 显示"被实墙阻挡"提示
        UI-->>Player: 玩家无法穿越
    else 路径连通
        Room->>Core: 发布 OnRoomComplete 事件
        Core->>UI: 显示通关画面 + 出口脉冲
        Core->>Audio: 播放通关音 (-6dB)
        Core->>Save: 写入存档 (chapterId + roomId)
        Save->>Save: 序列化 SaveData + AES-256 加密
        Save->>Save: 写入 savegame.json (≤ 50ms)
        alt 写入失败
            Save->>Save: 加载 backup
            Save-->>Core: 写入失败提示
        end
        Core->>Core: 状态机 Win → ChapterTransition
        Core->>UI: 屏幕渐白 0.5s → 黑屏 2s
        alt 非末房间
            Core->>Core: 加载下一房间 (≤ 1s)
            Core->>Core: 状态机 → RoomEntry
        else 末房间
            Core->>Core: 状态机 → ChapterComplete / GameComplete
        end
    end
```

### 5.3 存档写入时序图（v1.0 本地）

```mermaid
sequenceDiagram
    participant Game as M01 Core
    participant Save as M07 SaveSystem
    participant File as 本地磁盘
    participant Backup as BackupManager
    participant Telemetry as M10 Telemetry
    
    Game->>Save: 触发自动存档 (OnRoomComplete)
    Save->>Save: 构造 SaveData 对象
    Save->>Save: AES-256 加密 (≤ 30ms)
    Save->>File: 写入 savegame.json (≤ 20ms)
    alt 写入失败
        Save->>Backup: 加载 savegame.json.bak
        alt backup 有效
            Save->>Save: 降级为 backup 版本
            Save->>Game: 写入失败提示
        else backup 也失败
            Save->>Game: 无存档模式（提示玩家）
            Save->>Telemetry: 记录 save_failure 事件
        end
    else 写入成功
        Save->>File: 备份 savegame.json.bak (旧版)
        Save->>Game: 写入成功
        Save->>Telemetry: 记录 save_success 事件
    end
```

### 5.4 v2.0+ 云存档同步时序图

```mermaid
sequenceDiagram
    actor Player as 玩家
    participant Client as Unity Client
    participant APIGW as API Gateway
    participant SaveSvc as Save Service
    participant Cache as Redis Cache
    participant DB as PostgreSQL Primary
    participant Replica as PostgreSQL Replica
    
    Player->>Client: 触发云同步 (按 ESC → 同步)
    Client->>Client: 检查本地 SaveData + 时间戳
    Client->>APIGW: POST /api/v1/sync (Authorization: Bearer JWT)
    APIGW->>APIGW: 验证 JWT + 限流检查
    APIGW->>SaveSvc: 转发请求
    SaveSvc->>Cache: 获取分布式锁 (save:{playerId}:lock)
    alt 锁被占用
        SaveSvc-->>Client: 409 CONFLICT
    else 获取锁成功
        SaveSvc->>DB: 查询当前服务器 SaveData
        DB-->>SaveSvc: 返回 serverSaveData
        SaveSvc->>SaveSvc: 比对 localSaveData.lastUpdated vs serverSaveData.lastUpdated
        alt local 更新
            SaveSvc->>DB: 写入 localSaveData (Last-Write-Wins)
            DB-->>Replica: 主从同步
            SaveSvc->>Cache: 失效 save:{playerId} 缓存
            SaveSvc-->>Client: 200 OK + 新 SaveData
        else server 更新
            SaveSvc-->>Client: 200 OK + serverSaveData (玩家本地覆盖)
        end
        SaveSvc->>Cache: 释放分布式锁
    end
```

## 6. 部署图 (Deployment Diagram)

> 详见 `system-overview.md` §4。本节摘要。

### 6.1 v1.0 部署

```mermaid
flowchart LR
    PlayerPC["玩家 PC/Mac"]
    Steam[Steam]
    ItchIO[Itch.io]
    
    PlayerPC -->|"下载/更新"| Steam
    PlayerPC -->|"下载/更新"| ItchIO
```

### 6.2 v2.0+ 部署

```mermaid
flowchart TB
    PlayerDevice["玩家设备"]
    AWS["AWS 区域"]
    PlayerDevice -->|"HTTPS REST"| AWS
```

**完整部署图见 `system-overview.md` §4。**

## 7. 配置表 (Configuration)

| 字段 | 取值范围 | 默认值 | 单位 | 场景 |
|------|---------|-------|------|------|
| `c4.level1.contexts` | [1, 10] | 4 | 个 | L1 外部系统数 |
| `c4.level2.containers` | [1, 30] | 17 | 个 | v2.0+ 容器数 |
| `c4.level3.components` | [1, 50] | 14 | 个 | L3 组件数（客户端） |
| `c4.level3.serverComponents` | [1, 50] | 10 | 个 | L3 组件数（服务端） |
| `sequence.switchForward.maxMs` | [10, 50] | 16 | ms | 切换响应 |
| `sequence.roomComplete.maxMs` | [500, 2000] | 1000 | ms | 房间通关 |
| `sequence.saveWrite.maxMs` | [10, 200] | 50 | ms | 存档写入 |
| `sequence.cloudSync.maxMs` | [500, 3000] | 1500 | ms | 云同步 |

## 8. 边界条件 (Edge Cases)

| # | 触发 | 预期行为 | 涉及图 |
|---|------|---------|-------|
| **CE1** | 槽位在 Switching 中收到 R 键 | R 优先级 > 切换，重置当前房间 | §5.1 序列图 |
| **CE2** | 房间通关时存档写入失败 | 加载 backup + 降级无存档 | §5.3 序列图 |
| **CE3** | v2.0+ 云同步冲突（多设备同时编辑） | Last-Write-Wins + 服务器优先 | §5.4 序列图 |
| **CE4** | 客户端时钟漂移导致 JWT 失效 | ±5min 容差 + 401 重试 | §5.4 序列图 |
| **CE5** | API Gateway 限流触发 | 客户端降级为本地模式 | §3.2 L2 图 |
| **CE6** | PostgreSQL 主库故障 | 自动 failover 到 Replica | §3.2 L2 图 |
| **CE7** | Redis Sentinel 节点故障 | 自动选主新主节点 | §3.2 L2 图 |
| **CE8** | CloudFront CDN 缓存未命中 | 回源到 S3 + 边缘缓存 | §3.2 L2 图 |

## 9. 验收标准 (Acceptance Criteria)

- [x] **AC-01：** 文档包含完整 Frontmatter（7 字段）
- [x] **AC-02：** 文档包含 6 必填通用章节（目的 / 范围 / 配置表 / 边界条件 / 验收标准 / 风险与开放问题）
- [x] **AC-03：** C4 Level 1 System Context Mermaid 图（1 张，覆盖玩家 + 7 平台 + CDN + 数据库）
- [x] **AC-04：** C4 Level 2 Container Mermaid 图（v1.0 单机 + v2.0+ 微服务，2 张）
- [x] **AC-05：** C4 Level 3 Component Mermaid 图（客户端 14 模块 + 服务端 10 微服务，2 张）
- [x] **AC-06：** 序列图 ≥ 3 张（切换槽位 + 房间通关 + 存档写入 + 云同步，4 张）
- [x] **AC-07：** L1/L2/L3 三层容器/组件清单表格完整（v1.0 6 容器 + v2.0+ 17 容器 + 14 客户端组件 + 10 服务端组件）
- [x] **AC-08：** 边界条件 ≥ 6 条（CE1-CE8）
- [x] **AC-09：** 关联文档 / 关联代码 / 变更日志 / 待办事项齐全
- [x] **AC-10：** Mermaid 图 ≥ 6 张（实际 9+ 张：1 L1 + 2 L2 + 2 L3 + 4 Sequence）
- [x] **AC-11：** 文档总行数 ≥ 250 行（实际 ~500 行）

## 10. 关联文档

### 上游（本文档依赖）

- [`README.md`](./README.md) — 架构总览
- [`system-overview.md`](./system-overview.md) — 系统边界 + 14 模块 + 部署图
- [`module-breakdown.md`](./module-breakdown.md) — 14 模块详解
- [`docs/01-overview-v2.md`](../../docs/01-overview-v2.md) — 总览
- [`docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — SwitchSlot + 4 槽位
- [`docs/04-gameplay-flow-v2.md`](../../docs/04-gameplay-flow-v2.md) — 12 态状态机 + 主循环
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台
- [`design/api/README.md`](../api/README.md) — 18 端点 + 12 数据模型

### 下游（本文档被依赖）

- [`data-flow.md`](./data-flow.md) — 完整事件流 + 状态机
- [`tech-stack.md`](./tech-stack.md) — 技术栈详解
- [`deployment.md`](./deployment.md) — 7 平台分发策略
- [`risks-and-decisions.md`](./risks-and-decisions.md) — 风险 + ADR
- `docs/wiki/` — Wiki / Confluence 嵌入的 Mermaid 图

## 11. 关联代码模块

| 模块 | 路径 | 状态 | 引用 |
|------|------|:----:|------|
| 14 客户端模块 | `src/*` | 待创建 | 见 module-breakdown.md §2 |
| API Gateway 配置 | `infrastructure/gateway/nginx.conf` | 待创建 | §3.2 L2 |
| 10 服务端微服务 | `src/Server/Services/*` | 待创建 | §4.2 L3 |
| API Client | `src/Api/Client/AnzhongApiClient.cs` | 待创建 | §4.1 L3 |
| Docker Compose | `infrastructure/docker-compose.yml` | 待创建 | §3.2 L2 |
| Helm Charts (v2.0+) | `infrastructure/helm/` | 待创建 | §3.2 L2 |

## 12. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| R-01 | **C4 图随模块演化失同步** | 低 | 50% | 每次新增模块同步更新 + CI 检查 Mermaid 语法 | 已规划 |
| R-02 | **v2.0+ 服务端部署成本高** | 中 | 30% | ECS Fargate 按需付费 + 1 人 Solo 自动伸缩 | 已规划 |
| R-03 | **CloudFront 边缘缓存命中率低（首周）** | 低 | 40% | 预热脚本 + TTL 优化 | 已规划 |
| R-04 | **PostgreSQL 主从同步延迟导致读不一致** | 低 | 20% | 主从延迟 ≤ 100ms + 关键路径强制读主 | 已规划 |
| Q-01 | **是否在 Confluence 维护镜像** | 低 | — | GitHub README 嵌入即可；Confluence 镜像可选 | 倾向 GitHub |
| Q-02 | **是否使用 Structurizr DSL 生成图（vs 手写 Mermaid）** | 低 | — | v1.0 手写 Mermaid；v2.0 评估 Structurizr | 倾向手写 |

## 13. 待办事项 (TODO)

- [ ] **P0：** 实现 M01 Core 12 态状态机（与 L3 图一致）— 阻塞后续
- [ ] **P0：** 实现 M02 SwitchSlot 4 槽位 + 5 态机（与序列图 §5.1 一致）
- [ ] **P0：** 实现 M03 Room 通关判定三重条件（与序列图 §5.2 一致）
- [ ] **P0：** 实现 M07 SaveSystem 写入时序（与序列图 §5.3 一致）
- [ ] **P1：** 实现 v2.0+ AnzhongApiClient REST SDK（与序列图 §5.4 一致）
- [ ] **P1：** 编写 L1/L2/L3 图的 Confluence 镜像（可选）
- [ ] **P2：** 评估 Structurizr DSL 自动生成 C4 图
- [ ] **P2：** CI 检查 Mermaid 图语法（防图失同步）

## 14. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建**：C4 L1 System Context（1 图）+ C4 L2 Container v1.0 + v2.0+（2 图）+ C4 L3 Component 客户端 14 模块 + 服务端 10 微服务（2 图）+ 序列图 4 张（切换槽位 + 房间通关 + 存档写入 + 云同步）+ 部署图 2 张 + 边界条件 8 条（CE1-CE8）+ 容器/组件清单 30+ 项 + 配置表 8 字段 + 风险 4 项 + 待办 P0×5 P1×2 P2×2。 |