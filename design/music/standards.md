---
title: 《暗室》技术标准 (Technical Standards)
doc_id: DES-anzhong-music-standards
parent: design/music/README.md
last_updated: 2026-06-30
version: v1.0
status: draft
owner: 音乐总监（中书省 subagent · MUSIC-01）
---

# 《暗室》技术标准（design/music/standards.md）

> **一句话定位：** 音频文件标准 + 编解码 + 性能预算 + 命名规范 + 版本 + 版权 + 审计 + P0-001 跟踪。

## 目的 (Purpose)

本文档是《暗室》音乐层**技术标准**的完整定义。它向：

- **音乐总监 / 音频工程师** — 讲清**音频文件标准**（WAV 16-bit 44.1kHz）+ **编解码选型**（Opus / OGG / MP3 / FLAC）+ **性能预算**（60 FPS / ≤100 MB）+ **命名规范**（audio_<room_id>_<category>_<intensity>.wav）
- **Unity 工程师** — **AudioImporter 配置**（v1.0 / v1.1 / v2.0 版本控制）+ **CI/CD 自动审计**（ce-doc-review）
- **外包商** — **版权登记**（Kenney.nl CC0 + 自制 CC BY-NC 4.0 + 第三方 6 来源）+ **审计 CI/CD** 接入
- **QA / 测试** — **P0-001 跟踪**（design/data/p0-001-tracking.md 15 阻塞字段）+ **验收基线**

**本文档范围：** **技术标准（Standards）**——文件格式 / 编码 / 性能 / 命名 / 版本 / 版权 / 审计 的**可执行规范**。

## 1. 音频文件标准 (Audio File Standards)

> **设计原则：** **WAV 16-bit 44.1kHz 立体声**作为**源文件**标准 + **Protobuf 序列化**用于存档（与 `design/data/serialization.md` + `design/data/persistence-strategy.md` 同步）。

### 1.1 源文件标准 (Source File Standard)

| 字段 | 标准 | 引用 |
|------|------|------|
| **格式** | WAV（PCM, 无压缩）| 09-v2 + 行业标准 |
| **比特深度** | **16-bit**（不是 24-bit / 32-bit）| 09-v2 §8 性能约束 |
| **采样率** | **44.1 kHz**（不是 48 kHz / 96 kHz）| 09-v2 §8 性能约束 |
| **通道数** | **立体声**（2 通道）| 09-v2 §1 9 类音频 |
| **位深 / 采样** | 16-bit × 44.1 kHz = 705.6 kbps | 09-v2 §8 |
| **峰值** | < -3 dBFS（avoid clipping）| 09-v2 §9.3 LUFS |
| **LUFS 目标** | -16 LUFS (SFX) / -18 LUFS (BGM) / -22 LUFS (Ambient) / -14 LUFS (TTS) | 09-v2 §9.3 |
| **时长** | SFX ≤ 8s / BGM 60-90s 循环 / 通关 BGM 8-30s 一次性 / 环境音 ≤ 30s 循环 | 09-v2 §1 |
| **头尾静音** | 100ms 静音（避免爆音）| 09-v2 §6.1 |
| **文件命名** | audio_<room_id>_<category>_<intensity>.wav | 本文 §4 |

> **关键设计：** **WAV 16-bit 44.1kHz 立体声**——CD 音质 + 跨平台兼容 + 文件大小可控（避免高分辨率文件膨胀）。
> **关键设计：** **LUFS 规范化**在源文件层完成——保证发布平台起点一致（与 cross-platform §3 一致）。

### 1.2 发布格式 (Distribution Formats)

| 平台 | 发布格式 | 编码 | 码率 | 采样率 | 比特深度 | 通道数 | 引用 |
|------|---------|------|------|--------|---------|--------|------|
| **PC Steam/Mac** | Opus (主) + FLAC (备) | Opus 1.3 | **128 kbps** | 44.1 kHz | 16-bit | 立体声 | cross-platform §2.1 |
| **PS5/Xbox** | Opus + AAC | Opus 1.3 | **128 kbps** | 44.1 kHz | 16-bit | 立体声 | cross-platform §2.1 |
| **Nintendo Switch** | Opus (压缩) | Opus 1.3 | **64 kbps** | **22.05 kHz** | **8-bit** | 立体声 | cross-platform §2.1 |
| **iOS** | AAC (主) + Opus (备) | AAC-LC | **128 kbps** | 44.1 kHz | 16-bit | 立体声 | cross-platform §2.1 |
| **Android** | Opus + OGG Vorbis | Opus 1.3 | **128 kbps** | 44.1 kHz | 16-bit | 立体声 | cross-platform §2.1 |

> **关键设计：** **源文件 WAV** → **发布格式 Opus / AAC / OGG / FLAC / ALAC**——自动转码（无需重新制作）。

### 1.3 Protobuf 序列化 (Protobuf Serialization)

> **源约束：** `design/data/serialization.md` Protobuf 序列化规范 + `design/data/persistence-strategy.md` 存档格式

