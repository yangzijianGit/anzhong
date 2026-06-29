---
title: 《暗室》跨平台音频 (Cross-Platform Audio)
doc_id: DES-anzhong-music-cross-platform
parent: design/music/README.md
last_updated: 2026-06-30
version: v1.0
status: draft
owner: 音乐总监（中书省 subagent · MUSIC-01）
---

# 《暗室》跨平台音频（design/music/cross-platform.md）

> **一句话定位：** 7 平台音频规格 × 7 平台文件格式 × 7 平台音量统一 × 7 平台空间音频 × 7 平台性能预算 × 7 平台分发策略 × 7 平台合规。

## 目的 (Purpose)

本文档是《暗室》音乐层**跨平台适配**的完整定义。它向：

- **音乐总监 / 音频工程师** — 讲清**7 平台音频规格** + **7 平台文件格式选型** + **7 平台音量统一规范**
- **Unity 工程师** — `PlatformAudioAdapter.cs` 模块的 7 平台 AudioImporter 配置
- **平台发布负责人** — **7 平台合规**清单（ESRB / PEGI / CERO / IARC + GDPR + 平台协议）
- **QA / 测试** — **7 平台性能预算**（60/30 FPS + 100/80/50 MB + ≤16ms 延迟）的验收基线
- **外包商** — **7 平台分发策略**（Itch.io 1-1~1-5 + Steam 全套 + Switch v1.1 + PS/Xbox/iOS/Android v2.0）

**本文档范围：** **跨平台（Cross-Platform）**——7 平台的音频格式 / 编码 / 通道 / 性能 / 合规差异。

## 1. 7 平台音频规格 (7-Platform Audio Spec)

> **源约束：** `docs/11-release-v2.md` §1 平台选择 + §5 合规与法务 + `design/implementation/deployment-runbook.md` §2-4 7 平台部署

### 1.1 7 平台音频规格总览 (7-Platform Audio Spec Overview)

| 平台 | 选型理由 | 一次性成本 | 持续成本 | 音频规格 | 风险 | 优先级 |
|------|---------|:---------:|:--------:|---------|------|:----:|
| **PC Steam** | 大流量 + Indie 友好 + $4.99 | $100 | Steam Direct 30% | Opus (主) + WAV 源 + 立体声 + Dolby Atmos 5.1/7.1 | 审核 ≥ 7 工作日 | **P0** (M11/M12) |
| **PC Mac** | Steam 共享 / Apple Silicon 原生 | $0 | Steam 30% | Opus + ALAC + 立体声 + Dolby Atmos | Metal 渲染兼容 | **P0** (M11/M12) |
| **PS5** | Indie 主机主流 / Trophy / DualSense | $0 | Sony 30% + $25K/年 (Indie 减免) | Opus + AAC + 立体声 + Tempest 3D Audio | 认证 ≥ 12 周 / NDA | **P2** (v2.0) |
| **Xbox Series X\|S** | Game Pass 流量 / 跨平台 | $0 | MS 30% | Opus + AAC + 立体声 + Project Acoustics | 认证 ≥ 6 周 / Xbox Live | **P2** (v2.0) |
| **Nintendo Switch** | Indie 独立游戏聚集地 / 便携性 | $0 | Nintendo 30% | **Opus 64 kbps（压缩）** + 立体声 | Lotcheck ≥ 6 周 / 移植 200h | **P1** (v1.1) |
| **iOS (iPhone/iPad)** | App Store 大流量 / 触达易 | $99/年 | Apple 30%（小开发者 15%）| AAC + Opus + 立体声 + Dolby Atmos (iPhone 12+) | 适配成本 / 审核 ≥ 3 天 | **P2** (v2.0) |
| **Android (Google Play)** | 用户基数最大 / 多端覆盖 | $25 | Google 30% | Opus (主) + OGG Vorbis + 立体声 + Dolby Atmos (Android 13+) | 设备碎片化 / 性能优化 | **P2** (v2.0) |

> **关键设计：** v1.0 仅 Steam PC/Mac + Itch.io（1-1 ~ 1-5 试玩版）上线；Switch 在 v1.1 扩展；PS5/Xbox/iOS/Android 在 v2.0 全平台。
> **关键设计：** 7 平台**主旋律统一**（v1.0 制作 1 套主旋律，平台发布时**自动转码**）。

### 1.2 v1.0 → v1.1 → v2.0 平台扩展时间表

