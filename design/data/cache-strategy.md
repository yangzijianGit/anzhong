---
title: 《暗室》缓存策略 (Cache Strategy)
doc_id: DESIGN-anzhong-data-cache
parent: design/data/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》缓存策略 (Cache Strategy)

> **一句话定位：** 客户端 L1 (内存) + L2 (Addressables 二级缓存) + Redis 7 L3 + CDN L4 四级缓存架构，覆盖 8 类热数据 + 5 类冷数据 + 3 档 TTL + LRU 淘汰 + 写穿透 + 缓存雪崩/穿透/击穿防护。

## 目的 (Purpose)

本文档是《暗室》**缓存层 (Cache Layer)** 的**唯一权威策略规格**。它向：

- **Unity 客户端工程师** — 定义 L1 内存缓存 + L2 Addressables 二级缓存
- **服务端工程师 (v2.0+)** — 定义 Redis 7 缓存键名空间 + 3 档 TTL + LRU 淘汰
- **DevOps / SRE** — 定义缓存监控指标 + 告警阈值 + 雪崩/穿透/击穿防护
- **架构师** — 提供四级缓存架构 (L1-L4) + 一致性策略 + 失效广播

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的全部缓存策略——**4 级缓存 (L1 内存 + L2 Addressables + L3 Redis + L4 CDN) + 3 档 TTL (5min/1h/24h/永久) + LRU 淘汰 + 写穿透 + 8 类热数据 + 5 类冷数据 + 3 大防护 (雪崩/穿透/击穿) + 监控告警**——**第一次**用统一文档描述，作为 phase3 → phase4 实施的"缓存合同"。

## 范围 (Scope)

### 包含

- **4 级缓存架构**: L1 客户端内存 / L2 Addressables / L3 Redis 7 / L4 CDN
- **8 类热数据缓存键**: session / save / progress / settings / chapter / room / leaderboard / ratelimit
- **3 档 TTL**: 5 min (短) / 1 h (中) / 24 h (长) / 永久 (静态)
- **淘汰策略**: LRU (Redis maxmemory-policy) + 主动失效 (write-through + invalidate)
- **一致性**: Cache-Aside / Write-Through / Write-Behind
- **防护**: 雪崩 (随机 TTL) / 穿透 (布隆过滤器) / 击穿 (互斥锁)
- **监控**: 命中率 / 延迟 / 内存 / QPS + 告警阈值

### 不包含 (Out of Scope)

- API 端点契约 → 见 `../api/endpoints.md`
- 持久化策略 → 见 `persistence-strategy.md`
- 数据库 schema → 见 `database-schema.md`
- 序列化格式 → 见 `serialization.md`
- 迁移路径 → 见 `migrations.md`

## 一句话描述 (One-liner)

> **"L1 内存 + L2 Addressables + L3 Redis + L4 CDN 四级缓存，3 档 TTL + LRU + 写穿透 + 3 大防护。"**

## 1. 四级缓存架构 (4-Level Cache)

```
┌──────────────────────────────────────────────────────────────┐
│ Level 1 (L1): 客户端内存 · ≤ 50 MB · 进程内 Dictionary      │
│   - 启动时加载: 玩家设置 / 当前章节 / 当前房间配置            │
│   - TTL: 进程生命周期 (无 TTL)                                 │
└───────────────────────────┬──────────────────────────────────┘
                            │ miss
┌───────────────────────────▼──────────────────────────────────┐
│ Level 2 (L2): Addressables 二级缓存 · ≤ 500 MB · 磁盘/内存   │
│   - 静态资源: 房间配置 / 章节主题 / 9 类音频 / Sprite Atlas    │
│   - TTL: 永久 (资源版本化)                                     │
└───────────────────────────┬──────────────────────────────────┘
                            │ miss (服务端调用)
┌───────────────────────────▼──────────────────────────────────┐
│ Level 3 (L3): Redis 7 集群 · Sentinel HA · ≤ 4 GB            │
│   - 热数据: session / save / progress / leaderboard / 限流     │
│   - TTL: 5 min / 1 h / 24 h / 永久                            │
└───────────────────────────┬──────────────────────────────────┘
                            │ miss (数据库查询)
┌───────────────────────────▼──────────────────────────────────┐
│ Level 4 (L4): PostgreSQL 16 主从 · 真理之源                   │
│   - 所有数据最终落盘 PostgreSQL                                │
└──────────────────────────────────────────────────────────────┘
```