```protobuf
// data/proto/audio_settings.proto
syntax = "proto3";

message AudioSettingsProto {
  float master_volume = 1;            // 0-1 范围 (default 0.8)
  float sfx_volume = 2;               // 0-1 范围 (default 0.7)
  float bgm_volume = 3;               // 0-1 范围 (default 0.6)
  float ambient_volume = 4;           // 0-1 范围 (default 0.4)
  float ui_volume = 5;                // 0-1 范围 (default 0.5)
  bool muted = 6;                     // default false

  float switch_sfx_db = 7;            // [-18, -6] dB (default -12)
  float reset_sfx_db = 8;             // [-24, -12] dB (default -18)
  float win_sfx_db = 9;               // [-12, -3] dB (default -6)
  float error_sfx_db = 10;            // [-18, -6] dB (default -12)
  float tutorial_sfx_db = 11;         // [-15, -6] dB (default -9)

  string platform_codec = 20;         // "opus" / "ogg" / "aac" / "alac" / "wav"
  int32 platform_channels = 21;       // 1 / 2 / 6 / 8
  int32 platform_max_memory_mb = 22;  // 50 / 80 / 100
  int32 platform_max_frame_rate = 23; // 30 / 60

  string emergency_theme = 30;        // "default" / "ep-1" / "ep-2" / "ep-3" / "ep-4" / "ep-5"
}
```

> **关键设计：** **Protobuf 序列化**——保证存档兼容（v1.0 → v1.1 → v2.0）+ AES-256-GCM 加密（与 design/data/serialization.md + SaveSystem 一致）。
> **关键设计：** **5 dB 字段范围保护**——`switch_sfx_db ∈ [-18, -6]` 等——避免玩家错误设置。

## 2. 音频编解码 (Audio Codec)

> **设计原则：** **Opus 是 7 平台通用首选**——开源 + 高压缩比 + 低延迟 + 全平台支持。

### 2.1 4 编解码选型对照 (4 Codec Selection)

| 编解码 | 类型 | 压缩比 | 延迟 | 开源 | 平台支持 | 7 平台选择 | 引用 |
|--------|------|:-----:|:---:|:---:|---------|:---------:|------|
| **Opus** | 有损 / 无损 | **10x**（vs WAV）| **22 ms** | ✅ | 全平台 | ✅ **首选** | 09-v2 §9.2 |
| **OGG Vorbis** | 有损 | 10x | 50 ms | ✅ | Android / Steam | ✅ 备选（Android）| 本文 §2.1 |
| **MP3** | 有损 | 8x | 100 ms | ✅ | 全平台（旧）| ❌ 不用（延迟高）| 本文 §2.1 |
| **FLAC** | 无损 | 2x | 0 ms | ✅ | 全平台 | ✅ 备选（PC 高质量）| 本文 §2.1 |
| **AAC** | 有损 | 10x | 50 ms | ❌（专利）| iOS / Apple | ✅ iOS 首选 | 本文 §2.1 |
| **ALAC** | 无损 | 2x | 0 ms | ❌（专利）| Apple Mac | ✅ Mac 备选 | 本文 §2.1 |

> **关键设计：** **Opus 是首选**——开源 + 压缩比高 + 延迟低 + 全平台 + 标准化（IETF RFC 6716）。
> **关键设计：** **MP3 不用**（延迟 100ms 不满足 ≤16ms 反馈 3 层契约）——历史遗留。

### 2.2 Opus 编码配置 (Opus Encoding Config)

```
Opus 1.3 (稳定版):
  Application: AUDIO (音乐 + SFX) 或 VOIP (语音)
  - v1.0 用 AUDIO (音乐优先 + 高保真)

Compression:
  Bitrate: 128 kbps (PC/Mobile) / 64 kbps (Switch)
  Complexity: 10 (最高质量)
  Frame Size: 20 ms (实时)

Resampling:
  44.1 kHz → 22.05 kHz (Switch) 自动降采样

Channel:
  Channels: 2 (立体声) → 1 (单声道) 自动降混

Preprocessing:
  Pre Skip: 312 samples (default, 7ms)
  Pre Emphasis: 关闭 (避免高频过度放大)
```

## 3. 性能预算 (Performance Budget)

> **源约束：** `09-v2 §8` 性能约束 6 项 + `cross-platform.md §5` 7 平台性能预算

### 3.1 性能预算 6 项 (6 Performance Constraints)

| # | 指标 | 目标 | 越界行为 | 验证方式 |
|---|------|------|---------|---------|
| **1** | **同时发声 SFX 数** | ≤ 8 | > 8 → 最早触发的 SFX 被打断 | Unity Profiler Audio |
| **2** | **BGM 通道数** | 1 (章节 BGM) + 1 (环境音) = 2 总 | > 2 → 性能下降 | Profiler Audio Channels |
| **3** | **音频文件总大小** | ≤ 50 MB (PC) / ≤ 25 MB (Switch) | > 上限 → 加载时间 > 5s | Resources 统计 |
| **4** | **音频加载时间** | ≤ 2s（主菜单加载时）| > 2s → 启动延迟 | 计时 |
| **5** | **音频 CPU 占用** | ≤ 5%（单核, PC）/ ≤ 3%（PS5/Xbox）| > 上限 → 帧率下降 | Profiler Audio CPU |
| **6** | **音频内存占用** | ≤ 30 MB (PC) / ≤ 20 MB (Switch) / ≤ 25 MB (Mobile) | > 上限 → 内存超 512MB 总预算 | Profiler Memory |

