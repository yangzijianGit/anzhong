---
title: 《暗室》API 端点详细说明
doc_id: DESIGN-anzhong-api-endpoints
parent: design/api/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 策划总监
---

# 《暗室》API 端点详细说明

> **一句话定位：** 18 端点逐个详解 (HTTP method / path / 请求 / 响应 / 错误码 / 示例) — 与 `api-spec.yaml` (OpenAPI 3.0) 一一对应。

## 目的 (Purpose)

本文档是 `api-spec.yaml` 的**人类可读伴生文档**。它向：

- **客户端/Unity 工程师** — 给出每端点的实际请求/响应示例
- **后端/服务端工程师** — 解释契约背后的设计意图 + 边界条件
- **QA/测试** — 提供 contract test 编写依据
- **策划/PM** — 理解每个端点对应 GDD 的哪个章节

**本版本（v1.0）的目的：** 把 18 端点从机器可读的 OpenAPI YAML 转为**带业务语义的说明**，每个端点含 5 段：用途 / HTTP 契约 / 请求示例 / 响应示例 / 边界与错误。

## 范围 (Scope)

### 包含

- 18 端点逐个详解（**E01-E18**）
- 每个端点：HTTP method / path / 请求 schema / 响应 schema / 业务示例 / 错误码 / 边界条件
- 端点 ↔ GDD 章节对照表
- 端点调用顺序示例（典型玩家旅程）

### 不包含 (Out of Scope)

- OpenAPI YAML 机器可读定义 → 见 `api-spec.yaml`
- 数据模型字段定义 → 见 `data-models.md`
- 错误码全量列表 → 见 `error-codes.md`
- 鉴权 Token 颁发流程 → 见 `authentication.md`
- 限流详细计算 → 见 `rate-limiting.md`

## 1. 端点分组 (Endpoint Groups)

> 18 端点按业务域分为 9 组。

| 组 | 端点数 | 端点 | 业务域 | GDD 主引用 |
|---|:---:|------|--------|----------|
| **G1 Session** | 2 | E01-E02 | 会话生命周期 | 04 §1.1 |
| **G2 Room** | 5 | E03-E05, E07, E18 | 房间进入/退出/通关/重置/章节列表 | 03 §5, 04 §4 |
| **G3 Slot** | 1 | E06 | SwitchSlot 切换 | 02 §2/§3.1 |
| **G4 Save** | 2 | E08-E09 | 存档保存/读取 | 04 §10 |
| **G5 Leaderboard** | 1 | E10 | 排行榜 | 05 §9.3, 06 §11.3 |
| **G6 Progress** | 1 | E11 | 进度上报 | 05 §4.1, 06 §9.2 |
| **G7 Feedback** | 1 | E12 | 玩家反馈 | 08 §7 |
| **G8 Settings** | 2 | E13, E14 | 音频/无障碍设置 | 09 + 06 §10 |
| **G9 Sync/Telemetry/Hint** | 3 | E15, E16, E17 | 跨平台同步/遥测/Hint | 11 §1.1, 05 §4.1, 06 §11.2.1 |

## 2. 端点逐个详解 (E01-E18)

> 完整 OpenAPI 定义见 `api-spec.yaml`。本文档给出业务语义、示例、边界。

---

### E01 — POST /sessions

| 字段 | 内容 |
|------|------|
| **用途** | 开始一次游戏会话（玩家启动游戏） |
| **HTTP** | `POST /api/v1/sessions` |
| **鉴权** | 公开（用 deviceId 即可） |
| **限流** | 10 req/h/device |
| **GDD** | 04 §1.1 全局状态机 MENU → CHAPTER_SELECT |

**请求体：**
```json
{
  "deviceId": "dev_a1b2c3d4e5f6",
  "platform": "steam",
  "clientVersion": "1.0.0",
  "locale": "zh-CN"
}
```

