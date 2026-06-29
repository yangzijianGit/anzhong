---
title: 《暗室》限流策略
doc_id: DESIGN-anzhong-api-rate-limiting
parent: design/api/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 策划总监
---

# 《暗室》限流策略 (Rate Limiting)

> **一句话定位：** 60 req/min 客户端 + 1000 req/h 用户 + burst 10 req/s + 防滥用 4 档，匹配"无战斗银河恶魔城"游戏节奏。

## 目的 (Purpose)

本文档是《暗室》API 限流层的**唯一权威规格**。它向：

- **后端/服务端工程师** — 定义网关层限流规则 + Token Bucket 配置
- **Unity 客户端工程师** — 定义客户端退避策略 + 离线队列
- **QA/测试** — 提供限流测试用例
- **运维** — 定义监控指标与告警阈值

**本版本（v1.0）的目的：** 把"无战斗银河恶魔城解谜游戏"的 API 调用频率**第一次**量化为 4 档限流（60/1000/10 + 防滥用），匹配游戏节奏（单房间 P50 60-1200 秒），避免误伤正常玩家 + 阻断恶意刷量。

## 范围 (Scope)

### 包含

- **3 档基础限流** (60/min 客户端 + 1000/h 用户 + 10/s burst)
- **3 档特殊限流** (30/房间重置 + 10/天新会话 + 6/h Hint)
- **限流算法** (Token Bucket + Sliding Window)
- **限流响应格式** (429 + Retry-After)
- **客户端退避策略** (Exponential Backoff)
- **离线队列** (弱网/移动端)
- **监控指标 + 告警阈值**

### 不包含 (Out of Scope)

- 服务端实现细节 (Redis / 内存缓存) → 实施时由后端工程师设计
- DDoS 防护 (Cloudflare / AWS Shield) → 基础设施层
- 业务异常监控 (Sentry / Datadog) → 运维层

## 1. 限流总表 (Rate Limit Master Table)

> 4 档基础 + 3 档特殊 = 7 档限流。

| 档 | 限流 | 范围 | 用途 | 端点 | GDD 引用 |
|---|------|------|------|------|---------|
| **L1 客户端** | 60 req/min | per client_id | 单客户端基础限流 | 全部端点 | 05 §3.2 切换节奏 |
| **L2 用户** | 1000 req/h | per user_id | 单用户累计限流 | 全部端点 | — |
| **L3 Burst** | 10 req/s | per client_id | 突发流量限流 | E06 switch / E16 telemetry | 02 §3.2 300ms 冷却 |
| **L4 房间重置** | 30 req/h | per room | 防 R 键滥用 | E07 reset | 05 §5.3 + 07 §3.3 |
| **L5 新会话** | 10/day | per deviceId | 防频繁重启 | E01 sessions | 04 §1.1 |
| **L6 Hint** | 6/h | per player | 防 Hint 刷量 | E17 hints | 06 §11.2.1 |
| **L7 同步** | 10/h | per player | 跨设备同步 | E15 sync | 11 §1.1 |

## 2. 限流算法 (Algorithms)

### 2.1 Token Bucket (L1/L2/L3)

> 用于基础限流 + Burst 限流。

```
Bucket 配置：
- capacity: 60 (L1) / 1000 (L2) / 10 (L3)
- refill rate: 1 token/second (L1: 60/60) / 1000/3600 ≈ 0.28/s (L2) / 10/1 = 10/s (L3)
- refill policy: 连续补充（不跳变）
```

**示例 L1 (60 req/min)：**
- 玩家 1 分钟内连续发 60 个请求 → 第 61 个请求被拒
- 等待 1 秒 → bucket 补 1 token → 可以发 1 个请求
- 等待 60 秒 → bucket 补满 60 tokens → 可以再发 60 个请求

### 2.2 Sliding Window (L4/L5/L6/L7)

> 用于防滥用 + 特殊限流。

```
Window 配置：
- window: 1 hour (L4/L6/L7) / 1 day (L5)
- max: 30 (L4) / 6 (L6) / 10 (L7) / 10 (L5)
- policy: 滑动窗口（精确计数）
```

**示例 L4 (30 req/h/room)：**
- 玩家 1 小时内对房间 1-1 调用 30 次 reset → 第 31 次触发防滥用
- 等待 1 小时 → 窗口滑动 → 可以再调用 30 次

## 3. 限流响应格式 (Rate Limit Response)

> 与 `error-codes.md` `RATE_LIMIT_EXCEEDED` 错误码一致。

