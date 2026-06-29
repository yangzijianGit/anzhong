---
title: 《暗室》API 版本演进
doc_id: DESIGN-anzhong-api-versioning
parent: design/api/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 策划总监
---

# 《暗室》API 版本演进 (Versioning)

> **一句话定位：** v1.0 (Steam/Mac 2 平台) → v1.1 (Switch 1 平台) → v2.0 (全 7 平台) + URL 路径版本化 + 向后兼容策略。

## 目的 (Purpose)

本文档是《暗室》API 版本演进路线的**唯一权威规格**。它向：

- **后端/服务端工程师** — 定义 URL 版本化方案 + 兼容性策略
- **Unity 客户端工程师** — 定义客户端版本 ↔ API 版本映射
- **运维** — 定义多版本并行部署 + 流量切分
- **策划/PM** — 理解 v1.0/v1.1/v2.0 与 7 平台分档的对应

**本版本（v1.0）的目的：** 把"无战斗银河恶魔城解谜游戏"的 API **第一次**显式分为 3 档版本（v1.0 / v1.1 / v2.0），与 11 §1.1 7 平台分 3 档（P0/P1/P2）+ 10 §1 4 阶段（Pre-production / Alpha / Beta / RC+Release）严格对齐，**URL 路径版本化**（`/api/v{major}/...`）保证客户端可锁定 API 版本。

## 范围 (Scope)

### 包含

- **3 档版本规划** (v1.0 / v1.1 / v2.0) + 与 7 平台对应
- **URL 路径版本化** + 客户端 ↔ API 版本映射
- **向后兼容策略** (SemVer 兼容矩阵)
- **API 弃用流程** (Deprecation Policy)
- **存档迁移** (SaveData Versioning)
- **新端点/新字段引入规则**
- **v1.0 范围** (18 端点 + 12 模型) 详细清单

### 不包含 (Out of Scope)

- 服务端实现细节 (蓝绿部署 / 金丝雀) → 实施时由运维设计
- 数据库 schema migration → 实施时由后端工程师设计
- 客户端强制升级策略 → `11-release-v2.md` §3.4

## 1. 一句话演进路径 (One-liner Roadmap)

> **"v1.0 (2 平台) → v1.1 (3 平台 + Switch) → v2.0 (7 平台 + WebSocket)"**

| 版本 | 阶段 | 上线时间 | 平台数 | 新增端点 | 新增字段 | 弃用端点 | 里程碑 |
|------|------|---------|:------:|---------|---------|---------|--------|
| **v1.0** | RC + Release | W12 (M11/M12) | 2 (Steam/Mac) | 18 | 0 | 0 | Day-84 |
| **v1.1** | T+3m Post-Release | T+3m | 3 (+ Switch) | +2 (WebSocket) | +5 | 0 | v1.1 Release |
| **v2.0** | T+6m Full Platform | T+6m | 7 (全平台) | +5 (Social/Proto) | +15 | 0 | v2.0 Release |

## 2. 3 档版本详解 (3 Versions Detail)

### 2.1 v1.0 — Day-84 上线 (RC + Release)

> 与 10 §1 阶段 3 + 11 §1.3 v1.0 P0 平台严格对齐。

**上线时间：** Day-84 (W12 里程碑)
**平台：** Steam (Win/Mac) + Itch.io 试玩版
**端点数：** 18 (E01-E18)
**模型数：** 12 (M01-M12)
**鉴权：** L1 匿名 (DeviceId) + L2 JWT (Steam/Mac OAuth)
**限流：** 7 档 (L1-L7)

**v1.0 全部端点清单：**

