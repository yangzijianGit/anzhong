---
title: 《暗室》风格定义 (Style)
doc_id: DES-anzhong-music-style
parent: design/music/README.md
last_updated: 2026-06-30
version: v1.0
status: draft
owner: 音乐总监（中书省 subagent · MUSIC-01）
---

# 《暗室》风格定义（design/music/style.md）

> **一句话定位：** dark ambient + puzzle + minimalist — 4 Pillar 对应 4 风格层 → 4 参考作品 → 5 段落音色 → 7 平台音频风格 → 4 强度反馈对应风格 → 6 风格限制。

## 目的 (Purpose)

本文档是《暗室》音乐**整体风格契约**的完整定义。它向：

- **音乐总监 / 音频工程师** — 讲清**风格哲学**（为什么这个游戏用 dark ambient + puzzle + minimalist）+ **4 Pillar 映射** + **6 风格限制**
- **外包商** — 4 参考作品（Dear Esther + Gorogoa + 锈湖系列 + Inside）的音乐风格理解基准
- **Unity 工程师** — 4 强度反馈对应风格层（L1 微弱 / L2 明显 / L3 强烈 / L4 危机）
- **QA / 测试** — 6 风格限制（避免旋律化 / 人声 / 复杂节奏 / 突然惊吓等）的验收基线

**本文档范围：** **风格（Style）**——音乐情绪、参考作品、风格限制；不涉及具体乐器 / 混音 / 编码（详见 `instruments.md` + `layer-mixing.md` + `standards.md`）。

## 1. 总体风格 (Overall Style)

> **核心风格：** **dark ambient + puzzle + minimalist** = 沉浸感 + 拓扑重构惊喜 + 视觉欺骗 + 一致性

### 1.1 4 Pillar → 4 风格层映射 (4 Pillars × 4 Style Layers)

| Pillar | 设计哲学 | 音乐风格 | 风格层 | 典型应用 |
|--------|---------|---------|--------|---------|
| **P1 沉浸感** | 玩家"进入"废弃设施而非"观看"它 | **dark ambient**（深沉 + 空间感）| 风格 L1: 基础氛围层 | 章节 BGM 基础层（pad + 环境音 + 极远混响）|
| **P2 拓扑重构惊喜** | 切换动画是核心体验"顿悟"时刻的视觉放大 | **puzzle**（不和谐音 + 突然安静）| 风格 L2: 切换冲击层 | 4 槽位切换动机 + 章节过渡 |
| **P3 视觉欺骗** | Ch3 引入"视觉对称 ≠ 逻辑对称"颠覆 | **minimalist**（极简 + 留白）| 风格 L3: 简约留白层 | 19 房间 BGM 变奏（5 音符）+ 极简节奏 |
| **P4 一致性** | 19 房间视觉风格统一（章节内统一，章节间渐进）| **主题循环**（重复 + 渐变）| 风格 L4: 主题回归层 | 主旋律 90s 循环 + 通关时主旋律回归 |

> **关键设计：** 4 风格层**叠加**——同一首 BGM 可以同时包含 dark ambient（基础）+ puzzle（切换）+ minimalist（旋律）+ 主题循环（结构）。

### 1.2 风格频谱定位 (Style Spectrum Positioning)

```
极简 (minimalist) ←———————|——→ 复杂 (complex)
              ●（dark ambient）         
                              ●（orchestral）   
                                    
低能量 ←———————|——→ 高能量
   ●（calm / explore）      
                          ●（intense / rhythm）
                              
2 轨道 ←———————|——→ 12 轨道
   ●（BGM + 1 Ambient）     
                       ●（full orchestra）
                           
1/8 音符 ←———————|——→ 16 分音符
   ●（simple rhythm）     
                            ●（flamenco / 鼓点 +）
```

**《暗室》风格定位：**
- 复杂度：3-5 轨道（章节 BGM 基础 + 节奏层 + 紧张层 + 切换动机 + 环境音）
- 能量：低-中（章节 BGM 基础层 mp / 章节过渡 mf / 通关 ff）
- 节奏：简单（无 16 分音符 + 切分音，避免紧张感）
- 频率范围：100Hz-8kHz（极低 80Hz 鼓点 + 极高 8kHz 风铃，避免人声 200-1000Hz 主频）

### 1.3 风格统一性 (Style Consistency)