```
v1.0 (Day-84, M11/M12):
  ✓ Steam PC + Mac (P0)
  ✓ Itch.io 试玩版（1-1 ~ 1-5）(P0)
  → 音频格式：WAV (源) + Opus 128 kbps (发布)

v1.1 (T+3m, ~90 天后):
  + Nintendo Switch (P1, 200h 移植工作量)
  → 音频格式：Opus 64 kbps（压缩）

v2.0 (T+6m, ~180 天后):
  + PS5 + Xbox Series X|S + iOS + Android (P2, 合计 560h)
  → 音频格式：Opus / AAC / OGG Vorbis（依平台）
```

> **关键设计：** 音频**不重新制作**——v1.0 制作 1 套主旋律，v1.1/v2.0 平台发布时**自动转码**（Opus / OGG / AAC / ALAC）。

## 2. 7 平台音频文件格式 (7-Platform Audio File Format)

> **设计原则：** **7 种编码格式选最优**——Opus（主）/ OGG Vorbis / MP3 / FLAC / Opus / AAC / ALAC。

### 2.1 7 平台文件格式选型表 (7-Platform Format Selection)

| 平台 | 源文件 | 发布格式 | 编码码率 | 采样率 | 比特深度 | 通道数 | 备注 |
|------|--------|---------|---------|--------|---------|--------|------|
| **PC Steam** | WAV 16-bit 44.1kHz | **Opus** (主) + FLAC (备) | 128 kbps (Opus) | 44.1 kHz | 16-bit | 立体声 | Opus 开源 + 跨平台 |
| **PC Mac** | WAV 16-bit 44.1kHz | **Opus** (主) + ALAC (备) | 128 kbps (Opus) | 44.1 kHz | 16-bit | 立体声 | Opus 开源 + ALAC Apple 原生 |
| **PS5** | WAV 16-bit 44.1kHz | **Opus** (主) + AAC (备) | 128 kbps (Opus) | 44.1 kHz | 16-bit | 立体声 + Tempest 3D | Opus + Sony Tempest |
| **Xbox Series X\|S** | WAV 16-bit 44.1kHz | **Opus** (主) + AAC (备) | 128 kbps (Opus) | 44.1 kHz | 16-bit | 立体声 + Project Acoustics | Opus + MS Project Acoustics |
| **Nintendo Switch** | WAV 16-bit 44.1kHz | **Opus 64 kbps**（压缩）| **64 kbps** | **22.05 kHz** | **8-bit** | 立体声 | Switch 带宽限制 |
| **iOS (iPhone/iPad)** | WAV 16-bit 44.1kHz | **AAC** (主) + Opus (备) | 128 kbps (AAC) | 44.1 kHz | 16-bit | 立体声 | AAC Apple 原生 |
| **Android** | WAV 16-bit 44.1kHz | **Opus** (主) + OGG Vorbis (备) | 128 kbps (Opus) | 44.1 kHz | 16-bit | 立体声 | Opus + OGG Android 原生 |

> **关键设计：** 源文件**统一 WAV 16-bit 44.1kHz 立体声**——平台发布时**自动转码**（无需重新制作）。
> **关键设计：** **Opus** 是 7 平台中**最通用**的编码——开源 + 高压缩比 + 低延迟 + 全平台支持。

### 2.2 7 平台文件格式大小估算 (7-Platform File Size Estimation)

```
源文件 (WAV 16-bit 44.1kHz 立体声):
  1 秒音频大小 = 44100 × 2 (声道) × 2 字节 = 176.4 KB/s
  9 类音频总时长：~120s（28 文件 + 5 BGM）
  WAV 源文件总大小：~21 MB

Opus 128 kbps（发布格式）:
  1 秒音频大小 = 128 / 8 = 16 KB/s
  Opus 发布总大小：~2 MB
  压缩率：~10x（vs WAV 源文件）

Opus 64 kbps (Switch 压缩):
  1 秒音频大小 = 64 / 8 = 8 KB/s
  Opus Switch 总大小：~1 MB
  压缩率：~20x（vs WAV 源文件）

总音频文件预算：≤ 50 MB（PC/Mac/PS/Xbox/iOS/Android）+ ≤ 25 MB（Switch）
```

> **关键设计：** **Opus 压缩比 10x + Switch 20x**——保证 7 平台音频**总大小 ≤ 50 MB**（PC） / ≤ 25 MB（Switch）（与 09-v2 §8 性能约束 50 MB 一致）。

## 3. 7 平台音量统一 (7-Platform Volume Normalization)

> **设计原则：** **各平台基准 -12dB** + **自动归一化**——确保玩家跨平台听到的音量**一致**（与 09-v2 §9.3 LUFS 规范化 + design/api/api-spec.yaml M09 AudioSettings 一致）。

### 3.1 7 平台音量统一规范 (7-Platform Volume Normalization)