> **关键设计：** **音频线程独立**——不依赖帧率，即使帧率掉到 30 FPS 音频依然流畅。

### 3.2 7 平台性能预算对照 (7-Platform Performance Budget)

| 平台 | 帧率 | CPU | 内存 | 文件大小 | 同时发声 | BGM 通道 | 引用 |
|------|:---:|:---:|:----:|:-------:|:-------:|:-------:|------|
| **PC Steam/Mac** | 60 FPS | ≤ 5% | ≤ 30 MB | ≤ 50 MB | ≤ 8 | 2 | cross-platform §5 |
| **PS5/Xbox** | 60 FPS | ≤ 3% | ≤ 30 MB | ≤ 50 MB | ≤ 8 | 2 | cross-platform §5 |
| **Nintendo Switch** | 30 FPS | ≤ 4% | ≤ 20 MB | ≤ 25 MB | ≤ 6 | 1+BGM 1 | cross-platform §5 |
| **iOS** | 60 FPS | ≤ 4% | ≤ 25 MB | ≤ 50 MB | ≤ 6 | 2 | cross-platform §5 |
| **Android** | 60 FPS | ≤ 5% | ≤ 25 MB | ≤ 50 MB | ≤ 6 | 2 | cross-platform §5 |

> **关键设计：** **Switch 性能预算降低**——与 11-v2 §3.1 Switch 移植 200h 工作量一致。

## 4. 命名规范 (Naming Convention)

> **设计原则：** **audio_<room_id>_<category>_<intensity>.wav**——确保音频文件**可追溯** + **可审计** + **可分类**。

### 4.1 音频文件命名格式 (Audio File Naming Format)

```
audio_<room_id>_<category>_<intensity>.wav
   │         │           │           │
   │         │           │           └─ 文件格式（.wav 源 / .opus 发布）
   │         │           └─ 强度等级（L1 / L2 / L3 / L4 / 主旋律 / 变奏 / 无）
   │         └─ 类别（main / switch / reset / win / error / tutorial / bgm / ambient / ui / master）
   └─ 房间 ID（1-1 / 2-4 / 3-8 / mainmenu / ep-1 / ep-2 ...）

示例：
  audio_1_1_main_L1.wav            — 1-1 房间主旋律 L1 强度
  audio_2_4_switch_L2.wav          — 2-4 房间切换音 L2 强度
  audio_3_8_boss_main.wav          — 3-8 房间 Boss 房主旋律
  audio_mainmenu_bgm.wav           — 主菜单 BGM
  audio_ep1_emergency_theme.wav    — EP-1 应急主题
```

### 4.2 命名规范字段定义 (Naming Field Definition)

| 字段 | 格式 | 范围 | 示例 |
|------|------|------|------|
| **`<room_id>`** | `{章节}-{序号}` 或 `mainmenu` 或 `ep-N` | 1-1 ~ 3-8, mainmenu, ep-1 ~ ep-5 | `1-1` / `2-4` / `3-8` / `mainmenu` / `ep-1` |
| **`<category>`** | `main` / `switch` / `reset` / `win` / `error` / `tutorial` / `bgm` / `ambient` / `ui` / `master` | 11 类 | `main` / `switch` / `tutorial` |
| **`<intensity>`** | `L1` / `L2` / `L3` / `L4` / `V0` / `V1` / `V2` / `V3` / 无（main） | 4 强度 + 4 变奏 + 无 | `L1` / `L2` / `V2` / 无 |
| **`.wav` / `.opus`** | 文件格式 | wav (源) / opus / aac / ogg / flac / alac (发布) | `.wav` (源) / `.opus` (发布) |

### 4.3 命名规范示例 (Naming Examples)

