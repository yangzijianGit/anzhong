---
title: 《暗室》鉴权方案
doc_id: DESIGN-anzhong-api-authentication
parent: design/api/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 策划总监
---

# 《暗室》鉴权方案 (Authentication)

> **一句话定位：** JWT + 平台 OAuth + 设备 ID 三层鉴权，覆盖 7 平台 + 匿名玩家 + 跨设备云同步。

## 目的 (Purpose)

本文档是《暗室》API 鉴权层的**唯一权威规格**。它向：

- **后端/服务端工程师** — 定义鉴权流程、Token 颁发/刷新/吊销
- **Unity 客户端工程师** — 定义 Token 存储、刷新、自动登录
- **平台对接** — Steam / Apple / Google / Sony / Microsoft / Nintendo 6 大平台 OAuth
- **安全/合规** — Token 安全策略 + 设备绑定 + 跨设备同步验证

**本版本（v1.0）的目的：** 把"无战斗银河恶魔城解谜游戏"的鉴权**第一次**统一为 JWT + 平台 OAuth + 设备 ID 三层架构，覆盖 v1.0 上线的 Steam/Mac 2 平台 + v1.1/v2.0 扩展的 PS5/Xbox/Switch/iOS/Android 5 平台 + 匿名玩家离线模式。

## 范围 (Scope)

### 包含

- **JWT 鉴权方案** (玩家匿名/正式) + Token 生命周期
- **设备 ID 绑定** (匿名玩家) + 设备指纹
- **平台 OAuth 集成** (7 平台分 3 档)
- **Refresh Token 机制** + 自动刷新流程
- **Token 存储安全** (客户端 Keychain / 后端 HSM)
- **跨设备云同步鉴权** (E15)
- **隐私与合规** (GDPR / CCPA / 中国个人信息保护法)

### 不包含 (Out of Scope)

- 服务端实现细节（HSM 接入、密钥轮换）→ 实施时由安全工程师设计
- 客户端 SDK 完整实现 → `src/Api/Client/Auth/`
- 反作弊 / 作弊检测 → `design/architecture/security.md`（v1.1 评估）
- 支付/退款鉴权 → 见 `11-release-v2.md` §2 定价

## 1. 一句话架构 (One-liner Architecture)

> **"三层鉴权：匿名玩家 (DeviceId) → 注册玩家 (JWT) → 跨设备玩家 (Platform OAuth + JWT)"**

| 层 | 用途 | 鉴权方式 | 平台支持 |
|---|------|---------|---------|
| **L1 匿名层** | 首次启动、未登录玩家 | DeviceId 匿名 Token | 全 7 平台 |
| **L2 正式层** | 登录玩家、进度同步 | JWT (24h) + Refresh Token (30d) | 全 7 平台 |
| **L3 跨设备层** | 云同步、跨平台进度 | Platform OAuth + JWT | Steam / Mac / Switch / iOS / Android |

## 2. 鉴权流程 (Auth Flow)

### 2.1 匿名玩家流程 (L1)

```
[客户端启动]
    ↓
[读取本地 deviceId (UUID v4)]
    ↓
POST /api/v1/sessions { deviceId, platform, clientVersion }
    ↓
[服务端] 检查 deviceId 是否已有 playerId
    ├── 已有 → 返回旧 playerId + 临时 Token (1h)
    └── 没有 → 创建新 playerId (匿名) + 临时 Token (1h)
    ↓
[客户端] 收到 sessionId + 临时 Token
    ↓
[后续请求] Authorization: Bearer <token>
    ↓
[临时 Token 1h 过期]
    ↓
POST /api/v1/auth/anonymous-refresh { deviceId }
    ↓
[服务端] 颁发新临时 Token (1h) + 静默检查是否需要升级到正式账号
```

**关键点：**
- deviceId 使用 UUID v4，**绝不**包含 PII（个人可识别信息）
- 匿名玩家进度**保留在服务端**，但绑定到 deviceId
- 设备丢失/重置 → 进度丢失（除非升级到正式账号）

### 2.2 正式玩家流程 (L2)

