---
title: 《暗室》数据迁移 (Migrations)
doc_id: DESIGN-anzhong-data-migrations
parent: design/data/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》数据迁移 (Migrations)

> **一句话定位：** Alembic (服务端) + EF Core (客户端 SQLite) 双轨迁移 + 3 阶段演进 (v1.0 本地 → v1.1 云存档 → v2.0 全平台) + 存档版本兼容 (v1.0.0 → v1.1.0 → v2.0.0) + 零停机部署 + 前后向兼容。

## 目的 (Purpose)

本文档是《暗室》**数据迁移层 (Migration Layer)** 的**唯一权威策略规格**。它向：

- **服务端工程师 (v2.0+)** — 定义 Alembic 迁移脚本、版本链、回滚路径、零停机策略
- **Unity 客户端工程师** — 定义 EF Core SQLite 迁移、存档版本兼容、字段演化
- **DevOps / SRE** — 定义蓝绿部署、回滚 SOP、监控告警
- **架构师** — 提供 3 阶段数据演进 (v1.0 → v1.1 → v2.0) + 兼容矩阵

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的全部数据迁移策略——**3 阶段演进 (v1.0 SQLite 本地 → v1.1 Steam Cloud 同步 → v2.0 PostgreSQL 全平台) + Alembic 服务端迁移 + EF Core 客户端迁移 + 存档版本兼容 (v1.0.0 → v1.1.0 → v2.0.0) + 零停机 + 前后向兼容**——**第一次**用统一文档描述，作为 phase3 → phase4 实施的"迁移合同"。

## 范围 (Scope)

### 包含

- **3 阶段演进**: v1.0 (Day-84) → v1.1 (T+3m) → v2.0 (T+6m)
- **服务端迁移**: Alembic + 自动生成 + 版本链 + 回滚
- **客户端迁移**: EF Core SQLite + 存档版本号 + 字段兼容
- **存档兼容**: v1.0.0 → v1.1.0 → v2.0.0 升级路径
- **零停机**: 蓝绿部署 + 双写 + 灰度
- **数据迁移**: 旧版本 → 新版本字段映射 + 数据回填

### 不包含 (Out of Scope)

- API 端点契约 → 见 `../api/endpoints.md`
- 数据库 schema → 见 `database-schema.md`
- 持久化策略 → 见 `persistence-strategy.md`
- 序列化格式 → 见 `serialization.md`
- 备份与恢复 → 见 `backup-and-recovery.md`

## 一句话描述 (One-liner)

> **"3 阶段演进 + Alembic + EF Core 双轨迁移 + 存档版本兼容 + 零停机部署。"**

## 1. 3 阶段数据演进 (3-Phase Data Evolution)

| 阶段 | 时间 | 客户端 | 服务端 | 同步 | 关键变化 |
|------|------|--------|--------|------|---------|
| **v1.0 (Day-84)** | W12 | SQLite 9 表 | ❌ 无 | Steam Cloud (可选) | 本地真理之源 |
| **v1.1 (T+3m)** | W24 | SQLite 9 表 + JSON 设置 v1.1 | ❌ 无 | Steam Cloud (主) + iCloud/PSN | 新增 screen_reader / hard difficulty / 多端云存档 |
| **v2.0 (T+6m)** | W48 | SQLite 9 表 v2.0 + 客户端缓存 | PostgreSQL 18 表 + Redis 7 | 跨设备 OAuth + 实时同步 | 服务端引入 + 跨设备 + 完整 leaderboard |
| **v2.1+ (T+12m)** | TBD | 增量 | 增量 | CRDT (评估) | 持续优化 |

### 1.1 v1.0 → v1.1 关键迁移 (存档版本兼容)

**新增字段：**
- `accessibility_settings.screen_reader` (v1.1 新增)
- `accessibility_settings.difficulty = 'hard'` (v1.1 新增枚举值)
- `audio_settings.spatial_audio` (v1.1 空间音频)

**修改字段：**
- 无字段类型变化 (仅新增)

**删除字段：**
- 无

### 1.2 v1.1 → v2.0 关键迁移 (服务端引入)