> **设计原则：** 19 房间音乐风格**统一**——只有"颜色 + 节奏"变化，"结构 + 动机"不变。

| 维度 | 一致（不变）| 多样（变） |
|------|-----------|----------|
| **乐器** | 大提琴 + 钢琴 + pad（始终在场）| 章节加合成器 / Ch3 加鼓点 / Boss 房加不和谐 |
| **和声** | C 大调五声音阶（C-D-E-G-A）| 章节副调和声（Ch1 I-IV-V / Ch2 i-iv-v / Ch3 增四度）|
| **节奏型** | 4/4 拍，1/4 音符底 | 章节节奏层密度（Ch1 0% / Ch2 50% / Ch3 100%）|
| **力度** | mp（中弱）基础 | 强度反馈触发 mf / f / ff |
| **循环点** | 60-90s 主旋律循环 | 间章 8s 一次性 / 通关 30s 一次性 |
| **动机** | M1 转换 + M2 觉醒 + M3 连接 | 章节加主导动机（Ch3 加不和谐）|

## 2. 参考作品 (4 Reference Works)

> **设计原则：** 4 参考作品**功能互补**——Dear Esther 提供 ambient / Gorogoa 提供极简 / 锈湖提供 puzzle / Inside 提供 narrative。

### 2.1 4 参考作品对比表 (4 Reference Comparison)

| 参考作品 | 类型 | 关键音乐特征 | 《暗室》学到的 | 引用 |
|---------|------|------------|-------------|------|
| **Dear Esther** (2012) | Walking simulator | 第一人称 + ambient pad + 极远混响 + 无节奏 | **沉浸感风格**——第一人称视效配合 dark ambient pad，强化"我在这里" | 12-v2 §2.1 |
| **Gorogoa** (2017) | Puzzle | 极简 + 留白 + 钢琴独奏 + 无鼓点 | **极简风格**——留白 + 钢琴，强化"我在思考" | 12-v2 §2.1 |
| **锈湖系列 (Rusty Lake)** (2015-) | Puzzle horror | 诡异 + 不和谐 + 短动机 + 突变音 | **puzzle 风格**——诡异 + 不和谐强化"我被骗了"，但避免突然惊吓 | 12-v2 §2.1 |
| **Inside** (2016) | Puzzle platformer | 大型管弦乐 + 沉默 + 长混响 + 极度克制 | **主题回归风格**——通关时主旋律回归 + 长混响，强化"我完成了" | 12-v2 §2.1 |

> **关键设计：** 4 参考作品**不混合**——每个 Pillar 仅学 1 个参考作品的**核心风格**，避免"风格拼贴"。

### 2.2 4 Pillar 风格细节 (4 Pillar Style Details)

#### P1 dark ambient —— Dear Esther 风

- **特征：** 长混响（>5s）+ 低频 pad（80-200Hz）+ 极远声场（pan -0.3 ~ -0.8）+ 无明显旋律
- **频谱：** 100-500Hz 主导 + 1-3kHz pad 衬底 + 8kHz+ 极高频点缀
- **力度：** pp (pianissimo) 到 p (piano)
- **轨道：** 1-3 轨道（pad + ambient + drone）
- **典型应用：** 章节 BGM 基础层（bgm_chapter1_01 / bgm_chapter2_01 / bgm_chapter3_01）

#### P2 puzzle —— 锈湖风

- **特征：** 不和谐音程（增四度 / 减五度）+ 短动机（4-8 小节）+ 突变音（0.5s 留白后突变）
- **频谱：** 300-2000Hz 主导 + 短促打击音
- **力度：** mp 到 mf（中等强度）
- **轨道：** 1-2 轨道（动机 + 节奏）
- **典型应用：** 4 槽位切换动机 + 章节过渡（3-8 → 1-1）

#### P3 minimalist —— Gorogoa 风

- **特征：** 极简 5 音符（C-D-E-G-A）+ 大留白（每 4-8 小节 1 次静音）+ 钢琴独奏
- **频谱：** 500-2000Hz 主导（钢琴中音区）
- **力度：** pp 到 p
- **轨道：** 1 轨道（钢琴独奏 + 偶尔 pad）
- **典型应用：** 1-4 / 2-6 喘息房（节奏层 0%）+ 主旋律原型

#### P4 主题循环 —— Inside 风