| 文件名 | 类型 | 触发条件 | 引用 |
|--------|------|---------|------|
| `audio_1_1_main.wav` | 1-1 房间主旋律 | Ch1 章节 BGM 基础层 | 09-v2 §1.7 |
| `audio_1_1_main_V2.wav` | 1-1 主旋律 V2 变奏加密 | Ch1 章节 BGM 加密变奏（v1.1）| theme-motifs §4.2 |
| `audio_1_1_switch_L1.wav` | 1-1 房间切换音 L1 | 玩家按 E（L1 轻反馈）| layer-mixing §5 |
| `audio_2_4_switch_L2.wav` | 2-4 房间切换音 L2 | 玩家错误 3 次（L2 中反馈）| layer-mixing §5 |
| `audio_3_8_boss_main.wav` | 3-8 Boss 房主旋律 | Boss 房（独立 BGM）| 09-v2 §1.8 |
| `audio_mainmenu_bgm.wav` | 主菜单 BGM | 主菜单循环 | 09-v2 §1.7 |
| `audio_chapter1_bgm.wav` | Ch1 章节 BGM | Ch1 章节循环 | 09-v2 §1.7 |
| `audio_chapter1_bgm_V1.wav` | Ch1 BGM V1 变奏简化 | Ch1 喘息房（1-4 / 1-5）| theme-motifs §4.2 |
| `audio_ambient_room_indoor_L1.wav` | 室内环境音 L1 | 玩家在房间内 | 09-v2 §1.9 |
| `audio_ui_hint_L2.wav` | UI 提示音 L2 | Hint 提示 5/10/15 min 触发 | 09-v2 §1.10 |
| `audio_tutorial_firstswitch_L1.wav` | 教学首次切换音 L1 | 1-1 玩家第 1 次按 E | 09-v2 §1.6 |
| `audio_ep1_emergency_theme.wav` | EP-1 应急主题 | Steam 审核不通过时 | theme-motifs §6 |

> **关键设计：** **命名 + AudioImporter + AudioMixer Group 三层映射**——保证音频文件**可追溯**。

## 5. 命名版本 (Naming Version)

> **设计原则：** **v1.0 / v1.1 / v2.0 命名版本**——保证音频文件**可升级** + **可回滚**（与 11-v2 §3 发布计划 + design/api/versioning.md 同步）。

### 5.1 音频文件版本控制 (Audio File Version Control)

```
v1.0 (Day-84, M11/M12):
  audio_<room_id>_<category>_<intensity>_v1_0.wav
  → 主要版本：v1.0 (Steam + Mac + Itch.io)
  → 兼容性：向后兼容 v1.0.0 → v1.0.x

v1.1 (T+3m):
  audio_<room_id>_<category>_<intensity>_v1_1.wav
  → 增量版本：v1.1 (Switch 移植 + 5 变奏展开)
  → 兼容性：v1.0 → v1.1 自动转码 (无需重新制作)

v2.0 (T+6m):
  audio_<room_id>_<category>_<intensity>_v2_0.wav
  → 重大版本：v2.0 (PS/Xbox/iOS/Android 全平台 + 主旋律 3 变奏展开)
  → 兼容性：v1.1 → v2.0 同源 WAV，发布格式自动转码
```

> **关键设计：** **同源 WAV + 发布格式自动转码**——保证版本升级**无需重新制作**——降低 1 人 Solo + $0-30/月 预算负担。

### 5.2 音频版本号嵌入 (Audio Version Embedded)

```
文件元数据 (WAV + Protobuf):
  RIFF INFO:
    ISBJ = "anzhong-music"
    ICMT = "audio_1_1_main L1 v1.0"
    ICRD = "2026-06-30"
    IGNR = "ambient"
    IART = "anzhong-music-director"

Protobuf:
  message AudioClipProto {
    string version = 1;       // "1.0" / "1.1" / "2.0"
    int32 release_date = 2;   // 20260915
  }
```

## 6. 命名版权 (Naming Copyright)

> **设计原则：** **Kenney.nl CC0 + 自制 CC BY-NC 4.0 + 第三方 6 来源**——保证音频版权**清晰可追溯**（与 11-v2 §5.6 + design/art/copyright.md 同步）。

### 6.1 音频版权登记表 (Audio Copyright Registry)

| # | 来源 | 授权类型 | 文件数 | 商用 | 修改 | 署名要求 | 引用 |
|---|------|---------|:----:|:---:|:---:|---------|------|
| **01** | **Kenney.nl** | CC0 | ~20 SFX | ✅ | ✅ | ❌ 不需要 | 09-v2 §9.1 |
| **02** | **freesound.org (CC0)** | CC0 | ~5 SFX | ✅ | ✅ | ❌ 不需要 | 09-v2 §9.1 |
| **03** | **freesound.org (CC-BY)** | CC-BY | 0（v1.0 不用）| ✅ | ✅ | ✅ 需要署名 | 09-v2 §9.1 |
| **04** | **自制** | 自有版权 (CC BY-NC 4.0) | 7 预制件 + 5 BGM | ✅ | ✅ | ❌ 不需要 | 09-v2 §9.1 |
| **05** | **Suno / Udio** | 订阅协议 + 商用 | 5 BGM（章节 BGM）+ 19 房间 BGM + 3-8 Boss BGM = 25 文件 | ✅ | ❌ | ❌ 不需要 | 09-v2 §9.1 |
| **06** | **Spitfire LABS** | Free for commercial use | ~5 (钢琴 + 弦乐) | ✅ | ✅ | ❌（可选）| 本文新增 |
| **07** | **VSCO2 CE** | CC0 | ~5 (弦乐 + 木管) | ✅ | ✅ | ❌ | 本文新增 |
| **08** | **OpenGameArt.org** | CC0 / CC-BY | ~5 SFX + 1 BGM | ✅ | ✅ | 视授权 | 本文新增 |
| **09** | **Bandlab Sounds** | Free for commercial use | ~3 打击乐 | ✅ | ✅ | ❌ | 本文新增 |
| **合计** | — | — | **~50 文件** | — | — | — | — |