| 资产类型 | 目标 LUFS | 允许范围 | 引用 |
|---------|:--------:|:-------:|------|
| **SFX（切换 / 重置 / 错音 / 教学）** | -16 LUFS | -18 ~ -14 LUFS | 09-v2 §9.3 |
| **BGM（章节）** | -18 LUFS | -20 ~ -16 LUFS | 09-v2 §9.3 |
| **环境音（室内 / 走廊 / 电流）** | -22 LUFS | -24 ~ -20 LUFS | 09-v2 §9.3 |
| **语音 TTS** | -14 LUFS | -16 ~ -12 LUFS | 09-v2 §9.3 |

> **关键设计：** **LUFS (Loudness Units Full Scale)** 是行业标准（EBU R128）——比 dBFS 更精确感知响度（考虑人耳非线性）。

### 3.2 7 平台音量归一化策略 (7-Platform Volume Normalization Strategy)

```
源文件 (WAV + LUFS 规范化)
       ↓
   [自动 LUFS 检测 + 归一化]
       ↓
PC/Mac/PS/Xbox/iOS/Android (-12dB 基准 + Opus/AAC/OGG/ALAC)
Nintendo Switch (-12dB 基准 + Opus 64 kbps 压缩)

音量归一化数据流：
1. AudioImporter 读取 WAV 文件
2. LUFS Meter 检测实际 LUFS
3. AudioImporter.Normalize(target_LUFS = -16 / -18 / -22 / -14)
4. AudioImporter.Export(opus/aac/ogg/alac, bitrate = 128/64)
```

> **关键设计：** **音频检测在源文件层**——保证所有发布平台**起点一致**。

### 3.3 7 平台音量 PlayerPrefs 配置 (7-Platform Volume Prefs)

```csharp
// src/Audio/PlatformAudioAdapter.cs
public class PlatformAudioPrefs
{
    public float MasterVolume = 0.8f;            // 0-1 范围
    public float SfxVolume = 0.7f;               // 0-1 范围
    public float BgmVolume = 0.6f;               // 0-1 范围
    public float AmbientVolume = 0.4f;           // 0-1 范围
    public float UiVolume = 0.5f;                // 0-1 范围
    public bool Muted = false;

    // 7 平台音量基准
    public float PlatformBaseDb = -12f;           // 各平台基准 -12dB
    public float SwitchBaseDb = -12f;             // Switch 基准 -12dB（与 PC 一致）
    public float MobileBaseDb = -15f;             // 移动端基准 -15dB（防爆音）
}
```

> **关键设计：** 移动端基准 -15dB（比 PC -12dB 略低）——防止手机扬声器爆音。

## 4. 7 平台空间音频 (7-Platform Spatial Audio)

> **设计原则：** **PC/PS5/Xbox Series/Switch OLED/iPhone/Android 旗舰/Mac M1+** 支持 **Dolby Atmos**；其他平台：立体声。

### 4.1 7 平台空间音频对照表 (7-Platform Spatial Audio)

| 平台 | 空间音频 | 实现 | 启用条件 | 优先级 |
|------|---------|------|---------|:----:|
| **PC Steam** | **Dolby Atmos 5.1/7.1** | Dolby Atmos for Games / Windows Sonic | 玩家耳机 / 5.1/7.1 音箱 | **P0** |
| **PC Mac** | **Dolby Atmos** | macOS Spatial Audio | Apple Silicon Mac + AirPods Pro | **P0** |
| **PS5** | **Tempest 3D Audio** | Sony 官方引擎 | PS5 主机 + 耳机 | **P2** |
| **Xbox Series X\|S** | **Project Acoustics** | MS 官方引擎 | Xbox Series + 耳机 | **P2** |
| **Nintendo Switch** | **立体声**（无空间）| 立体声降混 | 全部 | **P1** |
| **iOS (iPhone 12+)** | **Dolby Atmos** | iOS Spatial Audio | iPhone 12+ + AirPods Pro | **P2** |
| **Android (Android 13+)** | **Dolby Atmos** | Android 13+ Spatial Audio | Android 13+ + 旗舰机 | **P2** |

> **关键设计：** **当前距 Switch 房屋**（450 m）**Dolby Atmos** 在 5.1/7.1 多音箱系统下提供"沉浸式"——玩家可感知方向与距离。
> **关键设计：** Switch **仅立体声**（无空间音频）——带宽 + 性能限制（与 11-v2 §1.1 Switch 风险一致）。

### 4.2 7 平台空间音频技术细节 (7-Platform Spatial Audio Tech)

