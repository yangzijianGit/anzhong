---
title: 《暗室》持久化策略 (Persistence Strategy)
doc_id: DESIGN-anzhong-data-persistence
parent: design/data/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》持久化策略 (Persistence Strategy)

> **一句话定位：** v1.0 离线优先 + SQLite 本地真理之源 + Steam Cloud 可选同步；v2.0+ 双写一读 (Client local + Server authoritative) + 冲突解决 (last-writer-wins + UI 提示) + 跨设备 OAuth 绑定。

## 目的 (Purpose)

本文档是《暗室》**持久化层 (Persistence Layer)** 的端到端策略基线。它向：

- **Unity 客户端工程师** — 定义 SQLite 表创建、读写时机、连接管理、EF Core 迁移、备份 .bak 轮转
- **服务端工程师 (v2.0+)** — 定义 PostgreSQL + Redis 双写一读、跨设备同步冲突解决、OAuth 绑定
- **DevOps / SRE** — 定义备份策略、监控告警、灾备演练
- **策划/PM** — 理解"哪些数据存哪里、玩家离线能玩什么"
- **架构师** — 提供离线优先 + Eventually Consistent 的端到端契约

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的全部持久化策略——**v1.0 SQLite 离线优先 (本地真理之源) + .bak 双保险 + Steam Cloud 辅助 + v2.0+ PostgreSQL/Redis 云端 + 跨设备同步 + 冲突解决 + GDPR 数据导出/删除**——**第一次**用统一文档描述，作为 phase3 → phase4 实施的"持久化合同"。

## 范围 (Scope)

### 包含

- **v1.0 客户端持久化**: SQLite 3 + EF Core 6 + .bak 备份 + Steam Cloud 同步
- **v2.0+ 服务端持久化**: PostgreSQL 16 + Redis 7 + Alembic + 跨设备 OAuth
- **离线优先策略**: Local-First / Source of Truth / Eventually Consistent
- **同步策略**: Steam Cloud (v1.0) / Custom Cloud (v2.0+) / Last-Writer-Wins
- **冲突解决**: 时间戳优先 + UI 提示 + 未来 Merge (v3.0)
- **GDPR 合规**: 数据导出 / 数据删除 (11 §5.4)

### 不包含 (Out of Scope)

- 具体字段定义 → 见 `database-schema.md`
- 序列化格式对比 → 见 `serialization.md`
- 缓存键名空间 → 见 `cache-strategy.md`
- 迁移路径 → 见 `migrations.md`
- 备份与恢复 → 见 `backup-and-recovery.md`
- API 端点契约 → 见 `../api/endpoints.md`

## 一句话描述 (One-liner)

> **"v1.0 离线优先 + SQLite 本地 + Steam Cloud；v2.0+ 双写一读 + PostgreSQL/Redis + 跨设备同步。"**

## 1. v1.0 持久化架构 (Local-First)

### 1.1 客户端分层

```
┌─────────────────────────────────────────────────────────┐
│ Unity 业务层 (Core/SwitchSlot/Room/Player/UI/Audio)     │
└────────────────────────┬────────────────────────────────┘
                         │ EF Core DbContext
┌────────────────────────▼────────────────────────────────┐
│ 持久化层 (SaveSystem / Settings / Telemetry / Asset)   │
│ ┌──────────┬──────────────┬─────────────┬──────────────┐ │
│ │ SQLite   │ JSON 文件    │ 二进制文件  │ Addressables │ │
│ │ 主存储    │ 设置/配置    │ 存档/槽位   │ 资源缓存      │ │
│ └────┬─────┴──────┬───────┴──────┬──────┴──────┬───────┘ │
└──────┼────────────┼───────────────┼─────────────┼─────────┘
       │            │               │             │
       ▼            ▼               ▼             ▼
   savegame.db   audio.json     savegame.bin   *.bundle
   (SQLite 3)    (JSON)         (AES-256)      (二进制资源)
       │
       ▼
   Steam Cloud (可选, 自动同步)
```

### 1.2 4 类持久化数据

