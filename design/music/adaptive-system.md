---
title: 《暗室》自适应系统 (Adaptive System)
doc_id: DES-anzhong-music-adaptive-system
parent: design/music/README.md
last_updated: 2026-06-30
version: v1.0
status: draft
owner: 音乐总监（中书省 subagent · MUSIC-01）
---

# 《暗室》自适应系统（design/music/adaptive-system.md）

> **一句话定位：** 玩家状态 6 档 + 房间主题自适应 + 心流曲线 7 阶段 + 教学曲线 9 触发 + 反馈 3 层同步 + 7 平台自适应 + 难度自适应（P0-001 跟踪）。

## 目的 (Purpose)

本文档是《暗室》音乐层**自适应系统**的完整定义。它向：

- **音乐总监 / 音频工程师** — 讲清**玩家状态 6 档自适应** + **房间主题 19 间自适应** + **心流 7 阶段自适应**
- **关卡策划** — **教学曲线 9 触发**音频参数 + **反馈 3 层同步**契约 + **难度自适应**（P0-001 跟踪）
- **Unity 工程师** — `AdaptiveAudioDirector.cs` 模块的 7 触发契约
- **测试玩家** — 验证自适应**实时响应**（不卡顿 / 不爆音 / 不延迟）
- **未来外包商** — **7 平台自适应**差异（PC / Switch / 移动 / 主机）

**本文档范围：** **自适应（Adaptive）**——根据玩家状态 / 房间进度 / 教学阶段 / 平台规格**动态调整音频参数**。

## 1. 玩家状态触发 (Player-State Triggered Adaptation)

> **核心设计原则：** **焦虑-无聊平衡 6 档**——基于玩家**错误次数 + 停留时长 + 心流阶段**动态调整音频参数（与 06-v2 §11.2 焦虑-无聊 + 09-v2 §2.3 难度递增一致）。

### 1.1 焦虑-无聊平衡 6 档 (Anxiety-Boredom 6 Tiers)

| 档位 | 玩家状态 | 触发条件 | L1 主旋律 | L3 节奏 | L5 效果层 | 与 Pillar 关系 |
|------|---------|---------|:--------:|:------:|:------:|--------------|
| **P1 极焦虑** | 玩家多次错误 + 长时间停留 | 错误 ≥ 5 + 停留 15+ min | L4 兜底 0 dB | +18 dB | 静音 → -12 dB 风铃 | P2 puzzle 突变 |
| **P2 焦虑** | 玩家多次错误 | 错误 ≥ 3 | L3 重反馈 -6 dB | -6 dB | 错音 ×2 -12 dB | P2 puzzle 不和谐 |
| **P3 微焦虑** | 玩家 1-2 次错误 | 错误 1-2 | L2 中反馈 -12 dB | -12 dB | 错音 -12 dB | P2 puzzle 短动机 |
| **P4 心流** | 玩家在挑战与技能匹配 | 错误 0 + 房间停留 3-15 min | -6 dB | -12 dB | -12 dB 切换音 | P1+P4 沉浸 + 一致 |
| **P5 微无聊** | 玩家快速通关 | 错误 0 + 房间停留 < 3 min | -6 dB | **-12 dB（无变化）**| -12 dB 切换音 | P4 一致性 |
| **P6 无聊** | 玩家长时间无操作 | 错误 0 + 停留 15+ min | **-9 dB + 节拍变化**| **-15 dB（轻降）**| -12 dB 切换音 | P4 一致性 + 微变化 |

> **关键设计：** 6 档**音量呈 U 型曲线**——P1/P6（极端）+ P3/P5（弱）+ P4（心流，正常）。
> **关键设计：** P1 极焦虑**用 L4 兜底**——而非"突然大声"——传达"游戏不惩罚你"（与 07-v2 §7 L4 + 09-v2 §5.1 一致）。

### 1.2 6 档自适应触发数据 (6-Tier Trigger Data)

```
玩家状态数据源：
1. PlayerPrefs.errorCount (int)         — 玩家错误次数
2. TimeInRoom (float, seconds)          — 玩家房间停留时长
3. PlayerPrefs.currentRoomId (string)  — 当前房间
4. PlayerPrefs.lastResetTime (long)     — 上次重置时间戳
5. AdaptiveDirector.FlowPhase (enum)    — 心流阶段 (S1-S7)

自动适配参数（每 5 秒检测一次）：
- 错误次数 ≥ 5 + 停留 ≥ 15 min → P1 极焦虑（L4 兜底）
- 错误次数 ≥ 3 + 停留 < 15 min → P2 焦虑（L3 重反馈）
- 错误次数 1-2 → P3 微焦虑（L2 中反馈）
- 错误次数 0 + 停留 3-15 min  → P4 心流（正常）
- 错误次数 0 + 停留 < 3 min   → P5 微无聊（章节 BGM 持续）
- 错误次数 0 + 停留 ≥ 15 min  → P6 无聊（章节 BGM + 节拍变化）
```