```
PC + PS5 + Xbox + iPhone + Android 旗舰:
  电流声方向感 (L4 氛围层) → Dolby Atmos 5.1/7.1 渲染
  - 玩家距 SwitchSlot 距离 → 各声道音量不同
  - 玩家朝向 → 电流声在前方 / 后方
  - 玩家高度 → 电流声在水平面 / 上方 / 下方

Switch + 移动端普通机:
  电流声方向感 (L4 氛围层) → 立体声 pan
  - 玩家距 SwitchSlot 距离 → L/R 声道 pan
  - 玩家朝向 → pan 偏移
  - 玩家高度 → 不支持（仅立体声水平）
```

> **关键设计：** **电流声方向感** 是 06-v2 §11.1.2 沉浸感细节——7 平台**统一支持**（Dolby Atmos / 立体声 pan 两种模式）。

## 5. 7 平台性能预算 (7-Platform Performance Budget)

> **设计原则：** **60 FPS (PC/PS/Xbox/iOS/Android) / 30 FPS (Switch)** + **16-bit 44.1kHz** + **通道数** + **延迟 ≤16ms** + **内存预算**——确保 7 平台音频性能**稳定**（与 09-v2 §8 性能约束 6 项 一致）。

### 5.1 7 平台性能预算对照表 (7-Platform Performance Budget)

| 平台 | 帧率 | 音频采样率 | 比特深度 | 通道数 | 音频延迟 | 音频 CPU | 音频内存 | 同时发声 | BGM 通道 |
|------|:---:|:--------:|:-------:|:------:|:-------:|:-------:|:-------:|:-------:|:-------:|
| **PC Steam** | 60 FPS | 44.1 kHz | 16-bit | 立体声 + Atmos 5.1/7.1 | ≤ 16 ms | ≤ 5% (单核) | ≤ 30 MB | ≤ 8 | 2 |
| **PC Mac** | 60 FPS | 44.1 kHz | 16-bit | 立体声 + Atmos | ≤ 16 ms | ≤ 5% | ≤ 30 MB | ≤ 8 | 2 |
| **PS5** | 60 FPS | 44.1 kHz | 16-bit | 立体声 + Tempest 3D | ≤ 16 ms | ≤ 3% | ≤ 30 MB | ≤ 8 | 2 |
| **Xbox Series X\|S** | 60 FPS | 44.1 kHz | 16-bit | 立体声 + Project Acoustics | ≤ 16 ms | ≤ 3% | ≤ 30 MB | ≤ 8 | 2 |
| **Nintendo Switch** | 30 FPS | **22.05 kHz** | **8-bit** | 立体声 | ≤ 32 ms | ≤ 4% | ≤ 20 MB | ≤ 6 | 1（BGM 1 + Ambient 1 = 2 总通道）|
| **iOS** | 60 FPS | 44.1 kHz | 16-bit | 立体声 + Atmos | ≤ 16 ms | ≤ 4% | ≤ 25 MB | ≤ 6 | 2 |
| **Android** | 60 FPS | 44.1 kHz | 16-bit | 立体声 + Atmos | ≤ 16 ms | ≤ 5% | ≤ 25 MB | ≤ 6 | 2 |

> **关键设计：** **Nintendo Switch 性能预算降低**（30 FPS + 22.05 kHz + 8-bit + ≤32ms 延迟）——与 11-v2 §3.1 移植工作量 200h 一致（Switch 移植时性能预算调整）。
> **关键设计：** **音频延迟 ≤16ms**（与 08-v2 §7 反馈 3 层同步契约一致）——除 Switch ≤32ms 例外。

### 5.2 7 平台音频 CPU / 内存基准 (7-Platform CPU/Memory Baseline)

```
PC / Mac / PS5 / Xbox:
  音频 CPU ≤ 5% (单核, Profiler Audio)
  音频内存 ≤ 30 MB (Profiler Memory)
  音频文件总大小 ≤ 50 MB (Resources 统计)

Switch:
  音频 CPU ≤ 4%
  音频内存 ≤ 20 MB (带宽限制)
  音频文件总大小 ≤ 25 MB (50 MB 的 50%)

iOS / Android:
  音频 CPU ≤ 4-5%
  音频内存 ≤ 25 MB (移动端限制)
  音频文件总大小 ≤ 50 MB
```

> **关键设计：** **资源总量限制**——总音频资源必须**符合平台限制**——Switch 25 MB 是 PC 50 MB 的 50%。

## 6. 7 平台分发策略 (7-Platform Distribution Strategy)

> **设计原则：** **Itch.io 试玩版 → Steam v1.0 正式 → Switch v1.1 → PS/Xbox/iOS/Android v2.0**——分阶段发布 + 按平台优先级（与 10-v2 12 里程碑 + 11-v2 §3 发布计划 一致）。

### 6.1 7 平台分发阶段表 (7-Platform Distribution Phases)

