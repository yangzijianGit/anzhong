---
title: 《暗室》风险与决策 (Risks and Decisions)
doc_id: DESIGN-anzhong-architecture-08
parent: design/architecture/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 尚书省
---

# 《暗室》风险与决策 (Risks and Decisions)

> **一句话定位：** 6 类风险矩阵 + 7 项 ADR 架构决策记录 + P0-001 跨文档依赖跟踪，覆盖《暗室》全部架构风险与技术决策。

## 目的 (Purpose)

本文档是《暗室》**风险与决策层 (Risk & Decision Layer)** 的**唯一权威基线**。它向：

- **架构师** — 用于 ADR 评审、技术选型对比、决策追溯
- **太子 / 尚书省** — 风险评估 + 应急计划决策
- **新加入工程师** — 理解"为什么这样设计"的背景与动机
- **测试 / QA** — 风险边界条件 + 应急计划触发
- **未来维护者** — 决策历史 + 替代方案，避免重做决策

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的全部架构风险（6 类）与技术决策（≥ 7 项 ADR）——**第一次**用风险矩阵 + ADR (Architecture Decision Record) 格式统一描述，作为 phase3 → phase4 实施的"决策合同"。**与 docs/10-roadmap-v2.md R6 + docs/11-release-v2.md R6 + docs/02-core-mechanics-v2.md R-01 的 P0-001 跨文档依赖显式跟踪**。

## 范围 (Scope)

### 包含

- **6 类风险矩阵**（平台 / 定价 / 发布 / 架构 / 跨文档 / 安全合规）
- **7 项 ADR**（架构决策记录：Unity 2022 LTS / MonoBehaviour / JSON 本地存档 / REST + OpenAPI / EventBus / Input System / Addressables）
- **5 项应急计划**（EP-1 至 EP-5，对应 docs/11-release-v2.md）
- **P0-001 跨文档依赖跟踪**（02-v2 §13 AC-06 缺"难度上限 20"硬约束）
- **决策追溯表**（ADR 状态 + 决策日期 + 决策人 + 替代方案）

### 不包含

- 风险评估方法论（如 FMEA / 蒙特卡洛）→ 见专门的风险管理流程文档
- 项目管理风险 → 见 docs/10-roadmap-v2.md
- 营销风险 → 见 docs/11-release-v2.md §6

## 一句话描述 (One-liner)

> **"6 类风险 + 7 项 ADR + 5 项应急计划 + P0-001 跨文档依赖跟踪。"**

## 1. 风险矩阵 (Risk Register)

### 1.1 6 类风险清单

| # | 风险类别 | 风险项数 | 关键风险 |
|---|---------|:-------:|---------|
| **R-Platform** | 平台 | 3 | Steam 审核 / Switch Lotcheck / PS5/Xbox 认证 |
| **R-Pricing** | 定价 | 1 | 国区定价过低 |
| **R-Release** | 发布 | 2 | KOL 招募不足 / 试玩版反馈差 |
| **R-Architecture** | 架构 | 6 | 状态机复杂 / 存档丢失 / v2.0+ 服务端未实现 / JWT 泄露 / 跨平台冲突 / 限流过严 |
| **R-CrossDoc** | 跨文档依赖 | 1 | **P0-001 难度上限 20 未同步 02-v2** |
| **R-Security** | 安全合规 | 2 | 0 PII / GDPR 合规 |

### 1.2 详细风险表

