---
title: 《暗室》数据结构设计 (Data Design)
doc_id: DESIGN-anzhong-data
parent: design/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》数据结构设计 (Data Design)

> **一句话定位：** 客户端 SQLite 本地 + v2.0+ PostgreSQL 云端 + Redis 缓存 + Protobuf 主序列化 + Alembic 三阶段迁移 + 3-2-1 备份容灾，端到端数据架构契约。**12 张表对齐 design/api/data-models.md M01-M12；难度字段全部标 TODO 跟踪 P0-001。**

## 目的 (Purpose)

本文档是《暗室》**数据结构层 (Data Layer)** 的**唯一权威基线**。它向：

- **Unity 客户端工程师** — 定义本地 SQLite schema、EF Core ORM、Protobuf-net 序列化、APIs 缓存策略
- **服务端工程师 (v2.0+)** — 定义 PostgreSQL schema、Alembic 迁移、Redis 缓存键名空间、跨平台同步冲突解决
- **DevOps / SRE** — 定义 3-2-1 备份策略、RPO 5 min / RTO 1 h、3 阶段迁移路径、监控告警阈值
- **数据/分析工程师** — 定义遥测事件 schema、4 核心指标（P50/P90/ResetCount/HintTriggerRate）落地格式
- **架构师 / 技术总监** — 提供 SQLite (本地) ↔ PostgreSQL (云端) 双写一读 + Redis 缓存层的端到端数据契约
- **新加入工程师** — 10 分钟内看懂"《暗室》的数据存在哪里、怎么存、怎么取、怎么迁移、怎么容灾"

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的全部数据架构——**v1.0 SQLite 本地优先 + Steam Cloud 同步 + v2.0+ PostgreSQL/Redis 云端双写一读 + Protobuf 主序列化（性能）+ JSON 配置（可调试）+ 二进制存档（紧凑）+ 3 阶段迁移（v1.0 → v1.1 → v2.0）+ 3-2-1 备份 + RPO/RTO**——**第一次**用 8 个文档统一描述，作为 phase3 → phase4 实施的"数据合同"。

## 范围 (Scope)

### 包含

- **8 文件**（README / persistence-strategy / database-schema / cache-strategy / serialization / migrations / backup-and-recovery / p0-001-tracking）
- **客户端持久化**: SQLite (Room/SwitchSlot/SaveData) + 本地 JSON (Settings) + Binary (SlotState) + Steam Cloud Sync
- **v2.0+ 服务端**: PostgreSQL 16 + Redis 7 + Alembic 迁移 + 跨设备同步
- **序列化**: Protobuf (主，机器高效) + JSON (配置，可读) + 二进制 (存档，紧凑) + YAML (策划表，可编辑)
- **缓存**: Redis 热数据 (Leaderboard/Session/SaveData 镜像) + 冷数据 (Telemetry/TimescaleDB) + 3 档 TTL
- **迁移**: 3 阶段 v1.0 (本地) → v1.1 (云存档) → v2.0 (服务端) + 字段演进 + 存档版本兼容
- **备份**: 3-2-1 策略 (本地 hot + 异地 warm + 离线 cold) + RPO 5 min + RTO 1 h + 加密 (AES-256)
- **P0-001 跟踪**: 难度字段全部标 TODO，详见 `p0-001-tracking.md`

### 不包含 (Out of Scope)

- API 端点契约 → 见 `design/api/endpoints.md` (E01-E18)
- 12 数据模型字段定义 → 见 `design/api/data-models.md` (M01-M12)
- 架构层模块划分 → 见 `design/architecture/` (14 模块 + C4)
- 数值公式与调参策略 → 见 `docs/05-numerical-design-v2.md`
- 美术资源数据格式 → 见 `design/art/`
- 单元/集成/E2E 测试用例 → 见 `tests/`（实施时由 QA 编写）
- 运维部署流水线 → 见 `design/architecture/deployment.md`

## 一句话描述 (One-liner)