> **实现：** `src/Audio/AdaptiveAudioDirector.cs` 5 秒检测一次，根据 5 个数据源综合判定 P1-P6 档位。

### 1.3 6 档自适应播放器实现 (6-Tier AudioMixer Snapshot)

```csharp
// src/Audio/AdaptiveAudioDirector.cs

public enum PlayerAudioState { ExtremeAnxiety, Anxiety, MildAnxiety, Flow, MildBoredom, Boredom }

public class AdaptiveAudioDirector : MonoBehaviour
{
    public AudioMixerSnapshot[] _6snapshots = new AudioMixerSnapshot[6];
    // 6 snapshot 配置：
    //   P1: L1=0dB L3=+18dB L4=-25dB L5=静音
    //   P2: L3 重反馈 = L1=-6dB L3=-6dB L5=-12dB x2
    //   P3: L2 中反馈 = L1=-12dB L3=-12dB L5=-12dB
    //   P4: 心流 = L1=-6dB L3=-12dB L5=-12dB
    //   P5: 微无聊 = L1=-6dB L3=-12dB L5=-12dB
    //   P6: 无聊 = L1=-9dB L3=-15dB L5=-12dB + 节拍变化

    public void TransitionTo(PlayerAudioState state)
    {
        _6snapshots[(int)state].TransitionTo(0.5f);  // 0.5s snapshot transition
    }
}
```

## 2. 房间主题自适应 (Room-Theme Adaptive)

> **设计原则：** **19 房间 1 主题 + 切换时渐变 + 通关时主旋律回归**——保持主题稳定 + 切换平滑（与 09-v2 §3.2 + theme-motifs §2 一致）。

### 2.1 房间主题自适应规则 (Room Theme Adaptive Rules)

| 触发点 | 音频动作 | 时长 | 引用 |
|-------|---------|------|------|
| **玩家进入房间** | 当前房间主题 BGM fade-in（若与章节 BGM 不同）| 2s | 09-v2 §2.4 |
| **房间通关 → 加载下间** | 章节 BGM 持续，无主题渐变 | 0s | 09-v2 §2.4 |
| **章节通关 → 章节标题画面** | 章节主题 BGM 持续 + 主旋律回归 | 0s（叠播）| theme-motifs §5.1 |
| **通关 (3-8)** | 章节 BGM 渐出 + 通关 BGM 渐入 + 主旋律完整回归 | 3s cross-fade | 09-v2 §2.2 |
| **玩家重玩已通关房** | 章节 BGM 持续，无主题回归 | 0s | theme-motifs §5.2 |

### 2.2 房间主题与 5 段落音色对应 (Room Theme × 5 Sections)

| 5 段落 | 对应房间 | 房间主题自适应策略 |
|--------|---------|------------------|
| **序章**（主菜单）| 主菜单 | 0 个房间，单独 BGM |
| **第一章** (Ch1) | 1-1 ~ 1-5 | 复用 `bgm_chapter1_01` + 变奏（5 主旋律变奏点）|
| **间章**（章节过渡）| 1-5 → 2-1 之间 | 间章 BGM（8s 一次性）+ 章节提示音 |
| **第二章** (Ch2) | 2-1 ~ 2-6 | 复用 `bgm_chapter2_01` + 变奏（6 主旋律变奏点）|
| **第三章** (Ch3) | 3-1 ~ 3-7 | 复用 `bgm_chapter3_01` + 变奏（7 主旋律变奏点）|
| **第三章** (Boss 房 3-8) | 3-8 | **Boss 房独立 BGM**（基于 M1 + M3 + 不和谐）|

> **关键设计：** v1.0 仅 Boss 房 (3-8) 有**专属房间主题**——其他 18 房间复用章节 BGM + 变奏（不需新音频文件，与 09-v2 §1.8 一致）。

### 2.3 通关时主旋律回归 (Main Theme Recurrence)