| 端点 | 用途 | 阶段 | 阻塞 |
|------|------|:----:|------|
| E01 sessions | 会话启动 | v1.0 | — |
| E02 sessions end | 会话结束 | v1.0 | — |
| E03 rooms enter | 进入房间 | v1.0 | — |
| E04 rooms exit | 离开房间 | v1.0 | — |
| E05 rooms complete | 房间通关 | v1.0 | — |
| E06 slots switch | 切换槽位 | v1.0 | — |
| E07 rooms reset | 重置房间 | v1.0 | — |
| E08 saves | 保存存档 | v1.0 | — |
| E09 saves GET | 读取存档 | v1.0 | — |
| E10 leaderboard | 排行榜 | v1.0 | — |
| E11 progress | 进度上报 | v1.0 | — |
| E12 feedback | 玩家反馈 | v1.0 | — |
| E13 audio settings | 音频设置 | v1.0 | — |
| E14 accessibility settings | 无障碍设置 | v1.0 | — |
| E15 sync | 跨平台同步 | v1.0 (Steam ↔ Mac) | — |
| E16 telemetry | 遥测事件 | v1.0 | — |
| E17 hints | 触发 Hint | v1.0 | — |
| E18 chapters | 章节列表 | v1.0 | — |

### 2.2 v1.1 — T+3m 上线 (Post-Release 扩展)

> 与 10 §1 阶段 3 + 11 §1.3 v1.1 P1 平台对齐。

**上线时间：** T+3m (W16-W18, 推迟 1 月)
**平台：** + Nintendo Switch (1 平台)
**端点数：** 18 + 2 (E19-E20) = 20
**新增模型：** 1 (M13 WebSocketMessage)
**鉴权：** + L3 Nintendo Account OAuth
**限流：** + 1 档 (L8 WebSocket 推送 100/h)

**v1.1 新增端点：**

| 端点 | 用途 | 与 v1.0 关系 |
|------|------|------------|
| **E19 WebSocket /ws/rooms/{id}** | 房间状态实时推送 (服务端 → 客户端) | 全新 |
| **E20 GET /leaderboard/friends** | 好友排行榜 (Steam 好友集成) | 全新 |

**v1.1 新增字段 (5)：**
- `Room.webrtcSupport` (bool) — Switch 联机支持
- `Player.nintendoFriendCode` (string) — Switch 好友码
- `TelemetryEvent.controllerType` (string) — Joy-Con / Pro Controller
- `AudioSettings.hapticEnabled` (bool) — Switch HD 振动
- `AccessibilitySettings.colorFilter` (enum) — Switch 屏幕色弱滤镜

**v1.1 文档更新：**
- `api-spec.yaml` → 升级为 `api-spec-v1.1.yaml` (保留 v1.0)
- `endpoints.md` → 增加 E19-E20 章节
- `data-models.md` → 增加 M13 章节
- `versioning.md` → 本文档 v1.1 增量更新

### 2.3 v2.0 — T+6m 上线 (Full Platform)

> 与 10 §1 阶段 3 + 11 §1.3 v2.0 P2 平台对齐。

**上线时间：** T+6m
**平台：** + PS5 + Xbox Series X|S + iOS + Android (4 平台) = 全 7 平台
**端点数：** 20 + 5 (E21-E25) = 25
**新增模型：** 3 (M14 Friend / M15 ChatMessage / M16 Achievement)
**鉴权：** + L3 PSN/Xbox Live/Sign in with Apple/Google Sign-In
**限流：** + 2 档 (L9 Chat 30/min + L10 Achievement 5/min)

**v2.0 新增端点：**

| 端点 | 用途 | 与 v1.1 关系 |
|------|------|------------|
| **E21 GET /friends** | 好友列表 (Steam/PSN/Xbox) | 全新 |
| **E22 POST /friends/{id}/invite** | 邀请好友 | 全新 |
| **E23 GET /chat/{friendId}** | 聊天消息 (限本游戏) | 全新 |
| **E24 GET /achievements** | 成就列表 (6 隐藏 + 6 公开) | 升级 (从 E10 拆分) |
| **E25 POST /achievements/{id}/share** | 分享成就 | 全新 |

**v2.0 新增字段 (15)：**
- `Player.psnId` / `Player.xboxGamertag` / `Player.appleId` (string × 3)
- `Friend.relationship` (enum: friend/pending/blocked) (3 字段)
- `ChatMessage.from` / `content` / `timestampMs` (3 字段)
- `Achievement.id` / `name` / `description` / `unlockedAt` (4 字段)
- `TelemetryEvent.platformLatencyMs` (int) — 主机平台特有
- `SyncRequest.encrypted` (bool) — 主机平台 E2E 加密