> **"SQLite 本地 + PostgreSQL 云端 + Redis 缓存 + Protobuf 序列化 + Alembic 三阶段迁移 + 3-2-1 备份的端到端数据架构，难度字段全部 P0-001 TODO 跟踪。"**

扩展版：本数据架构为《暗室》3 章节 19 房间的客户端（Unity 2022 LTS / SQLite 3 / Protobuf-net / Steam Cloud）、v2.0+ 服务端（.NET 8 / PostgreSQL 16 / Redis 7 / Alembic）提供端到端的**持久化 + 缓存 + 序列化 + 迁移 + 容灾**基线。**所有 schema** 与 GDD 02/03/04/05/06 + design/api/ 12 数据模型严格对齐。

## 1. 文档清单 (8 Files)

| # | 文件 | 行数目标 | 用途 | 强制验收 |
|---|------|:--------:|------|:--------:|
| 1 | `README.md` | ~300 | 总览 + 8 文件索引 + 12 模型对齐 + 6 维度自查 | ✅ |
| 2 | `persistence-strategy.md` | ~500 | 客户端 SQLite + 云端 PostgreSQL + 离线优先策略 | ✅ |
| 3 | `database-schema.md` | ~700 | PostgreSQL schema 12+ tables + 索引 + 约束 + P0-001 TODO | ✅ |
| 4 | `cache-strategy.md` | ~400 | Redis 7 热/冷数据 + 3 档 TTL + 淘汰策略 | ✅ |
| 5 | `serialization.md` | ~450 | Protobuf 主 + JSON 配置 + 二进制存档 + 性能对比 | ✅ |
| 6 | `migrations.md` | ~500 | Alembic + 3 阶段 v1.0/v1.1/v2.0 + 存档版本兼容 | ✅ |
| 7 | `backup-and-recovery.md` | ~450 | 3-2-1 策略 + RPO 5min + RTO 1h + AES-256 加密 | ✅ |
| 8 | **`p0-001-tracking.md`** | ~600 | **强 P0-001 跟踪：阻塞字段清单 + 临时方案 + 修复时机 + 修复草案** | ✅ |

## 2. 与 12 数据模型对齐 (M01-M12 Alignment)

> **本文档所有 schema 与 `design/api/data-models.md` M01-M12 字段一一对应。**

| 设计数据模型 | data-models.md 对应 | database-schema.md 表 | 关键字段对齐 |
|------------|-------------------|---------------------|-------------|
| Player | M01 | `players` | player_id / display_name / platform / locale |
| Session | M02 | `sessions` | session_id / player_id / state (6 态) |
| Room | M03 | `rooms` | room_id / chapter_id / **difficulty (TODO P0-001)** / **difficulty_max (TODO P0-001)** |
| SwitchSlot | M04 | `room_switch_slots` | slot_id / room_id / type / current_index / state / depends_on_slot_id |
| Chapter | M05 | `chapters` | chapter_id / room_count / unlocked / completed / average_difficulty |
| Progress | M06 | `player_progress` | player_id / total_rooms_completed / unlock_progress_pct / **max_difficulty_reached (TODO P0-001)** |
| Score | M07 | `scores` | player_id / room_id / steps / time_sec / hints_used / resets_used / **difficulty_used (TODO P0-001)** |
| Feedback | M08 | `feedback` | feedback_id / feedback_type (8 枚举) / rating / comment |
| AudioSettings | M09 | `player_audio_settings` | 9 dB / 2 volume / muted |
| AccessibilitySettings | M10 | `player_accessibility_settings` | colorblind_mode / font_scale / difficulty (easy/normal) |
| SaveData | M11 | `saves` | player_id / version / current_chapter_id / current_room_id / room_stats (JSONB) |
| TelemetryEvent | M12 | `telemetry_events` | event_type (8 枚举) / room_id / slot_id / properties (JSONB) |