- **特征：** 主旋律 90s 完整循环 + 通关时主旋律回归 + 长混响（>3s）+ 大型管弦乐（通关瞬间）
- **频谱：** 200-4000Hz 全频覆盖
- **力度：** p 基础 → f 通关 → ff 终章通关
- **轨道：** 4-12 轨道（管弦乐全编制 + 钟声 + 混响）
- **典型应用：** 通关音（sfx_win_game_01） + bgm_ending_01 + 主旋律回归

## 3. 5 段落音色风格细节 (5-Section Style Details)

### 3.1 5 段落音色风格对照表 (5-Section Style Table)

| 段落 | 章节 | 风格层 | Pillar 主导 | 风格细节 | 典型乐器 | 轨道数 | bpm |
|------|------|--------|------------|---------|---------|:-----:|:---:|
| **序章** | 主菜单 | L1 dark ambient | P1 沉浸感 | 长混响 + 极远声场 + 神秘 | 大提琴 + pad + 风声 | 1-3 | 60 |
| **第一章** | Ch1 觉醒 | L3 minimalist | P3 视觉欺骗 / P1 沉浸感 | 极简钢琴 + 大留白 + 平静 | 钢琴 + 弦乐 + 风声 | 2-3 | 72 |
| **间章** | 章节过渡 | L2 puzzle + L3 minimalist | P2 拓扑重构 | 短动机 + 突变 + 留白 | 风铃 + pad + 钢琴高音 | 2-3 | 72 |
| **第二章** | Ch2 深掘 | L1 dark ambient + L2 puzzle | P1 沉浸感 / P4 一致性 | 低沉 + 远处机械声 + 不和谐 | 合成器 + 低频 + 鼓点 | 3-4 | 72 |
| **第三章** | Ch3 迷途 | L4 主题循环 + L2 puzzle | P2 拓扑重构 / P3 视觉欺骗 / P4 一致性 | 不和谐 + 鼓点 + 突变 + 不和谐 | 合成器 + 鼓点 + 突变音 + 全乐器 | 4-6 | 80-100 |

> **关键设计：** 5 段落**强度递增**（序章 bpm 60 → 第三章 Boss 房 bpm 100），强化"难度递增 = 节奏递增"。

### 3.2 段落间过渡风格契约 (Section Transition Style)

| 过渡点 | 风格变化 | 切换时长 | 引用 |
|-------|---------|---------|------|
| **BootUp → MainMenu** | silence → dark ambient | 0s（直接播放）| 09-v2 §2.2 |
| **MainMenu → Ch1** | dark ambient + 长混响 → minimalist 钢琴 | 2s cross-fade | 09-v2 §2.2 |
| **Ch1 → Interlude** | minimalist 钢琴 → puzzle 风铃 + pad | 3s cross-fade | 本文 §4.3 |
| **Interlude → Ch2** | puzzle 风铃 → dark ambient 合成器 + 低频 | 2s cross-fade | 本文 §4.3 |
| **Ch2 → Interlude** | dark ambient + 低频 → puzzle 风铃 + pad | 3s cross-fade | 本文 §4.3 |
| **Interlude → Ch3** | puzzle 风铃 → 主题循环 + puzzle + minimalist | 2s cross-fade | 本文 §4.3 |
| **Ch3 → End** | 主题循环 + 鼓点 → Inside 风管弦乐 + 钟声 + 主旋律回归 | 3s cross-fade | 09-v2 §2.2 |

## 4. 7 平台音频风格 (7-Platform Audio Style)

> **设计原则：** 7 平台音频风格**统一主旋律**（不变），仅**格式 / 编码 / 通道数**变化。

### 4.1 7 平台音频风格对照表 (7-Platform Audio Style)

| 平台 | 风格保持度 | 编码偏好 | 通道数 | 空间音频 | 性能预算 | 优先级 |
|------|:---------:|---------|:-----:|---------|---------|:----:|
| **PC Steam** | 100% (全功能) | Opus (主) + WAV 源文件 | 立体声 | Dolby Atmos (5.1/7.1) | 60 FPS | **P0** |
| **PC Mac** | 100% (全功能) | Opus + ALAC | 立体声 | Dolby Atmos | 60 FPS | **P0** |
| **PS5** | 100% (全功能) | Opus + AAC | 立体声 + Tempest 3D | Sony 360 Reality Audio | 60 FPS | **P2** |
| **Xbox Series X\|S** | 100% (全功能) | Opus + AAC | 立体声 + Project Acoustics | Dolby Atmos | 60 FPS | **P2** |
| **Nintendo Switch** | 90% (压缩) | Opus 64 kbps | 立体声（无空间）| 立体声 | 30 FPS | **P1** |
| **iOS (iPhone/iPad)** | 100% (立体声) | AAC + Opus | 立体声 | Dolby Atmos (iPhone 12+) | 60 FPS | **P2** |
| **Android** | 100% (立体声) | Opus (主) + OGG Vorbis | 立体声 | Dolby Atmos (Android 13+) | 60 FPS | **P2** |

