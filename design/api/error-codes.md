---
title: 《暗室》错误码表
doc_id: DESIGN-anzhong-api-error-codes
parent: design/api/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 策划总监
---

# 《暗室》错误码表 (Error Codes)

> **一句话定位：** 4xx/5xx 全量错误码 + 与 GDD 08 §7 反馈 3 层同步关联 + 客户端处理指南。

## 目的 (Purpose)

本文档是《暗室》API 错误码的**唯一权威列表**。它向：

- **客户端/Unity 工程师** — 给定 `errorCode` 即可定位 UI 提示/重试逻辑/降级策略
- **后端/服务端工程师** — 定义错误响应格式与命名规范
- **QA/测试** — 提供负面测试用例基准
- **国际化 (i18n)** — 每个 errorCode 对应一个 i18n 翻译 key

**本版本（v1.0）的目的：** 把 18 端点可能抛出的所有错误**统一编码**为 `<DOMAIN>_<REASON>` 格式，按 HTTP 4xx/5xx 分组，与 08 §7 反馈 3 层（视觉/音效/动效）显式关联，作为客户端错误处理的"字典"。

## 范围 (Scope)

### 包含

- **4xx 客户端错误** (35 错误码) — BAD_REQUEST / UNAUTHORIZED / FORBIDDEN / NOT_FOUND / CONFLICT / GONE / PAYLOAD_TOO_LARGE / RATE_LIMITED / UNPROCESSABLE
- **5xx 服务端错误** (8 错误码) — INTERNAL / NOT_IMPLEMENTED / BAD_GATEWAY / SERVICE_UNAVAILABLE / STORAGE_FULL / DATA_CORRUPTION
- **领域特定错误** (15 错误码) — SLOT_* / ROOM_* / CHAPTER_* / SAVE_* / SYNC_* / AUDIO_* / DIFFICULTY_* / RESET_ABUSE_THRESHOLD
- **错误响应格式** + **客户端处理指南** + **i18n 翻译 key 规范**

### 不包含 (Out of Scope)

- 业务异常堆栈追踪（服务端日志保留 30 天）
- 性能错误（OOM、timeout）→ 服务端监控告警
- 第三方平台错误（Steam OAuth 失败、PSN 不可用）→ 详见 `authentication.md` §4

## 1. 错误响应格式 (Error Response Schema)

> 与 `api-spec.yaml` `components.schemas.ErrorResponse` 一致。

```json
{
  "errorCode": "STRING_ERROR_CODE",
  "message": "Human readable message (i18n key)",
  "field": "fieldName",
  "retryAfterMs": 200,
  "currentValue": 25,
  "minValue": 1,
  "maxValue": 20,
  "limit": 60,
  "windowSec": 60,
  "currentCount": 31,
  "currentProgress": "3/5",
  "requiredProgress": "5/5",
  "dependentSlotId": "slot_1_1_b",
  "note": "Cardinal point detected"
}
```

**字段说明：**
- `errorCode` 必填，全大写 + 下划线，命名规范 `<DOMAIN>_<REASON>`
- `message` 必填，i18n key（如 `error.slot.cooldown`），客户端查表翻译
- 其他字段按错误类型可选出现

## 2. 命名规范 (Naming Convention)

| 段 | 含义 | 示例 |
|---|------|------|
| `BAD_*` | 通用 400 系列 | `BAD_REQUEST` |
| `UNAUTHORIZED` | 401 鉴权 | — |
| `FORBIDDEN` | 403 权限 | — |
| `NOT_FOUND` | 404 资源 | — |
| `CONFLICT` | 409 冲突 | — |
| `RATE_LIMIT` | 429 限流 | `RATE_LIMIT_EXCEEDED` |
| `STORAGE_*` | 507 存储 | `STORAGE_FULL` |
| `INTERNAL_*` | 500 服务端 | `INTERNAL_ERROR` |
| `SLOT_*` | 槽位领域 | `SLOT_COOLDOWN_ACTIVE` |
| `ROOM_*` | 房间领域 | `ROOM_LOCKED` |
| `CHAPTER_*` | 章节领域 | `CHAPTER_LOCKED` |
| `SAVE_*` | 存档领域 | `SAVE_NOT_FOUND` |
| `SYNC_*` | 同步领域 | `SYNC_CONFLICT` |
| `AUDIO_*` | 音频领域 | `AUDIO_DB_OUT_OF_RANGE` |
| `DIFFICULTY_*` | 难度领域 | `DIFFICULTY_OUT_OF_RANGE` |
| `RESET_*` | 重置领域 | `RESET_ABUSE_THRESHOLD` |
| `HINT_*` | Hint 领域 | `HINT_ALREADY_SHOWN` |
| `CLIENT_*` | 客户端版本 | `CLIENT_VERSION_TOO_OLD` |
| `TELEMETRY_*` | 遥测领域 | `TELEMETRY_BATCH_TOO_LARGE` |