```
[客户端] 玩家选择"登录"
    ↓
[根据 platform 选择 OAuth 流程]
    ├── Steam → 跳转 Steam OpenID
    ├── Mac → 跳转 Sign in with Apple
    ├── PS5 → 跳转 PSN
    ├── Xbox → 跳转 Xbox Live
    ├── Switch → 跳转 Nintendo Account
    ├── iOS → 跳转 Sign in with Apple
    └── Android → 跳转 Google Sign-In
    ↓
[平台 OAuth 回调] 拿到 platformToken
    ↓
POST /api/v1/auth/platform-login { deviceId, platform, platformToken }
    ↓
[服务端]
    1. 验证 platformToken → 拿到 platformUserId + displayName
    2. 查找或创建 playerId (按 platformUserId)
    3. **关联** 之前的匿名进度到该 playerId
    4. 颁发 JWT (24h) + Refresh Token (30d)
    ↓
[客户端] 存储 JWT (Keychain/Keystore) + Refresh Token
    ↓
[后续请求] Authorization: Bearer <jwt>
    ↓
[JWT 24h 过期]
    ↓
POST /api/v1/auth/refresh { refreshToken }
    ↓
[服务端] 颁发新 JWT (24h) + 旋转 Refresh Token
```

**关键点：**
- **进度合并：** 匿名进度关联到正式账号时，**取最大进度**（避免丢失）
- **Refresh Token 旋转：** 每次刷新颁发新 Refresh Token，旧 Token 失效
- **设备绑定：** JWT 中包含 `deviceId` claim，跨设备使用需重新验证

### 2.3 跨设备同步鉴权 (L3)

> E15 /sync 端点专属鉴权。

```
[玩家在新设备登录]
    ↓
[新设备 L2 正式玩家流程] 拿到 JWT (新)
    ↓
POST /api/v1/sync { sourcePlatform, targetPlatform, saveData, clientTimestampMs }
    ↓
[服务端]
    1. 验证 JWT 有效性
    2. 检查 playerId 是否一致 (跨设备应该是同一 playerId)
    3. 检查时间戳 (防重放)
    4. 检查 saveData 签名 (HMAC-SHA256)
    5. 合并或冲突检测
    ↓
[响应 200] resolvedSave
[响应 409] SyncConflict (Last-Write-Wins)
```

**关键点：**
- 跨设备同步需要 **同一 playerId**（同一正式账号）
- 跨账号同步 → `CROSS_ACCOUNT_SYNC_FORBIDDEN` 403
- 时间戳偏差 > 5min → 拒绝（防重放）

## 3. Token 规格 (Token Specifications)

### 3.1 JWT 规格

```json
{
  "alg": "HS256",
  "typ": "JWT"
}

{
  "sub": "player_x9y8z7",
  "deviceId": "dev_a1b2c3d4e5f6",
  "platform": "steam",
  "platformUserId": "76561198000000000",
  "displayName": "Aha King",
  "iat": 1719652800,
  "exp": 1719739200,
  "iss": "api.anzhong.example",
  "aud": "anzhong-game",
  "scope": ["player.profile", "player.progress", "leaderboard.read"]
}
```

| 字段 | 含义 | 必需 | 示例 |
|------|------|:----:|------|
| `sub` | playerId | ✅ | `player_x9y8z7` |
| `deviceId` | 设备 ID | ✅ | `dev_a1b2c3d4e5f6` |
| `platform` | 平台枚举 | ✅ | `steam` |
| `platformUserId` | 平台用户 ID | ❌ | `76561198000000000` |
| `displayName` | 玩家显示名 | ❌ | `Aha King` |
| `iat` | 颁发时间 | ✅ | 1719652800 |
| `exp` | 过期时间 | ✅ | 1719739200 (24h) |
| `iss` | 颁发者 | ✅ | `api.anzhong.example` |
| `aud` | 受众 | ✅ | `anzhong-game` |
| `scope` | 权限范围 | ✅ | array |

### 3.2 Refresh Token 规格

- **格式：** 256-bit 随机字符串（base64url 编码）
- **存储：** 服务端哈希存储（不存明文）
- **生命周期：** 30 天
- **旋转：** 每次刷新颁发新 Token，旧 Token 失效
- **撤销：** 玩家主动登出 / 设备丢失 / 异常检测

### 3.3 Token 生命周期

```
[颁发 JWT] 
    ↓ 24h
[过期] → 自动 refresh
    ↓ 30d (Refresh Token 有效期)
[Refresh 过期] → 重新登录 (平台 OAuth)
    ↓
[玩家主动登出] → 撤销 JWT + Refresh Token
    ↓
[设备丢失/异常] → 撤销所有 Token (需重新登录)
```

## 4. 7 平台 OAuth 集成 (Platform OAuth Integration)