> **详见 theme-motifs.md §5** —— 通关时主旋律回归是**自适应系统的最高潮**——传达"我从转换走到觉醒"的旅程。

| 触发点 | 主旋律回归动作 | 音量 dB | 引用 |
|-------|--------------|---------|------|
| **通关 (3-8)** | 主旋律完整回归 + 弦乐合奏 + 钟声 | -3（最高）| theme-motifs §5.1 |
| **章节通关 (1-5 / 2-6)** | 主旋律简化回归 + 章节提示音叠加 | -3 | theme-motifs §5.1 |
| **房间通关（普通房）** | 主旋律片段（2 小节）+ 通关音 | -6 | theme-motifs §5.1 |

## 3. 心流曲线 (Flow Curve)

> **设计原则：** 心流（Flow）= Csikszentmihalyi 模型——**挑战与技能匹配**——7 阶段音频对应**与玩家状态匹配**（与 06-v2 §6 + layer-mixing §6 一致）。

### 3.1 心流 7 阶段 (Flow 7 Phases)

| 阶段 | 名称 | 玩家状态 | 音频对应 | 与 Pillar 关系 |
|------|------|---------|---------|--------------|
| **S1** | **起始** | 玩家进入游戏 | 主菜单 BGM 渐入 | P1 沉浸感 |
| **S2** | **探索** | 玩家探索新房间 | 章节 BGM 基础层 | P1+P4 沉浸 + 一致 |
| **S3** | **挑战** | 玩家进入难房 | 章节 BGM + 节奏层 | P4 一致性 |
| **S4** | **挫败** | 玩家多次错误 | L4 强度反馈触发 | P2 puzzle 突变 |
| **S5** | **突破** | 玩家即将通关 | 章节 BGM 加密 + 短促通关音预告 | P2+P4 |
| **S6** | **掌控** | 玩家通关房间 | 主旋律回归（片段）| P1+P4 |
| **S7** | **完成** | 玩家通关全章 / 通关游戏 | 主旋律完整回归 + 全乐器 ff + 钟声 | P4 主题循环 |

> **关键设计：** 7 阶段**音量逐级上升**（S1 -6dB → S7 0dB）——强化"我从挑战走到掌控"。

### 3.2 心流 7 阶段音频参数表 (Flow 7 Phase Audio)

| 阶段 | L1 主旋律 | L2 伴奏 | L3 节奏 | L4 氛围 | L5 效果 | 风格层 |
|------|:--------:|:------:|:------:|:------:|:------:|--------|
| **S1** | -6 dB | -12 dB | -18 dB | -18 dB | -12 dB | L1 dark ambient |
| **S2** | -6 dB | -12 dB | -18 dB | -18 dB | -12 dB | L1+L3 minimal |
| **S3** | -6 dB | -12 dB | -12 dB | -18 dB | -12 dB | L3 节奏 |
| **S4** | -12 dB | -12 dB | -12 dB | -15 dB | -12 dB | L4 强度反馈 |
| **S5** | -3 dB | -9 dB | -6 dB | -18 dB | -6 dB | L3 加密 + L5 短促 |
| **S6** | -3 dB | -9 dB | -12 dB | -18 dB | -3 dB | L1 主旋律回归 |
| **S7** | 0 dB | -3 dB | -3 dB | -18 dB | -3 dB | L1+L2+L3 全开 |

> **关键设计：** S4 挫败**用 L4 强度反馈触发**——而非"突然大声"——传达"游戏不惩罚你"（与 07-v2 §7 L4 + style.md §5 一致）。

### 3.3 心流阶段判定算法 (Flow Phase Detection Algorithm)

```
判定流程（每 5 秒检测一次）：
1. 读取 PlayerPrefs.errorCount + TimeInRoom + currentRoomId
2. 计算玩家「挑战-技能匹配度」(Challenge-Skill Fit)
   = 1.0 - (errorCount / 10) - (|TimeInRoom - 5min| / 30min)
3. 根据匹配度判定心流阶段：
   - 匹配度 0.0-0.2  → S1 起始（玩家刚进入）
   - 匹配度 0.2-0.5 + 错误 = 0  → S2 探索
   - 匹配度 0.5-0.8 + 错误 0-2  → S3 挑战
   - 匹配度 0.0-0.2 + 错误 ≥ 5 → S4 挫败
   - 匹配度 0.8-0.9 + 错误 0   → S5 突破
   - 玩家通关房间瞬间         → S6 掌控
   - 玩家通关章节瞬间         → S7 完成
4. 触发 AudioMixer Snapshot 渐变（0.5s transition）
```