**响应 201：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "playerId": "player_x9y8z7",
  "startedAt": "2026-06-29T10:00:00Z",
  "platform": "steam",
  "clientVersion": "1.0.0",
  "state": "chapter_select"
}
```

**错误码：** `BAD_REQUEST` (400) / `RATE_LIMIT_EXCEEDED` (429)

**边界条件：**
- 同一 deviceId 已有活跃会话 → 复用旧 sessionId，不创建新会话
- 客户端版本低于 v1.0.0 → `CLIENT_VERSION_TOO_OLD` 提示升级
- platform 不在 7 平台枚举内 → 强制 `anonymous`

---

### E02 — POST /sessions/{id}/end

| 字段 | 内容 |
|------|------|
| **用途** | 结束游戏会话（玩家退出/崩溃/断电） |
| **HTTP** | `POST /api/v1/sessions/{id}/end` |
| **鉴权** | Bearer JWT |
| **限流** | 60 req/min |
| **GDD** | 04 §1.1 WIN/PAUSE → MENU |

**请求体：**
```json
{
  "exitReason": "user_quit",
  "finalState": "room_playing",
  "currentRoomId": "2-3"
}
```

**响应 200：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "endedAt": "2026-06-29T11:30:00Z",
  "durationSec": 5400,
  "finalRoomId": "2-3",
  "roomsCompleted": 8,
  "totalResets": 12
}
```

**错误码：** `UNAUTHORIZED` (401) / `NOT_FOUND` (404) / `SESSION_ALREADY_ENDED` (409)

**边界条件：**
- 玩家崩溃（`crash` / `power_off`）→ 服务端在 5 分钟心跳超时后自动调用
- `exitDuring=switching` 状态存档保留至 `room_entered` 入口（02 §2.4 边界）

---

### E03 — POST /rooms/{id}/enter

| 字段 | 内容 |
|------|------|
| **用途** | 进入房间，加载房间配置 + 槽位状态 |
| **HTTP** | `POST /api/v1/rooms/{id}/enter` |
| **鉴权** | Bearer JWT |
| **限流** | 60 req/min |
| **GDD** | 03 §5 19 房间配置 + 04 §4 房间内循环 |

**请求体：**
```json
{ "sessionId": "sess_20260629_abc123", "timestamp": 1719652800000 }
```

**响应 200：** 返回完整 `Room` 对象（含 slots + difficultyMax=20 + targetDurationP50/P90）。

**错误码：** `CHAPTER_LOCKED` (403) / `NOT_FOUND` (404) / `ROOM_LOCKED` (403)

**边界条件：**
- 房间 `difficulty > 20`（违反 05 §5.2 硬约束）→ 服务端 `Room.validate()` 拒绝
- ConditionalSlot 依赖的槽位不在房间内 → `INVALID_DEPENDENCY` 500
- 玩家已在房间内 → 直接返回当前状态（幂等）

---

### E04 — POST /rooms/{id}/exit

| 字段 | 内容 |
|------|------|
| **用途** | 离开房间（含 Switching 中退出） |
| **HTTP** | `POST /api/v1/rooms/{id}/exit` |
| **鉴权** | Bearer JWT |
| **限流** | 60 req/min |
| **GDD** | 04 §4.4 + 02 §2.4 |

**请求体：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "exitDuring": "switching",
  "progressSaved": true
}
```

**响应 200：**
```json
{
  "roomId": "1-1",
  "exitAt": "2026-06-29T10:05:00Z",
  "savePointId": "sp_ch1_1_1_partial",
  "resumeToken": "rtk_xyz789"
}
```

**错误码：** `NOT_FOUND` (404)

**边界条件：**
- `exitDuring=switching` → 状态不存档（02 §2.4 边界）
- `exitDuring=completed` → 已在 E05 中处理，此端点幂等返回 200

---

### E05 — POST /rooms/{id}/complete

| 字段 | 内容 |
|------|------|
| **用途** | 房间通关判定 |
| **HTTP** | `POST /api/v1/rooms/{id}/complete` |
| **鉴权** | Bearer JWT |
| **限流** | 60 req/min |
| **GDD** | 04 §4.3 通关判定 + 03 §6 难度奖励 + 06 §4 情感曲线 |

**请求体：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "completionTimeSec": 95,
  "switchCount": 3,
  "resetCount": 0,
  "hintsTriggered": 0
}
```