### 1.1 各级缓存特征

| 级别 | 存储 | 容量 | 延迟 | 命中率目标 | 适用 |
|:----:|------|------|:----:|:---------:|------|
| **L1** | 进程内 `Dictionary<string, object>` | ≤ 50 MB | ~0 ms (无 IO) | ≥ 99% | 玩家设置 / 当前房间 |
| **L2** | Addressables (`UnityEngine.ResourceManagement`) | ≤ 500 MB | ~5 ms | ≥ 95% | 静态资源 (房间配置/Sprite/音频) |
| **L3** | Redis 7 (Sentinel HA, 1 主 + 2 从) | ≤ 4 GB | ~1 ms | ≥ 90% | session/save/progress/leaderboard |
| **L4** | PostgreSQL 16 (1 主 + 2 Replica) | TB 级 | ~5 ms | — | 真理之源 (Source of Truth) |

## 2. 8 类热数据缓存键 (Hot Data Keys)

> 所有 Redis 键格式: `{namespace}:{identifier}:{qualifier}` (冒号分隔, 易监控)

### 2.1 热数据清单

| 数据类型 | Redis Key 模式 | 类型 | TTL | 一致性策略 | 用途 |
|---------|---------------|:----:|:---:|-----------|------|
| **会话状态** | `session:{sessionId}` | Hash | 24 h | 写穿透 + 心跳续期 | E01-E02 会话管理 |
| **玩家存档** | `save:{playerId}` | String (JSON) | 5 min | 写穿透 + 主动失效 | E08-E09, E15 |
| **玩家进度** | `progress:{playerId}` | Hash | 1 h | Cache-Aside + 写穿透 | E11, E18 |
| **玩家设置** | `settings:{playerId}:{type}` | String (JSON) | 24 h | 写穿透 + 版本号 | E13-E14 (type=audio/accessibility) |
| **章节列表** | `chapter:list` | String (JSON) | 永久 | 启动加载 + 主动失效 | E18 chapters |
| **房间配置** | `room:{roomId}` | String (JSON) | 永久 | 启动加载 + 主动失效 | E03-E05 |
| **排行榜** | `lb:{chapterId}:top100` | Sorted Set | 永久 | 异步更新 + 主动失效 | E10 |
| **限流计数** | `ratelimit:{apiKey}:{minute}` | String | 60 s | 滑动窗口 | 所有 API |

### 2.2 Redis Key 详细规范

#### 2.2.1 `session:{sessionId}` (会话 Hash)

```
HSET session:sess_20260629_abc123 \
    player_id "player_x9y8z7" \
    platform "steam" \
    client_version "1.0.0" \
    state "room_playing" \
    started_at "2026-06-29T10:00:00Z" \
    last_heartbeat "2026-06-29T10:30:00Z"
EXPIRE 86400  # 24 小时 TTL
```

**更新时机：** E01 创建 / E02 结束 / 心跳每 5 分钟续期

**失效时机：** 显式 DEL on E02 success / 5 分钟心跳超时自动清理

#### 2.2.2 `save:{playerId}` (存档 JSON)

```
SET save:player_x9y8z7 '{"version":"1.0.0","currentChapterId":"ch2","currentRoomId":"2-3",...}' EX 300
```

**更新时机：** E08 写入 (Write-Through, 写 DB 后立即写 Redis)

**失效时机：** E09 读取后立即失效 (强一致性) / E15 sync 后失效

**大小：** ≤ 100 KB / 玩家 (Protobuf 二进制)

#### 2.2.3 `progress:{playerId}` (玩家进度 Hash)

