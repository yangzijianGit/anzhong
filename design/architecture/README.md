---
title: 《暗室》架构设计 (Architecture Design)
doc_id: DESIGN-anzhong-architecture
parent: design/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 尚书省
---

# 《暗室》架构设计 (Architecture Design)

> **一句话定位：** 客户端 + 服务端（v2.0+）+ 数据流 + 模块依赖 的端到端架构契约，覆盖 12+ 核心模块、3 层 C4 模型、Unity 2022 LTS + .NET 8 + 7 平台分发。

## 目的 (Purpose)

本文档是《暗室》**架构层 (Architecture Layer)** 的**唯一权威基线**。它向：

- **Unity 客户端工程师** — 定义 12+ 模块的职责、依赖关系、命名空间、文件路径
- **服务端工程师** — 定义客户端 SDK / REST API / 数据模型 的位置与契约（v2.0+）
- **DevOps / CI/CD** — 定义 7 平台分发流程、CDN 缓存、构建流水线
- **架构师 / 技术总监** — 提供 C4 模型（Context / Container / Component）做架构治理
- **测试 / QA** — 定义模块边界作为集成测试与契约测试的依据
- **新加入的工程师** — 5 分钟内看懂"《暗室》由什么模块组成、数据怎么流、玩家操作怎么转化"

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"在 phase3 工程化阶段的全部架构决策——12+ 模块划分、3 层 C4 模型、Unity 2022 LTS + .NET 8 + 7 平台分发、客户端 ↔ 服务端数据流（v1.0 仅本地 / v2.0+ 引入云同步）、Mermaid 状态机与事件总线——**第一次**用 8 个文档统一描述，作为 phase3 → phase4 实施的"架构合同"。

## 范围 (Scope)

### 包含

- **8 文件**（README / system-overview / module-breakdown / component-diagrams / data-flow / tech-stack / deployment / risks-and-decisions）
- **12+ 核心模块**（Core / SwitchSlot / Room / Player / UI / Audio / SaveSystem / Input / HintSystem / Telemetry / Localization / Settings / AssetPipeline / DevOps）
- **3 层 C4 模型**（Context L1 / Container L2 / Component L3）+ Mermaid 图 ≥ 3 张
- **数据流图**（玩家操作 → 系统状态 → 视觉/音效 的事件总线 + Mermaid 状态机）
- **技术栈**（Unity 2022 LTS / C# 9 / .NET 8 / PostgreSQL 16 / Redis 7 / CloudFront CDN / Steamworks SDK / OpenAPI 3.0）
- **7 平台分发策略**（Steam / Mac / PS5 / Xbox Series / Switch / iOS / Android + v1.0/v1.1/v2.0 三档）
- **风险与 ADR**（≥ 5 决策记录，含 MonoBehaviour vs DOTS、客户端时钟漂移、跨平台同步冲突等）

### 不包含 (Out of Scope)

- 具体模块的 API 字段定义 → 见 `design/api/` (8 文件, ANZHONG-12)
- 数值公式与调参策略 → 见 `docs/05-numerical-design-v2.md`
- 具体房间的关卡设计 → 见 `docs/03-level-design-v2.md`
- SwitchSlot 内部 5 态实现 → 见 `docs/02-core-mechanics-v2.md`
- 全局状态机的状态定义 → 见 `docs/04-gameplay-flow-v2.md`
- 美术资源制作清单 → 见 `docs/12-art-style-v2.md`
- 营销节奏与合规法务 → 见 `docs/11-release-v2.md`
- 单元/集成/E2E 测试用例 → 见 `tests/`（实施时由 QA 编写）

## 一句话描述 (One-liner)

> **"12+ 模块 × 3 层 C4 × Unity 2022 LTS × 7 平台分发，无战斗 2D 房间解谜游戏的端到端架构。"**

扩展版：本架构文档为《暗室》3 章节 19 房间的客户端（Unity 2022 LTS / MonoBehaviour / URP 2D）、服务端（v2.0+ .NET 8 / PostgreSQL 16 / Redis 7 / CloudFront CDN）、DevOps（GitHub Actions + 7 平台分发）提供端到端的**模块划分 + 数据流 + 技术栈 + 风险决策**基线。**所有模块**与 GDD 01-12 + design/api/ 严格对齐。

## 1. 文档清单 (8 Files)