**响应 200：**
```json
{
  "roomId": "1-1",
  "chapterId": "ch1",
  "chapterComplete": false,
  "nextRoomId": "1-2",
  "score": { "steps": 3, "timeSec": 95, "hintsUsed": 0, "resetsUsed": 0 },
  "unlockProgress": 0.053,
  "achievementUnlocked": ["first_aha"]
}
```

**错误码：** `NOT_FOUND` (404) / `ALREADY_COMPLETED` (409)

**边界条件：**
- 玩家已通关该房间 → `ALREADY_COMPLETED` 409（不重复计分）
- `chapterComplete=true` → 触发 E18 chapter 列表更新
- 末间 3-8 → `gameCompleted=true` + 触发通关动画 + 6 隐藏成就检查

---

### E06 — POST /slots/{id}/switch

| 字段 | 内容 |
|------|------|
| **用途** | 切换 SwitchSlot (E/Q 键) |
| **HTTP** | `POST /api/v1/slots/{id}/switch` |
| **鉴权** | Bearer JWT |
| **限流** | **burst 10 req/s** + 60 req/min (05 §3.2 300ms 冷却) |
| **GDD** | 02 §2 状态机 + §3.1 4 槽位 + 05 §3.2 |

**请求体：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "direction": "clockwise",
  "clientTimestampMs": 1719652800000
}
```

**响应 200：**
```json
{
  "slotId": "slot_1_1_a",
  "previousIndex": 0,
  "currentIndex": 1,
  "newState": "hover",
  "animationMs": 200,
  "audioAsset": "sfx_switch_toggle_01.wav"
}
```

**错误码：**
- `SLOT_COOLDOWN_ACTIVE` (400) — 300ms 冷却未过
- `SLOT_LOCKED_DEPENDENCY` (400) — ConditionalSlot 依赖不满足
- `SLOT_LOCKED_OBJECTIVE` (400) — LockedSlot 目标未达成
- `INVALID_SLOT_TYPE` (400) — 槽位类型不支持该 direction

**边界条件：**
- ConditionalSlot 依赖对象不存在 → 500 `INVALID_DEPENDENCY`
- CycleSlot 顺时针到末尾 → currentIndex 循环回 0
- LockedSlot 玩家尝试切换 → 不改 currentIndex，返回 `LOCKED` 状态

---

### E07 — POST /rooms/{id}/reset

| 字段 | 内容 |
|------|------|
| **用途** | 重置房间（R 键） |
| **HTTP** | `POST /api/v1/rooms/{id}/reset` |
| **鉴权** | Bearer JWT |
| **限流** | 30 req/h/room（防滥用，05 §5.3）+ 500ms 客户端冷却 |
| **GDD** | 07 §3 槽位重置规则 + 02 §3.1 R 键 + 07 §3.3 ConditionalSlot |

**请求体：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "clientTimestampMs": 1719652800000
```

**响应 200：**
```json
{
  "roomId": "1-1",
  "resetCount": 2,
  "slotsAfterReset": [
    { "slotId": "slot_1_1_a", "currentIndex": 0, "state": "hover" }
  ],
  "audioAsset": "sfx_reset_room_01.wav"
}
```

**错误码：** `RESET_ABUSE_THRESHOLD` (429) — >30 次触发 02 §10.10 暗脉冲

**边界条件：**
- ConditionalSlot 依赖不满足时重置 → 重置后依赖对象也回初始，ConditionalSlot 自然回到 hover（07 §3.3）
- LockedSlot 重置 → 不影响 LS 锁定状态（07 §3.4）
- Switching 中重置 → 动画立即完成，所有槽位回初始

---

### E08 — POST /saves

| 字段 | 内容 |
|------|------|
| **用途** | 保存玩家进度（自动/手动） |
| **HTTP** | `POST /api/v1/saves` |
| **鉴权** | Bearer JWT |
| **限流** | 60 req/min |
| **GDD** | 04 §10.1-10.4 |