## 3. 4xx 客户端错误 (Client Errors)

### 3.1 通用 4xx (15 错误码)

| errorCode | HTTP | 含义 | 客户端处理 | GDD 引用 |
|-----------|:----:|------|----------|---------|
| `BAD_REQUEST` | 400 | 请求体/参数错误 | 检查请求体字段，提示"请求格式错误" | — |
| `UNAUTHORIZED` | 401 | 未鉴权 / Token 失效 | 重定向到登录页 / 刷新 Token | authentication §2 |
| `FORBIDDEN` | 403 | 权限不足 | 提示"无权限" | — |
| `NOT_FOUND` | 404 | 资源不存在 | 提示"资源未找到"，回退到上一界面 | — |
| `CONFLICT` | 409 | 资源冲突 | 提示并刷新状态 | — |
| `GONE` | 410 | 资源已永久删除 | 提示并放弃 | — |
| `PAYLOAD_TOO_LARGE` | 413 | 请求体过大 | 拆分批量上报 | E08/E11/E16 |
| `UNPROCESSABLE_ENTITY` | 422 | 语义错误 | 检查字段语义 | — |
| `RATE_LIMIT_EXCEEDED` | 429 | 限流触发 | 等 `retryAfterMs` 后重试 | rate-limiting §1 |
| `CLIENT_VERSION_TOO_OLD` | 400 | 客户端版本过低 | 强制升级 | E01 |
| `CLIENT_TIME_DRIFT` | 401 | 客户端时钟漂移 | 校准客户端时钟 | E08 |
| `CROSS_ACCOUNT_SYNC_FORBIDDEN` | 403 | 跨账号同步 | 提示"账号不一致" | E15 |
| `DIFFICULTY_NOT_SUPPORTED` | 422 | 难度选项 v1.0 不支持 | 降级到 easy/normal | E14 |
| `INVALID_FEEDBACK_TYPE` | 400 | 反馈类型不在枚举 | 检查 feedbackType | E12 |
| `INVALID_EVENT_TYPE` | 400 | 事件类型不在枚举 | 检查 eventType | E16 |

### 3.2 槽位领域 (5 错误码)

> 与 02 §2.4 状态转换触发器 + 05 §3.2 切换参数严格对齐。

| errorCode | HTTP | 含义 | 客户端处理 | GDD 引用 |
|-----------|:----:|------|----------|---------|
| `SLOT_COOLDOWN_ACTIVE` | 400 | E/Q 切换冷却中 (300ms) | 显示"切换中…(200ms)"提示，等冷却 | 02 §3.2 + 05 §3.2 |
| `SLOT_LOCKED_DEPENDENCY` | 400 | ConditionalSlot 依赖不满足 | 提示"先切换 dependentSlotId" + 视觉指引 | 02 §3.1 |
| `SLOT_LOCKED_OBJECTIVE` | 400 | LockedSlot 目标未达成 | 提示"完成其他槽位后再来" | 02 §3.1 |
| `INVALID_SLOT_TYPE` | 400 | 槽位类型不支持该 direction | 不可能发生 → 500 升级 | 02 §3.1 |
| `INVALID_DEPENDENCY` | 500 | ConditionalSlot 依赖对象不存在 | 报告服务端 bug | 02 §3.1 |

### 3.3 房间/章节领域 (6 错误码)

