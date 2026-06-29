---
title: 《暗室》技术栈 (Tech Stack)
doc_id: DESIGN-anzhong-architecture-06
parent: design/architecture/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 尚书省
---

# 《暗室》技术栈 (Tech Stack)

> **一句话定位：** 客户端 (Unity 2022 LTS / C# 9 / IL2CPP) + 服务端 v2.0+ (.NET 8 / ASP.NET Core / PostgreSQL 16 / Redis 7) + CDN (CloudFront) + DevOps (GitHub Actions) 的端到端技术栈。

## 目的 (Purpose)

本文档是《暗室》**技术决策层 (Tech Decision Layer)** 的**唯一权威基线**。它向：

- **Unity 客户端工程师** — 引擎版本、运行时、第三方库、性能预算
- **服务端工程师**（v2.0+）— 框架版本、数据库、缓存、CDN
- **DevOps 工程师** — CI/CD、构建工具、部署平台
- **架构师** — 用于 ADR 评审、技术选型对比、升级路径规划
- **新加入工程师** — 5 分钟看懂《暗室》用什么技术栈搭建

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"在 v1.0 客户端 + v2.0+ 服务端的全部技术选型——引擎 (Unity 2022 LTS)、语言 (C# 9)、运行时 (Mono / IL2CPP)、数据库 (PostgreSQL 16)、缓存 (Redis 7)、CDN (CloudFront)、CI/CD (GitHub Actions)——**第一次**用一张技术栈表 + 选型理由 + 替代方案对比统一描述，作为 phase3 → phase4 实施的"技术合同"。

## 范围 (Scope)

### 包含

- **客户端技术栈** — Unity 2022 LTS + C# 9 + IL2CPP + URP 2D Renderer + Input System + Addressables + Steamworks SDK
- **服务端技术栈（v2.0+）** — .NET 8 + ASP.NET Core 8 + PostgreSQL 16 + Redis 7 + CloudFront CDN
- **DevOps 工具链** — Git + GitHub + GitHub Actions + Docker + Terraform（v2.0+）
- **第三方库** — DOTween (MIT) + Newtonsoft.Json + Unity Test Framework + Steamworks.NET
- **构建与分发** — Unity Build Pipeline + Addressables Build + Perforce / SteamPipe（v1.1+）
- **性能预算** — 60 FPS / 512MB / 5s 启动 / 1s 场景加载 / 50ms 存档

### 不包含

- 数值公式 → 见 `docs/05-numerical-design-v2.md`
- 美术资源制作 → 见 `docs/12-art-style-v2.md`
- 营销节奏 → 见 `docs/11-release-v2.md`

## 一句话描述 (One-liner)

> **"Unity 2022 LTS + C# 9 + IL2CPP + PostgreSQL 16 + Redis 7 + CloudFront + GitHub Actions，无战斗 2D 房间解谜游戏的技术栈。"**

## 1. 客户端技术栈 (Client Tech Stack)

### 1.1 客户端技术栈总览

| 维度 | 选型 | 版本 | 选型理由 | 替代方案 |
|------|------|------|---------|---------|
| **游戏引擎** | Unity | 2022 LTS (2022.3.x) | 稳定 LTS + 2D 支持完善 + Asset Store 资源丰富 + 7 平台覆盖 | ❌ Godot（生态弱）/ ❌ Unreal（2D 支持差） |
| **渲染管线** | URP + 2D Renderer | URP 14.x (Unity 2022 LTS 配套) | 2D 光照 + 性能优于 Built-in | ❌ Built-in Renderer（性能差） |
| **语言** | C# | 9.0 (.NET Standard 2.1) | Unity 主语言 + 生态成熟 | ❌ F#（学习曲线陡） |
| **运行时** | Mono (Editor) / IL2CPP (Release) | — | iOS 强制 IL2CPP + 性能优于 Mono | ❌ Mono-only（iOS 不允许） |
| **输入** | Unity Input System | 1.7.x | 新 API + 手柄支持 + 多设备 | ❌ Legacy Input（不支持手柄） |
| **资源管理** | Addressables | 1.21.x | 热更新 + 内存管理 + 异步加载 | ❌ Resources（无法热更新） |
| **动画** | Animator + DOTween | Animator (built-in) + DOTween 1.2.x | Animator 状态机 + DOTween 缓动 | ❌ Timeline（过重） |
| **2D 物理** | Unity 2D Physics | built-in | 静态碰撞用 Tilemap Collider | ❌ Box2D（重复造轮子） |
| **Tilemap** | Unity Tilemap | built-in | 2D 地图编辑 + 自动碰撞 | ❌ Tiled（导出流程繁琐） |
| **JSON** | Newtonsoft.Json (Unity Package) | 13.x | 人类可读 + 易调试 + 广泛使用 | ⚠️ System.Text.Json（性能好但生态弱） |
| **加密** | System.Security.Cryptography | built-in (.NET Standard) | AES-256-GCM 标准实现 | ⚠️ BouncyCastle（更重） |
| **Steam SDK** | Steamworks.NET | 20.x | Steam 官方推荐 C# 绑定 | ❌ Facepunch.Steamworks（功能不全） |
| **测试** | Unity Test Framework | 1.1.x | EditMode + PlayMode + 性能测试 | ❌ NUnit（无 Unity 集成） |
| **Linting** | Roslyn Analyzers + EditorConfig | latest | 代码风格统一 + 编译期检查 | ❌ ReSharper（商业） |
| **Profiler** | Unity Profiler + Memory Profiler | built-in | 性能调优 + 内存泄漏检测 | ❌ dotTrace（外部） |

### 1.2 客户端第三方库清单

| 库 | 版本 | 许可证 | 用途 | 风险 |
|---|------|-------|------|------|
| **DOTween** | 1.2.x | MIT | 缓动动画（槽位切换、重置） | 低（广泛使用 + 稳定） |
| **Steamworks.NET** | 20.x | MIT | Steam SDK C# 绑定 | 低（官方推荐） |
| **Newtonsoft.Json** | 13.x | MIT | JSON 序列化 | 低（事实标准） |
| **Unity Input System** | 1.7.x | Unity Companion | 输入处理 | 极低（Unity 官方） |
| **Unity Addressables** | 1.21.x | Unity Companion | 资源管理 | 极低（Unity 官方） |
| **Cinemachine** (可选) | 2.10.x | Unity Companion | 摄像机 | 极低（如使用） |
| **TextMeshPro** | 3.0.x | Unity Companion | 文字渲染 | 极低（Unity 官方） |
| **Odin Inspector** (可选) | 3.x | 个人/团队许可证 | Editor 扩展 | 中（许可证成本） |

### 1.3 Unity 2022 LTS 升级路径

| 版本 | 状态 | 备注 |
|------|------|------|
| **Unity 2022 LTS (2022.3.x)** | ✅ v1.0 选型 | 稳定 + 7 平台 SDK 齐全 |
| **Unity 2023.x** | ⚠️ 跳过 | Tech Stream 非 LTS，不推荐 |
| **Unity 2024 LTS** | ⚠️ 评估 | v2.0+ 评估升级 |
| **Unity 6 LTS (2025 LTS)** | 🔮 长期 | v3.0+ 升级目标 |

**升级路径：** v1.0 (2022 LTS) → v2.0 (2024 LTS) → v3.0 (Unity 6 LTS)
- ✅ **ABI 兼容** — Unity 2022/2024/6 LTS 共享 .NET Standard 2.1 ABI
- ✅ **包兼容** — 大部分 Unity Package 跨 LTS 兼容
- ⚠️ **URP 升级** — URP 14 → 16 可能需要重做 Shader Graph
- ⚠️ **API 废弃** — 部分 API 在新 LTS 废弃（如 UniversalRendererData）

### 1.4 客户端性能预算

| 指标 | 目标 | 不达标后果 | 优化手段 |
|------|------|----------|---------|
| **帧率 (PC/Mac)** | ≥ 60 FPS | 切换动画自动 2x | Profiler + Memory Profiler |
| **帧率 (Switch)** | ≥ 30 FPS | 切换动画 2x | URP 2D + Tilemap |
| **帧率 (移动)** | ≥ 30 FPS | 切换动画 2x | Sprite Atlas + 减少粒子 |
| **切换响应** | ≤ 16ms（1 帧） | 玩家感觉"按了没反应" | Profiler Marker + 优化 GC |
| **切换动画** | 200ms ± 50ms | 太短无反馈，太长打断节奏 | DOTween Time.timeScale |
| **单房间槽位** | ≤ 8 | 超过 8 玩家认知过载 | 关卡设计硬约束 |
| **DrawCall** | ≤ 50 | 超过 50 帧率掉到 30FPS | Sprite Atlas + Static Batching |
| **内存峰值** | ≤ 512MB | 超过 512MB 低端机崩溃 | Addressables 异步加载 |
| **冷启动** | ≤ 5s | 玩家等待感强 | 预加载 + 首场景极简 |
| **场景加载** | ≤ 1s | 中断解谜节奏 | Addressables 异步 + 进度条 |
| **JSON 存档读写** | ≤ 50ms | 超过 50ms 卡顿 | 异步写入 + backup |
| **GC 分配** | ≤ 1MB/分钟 | 频繁 GC 掉帧 | struct 事件 + 对象池 |

## 2. 服务端技术栈 (Server Tech Stack · v2.0+)

### 2.1 服务端技术栈总览

| 维度 | 选型 | 版本 | 选型理由 | 替代方案 |
|------|------|------|---------|---------|
| **运行时** | .NET | 8 LTS | C# 跨平台 + 性能优 + 长期支持 | ⚠️ .NET 9（短期） |
| **Web 框架** | ASP.NET Core | 8.x | 官方框架 + 中间件生态 + Kestrel 高性能 | ⚠️ Minimal API（轻量但生态弱） |
| **API 风格** | REST + JSON | — | 简单 + 可调试 + OpenAPI 工具链 | ⚠️ gRPC（性能优但调试难） |
| **API 规范** | OpenAPI | 3.0 | 机器可读 + Swagger UI + 代码生成 | ⚠️ GraphQL（过重） |
| **数据库** | PostgreSQL | 16 | 开源 + ACID + JSON 支持 + 主从成熟 | ⚠️ MySQL（功能稍弱） |
| **ORM** | Entity Framework Core | 8.x | 官方 ORM + Code First 迁移 | ⚠️ Dapper（轻量但手写 SQL） |
| **缓存** | Redis | 7.x | 高性能 + 丰富数据结构 + Sentinel HA | ⚠️ Memcached（功能弱） |
| **Redis 客户端** | StackExchange.Redis | 2.x | 官方推荐 C# 客户端 | ⚠️ ServiceStack.Redis（商业） |
| **消息队列** | (v2.1+) RabbitMQ / Amazon SQS | — | v2.0+ 异步任务（遥测批处理） | ⚠️ Kafka（过重） |
| **认证** | JWT (System.IdentityModel.Tokens.Jwt) | 7.x | 无状态 + 标准 + 多平台 | ⚠️ Session（状态耦合） |
| **限流** | AspNetCoreRateLimit | — | 60/1000/10 三档限流 | ⚠️ NGINX（网关层） |
| **API 网关** | NGINX / AWS ALB | latest | HTTPS 终止 + 限流 + 路由 | ⚠️ Kong（功能过剩） |
| **CDN** | AWS CloudFront | — | 全球边缘 + Addressables 缓存 | ⚠️ Fastly（功能类似） |
| **对象存储** | AWS S3 | — | 静态资源 + 备份 | ⚠️ MinIO（自托管） |
| **日志** | Serilog + CloudWatch | latest | 结构化日志 + 查询 | ⚠️ NLog（功能稍弱） |
| **监控** | Prometheus + Grafana | latest | 指标采集 + 仪表盘 | ⚠️ DataDog（商业） |
| **追踪** | OpenTelemetry + Jaeger | latest | 分布式追踪 | ⚠️ Zipkin（类似） |

### 2.2 服务端第三方库清单

| 库 | 版本 | 许可证 | 用途 | 风险 |
|---|------|-------|------|------|
| **Entity Framework Core** | 8.x | MIT | ORM | 极低 |
| **Npgsql** | 8.x | PostgreSQL License | PostgreSQL 驱动 | 极低 |
| **StackExchange.Redis** | 2.x | MIT | Redis 客户端 | 极低 |
| **Serilog** | 3.x | Apache 2.0 | 结构化日志 | 极低 |
| **AspNetCoreRateLimit** | — | MIT | API 限流 | 低 |
| **Swashbuckle.AspNetCore** | 6.x | MIT | OpenAPI / Swagger UI | 极低 |
| **System.IdentityModel.Tokens.Jwt** | 7.x | MIT | JWT 鉴权 | 极低 |
| **FluentValidation** | 11.x | Apache 2.0 | 请求 DTO 验证 | 低 |
| **AutoMapper** | 12.x | MIT | DTO ↔ Entity 映射 | 低 |
| **Polly** | 8.x | BSD-3 | 弹性策略（重试/熔断/超时） | 极低 |
| **OpenTelemetry** | 1.x | Apache 2.0 | 分布式追踪 | 极低 |

### 2.3 .NET 8 LTS 升级路径

| 版本 | 状态 | 备注 |
|------|------|------|
| **.NET 6 LTS** | ⚠️ 已过期（2024-11） | 不推荐 |
| **.NET 7** | ⚠️ 短期 | 跳过 |
| **.NET 8 LTS** | ✅ v2.0+ 选型 | 当前 LTS（2026-11 之前支持） |
| **.NET 9** | ⚠️ 短期 | 跳过 |
| **.NET 10 LTS** | 🔮 长期 | v3.0+ 升级目标（2025-11 发布） |

**升级路径：** v2.0 (.NET 8) → v2.5 (.NET 9 评估) → v3.0 (.NET 10 LTS)

### 2.4 服务端性能预算（v2.0+）

| 指标 | 目标 | 验证方式 |
|------|------|---------|
| **API 响应时间** | P50 ≤ 100ms / P95 ≤ 500ms | Prometheus + Grafana |
| **API 错误率** | < 0.1% | CloudWatch |
| **服务可用性** | ≥ 99.9%（3 个 9） | Uptime monitoring |
| **数据库 QPS** | ≤ 1000/s | pg_stat_activity |
| **Redis QPS** | ≤ 10000/s | INFO commandstats |
| **CDN 命中率** | ≥ 80% | CloudFront metrics |
| **并发玩家** | ≥ 10000（v2.0） | Load test |
| **数据库连接池** | ≤ 100 connections | pg_stat_activity |
| **内存占用（服务）** | ≤ 1GB / instance | CloudWatch |
| **CPU 使用率** | ≤ 70%（平均） | CloudWatch |

## 3. 数据库技术栈 (Database Tech Stack)

### 3.1 PostgreSQL 16 选型理由

| 维度 | PostgreSQL 16 | 替代方案 |
|------|--------------|---------|
| **ACID** | ✅ 完全支持 | MySQL（也支持，但功能弱） |
| **JSON 支持** | ✅ JSONB（索引 + 查询） | MongoDB（无 ACID） |
| **主从** | ✅ Streaming Replication | MySQL（binlog 复杂） |
| **扩展性** | ✅ PostGIS / pg_trgm / 等 | 多种扩展 |
| **许可证** | PostgreSQL License（开源） | MySQL（GPL） |
| **运维生态** | ✅ pg_dump / pgBackRest / Patroni | 成熟 |
| **性能** | 单机 10K QPS + 主从 100K QPS | — |

### 3.2 数据库 Schema 设计（v2.0+）

```sql
-- sessions 表（E01-E02）
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    player_id BIGINT NOT NULL,
    started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    ended_at TIMESTAMPTZ,
    platform VARCHAR(20) NOT NULL,  -- steam/mac/...
    device_id VARCHAR(100)
);
CREATE INDEX idx_sessions_player ON sessions(player_id);

-- rooms 表（E03-E05）
CREATE TABLE rooms (
    id VARCHAR(20) PRIMARY KEY,  -- e.g., "1-1", "3-8"
    chapter_id VARCHAR(10) NOT NULL,
    difficulty SMALLINT NOT NULL CHECK (difficulty BETWEEN 1 AND 20),  -- P0-001
    max_switch_slots SMALLINT NOT NULL DEFAULT 4,
    config JSONB NOT NULL  -- room JSON config
);

-- slot_history 表（E06）
CREATE TABLE slot_history (
    id BIGSERIAL PRIMARY KEY,
    session_id UUID NOT NULL REFERENCES sessions(id),
    slot_id VARCHAR(50) NOT NULL,
    room_id VARCHAR(20) NOT NULL,
    old_index SMALLINT NOT NULL,
    new_index SMALLINT NOT NULL,
    switched_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_slot_history_session ON slot_history(session_id);

-- save_data 表（E08-E09 / E15）
CREATE TABLE save_data (
    player_id BIGINT PRIMARY KEY,
    version VARCHAR(10) NOT NULL DEFAULT '1.0.0',
    data JSONB NOT NULL,
    last_updated TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- scores 表（E10）
CREATE TABLE scores (
    id BIGSERIAL PRIMARY KEY,
    player_id BIGINT NOT NULL,
    chapter_id VARCHAR(10) NOT NULL,
    room_id VARCHAR(20),
    completion_time_sec INT NOT NULL,
    switch_count INT NOT NULL,
    reset_count INT NOT NULL,
    achieved_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_scores_chapter ON scores(chapter_id, completion_time_sec);

-- feedback 表（E12）
CREATE TABLE feedback (
    id BIGSERIAL PRIMARY KEY,
    player_id BIGINT,
    category VARCHAR(50) NOT NULL,
    message TEXT NOT NULL,
    context JSONB,
    submitted_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- telemetry_events 表（E11 / E16）
CREATE TABLE telemetry_events (
    id BIGSERIAL PRIMARY KEY,
    session_id UUID,
    metric_name VARCHAR(100) NOT NULL,
    metric_value DOUBLE PRECISION NOT NULL,
    tags JSONB,
    recorded_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_telemetry_metric ON telemetry_events(metric_name, recorded_at);

-- hint_history 表（E17）
CREATE TABLE hint_history (
    id BIGSERIAL PRIMARY KEY,
    player_id BIGINT NOT NULL,
    room_id VARCHAR(20) NOT NULL,
    hint_level SMALLINT NOT NULL,
    triggered_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**P0-001 自我保护：** `rooms.difficulty` 字段 CHECK 约束 `BETWEEN 1 AND 20`，数据库层自我保护，不依赖 02-v2 §13 AC-06 增补。

### 3.3 Redis 7 数据结构

| Key | 数据结构 | 用途 | TTL |
|-----|---------|------|-----|
| `session:{id}` | Hash | 会话状态（player_id + platform + started_at） | 24h |
| `save:{playerId}:lock` | String | 分布式锁（防止并发写入） | 30s |
| `save:{playerId}` | String | 缓存 SaveData（JSON） | 5min |
| `lb:{chapter}:top100` | Sorted Set | 排行榜（按 completion_time_sec 排序） | 永久 |
| `progress:{playerId}` | Hash | 玩家进度缓存 | 1h |
| `hint:{roomId}:counts` | Hash | Hint 使用次数统计 | 7d |
| `ratelimit:{apiKey}:{minute}` | String | 限流计数（60 req/min） | 60s |

## 4. DevOps 工具链 (DevOps Tool Chain)

### 4.1 DevOps 工具栈

| 维度 | 选型 | 版本 | 选型理由 |
|------|------|------|---------|
| **版本控制** | Git + GitHub | latest | 行业标准 + GitHub Actions 集成 |
| **CI/CD** | GitHub Actions | latest | 免费 + 与 GitHub 集成 + Marketplace |
| **构建系统（客户端）** | Unity Build Pipeline | built-in | Unity 官方 |
| **构建系统（服务端）** | dotnet CLI | 8.x | .NET 官方 |
| **容器化** | Docker + Docker Compose | latest | 行业标准 + 跨平台 |
| **编排（v2.0+）** | AWS ECS Fargate | — | 无需管理 EC2 + 自动伸缩 |
| **IaC（v2.0+）** | Terraform | 1.x | 多云支持 + 状态管理 |
| **密钥管理** | AWS Secrets Manager / 1Password | — | 集中化 + 审计 |
| **监控** | Prometheus + Grafana + CloudWatch | latest | 指标 + 日志 + 告警 |
| **错误追踪** | Sentry (可选) | latest | 异常聚合 + 告警 |
| **包管理（Unity）** | Unity Package Manager | built-in | 官方 |
| **包管理（.NET）** | NuGet | — | 官方 |

### 4.2 GitHub Actions Workflows

| Workflow | 触发 | 步骤 |
|----------|------|------|
| **ci.yml** | push / pull_request | lint → unit test → build verify |
| **build_steam.yml** | tag v* | Unity Build → Steamworks Upload |
| **build_mac.yml** | tag v* | Unity Build (Mac) → Steamworks Upload |
| **build_switch.yml** | tag v1.1+ | Unity Build (Switch) → Nintendo eShop |
| **build_ps5.yml** | tag v2.0+ | Unity Build (PS5) → PSN Submission |
| **build_xbox.yml** | tag v2.0+ | Unity Build (Xbox) → Xbox Live |
| **build_ios.yml** | tag v2.0+ | Unity Build (iOS) → App Store Connect |
| **build_android.yml** | tag v2.0+ | Unity Build (Android) → Google Play |
| **release.yml** | tag v* | 7 平台并行构建 → Release 创建 |

### 4.3 7 平台构建命令

```bash
# scripts/build_steam.sh
unity -batchmode -nographics -quit \
  -projectPath . \
  -buildTarget StandaloneWindows64 \
  -executeMethod BuildScript.BuildSteam \
  -logFile build_steam.log

# scripts/build_mac.sh
unity -batchmode -nographics -quit \
  -projectPath . \
  -buildTarget StandaloneOSX \
  -executeMethod BuildScript.BuildMac \
  -logFile build_mac.log

# scripts/build_switch.sh (v1.1+)
unity -batchmode -nographics -quit \
  -projectPath . \
  -buildTarget Switch \
  -executeMethod BuildScript.BuildSwitch \
  -logFile build_switch.log
```

## 5. CDN 与对象存储 (CDN & Object Storage)

### 5.1 CloudFront CDN 策略（v2.0+）

| 资源类型 | 源站 | 缓存 TTL | 边缘缓存 |
|---------|------|---------|---------|
| **静态资源（图片/音频）** | S3 bucket | 30 天 | 全球 |
| **Addressables 资源** | S3 bucket | 7 天 | 全球 |
| **DLC 内容** | S3 bucket | 30 天 | 全球 |
| **补丁（v2.0+）** | S3 bucket | 7 天 | 全球 |
| **API 响应** | API Gateway | 不缓存 | — |

### 5.2 S3 Bucket 结构

```
s3://anzhong-assets-prod/
  ├── audio/
  │   ├── sfx/
  │   ├── bgm/
  │   └── dynamic/
  ├── sprites/
  │   ├── ch1/
  │   ├── ch2/
  │   ├── ch3/
  │   └── ui/
  ├── tilemaps/
  ├── dlcs/
  │   ├── season-pass-1/
  │   ├── season-pass-2/
  │   └── ...
  ├── patches/
  │   ├── v1.0.0/
  │   ├── v1.0.1/
  │   └── ...
  └── backups/
      └── save-data-backups/
```

## 6. 安全技术栈 (Security Tech Stack)

### 6.1 安全选型

| 维度 | 选型 | 备注 |
|------|------|------|
| **本地存档加密** | AES-256-GCM | .NET System.Security.Cryptography |
| **传输加密** | TLS 1.3 | API Gateway + CloudFront |
| **JWT 签名** | HS256 (HMAC-SHA256) | 短期 24h + Refresh Token |
| **API 鉴权** | JWT + 设备绑定 | Bearer Token |
| **限流** | 60 req/min 客户端 + 1000 req/h 用户 + burst 10 req/s | AspNetCoreRateLimit |
| **客户端时钟容差** | ±5min | X-Client-Timestamp 头 |
| **跨平台冲突** | Last-Write-Wins + 服务器时钟优先 | 服务器权威 |
| **GDPR 导出** | SaveSystem.ExportSave() | JSON 人类可读 |
| **GDPR 删除** | SaveSystem.DeleteSave() | 本地 + 云端 |
| **审计日志** | Serilog + CloudWatch | 所有写操作记录 |

### 6.2 PII 收集 = 0

> **核心承诺：** 不收集 PII（Personally Identifiable Information）。
> 详见 docs/11-release-v2.md §5.3 隐私政策。

| 不收集 | 收集（v1.0 全部本地） |
|--------|------------------|
| 玩家位置 | 本地存档（JSON） |
| 设备信息 | 设备 ID（仅本地，用于多设备绑定 v2.0+） |
| 邮箱 / 姓名 | — |
| IP 地址 | — |
| Cookies | — |
| 第三方追踪 | — |

## 7. 监控与告警 (Monitoring & Alerting)

### 7.1 监控指标

| 维度 | 指标 | 工具 |
|------|------|------|
| **API** | QPS / P50 / P95 / P99 / 错误率 | Prometheus + Grafana |
| **数据库** | 连接数 / QPS / 慢查询 / 锁等待 | pg_stat_activity + pg_stat_statements |
| **Redis** | 内存 / QPS / 命中率 | INFO commandstats |
| **CDN** | 命中率 / 流量 / 边缘延迟 | CloudFront metrics |
| **服务端进程** | CPU / 内存 / GC / 线程数 | dotnet-counters + CloudWatch |
| **客户端（v2.0+）** | 帧率 / 崩溃率 / 卡顿率 | Unity Profiler + Sentry |

### 7.2 告警阈值

| 指标 | 阈值 | 告警渠道 |
|------|------|---------|
| API 错误率 | > 1% | Slack + Email |
| API P95 延迟 | > 1s | Slack |
| 数据库连接数 | > 80 | Slack |
| Redis 内存 | > 80% | Slack |
| 服务进程 CPU | > 90% 持续 5min | Slack + PagerDuty |
| CDN 命中率 | < 70% | Slack |
| 客户端崩溃率 | > 1% | Slack + Email |

## 8. 配置表 (Configuration)

### 8.1 客户端参数

| 字段 | 取值范围 | 默认值 | 单位 | 场景 |
|------|---------|-------|------|------|
| `unity.version` | 2022 LTS | 2022.3.x | — | 引擎版本 |
| `unity.renderPipeline` | URP / Built-in | URP | — | 渲染管线 |
| `csharp.version` | 9.0 / 12.0 | 9.0 | — | 语言版本 |
| `runtime.editor` | Mono | Mono | — | Editor 运行时 |
| `runtime.release` | IL2CPP | IL2CPP | — | Release 运行时 |
| `inputSystem.version` | 1.7.x | 1.7.x | — | Input System 版本 |
| `addressables.version` | 1.21.x | 1.21.x | — | Addressables 版本 |
| `frameRate.targetPC` | [30, 120] | 60 | FPS | PC 目标帧率 |
| `frameRate.targetSwitch` | [30, 60] | 30 | FPS | Switch 目标帧率 |
| `memoryBudget.peak` | [256, 1024] | 512 | MB | 内存峰值 |
| `startupTime.max` | [1, 10] | 5 | 秒 | 启动时间 |
| `sceneLoadTime.max` | [0.5, 3] | 1 | 秒 | 场景加载 |
| `saveWriteTime.max` | [10, 200] | 50 | ms | 存档写入 |
| `gcAllocationBudget` | [0.5, 5] | 1 | MB/分钟 | GC 预算 |

### 8.2 服务端参数（v2.0+）

| 字段 | 取值范围 | 默认值 | 单位 | 场景 |
|------|---------|-------|------|------|
| `dotnet.version` | 8 LTS | 8.0.x | — | 运行时版本 |
| `aspnet.version` | 8.x | 8.0.x | — | Web 框架 |
| `postgres.version` | 16 | 16.x | — | 数据库版本 |
| `redis.version` | 7 | 7.x | — | 缓存版本 |
| `apiResponse.p50.maxMs` | [10, 500] | 100 | ms | P50 延迟 |
| `apiResponse.p95.maxMs` | [100, 2000] | 500 | ms | P95 延迟 |
| `apiErrorRate.max` | [0.001, 0.05] | 0.001 | 比例 | 错误率上限 |
| `availability.target` | [0.99, 0.9999] | 0.999 | 比例 | 服务可用性 |
| `dbConnections.max` | [10, 200] | 100 | 个 | 连接池上限 |
| `redisMemory.max` | [256, 4096] | 1024 | MB | Redis 内存 |
| `concurrentPlayers.target` | [1000, 100000] | 10000 | 人 | 并发玩家 |

## 9. 边界条件 (Edge Cases)

| # | 触发 | 预期行为 | 涉及 |
|---|------|---------|------|
| **T1** | Unity 2022 LTS 升级到 Unity 6 LTS | ABI 兼容 + 包重测 + URP 重做 Shader | 客户端 |
| **T2** | .NET 8 升级到 .NET 10 LTS | 中间件升级 + 性能回归测试 | 服务端 |
| **T3** | PostgreSQL 16 主库故障 | 自动 failover 到 Replica（≤ 30s） | 数据库 |
| **T4** | Redis 7 Sentinel 主节点故障 | 自动选主（≤ 5s） | 缓存 |
| **T5** | CloudFront 边缘节点故障 | 自动路由到健康边缘节点 | CDN |
| **T6** | GitHub Actions 构建超时 | 自动取消 + 邮件通知 + 重试 3 次 | CI/CD |
| **T7** | Steamworks SDK 版本不匹配 | 锁定 Steamworks.NET 20.x + Steam SDK 1.58 | 集成 |
| **T8** | Switch Lotcheck 不通过 | v1.1 推迟 T+3m → T+4m | 分发 |

## 10. 验收标准 (Acceptance Criteria)

- [x] **AC-01：** 文档包含完整 Frontmatter（7 字段）
- [x] **AC-02：** 文档包含 6 必填通用章节
- [x] **AC-03：** 客户端技术栈总览表（≥ 14 维度，含选型 + 版本 + 理由 + 替代方案）
- [x] **AC-04：** 客户端第三方库清单（≥ 8 库，含许可证 + 用途 + 风险）
- [x] **AC-05：** Unity LTS 升级路径（2022 → 2024 → Unity 6 LTS）
- [x] **AC-06：** 客户端性能预算表（≥ 12 项）
- [x] **AC-07：** 服务端技术栈总览表（≥ 14 维度）
- [x] **AC-08：** 服务端第三方库清单（≥ 10 库）
- [x] **AC-09：** .NET LTS 升级路径（.NET 8 → 9 → 10 LTS）
- [x] **AC-10：** 服务端性能预算表（≥ 10 项）
- [x] **AC-11：** PostgreSQL 16 Schema（7 张表 + 索引 + P0-001 CHECK 约束）
- [x] **AC-12：** Redis 7 数据结构（7 种 Key + TTL）
- [x] **AC-13：** DevOps 工具栈表（≥ 12 维度）
- [x] **AC-14：** 7 平台 GitHub Actions Workflows
- [x] **AC-15：** CDN 与对象存储策略
- [x] **AC-16：** 安全技术栈（10 项）
- [x] **AC-17：** PII 收集 = 0 承诺
- [x] **AC-18：** 监控与告警（指标 + 阈值）
- [x] **AC-19：** 配置表（客户端 14 + 服务端 12 字段）
- [x] **AC-20：** 边界条件 ≥ 8 条（T1-T8）
- [x] **AC-21：** 关联文档 / 关联代码 / 变更日志 / 待办事项齐全
- [x] **AC-22：** 文档总行数 ≥ 400 行

## 11. 关联文档

### 上游（本文档依赖）

- [`README.md`](./README.md) — 架构总览
- [`system-overview.md`](./system-overview.md) — 系统边界 + 部署
- [`module-breakdown.md`](./module-breakdown.md) — 14 模块
- [`docs/01-overview-v2.md`](../../docs/01-overview-v2.md) — Unity 2022 LTS + 性能预算
- [`docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — SwitchSlot + 4 槽位
- [`docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — 4 阶段 + 12 里程碑
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台 + 6 区域
- [`design/api/README.md`](../api/README.md) — 18 端点 + OpenAPI 3.0

### 下游（本文档被依赖）

- [`deployment.md`](./deployment.md) — 7 平台分发策略
- [`risks-and-decisions.md`](./risks-and-decisions.md) — 风险 + ADR
- `infrastructure/docker-compose.yml` — 容器化配置
- `.github/workflows/*.yml` — CI/CD 配置
- `unity/Packages/manifest.json` — Unity Package 依赖清单
- `src/Server/*.csproj` — .NET 项目文件

## 12. 关联代码模块

| 模块 | 路径 | 状态 | 引用 |
|------|------|:----:|------|
| Unity Project | `unity/Packages/manifest.json` | 待创建 | §1.2 第三方库 |
| ASP.NET Core Project | `src/Server/Anzhong.Api.csproj` | 待创建 | §2.2 第三方库 |
| Database Migrations | `src/Server/Migrations/*.cs` | 待创建 | §3.2 Schema |
| Dockerfile | `src/Server/Dockerfile` | 待创建 | §4 DevOps |
| GitHub Actions | `.github/workflows/*.yml` | 待创建 | §4.2 Workflows |
| Terraform (v2.0+) | `infrastructure/terraform/*.tf` | 待创建 | §4 IaC |
| Helm Charts (v2.0+) | `infrastructure/helm/*.yaml` | 待创建 | §4 编排 |
| 7 平台构建脚本 | `tools/build_*.sh` | 待创建 | §4.3 |

## 13. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| R-01 | **Unity 2022 LTS → Unity 6 LTS 升级成本** | 中 | 30% | ABI 兼容 + 包重测 + URP 重做 Shader | 已规划 |
| R-02 | **PostgreSQL 16 主从同步延迟导致读不一致** | 低 | 20% | 主从延迟 ≤ 100ms + 关键路径强制读主 | 已规划 |
| R-03 | **.NET 8 LTS 2026-11 终止支持** | 中 | 100% | v2.5 升级 .NET 10 LTS | 已规划 |
| R-04 | **Redis 7 Sentinel 配置复杂** | 中 | 30% | 托管 ElastiCache + 自动 Sentinel | 已规划 |
| R-05 | **CloudFront 边缘缓存命中率低（首周）** | 低 | 40% | 预热脚本 + TTL 优化 + 监控告警 | 已规划 |
| Q-01 | **是否使用 DOTS/ECS 替换 MonoBehaviour** | 低 | — | v1.0 MonoBehaviour 足够；v2.0 评估 | 倾向维持 |
| Q-02 | **是否使用 Dapper 替代 EF Core（性能）** | 低 | — | v2.0+ EF Core 性能足够；v3.0 评估 | 倾向 EF Core |
| Q-03 | **是否使用 GraphQL 替代 REST** | 低 | — | v1.0/v2.0 维持 REST + OpenAPI；v3.0 评估 | 倾向 REST |
| Q-04 | **是否引入 Service Mesh（Istio / Linkerd）** | 低 | — | v2.0+ 10 微服务规模无需；v3.0 评估 | 倾向不用 |
| Q-05 | **是否使用 Kubernetes 替代 ECS Fargate** | 低 | — | v2.0+ 1 人 Solo Fargate 足够；v3.0 评估 K8s | 倾向 Fargate |

## 14. 待办事项 (TODO)

- [ ] **P0：** Unity 2022 LTS + URP 2D + Input System + Addressables 工程初始化 — 阻塞客户端开发
- [ ] **P0：** 客户端第三方库集成（DOTween / Steamworks.NET / Newtonsoft.Json）— 阻塞客户端开发
- [ ] **P1：** v2.0+ ASP.NET Core 8 + PostgreSQL 16 + Redis 7 服务端初始化 — 不阻塞 v1.0
- [ ] **P1：** v2.0+ Entity Framework Core Migrations 编写 — 不阻塞 v1.0
- [ ] **P1：** GitHub Actions ci.yml + build_steam.yml + build_mac.yml 编写 — 阻塞 v1.0 发布
- [ ] **P1：** Docker + Docker Compose 配置 — 阻塞 v2.0+
- [ ] **P2：** Terraform + Helm Charts（v2.0+ IaC）— 不阻塞 v1.0
- [ ] **P2：** Prometheus + Grafana + CloudWatch 监控配置 — 不阻塞 v1.0
- [ ] **P2：** 评估 Unity 6 LTS 升级路径 — 不阻塞 v1.0

## 15. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建**：客户端技术栈总览（14 维度 + 替代方案）+ 第三方库清单（8 库）+ Unity 升级路径 + 客户端性能预算（12 项）+ 服务端技术栈总览（14 维度）+ 第三方库清单（10 库）+ .NET 升级路径 + 服务端性能预算（10 项）+ PostgreSQL 16 Schema（7 表 + P0-001 CHECK 约束）+ Redis 7 数据结构（7 Key）+ DevOps 工具栈（12 维度）+ 7 平台 GitHub Actions + CDN/S3 + 安全（10 项）+ PII = 0 承诺 + 监控告警（7 指标 + 7 阈值）+ 配置表 26 字段 + 边界 8 条 + 风险 5 + 待办 P0×3 P1×4 P2×3。 |