| 阶段 | 时间 | 上线平台 | 内容范围 | 音频规格 |
|------|------|---------|---------|---------|
| **W11 (M11)** | 2026-09 | **Itch.io 试玩版**（P0）| 1-1 ~ 1-5 | Opus 128 kbps + 立体声 + 22 KB/s 总 |
| **W12 (M12)** | 2026-09 | **Steam PC/Mac**（P0）| 19 房间完整 | Opus 128 kbps + 立体声 + Dolby Atmos + ~2 MB |
| **v1.1 (T+3m)** | 2026-12 | **Nintendo Switch**（P1）| 19 房间 + 200h 移植工作量 | Opus 64 kbps + 立体声 + ~1 MB |
| **v2.0 (T+6m)** | 2027-03 | **PS5 + Xbox + iOS + Android**（P2）| 19 房间 + 560h 移植工作量 | Opus / AAC / OGG + Dolby Atmos / Tempest 3D / Project Acoustics |

> **关键设计：** **Itch.io 试玩版仅 1-1 ~ 1-5**——5 房间 + 试玩版时长 ≤ 30 min（与 Itch.io 试玩版时长限制一致）。
> **关键设计：** **Steam 1.0 完整版** —— 19 房间 + 试玩版时长 3-5h（与 11-v2 §2.1 4 定价模式 + §3.1 4 阶段发布计划 一致）。

### 6.2 7 平台分发格式 (7-Platform Distribution Format)

```
Itch.io (P0):
  - 文件：HTML5 Build + 独立 Build (Win64)
  - 音频：Opus 128 kbps + 立体声 + 总 ~2 MB
  - 价格：Free (试玩版) / Pay What You Want (可选)

Steam PC/Mac (P0):
  - 文件：Steam Build + depots
  - 音频：Opus 128 kbps + 立体声 + Dolby Atmos + 总 ~2 MB
  - 价格：$4.99 (USD, 6 区域差异定价)
  - DLC：豪华版 + 数字画集 9 张 + 制作花絮 1 分钟

Nintendo Switch (P1, v1.1):
  - 文件：Nintendo Switch Build (eShop submission)
  - 音频：Opus 64 kbps + 立体声 + 总 ~1 MB
  - 价格：$4.99 USD

PS5 + Xbox + iOS + Android (P2, v2.0):
  - 文件：Sony PS5 Build + MS Xbox Build + Apple iOS Build + Google Android APK
  - 音频：Opus / AAC / OGG Vorbis / ALAC + Dolby Atmos / Tempest 3D / Project Acoustics + 总 ~2 MB
  - 价格：$4.99 USD（PS5 / Xbox）+ $2.99 USD（iOS / Android, mobile pricing）
```

> **关键设计：** **音频格式依平台选最优**——PC Opus / Mac ALAC / iOS AAC / Android OGG Vorbis / Switch Opus 64 kbps。

### 6.3 7 平台分发回滚策略 (7-Platform Distribution Rollback)

| 平台 | 回滚机制 | 回滚时长 | 引用 |
|------|---------|---------|------|
| **Steam PC/Mac** | Steam Branch (default / beta / canary) | 1-3 工作日 | 11-v2 §7.1 |
| **Itch.io** | 重新上传新版本 + 旧版本保留 | < 1 小时 | 11-v2 §7.2 |
| **Nintendo Switch** | Nintendo eShop Lotcheck 重新提交 | 6 周（与首次同）| 11-v2 §7.4 |
| **PS5 / Xbox** | Console store rollback（紧急）| 1-2 周 | 11-v2 §7.4 |
| **iOS** | App Store 紧急更新（平均 1-2 天）| 1-3 天 | 11-v2 §7.3 |
| **Android** | Google Play 紧急回滚（30 分钟）| 30 分钟 | 11-v2 §7.3 |

> **关键设计：** **iOS / Android 回滚最快**——30 分钟 ~ 1-3 天——紧急热修首选移动端。

## 7. 7 平台合规 (7-Platform Compliance)

> **设计原则：** **6 项合规清单**（分级 / 隐私 / GDPR / 退款 / 版权 / 平台协议）+ **5 区域 IARC**——保证 7 平台**合规上架**（与 11-v2 §5 合规与法务 一致）。

### 7.1 6 项合规清单 (6 Compliance Items)