**关键变化：**
- 引入 PostgreSQL + Redis (服务端)
- 跨设备同步协议 (E15)
- 服务端排行榜
- GDPR 数据导出/删除 API
- 玩家跨平台账号系统

**字段映射 (v1.1 → v2.0)：**
- `local_player` (SQLite 单设备) → `players` (PostgreSQL 多设备绑定)
- `save_data` (SQLite 单存档) → `saves` (PostgreSQL + Redis 缓存)
- (新增) `device_links`, `audit_logs`, `leaderboard_entries`

## 2. 服务端 Alembic 迁移 (v2.0+)

### 2.1 Alembic 工作流

```bash
# 初始化 (一次性)
alembic init migrations

# 配置 alembic.ini
# sqlalchemy.url = postgresql://anzhong:****@db.anzhong.game/anzhong

# 自动生成迁移 (从 model 变更检测)
alembic revision --autogenerate -m "add device_links table"

# 手动编写迁移 (复杂变更)
alembic revision -m "split player_progress.total_resets into per_chapter"

# 应用迁移 (生产)
alembic upgrade head

# 回滚迁移
alembic downgrade -1  # 回滚一个版本
alembic downgrade base  # 回滚到初始
```

### 2.2 迁移脚本目录结构

```
migrations/
├── alembic.ini
├── env.py
├── script.py.mako
└── versions/
    ├── 2026_07_01_0001_initial_schema_v2_0.py
    ├── 2026_07_15_0002_add_device_links.py
    ├── 2026_08_01_0003_add_leaderboard.py
    ├── 2026_08_15_0004_add_audit_logs.py
    └── 2026_09_01_0005_p0_001_difficulty_backfill.py  # P0-001 解决后
```

### 2.3 示例迁移脚本

```python
# migrations/versions/2026_07_01_0001_initial_schema_v2_0.py
"""initial schema v2.0

Revision ID: 2026_07_01_0001
Revises:
Create Date: 2026-07-01 00:00:00.000000

"""
from alembic import op
import sqlalchemy as sa
from sqlalchemy.dialects import postgresql

# revision identifiers
revision = '2026_07_01_0001'
down_revision = None
branch_labels = None
depends_on = None


def upgrade():
    # 1. players
    op.create_table('players',
        sa.Column('id', sa.BigInteger(), nullable=False),
        sa.Column('device_id', sa.String(100), nullable=False),
        sa.Column('display_name', sa.String(32), nullable=False),
        sa.Column('platform', sa.String(20), nullable=False),
        sa.Column('locale', sa.String(10), nullable=False, server_default='zh-CN'),
        sa.Column('created_at', sa.TIMESTAMP(timezone=True), nullable=False, server_default=sa.text('NOW()')),
        sa.Column('total_play_time_sec', sa.Integer(), nullable=False, server_default='0'),
        sa.Column('achievements_unlocked', postgresql.JSONB(), nullable=False, server_default='[]'),
        sa.Column('deleted_at', sa.TIMESTAMP(timezone=True), nullable=True),
        sa.Column('anonymized', sa.Boolean(), nullable=False, server_default=sa.text('FALSE')),
        # TODO P0-001: 等 02-v2 §13 AC-06 增补后启用
        sa.Column('total_difficulty_completed', sa.Integer(), nullable=True),
        sa.Column('max_difficulty_ever_played', sa.Integer(), nullable=True),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('device_id'),
        sa.CheckConstraint('length(display_name) BETWEEN 1 AND 32', name='ck_players_display_name'),
        sa.CheckConstraint("platform IN ('steam','mac','switch','ps5','xbox','ios','android','itch','anonymous')", name='ck_players_platform'),
    )
    op.create_index('idx_players_platform', 'players', ['platform'], postgresql_where=sa.text('deleted_at IS NULL'))
    op.create_index('idx_players_created', 'players', ['created_at'], postgresql_where=sa.text('deleted_at IS NULL'))

    # 2. sessions
    op.create_table('sessions',
        sa.Column('id', postgresql.UUID(), nullable=False, server_default=sa.text('gen_random_uuid()')),
        sa.Column('player_id', sa.BigInteger(), nullable=False),
        sa.Column('started_at', sa.TIMESTAMP(timezone=True), nullable=False, server_default=sa.text('NOW()')),
        sa.Column('ended_at', sa.TIMESTAMP(timezone=True), nullable=True),
        sa.Column('platform', sa.String(20), nullable=False),
        sa.Column('client_version', sa.String(20), nullable=False),
        sa.Column('state', sa.String(20), nullable=False, server_default='menu'),
        sa.Column('rooms_completed', sa.Integer(), nullable=False, server_default='0'),
        sa.Column('total_resets', sa.Integer(), nullable=False, server_default='0'),
        sa.Column('exit_reason', sa.String(20), nullable=True),
        sa.ForeignKeyConstraint(['player_id'], ['players.id'], ondelete='CASCADE'),
        sa.PrimaryKeyConstraint('id'),
        sa.CheckConstraint("state IN ('menu','chapter_select','room_playing','paused','win','chapter_transition')", name='ck_sessions_state'),
    )
    op.create_index('idx_sessions_player', 'sessions', ['player_id', sa.text('started_at DESC')])
    op.create_index('idx_sessions_active', 'sessions', ['player_id'], postgresql_where=sa.text('ended_at IS NULL'))

    # ... 其他 16 张表 (T03-T18)


def downgrade():
    op.drop_table('sessions')
    op.drop_table('players')
    # ... 反向 DROP
```