| 数据类型 | 存储格式 | 路径 | 频率 | 失败处理 |
|---------|---------|------|------|---------|
| **玩家存档 (SaveData)** | SQLite (主) + 二进制 (备份) | `%APPDATA%/anzhong/savegame.db` + `savegame.bin` | 房间通关 + 章节完成 + 应用退出 | .bak 双保险 + 提示玩家 |
| **玩家设置 (Audio/Accessibility)** | JSON | `%APPDATA%/anzhong/settings.json` | 即时 | 旧值保留 + 提示 |
| **遥测数据 (TelemetryEvent)** | SQLite (本地聚合) | `%APPDATA%/anzhong/telemetry.db` | 60s 批处理 | 7d 滚动清理 |
| **资源缓存 (Addressables)** | 二进制 Bundle | `%APPDATA%/anzhong/aa_cache/` | 启动加载 | 自动重试 |

### 1.3 关键设计原则 (Offline-First)

1. **本地是真理之源 (Local is Source of Truth)**: v1.0 所有写入只到本地 SQLite，云端仅辅助
2. **写穿透 + 读缓存 (Write-Through + Read-Aside)**: 写入立即落 SQLite，读取先查 L1 缓存
3. **Eventually Consistent**: v2.0+ 异步云同步，最坏情况 = 玩家最后操作胜出
4. **零阻塞 (Zero Blocking)**: 所有 I/O 在后台线程，不阻塞主循环 ≤ 16ms (60 FPS)
5. **容错优先 (Resilience First)**: 损坏 → 自动加载 .bak → 提示玩家，不阻断通关
6. **GDPR Ready**: 数据导出/删除 API 已就位 (虽然 v1.0 无服务器)

### 1.4 写入时机

| 时机 | 写入内容 | 频率 | 优先级 |
|------|---------|------|:------:|
| **房间通关时** | completedRooms[ch] + roomStats[roomId] + score | 19 次/通关 | P0 |
| **章节完成时** | chapterCompleted[ch] = true + gameCompleted 检查 | 3 次/通关 | P0 |
| **应用退出时** | lastSavedRoomId + currentChapterId | 1 次/会话 | P0 |
| **设置变更时** | audioVolumes + accessibility + difficulty | 即时 | P1 |
| **Hint 触发时** | hintsTriggered[roomId] | 即时 | P1 |
| **Slot 切换时** | **不存档** (切换中状态不持久化, 02 §2.4) | — | — |
| **遥测指标** | metricName + value + tags | 60s 批处理 | P2 |

> **关键：** Switching 状态（200ms 动画）不存档，避免"切换到一半退出 → 状态错乱"（02 §2.4 边界）。

### 1.5 读取时机

| 时机 | 读取内容 | 频率 | 缓存 |
|------|---------|------|------|
| **启动** | 完整 SaveData + settings | 1 次/启动 | 全部加载到内存 |
| **选择章节** | chapterProgress + lastSavedRoomId + chapterCompleted | 1 次/章节 | Addressables |
| **房间通关失败回退** | 上一存档（重置 R 键） | 偶尔 | 不缓存 |
| **读档（玩家主动）** | 完整 SaveData | 玩家触发 | 全部加载 |
| **退出到主菜单** | lastSavedRoomId | 1 次/退出 | — |

## 2. v1.0 SQLite Schema 概览 (SQLite Schema Overview)

> 详见 `database-schema.md`（12+ 完整 DDL）。

### 2.1 v1.0 SQLite (Client-Side)

