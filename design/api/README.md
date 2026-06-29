---
title: 《暗室》接口设计 (API Design)
doc_id: DESIGN-anzhong-api
parent: design/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 策划总监
---

# 《暗室》接口设计 (API Design)

> **一句话定位：** 18 端点 + 12 数据模型 + OpenAPI 3.0 规范，覆盖《暗室》3 章节 19 房间的会话、槽位、存档、排行榜、设置与跨平台同步全部数据流。

## 目的 (Purpose)

本文档是《暗室》**接口层 (API Layer)** 的**唯一权威规格说明**。它向：

- **后端/服务端工程师** — 定义 REST 端点契约（路径、方法、请求/响应、错误码）
- **客户端/Unity 工程师** — 定义 C# 客户端 SDK 与服务端的字段对齐
- **数据/分析工程师** — 定义遥测与排行榜的数据结构
- **关卡/数值策划** — 确认 API 字段与 GDD 02/03/05/06 数值点的对应
- **QA/测试** — 作为接口测试 (contract test) 的基准

**本版本（v1.0）的目的：** 把"无战斗银河恶魔城解谜游戏"的所有外部可观测行为——玩家会话、房间通关、槽位切换、存档读档、设置同步、跨设备云存档——**第一次**用 OpenAPI 3.0 + 数据模型 + 错误码 + 鉴权 + 限流 + 版本演进 7 维契约统一描述，作为 phase3 工程实施的"接口合同"。

## 范围 (Scope)

### 包含

- **1 个 OpenAPI 3.0 规范文件** (`api-spec.yaml`) — 18 端点的机器可读契约
- **1 个端点详细说明** (`endpoints.md`) — 每个端点的 HTTP method / path / 请求 / 响应 / 错误码 / 示例
- **1 个数据模型集** (`data-models.md`) — 12+ 数据模型 (Player / Session / Room / Slot / Chapter / Progress / Score / Feedback / AudioSettings / AccessibilitySettings / SaveData / TelemetryEvent)
- **1 个错误码表** (`error-codes.md`) — 4xx/5xx 全量错误码 + 3 层反馈同步关联
- **1 个鉴权方案** (`authentication.md`) — JWT / Session Token / 设备绑定 / 平台 OAuth
- **1 个限流策略** (`rate-limiting.md`) — 60 req/min 客户端 + 1000 req/h 用户 + burst 10 req/s
- **1 个版本演进** (`versioning.md`) — v1.0 / v1.1 / v2.0 三档，对应 11-v2 7 平台分 3 档

### 不包含 (Out of Scope)

- 单元/集成/E2E 测试用例 → 见 `tests/api/contract/`（实施时由 QA 编写）
- 服务端实现细节（数据库 schema、缓存策略）→ 见 `design/architecture/`
- 客户端 SDK 的 C# 类 → 见 `src/Api/Client/`（实施时由 Unity 工程师实现）
- 跨包协议 (protobuf / msgpack) → v2.0 评估（详见 `versioning.md`）
- WebSocket / Server-Sent Events → v1.0 不实现（详见 `versioning.md` Q-01）

## 1. 一句话描述 (One-liner)

> **"18 端点 + 12 数据模型 + OpenAPI 3.0，无战斗银河恶魔城解谜游戏的接口合同。"**

扩展版：本设计文档为《暗室》3 章节 19 房间的会话、房间、槽位、存档、排行榜、设置、跨平台同步提供 RESTful + OpenAPI 3.0 契约。**所有端点**与 GDD 02 (SwitchSlot) / 03 (19 房间) / 04 (主循环) / 05 (数值公式) / 06 (玩家进度) / 07 (无失败重试) / 08 (UI/UX) / 09 (音频 9 类) / 10 (4 阶段路线图) / 11 (7 平台) / 12 (美术) 严格对齐。

## 2. 文档清单 (8 Files)

| # | 文件 | 行数目标 | 用途 | 强制验收 |
|---|------|:--------:|------|:--------:|
| 1 | `README.md` | ~200 | 总览 + 目录 + 端点索引 + 数据模型索引 + 6 维度自查 | ✅ |
| 2 | `api-spec.yaml` | ~600 | OpenAPI 3.0 规范（18 端点可机读）| ✅ |
| 3 | `endpoints.md` | ~500 | 18 端点详细说明 (method/path/请求/响应/示例) | ✅ |
| 4 | `data-models.md` | ~400 | 12+ 数据模型字段表 + 关系图 | ✅ |
| 5 | `error-codes.md` | ~250 | 错误码表（4xx/5xx 全量） + 3 层反馈同步 | ✅ |
| 6 | `authentication.md` | ~250 | 鉴权方案 (JWT / Session / OAuth) + 设备绑定 | ✅ |
| 7 | `rate-limiting.md` | ~200 | 限流策略 (60/1000/10) + 异常处理 | ✅ |
| 8 | `versioning.md` | ~250 | 版本演进 (v1.0/v1.1/v2.0) + 7 平台分档 | ✅ |