> **关键设计：** 音频风格**不随平台变化**——v1.0 制作 1 套主旋律，平台发布时**自动转码**（Opus / OGG / AAC）+ **通道数调整**（立体声 / Dolby Atmos）。

### 4.2 7 平台压缩策略 (7-Platform Compression Strategy)

```
源文件 (44.1kHz 16-bit WAV + FLAC)
       ↓
   [自动转码]
       ↓
PC/Mac/PS/Xbox     Switch (压缩 50%)    iOS/Android (立体声)
WAV + Opus 128kbps   Opus 64 kbps         AAC 128kbps
44.1kHz 16-bit      22.05kHz 8-bit        44.1kHz 16-bit
立体声                立体声                立体声
```

> **关键设计：** v1.0 源文件**统一 44.1kHz 16-bit WAV**——平台发布时**自动转码**（无需重新制作）。详见 `standards.md` §1 + `cross-platform.md` §2。

## 5. 4 强度反馈对应风格 (4 Intensity Feedback Style)

> **设计原则：** 4 强度反馈**风格递增**（L1 微弱 / L2 明显 / L3 强烈 / L4 危机）——传达"问题越来越严重但游戏不惩罚你"。

### 5.1 4 强度反馈风格对照表 (4 Intensity Style)

| 强度 | 视觉反馈 | 音频风格层 | 音色细节 | 力度 | 与 Pillar 关系 |
|------|---------|----------|---------|------|--------------|
| **L1 轻反馈** | 无额外视觉 | 切换音 + 基础 pad | -12dB mp 短促 | mp | P2 puzzle 短动机 |
| **L2 中反馈** | 槽位暗淡脉冲 -30% | 错音 + 不和谐短音 | -12dB mp 不和谐 | mp | P2 puzzle 不和谐 |
| **L3 重反馈** | 槽位暗淡脉冲 -50% | 错音 ×2（连发）+ 鼓点 | -12dB mp ×2 鼓点 | mp → mf | P2 puzzle + L1 dark |
| **L4 兜底** | 全房间光线变暗 | 静音（突然）+ 5s 后风铃 | 静音 → -12dB 风铃 | silence → p | P2 puzzle 突变 |

> **关键设计：** L4 兜底**突然静音**反而是**最大警告**——配合"全房间光线变暗"形成"无助感"暗示（与 07-v2 §7 L4 一致）。
> **关键设计：** 4 强度**音量递增**（L1 -12dB → L2 -12dB → L3 -12dB×2 → L4 静音），**风格强度递增**——传达"问题越来越严重"。

### 5.2 4 强度反馈触发契约 (4 Intensity Trigger Contract)

```
玩家错误次数 1 次  → L1 轻反馈（切换音 -12dB 短促）
玩家错误次数 3 次  → L2 中反馈（错音 -12dB 不和谐短音）
玩家错误次数 5+ 次 → L3 重反馈（错音 ×2 连发 + 鼓点 mp → mf）
玩家停留 30+ min  → L4 兜底（静音 → 5s 后风铃）
```

> **触发数据驱动：** 由 `src/Audio/IntensityFeedbackAudio.cs` 模块读取 `PlayerPrefs.errorCount` + `TimeInRoom`，触发对应强度音频。

## 6. 风格限制 (Style Constraints)

> **设计原则：** 6 风格限制**严格遵守**——避免破坏 dark ambient + puzzle + minimalist 的核心风格。