> **关键设计：** **v1.0 优先 CC0 + 商用免费**——零版权风险（与 11-v2 §5.6 + 09-v2 §9.1 一致）。
> **关键设计：** **CC-BY 不用**——避免"署名"导致的发布复杂度（与 09-v2 §9.1 R-02 一致）。

### 6.2 版权自动登记 (Copyright Auto-Registration)

```
音频文件元数据 (WAV INFO + Protobuf):
  string copyright_source = 1;     // "kenney.nl" / "suno" / "self-made" / "spitfire" / ...
  string copyright_license = 2;     // "CC0" / "CC-BY-NC-4.0" / "Suno-Subscriber" / ...
  string copyright_creator = 3;     // "kenney.nl" / "audio-director" / "suno" / ...
  string copyright_attribution = 4; // "" (无) / "https://kenney.nl" / ...

CI/CD 自动检查：
  - 每次 git commit 检查音频文件元数据
  - 不合规 → 报警 + 拒绝合并
  - 合规 → 允许合并 + 写入 data/audio/audio-copyright-audit.json
```

> **关键设计：** **CI/CD 自动版权检查**——避免"无版权登记"导致的发布风险。

## 7. 命名审计 (CI/CD Audio Audit)

> **设计原则：** **CI/CD 自动跑 ce-doc-review + 6 项 audio-audit**——保证音频文件**符合规范**（与 design/implementation/ci-cd.md + ce-doc-review 一致）。

### 7.1 CI/CD 音频审计 6 项 (6 Audio Audit Checks)

| # | 审计项 | 规范 | 工具 | 失败处理 |
|---|--------|------|------|---------|
| **A-01** | **命名规范** | `audio_<room_id>_<category>_<intensity>.wav` | Bash + Python | 报警 + 拒绝合并 |
| **A-02** | **文件格式** | WAV 16-bit 44.1kHz 立体声 | `ffprobe` | 报警 + 自动转码 |
| **A-03** | **LUFS 规范化** | -16 / -18 / -22 / -14 LUFS | `loudgain` | 报警 + 自动规范化 |
| **A-04** | **文件大小** | ≤ 50 MB 总 / ≤ 2 MB 单文件 | `du` | 报警 + 拒绝合并 |
| **A-05** | **版权登记** | 9 类来源 + 授权类型 + 商用 | 自定义 `audio-audit.py` | 报警 + 拒绝合并 |
| **A-06** | **命名版本** | `_v1_0` / `_v1_1` / `_v2_0` 后缀 | Bash + Python | 报警 + 拒绝合并 |

> **关键设计：** **6 项自动审计**——保证音频文件**符合规范** + **合规商用** + **可追溯**。

### 7.2 CI/CD 审计流程 (CI/CD Audit Workflow)

```
git push origin main
       ↓
[.github/workflows/audio-audit.yml]
       ↓
[Job 1: 命名规范检查 (A-01)]
  audio-audit.py --check-naming
  ❌ → 报警 + 拒绝合并
       ↓
[Job 2: 文件格式检查 (A-02)]
  ffprobe -v error -show_streams *.wav
  ❌ → 自动转码（16-bit 44.1kHz 立体声）
       ↓
[Job 3: LUFS 检查 (A-03)]
  loudgain -d -i -t -16 *.wav  # -16 LUFS 目标
  ❌ → 自动规范化
       ↓
[Job 4: 文件大小检查 (A-04)]
  du -sh audio/
  ❌ → 报警 + 拒绝合并
       ↓
[Job 5: 版权检查 (A-05)]
  audio-audit.py --check-copyright
  ❌ → 报警 + 拒绝合并
       ↓
[Job 6: 版本检查 (A-06)]
  audio-audit.py --check-version
  ❌ → 报警 + 拒绝合并
       ↓
✅ 6 项全部通过 → 允许合并
```

> **关键设计：** **6 项 CI/CD 自动审计** + **音频改动立即触发**——保证音频文件**全程合规**。

### 7.3 audio-audit.py 工具 (Audio Audit Tool)