**HTTP 429 响应：**
```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1719652860
Retry-After: 60

{
  "errorCode": "RATE_LIMIT_EXCEEDED",
  "message": "Too many requests",
  "retryAfterMs": 60000,
  "limit": 60,
  "windowSec": 60
}
```

**响应头：**
- `X-RateLimit-Limit` — 限流上限
- `X-RateLimit-Remaining` — 剩余可用次数
- `X-RateLimit-Reset` — 重置时间 (Unix 秒)
- `Retry-After` — 客户端重试等待秒数 (RFC 6585)

## 4. 端点 ↔ 限流对照表 (Endpoint × Rate Limit)

| 端点 | L1 60/min | L2 1000/h | L3 10/s | L4 30/h/room | L5 10/day | L6 6/h | L7 10/h | 备注 |
|------|:--------:|:--------:|:------:|:-----------:|:---------:|:------:|:-------:|------|
| E01 sessions | ✅ | ✅ | ❌ | ❌ | ✅ | ❌ | ❌ | L5 防频繁重启 |
| E02 sessions end | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| E03 rooms enter | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| E04 rooms exit | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| E05 rooms complete | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| **E06 slots switch** | ✅ | ✅ | **✅** | ❌ | ❌ | ❌ | ❌ | **L3 burst 10/s** |
| **E07 rooms reset** | ✅ | ✅ | ❌ | **✅** | ❌ | ❌ | ❌ | **L4 30/h/room** |
| E08 saves | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| E09 saves GET | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| E10 leaderboard | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| E11 progress | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | 批量 100 条/次 |
| E12 feedback | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| E13 audio | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| E14 accessibility | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |
| **E15 sync** | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | **✅** | **L7 10/h** |
| E16 telemetry | ✅ | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | L3 batch 500 |
| **E17 hints** | ✅ | ✅ | ❌ | ❌ | ❌ | **✅** | ❌ | **L6 6/h** |
| E18 chapters | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ | — |

## 5. 关键限流详解 (Critical Rate Limits)

### 5.1 L3 Burst — E06 slot switch (10 req/s)

**为什么需要：** 防止玩家在焦虑时疯狂连按 E 键（300ms 冷却内多次请求）。

**配置：**
- 10 req/s (≈ 33ms/req)
- 与 02 §3.2 客户端 300ms 冷却配合 (服务端 + 客户端双重防护)
- 超限 → `SLOT_COOLDOWN_ACTIVE` (400) 或 `RATE_LIMIT_EXCEEDED` (429)

**客户端预期：**
- 玩家 E 键连按 5 次/秒 → 第 6 次触发 429
- 客户端退避 100ms 后重试

### 5.2 L4 — E07 room reset (30 req/h/room)

**为什么需要：** 防止恶意 R 键滥用 + 匹配 05 §5.3 防滥用阈值。

**配置：**
- 30 req/h/room (单房间累计)
- 超限 → `RESET_ABUSE_THRESHOLD` (429)
- 触发后客户端显示 02 §10.10 暗脉冲提示

**客户端预期：**
- 玩家在 1-1 房间 R 键 30 次 → 第 31 次触发 429
- 客户端提示"重置次数过多，请休息或使用提示"
- 等 1 小时后或主动用 Hint 解锁

### 5.3 L5 — E01 sessions (10/day/deviceId)

**为什么需要：** 防止恶意频繁创建会话刷量。

**配置：**
- 10 req/day/deviceId
- 超限 → `RATE_LIMIT_EXCEEDED` (429) + 24h 冷却
- **不阻塞正常玩家**（正常游戏一天重启 1-3 次）

**客户端预期：**
- 玩家 1 天内启动游戏 10 次 → 第 11 次触发 429
- 客户端显示"今日启动次数已达上限，请明天再试"

### 5.4 L6 — E17 hints (6/h/player)

**为什么需要：** 防止 Hint 刷量 + 鼓励玩家独立思考（06 §11.2.1）。

**配置：**
- 6 req/h/player
- 与 03 §10 E1-E5 边界 + 08 §8.3 渐进式 Hint 配合
- 超限 → `RATE_LIMIT_EXCEEDED` (429)

**客户端预期：**
- 玩家 1 小时内主动触发 Hint 6 次 → 第 7 次触发 429
- 客户端提示"提示次数过多，请先尝试自己解决"

### 5.5 L7 — E15 sync (10/h/player)

**为什么需要：** 防止恶意云同步刷量 + DDoS 防护。

**配置：**
- 10 req/h/player
- 与 11 §1.1 7 平台跨设备同步配合
- 超限 → `RATE_LIMIT_EXCEEDED` (429)