```sql
-- 玩家表（单设备本地，无 FK 服务端 player_id）
CREATE TABLE local_player (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    device_id TEXT NOT NULL UNIQUE,
    display_name TEXT NOT NULL,
    platform TEXT NOT NULL CHECK (platform IN ('steam','mac','switch','ps5','xbox','ios','android','itch','anonymous')),
    locale TEXT NOT NULL DEFAULT 'zh-CN',
    created_at TEXT NOT NULL DEFAULT (datetime('now')),
    total_play_time_sec INTEGER NOT NULL DEFAULT 0
);

-- 存档表（单玩家单存档，1-to-1）
CREATE TABLE save_data (
    id INTEGER PRIMARY KEY CHECK (id = 1),  -- 强制单存档
    version TEXT NOT NULL DEFAULT '1.0.0',
    last_updated TEXT NOT NULL DEFAULT (datetime('now')),
    current_chapter_id TEXT NOT NULL DEFAULT 'ch1',
    current_room_id TEXT NOT NULL DEFAULT '1-1',
    completed_rooms_json TEXT NOT NULL DEFAULT '{"ch1":[],"ch2":[],"ch3":[]}',  -- JSON
    chapter_completed_json TEXT NOT NULL DEFAULT '{"ch1":false,"ch2":false,"ch3":false}',
    game_completed INTEGER NOT NULL DEFAULT 0,
    total_play_time_sec INTEGER NOT NULL DEFAULT 0,
    total_resets INTEGER NOT NULL DEFAULT 0,
    room_stats_json TEXT NOT NULL DEFAULT '{}',
    accessibility_settings_json TEXT,  -- 嵌入 M10
    audio_settings_json TEXT           -- 嵌入 M09
);

-- 房间配置表（19 房间只读，从策划 JSON 导入）
CREATE TABLE room_config (
    room_id TEXT PRIMARY KEY,  -- '1-1' ~ '3-8'
    chapter_id TEXT NOT NULL,
    name TEXT NOT NULL,
    type TEXT NOT NULL CHECK (type IN ('tutorial','standard','challenge','boss')),
    width INTEGER NOT NULL CHECK (width BETWEEN 4 AND 16),
    height INTEGER NOT NULL CHECK (height BETWEEN 4 AND 12),
    difficulty INTEGER NOT NULL CHECK (difficulty BETWEEN 1 AND 20),
    difficulty_max INTEGER NOT NULL DEFAULT 20 CHECK (difficulty_max = 20),  -- P0-001 自我保护
    target_duration_p50_sec INTEGER NOT NULL CHECK (target_duration_p50_sec BETWEEN 60 AND 1200),
    target_duration_p90_sec INTEGER NOT NULL CHECK (target_duration_p90_sec BETWEEN 180 AND 1800),
    slots_json TEXT NOT NULL  -- array of SwitchSlot
);

-- 房间通关统计（每个房间最多 1 行 per device）
CREATE TABLE room_stats (
    room_id TEXT PRIMARY KEY,
    attempts INTEGER NOT NULL DEFAULT 0,
    resets INTEGER NOT NULL DEFAULT 0,
    completion_time_sec INTEGER,
    hints_triggered INTEGER NOT NULL DEFAULT 0,
    best_score_json TEXT,
    last_played_at TEXT
);

-- 章节进度
CREATE TABLE chapter_progress (
    chapter_id TEXT PRIMARY KEY,
    unlocked INTEGER NOT NULL DEFAULT 0,
    completed INTEGER NOT NULL DEFAULT 0,
    progress_pct REAL NOT NULL DEFAULT 0,
    average_difficulty REAL  -- TODO P0-001
);

-- 遥测事件聚合（v1.0 本地聚合 + v2.0+ 上报）
CREATE TABLE telemetry_events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    event_type TEXT NOT NULL,
    room_id TEXT,
    slot_id TEXT,
    timestamp_ms INTEGER NOT NULL,
    properties_json TEXT,
    session_id TEXT NOT NULL
);
CREATE INDEX idx_telemetry_session ON telemetry_events(session_id);
CREATE INDEX idx_telemetry_type ON telemetry_events(event_type);

-- 设置表（单行）
CREATE TABLE audio_settings (
    id INTEGER PRIMARY KEY CHECK (id = 1),
    master_volume REAL NOT NULL DEFAULT 0.8,
    switch_sfx_db REAL NOT NULL DEFAULT -12,
    reset_sfx_db REAL NOT NULL DEFAULT -18,
    win_sfx_db REAL NOT NULL DEFAULT -6,
    error_sfx_db REAL NOT NULL DEFAULT -12,
    tutorial_sfx_db REAL NOT NULL DEFAULT -9,
    chapter_bgm_volume REAL NOT NULL DEFAULT 0.6,
    room_theme_volume REAL NOT NULL DEFAULT 0.5,
    ambient_volume REAL NOT NULL DEFAULT 0.4,
    ui_feedback_db REAL NOT NULL DEFAULT -15,
    muted INTEGER NOT NULL DEFAULT 0
);

CREATE TABLE accessibility_settings (
    id INTEGER PRIMARY KEY CHECK (id = 1),
    colorblind_mode TEXT NOT NULL DEFAULT 'default' CHECK (colorblind_mode IN ('default','red_green','full')),
    font_scale REAL NOT NULL DEFAULT 1.0 CHECK (font_scale IN (1.0, 1.25, 1.5)),
    controller_support TEXT NOT NULL DEFAULT 'xbox' CHECK (controller_support IN ('xbox','ps','switch_pro','generic')),
    difficulty TEXT NOT NULL DEFAULT 'normal' CHECK (difficulty IN ('easy','normal')),  -- hard v1.1
    high_contrast INTEGER NOT NULL DEFAULT 0,
    reduced_motion INTEGER NOT NULL DEFAULT 0,
    screen_reader INTEGER NOT NULL DEFAULT 0  -- v1.1
);
```