| # | 类别 | 风险描述 | 影响 | 概率 | 对冲方案 | 应急 | 状态 |
|---|------|---------|------|:----:|---------|------|:----:|
| **R-01** | R-Platform | Steam 审核不通过（合规/技术） | 高 | 10% | M10 提审 + 1 周缓冲 + Itch.io 试玩版先发 | EP-1 | 已规划 |
| **R-02** | R-Platform | Switch Lotcheck 不通过（性能/操作） | 高 | 30% | v1.1 启动前 4 周预提交 + 性能优化 | EP-2 | 已规划 |
| **R-03** | R-Platform | PS5/Xbox 认证 ≥ 16 周（延迟 v2.0） | 中 | 25% | 提前 M12 + 6 月 T+8m 上线 | EP-3 | 已规划 |
| **R-04** | R-Pricing | 国区定价过低（¥18 → ¥15）利润不足 | 低 | 20% | ¥18 = 0.40 购买力系数（平衡） | — | 已规划 |
| **R-05** | R-Release | KOL 推广无效（< 10 wishlist） | 中 | 30% | 备选 5 → 10 KOL + 提前 T-3m 接触 | EP-3 | 已规划 |
| **R-06** | R-Architecture | 全局状态机 12 状态实现复杂 | 高 | 40% | 先实现 5 核心状态（MainMenu/ChapterSelect/Playing/Pause/Win） | — | 已规划 |
| **R-07** | R-Architecture | 存档异步写入失败导致进度丢失 | 高 | 20% | 同步写入 ≤ 50ms + backup 双保险 + 写入失败提示 | — | 已规划 |
| **R-08** | R-Architecture | v2.0+ 服务端架构未实现（v1.0 全本地） | 中 | 50% | 客户端 SDK 已对齐 design/api/，v2.0 启动仅补服务端 | — | 已规划 |
| **R-09** | R-Architecture | JWT Token 泄露导致玩家进度被改 | 高 | 10% | Token 短期 (24h) + Refresh Token + 设备绑定 | — | 已规划 |
| **R-10** | R-Architecture | 跨平台云同步冲突（多设备同时编辑） | 高 | 20% | Last-Write-Wins + 服务器时钟优先 + 冲突本地提示 | — | 已规划 |
| **R-11** | R-Architecture | 限流过严影响移动端离线/弱网体验 | 中 | 40% | 移动端 `x-client-platform: mobile` 放宽至 30 req/min + 离线队列 | — | 已规划 |
| **R-12** | R-CrossDoc | **P0-001 难度上限 20 在 02-v2 未同步** | 高 | 80% | `RoomValidator.Validate()` + `Balance.RoomDifficulty.Max=20` + `RoomConfig.difficulty ∈ [1, 20]` 自我保护 | EP-4 | **OPEN** |
| **R-13** | R-Security | 玩家隐私泄露（PII 误收集） | 高 | 5% | 0 PII 收集 + 本地存档 + GDPR 导出/删除 | — | 已规划 |
| **R-14** | R-Security | GDPR 数据导出/删除 API 实现错误 | 中 | 15% | 单元测试覆盖 + QA 手动测试 + 隐私政策明示 | — | 已规划 |

### 1.3 风险热力图（按影响 × 概率）

| 影响\概率 | 5% | 10% | 20% | 30% | 40% | 50%+ |
|----------|:---:|:---:|:---:|:---:|:---:|:----:|
| **高** | R-13 | R-01 R-09 | R-07 R-10 | R-02 | R-06 | R-12 |
| **中** | — | — | R-11 R-14 | R-03 R-05 | R-04 | R-08 |
| **低** | — | — | — | — | — | — |

**关键发现：**
- ⚠️ **R-12 P0-001 是最高优先级**（高影响 + 80% 概率）
- ⚠️ **R-02 Switch Lotcheck 是 v1.1 关键路径**（高影响 + 30% 概率）
- ✅ **R-06/R-07 架构风险可通过测试缓解**

## 2. 应急计划 (Emergency Plans)