| # | 合规项 | v1.0 计划 | 状态 | 触发 | 音频相关 | 引用 |
|---|--------|---------|:---:|------|--------|------|
| **1. 分级 (Age Rating)** | ESRB E (NA) / PEGI 3 (EU) / CERO A (JP) / 全年龄 (CN) / GRAC ALL (KR) | IARC 通用问卷 | M11 必填 | 全部 | 11-v2 §5.1 |
| **2. 隐私政策 (Privacy Policy)** | URL (GitHub Pages) — 仅本地存档，不收集 PII | M10 完成 | 必填 | 全部（音频设置不收集 PII） | 11-v2 §5.3 |
| **3. GDPR (欧盟数据保护)** | 同隐私政策 + 数据导出/删除 API（无服务器）| M10 完成 | 必填（EU） | 全部 | 11-v2 §5.4 |
| **4. 退款政策 (Refund Policy)** | Steam 14 天/2 小时 + Itch.io 不退款（明示）| M11 公告 | 必填 | 全部 | 11-v2 §5.5 |
| **5. 版权 (Copyright)** | CC0 + 自制 + 商用授权（引用 09-v2 §版权表）| M10 完成 | 必填 | 全部（Kenney.nl CC0 + Suno/Udio 商用） | 11-v2 §5.6 + 09-v2 §9 |
| **6. 平台协议 (Platform Agreement)** | Steam Subscriber Agreement + Apple EULA + Sony NDA + MS GDK + Nintendo EULA | M11-M12 | 必填 | 全部 | 11-v2 §5.7 |

> **关键设计：** **音频不收集 PII**——隐私政策**仅适用于键盘设置 + 字体缩放 + 色盲模式**——音频设置（dB / volume）属于本地存档。
> **关键设计：** **GDPR 数据导出 API**——`/api/v1/gdpr/export` + `/api/v1/gdpr/delete`——音频设置包含在导出范围内。

### 7.2 5 大区域分级细则 (5-Region Age Rating)

| 区域 | 评级机构 | 申请 | 费用 | 周期 | 备注 |
|------|---------|------|:----:|:----:|------|
| **NA** | ESRB | IARC 通用问卷 | $0 | 1-3 工作日 | 无战斗 / 无暴力 / 无赌博 → **必 E (Everyone)** |
| **EU** | PEGI | IARC 通用问卷 | $0 | 1-3 工作日 | 跨 27 国 → **必 PEGI 3+** |
| **JP** | CERO | IARC + CERO 提交 | $0 | 7-14 工作日 | 无性 / 无暴力 / 无赌博 → **必 A (全年龄)** |
| **CN** | 无（单机无需版号）| 不需要 | $0 | — | 单机游戏不需版号（2018 规定）|
| **KR** | GRAC | IARC + GRAC 提交 | $0 | 5-10 工作日 | **必 ALL (全年龄)** |

> **关键设计：** **IARC 通用问卷**——1 份问卷 → 5 大区域评级同步生效，工时 < 4h（5 区域 + 1 问卷 + 1 提交）。

### 7.3 GDPR 音频相关合规 (GDPR Audio Compliance)

> **GDPR 6 条款**：lawfulness / fairness / transparency / purpose limitation / data minimization / accuracy / storage limitation / integrity / accountability

```
法律依据（Lawfulness）：
  音频设置（dB / volume / muted）+ 字体缩放 + 色盲模式 — 玩家主动提供同意
  → 不属于 PII（无姓名 / 无邮箱 / 无 IP）
  → 不需要 Cookie 同意

数据最小化（Data Minimization）：
  收集字段：PlayerPrefs.audio.masterVolume + SFX + BGM + Ambient + UI + muted
  收集字段：PlayerPrefs.accessibility.colorblindMode + fontScale + highContrast + reducedMotion + screenReader
  总字段：≤ 10 字段
  不收集：姓名 / 邮箱 / IP / 设备 ID / 游戏时长 / 通关次数（v1.0）

存储限制（Storage Limitation）：
  存档保留时长：玩家主动删除前持续保留
  导出 API：/api/v1/gdpr/export (JSON 格式)
  删除 API：/api/v1/gdpr/delete

透明度（Transparency）：
  隐私政策 URL（GitHub Pages） — M10 完成
  玩家首次启动游戏时显示

数据完整性（Integrity）：
  Protobuf 序列化 + AES-256-GCM 加密（M11 SaveSystem）
  数据导出 JSON 格式 + 数据校验
```

> **关键设计：** 音频设置**不属于 PII**——v1.0 无服务器 / 无数据收集 / 仅本地存档——**GDPR 合规简化**（仅隐私政策 + 导出/删除 API）。

### 7.4 7 平台协议 (7-Platform Agreements)