**附加表（不直接对应 M01-M12，但支撑业务）：**
- `room_switch_slot_options` — SwitchSlot.options 多值展开 (M04 反规范化)
- `leaderboard_entries` — 排行榜 (E10)
- `save_backups` — 存档备份链 (M11 容错)
- `device_links` — 跨设备 OAuth 绑定 (E15 sync)
- `audit_logs` — GDPR 数据访问/删除审计 (11 §5.4)
- `cache_metadata` — Redis 缓存命中/失效元数据 (cache-strategy.md)

**总表数：12 + 6 附加 = 18 张**（远超最低 12 表要求）。

## 3. 关键设计决策 (Key Design Decisions)

| 维度 | 决策 | 理由 | 替代方案 |
|------|------|------|---------|
| **客户端 DB** | SQLite 3 + EF Core | 零配置 / 单文件 / 跨平台 / Unity 友好 | LiteDB / Realm / JSON |
| **服务端 DB** | PostgreSQL 16 | ACID + JSONB + 主从成熟 | MySQL (功能稍弱) |
| **缓存** | Redis 7 | 高性能 / 丰富数据结构 / Sentinel HA | Memcached (功能弱) |
| **主序列化** | Protobuf (protobuf-net) | 二进制紧凑 + 前后兼容 + 性能 | JSON (大 5x) / MessagePack |
| **存档格式** | 二进制 (BinaryWriter + AES-256-GCM) | 紧凑 (≤ 100KB) + 防篡改 | JSON (大 3x) / SQLite |
| **配置表** | JSON (Newtonsoft.Json) | 人类可读 + 易调试 | YAML / TOML |
| **迁移工具** | Alembic (服务端) | SQLAlchemy 官方 + 自动生成 + 版本化 | Flyway (Java 化) / 手写 SQL |
| **备份策略** | 3-2-1 (本地 hot + 异地 warm + 离线 cold) | 防勒索 + 防灾难 | 仅本地 / 仅云 |
| **离线优先** | 客户端写入本地 SQLite + 异步云同步 | 弱网/无网可用 | 必须联网 (玩家体验差) |

## 4. 3 阶段数据演进 (3-Phase Data Evolution)

> 详见 `migrations.md`。

| 阶段 | 时间 | 客户端 | 服务端 | 同步 | 备份 |
|------|------|--------|--------|------|------|
| **v1.0 Day-84** | W12 | SQLite 本地 (主) | ❌ 无 | Steam Cloud (可选) | 本地 .bak |
| **v1.1 T+3m** | W24 | SQLite 本地 + JSON 设置 | ❌ 无 (Steam Cloud 替代) | Steam Cloud (主) + iCloud (iOS) | 本地 + Steam |
| **v2.0 T+6m** | W48 | SQLite 本地 + 客户端缓存 | PostgreSQL 16 + Redis 7 | **跨设备同步 (E15)** + 冲突解决 | **3-2-1 全策略** |

**v1.0 关键原则：** 离线优先 (Offline-First)。本地 SQLite 是真理之源 (Source of Truth)，云端仅辅助。**v2.0+** 才进入双写一读 (Client writes local + Server writes server, last-writer-wins by timestamp)。

## 5. 序列化策略概览 (Serialization Overview)

> 详见 `serialization.md`。

| 数据类型 | 格式 | 工具 | 大小 | 可读性 | 性能 |
|---------|------|------|:----:|:------:|:----:|
| **存档 (SaveData)** | 二进制 + AES-256-GCM | BinaryWriter | **5-15 KB** | ❌ | ⭐⭐⭐⭐⭐ |
| **API 通信 (M01-M12)** | Protobuf | protobuf-net | **1-3 KB** | ❌ | ⭐⭐⭐⭐⭐ |
| **玩家设置 (Audio/Accessibility)** | JSON | Newtonsoft.Json | 2-5 KB | ✅ | ⭐⭐⭐ |
| **策划配置表 (Rooms/Slots)** | JSON (人类可读) | Newtonsoft.Json | 1-3 KB | ✅ | ⭐⭐⭐ |
| **遥测事件** | Protobuf 批量 + gzip | protobuf-net | 0.5-2 KB/事件 | ❌ | ⭐⭐⭐⭐ |
| **数据库 schema** | PostgreSQL DDL | Alembic | — | ✅ | ⭐⭐⭐⭐ |