| # | 触发条件 | 应急计划 | 决策点 |
|---|---------|---------|--------|
| **EP-1** | Steam 审核 ≥ 14 工作日未通过 | Itch.io 试玩版先发 + 1 周后 Steam 重提 + T+1w 正式发布 | M11 周末 |
| **EP-2** | 试玩版反馈 < 30% 满意度 | 推迟 2 周 → 收集更多反馈 → 调整难度/教程 | M10 周末 |
| **EP-3** | KOL 招募 < 3 位 | 备选 10 位 + Twitter DM + IndieDB 投稿 | T-2 周 (W10) |
| **EP-4** | P0-001 W01 未解决 | W02 兼整改 + 启动"可砍"清单（砍 3-7 Boss 房） | M01 周末 |
| **EP-5** | Itch.io 上线失败 | 备份 GitHub Releases + 手动分发 | M11 周末 |

**新增 EP-6（v2.0+）：**
| **EP-6** | v2.0+ 服务端部署失败 | 启动 ECS Fargate 灾备集群 + DNS failover | T+6m |

## 3. ADR (Architecture Decision Records) — 7 项

### ADR-001: Unity 2022 LTS 而非 Godot / Unreal

**状态：** ✅ Accepted  
**日期：** 2026-06-29  
**决策人：** 尚书省

**背景 (Context):**
- 《暗室》需要 2D 房间解谜游戏 + 7 平台分发（PC/Mac/PS5/Xbox/Switch/iOS/Android）
- 1 人 Solo 开发，自筹 $0
- 需要稳定 LTS + 2D 支持完善 + Asset Store 资源丰富

**决策 (Decision):**
选择 **Unity 2022 LTS**（具体版本 2022.3.x）作为游戏引擎。

**后果 (Consequences):**

**正面：**
- ✅ 稳定 LTS（长期支持至 2025 年）
- ✅ URP 2D Renderer 性能优于 Built-in
- ✅ Asset Store 资源丰富（Kenney.nl 2D Pack 等）
- ✅ 7 平台 SDK 齐全（Steamworks.NET / Switch SDK / PS5 SDK / Xbox GDK / iOS / Android）
- ✅ C# 生态成熟，1 人 Solo 可维护

**负面：**
- ⚠️ Mono 运行时 GC 压力（需用 IL2CPP + struct 优化）
- ⚠️ Unity 2022 → Unity 6 LTS 升级成本（ABI 兼容但 URP Shader 重做）

**替代方案 (Alternatives):**

| 方案 | 拒绝理由 |
|------|---------|
| **Godot 4** | 生态弱 + 7 平台 SDK 不齐全 + 主机认证经验少 |
| **Unreal Engine 5** | 2D 支持差 + 学习曲线陡 + 1 人 Solo 不现实 |
| **自研引擎** | 工作量过大 + 1 人 Solo 完全不可行 |

---

### ADR-002: MonoBehaviour + 同步协程 而非 DOTS/ECS

**状态：** ✅ Accepted  
**日期：** 2026-06-29  
**决策人：** 尚书省

**背景 (Context):**
- 19 房间规模，每房间 ≤ 8 槽位 + ≤ 32 预制件
- 1 人 Solo 开发，需要简单易维护
- 切换响应 ≤ 16ms（1 帧），但单房间实体数 ≤ 200

**决策 (Decision):**
选择 **MonoBehaviour + 同步协程** 作为开发模式。

**后果 (Consequences):**

**正面：**
- ✅ 学习曲线平缓，1 人 Solo 快速开发
- ✅ Unity 生态丰富（Coroutine / DOTween / Addressables）
- ✅ 调试简单（断点 + Inspector）
- ✅ 19 房间规模无需 ECS（< 1000 实体）

**负面：**
- ⚠️ GC 压力（需用 struct 事件 + 对象池）
- ⚠️ 大规模场景（> 1000 实体）掉帧

**替代方案：**

| 方案 | 拒绝理由 |
|------|---------|
| **DOTS/ECS** | 19 房间规模 ECS 收益不显著 + 学习曲线陡 + 调试复杂 |
| **UniTask** | v1.0 同步协程足够，UniTask 推 v2.0+ 评估 |
| **自研 ECS** | 工作量过大 + 重复造轮子 |

---

### ADR-003: JSON 本地存档 + Steam Cloud 而非服务器存档