**请求体：** 完整 `SaveData` 对象（与 04 §10.2 TS interface 一一对应）。

**响应 200：**
```json
{
  "saveId": "save_20260629_103000_player_x9y8z7",
  "savedAt": "2026-06-29T10:30:00Z",
  "bytesWritten": 2847,
  "backupId": "save_20260629_103000_player_x9y8z7.bak"
}
```

**错误码：** `STORAGE_FULL` (507) / `SAVE_VERSION_MISMATCH` (409)

**边界条件：**
- 写入失败 → 保留旧存档 + 提示玩家（04 §10.4 容错）
- 客户端时钟漂移 > 5 分钟 → `CLIENT_TIME_DRIFT` 401 拒绝写入
- 单个存档 > 100KB → `SAVE_TOO_LARGE` 413

---

### E09 — GET /saves/{playerId}

| 字段 | 内容 |
|------|------|
| **用途** | 读取最近一次存档 |
| **HTTP** | `GET /api/v1/saves/{playerId}` |
| **鉴权** | Bearer JWT |
| **限流** | 60 req/min |
| **GDD** | 04 §10.3 读档语义 |

**查询参数：** `includeBackup` (bool, default=false)

**响应 200：** 完整 `SaveData` + `backupAvailable` + `backupTimestamp`

**错误码：** `SAVE_NOT_FOUND` (404) — 新玩家或存档损坏

**边界条件：**
- 主存档损坏 + backup 存在 → 自动加载 backup（04 §10.3 容错）
- 存档版本不兼容 → `SAVE_VERSION_INCOMPATIBLE` 409，提示从 Ch1-1 重新开始
- 多设备并发读取 → Last-Writer-Wins + 服务端时间戳优先

---

### E10 — GET /leaderboard

| 字段 | 内容 |
|------|------|
| **用途** | 查询通关排行榜 |
| **HTTP** | `GET /api/v1/leaderboard` |
| **鉴权** | Bearer JWT（读取个人排名） |
| **限流** | 60 req/min |
| **GDD** | 05 §9.3 隐藏成就 + 06 §11.3 M3 成就 |

**查询参数：**
- `chapterId` — ch1 / ch2 / ch3 (可选)
- `difficultyMax` — 1-20 (default 20, **P0-001 自我保护**)
- `sortBy` — steps / timeSec / score / completionDate (default steps)
- `limit` — 1-100 (default 20)

**响应 200：**
```json
{
  "chapterId": "ch1",
  "difficultyMax": 20,
  "sortBy": "steps",
  "entries": [
    {
      "rank": 1,
      "playerId": "player_aaa",
      "displayName": "Aha King",
      "score": { "steps": 8, "timeSec": 720, "hintsUsed": 0 },
      "completionDate": "2026-06-15T14:30:00Z"
    }
  ],
  "totalEntries": 1234
}
```

**边界条件：**
- 玩家未完成任何房间 → 排行榜不显示该玩家
- 通关作弊检测 → `LEADERBOARD_SUSPICIOUS_SCORE` 标记审核
- 难度上限筛选 (difficultyMax=12 对应 easy 档)

---

### E11 — POST /progress

| 字段 | 内容 |
|------|------|
| **用途** | 上报 4 核心指标 (P50/P90/ResetCount/HintTriggerRate) |
| **HTTP** | `POST /api/v1/progress` |
| **鉴权** | Bearer JWT |
| **限流** | 60 req/min（批量 100 条/次） |
| **GDD** | 05 §4.1 调参驱动 + 06 §9.2 |