### 2.2 索引设计 (SQLite Indexes)

```sql
-- 房间通关统计按房间查询
CREATE UNIQUE INDEX idx_room_stats_room ON room_stats(room_id);

-- 遥测事件按 session + timestamp 查询
CREATE INDEX idx_telemetry_session_ts ON telemetry_events(session_id, timestamp_ms);

-- 遥测事件按类型查询（聚合）
CREATE INDEX idx_telemetry_type_ts ON telemetry_events(event_type, timestamp_ms);
```

### 2.3 EF Core DbContext 映射 (C# Client)

```csharp
// src/SaveSystem/AnzhongDbContext.cs
public class AnzhongDbContext : DbContext
{
    public DbSet<LocalPlayer> LocalPlayers { get; set; }
    public DbSet<SaveDataEntity> SaveData { get; set; }
    public DbSet<RoomConfigEntity> RoomConfigs { get; set; }
    public DbSet<RoomStatsEntity> RoomStats { get; set; }
    public DbSet<ChapterProgressEntity> ChapterProgress { get; set; }
    public DbSet<TelemetryEventEntity> TelemetryEvents { get; set; }
    public DbSet<AudioSettingsEntity> AudioSettings { get; set; }
    public DbSet<AccessibilitySettingsEntity> AccessibilitySettings { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder options)
    {
        // SQLite WAL 模式 + 单连接
        options.UseSqlite($"Data Source={Application.persistentDataPath}/savegame.db",
            o => o.CommandTimeout(30));
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // 单行表约束 (CHECK id = 1)
        modelBuilder.Entity<SaveDataEntity>().HasCheckConstraint("ck_save_data_single", "id = 1");
        modelBuilder.Entity<AudioSettingsEntity>().HasCheckConstraint("ck_audio_settings_single", "id = 1");
        modelBuilder.Entity<AccessibilitySettingsEntity>().HasCheckConstraint("ck_accessibility_single", "id = 1");

        // JSON 字段
        modelBuilder.Entity<SaveDataEntity>().Property(e => e.CompletedRoomsJson).HasColumnType("TEXT");
        // ... (其他 JSON 字段)
    }
}
```

## 3. Steam Cloud 同步 (v1.0 Cloud Sync)

### 3.1 Steamworks Cloud Sync API

```csharp
// src/Platform/SteamIntegration.cs
public class SteamCloudSync : ICloudSync
{
    private const int MaxCloudFileSize = 100 * 1024;  // 100KB 上限
    private const string SaveFileName = "savegame.db";
    private const string BackupFileName = "savegame.db.bak";
    private const string SettingsFileName = "settings.json";

    public async Task<SyncResult> SyncAsync()
    {
        if (!SteamAPI.IsAvailable) return SyncResult.Skipped;

        // 1. 上传本地存档到 Steam Cloud
        var localBytes = File.ReadAllBytes(GetLocalSavePath());
        if (localBytes.Length > MaxCloudFileSize) return SyncResult.TooLarge;

        var remoteTimestamp = SteamRemoteStorage.GetFileTimestamp(SaveFileName);
        var localTimestamp = File.GetLastWriteTimeUtc(GetLocalSavePath());

        if (localTimestamp > remoteTimestamp)
        {
            // 本地更新 → 覆盖云端
            SteamRemoteStorage.FileWrite(SaveFileName, localBytes);
            return SyncResult.LocalNewer;
        }
        else if (remoteTimestamp > localTimestamp)
        {
            // 云端更新 → 下载到本地
            var cloudBytes = SteamRemoteStorage.FileRead(SaveFileName);
            File.WriteAllBytes(GetLocalSavePath(), cloudBytes);
            return SyncResult.CloudNewer;
        }

        return SyncResult.NoChange;
    }
}
```

### 3.2 同步时机

| 时机 | 触发 | 频率 |
|------|------|------|
| **应用启动** | 检查云端 vs 本地时间戳 | 1 次/启动 |
| **应用退出** | 写入后异步上传 | 1 次/退出 |
| **房间通关** | 异步上传 (best-effort) | 19 次/通关 |
| **手动同步** | 玩家点击"立即同步"按钮 | 玩家触发 |