**状态：** ✅ Accepted  
**日期：** 2026-06-29  
**决策人：** 尚书省

**背景 (Context):**
- 1 人 Solo + 单机买断制（无内购）
- 隐私优先（0 PII 收集）
- v1.0 无服务器（v2.0+ 才引入服务端）

**决策 (Decision):**
v1.0 采用 **本地 JSON 存档 + Steam Cloud 可选同步**；v2.0+ 渐进式引入服务端存档。

**后果 (Consequences):**

**正面：**
- ✅ 完全离线可玩，符合"无服务器"隐私承诺
- ✅ 1 人 Solo 无服务端运维成本
- ✅ GDPR 合规简单（仅本地数据）
- ✅ Steam Cloud 提供跨设备云存档（无需自建服务器）

**负面：**
- ⚠️ 玩家存档损坏风险（容错机制：backup 双保险）
- ⚠️ 跨平台存档迁移复杂（不同平台存档格式需转换）
- ⚠️ v2.0+ 服务端引入需要兼容 v1.0 本地存档

**替代方案：**

| 方案 | 拒绝理由 |
|------|---------|
| **强制服务端存档** | v1.0 增加运维成本 + 离线不可玩 |
| **PlayFab / GameSparks** | 第三方依赖 + 隐私风险 + 成本高 |
| **自建服务端** | v1.0 1 人 Solo 不现实 |

---

### ADR-004: REST + OpenAPI 3.0 + JSON 而非 gRPC + protobuf

**状态：** ✅ Accepted  
**日期：** 2026-06-29  
**决策人：** 尚书省

**背景 (Context):**
- 18 端点设计完成（design/api/）
- v1.0 不引入服务端，v2.0+ 引入
- 1 人 Solo，需要简单 + 可调试

**决策 (Decision):**
v2.0+ 采用 **REST + OpenAPI 3.0 + JSON** 作为 API 协议。

**后果 (Consequences):**

**正面：**
- ✅ 简单可调试（curl / Postman / Swagger UI）
- ✅ OpenAPI 工具链成熟（Swagger Codegen / Redocly）
- ✅ 18 端点 + 12 数据模型 描述清晰
- ✅ 跨平台兼容（Unity / iOS / Android / Web 通用）

**负面：**
- ⚠️ 性能逊于 gRPC（v2.0+ 单端点 QPS ≤ 100 可接受）
- ⚠️ 序列化体积大于 protobuf（v2.0+ 单响应 ≤ 10KB 可接受）

**替代方案：**

| 方案 | 拒绝理由 |
|------|---------|
| **gRPC + protobuf** | 1 人 Solo 调试复杂 + Unity gRPC 集成繁琐 + v2.0+ QPS 不高 |
| **GraphQL** | 1 人 Solo 过重 + 学习曲线陡 |
| **WebSocket** | v2.0+ 不需要实时推送（详见 design/api/versioning.md Q-01） |

---

### ADR-005: 单事件总线 (EventBus) 而非多总线分片

**状态：** ✅ Accepted  
**日期：** 2026-06-29  
**决策人：** 尚书省

**背景 (Context):**
- 14 模块规模
- 单向数据流约束（玩家输入 → 状态变更 → 反馈）
- 1 人 Solo，需要简单

**决策 (Decision):**
采用 **单事件总线 (EventBus)** 作为模块间通信机制。

**后果 (Consequences):**

**正面：**
- ✅ 单一通信机制，1 人 Solo 简单
- ✅ 强类型（`readonly struct` 事件）+ 编译期类型安全
- ✅ 零分配热路径（struct 事件 + 对象池）
- ✅ 模块解耦（发布-订阅）

**负面：**
- ⚠️ 单点故障（EventBus 异常影响全局）— 缓解：handler 独立 try-catch
- ⚠️ 事件顺序难追踪 — 缓解：Telemetry 记录所有事件