| 平台 | 协议 | NDA | 申请 | 工时 | 成本 | 优先级 |
|------|------|:---:|------|:----:|:----:|:----:|
| **Steam** | Steam Subscriber Agreement | ❌ | Steamworks SDK | 4h | $100 | **P0** |
| **Apple Mac** | Apple EULA | ❌ | Apple Developer ID | 2h | $0 | **P0** |
| **PS5** | Sony PSN License Agreement | ✅ | Sony Developer Registration | 8h + 12 周 | $0 (Indie) | **P2** |
| **Xbox** | MS GDK License Agreement | ✅ | Microsoft Developer Registration | 8h + 6 周 | $0 (Indie) | **P2** |
| **Nintendo Switch** | Nintendo EULA | ✅ | Nintendo Developer Registration | 8h + 6 周 | $0 (Indie) | **P1** |
| **iOS** | Apple EULA + App Store Review Guidelines | ❌ | Apple Developer ID | 4h | $99/年 | **P2** |
| **Android** | Google Play Developer Distribution Agreement | ❌ | Google Play Developer Console | 4h | $25 | **P2** |

> **关键设计：** **主机平台（PS5 / Xbox / Nintendo）需 NDA + 12 周 / 6 周认证**——v2.0 扩展时**预留时间**（与 11-v2 §1.1 风险一致）。

## 8. 边界条件 (Edge Cases)

1. **玩家在低规格 PC 上运行 v1.0（4GB RAM / 集成显卡）**
   - **触发条件：** 玩家硬件低于推荐配置
   - **音频预期：** 5 层级混音**降级**——L1+L2 持续 + L3+L4+L5 暂时禁用（CPU 限制）

2. **Switch 在掌机模式（720p 30 FPS）vs 主机模式（1080p 60 FPS）**
   - **触发条件：** 玩家切换掌机 / 主机模式
   - **音频预期：** 掌机模式**音频压缩**（Opus 64 kbps 立体声）+ 主机模式**音频全开**（Opus 128 kbps）

3. **移动端音频会话中断（来电 / 通知）**
   - **触发条件：** 玩家接到电话 / 推送通知
   - **音频预期：** 全部音频**暂停**（AudioManager.PauseAll）+ 通话结束后**恢复**

4. **主机平台 Dolby Atmos 不支持（如旧 Soundbar）**
   - **触发条件：** 玩家使用旧 Soundbar（无 Dolby Atmos）
   - **音频预期：** **自动降混**到立体声（PlatformAudioAdapter 兼容）

5. **iOS 后台播放限制（iOS 不允许长时间后台播放）**
   - **触发条件：** 玩家切换到其他 App
   - **音频预期：** **暂停全部音频**——iOS 不允许后台播放 v1.0 无 BGM-only 模式

6. **PS5 / Xbox HDR + Dolby Vision + Dolby Atmos 组合**
   - **触发条件：** 玩家在 PS5 / Xbox + 4K HDR + Dolby Atmos 设备
   - **音频预期：** **自动启用 Atmos**（平台 API 自动检测）

7. **平台发布时音频格式不兼容（罕见）**
   - **触发条件：** 平台 API 限制（如 Switch 早期 SDK 不支持 Opus 64 kbps）
   - **音频预期：** **降级为 OGG Vorbis**（32 kbps）——确保兼容性

## 9. 验收标准 (Acceptance Criteria)

- [x] **AC-01** 7 平台音频规格总览（选型理由 + 成本 + 风险 + 优先级）
- [x] **AC-02** v1.0 → v1.1 → v2.0 平台扩展时间表
- [x] **AC-03** 7 平台文件格式选型表（源 + 发布 + 编码 + 采样率 + 比特 + 通道）
- [x] **AC-04** 7 平台文件格式大小估算（WAV 21 MB + Opus 2 MB + Opus 64 kbps 1 MB）
- [x] **AC-05** 7 平台音量统一规范（LUFS 4 类资产目标 + 平台基准 -12dB）
- [x] **AC-06** 7 平台音量归一化策略（源文件 LUFS 检测 + 自动归一化）
- [x] **AC-07** 7 平台空间音频对照表（Dolby Atmos / Tempest 3D / Project Acoustics / 立体声）
- [x] **AC-08** 7 平台空间音频技术细节（电流声方向感 + Dolby Atmos / 立体声 pan）
- [x] **AC-09** 7 平台性能预算（60/30 FPS + 16/8-bit + 44.1/22.05 kHz + ≤16ms 延迟 + 内存预算）
- [x] **AC-10** 7 平台音频 CPU / 内存基准（PC 5%/30 MB + Switch 4%/20 MB + Mobile 4-5%/25 MB）
- [x] **AC-11** 7 平台分发阶段表（W11 Itch.io + W12 Steam + v1.1 Switch + v2.0 全平台）
- [x] **AC-12** 7 平台分发格式（Itch.io + Steam + Switch + PS5/Xbox/iOS/Android）
- [x] **AC-13** 7 平台分发回滚策略（Steam 1-3 天 / Itch.io < 1 小时 / Switch 6 周 / Mobile 30 分钟-3 天）
- [x] **AC-14** 6 项合规清单（分级 + 隐私 + GDPR + 退款 + 版权 + 平台协议）
- [x] **AC-15** 5 大区域分级细则（NA / EU / JP / CN / KR）+ IARC 通用问卷
- [x] **AC-16** GDPR 音频相关合规（Lawfulness / Data Minimization / Storage / Transparency / Integrity）
- [x] **AC-17** 7 平台协议（Steam / Apple Mac / PS5 / Xbox / Nintendo / iOS / Android）+ NDA + 工时 + 成本
- [x] **AC-18** 边界条件 7 条（每条含触发条件 + 音频预期）
- [x] **AC-19** 与 09-v2 + 11-v2 + design/implementation/deployment-runbook.md 引用 7+ 处