```
HSET progress:player_x9y8z7 \
    total_rooms_completed 8 \
    total_rooms 19 \
    unlock_progress_pct 42.1 \
    current_chapter_id "ch2" \
    last_updated "2026-06-29T10:30:00Z"
EXPIRE 3600  # 1 小时 TTL
```

**更新时机：** E11 上报后 / 章节完成触发器

**失效时机：** 章节解锁时 (ch2 unlock → 全部 ch1 progress 失效)

#### 2.2.4 `settings:{playerId}:{type}` (玩家设置 JSON)

```
SET settings:player_x9y8z7:audio '{"masterVolume":0.8,"switchSfxDb":-12,...}' EX 86400
SET settings:player_x9y8z7:accessibility '{"colorblindMode":"default","fontScale":1.0,...}' EX 86400
```

**更新时机：** E13 PUT / E14 PUT (Write-Through)

**失效时机：** 不失效 (玩家设置极少变更, 24h 自然过期)

#### 2.2.5 `chapter:list` (章节列表 JSON, 静态)

```
SET chapter:list '[{"chapterId":"ch1","name":"觉醒",...},...]'
# 永久 (EX 0 或不设 TTL)
```

**更新时机：** 章节配置变更时 (策划手动) — 极少

**失效时机：** 主动 DEL on 章节配置更新

#### 2.2.6 `room:{roomId}` (房间配置 JSON, 静态)

```
SET room:1-1 '{"roomId":"1-1","name":"入门","type":"tutorial","difficulty":2,"slots":[...]}'
# 永久
```

**更新时机：** 房间配置变更时 (策划手动) — 极少

**失效时机：** 主动 DEL on 房间配置更新

#### 2.2.7 `lb:{chapterId}:top100` (排行榜 Sorted Set)

```
ZADD lb:ch1:top100 8 "player_aaa:1719652800000"  # score=steps
ZADD lb:ch1:top100 12 "player_bbb:1719652700000"
# 永久
```

**更新时机：** E05 complete → 异步更新 (Lua 脚本保证原子性)

**失效时机：** 不失效 (Top 100 滚动更新)

**关键：** Sorted Set score = 步数 (越少越好, ZRANGE 0 99 取 Top 100)

#### 2.2.8 `ratelimit:{apiKey}:{minute}` (限流 String)

```
SET ratelimit:player_x9y8z7:1719652800 5 EX 60  # 5 次访问
INCR ratelimit:player_x9y8z7:1719652800
```

**更新时机：** 每次 API 调用 INCR

**失效时机：** TTL 60s 自动过期 (滑动窗口)

## 3. 5 类冷数据 (Cold Data)

| 数据类型 | 存储 | 保留期 | 用途 | 清理策略 |
|---------|------|--------|------|---------|
| **遥测事件** | Redis Stream → PostgreSQL | 7 d (Stream) + 90 d (PG) | 实时分析 | XADD + XTRIM MAXLEN ~ 1M |
| **过期 save_backups** | PostgreSQL | 30 d | 存档恢复 | pg_cron 每日清理 |
| **过期 audit_logs** | PostgreSQL + S3 | 1 y (PG) + 永久 (S3) | 合规审计 | 1 年后归档到 S3 |
| **旧 sessions** | PostgreSQL | 1 y | 会话分析 | 1 年后归档 |
| **CDN 资源** | CloudFront | 30 d (静态) / 7 d (Addressables) | 资源分发 | TTL 自动过期 |

### 3.1 遥测事件流 (Redis Stream)

```
XADD telemetry:events \
    * session_id "sess_20260629_abc123" \
    event_type "room_completed" \
    room_id "1-1" \
    timestamp_ms 1719652800000 \
    properties '{"steps":3,"timeSec":95}'
```

**消费者：** Telemetry Worker (后台服务)
- 批量消费 (BATCH_SIZE=500, BLOCK=1000ms)
- 写入 PostgreSQL `telemetry_events` 表
- 触发实时分析 (P50/P90/ResetCount/HintTriggerRate)

**TTL：** `XTRIM telemetry:events MAXLEN ~ 1000000` (保留 ~1M 事件作为缓冲)