| # | 限制 | 原因 | 例外 | 验收 |
|---|------|------|------|------|
| **SC-01** | **避免旋律化**——BGM 不使用明显旋律 + 大跨度音程（≤ 5 度音程）| dark ambient + minimalist 不需要"歌曲式旋律" | 通关音可以用 5 度上升（C5→G5）+ Boss 房可以用不和谐 | 主旋律 5 音符约束 |
| **SC-02** | **避免人声**——BGM / SFX 不使用人声 / 语音（除 TTS 无障碍）| dark ambient + minimalist 不需要"歌手" | TTS（无障碍）必须 + Suno/Udio 可生成人声 BGM 但**禁止使用** | 所有 BGM 乐器化 |
| **SC-03** | **避免复杂节奏**——BGM 不使用 16 分音符 / 切分音 / 复合节奏 | minimalist + puzzle 不需要"复杂节奏" | Ch3 可用简单鼓点（4/4 + 1/4 音符底）| 节奏型简单 |
| **SC-04** | **避免突然惊吓**（No Jump Scares）——BGM / SFX 不使用突然大声（>10dB 突变）| 本游戏无死亡惩罚，无恐怖元素 | L4 兜底"突然静音"是允许的（配合视觉变暗）| 渐变 2-3s |
| **SC-05** | **避免主频 200-1000Hz 人声区**——BGM 不在该频段长期（>5s）持续 | 避免与 TTS 冲突 + 避免"歌手"感 | 主菜单可短时使用 200-500Hz 大提琴 | 大提琴 < 1 分钟 |
| **SC-06** | **避免歌词**——所有 BGM 无人声歌词 | minimalist 不需要歌词 | TTS（无障碍）必须，但**不算 BGM** | 所有 BGM 纯乐器 |

### 6.1 风格限制验收清单 (Style Constraint Checklist)

```
[ ] SC-01: 主旋律 5 音符（C-D-E-G-A）大调五声音阶
[ ] SC-02: 所有 5 首 BGM 无人声 / 无歌词（仅 TTS 用于无障碍）
[ ] SC-03: 所有 BGM 无 16 分音符 / 无复合节奏（Ch3 简单鼓点除外）
[ ] SC-04: 所有跨音 fade-in/fade-out 2-3s（无突然大声）
[ ] SC-05: BGM 主频避开 200-1000Hz 人声区（大提琴 ≤ 1min 可用）
[ ] SC-06: 5 首章节 BGM 纯乐器（无人声）
```

## 7. 边界条件 (Edge Cases)

1. **玩家使用耳机 vs 扬声器**
   - **触发条件：** 玩家选择音频输出设备
   - **音频预期：** 耳机模式下**电流声方向感**更明显（立体声 pan），扬声器模式下**整体音量**略低（防止爆音）

2. **玩家关闭音频（AudioSettings.mute）**
   - **触发条件：** 主菜单 → 设置 → 关闭音频
   - **音频预期：** 所有 SFX / BGM / 环境音静音，但视觉 / 触觉反馈照常工作（与 08-v2 §7 三层反馈契约）

3. **低帧率 30 FPS（性能边界）**
   - **触发条件：** 低端机或后台进程占用 CPU
   - **音频预期：** 音频**不依赖帧率**（独立线程），即使帧率掉到 30 FPS 音频依然流畅

4. **玩家已习惯 5 首 BGM 主旋律，无新鲜感**
   - **触发条件：** 玩家通关后重玩
   - **音频预期：** 已通关房 / 已通关章节**不回归**主旋律，仅章节 BGM 持续（无重新播放主旋律）

5. **Boss 房 (3-8) 30 min 未通关**
   - **触发条件：** 玩家停留 30+ min
   - **音频预期：** Boss 房 BGM 持续，但**强制 Hint 触发**——TTS 朗读"试试 4 选项的 CycleSlot 组合"

6. **Suno/Udio 生成 BGM 风格不符（戏剧化太重）**
   - **触发条件：** Suno/Udio AI 生成风格过于激烈 / 不符合 dark ambient
   - **音频预期：** 重新生成 + 提供更详细 prompt（4 Pillar 风格约束），最多 5 次重试

## 8. 验收标准 (Acceptance Criteria)