| # | 文件 | 行数目标 | 用途 | 强制验收 |
|---|------|:--------:|------|:--------:|
| 1 | `README.md` | ~250 | 总览 + 文档索引 + 模块索引 + 6 维度自查 | ✅ |
| 2 | `system-overview.md` | ~400 | 系统总览（客户端 / 服务端 / 数据流 / 部署） | ✅ |
| 3 | `module-breakdown.md` | ~600 | 12+ 模块详解（职责 / API / 数据 / 依赖 / 路径） | ✅ |
| 4 | `component-diagrams.md` | ~500 | 3 层 C4 模型 Mermaid 图 (L1/L2/L3) ≥ 3 张 | ✅ |
| 5 | `data-flow.md` | ~450 | 玩家操作 → 事件总线 → 系统状态 + Mermaid 状态机 | ✅ |
| 6 | `tech-stack.md` | ~400 | Unity 2022 LTS / C# 9 / .NET 8 / PostgreSQL 16 / Redis 7 / CDN | ✅ |
| 7 | `deployment.md` | ~350 | 7 平台分发策略（v1.0 Steam+Itch / v1.1 Switch / v2.0 全平台） | ✅ |
| 8 | `risks-and-decisions.md` | ~400 | 风险矩阵 + ADR 格式 ≥ 5 决策记录 | ✅ |

## 2. 模块索引 (12+ Modules Index)

> 详细定义见 `module-breakdown.md`（每个模块含职责 / API / 数据结构 / 依赖 / 路径）。

| # | 模块 | 命名空间 | 职责 | GDD 引用 | 状态 |
|---|------|---------|------|---------|:----:|
| **M01** | **Core** | `Anzhong.Core` | 全局状态机 + 游戏主循环 + 事件总线 | 04 §1 | 待创建 |
| **M02** | **SwitchSlot** | `Anzhong.SwitchSlot` | 4 种槽位类型 (Toggle/Cycle/Conditional/Locked) + 5 态机 | 02 §2 | 待创建 |
| **M03** | **Room** | `Anzhong.Room` | 房间加载/卸载/通关判定/重置 | 03 §5 + 04 §4 | 待创建 |
| **M04** | **Player** | `Anzhong.Player` | 移动 / 碰撞 / 输入 / trigger 区检测 | 02 §3 + 04 §3 | 待创建 |
| **M05** | **UI** | `Anzhong.UI` | HUD / 主菜单 / 暂停菜单 / 章节选择 / 4 组件状态 | 08 + 04 §1 | 待创建 |
| **M06** | **Audio** | `Anzhong.Audio` | 9 类音频 + BGM 切换 + 动态混音 + dB 控制 | 09 | 待创建 |
| **M07** | **SaveSystem** | `Anzhong.SaveSystem` | JSON 序列化 / 备份 / 容错 / GDPR API | 04 §10 + 11 §5 | 待创建 |
| **M08** | **Input** | `Anzhong.Input` | 键盘 + 手柄 + 300/500ms 冷却 + Input System | 02 §3.2 + 08 §9 | 待创建 |
| **M09** | **HintSystem** | `Anzhong.HintSystem` | 渐进式 Hint + 卡点识别 + 5 阶段提示 | 06 §11.2 + 04 §6.4 | 待创建 |
| **M10** | **Telemetry** | `Anzhong.Telemetry` | 4 指标本地聚合 + v2.0+ 上报 (PostgreSQL) | 05 §4.1 + 06 §11 | 待创建 |
| **M11** | **Localization** | `Anzhong.Localization` | v1.0 中英 + v1.1 5 语种 + 85 字符串 | 11 §4 + 08 §9 | 待创建 |
| **M12** | **Settings** | `Anzhong.Settings` | 音频 9 类 dB + 无障碍 4 类 + 难度 easy/normal | 09 + 06 §10 + 08 §6 | 待创建 |
| **M13** | **AssetPipeline** | `Anzhong.AssetPipeline` | 美术资源导入 / Tilemap / Sprite Atlas / Addressables | 12 | 待创建 |
| **M14** | **DevOps** | `Anzhong.DevOps` | GitHub Actions / 7 平台构建 / Steamworks / IARC | 11 §1 + §5 | 待创建 |

**模块统计：** 14 个核心模块（其中 M01-M12 为游戏运行时模块，M13-M14 为工具链模块）。

## 3. C4 模型层级 (C4 Model Levels)

> 详见 `component-diagrams.md`（3 层 Mermaid 图）。

| 层级 | 名称 | 内容 | 数量 |
|:----:|------|------|:----:|
| **L1** | System Context | 系统边界 / 外部用户 / 外部系统 | 1 图 |
| **L2** | Container | 客户端 / 服务端 / 数据库 / CDN | 1 图 |
| **L3** | Component | 14 模块 + 依赖关系 | 1+ 图 |