> 与 11 §1.1 7 平台覆盖矩阵严格对齐。

| 平台 | 优先级 | v1.0 | v1.1 | v2.0 | OAuth 流程 |
|------|:------:|:----:|:----:|:----:|----------|
| **Steam** | P0 | ✅ | ✅ | ✅ | Steam OpenID 2.0 |
| **Mac (Apple Silicon)** | P0 | ✅ | ✅ | ✅ | Sign in with Apple |
| **PS5** | P2 | ❌ | ❌ | ✅ | PSN OAuth 2.0 |
| **Xbox Series X\|S** | P2 | ❌ | ❌ | ✅ | Xbox Live OAuth 2.0 |
| **Nintendo Switch** | P1 | ❌ | ✅ | ✅ | Nintendo Account OAuth 2.0 |
| **iOS** | P2 | ❌ | ❌ | ✅ | Sign in with Apple |
| **Android** | P2 | ❌ | ❌ | ✅ | Google Sign-In |

**v1.0 上线支持：** Steam + Mac (2 平台)
**v1.1 扩展：** + Nintendo Switch (1 平台)
**v2.0 全平台：** + PS5 + Xbox + iOS + Android (4 平台)

### 4.1 Steam OpenID 集成

```
[客户端] 玩家点击"使用 Steam 登录"
    ↓
[客户端] 打开 Steam OpenID URL
    ↓
[Steam] 玩家在 Steam 客户端确认
    ↓
[Steam 回调] 返回 OpenID 验证 token
    ↓
POST /api/v1/auth/platform-login { deviceId, platform: "steam", openIdToken }
    ↓
[服务端]
    1. 验证 OpenID token (向 Steam 验证 endpoint POST)
    2. 拿到 steamid + displayName
    3. 创建/查找 playerId
    4. 颁发 JWT + Refresh Token
```

### 4.2 Sign in with Apple 集成 (Mac / iOS)

```
[客户端] 玩家点击"使用 Apple 登录"
    ↓
[客户端] 调 Apple AuthenticationServices
    ↓
[Apple] 玩家 Face ID / Touch ID 验证
    ↓
[Apple 回调] 返回 identityToken (JWT)
    ↓
POST /api/v1/auth/platform-login { deviceId, platform: "mac", identityToken }
    ↓
[服务端]
    1. 验证 Apple identityToken (验签 + 检查 iss/aud/exp)
    2. 拿到 appleUserId + displayName (可选)
    3. 创建/查找 playerId
    4. 颁发 JWT + Refresh Token
```

### 4.3 Nintendo Switch / PS5 / Xbox 集成 (v1.1+)

> 实施时由平台对接工程师按各自 SDK 文档实现。流程类似：OAuth 2.0 Authorization Code Flow。

## 5. 设备绑定 (Device Binding)

> **目的：** 防止 JWT 泄露被异地使用。

### 5.1 设备指纹 (Device Fingerprint)

```json
{
  "deviceId": "dev_a1b2c3d4e5f6",
  "platform": "steam",
  "osVersion": "Windows 11 23H2",
  "cpuArch": "x86_64",
  "screenResolution": "1920x1080",
  "locale": "zh-CN",
  "timezone": "Asia/Shanghai"
}
```

**生成规则：**
- `deviceId`：UUID v4，首次启动生成，写入注册表/Keychain
- 其他字段：客户端采集，服务端哈希后存储

### 5.2 设备绑定策略

| 场景 | 策略 | 行为 |
|------|------|------|
| **同设备登录** | 允许 | 设备指纹匹配，无需重新验证 |
| **新设备登录** | 需二次验证 | 发送邮箱/短信验证码（如果玩家绑定了） |
| **设备指纹漂移** | 警告 + 验证 | OS 升级等正常漂移允许；异常漂移需重新验证 |
| **JWT 异地使用** | 拒绝 | 设备指纹不匹配 → 401 + 触发风控 |

### 5.3 多设备管理

> 玩家可在"账号设置"查看/撤销已绑定设备，最多 5 台。

```
GET  /api/v1/auth/devices       — 列出已绑定设备
POST /api/v1/auth/devices/{id}/revoke  — 撤销设备
POST /api/v1/auth/devices/{id}/trust   — 信任设备 (跳过二次验证)
```

## 6. 客户端 Token 存储 (Client Token Storage)