### 2.4 复杂迁移示例 (字段重命名 + 数据回填)

```python
# migrations/versions/2026_09_01_0005_p0_001_difficulty_backfill.py
"""P0-001 difficulty backfill (等 02-v2 §13 AC-06 解决后)

Revision ID: 2026_09_01_0005
Revises: 2026_08_15_0004
Create Date: 2026-09-01 00:00:00.000000

本迁移依赖 P0-001 解决 (02-v2 §13 AC-06 增补"难度上限 20"硬约束)。
如果 P0-001 未解决, 此迁移禁用 (condition: P0_001_RESOLVED)

"""
from alembic import op
import sqlalchemy as sa

revision = '2026_09_01_0005'
down_revision = '2026_08_15_0004'


def upgrade():
    # 1. players.total_difficulty_completed: NULL → 0 (回填默认值)
    op.execute("UPDATE players SET total_difficulty_completed = 0 WHERE total_difficulty_completed IS NULL")
    op.alter_column('players', 'total_difficulty_completed', nullable=False, server_default='0')

    # 2. rooms.difficulty_max: 去掉 CHECK 约束 (允许 02-v2 同步后调整)
    op.drop_constraint('ck_rooms_difficulty_max', 'rooms', type_='check')

    # 3. 新增 P0-001 解决后的正式字段
    op.add_column('scores', sa.Column('difficulty_official', sa.Integer(), nullable=True))

    # 4. 数据回填 (基于 save_data JSON 解析)
    op.execute("""
        UPDATE scores s
        SET difficulty_official = (s.properties->>'difficulty_official')::INTEGER
        WHERE s.properties->>'difficulty_official' IS NOT NULL
    """)


def downgrade():
    op.drop_column('scores', 'difficulty_official')
    op.alter_column('players', 'total_difficulty_completed', nullable=True)
    op.create_check_constraint('ck_rooms_difficulty_max', 'rooms', 'difficulty_max = 20')
```

### 2.5 零停机迁移策略

**原则：** 任何破坏性变更必须分 3 步：

```
Step 1 (向后兼容): 新增字段 (nullable) → 老代码继续运行
Step 2 (双写): 新代码双写老字段 + 新字段 → 灰度发布
Step 3 (切换): 数据回填完成 → 切换读取新字段 → 删除老字段 (下次发布)
```

**示例 (字段重命名 `play_time` → `total_play_time_sec`)：**