## 10. 关联文档

### 10.1 上游（本文档依赖）

- [`docs/09-audio-v2.md`](../../docs/09-audio-v2.md) §8 性能约束 + §9.3 LUFS 规范化 + §9 版权表
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) §1 7 平台 + §5 6 项合规 + §3.3 软启动
- [`docs/05-numerical-design-v2.md`](../../docs/05-numerical-design-v2.md) §3.4 UI/音频反馈参数
- [`design/implementation/deployment-runbook.md`](../implementation/deployment-runbook.md) §2-4 7 平台部署 + §5 5 区域 IARC + §6 GDPR + §7 回滚策略
- [`standards.md`](./standards.md) §1 文件标准 + §2 编解码 + §3 性能预算

### 10.2 下游（本文档被依赖）

- [`standards.md`](./standards.md) §2 编解码 + §3 性能预算（与本文 §2 + §5 对齐）
- `src/Audio/PlatformAudioAdapter.cs` — 7 平台音频适配
- `data/audio/audio-platform-config.json` — 7 平台音频配置

## 11. 待办事项 (TODO)

- [ ] **P0：** 实现 PlatformAudioAdapter（7 平台 + 编码 / 通道 / 性能 / LUFS）— 阻塞 v1.0（v1.0 仅 Steam + Mac）[§1 + §3 + §6]
- [ ] **P0：** 实现 7 平台文件格式自动转码（Opus / OGG / AAC / ALAC + 128/64 kbps）— 阻塞 v1.0 [§2.1]
- [ ] **P0：** 实现 GDPR 数据导出 / 删除 API（含音频设置）— 阻塞 EU 上架 [§7.3]
- [ ] **P0：** 实现 IARC 通用问卷（5 区域评级 + 4h 工时）— 阻塞 M11 必填 [§7.2]
- [ ] **P0：** 实现 6 项合规清单（Steam Subscriber Agreement + Apple EULA 等）— 阻塞各平台协议 [§7.4]
- [ ] **P1：** 实现 7 平台空间音频检测（自动启用 Dolby Atmos / 立体声降混）— 不阻塞 v1.0 [§4]
- [ ] **P1：** 实现 v1.1 Switch 移植（200h 工作量 + Opus 64 kbps 压缩）— 不阻塞 v1.0 [§1.2]
- [ ] **P2：** 实现 v2.0 PS5/Xbox/iOS/Android 移植（合计 560h 工作量）— 不阻塞 v1.0 [§1.2]

## 12. 变更日志

| 日期 | 版本 | 变更人 | 内容 |
|------|------|--------|------|
| 2026-06-30 | v1.0 | 中书省 subagent (MUSIC-01) | **cross-platform.md 初版:** 7 平台音频规格总览 + v1.0 → v1.1 → v2.0 平台扩展时间表 / 7 平台文件格式选型表（Opus / OGG / AAC / ALAC + 128/64 kbps）+ 7 平台文件大小估算（WAV 21 MB + Opus 2 MB + Opus 64 kbps 1 MB）/ 7 平台音量统一规范（LUFS 4 类资产目标 + 平台基准 -12dB）+ 归一化策略 + PlayerPrefs 配置 / 7 平台空间音频对照表（Dolby Atmos / Tempest 3D / Project Acoustics / 立体声）+ 技术细节（电流声方向感）/ 7 平台性能预算（60/30 FPS + 16/8-bit + 44.1/22.05 kHz + ≤16ms 延迟 + 内存预算）+ CPU/内存基准 / 7 平台分发阶段表（W11 Itch.io + W12 Steam + v1.1 Switch + v2.0 全平台）+ 7 平台分发格式 + 7 平台分发回滚策略 / 6 项合规清单（分级 + 隐私 + GDPR + 退款 + 版权 + 平台协议）+ 5 大区域分级 + GDPR 音频相关合规 + 7 平台协议（NDA + 工时 + 成本）/ 7 边界条件 / 19 验收标准 / 10 关联引用 / 8 待办 |

---

**最后更新：** 2026-06-30
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）