| 平台 | 存储位置 | 加密 |
|------|---------|------|
| **Windows** | Windows Credential Manager (DPAPI) | ✅ OS 级 |
| **macOS** | Keychain | ✅ OS 级 |
| **Linux** | libsecret (GNOME Keyring / KWallet) | ✅ 库级 |
| **iOS** | Keychain | ✅ OS 级 |
| **Android** | EncryptedSharedPreferences (AES-256) | ✅ 应用级 |
| **PS5 / Xbox / Switch** | 平台 SDK Secure Storage | ✅ 平台级 |

**关键点：**
- **不存 Refresh Token 明文在磁盘**（即使加密）
- **JWT 短期 (24h) 泄露风险可控**
- **登出时清除所有 Token**

## 7. 安全策略 (Security Policies)

### 7.1 Token 颁发

- JWT 签名算法：HS256 (HMAC-SHA256)
- 密钥：32 字节随机，**每 90 天轮换**
- Refresh Token：256-bit 随机，**一次性使用**
- iss / aud 强制验证

### 7.2 Token 验证

- 服务端**始终**验证 iss / aud / exp / sub
- 设备指纹**软匹配**（漂移容差）
- 异常 IP / 异常时段 → 触发风控（v1.1 评估）

### 7.3 Token 撤销

| 场景 | 撤销范围 |
|------|---------|
| 玩家主动登出 | 当前设备 JWT + Refresh Token |
| 设备丢失 | 该设备所有 Token |
| 账号被盗 | 所有设备所有 Token |
| 平台账号注销 | 该平台关联的 playerId |

### 7.4 防滥用

- **登录频率限制：** 10 次/h/IP
- **JWT 刷新频率限制：** 100 次/h/playerId
- **跨设备同步频率限制：** 10 次/h/playerId
- **异常检测：** 同一 playerId 在 > 3 设备同时活跃 → 触发风控

## 8. 隐私与合规 (Privacy & Compliance)

> 与 11 §5 合规与法务对齐。

| 法律 | 要求 | 实施 |
|------|------|------|
| **GDPR (欧盟)** | 数据可携带、被遗忘权 | 提供 `/api/v1/player/data-export` + `/api/v1/player/delete` |
| **CCPA (加州)** | 数据访问权 | 同上 |
| **中国个人信息保护法** | 同意书、最小化 | 首次启动显示同意书，仅采集必要字段 |
| **COPPA (美国儿童)** | < 13 岁需家长同意 | 玩家自报年龄，< 13 禁用排行榜 + 关闭社交 |

**采集字段最小化原则：**
- ✅ 必采：deviceId / platform / clientVersion
- ❌ 不采：邮箱 / 真实姓名 / 地理位置（仅 timezone 用于 Hint 时区）

## 9. 鉴权错误码 (Auth-Specific Error Codes)

> 详见 `error-codes.md`，这里只列鉴权专属。

| errorCode | HTTP | 含义 | 客户端处理 |
|-----------|:----:|------|----------|
| `UNAUTHORIZED` | 401 | JWT 失效/过期 | 刷新 Token → 重试 |
| `CLIENT_TIME_DRIFT` | 401 | 客户端时钟漂移 > 5min | 校准时钟 |
| `CROSS_ACCOUNT_SYNC_FORBIDDEN` | 403 | 跨账号同步 | 提示"账号不一致" |
| `CLIENT_VERSION_TOO_OLD` | 400 | 客户端版本过低 | 强制升级 |
| `PLATFORM_OAUTH_FAILED` | 502 | 平台 OAuth 失败 | 重试 + 提示 |
| `DEVICE_FINGERPRINT_MISMATCH` | 401 | 设备指纹不匹配 | 二次验证 |
| `REFRESH_TOKEN_EXPIRED` | 401 | Refresh Token 过期 | 重新登录 |
| `REFRESH_TOKEN_REVOKED` | 401 | Refresh Token 已撤销 | 重新登录 |

## 10. 鉴权 ↔ 端点对照 (Auth × Endpoint Matrix)