## 4. 3 档 TTL 配置 (3 TTL Tiers)

| 档位 | TTL | 适用 | 失效模式 |
|:----:|-----|------|---------|
| **短 (Short)** | 5 min | save (高一致性要求) | 主动失效 + 强制刷新 |
| **中 (Medium)** | 1 h | progress (中等一致性) | 主动失效 + 自然过期 |
| **长 (Long)** | 24 h | settings (低一致性要求) | 自然过期 |
| **永久 (Static)** | 0 (无 TTL) | chapter / room (静态) | 仅主动失效 |

**TTL 选择原则：**
- **数据变更频率 × 一致性要求** = TTL
- 高变更 + 高一致 → 短 TTL + Write-Through
- 低变更 + 低一致 → 长 TTL + Cache-Aside
- 不变更 (静态) → 永久 + 启动加载

## 5. 淘汰策略 (Eviction Policy)

### 5.1 Redis maxmemory-policy

```conf
# redis.conf
maxmemory 4gb
maxmemory-policy allkeys-lru  # 全部键 LRU 淘汰 (适用于热数据 + 冷数据混合)
```

**为什么不用 volatile-lru？** 因为热数据 + 冷数据混合，淘汰最少使用的键（可能是热数据），保证整体命中率。

### 5.2 主动失效 (Invalidate)

| 场景 | 失效键 | 时机 |
|------|--------|------|
| **E08 save write** | `save:{playerId}` | 写 DB 后 DEL |
| **E15 sync** | `save:{playerId}` + `progress:{playerId}` | sync 后 DEL |
| **章节解锁** | `progress:{playerId}` + `lb:{chapterId}:top100` | 触发器 |
| **设置更新** | (不失效, 写穿透) | — |
| **房间配置变更** | `room:{roomId}` + `chapter:list` | 策划手动 |

**批量失效 Lua 脚本 (原子):**

```lua
-- scripts/invalidate_on_save.lua
-- KEYS[1] = save:{playerId}
-- KEYS[2] = progress:{playerId}
-- KEYS[3] = leaderboard key (optional)
redis.call('DEL', KEYS[1])
redis.call('DEL', KEYS[2])
if KEYS[3] then
    redis.call('DEL', KEYS[3])
end
return 1
```

## 6. 一致性策略 (Consistency Strategies)

### 6.1 Cache-Aside (读穿透)

```
Read(key):
  1. cache = GET key
  2. if cache != null: return cache
  3. cache = DB.read(key)
  4. SET key cache EX TTL
  5. return cache
```

**适用：** progress / settings (读多写少, 容忍短暂不一致)

### 6.2 Write-Through (写穿透)

```
Write(key, value):
  1. DB.write(key, value)
  2. SET key value EX TTL  # 立即写缓存
  3. return success
```

**适用：** save / settings (强一致性要求, 写完即可读)

### 6.3 Write-Behind (异步写回)

```
Write(key, value):
  1. SET key value EX TTL
  2. enqueue DB.write(key, value)  # 异步队列
  3. return success
```

**适用：** telemetry (高写入频率, 容忍秒级丢失)

### 6.4 Refresh-Ahead (提前刷新)

```
Background Task:
  for key in hot_keys:
    if TTL < threshold:
      value = DB.read(key)
      SET key value EX TTL  # 后台提前刷新
```

**适用：** leaderboard (热点数据, 避免过期后 cold start)

## 7. 缓存防护 (Cache Protection)

### 7.1 缓存雪崩 (Cache Avalanche) 防护

**问题：** 大量 key 同时过期 → DB 瞬时压力

**防护：**
- **随机 TTL** (避免同时过期)

```python
# 设置缓存时, TTL ± 10% 随机抖动
import random
base_ttl = 300  # 5 min
ttl = base_ttl + random.randint(-30, 30)  # 270-330s
redis.set('save:player_x', data, ex=ttl)
```

- **后台提前刷新** (Refresh-Ahead)
- **熔断降级** (DB 故障时返回 stale cache)

### 7.2 缓存穿透 (Cache Penetration) 防护