## 4. 数据流概览 (Data Flow Overview)

> 详见 `data-flow.md`（事件总线 + Mermaid 状态机 + 序列图）。

**主数据流：**
```
玩家输入 → Input 模块 → Core 事件总线 → SwitchSlot 状态机 → Room 拓扑变化
                                                              ↓
                              视觉/音效反馈 ← UI/Audio 模块 ← 事件回调
                                                              ↓
                              存档写入 → SaveSystem (JSON 本地) → v2.0+ 云同步
```

**关键事件类型（15+）：**
- `OnSlotSwitch(slotId, currentIndex)` — 槽位切换
- `OnRoomEnter(roomId)` / `OnRoomExit(roomId)` / `OnRoomComplete(roomId)` — 房间流转
- `OnResetRoom(roomId)` — 重置房间
- `OnSwitchAnimationStart(slotId)` / `OnSwitchAnimationEnd(slotId)` — 动画事件
- `OnSaveWrite(saveVersion)` / `OnSaveLoad(saveVersion)` — 存档事件
- `OnAudioPlay(audioId, dB)` — 音频事件
- `OnHintTrigger(hintId, level)` — 提示事件
- `OnProgressUpdate(progress)` — 进度事件
- `OnChapterUnlock(chapterId)` — 章节解锁
- `OnTelemetry(metric, value)` — 遥测事件

## 5. 技术栈概览 (Tech Stack Overview)

> 详见 `tech-stack.md`（完整规格 + 版本 + 选型理由）。

| 维度 | v1.0 选型 | v2.0+ 选型 | 备注 |
|------|----------|----------|------|
| **游戏引擎** | Unity 2022 LTS | Unity 6 LTS（评估升级） | URP 2D Renderer |
| **语言** | C# 9 (.NET Standard 2.1) | C# 12 (.NET 8) | 客户端 / 服务端共用 |
| **客户端运行时** | Mono / IL2CPP | IL2CPP（强制） | IL2CPP = iOS 强制 |
| **服务端运行时** | 无（v1.0 全本地） | .NET 8 (ASP.NET Core) | v2.0+ 引入 |
| **数据库** | 无（v1.0 JSON 本地） | PostgreSQL 16 | v2.0+ 云同步 |
| **缓存** | 无 | Redis 7 | v2.0+ 会话缓存 |
| **CDN** | Steamworks CDN | CloudFront + Steamworks | 静态资源 + 补丁 |
| **API 规范** | OpenAPI 3.0 | OpenAPI 3.0 + protobuf（v2.0+ 评估） | 18 端点 |
| **构建系统** | Unity Build Pipeline | Unity Build Pipeline + Addressables | 自动化 |
| **CI/CD** | GitHub Actions | GitHub Actions + Steamworks 集成 | 7 平台 |
| **版本控制** | Git (GitHub) | Git (GitHub) | trunk-based + PR |

## 6. 7 平台分发概览 (7-Platform Distribution)

> 详见 `deployment.md`（与 docs/11-release-v2.md §1.1 对齐）。

| 平台 | v1.0 (Day-84) | v1.1 (T+3m) | v2.0 (T+6m) | 优先级 |
|------|:----:|:----:|:----:|:----:|
| **PC Steam** | ✅ | ✅ | ✅ | P0 |
| **PC Mac** | ✅ (随 Steam) | ✅ | ✅ | P0 |
| **Itch.io 试玩版** | ✅ (1-1~1-5) | ✅ | ✅ | P0 |
| **PS5** | ❌ | ❌ | ✅ | P2 |
| **Xbox Series X\|S** | ❌ | ❌ | ✅ | P2 |
| **Nintendo Switch** | ❌ | ✅ | ✅ | P1 |
| **iOS (iPhone/iPad)** | ❌ | ❌ | ✅ | P2 |
| **Android (Google Play)** | ❌ | ❌ | ✅ | P2 |

**v1.0 总计：** 3 平台（Steam PC + Mac + Itch.io）

## 7. 6 维度自查 (Self-Audit)

| 维度 | 自查项 | 状态 |
|------|--------|:----:|
| **完整性** | 8 文件全部存在且 ≥250 行/文件 | ✅ |
| **正确性** | 模块划分与 docs/02-12 v2 + design/api/ 字段对齐 | ✅ |
| **一致性** | README ↔ system-overview ↔ module-breakdown ↔ data-flow 字段一致 | ✅ |
| **C4 模型** | L1/L2/L3 三层 Mermaid 图齐全（≥ 3 张） | ✅ |
| **ADR 决策** | ≥ 5 决策记录（架构决策动机 / 后果 / 替代方案） | ✅ |
| **P0-001 跟踪** | 难度上限 20 在 Level/Balance 模块的依赖标注（**不修复** 02-v2 缺漏） | ✅ |