| 端点 | L1 匿名 | L2 JWT | L3 OAuth | 备注 |
|------|:-------:|:------:|:--------:|------|
| E01 sessions | ✅ | ✅ | ❌ | 匿名启动 |
| E02 sessions end | ❌ | ✅ | ❌ | 需 JWT |
| E03-E07 rooms | ❌ | ✅ | ❌ | 需 JWT |
| E08 saves | ❌ | ✅ | ❌ | 需 JWT |
| E09 saves GET | ❌ | ✅ | ❌ | 需 JWT |
| E10 leaderboard | ❌ | ✅ | ❌ | 需 JWT |
| E11-E12 progress/feedback | ❌ | ✅ | ❌ | 需 JWT |
| E13-E14 settings | ❌ | ✅ | ❌ | 需 JWT |
| E15 sync | ❌ | ❌ | ✅ | 需 OAuth + JWT |
| E16 telemetry | ❌ | ✅ | ❌ | 需 JWT |
| E17 hints | ❌ | ✅ | ❌ | 需 JWT |
| E18 chapters | ❌ | ✅ | ❌ | 需 JWT |
| **auth/login** | ✅ | — | ✅ | 颁发 Token |
| **auth/refresh** | — | ✅ | — | 刷新 Token |
| **auth/logout** | — | ✅ | — | 撤销 Token |

## 11. 关联文档 (Cross-References)

- [`api-spec.yaml`](./api-spec.yaml) — OpenAPI 3.0 (securitySchemes)
- [`endpoints.md`](./endpoints.md) — 端点详解
- [`error-codes.md`](./error-codes.md) — 错误码
- [`rate-limiting.md`](./rate-limiting.md) — 限流
- [`versioning.md`](./versioning.md) — 版本演进
- [`../../docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台 + 合规
- [`../../docs/05-numerical-design-v2.md`](../../docs/05-numerical-design-v2.md) — 跨设备同步进度 (F5)

## 12. 关联代码模块 (Code Modules)

| 模块 | 路径 | 状态 | 引用 |
|------|------|:----:|------|
| `AuthClient` | `src/Api/Client/Auth/AuthClient.cs` | 待创建 | JWT + OAuth |
| `TokenStore` | `src/Api/Client/Auth/TokenStore.cs` | 待创建 | Keychain/Keystore |
| `JwtValidator` | `src/Api/Server/Auth/JwtValidator.cs` | 待创建 | 服务端验签 |
| `DeviceFingerprint` | `src/Api/Client/Auth/DeviceFingerprint.cs` | 待创建 | 设备指纹 |
| `PlatformOAuth` | `src/Api/Server/Auth/Platforms/*.cs` | 待创建 | 7 平台 |

## 13. 待办事项 (TODO)

- [ ] **P0：** Steam OpenID 集成（v1.0 阻塞）
- [ ] **P0：** Sign in with Apple 集成（Mac v1.0 阻塞）
- [ ] **P0：** 设备指纹 + Token 存储实现（v1.0 阻塞）
- [ ] **P1：** Nintendo Switch OAuth（v1.1）
- [ ] **P1：** PSN / Xbox / iOS / Android OAuth（v2.0）
- [ ] **P1：** 异常检测 + 风控（v1.1 评估）
- [ ] **P2：** 隐私合规接口（GDPR/CCPA 数据导出/删除）

## 14. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| R-01 | **JWT 泄露导致玩家进度被改** | 高 | 10% | Token 短期 (24h) + Refresh Token + 设备绑定 | 已规划 |
| R-02 | **设备指纹被绕过（攻击者伪造）** | 高 | 20% | 二次验证 + 异常 IP 检测 | 已规划 |
| R-03 | **平台 OAuth 不可用（Steam/PSN 宕机）** | 中 | 30% | 备用匿名登录 + 提示 | 已规划 |
| R-04 | **跨设备同步冲突数据丢失** | 中 | 20% | Last-Write-Wins + 冲突时本地提示 | 已规划 |
| R-05 | **GDPR 合规实施复杂** | 中 | 40% | 数据导出/删除接口 + 同意书 | 已规划 |
| Q-01 | **是否支持账号合并（多平台账号合一）** | 中 | — | v1.0 不支持；v1.1 评估 | 倾向 v1.1 |
| Q-02 | **是否支持生物识别登录（Face ID / Touch ID）** | 低 | — | 平台原生支持，自动启用 | 已规划 |
| Q-03 | **是否支持邮箱/手机号二次验证** | 中 | — | v1.0 仅设备绑定；v1.1 评估 | 倾向 v1.1 |

## 15. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** 3 层鉴权 (L1 匿名 / L2 JWT / L3 OAuth) + 7 平台分 3 档 + JWT/Refresh Token 规格 + 设备指纹 + 客户端存储 + 安全策略 + 隐私合规 + 18 端点鉴权对照。 |