```python
# Step 1: 新增 total_play_time_sec (nullable)
def upgrade_step1():
    op.add_column('players', sa.Column('total_play_time_sec', sa.Integer(), nullable=True))

# Step 2: 双写 (应用层 + 触发器)
def upgrade_step2():
    # 应用层: 写入时同时写 play_time + total_play_time_sec
    # 数据库触发器: 同步老字段到新字段 (兜底)
    op.execute("""
        CREATE OR REPLACE FUNCTION sync_play_time()
        RETURNS TRIGGER AS $$
        BEGIN
            IF NEW.total_play_time_sec IS NULL THEN
                NEW.total_play_time_sec = NEW.play_time;
            END IF;
            RETURN NEW;
        END;
        $$ LANGUAGE plpgsql;
        
        CREATE TRIGGER trg_sync_play_time
        BEFORE INSERT OR UPDATE ON players
        FOR EACH ROW EXECUTE FUNCTION sync_play_time();
    """)

# Step 3: 数据回填 + 切换读取 (下一次发布)
def upgrade_step3():
    op.execute("UPDATE players SET total_play_time_sec = play_time WHERE total_play_time_sec IS NULL")
    op.alter_column('players', 'total_play_time_sec', nullable=False)
    # 下一次发布: 应用层切换读取 total_play_time_sec
    # 下下次发布: DROP COLUMN play_time
```

## 3. 客户端 EF Core SQLite 迁移 (v1.0)

### 3.1 EF Core 工作流

```bash
# 添加迁移
dotnet ef migrations add InitialCreate --project src/SaveSystem

# 应用迁移
dotnet ef database update --project src/SaveSystem

# 回滚迁移
dotnet ef migrations remove --project src/SaveSystem

# 生成 SQL 脚本 (生产部署)
dotnet ef migrations script --project src/SaveSystem --output migrate.sql
```

### 3.2 C# 迁移类示例

```csharp
// src/SaveSystem/Migrations/20260629120000_InitialCreate.cs
using Microsoft.EntityFrameworkCore.Migrations;

public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // local_player
        migrationBuilder.CreateTable(
            name: "local_player",
            columns: table => new
            {
                id = table.Column<int>(nullable: false).Annotation("Sqlite:Autoincrement", true),
                device_id = table.Column<string>(maxLength: 100, nullable: false),
                display_name = table.Column<string>(maxLength: 32, nullable: false),
                platform = table.Column<string>(maxLength: 20, nullable: false),
                locale = table.Column<string>(maxLength: 10, nullable: false, defaultValue: "zh-CN"),
                created_at = table.Column<string>(nullable: false, defaultValueSql: "datetime('now')"),
                total_play_time_sec = table.Column<int>(nullable: false, defaultValue: 0)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_local_player", x => x.id);
                table.UniqueConstraint("AK_local_player_device_id", x => x.device_id);
            });

        // save_data (单存档, CHECK id = 1)
        migrationBuilder.CreateTable(
            name: "save_data",
            columns: table => new
            {
                id = table.Column<int>(nullable: false, defaultValue: 1),
                version = table.Column<string>(maxLength: 10, nullable: false, defaultValue: "1.0.0"),
                last_updated = table.Column<string>(nullable: false, defaultValueSql: "datetime('now')"),
                current_chapter_id = table.Column<string>(maxLength: 10, nullable: false, defaultValue: "ch1"),
                current_room_id = table.Column<string>(maxLength: 10, nullable: false, defaultValue: "1-1"),
                completed_rooms_json = table.Column<string>(nullable: false, defaultValue: "{\"ch1\":[],\"ch2\":[],\"ch3\":[]}"),
                chapter_completed_json = table.Column<string>(nullable: false, defaultValue: "{\"ch1\":false,\"ch2\":false,\"ch3\":false}"),
                game_completed = table.Column<int>(nullable: false, defaultValue: 0),
                total_play_time_sec = table.Column<int>(nullable: false, defaultValue: 0),
                total_resets = table.Column<int>(nullable: false, defaultValue: 0),
                room_stats_json = table.Column<string>(nullable: false, defaultValue: "{}"),
                accessibility_settings_json = table.Column<string>(nullable: true),
                audio_settings_json = table.Column<string>(nullable: true),
                // TODO P0-001: 等 02-v2 §13 AC-06 增补后启用
                max_difficulty_completed = table.Column<int>(nullable: true),
                max_difficulty_ever = table.Column<int>(nullable: true),
                last_difficulty_played = table.Column<int>(nullable: true)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_save_data", x => x.id);
                table.CheckConstraint("ck_save_data_single", "id = 1");
                table.CheckConstraint("ck_save_max_difficulty_todo", "max_difficulty_completed IS NULL OR max_difficulty_completed BETWEEN 1 AND 20");
            });

        // ... 其他 7 张表
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable("save_data");
        migrationBuilder.DropTable("local_player");
    }
}
```