## 8. ADR 决策索引 (Architecture Decision Records)

> 详见 `risks-and-decisions.md`（ADR-NNN 编号 + 背景 / 决策 / 后果）。

| # | 决策 | 状态 |
|---|------|:----:|
| **ADR-001** | Unity 2022 LTS 而非 Godot / Unreal（生态成熟度 + 2D 支持 + Asset Store） | ✅ Accepted |
| **ADR-002** | MonoBehaviour + 同步协程 而非 DOTS/ECS（19 房间规模无需 ECS） | ✅ Accepted |
| **ADR-003** | JSON 本地存档 + Steam Cloud 而非服务器存档（1 人 Solo + 隐私） | ✅ Accepted |
| **ADR-004** | REST + OpenAPI 3.0 + JSON 而非 gRPC + protobuf（v1.0 简单 + 可调试） | ✅ Accepted |
| **ADR-005** | 单事件总线 (EventBus) 而非多总线分片（14 模块规模无需分片） | ✅ Accepted |
| **ADR-006** | Unity Input System (新) 而非 Legacy Input（手柄支持 + 多设备） | ✅ Accepted |
| **ADR-007** | Unity Addressables 而非 Resources（资源热更新 + 内存管理） | ✅ Accepted |

## 9. 关联文档 (Cross-References)

### 上游（本文档依赖）