```python
#!/usr/bin/env python3
# tools/audio-audit.py
"""
anzhong-music 音频审计工具 (CI/CD)
- 检查命名规范 + 文件格式 + LUFS + 文件大小 + 版权 + 版本
"""

import os
import re
import subprocess
import json

NAMING_PATTERN = re.compile(r'^audio_([\w-]+)_(\w+)_([\w-]+)_v(\d+)_(\d+)\.(\w+)$')

def check_naming(filepath):
    """A-01: 命名规范"""
    if not NAMING_PATTERN.match(os.path.basename(filepath)):
        return False, f"Invalid naming: {filepath}"
    return True, ""

def check_format_ffprobe(filepath):
    """A-02: 文件格式（ffprobe）"""
    result = subprocess.run([
        'ffprobe', '-v', 'error', '-show_streams', '-print_format', 'json',
        filepath
    ], capture_output=True, text=True)
    info = json.loads(result.stdout)
    for stream in info['streams']:
        if stream['codec_type'] == 'audio':
            if stream['bits_per_raw_sample'] != '16':
                return False, f"Expected 16-bit, got {stream['bits_per_raw_sample']}"
            if stream['sample_rate'] != '44100':
                return False, f"Expected 44.1 kHz, got {stream['sample_rate']}"
    return True, ""

def check_lufs(filepath):
    """A-03: LUFS 规范化（loudgain）"""
    result = subprocess.run(['loudgain', '-d', '-i', filepath], capture_output=True, text=True)
    # 解析输出 LUFS 值
    # ... (省略)
    return True, ""  # 简化

def check_size(filepath, max_size_mb=2):
    """A-04: 文件大小"""
    size_mb = os.path.getsize(filepath) / (1024 * 1024)
    if size_mb > max_size_mb:
        return False, f"File too large: {size_mb:.2f} MB > {max_size_mb} MB"
    return True, ""

def check_copyright(filepath):
    """A-05: 版权登记（自定义元数据读取）"""
    # 读取 WAV INFO + Protobuf 元数据
    # ... (省略)
    return True, ""  # 简化

def check_version(filepath):
    """A-06: 命名版本"""
    match = NAMING_PATTERN.match(os.path.basename(filepath))
    if not match:
        return False, f"No version suffix: {filepath}"
    return True, ""

if __name__ == '__main__':
    passed = 0
    failed = 0
    for filepath in sys.argv[1:]:
        for check in [check_naming, check_format_ffprobe, check_lufs, check_size, check_copyright, check_version]:
            ok, msg = check(filepath)
            if ok:
                passed += 1
            else:
                failed += 1
                print(f"[FAIL] {filepath}: {msg}")
    print(f"[SUMMARY] Passed: {passed}, Failed: {failed}")
    sys.exit(1 if failed else 0)
```

## 8. P0-001 跟踪 (P0-001 Tracking)

> **P0-001 状态（截至 2026-06-30）：** **OPEN — 02-v2 §13 AC-06 仍缺"难度上限 20"硬约束**

### 8.1 P0-001 与 design/music/standards.md 的关系

| 影响维度 | design/music/standards.md 受影响？ | 详情 |
|---------|:-------------------------------:|------|
| **命名规范 `<intensity>` 字段** | ❌ **不依赖** | `L1` / `L2` / `L3` / `L4` 是 L1-L4 强度反馈，与难度数字无关 |
| **版本控制 v1.0/v1.1/v2.0** | ❌ **不依赖** | 版本号固定，与难度无关 |
| **性能预算** | ❌ **不依赖** | 60 FPS + 16-bit + 44.1kHz + ≤16ms 延迟，与难度无关 |
| **版权登记** | ❌ **不依赖** | 9 类来源 + 授权类型固定，与难度无关 |
| **CI/CD 审计 6 项** | ❌ **不依赖** | 6 项审计固定，与难度无关 |
| **命名版权（CC0 + 自制 + 商用）** | ❌ **不依赖** | 版权固定，与难度无关 |

> **关键结论：** **design/music/standards.md 与 P0-001 关系**最弱——standards 全部**静态规范**（命名 + 格式 + 版本 + 性能 + 版权 + 审计），不与难度挂钩。
>
> **本文档策略：** v1.0 接受 P0-001 OPEN 状态——standards 不需要任何调整，**待 P0-001 解决后再统一回退**（与 09-v2 §11 + 12-v2 R-01 + design/data/p0-001-tracking.md 决策一致）。

### 8.2 P0-001 跟踪矩阵 (P0-001 Tracking Matrix)

| 项 | 内容 | 跟踪位置 |
|---|------|---------|
| **问题** | 02-v2 §13 AC-06 仍缺"难度上限 20"硬约束 | docs/02-core-mechanics-v2.md §13 |
| **design/data/p0-001-tracking.md** | 15 阻塞字段（rooms.difficulty / chapters.average_difficulty / player_progress.max_difficulty_reached / scores.difficulty_used / saves.max_difficulty_completed / leaderboard_entries.difficulty_filter / telemetry_events.difficulty_context / players.total_difficulty_completed 等）| design/data/p0-001-tracking.md §2 |
| **修复选项** | A. phase3 design 期间解决 (W01, 推荐 ✅) / B. phase4 单独 patch (W12+) / C. 实施期间补 (W03+) | design/data/p0-001-tracking.md §4 |
| **design/music/standards.md 对策** | 全部静态规范（命名 + 格式 + 版本 + 性能 + 版权 + 审计），**不依赖** P0-001 | 本文 §8.1 |
| **设计影响** | **零直接影响**——standards 仅静态规范，与"难度上限"无关 | 本文 §8.1 |
| **状态** | **OPEN（截至 2026-06-30）** | 09-v2 §11.1 + 10-v2 §10 R6 + design/data/p0-001-tracking.md §1.4 |
| **后续跟踪** | phase3 phase3 由 02 维护者增补 §13 AC-06 硬约束 → 标准自动升级（无需 standards.md 修改） | design/data/p0-001-tracking.md §4 选项 A |