### 3.3 客户端启动时自动迁移

```csharp
// src/SaveSystem/SaveSystem.cs
public class SaveSystem
{
    public async Task InitializeAsync()
    {
        using var context = new AnzhongDbContext();

        // 自动应用所有 pending 迁移 (启动时)
        await context.Database.MigrateAsync();

        // 检查存档版本兼容
        var saveVersion = await GetSaveVersionAsync();
        if (saveVersion != null && saveVersion != CurrentVersion)
        {
            await MigrateSaveAsync(saveVersion, CurrentVersion);
        }
    }
}
```

## 4. 存档版本兼容 (Save Data Versioning)

### 4.1 版本号管理

存档版本用 **semver** (语义化版本):
- **MAJOR**: 不兼容老版本 (破坏性字段删除/类型变化)
- **MINOR**: 向后兼容 (新增字段/枚举)
- **PATCH**: 向后兼容 (bug fix)

| 版本 | 兼容性 | 处理 |
|------|:------:|------|
| **1.0.0** | 初始 | 默认 |
| **1.0.1** | 兼容 | 直接读取 |
| **1.1.0** | 兼容 | 读取 + 默认值填充新字段 |
| **2.0.0** | 不兼容 | 提示玩家从 Ch1-1 重新开始 |

### 4.2 存档版本迁移链

```
v1.0.0 (Day-84, 初始)
  ↓ MINOR +1 (新增 screen_reader)
v1.1.0 (T+3m, 多端云存档)
  ↓ MAJOR +1 (服务端引入, 字段重构)
v2.0.0 (T+6m, 全平台)
```

### 4.3 存档迁移实现 (C#)

```csharp
// src/SaveSystem/SaveMigration.cs
public class SaveMigration
{
    public SaveData Migrate(byte[] oldSaveData, string fromVersion, string toVersion)
    {
        SaveData save = Deserialize(oldSaveData);

        // v1.0.0 → v1.1.0 (新增字段, 默认值)
        if (fromVersion == "1.0.0" && toVersion.StartsWith("1.1."))
        {
            save.AccessibilitySettings ??= new AccessibilitySettings();
            save.AccessibilitySettings.ScreenReader = false;  // 默认 false (v1.1 新增)
            save.AudioSettings.SpatialAudio = false;  // 默认 false
            save.AccessibilitySettings.Difficulty = "normal";  // 默认 normal (hard 留给玩家选择)
        }

        // v1.1.0 → v2.0.0 (字段重构)
        if (fromVersion.StartsWith("1.1.") && toVersion.StartsWith("2.0."))
        {
            save.Version = "2.0.0";
            // max_difficulty_completed 字段从 save.max_difficulty_completed 保留
            // 跨设备 ID 由服务端生成, 客户端不存
        }

        save.Version = toVersion;
        return save;
    }

    public bool IsCompatible(string saveVersion, string clientVersion)
    {
        // MAJOR 必须匹配, MINOR/PATCH 兼容
        var saveParts = saveVersion.Split('.').Select(int.Parse).ToArray();
        var clientParts = clientVersion.Split('.').Select(int.Parse).ToArray();

        // 不同 MAJOR → 不兼容
        if (saveParts[0] != clientParts[0]) return false;

        // 老 MINOR 数据可在新 MINOR 运行 (向后兼容)
        if (saveParts[1] <= clientParts[1]) return true;

        // 新 MINOR 数据不可在老 MINOR 运行 (前向不兼容)
        return false;
    }
}
```

### 4.4 存档版本不兼容处理