**请求体：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "reports": [
    {
      "roomId": "1-1",
      "p50DurationSec": 95,
      "p90DurationSec": 180,
      "resetCount": 0,
      "hintsTriggered": 0,
      "switchCount": 3,
      "completed": true
    }
  ]
}
```

**响应 202：**
```json
{
  "accepted": 2,
  "rejected": 0,
  "flagged": [
    { "roomId": "1-2", "flag": "P90_EXCEEDS_P50_2X", "note": "Cardinal point detected" }
  ]
}
```

**边界条件：**
- `P90 > P50 × 2` → flagged 标记卡点房（05 §4.1）
- `ResetCount > F4 预测 × 2` → flagged 标记过难
- 批量 > 100 条 → `BATCH_TOO_LARGE` 413

---

### E12 — POST /feedback

| 字段 | 内容 |
|------|------|
| **用途** | 玩家提交反馈 |
| **HTTP** | `POST /api/v1/feedback` |
| **鉴权** | Bearer JWT |
| **限流** | 30 req/day/player |
| **GDD** | 08 §7 反馈 3 层（视觉/音效/动效）|

**请求体：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "roomId": "1-1",
  "feedbackType": "visual_layer",
  "rating": 4,
  "comment": "Slot glow is too dim on default",
  "clientTimestampMs": 1719652800000
}
```

**响应 201：**
```json
{
  "feedbackId": "fb_20260629_001",
  "recordedAt": "2026-06-29T10:30:00Z",
  "triageStatus": "queued"
}
```

**边界条件：**
- `feedbackType` 不在 8 枚举内 → `INVALID_FEEDBACK_TYPE` 400
- comment 长度 > 1000 字符 → 截断 + 提示

---

### E13 — GET/PUT /settings/audio

| 字段 | 内容 |
|------|------|
| **用途** | 读取/更新 9 类音频设置 |
| **HTTP** | `GET /api/v1/settings/audio` + `PUT /api/v1/settings/audio` |
| **鉴权** | Bearer JWT |
| **限流** | 30 req/min |
| **GDD** | 09 §1.1 9 类音频 + 05 §3.4 反馈参数 |

**GET 响应 200：** 完整 `AudioSettings` 对象。

**PUT 请求体：**
```json
{
  "masterVolume": 0.7,
  "switchSfxDb": -14,
  "resetSfxDb": -20,
  "winSfxDb": -6
}
```

**PUT 响应 200：** 更新后的 `AudioSettings`。

**错误码：** `AUDIO_DB_OUT_OF_RANGE` (400) — dB 超出 05 §3.4 安全边界

**边界条件：**
- 9 个 dB 字段全部有安全边界（如 switchSfxDb ∈ [-18, -6]）
- 任一字段越界 → 拒绝整个请求，不部分更新
- `muted=true` 时所有音量强制 0

---

### E14 — GET/PUT /settings/accessibility

| 字段 | 内容 |
|------|------|
| **用途** | 读取/更新无障碍设置 |
| **HTTP** | `GET /api/v1/settings/accessibility` + `PUT /api/v1/settings/accessibility` |
| **鉴权** | Bearer JWT |
| **限流** | 30 req/min |
| **GDD** | 06 §10 (4 类) + 08 §6 (5 开关) |

**GET 响应 200：** 完整 `AccessibilitySettings` 对象。

**PUT 请求体：**
```json
{
  "colorblindMode": "red_green",
  "fontScale": 1.25,
  "difficulty": "easy"
}
```

**边界条件：**
- `difficulty=hard` 在 v1.0 不支持 → 拒绝并提示
- `screenReader=true` 在 v1.0 不支持 → 拒绝并提示 v1.1

---

### E15 — POST /sync

| 字段 | 内容 |
|------|------|
| **用途** | 跨设备云存档同步 |
| **HTTP** | `POST /api/v1/sync` |
| **鉴权** | Bearer JWT |
| **限流** | 10 req/h/player |
| **GDD** | 11 §1.1 7 平台覆盖矩阵 |

**请求体：**
```json
{
  "playerId": "player_x9y8z7",
  "sourcePlatform": "steam",
  "targetPlatform": "switch",
  "saveData": { "version": "1.0.0", "currentChapterId": "ch2", "currentRoomId": "2-3" },
  "clientTimestampMs": 1719652800000
}
```

**响应 200：**
```json
{
  "conflictDetected": false,
  "resolvedSave": { "currentChapterId": "ch2", "currentRoomId": "2-3" },
  "lastSyncAt": "2026-06-29T10:30:00Z"
}
```