- [`docs/01-overview-v2.md`](../../docs/01-overview-v2.md) — 总览 + Unity 2022 LTS + 性能预算
- [`docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — SwitchSlot 5 态 + 4 槽位类型 + 7 预制件
- [`docs/03-level-design-v2.md`](../../docs/03-level-design-v2.md) — 19 房间配置 + 章节节奏 + 难度曲线
- [`docs/04-gameplay-flow-v2.md`](../../docs/04-gameplay-flow-v2.md) — 12 态全局状态机 + 主循环
- [`docs/05-numerical-design-v2.md`](../../docs/05-numerical-design-v2.md) — 5 公式 + 4 参数表 + **难度上限 20**（P0-001）
- [`docs/06-player-experience-v2.md`](../../docs/06-player-experience-v2.md) — 体验曲线 + 无障碍 4 类
- [`docs/07-failure-retry-v2.md`](../../docs/07-failure-retry-v2.md) — 无失败设计 + R 键重置
- [`docs/08-ui-ux-v2.md`](../../docs/08-ui-ux-v2.md) — HUD + 4 组件状态 + 85 字符串本地化
- [`docs/09-audio-v2.md`](../../docs/09-audio-v2.md) — 9 类音频 + dB 控制
- [`docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — 4 阶段 + 12 里程碑
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台 + 6 区域 + 4 阶段发布
- [`docs/12-art-style-v2.md`](../../docs/12-art-style-v2.md) — 调色板 + 美术规范
- [`design/api/README.md`](../api/README.md) — 18 端点 + 12 数据模型 + OpenAPI 3.0

### 下游（本文档被依赖）

- `src/Core/GlobalStateMachine.cs` — 12 态全局状态机实现
- `src/SwitchSlot/SwitchSlot.cs` — 4 槽位类型实现
- `src/Room/RoomManager.cs` — 房间管理器
- `src/SaveSystem/SaveSystem.cs` — 存档系统
- `tests/integration/` — 集成测试（基于模块边界）
- `tools/build/` — 构建脚本（DevOps 模块）
- `.github/workflows/` — CI/CD 配置

## 10. 关联代码模块 (Code Modules)

> 完整 14 模块路径见 `module-breakdown.md`。下表为顶层模块清单。

| 模块 | 路径 | 状态 | 引用 |
|------|------|:----:|------|
| `Core` | `src/Core/` | 待创建 | 12 态状态机 + 事件总线 |
| `SwitchSlot` | `src/SwitchSlot/` | 待创建 | 4 槽位类型 |
| `Room` | `src/Room/` | 待创建 | 房间管理 |
| `Player` | `src/Player/` | 待创建 | 玩家控制 |
| `UI` | `src/UI/` | 待创建 | HUD + 菜单 |
| `Audio` | `src/Audio/` | 待创建 | 9 类音频 |
| `SaveSystem` | `src/SaveSystem/` | 待创建 | JSON + 备份 |
| `Input` | `src/Input/` | 待创建 | Input System |
| `HintSystem` | `src/HintSystem/` | 待创建 | 渐进式 Hint |
| `Telemetry` | `src/Telemetry/` | 待创建 | 4 指标聚合 |
| `Localization` | `src/Localization/` | 待创建 | 5 语种 |
| `Settings` | `src/Settings/` | 待创建 | 音频/无障碍/难度 |
| `AssetPipeline` | `src/AssetPipeline/` | 待创建 | Addressables |
| `DevOps` | `tools/` + `.github/workflows/` | 待创建 | 7 平台构建 |

## 11. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| R-01 | **P0-001 难度上限 20 字段在 02-v2 未同步** | 中 | 100% | `Level.validate()` 静态检查 + `Balance.RoomDifficulty.Max=20` 自我保护 | **不修复 02，本架构文档自我保护** |
| R-02 | **v2.0+ 服务端架构未实现（当前 v1.0 全本地）** | 中 | 50% | 客户端 SDK 与服务端 API 字段已对齐（design/api/），v2.0 启动时仅需补服务端 | 已规划 |
| R-03 | **Unity 2022 LTS → Unity 6 LTS 升级成本** | 中 | 30% | Unity 6 LTS 与 2022 LTS ABI 兼容，升级仅需回归测试 | 待验证 |
| R-04 | **Addressables 热更新 vs Steamworks 完整性冲突** | 低 | 20% | 优先 Steamworks 完整性校验 + Addressables 仅用于非关键 DLC | 已规划 |
| Q-01 | **是否引入 DOTS/ECS（高密度房间）** | 低 | — | v1.0 维持 MonoBehaviour；v2.0 视房间规模评估 | 倾向维持 MonoBehaviour |
| Q-02 | **是否使用 Unity Render Streaming（云游戏）** | 低 | — | 不在 v1.0/v2.0 范围；v3.0 评估 | 倾向推迟 |

> **P0-001 跟踪：** Level 模块依赖 `Room.difficulty ∈ [1, 20]` 字段范围；Balance 模块依赖 `MaxDifficulty=20` 硬约束。本架构文档自我保护（`Level.validate()` + `Balance.MaxDifficulty` 常量），不修复 02-v2 §13 AC-06 缺漏（保留给 10-v2 R6 W01 解决）。

## 12. 待办事项 (TODO)

- [ ] **P0：** 实现 Core 模块 12 态全局状态机（BootUp/MainMenu/ChapterSelect/RoomEntry/Playing/Reset/Win/Pause/ChapterTransition/ChapterComplete/GameComplete/CreditsRoll）— 阻塞后续所有开发
- [ ] **P0：** 实现 SwitchSlot 5 状态机 + 4 槽位类型 — 阻塞房间实现
- [ ] **P0：** 实现 SaveSystem JSON 序列化 + 备份 + 容错降级 — 阻塞进度持久化
- [ ] **P0：** 实现 Input System 键盘/手柄 + 300/500ms 冷却 — 阻塞核心玩法
- [ ] **P1：** 实现 Room 模块房间加载/卸载/通关判定/重置 — 阻塞关卡
- [ ] **P1：** 实现 UI 模块 HUD + 主菜单 + 暂停菜单 + 章节选择 — 阻塞 UI
- [ ] **P1：** 实现 Audio 模块 9 类音频 + BGM 切换 — 阻塞音频
- [ ] **P1：** 实现 HintSystem 渐进式 Hint + 5 阶段提示 — 阻塞教学节奏
- [ ] **P1：** 实现 Telemetry 4 指标本地聚合 + v2.0+ 上报 — 阻塞数据分析
- [ ] **P1：** 实现 Localization v1.0 中英 (85 字符串) — 阻塞本地化
- [ ] **P2：** v2.0+ 服务端 .NET 8 + PostgreSQL 16 + Redis 7 — 不阻塞 v1.0
- [ ] **P2：** Addressables 热更新集成 — 不阻塞 v1.0
- [ ] **P2：** Unity 6 LTS 升级评估 — 不阻塞 v1.0

## 13. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建**：8 文件（README / system-overview / module-breakdown / component-diagrams / data-flow / tech-stack / deployment / risks-and-decisions）。14 模块（M01-M14）+ 3 层 C4 模型 + 7 平台分发 + 7 项 ADR + Unity 2022 LTS + .NET 8 (v2.0+) + PostgreSQL 16 (v2.0+) + Redis 7 (v2.0+) + CloudFront CDN。P0-001 跟踪：Level.validate() + Balance.MaxDifficulty=20 自我保护，不修改 02-v2。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）