```csharp
// 存档 MAJOR 版本不匹配 → 提示玩家
public async Task<LoadResult> LoadSaveAsync()
{
    var saveData = await ReadFromDiskAsync();
    var saveVersion = ExtractVersion(saveData);

    if (!IsCompatible(saveVersion, CurrentVersion))
    {
        // 不兼容 (MAJOR 不匹配)
        return new LoadResult
        {
            Success = false,
            ErrorCode = "SAVE_VERSION_INCOMPATIBLE",
            Message = $"存档版本 {saveVersion} 不兼容当前游戏版本 {CurrentVersion}。\n是否从 Ch1-1 重新开始？",
            Suggestion = "restart_from_chapter_1"
        };
    }

    // 兼容 → 迁移
    var migrated = Migrate(saveData, saveVersion, CurrentVersion);
    return new LoadResult { Success = true, SaveData = migrated };
}
```

### 4.5 存档版本矩阵

| 从 \ 到 | 1.0.0 | 1.0.1 | 1.1.0 | 2.0.0 |
|:------:|:-----:|:-----:|:-----:|:-----:|
| **1.0.0** | ✅ | ✅ 自动迁移 | ✅ 自动迁移 | ❌ 不兼容 |
| **1.0.1** | ❌ 前向不兼容 | ✅ | ✅ 自动迁移 | ❌ 不兼容 |
| **1.1.0** | ❌ | ❌ | ✅ | ✅ 自动迁移 |
| **2.0.0** | ❌ | ❌ | ❌ | ✅ |

**自动迁移：** 同一 MAJOR 内 MINOR/PATCH 变化 → 自动应用迁移函数
**不兼容：** MAJOR 变化 → 提示玩家重置进度 (数据导出备份)

## 5. 蓝绿部署 (Blue-Green Deployment)

### 5.1 架构

```
┌──────────────┐
│ Load Balancer│
└──────┬───────┘
       │
       ├──── Blue (v1.x, 老版本)
       │     └─ PostgreSQL v1.x schema
       │
       └──── Green (v2.x, 新版本)
             └─ PostgreSQL v2.x schema (新)
```

**步骤：**
1. Green 环境部署新代码 + 新 schema
2. 数据库迁移 (Alembic upgrade)
3. 流量切到 Green (5% → 25% → 50% → 100%)
4. 监控错误率 / 延迟
5. Blue 环境保留 1 周 (快速回滚)
6. 1 周后销毁 Blue

### 5.2 灰度策略

```yaml
# k8s-canary.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: anzhong-api
spec:
  http:
    - route:
        - destination:
            host: anzhong-api-blue
          weight: 90  # 90% 流量到老版本
        - destination:
            host: anzhong-api-green
          weight: 10  # 10% 流量到新版本 (灰度)
```

## 6. 数据迁移脚本 (Data Migration Scripts)

### 6.1 v1.0 → v1.1 客户端数据迁移

```csharp
// src/SaveSystem/Migrations/Client/v1_0_to_v1_1.cs
public static class Client_v1_0_to_v1_1
{
    public static void MigrateSave(SaveDataV1_0 oldSave)
    {
        var newSave = new SaveDataV1_1
        {
            // 复制老字段
            Version = "1.1.0",
            CurrentChapterId = oldSave.CurrentChapterId,
            CurrentRoomId = oldSave.CurrentRoomId,
            CompletedRooms = oldSave.CompletedRooms,
            ChapterCompleted = oldSave.ChapterCompleted,
            GameCompleted = oldSave.GameCompleted,
            TotalPlayTimeSec = oldSave.TotalPlayTimeSec,
            TotalResets = oldSave.TotalResets,
            RoomStats = oldSave.RoomStats,
            
            // 新字段默认值
            AccessibilitySettings = new AccessibilitySettings
            {
                ColorblindMode = "default",
                FontScale = 1.0f,
                ControllerSupport = "xbox",
                Difficulty = "normal",
                HighContrast = false,
                ReducedMotion = false,
                ScreenReader = false  // v1.1 新增
            },
            AudioSettings = oldSave.AudioSettings,  // 复用
            MaxDifficultyCompleted = null  // TODO P0-001
        };

        SaveNew(newSave);
    }
}
```

### 6.2 v1.1 → v2.0 服务端数据导入