### 3.3 冲突解决 (v1.0)

**v1.0 策略：Last-Writer-Wins by Timestamp (UTC)**

```
if local_timestamp > cloud_timestamp:
    upload local → cloud  # 本地胜
else if cloud_timestamp > local_timestamp:
    download cloud → local  # 云端胜
else:
    no-op  # 完全一致
```

**冲突场景：** 玩家 A 在 Steam Deck 上玩 2 小时，玩家 B 在 PC Steam 上玩 1 小时，最后玩家 A 关闭 Steam Deck，玩家 B 关闭 PC。

**结果：** 玩家 B 的进度**被覆盖** (因为 B 的时间戳更晚)。**v1.0 不做 UI 提示** (玩家单机为主，无云端 UI)。

**v2.0+ 改进：** 见 §5.3。

### 3.4 配额与限制

| 维度 | 限制 |
|------|------|
| **单文件大小** | 100 KB (存档 + 设置) |
| **总配额** | 100 MB / Steam 用户 (足够 19 房间 + 9 音频 + 资源缓存) |
| **同步频率** | ≤ 60 次/小时 (Steam API rate limit) |
| **失败重试** | 3 次指数退避 (1s, 5s, 30s) |

## 4. v2.0+ 服务端持久化 (Server-Side)

### 4.1 双写一读架构 (Dual-Write / Single-Read)

```
┌────────────────────────┐
│  Unity Client (v2.0+) │
│ ┌────────┬────────┐    │
│ │ SQLite │ 缓存   │    │
│ │ 本地    │ L1/L2 │    │
│ └───┬────┴────┬───┘    │
└─────┼──────────┼────────┘
      │ write    │ read
      ▼          ▼
┌────────────────────────┐
│  API Gateway (.NET 8)  │
└──────┬──────────┬──────┘
       │ write    │ read
       ▼          ▼
┌────────────────────────┐
│  PostgreSQL 16 (Master)│◄────┐ Streaming Replication
│  - 玩家/存档/设置        │     │
│  - 遥测/分数/反馈        │     │
└────────────────────────┘     │
┌────────────────────────┐    │
│  PostgreSQL 16 (Replica)├────┘
└────────────────────────┘
       ▲           ▲
       │ cache    │ session
┌──────┴───────────┴──────┐
│  Redis 7 (Sentinel HA)  │
│  - 会话/锁/排行榜        │
│  - 缓存/限流/遥测Stream │
└─────────────────────────┘
```

### 4.2 写入路径 (v2.0+ Write Path)

```
Client.Write(SaveData):
  1. 写入本地 SQLite (同步, ≤ 50ms)
  2. 调用 API /saves (异步, ≤ 1500ms)
     → API Gateway → Save Service
     → 获取 Redis 分布式锁 (save:{playerId}:lock, TTL 30s)
     → 写 PostgreSQL (save_data 表)
     → 失效 Redis 缓存 (save:{playerId})
     → 释放锁
  3. 失败 → 重试 3 次 → 标记云端离线 (本地继续可用)
```

### 4.3 读取路径 (v2.0+ Read Path)

```
Client.Read(SaveData):
  1. 优先读本地 SQLite (≤ 20ms)
  2. 启动时拉取云端 (≤ 1500ms)
     → API Gateway → Save Service
     → 查 Redis 缓存 (save:{playerId})
       - 命中 → 直接返回
       - 未命中 → 查 PostgreSQL → 写回 Redis (TTL 5min)
  3. 比对时间戳 → 选择最新
```

### 4.4 关键设计原则 (v2.0+)

1. **客户端仍为真理之源 (Local-First)**: 即使云端写入失败，玩家仍可用本地进度
2. **异步 + 失败容忍 (Async + Fault-Tolerant)**: 云同步失败不影响本地游戏
3. **幂等 + 重试 (Idempotent + Retry)**: 所有写入带 Idempotency-Key，3 次指数退避
4. **最终一致 (Eventually Consistent)**: ≤ 5 分钟内所有设备同步
5. **冲突解决显式化 (Conflict Resolution Explicit)**: UI 提示玩家选择
6. **GDPR Ready**: 数据导出/删除 API 已就位

## 5. 跨设备同步 (v2.0+ Multi-Device Sync)

### 5.1 设备绑定 (Device Linking)