**响应 409（冲突）：** `SyncConflict` 对象，resolution=server_wins + lostProgress

**边界条件：**
- sourcePlatform == targetPlatform → 跳过同步，返回 200 + lastSyncAt
- 服务端存档不存在 → 用客户端存档作为初始云存档
- 跨账号同步 → `CROSS_ACCOUNT_SYNC_FORBIDDEN` 403

---

### E16 — POST /telemetry

| 字段 | 内容 |
|------|------|
| **用途** | 客户端遥测事件批量上报 |
| **HTTP** | `POST /api/v1/telemetry` |
| **鉴权** | Bearer JWT |
| **限流** | 120 req/min（批量 500 条/次） |
| **GDD** | 05 §4.1 4 指标 + 06 §11 |

**事件类型枚举：**
- `room_entered` / `room_completed` / `room_reset`
- `slot_switched`
- `hint_triggered`
- `achievement_unlocked`
- `error`
- `performance_metric`

**请求体：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "events": [
    { "eventType": "room_completed", "roomId": "1-1", "timestampMs": 1719652800000, "properties": { "durationSec": 95, "switchCount": 3 } }
  ]
}
```

**响应 202：**
```json
{ "received": 2, "deduplicated": 0, "serverTimestamp": "2026-06-29T10:30:00Z" }
```

**边界条件：**
- 批量 > 500 条 → `BATCH_TOO_LARGE` 413
- 同一 eventId 重复 → deduplicated++

---

### E17 — POST /hints

| 字段 | 内容 |
|------|------|
| **用途** | 触发渐进式 Hint |
| **HTTP** | `POST /api/v1/hints` |
| **鉴权** | Bearer JWT |
| **限流** | 6 req/h/player |
| **GDD** | 06 §11.2.1 + 03 §10 E1-E5 + 08 §8.3 |

**触发条件：**
- `idle_timeout` — 教学房 3 分钟空闲 / Boss 房 20 分钟
- `user_requested` — 玩家主动点击 Hint 按钮
- `boss_20min` — Boss 房 20 分钟强制
- `retry_threshold` — 重置次数 > 30

**请求体：**
```json
{
  "sessionId": "sess_20260629_abc123",
  "roomId": "1-1",
  "hintLevel": 1,
  "trigger": "idle_timeout",
  "idleDurationSec": 200
}
```

**响应 200：**
```json
{
  "hintId": "hint_1_1_l1",
  "level": 1,
  "text": "试试走近那个发光的方块，按 E 键",
  "visualHint": "slot_pulse_highlight",
  "audioAsset": "sfx_tutorial_firstswitch_01.wav",
  "expiresAt": "2026-06-29T10:35:00Z"
}
```

**边界条件：**
- 已通关房间 → `ROOM_ALREADY_COMPLETED` 409
- `hintLevel > 3` → 强制降为 3（最大提示等级）

---

### E18 — GET /chapters

| 字段 | 内容 |
|------|------|
| **用途** | 章节列表 + 解锁状态 |
| **HTTP** | `GET /api/v1/chapters` |
| **鉴权** | Bearer JWT |
| **限流** | 30 req/min |
| **GDD** | 03 §5.2 章节门控 + 04 §9.1 章节解锁 |

**响应 200：**
```json
{
  "chapters": [
    {
      "chapterId": "ch1", "name": "觉醒", "nameEn": "First Light",
      "roomCount": 5, "unlocked": true, "completed": false,
      "progressPct": 60.0, "averageDifficulty": 3.6
    },
    {
      "chapterId": "ch2", "name": "深掘", "nameEn": "Deep Dig",
      "roomCount": 6, "unlocked": false,
      "unlockCondition": "Complete Ch1 (5/5 rooms)",
      "completed": false, "progressPct": 0, "averageDifficulty": 9.3
    }
  ]
}
```

**边界条件：**
- 章节全部解锁（19/19 房间通关）→ gameCompleted=true
- 重玩章节（章节选择 → 1-1）→ `progressPct` 反映该章节重玩进度

---

## 3. 典型玩家旅程 — 端点调用顺序 (Sample Call Sequence)

> 一个玩家从启动到通关的典型 API 调用链。

```
T+0s    POST /sessions                     (E01) 创建会话
T+5s    GET  /chapters                    (E18) 加载章节列表
T+10s   POST /rooms/1-1/enter             (E03) 进入 1-1
T+15s   POST /slots/slot_1_1_a/switch     (E06) 切换 1 次
T+20s   POST /slots/slot_1_1_a/switch     (E06) 切换 2 次（路径打通）
T+110s  POST /rooms/1-1/complete          (E05) 通关 1-1
        POST /saves                       (E08) 自动存档
        POST /progress                    (E11) 上报 1-1 数据