## 4. 教学曲线音频 (Tutorial Curve Audio)

> **设计原则：** **9 教学触发**——1-1 ~ 1-5 教学曲线 + 2-4 / 3-3 / 3-5 新预制件首次出现（与 04-v2 §5 + 09-v2 §6 同步）。

### 4.1 9 教学触发清单 (9 Tutorial Triggers)

| 房间 | 教学阶段 | 教学音 ID | 音量 dB | 触发条件 | 体验意图 | 引用 |
|------|---------|----------|--------|---------|---------|------|
| **1-1** | T1 零文字 | `sfx_tutorial_firstswitch_01` | -9 | 玩家第 1 次按 E | "我做了什么" | 09-v2 §6.1 |
| **1-1** | T1 零文字 | `sfx_win_room_01` | -6 | 玩家走到出口 | "我完成了" | 09-v2 §6.1 |
| **1-2** | T2 渐显 | `sfx_win_room_01` | -6 | 通关 | 巩固切换音认知 | 09-v2 §6.1 |
| **1-3** | T2 渐显 | `sfx_switch_cycle_01` | -12 | 玩家首次遇 CycleSlot | "我在循环" | 09-v2 §6.1 |
| **1-4** | T3 R 键教学 | `sfx_tutorial_firstreset_01` | -15 | 玩家第 1 次按 R | "我可以重来" | 09-v2 §6.1 |
| **1-5** | T4 章节完成 | `sfx_win_chapter_01` | -3 | 章节末间通关 | "章节完成" | 09-v2 §6.1 |
| **2-4** | T4 章节完成 | `sfx_tutorial_door_01` | -12 | 玩家首次遇 Door | "新预制件" | 09-v2 §6.1 |
| **3-3** | T4 章节完成 | `sfx_tutorial_fakefloor_01` | -12 | 玩家首次遇 FakeFloor | "新视觉欺骗" | 09-v2 §6.1 |
| **3-5** | T4 章节完成 | `sfx_tutorial_crumble_01` | -12 | 玩家首次遇 CrumblingFloor | "新预制件" | 09-v2 §6.1 |

> **关键设计：** 教学音**仅触发 1 次 / 房间**（用 PlayerPrefs 记录 `tutorial.firstSwitchSeen` 等标志）——避免"教学疲劳"。
> **关键设计：** 1-1 第 1 次切换音 -9dB（**比常规切换音 -12dB 高 3dB**）——强化"新事件"。

### 4.2 教学曲线音频情绪变化 (Tutorial Audio Emotional Arc)

```
1-1 (T1 零文字)  →  1-2 (T2 渐显)  →  1-3 (T2 渐显)  →  1-4 (T3 R 键)  →  1-5 (T4 章节)
音量: -9dB 教学   →   -12dB 常规   →   -12dB 常规   →   -15dB 重置   →   -3dB 章节
情绪: 好奇 / 引入  →  平静 / 思考   →  平静 / 方向感  →  喘息 / 重置   →  明亮 / 完成
```

### 4.3 教学音频自适应规则 (Tutorial Adaptive Rules)

```
玩家首次进入 1-1:
  PlayerPrefs.tutorial.firstSwitchSeen = false
  → 玩家首次按 E 时触发 sfx_tutorial_firstswitch_01 (-9dB)
  → PlayerPrefs.tutorial.firstSwitchSeen = true

玩家重玩 1-1 已通关房:
  PlayerPrefs.tutorial.firstSwitchSeen = true
  → 不再触发 sfx_tutorial_firstswitch_01
  → 用 sfx_switch_toggle_01 (-12dB) 替代
```

> **实现：** `src/Audio/TutorialAudioPlayer.cs` 读取 9 个 PlayerPrefs 标志，触发对应教学音。

## 5. 反馈 3 层同步 (Tri-Layer Feedback Synchronization)

> **核心原则：** **视觉 200ms / 音频 -12dB / 触觉 ≤ 16ms** —— 三通道同步（与 08-v2 §7.3 + 09-v2 §2.5 一致）。

### 5.1 反馈 3 层同步契约 (Tri-Layer Sync Contract)