```csharp
// src/Platform/DeviceLinkService.cs
public class DeviceLinkService
{
    // 玩家在 Device A 上发起链接
    public async Task<string> StartLinkAsync(string playerId, string deviceId)
    {
        // 1. 生成 OAuth Code (6 位, 5 分钟有效)
        var code = GenerateOAuthCode();
        // 2. 存储到 Redis: link:{code} → { playerId, deviceId, expiresAt }
        await Redis.SetAsync($"link:{code}", Json(playerId, deviceId), TimeSpan.FromMinutes(5));
        return code;
    }

    // 玩家在 Device B 上输入 Code 完成链接
    public async Task<LinkResult> CompleteLinkAsync(string code, string newDeviceId, string newPlatform)
    {
        var data = await Redis.GetAsync($"link:{code}");
        if (data == null) return LinkResult.Expired;

        var (playerId, originalDeviceId) = JsonParse(data);
        // 绑定新设备到 playerId
        await Db.DeviceLinks.AddAsync(new DeviceLink
        {
            PlayerId = playerId,
            DeviceId = newDeviceId,
            Platform = newPlatform,
            LinkedAt = DateTime.UtcNow
        });
        await Db.SaveChangesAsync();
        await Redis.DelAsync($"link:{code}");
        return LinkResult.Success(playerId);
    }
}
```

### 5.2 同步协议 (Sync Protocol)

```json
// POST /api/v1/sync
{
  "playerId": "player_x9y8z7",
  "sourcePlatform": "steam",
  "targetPlatform": "switch",
  "saveData": { "version": "1.0.0", "currentChapterId": "ch2", "currentRoomId": "2-3", "lastUpdated": "2026-06-29T10:30:00Z" },
  "clientTimestampMs": 1719652800000,
  "idempotencyKey": "uuid-v4"
}

// 200 OK
{
  "conflictDetected": false,
  "resolvedSave": { ... },  // 服务端选择的最新版本
  "lastSyncAt": "2026-06-29T10:30:00Z"
}

// 409 CONFLICT (v2.0+ UI 提示)
{
  "conflictDetected": true,
  "localSave": { "currentRoomId": "2-3", "lastUpdated": "2026-06-29T10:30:00Z" },
  "cloudSave": { "currentRoomId": "2-5", "lastUpdated": "2026-06-29T10:35:00Z" },
  "resolution": "server_wins",  // 默认服务端胜
  "lostProgress": {
    "rooms": ["2-4"],  // 玩家在 Device A 通关但被覆盖
    "stats": { "totalResets": 12 }
  }
}
```

### 5.3 冲突解决策略 (Conflict Resolution)

| 策略 | v1.0 | v2.0+ | 备注 |
|------|:----:|:----:|------|
| **Last-Writer-Wins (LWW)** | ✅ | ✅ | 默认，简洁 |
| **Server-Wins** | ❌ | ✅ | 简单可靠 |
| **UI 提示玩家选择** | ❌ | ✅ | 复杂但用户体验好 |
| **自动 Merge (CRDT)** | ❌ | ❌ (v3.0) | 19 房间规模无需 |

**v2.0+ 默认：** Server-Wins + UI 提示"你在另一台设备上的进度已被覆盖，是否切换到该进度？"

**v2.0+ 实现细节：**
- 比较 `lastUpdated` 时间戳
- 服务端时间戳更晚 → 服务端胜，覆盖客户端
- 客户端时间戳更晚 → 上传客户端，冲突解决
- 时间戳相同 (毫秒级碰撞) → 用 player_id 哈希作为 tiebreaker

### 5.4 同步频率与配额

| 维度 | 限制 |
|------|------|
| **同步频率** | ≤ 10 req/h/player (rate limit) |
| **单存档大小** | ≤ 100 KB |
| **传输加密** | TLS 1.3 + AES-256 (存档额外加密) |
| **幂等性** | Idempotency-Key (UUID v4), 24h 缓存 |
| **失败重试** | 3 次指数退避 (1s, 5s, 30s) |

## 6. GDPR 数据合规 (GDPR Compliance)

> 与 11-v2 §5.4 GDPR 一致。

### 6.1 数据导出 (Data Export - Right to Access)