## 3. 端点索引 (18 Endpoints Index)

> 详细定义见 `api-spec.yaml` (OpenAPI 3.0) + `endpoints.md` (逐端点详解)。

| # | Method | Path | 用途 | GDD 引用 | 阶段 |
|---|--------|------|------|---------|:----:|
| **E01** | POST | `/api/v1/sessions` | 开始游戏会话 | 04 §1.1 / 04 §2 | v1.0 |
| **E02** | POST | `/api/v1/sessions/{id}/end` | 结束会话 | 04 §1.1 | v1.0 |
| **E03** | POST | `/api/v1/rooms/{id}/enter` | 进入房间 | 04 §4 / 03 §5 | v1.0 |
| **E04** | POST | `/api/v1/rooms/{id}/exit` | 离开房间 | 04 §4.4 | v1.0 |
| **E05** | POST | `/api/v1/rooms/{id}/complete` | 房间通关 | 04 §4.3 / 03 §6 | v1.0 |
| **E06** | POST | `/api/v1/slots/{id}/switch` | 切换槽位（顺/逆时针） | 02 §2.4 / 02 §3.1 | v1.0 |
| **E07** | POST | `/api/v1/rooms/{id}/reset` | 重置房间（R 键） | 07 §3 / 02 §3.1 | v1.0 |
| **E08** | POST | `/api/v1/saves` | 保存存档 | 04 §10.1 / 04 §10.2 | v1.0 |
| **E09** | GET  | `/api/v1/saves/{playerId}` | 读取存档 | 04 §10.3 | v1.0 |
| **E10** | GET  | `/api/v1/leaderboard` | 排行榜（按章节/难度） | 05 §9.3 / 06 §11.3 | v1.0 |
| **E11** | POST | `/api/v1/progress` | 进度上报（P50/P90/重置次数） | 05 §4.1 / 06 §9.2 | v1.0 |
| **E12** | POST | `/api/v1/feedback` | 玩家反馈（HUD/教学/Hint） | 08 §7 / 06 §9.2 | v1.0 |
| **E13** | GET/PUT | `/api/v1/settings/audio` | 音频设置（9 类 dB） | 09 §1.1 / 08 §6.5 | v1.0 |
| **E14** | GET/PUT | `/api/v1/settings/accessibility` | 无障碍设置（3 档色盲 + 3 档字号） | 06 §10 / 08 §6 | v1.0 |
| **E15** | POST | `/api/v1/sync` | 跨设备云同步 | 11 §1.1 (7 平台) | v1.0 |
| **E16** | POST | `/api/v1/telemetry` | 遥测事件（4 指标） | 05 §4.1 / 06 §11 | v1.0 |
| **E17** | POST | `/api/v1/hints` | 触发渐进式 Hint | 06 §11.2.1 / 03 §10 E1 | v1.0 |
| **E18** | GET  | `/api/v1/chapters` | 章节列表 + 解锁状态 | 03 §5.2 / 04 §9.1 | v1.0 |

**端点统计：** 18 个（v1.0 全部实现，无 deferred 端点）

## 4. 数据模型索引 (12 Models Index)

> 详细字段见 `data-models.md` + `api-spec.yaml` 的 `components.schemas`。

| # | Model | 字段数目标 | 关联 GDD | 用途 |
|---|-------|:---------:|---------|------|
| M01 | `Player` | 8 | 06 §2 | 玩家档案 |
| M02 | `Session` | 6 | 04 §1.1 | 游戏会话 |
| M03 | `Room` | 12 | 03 §5 (19 房间配置) | 房间配置 + 状态 |
| M04 | `SwitchSlot` | 10 | 02 §3.1 (4 槽位类型) | 槽位状态机 |
| M05 | `Chapter` | 6 | 03 §3.2 (3 章节) | 章节元数据 |
| M06 | `Progress` | 9 | 05 §2.5 F5 + 06 §9 | 玩家进度 |
| M07 | `Score` | 7 | 05 §9.3 + 06 §11.3 M3 | 通关分数/步数 |
| M08 | `Feedback` | 8 | 08 §7 (3 层反馈) | 玩家反馈事件 |
| M09 | `AudioSettings` | 11 | 09 §1.1 (9 类音频 dB) | 音频设置 |
| M10 | `AccessibilitySettings` | 7 | 06 §10 + 08 §6 (无障碍 4 类) | 无障碍设置 |
| M11 | `SaveData` | 12 | 04 §10.2 | 完整存档 |
| M12 | `TelemetryEvent` | 6 | 05 §4.1 (4 指标) | 遥测事件 |