| 触发条件 | L5 效果层（音频）| 视觉层（屏幕）| 触觉层（手柄）| 同步 | 引用 |
|---------|----------|----------|----------|------|------|
| **玩家按 E (切换)** | attack ≤ 50ms / decay ≤ 200ms | 槽位发光 + 旧预制件淡出 (200ms) | 手柄震动 16ms | t=0ms | 08-v2 §7.2 |
| **玩家按 R (重置)** | attack ≤ 50ms / decay ≤ 200ms | 所有槽位淡出淡入 (200ms) | 手柄震动 16ms | t=0ms | 08-v2 §7.2 |
| **玩家按 Q (反向切换)** | attack ≤ 50ms / decay ≤ 200ms | 槽位发光 + 旧预制件淡入 (200ms) | 手柄震动 16ms | t=0ms | 08-v2 §7.2 |
| **房间通关** | 通关音 -6dB 0.5s | 出口脉冲 + 屏幕渐白 (200ms) | 手柄震动 16ms | t=0ms | 08-v2 §7.2 |
| **章节通关** | 通关音 -3dB 1s | 屏幕中央播放通关文字 (3s) | 手柄震动 16ms | t=0ms | 08-v2 §7.2 |

> **关键设计：** 3 层**完全同步**——视觉 200ms / 触觉 ≤16ms / 音频 attack ≤50ms + decay ≤200ms（与 08-v2 §7.3 反馈 3 层混音时序契约一致）。

### 5.2 反馈 3 层时序图 (Tri-Layer Timeline)

```
t=0ms     触觉触发（16ms 内）+ 视觉触发（200ms 内）+ 音频 attack（≤ 50ms）
            ↓
t=50ms    音频 attack 完成（避免爆音）
            ↓
t=200ms   视觉动画完成 + 音频 decay 完成 + 状态回归 Hover
            ↓
t=300ms   E/Q 冷却结束，可再次触发
```

> **关键设计：** 反馈 3 层**完全同步**——避免"按了没反应"（与 08-v2 §6 一致）。

### 5.3 听障 / 视障 自适应 (Accessibility Adaptive)

```
听障玩家（audio.captionsEnabled = true）：
  L5 效果层 → 字幕同步（[切换] / [重置] / [通关] / [错误：路径封闭]）
  视觉层 → 反馈强度 × 1.5（发光增强）

视障玩家（audio.enhancedAudio = true）：
  L5 效果层 → 反馈强度 × 1.2（音量 +2dB）
  L4 氛围层 → 距槽位电流声周期更短（增强方向感）
  TTS → 章节名 / 房间名 / 提示音

色盲 + 听障双重障碍：
  视觉层 → 强度 × 1.5（发光增强）
  L5 效果层 → 强度 × 1.2（音量 +2dB）
  字幕 → 强制开启（不能关闭）
  触觉层 → 强度 × 1.5（震动时长 × 1.5）
```

> **关键设计：** 字幕在双重障碍下**强制开启**——避免"看不见 + 听不到 + 无字幕"的信息真空（与 09-v2 §7.4 + 06-v2 §10 一致）。

## 6. 7 平台自适应 (7-Platform Adaptive)

> **设计原则：** 7 平台音频规格**统一主旋律**（不变），仅**格式 / 编码 / 通道数 / 性能预算**自适应（与 cross-platform.md + 09-v2 + 11-v2 一致）。

### 6.1 7 平台自适应对照表 (7-Platform Adaptive)

| 平台 | L1+L2 编码 | L4 通道 | 空间音频 | 性能预算 | BGM 通道 | 优先级 |
|------|:---------:|:------:|---------|---------|:-------:|:----:|
| **PC Steam** | Opus (主) + WAV | 立体声 | Dolby Atmos 5.1/7.1 | 60 FPS / 100 MB 总音频 | 2 | **P0** |
| **PC Mac** | Opus + ALAC | 立体声 | Dolby Atmos | 60 FPS / 100 MB | 2 | **P0** |
| **PS5** | Opus + AAC | 立体声 + Tempest 3D | Sony 360 Reality Audio | 60 FPS / 100 MB | 2 | **P2** |
| **Xbox Series X\|S** | Opus + AAC | 立体声 + Project Acoustics | Dolby Atmos | 60 FPS / 100 MB | 2 | **P2** |
| **Nintendo Switch** | Opus 64 kbps | 立体声（无空间）| 立体声 | 30 FPS / 50 MB | 1（BGM 1 + Ambient 1 = 2 总通道）| **P1** |
| **iOS** | AAC + Opus | 立体声 | Dolby Atmos (iPhone 12+) | 60 FPS / 80 MB | 2 | **P2** |
| **Android** | Opus (主) + OGG Vorbis | 立体声 | Dolby Atmos (Android 13+) | 60 FPS / 80 MB | 2 | **P2** |