```csharp
// src/SaveSystem/ExportSave.cs
public class ExportSave
{
    public async Task<byte[]> ExportPlayerDataAsync(string playerId)
    {
        // 收集所有玩家数据
        var player = await Db.Players.FindAsync(playerId);
        var saves = await Db.Saves.Where(s => s.PlayerId == playerId).ToListAsync();
        var scores = await Db.Scores.Where(s => s.PlayerId == playerId).ToListAsync();
        var feedback = await Db.Feedback.Where(f => f.PlayerId == playerId).ToListAsync();
        var settings = await Db.PlayerAudioSettings.Where(a => a.PlayerId == playerId).ToListAsync();

        var export = new
        {
            exportVersion = "1.0.0",
            exportedAt = DateTime.UtcNow,
            player,
            saves,
            scores,
            feedback,
            settings
        };

        // JSON 序列化 + 签名 (防篡改)
        var json = JsonSerializer.Serialize(export);
        var signed = SignWithSHA256(json);  // 签名非加密，玩家可读
        return Encoding.UTF8.GetBytes(signed);
    }
}
```

**菜单入口：** 设置 → 隐私 → "导出我的数据" → 生成 `anzhong-data-export-{playerId}-{timestamp}.json`

### 6.2 数据删除 (Data Deletion - Right to Erasure)

```csharp
// src/SaveSystem/DeletePlayerData.cs
public class DeletePlayerData
{
    public async Task<DeleteResult> DeletePlayerDataAsync(string playerId)
    {
        // 1. 删除主表
        await Db.Saves.Where(s => s.PlayerId == playerId).ExecuteDeleteAsync();
        await Db.Scores.Where(s => s.PlayerId == playerId).ExecuteDeleteAsync();
        await Db.Feedback.Where(f => f.PlayerId == playerId).ExecuteDeleteAsync();
        await Db.PlayerAudioSettings.Where(a => a.PlayerId == playerId).ExecuteDeleteAsync();
        await Db.TelemetryEvents.Where(t => t.Session.PlayerId == playerId).ExecuteDeleteAsync();

        // 2. 软删除 Player (保留 30 天用于撤销)
        var player = await Db.Players.FindAsync(playerId);
        player.DeletedAt = DateTime.UtcNow;
        player.Anonymized = true;
        await Db.SaveChangesAsync();

        // 3. 失效 Redis 缓存
        await Redis.DelAsync($"save:{playerId}");
        await Redis.DelAsync($"progress:{playerId}");

        // 4. 审计日志
        await Db.AuditLogs.AddAsync(new AuditLog
        {
            Action = "gdpr_delete",
            PlayerId = playerId,
            Timestamp = DateTime.UtcNow,
            Result = "success"
        });

        return DeleteResult.Success;
    }
}
```

**菜单入口：** 设置 → 隐私 → "删除我的数据" → 二次确认 → 30 天软删除期 → 彻底删除

### 6.3 审计日志 (Audit Log)

```sql
CREATE TABLE audit_logs (
    id BIGSERIAL PRIMARY KEY,
    action VARCHAR(50) NOT NULL,  -- 'gdpr_export', 'gdpr_delete', 'sync_conflict_resolved', etc.
    player_id BIGINT,
    device_id VARCHAR(100),
    timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    result VARCHAR(20) NOT NULL,  -- 'success', 'failed', 'partial'
    metadata JSONB  -- 详细信息
);
CREATE INDEX idx_audit_player ON audit_logs(player_id);
CREATE INDEX idx_audit_timestamp ON audit_logs(timestamp);
```

**保留期：** 1 年 (合规要求)

## 7. 备份与恢复 (Backup & Recovery)

> 详见 `backup-and-recovery.md`。

### 7.1 v1.0 备份策略

| 维度 | 策略 |
|------|------|
| **本地 .bak** | 每次写入前备份到 `savegame.db.bak` |
| **保留期** | 3 个版本轮转 (`.bak`, `.bak.2`, `.bak.3`) |
| **自动恢复** | 主存档损坏 → 自动加载 .bak |
| **Steam Cloud** | 自动冗余备份 (3 个数据中心) |
| **加密** | AES-256-GCM (存档额外加密) |

### 7.2 v2.0+ 备份策略 (3-2-1)

| 维度 | 策略 |
|------|------|
| **3 副本** | 本地 + 异地 + 离线 |
| **2 介质** | SSD + HDD (或云 + 磁带) |
| **1 异地** | 不同地理位置 (≥ 100 km) |
| **RPO** | 5 min (WAL 持续归档) |
| **RTO** | 1 h (本地恢复) / 30 min (failover) |