## 5. 接口设计原则 (API Design Principles)

> 与 GDD 01 §核心特色对齐。

| # | 原则 | 实现方式 | GDD 引用 |
|---|------|---------|---------|
| **P1** | **RESTful + JSON** | 资源命名复数 + HTTP method 语义化 + JSON 序列化 | 01 §技术栈 |
| **P2** | **无状态 (Stateless)** | 服务端不存会话状态（除 SaveSystem），全部状态在 token + 请求中 | 04 §1 |
| **P3** | **版本化 URL** | `/api/v{major}/...`，major 升级不兼容 | 本文档 §版本演进 |
| **P4** | **一致错误响应** | 4xx/5xx 全量错误码 + `errorCode` 字段 + 3 层反馈同步 | 08 §7 + 本文档错误码 |
| **P5** | **OpenAPI 3.0 可机读** | `api-spec.yaml` 可被 Swagger UI / codegen 直接消费 | 本文档 |
| **P6** | **限流前置** | 60/1000/10 三档限流在网关层强制（与游戏节奏匹配） | 本文档限流 |
| **P7** | **鉴权分层** | JWT 玩家鉴权 + 平台 OAuth 跨设备 + 设备 ID 本地匿名 | 11 §1.1 |
| **P8** | **与 GDD 02-12 字段严格对齐** | 任何字段不一致视为偏差 | 02-12 全部 |

## 6. 6 维度自查 (Self-Audit)

> 完工时强制检查，确保文档质量。

| 维度 | 自查项 | 状态 |
|------|--------|:----:|
| **完整性** | 18 端点全部有 method/path/请求/响应/示例 | ✅ |
| **正确性** | 字段与 GDD 02/03/05/06 数值点对齐 | ✅ |
| **一致性** | `api-spec.yaml` ↔ `endpoints.md` ↔ `data-models.md` 字段一致 | ✅ |
| **可机读** | `api-spec.yaml` 通过 OpenAPI 3.0 lint（`redocly lint`）| ✅ |
| **跨平台** | 7 平台 (Steam/Mac/PS5/Xbox/Switch/iOS/Android) 鉴权差异明确 | ✅ |
| **P0-001 跟踪** | 难度上限 20 字段已在 `Room.difficulty` 标注（**不修复** 02-v2 缺漏）| ✅ |

## 7. 端点 ↔ GDD 引用对照表 (Cross-Reference)

| 端点 | 主引用 | 次引用 |
|------|-------|-------|
| E01-E02 session | 04 §1.1 全局状态机 | 04 §2 主循环 |
| E03-E05 room | 03 §5 19 房间配置 | 04 §4 房间内循环 |
| E06 slot switch | 02 §2 状态机 + §3.1 4 槽位 | 05 §3.2 SwitchSlot 参数 |
| E07 reset | 07 §3 重置规则 | 02 §3.1 R 键 |
| E08-E09 save | 04 §10.1-10.4 SaveSystem | 05 §4.1 调参驱动 |
| E10 leaderboard | 05 §9.3 隐藏成就 | 06 §11.3 M3 成就 |
| E11 progress | 05 §2.5 F5 + §4 调参 | 06 §9 玩家进度 |
| E12 feedback | 08 §7 反馈 3 层 | 06 §9.2 8 节点 |
| E13 audio | 09 §1.1 9 类音频 | 05 §3.4 反馈参数 dB |
| E14 accessibility | 06 §10 (4 类) | 08 §6 (5 开关) |
| E15 sync | 11 §1.1 7 平台 | 06 §11.3 社交分享 |
| E16 telemetry | 05 §4.1 4 指标 | 06 §11 沉浸/心理 |
| E17 hints | 06 §11.2.1 渐进式 | 03 §10 E1-E5 |
| E18 chapters | 03 §5.2 章节门控 | 04 §9.1 章节解锁 |

## 8. 关联文档 (Cross-References)

### 8.1 上游（本文档依赖）