> **关键设计：** 7 平台**主旋律统一**（v1.0 制作 1 套主旋律，平台发布时**自动转码**——无需重新制作）。
> **关键设计：** Switch 由于带宽限制**双声道压缩 + 50 MB**——其他平台 100 MB / 80 MB。

### 6.2 7 平台音频自适应 PlayerPrefs (7-Platform Audio Prefs)

```
PlayerPrefs.audio.platformCodec = "opus" | "ogg" | "aac" | "alac" | "wav"
PlayerPrefs.audio.platformChannels = 1 | 2 | 6 | 8
PlayerPrefs.audio.platformMaxMemoryMB = 50 | 80 | 100
PlayerPrefs.audio.platformMaxFrameRate = 30 | 60
```

> **实现：** `src/Audio/PlatformAudioAdapter.cs` 启动时读取平台 + PlayerPrefs，配置 5 层级混音 AudioMixer 参数。

## 7. 难度自适应 (Difficulty Adaptive)

> **⚠️ P0-001 跟踪：** 02-v2 §13 AC-06 仍缺"难度上限 20"硬约束。本节依赖该约束——**待 P0-001 解决后生效**。

### 7.1 难度自适应规则 (Difficulty Adaptive Rules)

| 房间难度 | L3 节奏层密度 | L1 主旋律 dB | 体验意图 | 引用 |
|---------|:-----------:|:-----------:|---------|------|
| **1-5（难度 2-5）**| **0%**（仅章节 BGM 基础层）| -6 dB | Ch1 教学 — 平静 / 思考 | 09-v2 §2.3 |
| **2-1 ~ 2-6（难度 7-10）**| **50%**（L3 节奏层进入，密度 50%）| -6 dB | Ch2 综合 — 紧张 / 复合 | 09-v2 §2.3 |
| **3-1 ~ 3-7（难度 11-16）**| **100%**（L3 节奏层全密度）| -6 dB | Ch3 挑战 — 节奏 / 紧迫 | 09-v2 §2.3 |
| **3-8（难度 16，Boss 房）**| **100% + Boss 房专属 BGM** | **-3 dB（最强）** | 终极挑战 | 09-v2 §2.3 + §1.8 |

### 7.2 难度自适应上限保护 (Difficulty Cap Protection) — **P0-001 OPEN**

> **当前状态：** 02-v2 §13 AC-06 仍缺"难度上限 20"硬约束。**v1.0 接受 P0-001 OPEN 状态**——按当前 02-v2 实现，**待 P0-001 解决后再实现上限保护**。

**未来实现（待 P0-001 解决后）：**

```csharp
// src/Audio/AdaptiveAudioDirector.cs
public const int DIFFICULTY_MAX = 20;  // 难度上限 20（待 P0-001 解决后启用）

public float GetMaxLayerDensity(int difficulty)
{
    // 难度自适应上限保护（待 P0-001 解决后）
    if (difficulty > DIFFICULTY_MAX)
    {
        Debug.LogWarning(
            $"[P0-001 OPEN] 房间难度 {difficulty} > 上限 {DIFFICULTY_MAX}, 强制回退至 {DIFFICULTY_MAX}");
        difficulty = DIFFICULTY_MAX;
    }
    return difficulty / 20.0f;  // [0, 1]
}
```

> **关键设计：** **不在 v1.0 实现上限保护**——仅跟踪 P0-001，等 02 增补"难度上限 20"硬约束后启用。

### 7.3 难度自适应与 P0-001 跟踪 (Difficulty Adaptive × P0-001)

| 项 | 内容 | 跟踪位置 |
|---|------|---------|
| **问题** | 02-v2 §13 AC-06 仍缺"难度上限 20"硬约束 | docs/02-core-mechanics-v2.md §13 |
| **design/music/ 影响** | adaptive-system §7.1 难度自适应规则表 4 档（1-5 / 2-x / 3-x / 3-8）依赖该约束 | 本文 §7.1 + 09-v2 §2.3 + 11 P0-001 跟踪 |
| **本文对策** | v1.0 接受 P0-001 OPEN，按当前 02-v2 实现，**待 P0-001 解决后再实现上限保护** | 本节 + design/data/p0-001-tracking.md §1 |
| **解决路径** | phase3 由 02 维护者增补 §13 AC-06 "难度上限 20"硬约束 | 10-v2 R6 W01 必解决 |
| **状态** | **OPEN（截至 2026-06-30）** | 09-v2 §11.1 + 10-v2 §10 R6 + design/data/p0-001-tracking.md §1.4 |