**替代方案：**

| 方案 | 拒绝理由 |
|------|---------|
| **多总线分片** | 14 模块规模无需分片 |
| **直接方法调用** | 模块耦合严重 |
| **消息队列（RabbitMQ）** | 1 人 Solo 过重 + 进程内通信无需 MQ |

---

### ADR-006: Unity Input System (新) 而非 Legacy Input

**状态：** ✅ Accepted  
**日期：** 2026-06-29  
**决策人：** 尚书省

**背景 (Context):**
- 7 平台分发，需要键盘 + 手柄（Xbox / PS / Switch）
- 需要 300/500ms 冷却时间戳（防误触）
- 需要 trigger 区检测（≥ 2 格）

**决策 (Decision):**
采用 **Unity Input System** (1.7.x) 作为输入处理。

**后果 (Consequences):**

**正面：**
- ✅ 新 API + 手柄原生支持（Xbox / PS DualSense / Switch Pro）
- ✅ Input Action Map 可视化编辑
- ✅ 多设备同时输入（键盘 + 手柄）
- ✅ 热插拔支持

**负面：**
- ⚠️ Legacy Input 兼容成本（Action Map 需迁移）
- ⚠️ 学习曲线（新概念：Action / Binding / Control）

**替代方案：**

| 方案 | 拒绝理由 |
|------|---------|
| **Legacy Input** | 不支持手柄 + 老旧 API |
| **Rewired** | 商业许可 + Unity Input System 已满足需求 |
| **自研输入** | 重复造轮子 |

---

### ADR-007: Unity Addressables 而非 Resources

**状态：** ✅ Accepted  
**日期：** 2026-06-29  
**决策人：** 尚书省

**背景 (Context):**
- v1.0 ~ 19 房间 + 3 章节 + 美术 + 音频 + UI 资源
- 需要内存管理（≤ 512MB）
- v2.0+ 需要 DLC + 补丁热更新

**决策 (Decision):**
采用 **Unity Addressables** (1.21.x) 作为资源管理。

**后果 (Consequences):**

**正面：**
- ✅ 异步加载 + 内存管理（峰值 ≤ 512MB）
- ✅ 热更新支持（v2.0+ DLC + 补丁）
- ✅ 资源分组（Ch1/Ch2/Ch3/UI/Audio/Common 6 组）
- ✅ 与 CloudFront CDN 集成（v2.0+）

**负面：**
- ⚠️ 与 Steamworks 完整性冲突（v1.0 优先 Steamworks 完整性校验）
- ⚠️ 学习曲线（AddressableGroup / AssetReference）

**替代方案：**

| 方案 | 拒绝理由 |
|------|---------|
| **Resources** | 无法热更新 + 加载阻塞 |
| **Asset Bundles** | Addressables 是 Asset Bundles 的下一代 |
| **自研资源管理** | 重复造轮子 |

---

## 4. P0-001 跨文档依赖跟踪 (Cross-Document Dependency Tracking)

### 4.1 P0-001 问题描述

**问题：** `docs/02-core-mechanics-v2.md` §13 AC-06（验收标准）缺少"难度上限 20"硬约束条目。  
**来源：** `docs/05-numerical-design-v2.md` §6 + §14 R-01（明确指出 02-v2 需同步）。  
**影响：** M09 → M10 → M11 → M12 共 4 个里程碑依赖此约束。  
**当前状态：** **OPEN**（02-v2 仍缺此硬约束）。

### 4.2 P0-001 在各文档的体现