| errorCode | HTTP | 含义 | 客户端处理 | GDD 引用 |
|-----------|:----:|------|----------|---------|
| `ROOM_LOCKED` | 403 | 房间未达成前置条件 | 提示"先完成前一房间" | 03 §5.2 |
| `ROOM_ALREADY_COMPLETED` | 409 | 房间已通关 | 幂等返回，不重复计分 | E05 |
| `CHAPTER_LOCKED` | 403 | 章节未解锁 | 显示 unlockCondition 提示 | 03 §5.2 + 04 §9.1 |
| `CHAPTER_COMPLETED` | 409 | 章节已完成 | 跳到下一章节 | 04 §9.3 |
| `GAME_COMPLETED` | 409 | 游戏已通关 | 跳到通关后内容 | 06 §11.3 |
| `INVALID_ROOM_TYPE` | 400 | 房间类型枚举错误 | 不可能发生 | 03 §4.1 |

### 3.4 存档领域 (5 错误码)

> 与 04 §10.3 读档语义 + §10.4 容错降级对齐。

| errorCode | HTTP | 含义 | 客户端处理 | GDD 引用 |
|-----------|:----:|------|----------|---------|
| `SAVE_NOT_FOUND` | 404 | 无存档（新玩家） | 启动 Ch1-1 | 04 §10.3 |
| `SAVE_VERSION_MISMATCH` | 409 | 存档版本与服务端不一致 | 提示升级客户端 | 04 §10.3 |
| `SAVE_VERSION_INCOMPATIBLE` | 409 | 存档版本过旧无法迁移 | 强制从 Ch1-1 开始 | 04 §10.3 |
| `SAVE_CORRUPTED` | 500 | 存档解析失败 | 自动加载 backup | 04 §10.3 + §10.4 |
| `SAVE_TOO_LARGE` | 413 | 单个存档 > 100KB | 提示玩家清理 | E08 |

### 3.5 同步/重置/音频/遥测 (10 错误码)

| errorCode | HTTP | 含义 | 客户端处理 | GDD 引用 |
|-----------|:----:|------|----------|---------|
| `SYNC_CONFLICT` | 409 | 跨设备同步冲突 | 接受 server_wins，本地提示 | E15 + 11 §1.1 |
| `SYNC_SOURCE_EQUALS_TARGET` | 400 | 源/目标平台相同 | 跳过同步 | E15 |
| `RESET_ABUSE_THRESHOLD` | 429 | 重置次数 > 30/房间 | 触发 02 §10.10 暗脉冲 | 05 §5.3 + 07 §3.3 |
| `AUDIO_DB_OUT_OF_RANGE` | 400 | dB 超出安全边界 | 拒绝整个请求 + 显示范围 | 05 §3.4 + 09 §1 |
| `AUDIO_MASTER_OUT_OF_RANGE` | 400 | masterVolume 越界 | 同上 | 05 §3.4 |
| `TELEMETRY_BATCH_TOO_LARGE` | 413 | 批量上报 > 500 条 | 拆分批量 | E16 |
| `PROGRESS_BATCH_TOO_LARGE` | 413 | 进度批量 > 100 条 | 拆分批量 | E11 |
| `FEEDBACK_RATE_LIMITED` | 429 | 反馈频次 > 30/天/玩家 | 提示"明天再来" | E12 |
| `HINT_ALREADY_SHOWN` | 409 | 该 Hint 已显示 | 忽略 | E17 |
| `ROOM_ALREADY_COMPLETED` | 409 | 房间已通关触发 Hint | 提示房间已通关 | E17 |

### 3.6 难度/章节门控 (2 错误码)

| errorCode | HTTP | 含义 | 客户端处理 | GDD 引用 |
|-----------|:----:|------|----------|---------|
| `DIFFICULTY_OUT_OF_RANGE` | 422 | difficulty > 20 | 服务端校验失败，500 升级 | 05 §5.2 / **P0-001** |
| `INVALID_CHAPTER_ID` | 400 | 章节 ID 不在枚举 | 提示并重试 | E18 |

## 4. 5xx 服务端错误 (Server Errors)