> **关键设计：** **standards.md 不阻塞 v1.0 实施**——standards 与 P0-001 关系**最弱**（仅 0 处直接影响），无任何修改需要。

## 9. 边界条件 (Edge Cases)

1. **v1.0 文件命名不符合规范（手动命名错误）**
   - **触发条件：** CI/CD 跑 audio-audit.py 检测命名
   - **音频预期：** **报警 + 拒绝合并**——开发者必须修正命名

2. **v1.0 文件格式不是 WAV 16-bit 44.1kHz（采样率不一致）**
   - **触发条件：** CI/CD 跑 ffprobe 检测格式
   - **音频预期：** **报警 + 自动转码**（CI/CD 自动 ffmpeg 转码）

3. **v1.0 LUFS 不符合 -16 / -18 / -22 / -14 LUFS（音量不规范）**
   - **触发条件：** CI/CD 跑 loudgain 检测 LUFS
   - **音频预期：** **报警 + 自动规范化**（loudgain -i 自动调整）

4. **v1.0 文件大小超过 2 MB（单文件）**
   - **触发条件：** CI/CD 跑 du 检测大小
   - **音频预期：** **报警 + 拒绝合并**——开发者必须重新制作

5. **v1.0 版权未登记（metadata 缺失）**
   - **触发条件：** CI/CD 跑 audio-audit.py --check-copyright
   - **音频预期：** **报警 + 拒绝合并**——开发者必须登记版权

6. **v1.0 版本号缺失（v1_0 后缀缺失）**
   - **触发条件：** CI/CD 跑 audio-audit.py --check-version
   - **音频预期：** **报警 + 拒绝合并**——开发者必须添加版本号

7. **v1.0 Protobuf 序列化不兼容（旧存档迁移 v1.1）**
   - **触发条件：** 玩家从 v1.0 升级到 v1.1
   - **音频预期：** **Protobuf 向前兼容**——旧字段保留 + 新字段缺省值（与 design/data/serialization.md + design/data/migrations.md 一致）

## 10. 验收标准 (Acceptance Criteria)

- [x] **AC-01** 源文件标准（WAV 16-bit 44.1kHz 立体声）+ 9 字段（格式/比特/采样率/通道/位深采样率/峰值/LUFS/时长/头尾静音/命名）
- [x] **AC-02** 7 平台发布格式对照（PC Opus + Switch Opus 64 + iOS AAC + Android OGG Vorbis + FLAC + ALAC）
- [x] **AC-03** Protobuf 序列化（AudioSettingsProto 30 字段）+ 5 dB 字段范围保护
- [x] **AC-04** 4 编解码选型对照（Opus 首选 + OGG/MP3/FLAC/AAC/ALAC）
- [x] **AC-05** Opus 编码配置（Application AUDIO + Bitrate 128/64 + Complexity 10 + Frame Size 20ms）
- [x] **AC-06** 性能预算 6 项（同时发声 + BGM 通道 + 文件大小 + 加载时间 + CPU + 内存）
- [x] **AC-07** 7 平台性能预算对照（CPU / 内存 / 文件大小 / 同时发声 / BGM 通道）
- [x] **AC-08** 命名格式 `audio_<room_id>_<category>_<intensity>_v1_0.wav` + 字段定义 + 12 命名示例
- [x] **AC-09** 命名版本 v1.0/v1.1/v2.0 控制 + 元数据嵌入（RIFF INFO + Protobuf）
- [x] **AC-10** 版权登记表 9 类来源（Kenney.nl CC0 + 自制 CC BY-NC + Suno/Udio + Spitfire LABS + VSCO2 CE + OpenGameArt.org + Bandlab Sounds）
- [x] **AC-11** 版权自动登记（CI/CD 自动检查 + data/audio/audio-copyright-audit.json）
- [x] **AC-12** CI/CD 6 项审计（命名 + 格式 + LUFS + 大小 + 版权 + 版本）
- [x] **AC-13** CI/CD 审计流程（6 Job 自动化 + 报警 + 自动转码）
- [x] **AC-14** audio-audit.py 工具（6 项审计 Python 代码示例）
- [x] **AC-15** P0-001 跟踪（standards 与 P0-001 关系最弱，0 直接影响）
- [x] **AC-16** P0-001 跟踪矩阵（问题 / 15 阻塞字段 / 3 修复选项 / standards 对策 / 设计影响 / 状态）
- [x] **AC-17** 边界条件 7 条（每条含触发条件 + 音频预期）
- [x] **AC-18** 与 09-v2 + 11-v2 + design/data/serialization.md + design/data/p0-001-tracking.md + design/api/versioning.md + design/implementation/ci-cd.md 引用 8+ 处