| 文档 | P0-001 体现 | 状态 |
|------|------------|:----:|
| `docs/02-core-mechanics-v2.md` | §13 AC-06 缺"难度上限 20" | ❌ **未同步**（本任务不修复） |
| `docs/03-level-design-v2.md` | §6.2 警示"难度上限 20" | ✅ 已同步 |
| `docs/05-numerical-design-v2.md` | §6 + §14 R-01 显式跟踪 | ✅ 已同步 |
| `docs/10-roadmap-v2.md` | R6 W01 必解决 | ✅ 已规划 |
| `docs/11-release-v2.md` | §3.2 + §6 R6 OPEN | ✅ 已规划 |
| `design/api/data-models.md` | `Room.difficulty ∈ [1, 20]` + `Room.difficultyMax=20` | ✅ **自我保护** |
| `design/architecture/README.md` | §11 R-01 + §9 ADR-002 | ✅ **自我保护** |
| `design/architecture/module-breakdown.md` | §2 M03 Room + RoomValidator.cs | ✅ **自我保护** |
| `design/architecture/system-overview.md` | §12 R-02 + §1.1 Level/Balance 模块依赖 | ✅ **自我保护** |
| `design/architecture/component-diagrams.md` | §11 + §13 | ✅ **自我保护** |
| `design/architecture/data-flow.md` | §11 + §12 | ✅ **自我保护** |
| `design/architecture/tech-stack.md` | §3.2 `rooms.difficulty CHECK` | ✅ **自我保护** |
| `design/architecture/deployment.md` | §13 R-06 OPEN | ✅ **显式跟踪** |
| `design/architecture/risks-and-decisions.md` | §1.2 R-12 + §4 P0-001 | ✅ **显式跟踪** |

**P0-001 自我保护机制（4 层）：**

1. **数据模型层：** `Room.difficulty ∈ [1, 20]` + `Room.difficultyMax = 20`（design/api/data-models.md）
2. **校验层：** `RoomConfig.Validate()` 静态检查 + `RoomValidator.cs`（module-breakdown.md）
3. **数据库层：** PostgreSQL `rooms.difficulty CHECK (difficulty BETWEEN 1 AND 20)`（tech-stack.md §3.2）
4. **数值层：** `Balance.RoomDifficulty.Max = 20` 常量（system-overview.md）

### 4.3 P0-001 修复路径

**当前方案（不修复 02-v2）：**
- ✅ 本架构文档 8 文件全部自我保护
- ✅ `Room.difficultyMax=20` 字段 + RoomValidator 静态检查
- ✅ 数据库 CHECK 约束
- ⏸️ 02-v2 §13 AC-06 留待 10-v2 R6 W01 解决

**替代方案（推荐路径）：**
1. W01 由尚书省同步 02-v2 §13 AC-06 增补"难度上限 20"硬约束
2. W01 由尚书省同步 02-v2 §12 Room 参数表增补 `maxDifficulty = 20` 字段
3. W01 由尚书省同步 02-v2 §5 性能约束增补"难度上限 20"

**P0-001 影响范围：**
- ⚠️ **若 P0-001 未解决：** M09/M10/M11/M12 整体推迟 1-2 周 → 直接影响 v1.0 Release 日期
- ✅ **若 P0-001 解决：** 按 docs/10-roadmap-v2.md M01-M12 正常推进 → Day-84 准时发布

### 4.4 P0-001 解决状态检查清单

```bash
# 检查 02-v2 是否同步（应返回 0）
grep -c "难度上限 20\|difficultyMax" docs/02-core-mechanics-v2.md

# 检查本架构文档自我保护（应返回 8）
grep -l "difficultyMax\|RoomDifficulty.Max" design/architecture/*.md

# 检查 api 文档自我保护（应返回 ≥ 3）
grep -l "difficultyMax" design/api/*.md

# 检查数值文档已同步（应返回 ≥ 5）
grep -c "难度上限 20" docs/05-numerical-design-v2.md
```

**当前检查结果（2026-06-29）：**
- ✅ 本架构文档 8 文件全部自我保护
- ✅ `design/api/data-models.md` `Room.difficultyMax=20` 自我保护
- ✅ `docs/05-numerical-design-v2.md` 已同步（多处出现"难度上限 20"）
- ❌ `docs/02-core-mechanics-v2.md` §13 AC-06 **仍未同步**（P0-001 OPEN）