## 8. 边界条件 (Edge Cases)

1. **玩家快速切换 P1-P6 档位（每 5 秒变化）**
   - **触发条件：** 玩家快速切档（每 5 秒）导致 AudioMixer Snapshot 频繁切换
   - **音频预期：** Snapshot 切换间隔**强制 ≥ 5 秒**——避免"快照抖动"

2. **玩家进入房间后退回主菜单**
   - **触发条件：** 玩家进入 1-1 后按 ESC 回主菜单
   - **音频预期：** L1+L2 章节 BGM 渐出 + 主菜单 BGM 渐入（3s cross-fade）

3. **玩家重玩已通关章节**
   - **触发条件：** 玩家通关 Ch1 后重玩 1-1
   - **音频预期：** 章节 BGM 持续 + 无主旋律回归（theme-motifs §5.2）

4. **心流阶段误判（边界数据）**
   - **触发条件：** 玩家状态数据临界（如错误次数 5 + 停留 14.9 min）
   - **音频预期：** 心流阶段**保守判定**（优先 P3 微焦虑而非 P1 极焦虑）——避免"误报警"

5. **教学音已触发后玩家重玩**
   - **触发条件：** 玩家首次通关 1-1 后重玩 1-1
   - **音频预期：** 教学音**不重新触发**（已用 PlayerPrefs 记录）

6. **L4 兜底时玩家突然通关**
   - **触发条件：** 玩家停留 30+ min 触发 L4 兜底后 5s 内通关
   - **音频预期：** L4 兜底被 L3 通关音打断，主旋律回归立即触发

7. **平台规格超出预期（PS5 / Xbox）**
   - **触发条件：** 玩家在 PS5 / Xbox 上运行 v1.0（v1.0 仅 Steam + Mac）
   - **音频预期：** v1.0 PS5 / Xbox **不支持**——v2.0 扩展支持

## 9. 验收标准 (Acceptance Criteria)

- [x] **AC-01** 焦虑-无聊平衡 6 档（P1 极焦虑 ~ P6 无聊）+ 触发条件 + L1/L3/L5 音量 + Pillar 关系
- [x] **AC-02** 6 档自适应触发数据（5 数据源 + 5 秒检测一次）
- [x] **AC-03** 6 档 AudioMixer Snapshot 配置（6 snapshot C# 代码）
- [x] **AC-04** 房间主题自适应规则（5 触发点 + 音频动作 + 时长）
- [x] **AC-05** 房间主题与 5 段落音色对应表 + Boss 房独立 BGM
- [x] **AC-06** 通关时主旋律回归（3 触发分级）
- [x] **AC-07** 心流 7 阶段（S1 起始 ~ S7 完成）+ 玩家状态 + 音频对应
- [x] **AC-08** 心流 7 阶段音频参数表（5 层级音量 + 风格层）
- [x] **AC-09** 心流阶段判定算法（挑战-技能匹配度 + 4 公式）
- [x] **AC-10** 9 教学触发清单（1-1 ~ 1-5 + 2-4 + 3-3 + 3-5）
- [x] **AC-11** 教学曲线音频情绪变化 + 教学音频自适应规则
- [x] **AC-12** 反馈 3 层同步契约（视觉 200ms / 音频 -12dB / 触觉 16ms）
- [x] **AC-13** 反馈 3 层时序图（t=0ms ~ t=300ms）
- [x] **AC-14** 听障 / 视障 / 色盲+听障双重障碍自适应
- [x] **AC-15** 7 平台自适应对照表（编码 / 通道 / 空间 / 性能 / BGM 通道）
- [x] **AC-16** 7 平台音频自适应 PlayerPrefs 配置
- [x] **AC-17** 难度自适应规则（4 档 + L1 主旋律 dB + 体验意图）
- [x] **AC-18** 难度自适应上限保护（P0-001 OPEN 跟踪 + 未来 C# 代码）
- [x] **AC-19** 难度自适应与 P0-001 跟踪矩阵
- [x] **AC-20** 边界条件 7 条（每条含触发条件 + 音频预期）
- [x] **AC-21** 与 09-v2 + 06-v2 + 08-v2 + 04-v2 + 07-v2 + design/data/p0-001-tracking.md 引用 8+ 处

## 10. 关联文档