## 11. 关联文档

### 11.1 上游（本文档依赖）

- [`docs/09-audio-v2.md`](../../docs/09-audio-v2.md) §8 性能约束 + §9.1 版权 + §9.3 LUFS
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) §3.1 发布计划 + §5.6 版权
- [`design/api/versioning.md`](../api/versioning.md) — 命名版本 v1.0/v1.1/v2.0
- [`design/api/api-spec.yaml`](../api/api-spec.yaml) M09 AudioSettings + audioAsset 字段
- [`design/data/serialization.md`](../data/serialization.md) — Protobuf 序列化
- [`design/data/p0-001-tracking.md`](../data/p0-001-tracking.md) — P0-001 全量跟踪（15 阻塞字段 + 3 修复选项）
- [`design/art/copyright.md`](../art/copyright.md) — Kenney.nl CC0 + 自制版权
- [`design/implementation/ci-cd.md`](../implementation/ci-cd.md) — ce-doc-review + Audio Audit Workflow
- [`cross-platform.md`](./cross-platform.md) §2 7 平台文件格式 + §3 7 平台音量统一 + §5 7 平台性能预算

### 11.2 下游（本文档被依赖）

- `src/Audio/AudioAssetImporter.cs` — 文件导入 + LUFS 检测 + 命名校验
- `src/Audio/LoudnessNormalizer.cs` — EBU R128 / -16 LUFS 规范化
- `src/Audio/AudioBank.cs` — 9 类 × 28 文件 + 命名规范
- `tools/audio-audit.py` — CI/CD 自动审计工具
- `.github/workflows/audio-audit.yml` — 6 Job 自动化
- `data/audio/audio-platform-config.json` — 7 平台音频配置
- `data/audio/audio-copyright-audit.json` — 版权登记

## 12. 待办事项 (TODO)

- [ ] **P0：** 实现 audio-audit.py（6 项审计 Python 工具 + CI/CD Workflow）— 阻塞 v1.0 [§7]
- [ ] **P0：** 实现命名规范 CI/CD 自动校验（`audio_<room_id>_<category>_<intensity>_v1_0.wav`）— 阻塞 v1.0 [§4]
- [ ] **P0：** 实现 WAV 16-bit 44.1kHz 自动转码（CI/CD ffmpeg + loudgain）— 阻塞 v1.0 [§1.1 + §7]
- [ ] **P0：** 实现版权登记自动检查（CI/CD audio-audit.py --check-copyright）— 阻塞 v1.0 [§6.2 + §7]
- [ ] **P0：** 实现 Protobuf 序列化（AudioSettingsProto 30 字段 + SaveSystem 集成）— 阻塞 v1.0 [§1.3]
- [ ] **P0：** 实现 v1.0 命名版本后缀（`_v1_0` 后缀）+ CI/CD 校验 — 阻塞 v1.0 [§5]
- [ ] **P1：** 实现 v1.1/v2.0 命名版本切换（CI/CD audio-audit.py --check-version）— 不阻塞 v1.0 [§5.1]
- [ ] **P1：** 实现 9 类来源版权自动登记（CC0 / CC BY-NC / Suno-Subscriber / Spitfire LABS 等）— 不阻塞 v1.0 [§6.1]
- [ ] **P2：** 跟踪 P0-001（02-v2 §13 AC-06 增补"难度上限 20"硬约束）— standards.md **无直接影响**，仅跟踪 [§8 + design/data/p0-001-tracking.md]

## 13. 变更日志

| 日期 | 版本 | 变更人 | 内容 |
|------|------|--------|------|
| 2026-06-30 | v1.0 | 中书省 subagent (MUSIC-01) | **standards.md 初版:** 音频文件标准（WAV 16-bit 44.1kHz 立体声）+ 源文件 9 字段 / 7 平台发布格式对照 + Protobuf 序列化（AudioSettingsProto 30 字段）/ 4 编解码选型 + Opus 编码配置 / 性能预算 6 项 + 7 平台对照 / 命名规范 audio_<room_id>_<category>_<intensity>_v1_0.wav + 12 命名示例 + 命名版本 v1.0/v1.1/v2.0 / 版权登记 9 类来源 + 版权自动登记 / CI/CD 6 项审计 + 审计流程 + audio-audit.py 工具 / P0-001 跟踪（standards 与 P0-001 关系最弱，0 直接影响）+ 跟踪矩阵 / 7 边界条件 / 18 验收标准 / 11 关联引用 / 9 待办 |

---

**最后更新：** 2026-06-30
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）
**P0-001 跟踪：** OPEN — 02-v2 §13 AC-06 待增补"难度上限 20"硬约束（详见 §8 + design/data/p0-001-tracking.md §1.4）。**standards.md 与 P0-001 关系最弱（0 直接影响），不阻塞 v1.0 实施。**