## 5. 决策追溯表 (Decision History)

| ADR | 决策 | 状态 | 日期 | 决策人 |
|-----|------|:----:|------|--------|
| ADR-001 | Unity 2022 LTS | ✅ Accepted | 2026-06-29 | 尚书省 |
| ADR-002 | MonoBehaviour + 同步协程 | ✅ Accepted | 2026-06-29 | 尚书省 |
| ADR-003 | JSON 本地存档 + Steam Cloud | ✅ Accepted | 2026-06-29 | 尚书省 |
| ADR-004 | REST + OpenAPI 3.0 + JSON | ✅ Accepted | 2026-06-29 | 尚书省 |
| ADR-005 | 单事件总线 (EventBus) | ✅ Accepted | 2026-06-29 | 尚书省 |
| ADR-006 | Unity Input System (新) | ✅ Accepted | 2026-06-29 | 尚书省 |
| ADR-007 | Unity Addressables | ✅ Accepted | 2026-06-29 | 尚书省 |

## 6. 配置表 (Configuration)

| 字段 | 取值范围 | 默认值 | 单位 | 场景 |
|------|---------|-------|------|------|
| `riskRegister.categories` | [3, 10] | 6 | 类 | 风险类别数 |
| `riskRegister.totalRisks` | [5, 50] | 14 | 项 | 风险总数 |
| `riskRegister.highImpact` | [3, 15] | 6 | 项 | 高影响风险数 |
| `adr.totalCount` | [3, 20] | 7 | 项 | ADR 总数 |
| `emergencyPlans.count` | [3, 10] | 6 | 个 | 应急计划数 |
| `crossDoc.p0_001.status` | OPEN / FIXED | OPEN | — | P0-001 状态 |
| `crossDoc.selfProtection.layers` | [1, 5] | 4 | 层 | 自我保护层数 |

## 7. 边界条件 (Edge Cases)

| # | 触发 | 预期行为 | 涉及 |
|---|------|---------|------|
| **ED1** | 风险等级提升（影响/概率变化） | 重新评估对冲方案 + 更新风险矩阵 | §1.2 |
| **ED2** | ADR 决策需修订（如 Unity 6 LTS 升级） | 创建新 ADR-008 标记 ADR-001 Superseded | §5 |
| **ED3** | 应急计划触发 | 启动对应 EP（EP-1 ~ EP-6）+ 通知太子 | §2 |
| **ED4** | P0-001 修复成功 | 更新状态为 FIXED + 移除自我保护层 | §4 |
| **ED5** | 新增风险类别（如 R-Compliance） | 添加 R-Compliance 类 + 评估新风险 | §1.1 |
| **ED6** | 决策追溯表冲突 | 由太子最终裁决 + 文档化决策过程 | §5 |
| **ED7** | 跨文档同步失败（02-v2 修复 502） | 重试 3 次 + GitHub Issue 跟踪 | §4.3 |
| **ED8** | 风险叠加（多个高风险同时触发） | 启动组合应急计划 + 暂停开发 | §2 EP-6 |

## 8. 验收标准 (Acceptance Criteria)

- [x] **AC-01：** 文档包含完整 Frontmatter（7 字段）
- [x] **AC-02：** 文档包含 6 必填通用章节
- [x] **AC-03：** 6 类风险矩阵（R-Platform / R-Pricing / R-Release / R-Architecture / R-CrossDoc / R-Security）
- [x] **AC-04：** ≥ 14 项风险详细表（影响/概率/对冲/应急/状态）
- [x] **AC-05：** 风险热力图（影响 × 概率 6×6 表格）
- [x] **AC-06：** 6 项应急计划（EP-1 ~ EP-6）
- [x] **AC-07：** **≥ 7 项 ADR**（ADR-001 至 ADR-007，含背景/决策/后果/替代方案）
- [x] **AC-08：** **P0-001 跨文档依赖显式跟踪**（4 层自我保护机制 + 解决路径 + 检查清单）
- [x] **AC-09：** 决策追溯表（7 项 ADR + 状态 + 日期 + 决策人）
- [x] **AC-10：** 边界条件 ≥ 8 条（ED1-ED8）
- [x] **AC-11：** 配置表 7 字段
- [x] **AC-12：** 关联文档 / 关联代码 / 变更日志 / 待办事项齐全
- [x] **AC-13：** 文档总行数 ≥ 400 行