**客户端预期：**
- 玩家 1 小时内切换设备 10 次 → 第 11 次触发 429
- 客户端提示"同步次数过多，请稍后再试"

## 6. 客户端退避策略 (Client Backoff)

> 收到 429 后客户端必须按退避策略重试。

### 6.1 Exponential Backoff with Jitter

```
attempt 1: 等待 retryAfterMs (服务端指定) + 随机 0-100ms
attempt 2: 等待 retryAfterMs × 2 + 随机 0-200ms
attempt 3: 等待 retryAfterMs × 4 + 随机 0-400ms
attempt 4: 等待 retryAfterMs × 8 + 随机 0-800ms
attempt 5: 放弃 + 提示用户
```

**示例 (retryAfterMs=1000)：**
- attempt 1: 1000-1100ms
- attempt 2: 2000-2200ms
- attempt 3: 4000-4400ms
- attempt 4: 8000-8800ms
- attempt 5: 放弃

### 6.2 客户端实现

```typescript
// 伪代码
async function callWithBackoff(endpoint, payload, maxAttempts = 5) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    const response = await fetch(endpoint, payload);
    if (response.status !== 429) return response;
    
    const retryAfterMs = response.headers.get('Retry-After') * 1000 
                       || response.body.retryAfterMs;
    const jitter = Math.random() * 100 * attempt;
    await sleep(retryAfterMs * (2 ** (attempt-1)) + jitter);
  }
  throw new Error('Rate limit exhausted after ' + maxAttempts + ' attempts');
}
```

## 7. 离线队列 (Offline Queue)

> 移动端 + 弱网场景下，未发出的请求加入离线队列，恢复网络后批量重发。

### 7.1 离线策略

| 端点 | 离线行为 |
|------|---------|
| E06 slot switch | 不缓存（玩家已经在本地切换了，服务端状态滞后可接受） |
| E07 room reset | 不缓存 |
| E11 progress | ✅ 批量缓存（最多 1000 条），恢复后批量上报 |
| E16 telemetry | ✅ 批量缓存（最多 5000 条），恢复后批量上报 |
| E12 feedback | ✅ 缓存（最多 50 条），恢复后批量上报 |
| E08 saves | ✅ 缓存（最近 1 次），恢复后立即上报 |
| E15 sync | ❌ 不缓存（同步需在线） |
| E17 hints | ❌ 不缓存（Hint 触发需在线） |

### 7.2 离线队列持久化

- 存储位置：客户端本地 SQLite / JSON 文件
- 容量：5000 条（E16 telemetry）
- 清理策略：成功发送后立即删除
- 过期策略：超过 7 天的遥测事件自动丢弃

## 8. 监控指标 (Monitoring Metrics)

> 服务端 Prometheus 指标 + Grafana 仪表盘。

| 指标 | 标签 | 告警阈值 |
|------|------|---------|
| `rate_limit_exceeded_total` | endpoint, error_code | > 100/min 全局 |
| `rate_limit_remaining` | endpoint, client_id | < 10 持续 1min |
| `rate_limit_reset_total` | endpoint | > 50/min |
| `offline_queue_size` | endpoint, client_id | > 1000 持续 5min |
| `backoff_attempts_total` | endpoint, attempt | 5 次重试占比 > 5% |

**告警规则：**
- `rate_limit_exceeded_total > 100/min` → 服务端限流过严，考虑放宽
- `rate_limit_exceeded_total > 10000/min` → 恶意流量，启动 Cloudflare 防护
- `backoff_attempts_total{attempt="5"} > 5%` → 客户端退避失败，可能 DDoS

## 9. 移动端优化 (Mobile Optimization)

> 与 11 §1.1 iOS/Android 平台 + 11 §3.3 软启动计划对齐。

| 端点 | 移动端放宽 |
|------|----------|
| L1 客户端 | 30 req/min (移动端) vs 60 req/min (PC) |
| L3 Burst | 5 req/s (移动端) vs 10 req/s (PC) |
| L11 progress | 200 条/批次 (移动端) vs 100 条/批次 (PC) |
| L16 telemetry | 1000 条/批次 (移动端) vs 500 条/批次 (PC) |

**为什么放宽：**
- 移动端弱网频繁 → 需要更宽容的限流
- 移动端生命周期短（一次会话 5-15 min）→ 累计量低
- 与 11 §1.3 软启动计划 (iOS 优先) 对齐

## 10. 反作弊与异常检测 (Anti-Abuse)