### 10.1 上游（本文档依赖）

- [`docs/09-audio-v2.md`](../../docs/09-audio-v2.md) §2 动态混音 + §5.1 4 强度反馈 + §6 教学曲线 + §7 无障碍
- [`docs/06-player-experience-v2.md`](../../docs/06-player-experience-v2.md) §6 心流 + §11.2 焦虑-无聊 + §10 无障碍
- [`docs/08-ui-ux-v2.md`](../../docs/08-ui-ux-v2.md) §7 反馈 3 层同步契约
- [`docs/04-gameplay-flow-v2.md`](../../docs/04-gameplay-flow-v2.md) §5 教学曲线
- [`docs/07-failure-retry-v2.md`](../../docs/07-failure-retry-v2.md) §7 失败反馈机制 4 强度
- [`docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) §3 槽位类型 + §13 AC-06 **P0-001 难度上限 20**
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) §1 7 平台
- [`design/data/p0-001-tracking.md`](../data/p0-001-tracking.md) §1 + §2 + §3 + §4

### 10.2 下游（本文档被依赖）

- [`layer-mixing.md`](./layer-mixing.md) §6 心流曲线 + §2 4 强度反馈
- [`cross-platform.md`](./cross-platform.md) §1-5 7 平台自适应
- `src/Audio/AdaptiveAudioDirector.cs` — 6 档自适应 + 7 阶段心流
- `src/Audio/PlatformAudioAdapter.cs` — 7 平台自适应
- `src/Audio/TutorialAudioPlayer.cs` — 9 教学触发
- `src/Audio/AmbientAudioSystem.cs` — DM-01 距槽位渐强

## 11. 待办事项 (TODO)

- [ ] **P0：** 实现 AdaptiveAudioDirector（6 档 AudioMixer Snapshot + 5 秒检测 + 心流 7 阶段）— 阻塞 v1.0 [§1 + §3]
- [ ] **P0：** 实现 TutorialAudioPlayer（9 教学触发 + PlayerPrefs 标志）— 阻塞 v1.0 完整版 [§4]
- [ ] **P0：** 实现 FeedbackTriSync（3 层同步契约 200ms/-12dB/16ms）— 阻塞核心循环 [§5]
- [ ] **P0：** 实现 PlatformAudioAdapter（7 平台 + 编码 / 通道 / 性能）— 不阻塞 v1.0（v1.0 仅 Steam + Mac）[§6]
- [ ] **P1：** 实现心流阶段判定算法（挑战-技能匹配度 + 4 公式）— 不阻塞 v1.0（可硬编码 S1-S7）[§3.3]
- [ ] **P1：** 实现无障碍 3 通道（听障 / 视障 / 字幕）— 不阻塞 v1.0（v1.0 仅字幕）[§5.3]
- [ ] **P2：** 实现难度自适应上限保护（待 P0-001 解决后，DIFFICULTY_MAX = 20）— **P0-001 OPEN 跟踪** [§7.2 + design/data/p0-001-tracking.md]
- [ ] **P2：** 评估 Adaptive Music（动态自适应音乐，5 段落 3 变奏展开）— v1.1 评估 [Q-01]

## 12. 变更日志

| 日期 | 版本 | 变更人 | 内容 |
|------|------|--------|------|
| 2026-06-30 | v1.0 | 中书省 subagent (MUSIC-01) | **adaptive-system.md 初版:** 玩家状态 6 档（P1 极焦虑 ~ P6 无聊）+ 6 档触发数据 + 6 snapshot C# / 房间主题自适应（5 触发点 + Boss 房独立 BGM）/ 心流 7 阶段（S1 起始 ~ S7 完成）+ 7 阶段音频参数表 + 心流判定算法 / 9 教学触发清单（1-1 ~ 1-5 + 2-4 + 3-3 + 3-5）/ 反馈 3 层同步契约（200ms / -12dB / 16ms）+ 3 层时序图 + 听障/视障/双重障碍 / 7 平台自适应对照表 + 7 平台 PlayerPrefs / 难度自适应 4 档 + 上限保护（P0-001 OPEN 跟踪）+ 未来 C# 代码 / 7 边界条件 / 21 验收标准 / 11 关联引用 / 8 待办 |

---

**最后更新：** 2026-06-30
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）
**P0-001 跟踪：** OPEN — 02-v2 §13 AC-06 待增补"难度上限 20"硬约束（详见 §7.3 + design/data/p0-001-tracking.md §1.4）