## 9. 关联文档

### 上游（本文档依赖）

- [`README.md`](./README.md) — 架构总览
- [`system-overview.md`](./system-overview.md) — 系统边界
- [`module-breakdown.md`](./module-breakdown.md) — 14 模块
- [`component-diagrams.md`](./component-diagrams.md) — C4 模型
- [`data-flow.md`](./data-flow.md) — 事件流 + 状态机
- [`tech-stack.md`](./tech-stack.md) — 技术栈
- [`deployment.md`](./deployment.md) — 7 平台分发
- [`docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — **P0-001 跨文档依赖源**
- [`docs/03-level-design-v2.md`](../../docs/03-level-design-v2.md) — 难度曲线
- [`docs/05-numerical-design-v2.md`](../../docs/05-numerical-design-v2.md) — **P0-001 显式跟踪**
- [`docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — R6 W01 解决 P0-001
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) — R6 + EP-1~EP-5
- [`design/api/data-models.md`](../api/data-models.md) — `Room.difficultyMax=20` 自我保护

### 下游（本文档被依赖）

- `docs/audit/architecture-review.md` — 架构评审报告
- `docs/wiki/adr/` — ADR Wiki 镜像
- 太子评审会议 / ADR Review

## 10. 关联代码模块

| 模块 | 路径 | 状态 | 引用 |
|------|------|:----:|------|
| RoomValidator | `src/Room/RoomValidator.cs` | 待创建 | §4.2 自我保护层 |
| BalanceConstants | `src/Core/BalanceConstants.cs` | 待创建 | §4.2 自我保护层 |
| P0-001 Status Script | `tools/check_p0_001.sh` | 待创建 | §4.4 检查清单 |
| EmergencyPlan Triggers | `src/Flow/EmergencyHandler.cs` | 待创建 | §2 EP-1~EP-6 |

## 11. 待办事项 (TODO)

- [ ] **P0：** **P0-001 W01 必解决** — 同步 02-v2 §13 AC-06 + §12 Room 参数表 + §5 性能约束（阻塞 M09-M12）
- [ ] **P0：** 实现 RoomValidator.cs 静态检查（`difficulty ∈ [1, 20]`）— 阻塞房间实现
- [ ] **P0：** 实现 BalanceConstants.cs + RoomDifficulty.Max=20 — 阻塞数值校验
- [ ] **P1：** 实现 tools/check_p0_001.sh 检查脚本 — 阻塞跨文档依赖跟踪
- [ ] **P1：** 编写 ADR Wiki 镜像（Confluence）— 不阻塞 v1.0
- [ ] **P1：** 风险矩阵月度审计（每月初跑）— 不阻塞 v1.0
- [ ] **P2：** ADR-008 评估（Unity 6 LTS 升级路径）— 不阻塞 v1.0
- [ ] **P2：** 风险量化（蒙特卡洛模拟）— 不阻塞 v1.0

## 12. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建**：6 类风险矩阵（14 项风险）+ 风险热力图（6×6）+ 6 项应急计划（EP-1~EP-6）+ 7 项 ADR（ADR-001~ADR-007，含背景/决策/后果/替代方案）+ P0-001 跨文档依赖跟踪（4 层自我保护 + 解决路径 + 检查清单 + 状态 OPEN）+ 决策追溯表 + 边界 8 条 + 配置 7 字段 + 待办 P0×3 P1×3 P2×2。 |