T+115s  POST /rooms/1-2/enter             (E03) 进入 1-2
T+120s  POST /rooms/1-2/reset             (E07) 玩家 R 重置
T+150s  POST /slots/slot_1_2_a/switch     (E06) ...
... (省略 17 个房间)
T+7200s POST /rooms/3-8/complete          (E05) 通关 3-8
        POST /saves                       (E08) 存档
        POST /progress                    (E11) 上报 3-8 + 总进度
        GET  /leaderboard                 (E10) 查看通关排名
T+7205s POST /sessions/{id}/end           (E02) 结束会话
```

## 4. 端点 ↔ GDD 章节总对照 (Cross-Reference Master Table)

| 端点 | GDD 主引用 | GDD 次引用 |
|------|----------|----------|
| E01 session start | 04 §1.1 全局状态机 | 04 §2 主循环 |
| E02 session end | 04 §1.1 + §10.1 退出存档 | — |
| E03 room enter | 03 §5 19 房间配置 | 04 §4 房间内循环 |
| E04 room exit | 04 §4.4 衔接 | 02 §2.4 退出边界 |
| E05 room complete | 04 §4.3 通关判定 | 06 §4 情感曲线 +1 |
| E06 slot switch | 02 §2 状态机 + §3.1 4 槽位 | 05 §3.2 SwitchSlot 参数 |
| E07 room reset | 07 §3 重置规则 | 02 §3.1 R 键 |
| E08 save | 04 §10.1-10.4 SaveSystem | 05 §4.1 调参驱动 |
| E09 load save | 04 §10.3 读档语义 | 04 §10.4 容错降级 |
| E10 leaderboard | 05 §9.3 隐藏成就 | 06 §11.3 M3 成就动机 |
| E11 progress | 05 §4.1 4 指标 | 06 §9.2 玩家进度 |
| E12 feedback | 08 §7 反馈 3 层 | 06 §9.2 8 节点 |
| E13 audio settings | 09 §1.1 9 类 | 05 §3.4 dB 安全边界 |
| E14 accessibility | 06 §10 4 类 | 08 §6 5 开关 |
| E15 sync | 11 §1.1 7 平台 | 06 §11.3 社交分享 |
| E16 telemetry | 05 §4.1 4 指标 | 06 §11 沉浸/心理 |
| E17 hint | 06 §11.2.1 渐进式 | 03 §10 E1-E5 |
| E18 chapters | 03 §5.2 章节门控 | 04 §9.1 章节解锁 |

## 5. 关联文档 (Cross-References)

- [`api-spec.yaml`](./api-spec.yaml) — OpenAPI 3.0 机器可读规范
- [`data-models.md`](./data-models.md) — 12 数据模型详解
- [`error-codes.md`](./error-codes.md) — 错误码全量列表
- [`authentication.md`](./authentication.md) — 鉴权方案
- [`rate-limiting.md`](./rate-limiting.md) — 限流策略
- [`versioning.md`](./versioning.md) — 版本演进
- [`README.md`](./README.md) — 设计文档总览

## 6. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** 18 端点详解（E01-E18）+ 9 分组 + 典型调用序列 + GDD 总对照表。每个端点含用途/HTTP/鉴权/限流/GDD/请求/响应/错误/边界 9 字段。 |