| errorCode | HTTP | 含义 | 客户端处理 | GDD 引用 |
|-----------|:----:|------|----------|---------|
| `INTERNAL_ERROR` | 500 | 通用服务端异常 | 提示"服务器繁忙，请重试" + 上报 | — |
| `NOT_IMPLEMENTED` | 501 | 端点未实现 | 提示"功能开发中" | versioning §Q-* |
| `BAD_GATEWAY` | 502 | 上游服务失败 | 同上 + 检查平台状态 | — |
| `SERVICE_UNAVAILABLE` | 503 | 服务维护中 | 显示维护画面 | — |
| `STORAGE_FULL` | 507 | 存档空间不足 | 提示"清理存档" | E08 |
| `DATA_CORRUPTION` | 500 | 数据损坏 | 加载 backup + 提示 | 04 §10.3 |
| `INVALID_DEPENDENCY` | 500 | 配置错误 | 报告服务端 bug | 02 §3.1 |
| `TOO_MANY_SLOTS` | 422 | 房间槽位 > 8 | 拒绝房间配置 | 02 §3.1 |

## 5. 与 GDD 08 §7 反馈 3 层同步关联 (3-Layer Feedback Sync)

> 客户端处理错误码时，**必须同步触发 3 层反馈**（视觉/音效/动效），与 08 §7 反馈设计一致。

| errorCode | 视觉层 | 音效层 | 动效层 | 体验意图 |
|-----------|--------|--------|--------|---------|
| `SLOT_COOLDOWN_ACTIVE` | 槽位暗淡脉冲 | `sfx_error_cooldown_01.wav` (-15dB) | 0.3s 暗淡动画 | "我现在不能切换" |
| `SLOT_LOCKED_DEPENDENCY` | dependentSlotId 槽位高亮 + 锁图标 | `sfx_switch_locked_01.wav` (-18dB) | 0.3s 抖动 | "我需要先做别的" |
| `RESET_ABUSE_THRESHOLD` | 屏幕轻微灰化 + 暗脉冲 3 次 | 无 | 0.5s 渐变 | "我该休息一下了" |
| `CHAPTER_LOCKED` | 章节按钮灰显 + 锁图标 | 无 | 0.3s 抖动 | "我还没准备好" |
| `AUDIO_DB_OUT_OF_RANGE` | 设置页面字段红色边框 | `sfx_error_validation_01.wav` (-15dB) | 0.2s 抖动 | "这个值不行" |
| `STORAGE_FULL` | 设置页面红色横幅 | `sfx_error_fatal_01.wav` (-9dB) | 0.5s 红闪 | "我需要清理空间" |
| `INTERNAL_ERROR` | 错误对话框 | `sfx_error_fatal_01.wav` | 0.5s 红闪 | "出问题了" |
| `RATE_LIMIT_EXCEEDED` | Toast 提示"请求过快" | 无 | 0.3s 淡入 | "我需要等一下" |
| `SAVE_NOT_FOUND` | 友好提示"开始新游戏" | 无 | 0.3s 淡入 | "欢迎新玩家" |
| `GAME_COMPLETED` | 通关动画 + 隐藏成就提示 | `sfx_win_game_01.wav` (-3dB) | 8s 通关尾声音乐 | "我完成了旅程" |

**GDD 对齐：** 10 错误码 × 3 层反馈 = 30 反馈项，与 08 §7 反馈 3 层同步 + 08 §7.2 反馈强度分级一致。

## 6. 客户端处理决策树 (Client Handling Decision Tree)

```
收到 ErrorResponse
    │
    ├── errorCode ∈ {4xx}
    │   │
    │   ├── UNAUTHORIZED (401)
    │   │   └── 刷新 Token → 重试
    │   │
    │   ├── RATE_LIMIT_EXCEEDED (429)
    │   │   └── 等 retryAfterMs → 提示 → 重试
    │   │
    │   ├── CHAPTER_LOCKED / ROOM_LOCKED (403)
    │   │   └── 显示 unlockCondition → 玩家手动解锁
    │   │
    │   ├── SLOT_COOLDOWN_ACTIVE (400)
    │   │   └── 客户端本地 300ms 冷却 → 不重试
    │   │
    │   ├── SLOT_LOCKED_DEPENDENCY (400)
    │   │   └── 高亮 dependentSlotId → 玩家手动操作
    │   │
    │   ├── SAVE_NOT_FOUND (404)
    │   │   └── 启动 Ch1-1（新玩家）
    │   │
    │   ├── SAVE_CORRUPTED (500)
    │   │   └── 加载 backup → 仍失败则从 Ch1-1 重新开始
    │   │
    │   └── 其他 4xx
    │       └── 显示 message → 不重试
    │
    └── errorCode ∈ {5xx}
        │
        ├── STORAGE_FULL (507)
        │   └── 提示清理 → 玩家手动操作
        │
        ├── DATA_CORRUPTION (500)
        │   └── 加载 backup → 仍失败则从 Ch1-1 重新开始
        │
        └── 其他 5xx
            └── 错误对话框 + 上报 → 玩家重试或退出
```