- [x] **AC-01** 4 Pillar → 4 风格层映射（dark ambient / puzzle / minimalist / 主题循环）
- [x] **AC-02** 4 参考作品对比表（Dear Esther / Gorogoa / 锈湖 / Inside）+ 学到的核心风格
- [x] **AC-03** 4 Pillar 风格细节（频谱 / 力度 / 轨道 / 典型应用）
- [x] **AC-04** 5 段落音色风格细节表（5 段落 + 风格层 + 主导 Pillar + 乐器 + 轨道 + bpm）
- [x] **AC-05** 段落间过渡风格契约（7 过渡点 + 切换时长）
- [x] **AC-06** 7 平台音频风格对照表（风格保持度 + 编码 + 通道 + 空间 + 性能 + 优先级）
- [x] **AC-07** 4 强度反馈对应风格层（L1-L4 + 视觉 + 音频 + 力度 + Pillar 关系）
- [x] **AC-08** 6 风格限制（避免旋律化 / 人声 / 复杂节奏 / 突然惊吓 / 人声主频 / 歌词）+ 验收清单
- [x] **AC-09** 边界条件 6 条（每条含触发条件 + 音频预期）
- [x] **AC-10** 与 09-v2 + 12-v2 + 06-v2 + 07-v2 + 11-v2 + design/art/style-guide.md 引用 7+ 处

## 9. 关联文档

### 9.1 上游（本文档依赖）

- [`docs/09-audio-v2.md`](../../docs/09-audio-v2.md) §1.7 章节 BGM + §5.1 4 强度反馈
- [`docs/12-art-style-v2.md`](../../docs/12-art-style-v2.md) §1 4 Pillar + §2 3 参考作品
- [`docs/07-failure-retry-v2.md`](../../docs/07-failure-retry-v2.md) §7 失败反馈机制 3 通道 4 强度
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) §1.1 7 平台覆盖 + §5 6 项合规
- [`design/art/style-guide.md`](../art/style-guide.md) §6 7 平台美术规格

### 9.2 下游（本文档被依赖）

- [`instruments.md`](./instruments.md) §3 5 段落主乐器（按风格选型）
- [`layer-mixing.md`](./layer-mixing.md) §1 5 层级架构（4 Pillar 对应 5 层）
- [`adaptive-system.md`](./adaptive-system.md) §1 玩家状态 6 档（4 强度反馈 → 状态触发）
- `src/Audio/StyleConstraints.cs` — 6 风格限制自动校验（CI/CD）
- `src/Audio/ChapterBGMController.cs` — 5 段落风格切换契约

## 10. 待办事项 (TODO)

- [ ] **P0：** Suno/Udio 生成 5 首章节 BGM（提供 4 Pillar 风格 prompt）— 阻塞 v1.0 [§3.1 + 09-v2 §9.2]
- [ ] **P0：** 5 首 BGM 风格验收（dark ambient + puzzle + minimalist + 主题循环）— 阻塞 v1.0 [AC-01-04]
- [ ] **P0：** 6 风格限制自动校验脚本（StyleConstraints.cs + CI/CD）— 阻塞 v1.0 [§6.1]
- [ ] **P1：** 实现 4 强度反馈音频触发（与 09-v2 §5.1 + 07-v2 §7 同步）— 不阻塞 v1.0 [§5]
- [ ] **P1：** 实现 7 平台音频风格自动转码（Opus / OGG / AAC / ALAC）— 不阻塞 v1.0（v1.0 仅 PC Steam/Mac）[§4.1-2]
- [ ] **P2：** 评估 4 参考作品"功能互补"测试（让 KOL 听 4 参考 + 暗室后评分）— v1.1 评估 [§2.1]

## 11. 变更日志

| 日期 | 版本 | 变更人 | 内容 |
|------|------|--------|------|
| 2026-06-30 | v1.0 | 中书省 subagent (MUSIC-01) | **style.md 初版:** 4 Pillar → 4 风格层（dark ambient + puzzle + minimalist + 主题循环）/ 4 参考作品对比表（Dear Esther + Gorogoa + 锈湖 + Inside）+ 4 Pillar 风格细节 / 5 段落音色风格细节（5 段 + 风格层 + 主导 Pillar + 乐器 + 轨道 + bpm）/ 7 平台音频风格对照（风格保持度 + 编码 + 通道 + 空间 + 性能）/ 4 强度反馈风格层（L1-L4 + 视觉 + 音频 + 力度 + Pillar）/ 6 风格限制（避免旋律化 / 人声 / 复杂节奏 / 突然惊吓 / 人声主频 / 歌词）+ 验收清单 / 6 边界条件 / 10 验收标准 / 7+ 关联引用 / 6 待办 |

---

**最后更新：** 2026-06-30
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）