**v2.0 协议升级：**
- 评估 protobuf 二进制协议 (Q-02)
- 评估 GraphQL 查询接口 (Q-04)

## 3. URL 路径版本化 (URL Path Versioning)

> 与 11 §3.4 全球同步 vs 分批 + 04 §10.1 自动存档时机对齐。

### 3.1 版本化方案

```
https://api.anzhong.example/v1/sessions         ← v1.0
https://api.anzhong.example/v1.1/sessions       ← v1.1
https://api.anzhong.example/v2/sessions         ← v2.0
```

**规则：**
- URL 路径段 `v{major}[.{minor}]` 表示 API 主版本
- v1.0 → v1.1 是次版本，**保证向后兼容**
- v1.x → v2.0 是主版本，**可能不兼容**
- 客户端请求时**锁定**到 `v{x}`（如 `v1.0`）
- 服务端**同时维护** `v1.0` 和 `v1.1`（v1.0 已弃用时仅维护 6 个月）

### 3.2 客户端 ↔ API 版本映射

| 客户端版本 | API 版本 | 关系 |
|----------|---------|------|
| v1.0.x (Day-84) | v1.0 | 完全一致 |
| v1.1.x (T+3m) | v1.1 | 完全一致 + v1.0 兼容 |
| v2.0.x (T+6m) | v2.0 | 完全一致 + v1.1 兼容 |
| v1.0 客户端访问 v2.0 API | — | 拒绝 (`CLIENT_VERSION_TOO_OLD`) |

### 3.3 服务端多版本部署

```
[Cloudflare / Nginx]
    ↓
[API Gateway]
    ├── /v1/*   → v1.0 Service (Kubernetes Deployment v1.0)
    ├── /v1.1/* → v1.1 Service (Kubernetes Deployment v1.1)
    └── /v2/*   → v2.0 Service (Kubernetes Deployment v2.0)
    ↓
[Shared Database + Per-version Schema]
```

**关键点：**
- v1.0 服务**至少维护**到 v1.1 发布后 6 个月（即 T+9m）
- v1.0 → v1.1 数据迁移由客户端触发（首次启动 v1.1 客户端时）
- v1.x → v2.0 数据迁移由服务端后台任务（v2.0 上线时批量迁移）

## 4. 向后兼容策略 (Backward Compatibility)

> 与 11 §3.4 全球同步 + 04 §10.4 存档版本对齐。

### 4.1 兼容性矩阵 (SemVer 风格)

| 变更类型 | 兼容？ | 必须升 major？ | 必须升 minor？ |
|---------|:------:|:--------------:|:--------------:|
| **新增端点** | ✅ | ❌ | ✅ |
| **新增字段 (nullable)** | ✅ | ❌ | ✅ |
| **新增可选枚举值** | ✅ | ❌ | ✅ |
| **新增必填字段** | ❌ | ✅ | ❌ |
| **删除端点** | ❌ | ✅ | ❌ |
| **删除字段** | ❌ | ✅ | ❌ |
| **修改字段类型** | ❌ | ✅ | ❌ |
| **修改字段语义** | ❌ | ✅ | ❌ |
| **修改必填 → 可选** | ✅ | ❌ | ✅ |
| **修改可选 → 必填** | ❌ | ✅ | ❌ |
| **错误码新增** | ✅ | ❌ | ✅ |
| **错误码删除** | ❌ | ✅ | ❌ |

### 4.2 兼容性示例

**v1.0 → v1.1 兼容变更：**
```diff
# E13 audio settings
  AudioSettings:
    properties:
      masterVolume: ...      # v1.0
      switchSfxDb: ...       # v1.0
      resetSfxDb: ...        # v1.0
      winSfxDb: ...          # v1.0
+     hapticEnabled:         # v1.1 新增 (bool, optional)
+       type: boolean
+       default: false
```

✅ v1.0 客户端访问 v1.1 服务 → `hapticEnabled` 字段缺失，使用默认值，**完全兼容**。
✅ v1.1 客户端访问 v1.0 服务 → `hapticEnabled` 字段被服务端忽略，**完全兼容**。