| 模式 | 检测规则 | 响应 |
|------|---------|------|
| **同一 playerId 跨 3+ 设备同时活跃** | 实时检测 | 触发风控 + 强制二次验证 |
| **同一 playerId 1h 内 > 1000 req** | L2 触发 + 异常标记 | 临时封禁 1h + 人工审核 |
| **E07 reset 1h 内 > 30 次** | L4 触发 | `RESET_ABUSE_THRESHOLD` + 暗脉冲 |
| **E17 hints 1h 内 > 6 次** | L6 触发 | `RATE_LIMIT_EXCEEDED` + 提示 |
| **异常 IP 段（Tor / VPN）** | IP 库匹配 | 强制二次验证 |
| **客户端时钟漂移 > 5min** | 时间戳验证 | `CLIENT_TIME_DRIFT` 401 |

## 11. 配置示例 (Configuration Example)

> 服务端网关配置（伪代码）。

```yaml
# api-gateway.yml
rate_limits:
  - name: L1_client_per_min
    algorithm: token_bucket
    capacity: 60
    refill_rate: 1  # per second
    scope: client_id
    endpoints: ["*"]
  
  - name: L2_user_per_hour
    algorithm: token_bucket
    capacity: 1000
    refill_rate: 0.278  # 1000/3600
    scope: user_id
    endpoints: ["*"]
  
  - name: L3_burst_per_second
    algorithm: token_bucket
    capacity: 10
    refill_rate: 10
    scope: client_id
    endpoints: ["/slots/*/switch", "/telemetry"]
  
  - name: L4_reset_per_room
    algorithm: sliding_window
    window_sec: 3600
    max: 30
    scope: room_id
    endpoints: ["/rooms/*/reset"]
  
  - name: L5_sessions_per_day
    algorithm: sliding_window
    window_sec: 86400
    max: 10
    scope: device_id
    endpoints: ["/sessions"]
  
  - name: L6_hints_per_hour
    algorithm: sliding_window
    window_sec: 3600
    max: 6
    scope: player_id
    endpoints: ["/hints"]
  
  - name: L7_sync_per_hour
    algorithm: sliding_window
    window_sec: 3600
    max: 10
    scope: player_id
    endpoints: ["/sync"]
```

## 12. 关联文档 (Cross-References)

- [`api-spec.yaml`](./api-spec.yaml) — OpenAPI 3.0
- [`endpoints.md`](./endpoints.md) — 端点详解
- [`error-codes.md`](./error-codes.md) — 错误码 (RATE_LIMIT_EXCEEDED)
- [`authentication.md`](./authentication.md) — 鉴权 + 限流配合
- [`versioning.md`](./versioning.md) — 版本演进
- [`../../docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — 300ms 冷却 (02 §3.2)
- [`../../docs/05-numerical-design-v2.md`](../../docs/05-numerical-design-v2.md) — 30 次防滥用 (05 §5.3)
- [`../../docs/06-player-experience-v2.md`](../../docs/06-player-experience-v2.md) — Hint 触发 (06 §11.2.1)
- [`../../docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台 (11 §1.1) + 软启动 (11 §3.3)

## 13. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| R-01 | **限流过严影响移动端离线/弱网体验** | 中 | 40% | 移动端放宽至 30/min + 离线队列 | 已规划 |
| R-02 | **误伤正常玩家（高频玩家 E 键连按）** | 中 | 30% | L3 burst 10/s + 客户端 300ms 冷却 | 已规划 |
| R-03 | **DDoS 攻击绕过应用层限流** | 高 | 20% | Cloudflare / AWS Shield 基础设施层 | 已规划 |
| R-04 | **Token Bucket 内存占用** | 低 | 10% | Redis 集中存储 + TTL | 已规划 |
| Q-01 | **是否按平台差异化限流（PC/移动/主机）** | 中 | — | v1.0 PC=60/min, 移动=30/min；主机评估 | 倾向维持 |
| Q-02 | **是否按章节差异化限流（Ch1 教学房更宽容）** | 低 | — | v1.0 不差异化；v1.1 评估 | 倾向 v1.1 |
| Q-03 | **是否按 VIP 玩家差异化限流** | 低 | — | 本游戏无 VIP，N/A | N/A |

## 14. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** 7 档限流 (L1-L7) + Token Bucket + Sliding Window 算法 + 端点对照表 (18 端点) + 客户端退避策略 (Exponential Backoff + Jitter) + 离线队列 (5 端点) + 监控指标 + 移动端优化 + 反作弊检测 + 7 风险 3 开放问题。 |