```python
# tools/migrate_v1_1_to_v2_0.py
"""
v1.1 客户端存档 → v2.0 PostgreSQL 导入脚本
玩家首次登录 v2.0 时, 客户端上传 v1.1 存档, 服务端执行此脚本
"""
import asyncio
from sqlalchemy.ext.asyncio import AsyncSession

async def import_v1_1_to_v2_0(player_id: int, v1_1_save_data: dict, db: AsyncSession):
    # 1. 导入 saves
    save = Save(
        player_id=player_id,
        version='2.0.0',
        current_chapter_id=v1_1_save_data['currentChapterId'],
        current_room_id=v1_1_save_data['currentRoomId'],
        completed_rooms=v1_1_save_data['completedRooms'],
        chapter_completed=v1_1_save_data['chapterCompleted'],
        game_completed=v1_1_save_data['gameCompleted'],
        total_play_time_sec=v1_1_save_data['totalPlayTimeSec'],
        total_resets=v1_1_save_data['totalResets'],
        room_stats=v1_1_save_data['roomStats'],
        # TODO P0-001: 等 02-v2 AC-06 同步后, 从 saveData 计算 max_difficulty
        max_difficulty_completed=None  # 默认 NULL
    )
    db.add(save)

    # 2. 导入 player_audio_settings
    if 'audioSettings' in v1_1_save_data:
        audio = PlayerAudioSettings(player_id=player_id, **v1_1_save_data['audioSettings'])
        db.add(audio)

    # 3. 导入 player_accessibility_settings
    if 'accessibilitySettings' in v1_1_save_data:
        acc = PlayerAccessibilitySettings(player_id=player_id, **v1_1_save_data['accessibilitySettings'])
        db.add(acc)

    # 4. 导入 room_stats → scores (每个通关房间 1 行)
    for room_id, stats in v1_1_save_data.get('roomStats', {}).items():
        if stats.get('completionTimeSec'):
            score = Score(
                player_id=player_id,
                room_id=room_id,
                session_id=None,  # v1.1 离线通关无 session_id
                steps=stats.get('steps', 0),
                time_sec=stats['completionTimeSec'],
                hints_used=stats.get('hintsTriggered', 0),
                resets_used=stats.get('resets', 0),
                completed_at=datetime.fromtimestamp(stats.get('lastPlayedAt', 0) / 1000)
            )
            db.add(score)

    await db.commit()
```

## 7. 监控与告警 (Monitoring & Alerts)

### 7.1 迁移监控指标

| 指标 | 目标 | 告警阈值 |
|------|------|---------|
| **migration_duration_sec** | < 60s | > 300s (PagerDuty) |
| **migration_failure_count** | 0 | > 0 (PagerDuty) |
| **save_load_failure_rate** | < 0.1% | > 1% (Slack) |
| **save_migration_rate** | 100% (MAJOR) | < 100% (Slack) |
| **schema_drift_count** | 0 | > 0 (PagerDuty) |

### 7.2 回滚 SOP

```bash
# 1. 检测到迁移失败 (5xx 错误率突增)
kubectl logs -l app=anzhong-api | grep "MigrationError"

# 2. 立即回滚代码
kubectl rollout undo deployment/anzhong-api

# 3. 回滚数据库 (Alembic)
alembic downgrade -1

# 4. 验证恢复
curl https://api.anzhong.game/health

# 5. 通知团队
slack-notify "#anzhong-ops" "Migration rolled back: <commit_hash>"
```

## 8. 关联文档 (Cross-References)

- [`README.md`](./README.md) — 数据架构总览
- [`persistence-strategy.md`](./persistence-strategy.md) — 持久化策略
- [`database-schema.md`](./database-schema.md) — 18 表 schema
- [`cache-strategy.md`](./cache-strategy.md) — Redis 缓存
- [`serialization.md`](./serialization.md) — 序列化格式
- [`backup-and-recovery.md`](./backup-and-recovery.md) — 3-2-1 备份
- [`p0-001-tracking.md`](./p0-001-tracking.md) — P0-001 跟踪
- [`../../docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — 4 阶段路线图

## 9. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** 3 阶段演进 + Alembic 服务端迁移 + EF Core 客户端迁移 + 存档版本兼容 (semver + 迁移链) + 零停机 (蓝绿 + 双写 + 灰度) + 复杂迁移示例 (字段重命名 + 数据回填) + P0-001 迁移禁用条件 + 回滚 SOP + 监控告警。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）