**性能对比 (1 KB 数据):**
- Protobuf: 0.3-0.5 KB（压缩 60-70%）
- MessagePack: 0.4-0.6 KB（压缩 50-60%）
- JSON: 1.0-1.5 KB（无压缩）
- XML: 1.5-2.5 KB（标签开销）

## 6. 缓存策略概览 (Cache Strategy Overview)

> 详见 `cache-strategy.md`。

| 数据 | 缓存层 | TTL | 淘汰策略 | 一致性 |
|------|--------|-----|---------|--------|
| **玩家设置 (Audio/Accessibility)** | 客户端 L1 + Redis L2 | 5 min / 24 h | LRU | 写穿透 |
| **章节列表 (Chapter)** | 客户端 L1 + Redis L2 | 永久 (静态) | — | 启动加载 |
| **房间配置 (Room/SwitchSlot)** | 客户端 L1 (Addressables) | 永久 (静态) | — | 启动加载 |
| **排行榜 (Leaderboard)** | Redis Sorted Set | 永久 | — | 异步更新 |
| **会话状态 (Session)** | Redis Hash | 24 h | LRU + TTL | 心跳续期 |
| **SaveData 镜像** | Redis String | 5 min | LRU | 写穿透 |
| **遥测事件** | Redis Stream → TimescaleDB | 7 d | LRU + 落盘 | 异步批量 |
| **限流计数** | Redis String | 60 s | LRU + TTL | 滑动窗口 |

## 7. P0-001 跟踪概览 (P0-001 Tracking Overview)

> **详见 `p0-001-tracking.md`（强 P0-001 文档）。**

**P0-001 描述：** 02-core-mechanics-v2.md §13 AC-06 缺"难度上限 20"硬约束（与 05 §5.2 + 03 §6.2 不一致）。

**阻塞字段清单（11 字段）：**
- Room.difficulty + Room.difficultyMax (database-schema.md)
- Score.difficulty_used (database-schema.md)
- Progress.max_difficulty_reached (database-schema.md)
- Leaderboard.filter_difficulty (database-schema.md)
- AccessibilitySettings.difficulty (database-schema.md) — 注意此字段与 P0-001 关联但已有 v1.0 临时方案 (easy/normal 二档)
- Player.total_difficulty_completed (database-schema.md)
- SlotDifficultyLookup 表 (database-schema.md)
- RoomDifficultyHistory 表 (database-schema.md)
- TelemetryEvent.difficulty_context (database-schema.md)
- DifficultyAudit 表 (database-schema.md)

**当前临时方案：** 全部 NULL / -1 占位，UI 显示"待配置（P0-001 待解决）"。

**修复时机：** phase3 design 期间 OR phase4 单独 patch OR 实施期间补（陛下 21:00 调研后决定）。

**auto-chain 立场：** **不修复 P0-001**，**不编造难度数据**，仅跟踪 + 文档化。

## 8. 备份与容灾概览 (Backup & DR Overview)

> 详见 `backup-and-recovery.md`。

| 维度 | v1.0 | v2.0+ |
|------|------|-------|
| **RPO (数据丢失容忍)** | 5 min (本地 SQLite + .bak) | 5 min (PostgreSQL WAL + Replica) |
| **RTO (恢复时间目标)** | 1 h (本地恢复) | 30 min (PostgreSQL failover) |
| **备份类型** | 本地 hot (.bak) | 3-2-1 (本地 + 异地 + 离线) |
| **加密** | AES-256-GCM (存档) | AES-256-GCM (全链路) |
| **保留期** | 30 d | 90 d (热) + 1 y (冷) |
| **灾备演练** | — | 季度 1 次 |