**问题：** 查询不存在的数据 → 每次都打到 DB

**防护：**
- **布隆过滤器 (Bloom Filter)** 拦截不存在 key

```python
# 启动时加载所有 player_id 到布隆过滤器
bloom_filter = BloomFilter(capacity=1_000_000, error_rate=0.001)
for player_id in db.query("SELECT id FROM players"):
    bloom_filter.add(player_id)

# 查询时
if not bloom_filter.contains(player_id):
    return None  # 不存在, 不查 DB

# 否则正常查缓存 → DB
```

- **空值缓存** (TTL 5 min)

```
GET save:player_999 (不存在)
→ SET save:player_999 "" EX 300  # 空值缓存 5 min
→ return None
```

### 7.3 缓存击穿 (Cache Breakdown) 防护

**问题：** 热点 key 过期瞬间 → 大量并发打到 DB

**防护：**
- **分布式互斥锁 (Mutex Lock)** (Redis SETNX)

```python
def get_save_with_lock(player_id):
    # 1. 查缓存
    cached = redis.get(f'save:{player_id}')
    if cached:
        return cached
    
    # 2. 缓存 miss, 获取分布式锁
    lock_key = f'lock:save:{player_id}'
    if redis.set(lock_key, '1', nx=True, ex=10):  # 10s 锁
        try:
            # 3. 只让一个线程查 DB
            data = db.read_save(player_id)
            redis.set(f'save:{player_id}', data, ex=300)
            return data
        finally:
            redis.delete(lock_key)
    else:
        # 4. 其他线程等待 100ms 重试
        time.sleep(0.1)
        return get_save_with_lock(player_id)
```

- **永不过期 + 后台刷新** (Refresh-Ahead, 仅 leaderboard)

## 8. 性能指标 (Performance Metrics)

### 8.1 关键指标

| 指标 | 目标 | 告警阈值 |
|------|------|---------|
| **缓存命中率 (Hit Rate)** | ≥ 95% | < 90% (Slack) |
| **L1 命中率** | ≥ 99% | < 95% (Slack) |
| **L3 Redis 命中率** | ≥ 90% | < 80% (PagerDuty) |
| **Redis P95 延迟** | ≤ 5 ms | > 10 ms (Slack) |
| **Redis 内存使用** | ≤ 80% | > 80% (Slack), > 90% (PagerDuty) |
| **缓存重建时间** | ≤ 50 ms | > 200 ms (Slack) |
| **QPS** | ≤ 10K/s | > 8K/s (Slack) |

### 8.2 监控命令

```bash
# Redis 命中率
redis-cli INFO stats | grep keyspace_hits
# 公式: hit_rate = keyspace_hits / (keyspace_hits + keyspace_misses)

# Redis 内存使用
redis-cli INFO memory | grep used_memory_human

# Redis 慢查询
redis-cli SLOWLOG GET 10

# 缓存键数量
redis-cli DBSIZE
```

## 9. Redis 集群与高可用 (Redis Cluster & HA)

### 9.1 Sentinel HA (1 主 + 2 从 + 3 Sentinel)

```
┌──────────────┐
│ Sentinel 1   │──┐
└──────────────┘  │
┌──────────────┐  │ 监控 + 故障转移
│ Sentinel 2   │──┼──→ Master (读写)
└──────────────┘  │      ↓
┌──────────────┐  │   Async Replication
│ Sentinel 3   │──┘      ↓
└──────────────┘   ┌──────────────┐
                   │ Replica 1    │ (读)
                   └──────────────┘
                   ┌──────────────┐
                   │ Replica 2    │ (读)
                   └──────────────┘
```

**故障转移时间：** ≤ 30 秒 (Sentinel 自动检测 + failover)

**客户端重试：** StackExchange.Redis 自动重试 3 次

### 9.2 不分片 (Single Shard) 的理由

- 19 房间 + 100K 玩家 → 数据量 ≤ 4 GB, **单 Redis 足够**
- 分片增加复杂度 (resharding / hot key), 19 房间规模不必要
- v3.0+ 若超过 4 GB, 再评估 Cluster (16 shards × 256 MB)