**v1.1 → v2.0 不兼容变更：**
```diff
# M11 SaveData
  SaveData:
    required:
      - version
      - lastUpdated
+     - platformAccountId   # v2.0 新增必填
```

❌ v1.x 客户端访问 v2.0 服务 → `platformAccountId` 缺失，返回 400。
❌ v2.0 客户端访问 v1.x 服务 → `platformAccountId` 被服务端忽略，OK。

## 5. API 弃用流程 (Deprecation Policy)

> 与 11 §3.4 全球同步 + 04 §10.3 存档版本对齐。

### 5.1 弃用阶段

```
Phase 1: 标记弃用 (Deprecation Announced)
    ↓ 6 个月
Phase 2: 软弃用 (Soft Removal)
    ↓ 3 个月
Phase 3: 硬弃用 (Hard Removal)
    ↓
Phase 4: 完全删除 (Deletion)
```

### 5.2 弃用示例

**Day-X：v1.0 端点标记弃用**
```
[Response Headers]
Sunset: Sat, 01 Jan 2028 00:00:00 GMT
Deprecation: true
Link: </v2/sessions>; rel="successor-version"

[Response Body]
{
  "errorCode": "API_DEPRECATED",
  "message": "v1.0 /sessions is deprecated. Please migrate to v2/sessions.",
  "sunsetDate": "2028-01-01",
  "successorEndpoint": "/v2/sessions"
}
```

**Day-X+6m：v1.0 端点软弃用**
- 仍可用但**返回警告**
- 客户端显示"即将弃用，请升级"
- 服务端监控 v1.0 调用量

**Day-X+9m：v1.0 端点硬弃用**
- 返回 410 Gone
- 客户端必须升级

**Day-X+12m：v1.0 端点完全删除**
- 服务端代码删除
- 404 Not Found

## 6. 存档迁移 (SaveData Versioning)

> 与 04 §10.3 读档语义 + §10.4 容错降级对齐。

### 6.1 存档版本字段

```json
{
  "version": "1.0.0",     ← SaveData 版本
  ...
}
```

### 6.2 迁移矩阵

| 存档版本 | 客户端版本 | 行为 |
|---------|----------|------|
| 1.0.0 | v1.0.x | 直接读取 |
| 1.0.0 | v1.1.x | 读取 + 注入 v1.1 新增默认值 |
| 1.0.0 | v2.0.x | 读取 + 触发迁移脚本到 2.0.0 |
| 1.1.0 | v1.0.x | 降级读取 (忽略 v1.1 字段) |
| 1.1.0 | v1.1.x | 直接读取 |
| 1.1.0 | v2.0.x | 读取 + 触发迁移脚本到 2.0.0 |
| 2.0.0 | v1.0.x | ❌ 拒绝 (`SAVE_VERSION_INCOMPATIBLE`) |
| 2.0.0 | v1.1.x | ❌ 拒绝 (`SAVE_VERSION_INCOMPATIBLE`) |
| 2.0.0 | v2.0.x | 直接读取 |

### 6.3 迁移脚本

> 实施时由后端工程师在 `src/Api/Server/Migrations/` 创建。

```
v1.0.0 → v1.1.0:
  - 添加 hapticEnabled (default: false) 到 audioSettings
  - 添加 platform 字段到 playerStats

v1.1.0 → v2.0.0:
  - 添加 platformAccountId (必填) 
  - 拆分 roomStats 为 19 个独立字段
  - 添加 friendList 数组
  - 加密敏感字段 (email/phone)
```

## 7. 新端点/新字段引入规则 (Introduction Rules)

### 7.1 新端点引入

1. 在 OpenAPI 3.0 规范中添加路径
2. 更新 `endpoints.md` 详解
3. 升级 **minor 版本** (v1.0 → v1.1)
4. 不影响现有客户端
5. 在变更日志中记录

### 7.2 新字段引入 (兼容)

1. 在 `data-models.md` 中添加字段
2. 字段类型必须是**可选的** (nullable 或有默认值)
3. 升级 **minor 版本**
4. 现有客户端忽略新字段，行为不变
5. 在变更日志中记录