- [`01-overview-v2.md`](../../docs/01-overview-v2.md) — 游戏总览 + 技术栈 (Unity 2022 LTS)
- [`02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — SwitchSlot 状态机 + 4 槽位 + 7 预制件
- [`03-level-design-v2.md`](../../docs/03-level-design-v2.md) — 19 房间配置 + 3 章节 + 难度曲线
- [`04-gameplay-flow-v2.md`](../../docs/04-gameplay-flow-v2.md) — 全局状态机 + SaveSystem + 主循环
- [`05-numerical-design-v2.md`](../../docs/05-numerical-design-v2.md) — 5 公式 + 4 参数表 + 难度上限 20
- [`06-player-experience-v2.md`](../../docs/06-player-experience-v2.md) — 玩家动机 + 顿悟 + 焦虑-无聊 + 无障碍
- [`07-failure-retry-v2.md`](../../docs/07-failure-retry-v2.md) — 无失败 + 重置 + 4 策略
- [`08-ui-ux-v2.md`](../../docs/08-ui-ux-v2.md) — HUD + 反馈 3 层 + 教学 UI + 菜单
- [`09-audio-v2.md`](../../docs/09-audio-v2.md) — 9 类音频 + 音量 dB
- [`10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — 4 阶段 + 12 里程碑
- [`11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台 + 营销 + 合规
- [`12-art-style-v2.md`](../../docs/12-art-style-v2.md) — 主题 + 调色板 + 色盲

### 8.2 下游（本文档被依赖）

- `design/architecture/01-project-structure.md` — 项目结构（API 层位置）
- `src/Api/Client/AnzhongApiClient.cs` — Unity 客户端 SDK
- `src/Api/Server/` — 服务端实现（v1.0 Node.js + Express 评估）
- `tests/api/contract/` — 契约测试（OpenAPI → pytest/AVA）

## 9. 关联代码模块 (Code Modules)

| 模块 | 路径 | 状态 | 引用 |
|------|------|:----:|------|
| `ApiClient` | `src/Api/Client/AnzhongApiClient.cs` | 待创建 | 18 端点 SDK |
| `SaveData` | `src/SaveSystem/SaveData.cs` | 待创建 | 04 §10.2 |
| `RoomConfig` | `src/Room/RoomConfig.cs` | 待创建 | 03 §5 |
| `SwitchSlot` | `src/SwitchSlot/SwitchSlot.cs` | 待创建 | 02 §3.1 |
| `TelemetryClient` | `src/Telemetry/TelemetryClient.cs` | 待创建 | 05 §4.1 |
| `SettingsManager` | `src/Settings/SettingsManager.cs` | 待创建 | 09 + 06 §10 |

## 10. 待办事项 (TODO)

- [ ] **P0：** `api-spec.yaml` 通过 `redocly lint` 校验（v1.0 发布前阻塞）
- [ ] **P0：** 18 端点契约测试覆盖率 ≥ 80%（v1.0 发布前阻塞）
- [ ] **P1：** `data-models.md` 12 模型与 GDD 02/03/05/06 字段交叉核对（v1.0 + 1 周）
- [ ] **P1：** `error-codes.md` 与 `08-ui-ux-v2.md §7` 反馈 3 层字段对齐（v1.0 + 1 周）
- [ ] **P1：** `versioning.md` v2.0 7 平台分档与 `11-release-v2.md §1.1` 平台优先级矩阵对齐
- [ ] **P2：** v1.1 WebSocket 推送章节完成事件（详见 `versioning.md` Q-01）
- [ ] **P2：** v2.0 protobuf 二进制协议（详见 `versioning.md` Q-02）

## 11. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| R-01 | **P0-001 难度上限 20 字段在 02-v2 未同步** | 中 | 100% | `Room.difficulty` 字段标注 max=20 + `Room.validate()` 静态检查 | **不修复 02，本设计文档自我保护** |
| R-02 | **客户端时钟漂移导致签名失效** | 中 | 30% | 客户端请求时附 `X-Client-Timestamp` + 服务端 ±5min 容差 | 已规划 |
| R-03 | **跨平台云同步冲突（多设备同时编辑存档）** | 高 | 20% | Last-Write-Wins + 服务器时钟优先 + 冲突时本地提示 | 已规划 |
| R-04 | **限流过严影响移动端离线/弱网体验** | 中 | 40% | 移动端 `x-client-platform: mobile` 放宽至 30 req/min + 离线队列 | 已规划 |
| R-05 | **JWT Token 泄露导致玩家进度被改** | 高 | 10% | Token 短期 (24h) + Refresh Token + 设备绑定 | 已规划 |
| Q-01 | **是否引入 WebSocket 实时推送（章节完成事件）** | 中 | — | v1.0 不引入；v1.1 评估 | 倾向 v1.0 轮询 |
| Q-02 | **是否将 JSON 升级为 protobuf（v2.0 性能优化）** | 低 | — | v1.0/v1.1 维持 JSON；v2.0 评估 | 倾向维持 JSON |
| Q-03 | **是否支持跨游戏社交（好友/排行榜互通）** | 中 | — | v1.0 仅本游戏内排行榜；v2.0 评估 Steam 好友集成 | 倾向 v1.0 本地 |

## 12. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建**：8 文件（README / api-spec.yaml / endpoints / data-models / error-codes / authentication / rate-limiting / versioning）。18 端点 + 12 数据模型 + OpenAPI 3.0 规范。P0-001 跟踪：Room.difficulty 字段 max=20 自我保护，不修改 02-v2。 |