## 10. CDN 缓存 (L4, 静态资源)

### 10.1 CloudFront 配置 (v2.0+)

```yaml
# cdn-config.yaml
distributions:
  - id: E1A2B3C4D5E6F7
    domain: cdn.anzhong.game
    origins:
      - s3-art-assets  # S3 bucket: anzhong-art-2026
      - steam-cdn      # Steamworks CDN fallback (v1.0)
    behaviors:
      - path: /art/*
        ttl: 2592000  # 30 天 (静态图片/音频)
        compress: true
        cache_policy: CachingOptimized
      - path: /aa/*  # Addressables
        ttl: 604800   # 7 天
        compress: true
        cache_policy: CachingOptimized
      - path: /patch/*
        ttl: 604800   # 7 天 (补丁)
        compress: true
        cache_policy: CachingOptimized
```

**v1.0 CDN：** Steamworks CDN (内置, 零配置)

## 11. 缓存预热 (Cache Warmup)

### 11.1 启动时预热

```python
# 服务端启动时预热静态数据
def warmup_cache():
    # 1. 加载所有章节 (19 rooms + 3 chapters)
    for chapter in db.query("SELECT * FROM chapters"):
        redis.set(f'chapter:{chapter.id}', chapter.json())
    
    for room in db.query("SELECT * FROM rooms"):
        redis.set(f'room:{room.id}', room.json())
    
    # 2. 加载排行榜 Top 100 (3 chapters × 3 difficulty tiers = 9 keys)
    for chapter in ['ch1', 'ch2', 'ch3']:
        top100 = db.query("SELECT * FROM scores WHERE chapter_id = ? ORDER BY steps LIMIT 100", chapter)
        redis.zadd(f'lb:{chapter}:top100', {f'{s.player_id}:{s.completed_at}': s.steps for s in top100})
    
    logger.info(f"Cache warmed: {redis.dbsize()} keys")
```

### 11.2 玩家登录时预热

```python
# 玩家登录时预热个人数据
def warmup_player_cache(player_id):
    # 1. SaveData
    save = db.query("SELECT * FROM saves WHERE player_id = ?", player_id)
    if save:
        redis.set(f'save:{player_id}', save.json(), ex=300)
    
    # 2. Settings
    audio = db.query("SELECT * FROM player_audio_settings WHERE player_id = ?", player_id)
    if audio:
        redis.set(f'settings:{player_id}:audio', audio.json(), ex=86400)
    
    accessibility = db.query("SELECT * FROM player_accessibility_settings WHERE player_id = ?", player_id)
    if accessibility:
        redis.set(f'settings:{player_id}:accessibility', accessibility.json(), ex=86400)
    
    # 3. Progress
    progress = db.query("SELECT * FROM player_progress WHERE player_id = ?", player_id)
    if progress:
        redis.hset(f'progress:{player_id}', mapping=progress.dict())
        redis.expire(f'progress:{player_id}', 3600)
```

## 12. 关联文档 (Cross-References)

- [`README.md`](./README.md) — 数据架构总览
- [`persistence-strategy.md`](./persistence-strategy.md) — 持久化策略
- [`database-schema.md`](./database-schema.md) — 18 张表 schema
- [`serialization.md`](./serialization.md) — Protobuf + JSON 序列化
- [`migrations.md`](./migrations.md) — Alembic 迁移路径
- [`backup-and-recovery.md`](./backup-and-recovery.md) — 3-2-1 备份
- [`p0-001-tracking.md`](./p0-001-tracking.md) — P0-001 跟踪
- [`../api/endpoints.md`](../api/endpoints.md) — 18 端点
- [`../architecture/tech-stack.md`](../architecture/tech-stack.md) — Redis 7 技术栈

## 13. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** 4 级缓存 (L1-L4) + 8 类热数据键规范 + 5 类冷数据 + 3 档 TTL + LRU 淘汰 + 3 种一致性策略 + 3 大防护 (雪崩/穿透/击穿) + Sentinel HA + CDN 配置 + 缓存预热 + 监控告警。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）