### 7.3 字段必填化 (不兼容)

1. **必须**升级 major 版本 (v1.x → v2.0)
2. 提供 6 个月过渡期，旧版本客户端仍可运行
3. 过渡期内服务端**双向支持** (旧字段可选 + 新字段必填)
4. 过渡期结束硬弃用旧版本

### 7.4 字段弃用

1. 在 OpenAPI 3.0 规范中标记 `deprecated: true`
2. 添加 `X-Deprecation-Date` 响应头
3. **6 个月后** 软弃用 (返回警告)
4. **9 个月后** 硬弃用 (返回 410)
5. **12 个月后** 完全删除

## 8. v1.0 范围总览 (v1.0 Scope Summary)

> 与 18 端点 + 12 模型 + 4 阶段路线图 + 7 平台分档对齐。

| 维度 | v1.0 范围 |
|------|----------|
| **端点数** | 18 (E01-E18) |
| **模型数** | 12 (M01-M12) |
| **错误码** | 51 (4xx: 43 + 5xx: 8) |
| **限流档** | 7 (L1-L7) |
| **鉴权层** | 3 (L1 匿名 / L2 JWT / L3 OAuth) |
| **平台** | 2 (Steam / Mac) |
| **语言** | 2 (zh-CN / en-US) |
| **SaveData 版本** | 1.0.0 |
| **OpenAPI 版本** | 3.0.3 |
| **阻塞依赖** | P0-001 02-v2 §13 AC-06 同步"难度上限 20" (本设计自我保护) |

## 9. 关联文档 (Cross-References)

- [`api-spec.yaml`](./api-spec.yaml) — v1.0 OpenAPI 3.0 机器可读
- [`endpoints.md`](./endpoints.md) — v1.0 端点详解
- [`data-models.md`](./data-models.md) — v1.0 数据模型
- [`error-codes.md`](./error-codes.md) — v1.0 错误码
- [`authentication.md`](./authentication.md) — v1.0 鉴权
- [`rate-limiting.md`](./rate-limiting.md) — v1.0 限流
- [`../../docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — 4 阶段 12 里程碑
- [`../../docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台分 3 档
- [`../../docs/04-gameplay-flow-v2.md`](../../docs/04-gameplay-flow-v2.md) — SaveSystem 存档版本

## 10. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| R-01 | **v1.0 → v2.0 数据迁移失败** | 高 | 20% | 双写 + 后台异步迁移 + 客户端手动触发 | 已规划 |
| R-02 | **多版本服务端资源消耗** | 中 | 40% | v1.0 服务 6 个月后下线 | 已规划 |
| R-03 | **客户端锁定 v1.0 后无法使用 v1.1 新功能** | 中 | 30% | 客户端提示升级 + 强制升级策略 | 已规划 |
| R-04 | **弃用流程未充分公告** | 中 | 30% | 6 个月公告期 + 邮件/Steam 通知 | 已规划 |
| R-05 | **API 破坏性变更未识别** | 高 | 20% | SemVer 强制 + CI 测试 | 已规划 |
| Q-01 | **是否引入 WebSocket 实时推送（v1.1 候选）** | 中 | — | v1.1 已规划 E19 WebSocket | 已规划 |
| Q-02 | **是否将 JSON 升级为 protobuf（v2.0 候选）** | 低 | — | v1.0/v1.1 维持 JSON；v2.0 评估 | 倾向维持 |
| Q-03 | **是否引入 GraphQL 查询接口** | 中 | — | v2.0 评估；v1.0/v1.1 维持 REST | 倾向 v2.0 |
| Q-04 | **是否支持离线 API（本地优先）** | 中 | — | v1.0 客户端缓存 + v1.1 评估 | 倾向 v1.1 |

## 11. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** 3 档版本规划 (v1.0/v1.1/v2.0) + URL 路径版本化 + 向后兼容矩阵 (12 变更类型) + 弃用流程 (4 阶段) + 存档迁移 + 18 端点 v1.0 清单 + 5 风险 4 开放问题。**P0-001 跟踪：** v1.0 阻塞依赖标注，不阻塞设计文档发布。 |