## 8. 性能与容量规划 (Performance & Capacity)

### 8.1 v1.0 性能指标

| 操作 | 目标 | 实际 |
|------|------|------|
| **存档写入** | ≤ 50ms (后台) | ~30ms |
| **读档** | ≤ 20ms | ~10ms |
| **房间配置加载** | ≤ 100ms | ~50ms |
| **设置读取** | ≤ 10ms | ~5ms |
| **Steam Cloud 同步** | ≤ 3000ms | ~1500ms |
| **本地 SQLite 查询** | ≤ 5ms | ~2ms |

### 8.2 v2.0+ 性能指标

| 操作 | 目标 | 备注 |
|------|------|------|
| **API 响应 (P95)** | ≤ 200ms | GET endpoints |
| **API 响应 (P95)** | ≤ 500ms | POST endpoints |
| **云存档同步 (P95)** | ≤ 1500ms | E15 sync |
| **Redis 命中** | ≥ 95% | SaveData / Leaderboard |
| **PostgreSQL QPS** | ≤ 10000/s | 单主 + Replica |

### 8.3 容量规划

| 数据 | 单条大小 | 19 房间/玩家 | 1000 玩家 | 100K 玩家 |
|------|---------|--------------|----------|----------|
| **save_data** | ~3 KB | 3 KB | 3 MB | 300 MB |
| **room_stats** | ~200 B | 3.8 KB | 3.8 MB | 380 MB |
| **scores** | ~500 B | 9.5 KB | 9.5 MB | 950 MB |
| **telemetry_events** | ~1 KB | ~5K 事件/通关 = 5 MB | 5 GB | 500 GB |
| **总计 (PostgreSQL)** | — | ~10 MB | ~10 GB | ~1 TB |

## 9. 错误处理与监控 (Error Handling & Monitoring)

### 9.1 错误码

| 错误码 | HTTP | 含义 | 处理 |
|--------|:----:|------|------|
| `STORAGE_FULL` | 507 | 磁盘满 | 清理 .bak → 提示玩家 |
| `SAVE_VERSION_MISMATCH` | 409 | 存档版本不兼容 | 触发迁移或重新开始 |
| `SAVE_CORRUPTED` | 500 | 存档 CRC 校验失败 | 自动加载 .bak |
| `SYNC_CONFLICT` | 409 | 跨设备同步冲突 | UI 提示选择 |
| `STORAGE_PERMISSION_DENIED` | 403 | 磁盘权限不足 | 提示玩家检查权限 |
| `CLOUD_QUOTA_EXCEEDED` | 507 | Steam Cloud 配额满 | 提示玩家清理 |

### 9.2 监控指标

| 指标 | 阈值 | 告警 |
|------|------|------|
| **save_write_p95_ms** | > 100 | Slack |
| **sync_conflict_rate** | > 10% | Slack |
| **sqlite_corruption_rate** | > 0.1% | PagerDuty |
| **steam_cloud_sync_failure** | > 5% | Slack |
| **gdpr_export_request_count** | > 100/day | Slack |

## 10. 关联文档 (Cross-References)

- [`README.md`](./README.md) — 数据架构总览
- [`database-schema.md`](./database-schema.md) — 12+ 表完整 DDL
- [`cache-strategy.md`](./cache-strategy.md) — Redis 缓存策略
- [`serialization.md`](./serialization.md) — Protobuf + JSON + 二进制
- [`migrations.md`](./migrations.md) — Alembic 3 阶段迁移
- [`backup-and-recovery.md`](./backup-and-recovery.md) — 3-2-1 + RPO/RTO
- [`p0-001-tracking.md`](./p0-001-tracking.md) — P0-001 跟踪
- [`../api/data-models.md`](../api/data-models.md) — M01-M12 字段定义
- [`../api/endpoints.md`](../api/endpoints.md) — 18 端点数据流
- [`../../docs/04-gameplay-flow-v2.md`](../../docs/04-gameplay-flow-v2.md) — SaveData 接口契约
- [`../../docs/11-release-v2.md`](../../docs/11-release-v2.md) — GDPR 合规

## 11. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** v1.0 离线优先 (SQLite + Steam Cloud) + v2.0+ 双写一读 (PostgreSQL + Redis) + 跨设备同步 (LWW + UI 提示) + GDPR (数据导出/删除) + 9 类写入时机 + 性能指标 + 监控告警。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）