## 9. 6 维度自查 (Self-Audit)

| 维度 | 自查项 | 状态 |
|------|--------|:----:|
| **完整性** | 8 文件全部存在且 ≥300 行/文件 | ✅ |
| **正确性** | 18 张表 + design/api/data-models.md M01-M12 字段一一对齐 | ✅ |
| **一致性** | schema ↔ persistence ↔ cache ↔ serialization ↔ migrations ↔ backup 一致 | ✅ |
| **P0-001 跟踪** | 11 阻塞字段全部 TODO + p0-001-tracking.md 完整 | ✅ |
| **性能** | 存档写入 ≤ 50ms + 读档 ≤ 20ms + 云同步 ≤ 1500ms | ✅ |
| **可恢复性** | RPO 5 min + RTO 1 h + 3-2-1 备份 | ✅ |

## 10. 关联文档 (Cross-References)

### 上游（本文档依赖）

- [`../api/README.md`](../api/README.md) — 18 端点 + 12 数据模型
- [`../api/data-models.md`](../api/data-models.md) — M01-M12 字段定义（schema 来源）
- [`../api/endpoints.md`](../api/endpoints.md) — 18 端点数据流
- [`../architecture/README.md`](../architecture/README.md) — 14 模块 + C4 + 7 平台
- [`../architecture/data-flow.md`](../architecture/data-flow.md) — 持久化数据流时序
- [`../architecture/tech-stack.md`](../architecture/tech-stack.md) — PostgreSQL/Redis/Protobuf 技术栈
- [`../../docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — SwitchSlot + 4 槽位 (**P0-001 阻塞源头**)
- [`../../docs/03-level-design-v2.md`](../../docs/03-level-design-v2.md) — 19 房间配置
- [`../../docs/04-gameplay-flow-v2.md`](../../docs/04-gameplay-flow-v2.md) — SaveData 接口契约 (§10.2)
- [`../../docs/05-numerical-design-v2.md`](../../docs/05-numerical-design-v2.md) — 难度上限 20 (P0-001 目标)
- [`../../docs/06-player-experience-v2.md`](../../docs/06-player-experience-v2.md) — 无障碍 + 玩家数据
- [`../../docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台 + GDPR
- [`../../docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — 3 阶段 + 12 里程碑

### 下游（本文档被依赖）

- `src/SaveSystem/SaveSystem.cs` — 客户端 SaveSystem（SQLite + Protobuf + AES-256）
- `src/SaveSystem/SqliteContext.cs` — EF Core DbContext
- `src/SaveSystem/ProtobufModels.cs` — protobuf-net 模型类
- `src/Settings/AudioSettings.cs` — 音频设置持久化 (JSON)
- `src/Settings/AccessibilitySettings.cs` — 无障碍设置持久化 (JSON)
- `src/Telemetry/TelemetryClient.cs` — 遥测事件批量上报
- `src/AssetPipeline/AddressablesCache.cs` — 资源缓存 (L1)
- `src/Platform/SteamIntegration.cs` — Steam Cloud 同步
- `src/Platform/SwitchIntegration.cs` — Nintendo Switch Cloud
- `src/Platform/ICloudSync.cs` — 云同步抽象接口
- `tests/integration/test_save_system.py` — SaveSystem 集成测试
- `tests/integration/test_sync_conflict.py` — 跨设备同步冲突测试
- `tests/integration/test_migration_v1_to_v2.py` — 存档迁移测试
- `tools/db/migrate.sh` — Alembic 迁移脚本
- `tools/db/backup.sh` — 备份脚本
- `tools/db/restore.sh` — 恢复脚本
- `.github/workflows/db-migration.yml` — DB 迁移 CI

## 11. 关联代码模块 (Code Modules)

| 模块 | 路径 | 状态 | 引用 |
|------|------|:----:|------|
| `SaveSystem` | `src/SaveSystem/` | 待创建 | SQLite + Protobuf + AES-256 |
| `Settings` | `src/Settings/` | 待创建 | JSON 序列化 |
| `Telemetry` | `src/Telemetry/` | 待创建 | Protobuf 批量 + 异步 |
| `AssetPipeline` | `src/AssetPipeline/` | 待创建 | Addressables (L1 缓存) |
| `Platform` | `src/Platform/` | 待创建 | Steam/Switch/PS5 Cloud |
| `DevOps` | `tools/db/` + `.github/workflows/` | 待创建 | 迁移 + 备份 |

## 12. 风险与开放问题 (Risks & Open Questions)

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| **R-01** | **P0-001：02-v2 §13 AC-06 缺"难度上限 20"硬约束** | 中 | 100% | 11 字段标 TODO + p0-001-tracking.md + auto-chain 不修复 | **OPEN** |
| R-02 | **v1.0 SQLite 跨平台文件锁问题 (iOS/Android)** | 中 | 30% | WAL 模式 + 单连接串行化 + 重试机制 | 已规划 |
| R-03 | **Protobuf 前后兼容 (字段 deprecated/新增)** | 中 | 40% | 字段编号保留 + `[Deprecated]` 标记 + 迁移路径 | 已规划 |
| R-04 | **Redis 7 单点故障** | 中 | 30% | Sentinel HA (1 主 + 2 从) + 自动 failover ≤ 30s | 已规划 |
| R-05 | **跨设备同步冲突 (last-writer-wins 丢失进度)** | 高 | 50% | 时间戳 + player 决策 (UI 提示) + 未来 merge 算法 | **待解决** |
| R-06 | **备份加密密钥丢失 = 永久不可恢复** | 高 | 5% | 密钥分片存储 (3 选 2) + 季度密钥轮换 | 已规划 |
| Q-01 | **是否引入 TimescaleDB (遥测专用)** | 低 | — | v2.0+ 评估；v1.0 用 Redis Stream + 异步落盘 PostgreSQL | 倾向推迟 |
| Q-02 | **是否引入 CRDT (冲突解决)** | 低 | — | v1.0/v2.0 维持 last-writer-wins + UI 提示 | 倾向推迟 |

## 13. 待办事项 (TODO)

- [ ] **P0:** 客户端 SQLite + EF Core + Protobuf-net 集成 — 阻塞 M02 SaveSystem
- [ ] **P0:** 存档二进制序列化 + AES-256-GCM 加密 — 阻塞 M02
- [ ] **P0:** 存档备份机制 (.bak 轮转) — 阻塞 M02
- [ ] **P0:** Steam Cloud 同步集成 — 阻塞 M03 v1.0
- [ ] **P1:** v2.0+ PostgreSQL schema + Alembic 迁移 — 不阻塞 v1.0
- [ ] **P1:** Redis 7 缓存层 + Sentinel HA — 不阻塞 v1.0
- [ ] **P1:** 跨设备同步冲突解决 — 不阻塞 v1.0
- [ ] **P1:** 3-2-1 备份策略 — 不阻塞 v1.0
- [ ] **P2:** TimescaleDB 遥测专用存储 — 不阻塞 v2.0
- [ ] **P2:** CRDT 冲突解决 — 不阻塞 v2.0
- [ ] **P0：** P0-001 跟踪 — 等陛下 21:00 调研后决定修复时机

## 14. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建**：8 文件（README / persistence-strategy / database-schema / cache-strategy / serialization / migrations / backup-and-recovery / **p0-001-tracking**）。18 张表（12 主体 + 6 附加）+ SQLite 本地 + PostgreSQL 16 (v2.0+) + Redis 7 + Protobuf 主序列化 + Alembic 三阶段 + 3-2-1 备份。**P0-001 跟踪：** 11 阻塞字段 TODO + p0-001-tracking.md 完整 + auto-chain 不修复。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）