## 7. i18n 翻译 Key 规范 (i18n Keys)

> 每个 errorCode 对应一个 i18n key，客户端用 key 查翻译表（08 §9 本地化）。

| errorCode | i18n Key | 中文 (zh-CN) | 英文 (en-US) |
|-----------|----------|-------------|-------------|
| `BAD_REQUEST` | `error.bad_request` | 请求格式错误 | Invalid request |
| `UNAUTHORIZED` | `error.unauthorized` | 登录已失效，请重新登录 | Session expired |
| `RATE_LIMIT_EXCEEDED` | `error.rate_limit` | 请求过快，请稍后再试 | Too many requests |
| `SLOT_COOLDOWN_ACTIVE` | `error.slot.cooldown` | 切换中…(等待 {0}ms) | Switching... (wait {0}ms) |
| `SLOT_LOCKED_DEPENDENCY` | `error.slot.dependency` | 请先切换 {dependentSlotId} | Please switch {dependentSlotId} first |
| `CHAPTER_LOCKED` | `error.chapter.locked` | 完成 Ch1 后解锁 ({0}/5) | Complete Ch1 first ({0}/5) |
| `ROOM_LOCKED` | `error.room.locked` | 请先通关前一房间 | Complete previous room first |
| `SAVE_NOT_FOUND` | `error.save.not_found` | 未找到存档，开始新游戏 | No save found, starting new game |
| `SAVE_CORRUPTED` | `error.save.corrupted` | 存档损坏，已加载备份 | Save corrupted, loaded backup |
| `AUDIO_DB_OUT_OF_RANGE` | `error.audio.out_of_range` | 音量超出安全范围 | Volume out of safe range |
| `RESET_ABUSE_THRESHOLD` | `error.reset.abuse` | 重置次数过多，请休息或使用提示 | Too many resets, take a break or use a hint |
| `STORAGE_FULL` | `error.storage.full` | 存档空间已满 | Storage is full |
| `INTERNAL_ERROR` | `error.internal` | 服务器繁忙，请重试 | Server busy, please retry |
| `GAME_COMPLETED` | `error.game.completed` | 游戏已通关 | Game already completed |

> **完整 50 错误码 i18n 翻译表见** `data/i18n/error-codes.json`（实施时由国际化工程师创建）。

## 8. 错误码总统计 (Error Code Stats)

| 分类 | 错误码数 | 占比 |
|------|:--------:|:----:|
| 通用 4xx | 15 | 26% |
| 槽位领域 | 5 | 9% |
| 房间/章节领域 | 6 | 11% |
| 存档领域 | 5 | 9% |
| 同步/重置/音频/遥测 | 10 | 18% |
| 难度/章节门控 | 2 | 4% |
| 5xx 服务端 | 8 | 14% |
| **合计** | **51** | **100%** |

## 9. 关联文档 (Cross-References)

- [`api-spec.yaml`](./api-spec.yaml) — OpenAPI 3.0 机器可读
- [`endpoints.md`](./endpoints.md) — 端点详解
- [`data-models.md`](./data-models.md) — 数据模型
- [`authentication.md`](./authentication.md) — 鉴权相关错误
- [`rate-limiting.md`](./rate-limiting.md) — 限流错误
- [`versioning.md`](./versioning.md) — 版本相关错误
- [`../../docs/08-ui-ux-v2.md`](../../docs/08-ui-ux-v2.md) — 反馈 3 层 (08 §7)
- [`../../docs/04-gameplay-flow-v2.md`](../../docs/04-gameplay-flow-v2.md) — SaveSystem 容错 (04 §10.3)
- [`../../docs/05-numerical-design-v2.md`](../../docs/05-numerical-design-v2.md) — 难度上限 20 (05 §5.2) / P0-001

## 10. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** 51 错误码 (4xx: 43 + 5xx: 8) + 命名规范 + 客户端处理决策树 + 与 08 §7 反馈 3 层同步关联 (10 错误码 × 3 层 = 30 反馈项) + i18n 翻译 key 规范。 |
