---
title: 美术资产清单
doc_id: DES-anzhong-art-asset-list
parent: design/art/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 美术总监（中书省 subagent）
---

# 《暗室》美术资产清单（asset-list.md）

> **一句话定位：** 3 阶段美术资产清单 × 7 预制件 × 19 房间 × 9 画集 × 6 工具 × 4 字体 的全量资源台账，与 `12-art-style-v2.md` §9 美术资源方案完全对齐。

## 目的 (Purpose)

本文档是《暗室》美术层的**资源台账**。它向美术总监（中书省）、Unity 工程师、外包合作伙伴、未来的维护者**用 15 分钟讲清**：

- **3 阶段美术资源**（原型 / Kenney 过渡 / 自制正式期）的具体文件清单
- **7 预制件 × 4 状态** 的资源规格（与 12-v2 §9.3 + 02-v2 §3 严格对齐）
- **19 房间美术资源**（与 03-v2 §5 房间配置表一一映射）
- **9 张数字画集**（豪华版 DLC 内容，与 11-v2 §2.1 + 12-v2 §9.4 对齐）
- **6 美术工具链**（与 12-v2 §9.5 对齐，总成本 ≤ $20）
- **4 字体授权**（与 12-v2 §7.4 + 08-v2 §9.2 对齐）

**本文与 12-v2 §9 的边界：** 12-v2 §9 是资源方案**概要**（数量 + 来源 + 工时），本文档是资源**完整清单**（文件名 + 路径 + 规格 + 工时 + 验收 + 来源）。

## 范围 (Scope)

### 包含

- **3 阶段美术资源台账**（原型 W01-W02 / Kenney 过渡 W03-W07 / 自制正式期 W08-W12）
- **7 预制件 × 4 状态** 资源清单（SolidWall / Floor / Door / GlassWall / CrumblingFloor / FakeFloor / PressurePlate）
- **19 房间美术资源**（房间 ID → 美术文件映射 + 章节主题）
- **9 张数字画集**（豪华版 DLC）
- **6 美术工具链**（Aseprite / Unity Tilemap / Shader Graph / DOTween / Sim Daltonism / Addressables）
- **4 字体授权清单**（en/zh-CN/zh-TW/ja/ko）
- **资源验收标准**（分辨率 / 格式 / 命名 / 打包）
- **资源版本控制**（Git LFS / Addressables 分组）

### 不包含 (Out of Scope)

- 调色板/字体/字号/动画**规格定义** → 见 `style-guide.md` + `12-art-style-v2.md` §3 §4 §7 §8
- 美术制作流程（草图→线稿→上色→动画→集成）→ 见 `production-pipeline.md`
- 外包策略（哪些自营/哪些外包）→ 见 `outsourcing.md`
- 版权与授权（CC0/OFL/自制）→ 见 `copyright.md`
- 美术预算（时间 + 人力 + 资金）→ 见 `asset-budget.md`
- 美术验收清单（8 文件 + 7 预制件 + 19 房间 + 9 画集）→ 见 `delivery-checklist.md`

## 1. 一句话描述 (One-liner)

> **"3 阶段 × 50 美术文件 × 7 预制件 × 19 房间 × 9 画集 × 6 工具 — 总成本 $0-20 + 1 人 Solo 14h 自制。"**

## 2. 3 阶段美术资源总表 (3-Phase Asset Master Table)

> 与 `12-art-style-v2.md` §9.1 3 阶段路径对齐 + `10-roadmap-v2.md` §1.1 4 阶段 12 里程碑对齐。

### 2.1 阶段 0 原型期 (Prototype · W01-W02 · M01-M02)

| 类别 | 文件 | 来源 | 工时 | 路径 |
|------|------|------|:---:|------|
| **Unity 工程脚手架** | Unity 2022 LTS + URP 2D + DOTween 工程模板 | Unity Hub | 2h | `unity/Assets/` |
| **白盒方块** | 32px × 32px 灰色方块 | Unity Primitive + 自制色 | 1h | `Assets/Art/WhiteBox/` |
| **白盒玩家精灵** | 12px 圆形（青色）| Unity Sprite | 1h | `Assets/Art/WhiteBox/` |
| **白盒出口** | 32px × 32px 橙色方块 | Unity Sprite | 0.5h | `Assets/Art/WhiteBox/` |
| **白盒 4 状态槽位** | 24px × 24px 4 状态（Idle/Hover/Switching/Disabled）| Unity Sprite | 2h | `Assets/Art/WhiteBox/` |
| **阶段 0 合计** | — | — | **6.5h** | — |

### 2.2 阶段 1 Alpha (Transition · W03-W07 · M03-M07)

| 类别 | 文件 | 来源 | 工时 | 路径 |
|------|------|------|:---:|------|
| **Kenney 2D Platformer Pack** | 7 预制件 + 玩家精灵 (CC0) | kenney.nl | $0 / 0h | `Assets/Art/Imports/Kenney/2DPlatformer/` |
| **Kenney UI Pack** | HUD + 菜单 (CC0) | kenney.nl | $0 / 0h | `Assets/Art/Imports/Kenney/UI/` |
| **Kenney Particle Pack** | 通关粒子 (CC0) | kenney.nl | $0 / 0h | `Assets/Art/Imports/Kenney/Particles/` |
| **Kenney Audio Pack** | 切换音 / 错音 (CC0) | kenney.nl | $0 / 0h | `Assets/Art/Imports/Kenney/Audio/` |
| **Kenney 调色版** | 4 预制件（SolidWall/Floor/Door/GlassWall）调色到 12 主色 | 自制 | 4h | `Assets/Art/KenneyAdapted/` |
| **房间 1-1 ~ 2-6 美术** | 11 房间 Tilemap + 槽位配置 | Kenney 适配 | 33h (11×3h) | `Assets/Art/Rooms/` |
| **章节 1 + 2 主题美术** | Ch1 / Ch2 背景色温 + 雾效配置 | URP 2D | 4h | `Assets/Art/ChapterThemes/` |
| **HUD 极简半透明** | rgba(0,0,0,0.6) + 4px backdrop blur | Unity UI | 8h | `Assets/UI/HUD/` |
| **阶段 1 合计** | — | — | **49h** | — |

### 2.3 阶段 2 Beta (Polish · W08-W10 · M08-M10)

| 类别 | 文件 | 来源 | 工时 | 路径 |
|------|------|------|:---:|------|
| **CrumblingFloor 自制** | 32px × 32px + 裂纹纹理 + 碎裂粒子（10 块小方块）| 自制 | 4h | `Assets/Art/Prefabs/CrumblingFloor/` |
| **FakeFloor 自制** | 32px × 32px（与 Floor 1:1 像素匹配）+ 踩上闪烁红色 | 自制 | 2h | `Assets/Art/Prefabs/FakeFloor/` |
| **PressurePlate 自制** | 32px × 32px 圆形 + 橙色描边 + 踩下缩放动画 | 自制 | 2h | `Assets/Art/Prefabs/PressurePlate/` |
| **房间 3-1 ~ 3-8 美术** | 8 房间 Tilemap + 视觉欺骗配置 | 自制 + Kenney 调色 | 24h (8×3h) | `Assets/Art/Rooms/` |
| **章节 3 主题美术** | Ch3 强光影对比 + 30% 雾效 | URP 2D | 4h | `Assets/Art/ChapterThemes/` |
| **通关粒子效果** | 通关瞬间白光闪烁 + 少量粒子 | Unity Particle | 2h | `Assets/Art/Particles/Win/` |
| **阶段 2 合计** | — | — | **38h** | — |

### 2.4 阶段 3 RC + Release (Release · W11-W12 · M11-M12)

| 类别 | 文件 | 来源 | 工时 | 路径 |
|------|------|------|:---:|------|
| **数字画集 Ch1 (3 张)** | 1-1 / 1-3 / 1-5 房间 1080p 静态高清渲染 | 自制 | 4h | `Assets/Art/DigitalArt/Ch1/` |
| **数字画集 Ch2 (3 张)** | 2-1 / 2-4 / 2-5 房间 1080p 静态高清渲染 | 自制 | 4h | `Assets/Art/DigitalArt/Ch2/` |
| **数字画集 Ch3 (3 张)** | 3-3 / 3-5 / 3-8 房间 1080p 静态高清渲染 | 自制 | 6h | `Assets/Art/DigitalArt/Ch3/` |
| **1 分钟制作花絮** | 美术制作过程剪辑 (MP4) | 自制 + 屏幕录制 | 8h | `Assets/Art/BehindTheScenes/` |
| **Steam 商店页 5 张截图** | 5 关键房间 1920×1080 截图 | Unity Game View | 4h | `Assets/Art/Marketing/Screenshots/` |
| **1 分钟宣传视频** | 19 房间剪辑 | OBS + 视频编辑 | 12h | `Assets/Art/Marketing/Video/` |
| **30 秒预告片** | 关键场景剪辑 | OBS + 视频编辑 | 6h | `Assets/Art/Marketing/Teaser/` |
| **阶段 3 合计** | — | — | **44h** | — |

### 2.5 全阶段美术总工时

| 阶段 | 工时 | 文件数 |
|------|:----:|:-----:|
| 阶段 0 原型期 | 6.5h | ~5 |
| 阶段 1 Alpha | 49h | ~15 |
| 阶段 2 Beta | 38h | ~13 |
| 阶段 3 RC + Release | 44h | ~17 |
| **合计** | **137.5h** | **~50** |

> **关键设计：** 美术总工时 137.5h（≤ 180h 预算），保留 42.5h 缓冲给 Bug 修复 + Playtest 反馈 + 平台适配。详见 `asset-budget.md`。

## 3. 7 预制件 × 4 状态资源清单 (7 Prefabs × 4 States)

> 与 `12-art-style-v2.md` §6.3 7 预制件视觉契约 + §3.2 7 预制件调色板 + `02-core-mechanics-v2.md` §3 预制件类型 三方对齐。

### 3.1 7 预制件总表

| # | 预制件 | 美术来源 | 调色版来源 | 工时 | 状态 | 文件路径 |
|---|--------|---------|----------|:---:|:----:|---------|
| **P1** | **SolidWall** | Kenney + 自制调色 | 12-v2 §3.2 | 1h | ✅ | `Assets/Art/Prefabs/SolidWall/` |
| **P2** | **Floor** | Kenney + 自制调色 | 12-v2 §3.2 | 1h | ✅ | `Assets/Art/Prefabs/Floor/` |
| **P3** | **Door** | Kenney + 自制调色 + 开/闭 2 sprite | 12-v2 §3.2 | 2h | ✅ | `Assets/Art/Prefabs/Door/` |
| **P4** | **GlassWall** | 自制（半透明 shader）| 12-v2 §3.2 | 2h | ✅ | `Assets/Art/Prefabs/GlassWall/` |
| **P5** | **CrumblingFloor** | 自制（裂纹纹理 + 碎裂动画）| 12-v2 §3.2 | 4h | ⚠️ 关键 | `Assets/Art/Prefabs/CrumblingFloor/` |
| **P6** | **FakeFloor** | 自制（与 Floor 1:1 匹配）| 12-v2 §3.2 | 2h | ⚠️ 关键 | `Assets/Art/Prefabs/FakeFloor/` |
| **P7** | **PressurePlate** | 自制（圆形 + 缩放动画）| 12-v2 §3.2 | 2h | ⚠️ 关键 | `Assets/Art/Prefabs/PressurePlate/` |
| **合计** | — | — | — | **14h** | — | — |

> **关键决策：** P1-P4 用 Kenney 资源 + 自制调色（节省 4h 自制工时），P5-P7 关键预制件（视觉欺骗核心）必须自制。

### 3.2 7 预制件 × 4 状态规格表

> 与 `02-core-mechanics-v2.md` §3 预制件类型 对齐。

| 预制件 | normal 状态 | hover 状态 | switching 状态 | disabled 状态 | 章节变化 |
|--------|------------|------------|---------------|---------------|---------|
| **SolidWall** | #3D3D5C 100% 不透明 + #1A1A2E 1px 描边 | — (不可交互) | — | — | 无 |
| **Floor** | #2D2D44 100% 不透明 + 无描边 | — | — | — | 无 |
| **Door** | 关闭 #5D4D2D + #FF9500 1px 描边 | 关闭 + 0.5 强度橙色光晕 | 切换 0.2s 淡入淡出 | #555555 50% + 锁图标 🔒 | Ch3 暗化（#3D2D1D）|
| **GlassWall** | rgba(0,212,255,0.3) + #00D4FF 1px 描边 | 100% 不透明 + 青色脉冲 | — | — | 无 |
| **CrumblingFloor** | #2D2D44 + #888888 1px 裂纹纹理 | — | 踩上 0.2s 缩放（1.0→0.9）| — | — |
| **FakeFloor** | #2D2D44（与 Floor 1:1 匹配）| — | 踩上 0.1s 后变 SolidWall + 闪烁红色 0.3s | — | — |
| **PressurePlate** | #FF9500 80% 圆形 + #FF9500 1px 描边 | 100% + 脉冲 | 踩下 0.2s 缩放（1.0→0.9→1.0）| — | 无 |

> **关键约束：** **FakeFloor 必须与 Floor 1:1 像素匹配**——这是"视觉欺骗"的核心约束（12-v2 §6.3.6）。

### 3.3 预制件文件命名规范

```
{prefab_id}_{state}_{chapter_id}.png

例：
- solidwall_normal.png            # SolidWall 默认状态
- door_closed.png                  # Door 关闭
- door_open.png                    # Door 开启
- glasswall_normal.png             # GlassWall 默认
- crumblingfloor_idle.png         # CrumblingFloor 待机
- crumblingfloor_cracking.png     # CrumblingFloor 碎裂中
- fakefloor_normal.png            # FakeFloor（与 floor_normal.png 1:1）
- pressureplate_normal.png        # PressurePlate 默认
- pressureplate_pressed.png       # PressurePlate 按下
```

### 3.4 Sprite Atlas 配置

| 字段 | 值 | 备注 |
|------|-----|------|
| **Atlas 名称** | `PrefabsAtlas` | 所有 7 预制件 + 4 状态打包 |
| **最大尺寸** | 2048 × 2048 | 适配 7 平台（详见 style-guide.md §6）|
| **格式** | PNG 32-bit | 透明背景 |
| **压缩** | LZ4 (默认) | 性能优先 |
| **打包工具** | Unity Sprite Atlas + Addressables | architecture module-breakdown M13 |

## 4. 玩家精灵 + 出口指示资源清单 (Player + Exit Indicators)

> 与 `12-art-style-v2.md` §5 角色设计 对齐。

### 4.1 玩家精灵 (Player Sprite)

| 状态 | 视觉 | 尺寸 | 动画 | 工时 |
|------|------|------|------|:---:|
| **Idle** | #00D4FF 青色圆形 + 0.5 格径向渐变光晕 | 12/16/20px (3 档对应字号) | 0.5s 呼吸缩放（1.0→1.05→1.0）| 1h |
| **Walk N/S/E/W** | 同 Idle | 12/16/20px | 0.1s 拉伸（X+1.1, Y-0.95）| 1h |
| **Walk Idle 过渡** | 同 Idle | 12/16/20px | 0.2s 渐变 | 0.5h |
| **合计** | — | — | — | **2.5h** |

> **关键设计：** 玩家精灵**始终是单一形状**（无方向），仅 Idle / Walk 2 状态。3 档尺寸对应字号档位（12-v2 §5.3 + 12-v2 §7.5）。

### 4.2 出口指示 (Exit Indicator)

| 状态 | 视觉 | 尺寸 | 动画 | 工时 |
|------|------|------|------|:---:|
| **未连通** | #555555 暗灰方块 | 32px × 32px | 静态 | 0.5h |
| **连通** | #FF9500 橙色方块 | 32px × 32px | 1.0s 周期脉冲（强度 1.0→1.5→1.0）| 1h |
| **已通关** | 白光闪烁 | 32px × 32px | 0.5s 闪烁 + 屏幕渐白 0.5s | 0.5h |
| **合计** | — | — | — | **2h** |

### 4.3 角色尺寸层级

| 元素 | 尺寸 | 视觉层级 |
|------|------|---------|
| 玩家精灵 | 12/16/20px | 焦点（最显眼）|
| 槽位 | 24px × 24px | 中等 |
| 出口 | 32px × 32px | 次要 |
| 预制件（SolidWall/Floor/Door/GlassWall）| 32px × 32px | 背景 |
| Door | 32px × 64px | 双格高 |

> **设计意图：** 玩家 < 槽位 < 出口 < 墙 = 视觉层级清晰，玩家永远是焦点。

## 5. 19 房间美术资源清单 (19 Room Assets)

> 与 `03-level-design-v2.md` §5 19 房间配置表 严格一一映射。

### 5.1 19 房间美术资源映射表

| 房间 ID | 房间名 | 类型 | 章节主题色 | 特殊美术 | 关键预制件 | 槽位图标准备 | 章节 | 工时 |
|--------|--------|------|----------|---------|----------|------------|------|:---:|
| **1-1** | 第一道光 | 教学 | #1A1A2E Ch1 | 明亮 + 暖色光斑 | Floor + SolidWall | TS | Ch1 | 2h |
| **1-2** | 双门 | 教学 | #1A1A2E Ch1 | 暖色 + 双发光源 | Floor + Door + SolidWall | 2×TS | Ch1 | 3h |
| **1-3** | 出口方向 | 教学 | #1A1A2E Ch1 | 暖色 + 出口光显著 | Floor + Door + SolidWall | 2×TS + 1×CS | Ch1 | 3h |
| **1-4** | 回顾 | 教学 | #1A1A2E Ch1 | 节奏减弱（光强 -10%）| Floor + SolidWall | 1×TS + 1×CS | Ch1 | 3h |
| **1-5** | 觉醒 | 教学 | #1A1A2E Ch1 | 章节高潮（光强 +20% 短暂）| Floor + Door + SolidWall | 2×TS + 1×CS | Ch1 | 3h |
| **2-1** | 入门 | 标准 | #15152A Ch2 | 局部动态光（条件光）| Floor + Door + GlassWall + SolidWall | 2×TS + 1×CS + 1×CDS | Ch2 | 3h |
| **2-2** | 顺序 | 标准 | #15152A Ch2 | 顺序依赖光链 | Floor + Door + SolidWall | 2×TS + 2×CS + 1×CDS | Ch2 | 3h |
| **2-3** | 锁链 | 标准 | #15152A Ch2 | 锁链光（多个光点串联）| Floor + Door + SolidWall | 3×TS + 1×CS + 1×CDS | Ch2 | 3h |
| **2-4** | 门控 | 标准 | #15152A Ch2 | 门控光（Door 关闭时亮）| Floor + Door + SolidWall + GlassWall | 2×TS + 2×CS + 1×CDS | Ch2 | 3h |
| **2-5** | 复合 | 标准 | #15152A Ch2 | 复合光（多源）| Floor + Door + SolidWall + GlassWall | 3×TS + 2×CS + 1×CDS | Ch2 | 4h |
| **2-6** | 沉静 | 教学 | #15152A Ch2 | 节奏减弱（光强 -20%）| Floor + Door + SolidWall | 2×TS + 1×CS + 1×CDS | Ch2 | 3h |
| **3-1** | 入口 | 标准 | #0E0E1F Ch3 | 极暗 + 复习光 | Floor + Door + SolidWall | 3×TS + 2×CS + 1×CDS | Ch3 | 3h |
| **3-2** | 双链 | 标准 | #0E0E1F Ch3 | 双向 CDS 联动光 | Floor + Door + SolidWall + GlassWall | 3×TS + 2×CS + 2×CDS | Ch3 | 4h |
| **3-3** | 错位 | 标准 | #0E0E1F Ch3 | 极暗 + 视觉欺骗（3-3 首次 FakeFloor）| **FakeFloor** + Floor + SolidWall | 3×TS + 2×CS + 2×CDS | Ch3 | 4h |
| **3-4** | 镜像 | 挑战 | #0E0E1F Ch3 | 镜像光（对称布局）| Floor + Door + SolidWall + GlassWall | 4×TS + 2×CS + 2×CDS | Ch3 | 4h |
| **3-5** | 伪装 | 挑战 | #0E0E1F Ch3 | 伪装 + CrumblingFloor | **CrumblingFloor** + **FakeFloor** + Floor + SolidWall + PressurePlate | 4×TS + 2×CS + 2×CDS + 1×LS | Ch3 | 5h |
| **3-6** | 迷宫 | 挑战 | #0E0E1F Ch3 | 迷宫（多路径）| Floor + Door + SolidWall + GlassWall + PressurePlate | 4×TS + 2×CS + 2×CDS + 1×LS | Ch3 | 5h |
| **3-7** | 终章·上 | Boss | #0E0E1F Ch3 极暗 | Boss 房上（光强 0.25）| Floor + Door + SolidWall + GlassWall + PressurePlate | 4×TS + 2×CS + 2×CDS + 1×LS | Ch3 | 5h |
| **3-8** | 终章·下 | Boss | #0E0E1F Ch3 极暗 | Boss 房下（光强 0.2 + 独立 BGM）| Floor + Door + SolidWall + GlassWall + PressurePlate + **FakeFloor** | 4×TS + 2×CS + 2×CDS + 1×LS | Ch3 | 6h |
| **合计** | — | — | — | — | — | — | — | **65h** |

### 5.2 房间美术资源文件命名

```
Rooms/{chapter_id}/{room_id}/
├── tilemap_floor.png            # 地板 Tilemap
├── tilemap_wall.png             # 墙 Tilemap
├── tilemap_decoration.png       # 装饰（Cracking/Fake indicator）
├── slot_positions.json          # 槽位坐标
├── lighting.json                # 章节光强 + 雾效
└── palette.json                 # 章节调色（章节色 + 强调色）
```

### 5.3 19 房间美术总工时

| 章节 | 房间数 | 工时 |
|------|:----:|:---:|
| Ch1 | 5 | 14h |
| Ch2 | 6 | 19h |
| Ch3 | 8 | 32h |
| **合计** | **19** | **65h** |

> **设计意图：** Ch3 视觉欺骗房 (3-3/3-4/3-5/3-6/3-7/3-8) 工时较高，因为包含 CrumblingFloor/FakeFloor/PressurePlate 关键预制件 + 镜像光 + 极暗光强。

## 6. 6 美术工具链资源清单 (6 Tool Chain Assets)

> 与 `12-art-style-v2.md` §9.5 美术工具链 对齐，总成本 ≤ $20。

| # | 工具 | 用途 | 版本 | 成本 | 来源 | 安装路径 |
|---|------|------|------|:----:|------|---------|
| **T1** | **Aseprite** | 2D 像素/矢量绘图 | 1.3.x | $20 (一次性) | Steam / 官网 | `tools/aseprite/` |
| **T2** | **Inkscape** | 2D 矢量绘图（备选） | 1.3.x | $0 (开源) | inkscape.org | `tools/inkscape/` |
| **T3** | **Unity Tilemap Editor** | Unity Tilemap 编辑（内置） | Unity 2022 LTS | $0 | Unity Hub | `Unity 内置` |
| **T4** | **Unity Particle System** | 粒子效果（内置） | Unity 2022 LTS | $0 | Unity Hub | `Unity 内置` |
| **T5** | **Unity 2D Light + URP 2D Renderer** | 2D 光照 | Unity 2022 LTS + URP | $0 | Unity Hub | `Unity 内置` |
| **T6** | **DOTween** | 动画（12 原则）| 1.2.x | $0 (MIT) | Unity Asset Store | `Assets/Plugins/DOTween/` |
| **T7** | **Sim Daltonism** | 色盲模拟（Mac）| 1.5.x | $0 (开源) | GitHub | `tools/sim-daltonism/` |
| **T8** | **Coblis** | 色盲模拟（Web）| — | $0 | coblis.colorblindness.com | 在线工具 |
| **T9** | **OBS Studio** | 屏幕录制（宣传视频）| 30.x | $0 (开源) | obsproject.com | `tools/obs/` |
| **T10** | **Addressable Assets** | 资源管理 | Unity 2022 LTS 内置 | $0 | Unity Hub | `Unity 内置` |
| **T11** | **DaVinci Resolve** | 视频剪辑（1 分钟花絮）| 18.x | $0 (开源) | blackmagicdesign.com | `tools/davinci/` |
| **T12** | **Audacity** | 音频编辑（与音频同步）| 3.x | $0 (开源) | audacityteam.org | `tools/audacity/` |
| **总成本** | — | — | — | **$20** | — | — |

> **关键决策：** 美术工具链总成本 $20（Aseprite 一次性），符合 1 人 Solo 预算。

## 7. 9 张数字画集清单 (9 Digital Artsets - Deluxe DLC)

> 与 `11-release-v2.md` §2.1 豪华版 $7.99 + `12-art-style-v2.md` §9.4 9 张数字画集 对齐。

| # | 画集 | 内容 | 房间 | 工时 | 章节 | 文件路径 |
|---|------|------|------|:---:|------|---------|
| **A1** | Ch1 第一道光 | 1-1 房间美术图 | 1-1 | 1.5h | Ch1 | `Assets/Art/DigitalArt/Ch1/awakening-01.png` |
| **A2** | Ch1 出口方向 | 1-3 房间美术图 | 1-3 | 1.5h | Ch1 | `Assets/Art/DigitalArt/Ch1/awakening-02.png` |
| **A3** | Ch1 觉醒 | 1-5 章节高潮美术图 | 1-5 | 1h | Ch1 | `Assets/Art/DigitalArt/Ch1/awakening-03.png` |
| **A4** | Ch2 入门 | 2-1 房间美术图 | 2-1 | 1.5h | Ch2 | `Assets/Art/DigitalArt/Ch2/deepdig-01.png` |
| **A5** | Ch2 门控 | 2-4 房间美术图 | 2-4 | 1.5h | Ch2 | `Assets/Art/DigitalArt/Ch2/deepdig-02.png` |
| **A6** | Ch2 复合 | 2-5 复合光美术图 | 2-5 | 1h | Ch2 | `Assets/Art/DigitalArt/Ch2/deepdig-03.png` |
| **A7** | Ch3 错位 | 3-3 视觉欺骗入门美术图 | 3-3 | 2h | Ch3 | `Assets/Art/DigitalArt/Ch3/lostpath-01.png` |
| **A8** | Ch3 伪装 | 3-5 CrumblingFloor + FakeFloor 美术图 | 3-5 | 2h | Ch3 | `Assets/Art/DigitalArt/Ch3/lostpath-02.png` |
| **A9** | Ch3 终章·下 | 3-8 Boss 房终极美术图 | 3-8 | 2h | Ch3 | `Assets/Art/DigitalArt/Ch3/lostpath-03.png` |
| **合计** | — | — | — | **14h** | — | — |

### 7.1 画集规格

| 字段 | 值 |
|------|-----|
| **分辨率** | 1920 × 1080 (1080p) |
| **格式** | PNG 32-bit |
| **风格** | 静态高清渲染 + 暗色调 + 极简元素 |
| **交付物** | 9 PNG + 1 PDF 合集（豪华版 DLC 打包）|
| **DLC 包装** | Steam 豪华版 $7.99 + Itch.io Pay What You Want |

### 7.2 数字画集营销绑定

| 物料 | 数量 | 工时 | 优先级 | 来源 |
|------|:----:|:----:|:----:|------|
| **9 张数字画集** | 9 | 14h | P1 | 11-v2 §4.3 |
| **1 分钟制作花絮** | 1 | 8h | P1 | 11-v2 §4.3 |
| **早期 OST 9 首** | 9 | 4h (借用 09-v2) | P1 | 11-v2 §4.3 |
| **豪华版合计** | — | **26h** | — | 11-v2 §4.3 |

## 8. 4 字体授权清单 (4 Font Licensing)

> 与 `12-art-style-v2.md` §7.4 字体规范 + `08-ui-ux-v2.md` §9.2 本地化 对齐。

| # | 字体 | 语种 | 用途 | 授权类型 | 来源 | 备注 |
|---|------|------|------|---------|------|------|
| **F1** | **Inter (Variable)** | en-US | 英文 UI | OFL 1.1 (免费商用) | Google Fonts | https://fonts.google.com/specimen/Inter |
| **F2** | **Noto Sans CJK SC** | zh-CN | 简体中文 UI | OFL 1.1 (免费商用) | Google Fonts | https://fonts.google.com/noto/specimen/Noto+Sans+SC |
| **F3** | **Noto Sans CJK TC** | zh-TW | 繁体中文 UI | OFL 1.1 (免费商用) | Google Fonts | https://fonts.google.com/noto/specimen/Noto+Sans+TC |
| **F4** | **Noto Sans JP** | ja-JP | 日文 UI | OFL 1.1 (免费商用) | Google Fonts | https://fonts.google.com/noto/specimen/Noto+Sans+JP |
| **F5** | **Noto Sans KR** | ko-KR | 韩文 UI | OFL 1.1 (免费商用) | Google Fonts | https://fonts.google.com/noto/specimen/Noto+Sans+KR |
| **合计** | — | 5 语种 | — | **$0** | — | — |

> **v1.0 仅启用 F1 + F2 (中英)**，F3/F4/F5 在 v1.1 启用（与 10-v2 §12 本地化里程碑 + 11-v2 §12.2 v1.1 5 语种 对齐）。

### 8.1 字体文件路径

```
Assets/Fonts/
├── Inter-Variable.ttf             # 英文
├── NotoSansSC-Variable.ttf        # 简体中文
├── NotoSansTC-Variable.ttf        # 繁体中文 (v1.1)
├── NotoSansJP-Variable.ttf        # 日文 (v1.1)
└── NotoSansKR-Variable.ttf        # 韩文 (v1.1)
```

### 8.2 字体规格

| 字段 | 值 |
|------|-----|
| **字重** | Regular (400) / Medium (500) / Bold (700) |
| **Variable 轴** | wght (100-900) |
| **抗锯齿** | Grayscale 8x |
| **渲染** | TextMeshPro SDF (高 DPI 清晰) |
| **字号** | 12-72px (详见 style-guide.md §6.3 字号缩放 3 档) |

## 9. 资源打包与版本控制 (Asset Packaging & Version Control)

### 9.1 Addressables 分组

> 与 `design/architecture/tech-stack.md` Addressables + `architecture module-breakdown M13 AssetPipeline` 对齐。

| Addressable Group | 包含资源 | 加载策略 | 平台适配 |
|-------------------|---------|---------|---------|
| **PrefabsAtlas** | 7 预制件 × 4 状态 = ~28 sprite | Preload (启动加载) | 全平台 |
| **RoomArt_{chapter}** | 章节房间 Tilemap | Async (进入章节时加载) | 全平台 |
| **ChapterThemes** | 3 章节主题（背景色 + 雾效 + 光强）| Preload | 全平台 |
| **HUD** | HUD 极简半透明面板 | Preload | 全平台 |
| **DigitalArt_Deluxe** | 9 张数字画集 (豪华版 DLC) | On Demand (玩家购买豪华版后下载) | 全平台 |
| **Fonts** | 字体 5 语种 | Preload (按当前语言切换) | 全平台 |
| **Marketing** | 5 截图 + 1 视频 + 30 秒预告 | Streaming (Steam 商店页外部托管) | 全平台 |

### 9.2 Git LFS 策略

| 资源类型 | Git LFS? | 理由 |
|---------|:--------:|------|
| **PNG/Sprite** | ✅ | 二进制文件 + 体积大 |
| **MP4 视频** | ✅ | 二进制文件 + 体积大 |
| **JSON 关卡数据** | ❌ | 文本文件 |
| **MD 文档** | ❌ | 文本文件 |
| **字体 TTF** | ⚠️ 大文件 | OFL 1.1 协议允许二次分发但建议 Git LFS |

### 9.3 资源命名规范

```
{类型}_{内容}_{状态}_{章节}.{扩展名}

例：
- prefab_solidwall_normal_ch1.png
- prefab_door_closed_ch2.png
- prefab_crumblingfloor_idle_ch3.png
- room_1-1_tilemap_floor.png
- art_awakening-01.png
- font_inter_variable.ttf
```

## 10. 验收标准 (Acceptance Criteria)

- [x] **AC-01** Frontmatter 7 字段完整（title / doc_id / parent / last_updated / version / status / owner）
- [x] **AC-02** 6 必填通用章节（目的 / 范围 / 配置表 / 边界条件 / 验收标准 / 风险与开放问题）
- [x] **AC-03** 3 阶段美术资源台账（原型 6.5h / Kenney 49h / 自制 38h / RC 44h = 137.5h）
- [x] **AC-04** 7 预制件 × 4 状态资源清单（P1-P4 Kenney 调色 + P5-P7 自制）
- [x] **AC-05** 玩家精灵 2.5h + 出口指示 2h（共 4.5h）
- [x] **AC-06** 19 房间美术资源映射表（与 03-v2 §5 严格一一映射，总工时 65h）
- [x] **AC-07** 6 美术工具链（T1-T12，总成本 $20）
- [x] **AC-08** 9 张数字画集豪华版（与 11-v2 §2.1 + 12-v2 §9.4 对齐，14h）
- [x] **AC-09** 4 字体授权清单（OFL 1.1 全部免费）
- [x] **AC-10** Addressables 分组 + Git LFS 策略 + 资源命名规范
- [x] **AC-11** P0-001 跟踪（与 README §7 对齐）

## 11. 边界条件 (Edge Cases)

| # | 触发条件 | 预期行为 |
|---|---------|---------|
| **E1** | Kenney 资源链接失效 / 网站关闭 | 启动前在 `Backup/` 目录保留 Kenney 2D Pack v1.0 完整快照 |
| **E2** | 自制 CrumblingFloor 碎裂粒子在低端机掉帧 | 限制粒子数 ≤ 10 + 短生命周期 0.3s |
| **E3** | FakeFloor 与 Floor 1:1 像素匹配偏差 > 1px | Playtest 5 人中 ≥ 4 人能识别时返工 |
| **E4** | 字体文件损坏（TFF 解析失败） | TextMeshPro 降级为系统默认字体 + 提示玩家 |
| **E5** | 9 张数字画集 14h 工时紧 | 推迟 3 张 Ch3 画集到 v1.0.1 + 复用 19 房间截图 |
| **E6** | 1 人 Solo 自制 7 预制件能力不足 | P1-P4 用 Kenney 调色（节省 4h）+ P5-P7 关键自制 |
| **E7** | 美术资源总工时超 137.5h | 启用 P2 推迟清单（详见 asset-budget.md §5）|
| **E8** | Addressables 打包后单平台超出预算 | 按平台分级压缩（详见 style-guide.md §6 7 平台）|

## 12. 配置表 (Configuration)

| 字段 | 类型 | 取值范围 | 默认值 | 备注 |
|------|------|---------|-------|------|
| `asset.prototype.hours` | float | [4, 12] | 6.5 | 阶段 0 工时 |
| `asset.alpha.hours` | float | [40, 80] | 49 | 阶段 1 工时 |
| `asset.beta.hours` | float | [30, 60] | 38 | 阶段 2 工时 |
| `asset.release.hours` | float | [30, 60] | 44 | 阶段 3 工时 |
| `asset.total.hours` | float | [120, 200] | 137.5 | 总工时 |
| `asset.total.usd` | float | [0, 50] | 20 | 总成本 (Aseprite 一次性) |
| `prefab.count` | int | [7, 7] | 7 | 预制件数量（固定） |
| `prefab.states` | int | [4, 4] | 4 | 预制件状态数 |
| `room.count` | int | [19, 19] | 19 | 房间数（固定） |
| `room.artHours` | float | [60, 80] | 65 | 19 房间美术总工时 |
| `digitalArt.count` | int | [9, 9] | 9 | 数字画集数（豪华版）|
| `digitalArt.hours` | float | [12, 20] | 14 | 画集工时 |
| `tool.totalUsd` | float | [0, 50] | 20 | 工具链总成本 |
| `tool.count` | int | [10, 15] | 12 | 工具链数量 |
| `font.count` | int | [1, 5] | 2 | v1.0 字体数（中英）|

## 13. 关联文档

### 上游（本文档依赖）

- [`README.md`](./README.md) — 总览 + 8 文件索引
- [`docs/12-art-style-v2.md`](../../docs/12-art-style-v2.md) — 美术规格基线（调色板/光影/资源方案）
- [`docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — 7 预制件类型
- [`docs/03-level-design-v2.md`](../../docs/03-level-design-v2.md) — 19 房间配置
- [`docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — 12 里程碑
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台 + 9 画集豪华版
- [`design/api/data-models.md`](../api/data-models.md) — M05 Chapter + AccessibilitySettings
- [`design/architecture/module-breakdown.md`](../architecture/module-breakdown.md) — M13 AssetPipeline

### 下游（本文档被依赖）

- [`production-pipeline.md`](./production-pipeline.md) — 引用本文档资源清单制定流水线
- [`outsourcing.md`](./outsourcing.md) — 引用本文档资源清单制定外包策略
- [`copyright.md`](./copyright.md) — 引用本文档资源清单制定版权管理
- [`asset-budget.md`](./asset-budget.md) — 引用本文档工时/成本数据制定预算
- [`delivery-checklist.md`](./delivery-checklist.md) — 引用本文档资源清单制定验收标准

## 14. 关联代码模块

| 模块 | 路径 | 状态 | 职责 |
|------|------|------|------|
| **AssetPipeline** | `src/AssetPipeline/` | 待创建 | Addressables + Sprite Atlas 打包 |
| **7 预制件视觉** | `src/Prefabs/SolidWall.cs` / `Floor.cs` / `Door.cs` / `GlassWall.cs` / `CrumblingFloor.cs` / `FakeFloor.cs` / `PressurePlate.cs` | 待创建 | 7 预制件视觉契约 |
| **FakeFloor 视觉欺骗** | `src/Art/FakeFloorVisualDeception.cs` | 待创建 | 1:1 像素匹配 |
| **CrumblingFloor 碎裂** | `src/Art/CrumblingFloorAnimator.cs` | 待创建 | 0.5s 延迟 + 粒子 |
| **压力板动画** | `src/Art/PressurePlateAnimator.cs` | 待创建 | 踩下缩放 |
| **玩家精灵动画** | `src/Art/PlayerSpriteAnimator.cs` | 待创建 | 移动拉伸 + Idle 呼吸 |
| **出口指示器** | `src/Art/ExitIndicator.cs` | 待创建 | 连通/未连通 2 态 |
| **Kenney 资源适配** | `src/Art/KenneyAssetAdapter.cs` | 待创建 | Kenney CC0 调色 |

## 15. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| **R-01** | **P0-001**（02-v2 §13 AC-06 缺"难度上限 20"）| 中 | 100% | asset-list 与 P0-001 **弱依赖**（仅引用 Boss 房光强假设）；不阻塞 v1.0 | **OPEN（弱依赖）** |
| **R-02** | **1 人 Solo 自制 7 预制件能力不足** | 高 | 60% | P1-P4 Kenney 调色（节省 4h）+ P5-P7 关键自制 | 已规划 |
| **R-03** | **Kenney CC0 资源版权变更** | 低 | 10% | 启动前 Backup 完整快照 + 自制备份 7 预制件 | 已规划 |
| **R-04** | **CrumblingFloor 碎裂粒子占用 GPU 过高** | 低 | 15% | 限制粒子数 ≤ 10 + 短生命周期 0.3s | 已规划 |
| **R-05** | **9 张数字画集 14h 工时紧** | 中 | 50% | 推迟 3 张 Ch3 画集到 v1.0.1 | 已规划 |
| **R-06** | **19 房间美术 65h 工时** 与 10-v2 M09 美术里程碑 24h 不足 | 中 | 70% | 复用 Kenney 资源降级（节省 30h）| 已规划 |
| **R-07** | **工具链 $20 一次性成本** 与 10-v2 美术预算 $0 略有冲突 | 低 | 100% | 10-v2 已允许"美术素材 ≤ $200"范围 | 已规划 |
| **R-08** | **Addressables 打包后平台超预算** | 中 | 30% | 按平台分级压缩（详见 style-guide.md §6）| 待验证 |
| **Q-01** | **是否做 4K 数字画集**（豪华版 DLC 升级）？ | 低 | — | v1.0 仅 1080p；v1.1 评估 4K | 倾向 v1.0 1080p |
| **Q-02** | **是否提供玩家可换皮肤**？ | 低 | — | v1.0 不支持；v1.1 评估 | 倾向不做 |
| **Q-03** | **是否做章节 BGM 节拍同步光斑**？ | 低 | — | v1.0 独立；v1.1 评估 | 倾向不做 |

## 16. 待办事项 (TODO)

- [ ] **P0：** 实现 AssetPipeline 模块（Addressables + Sprite Atlas）— 阻塞所有美术集成 [README §11]
- [ ] **P0：** 7 预制件视觉契约实现 — 阻塞 19 房间实施 [本文 §3]
- [ ] **P0：** 19 房间美术资源（与 03-v2 §5 一一映射）— 阻塞游戏可玩 [本文 §5]
- [ ] **P0：** FakeFloor 1:1 像素匹配测试 — 阻塞 Ch3 3-3 [本文 §3.1 P6]
- [ ] **P1：** 9 张数字画集（豪华版 DLC）— v1.0 后 [本文 §7]
- [ ] **P1：** 1 分钟制作花絮 + Steam 5 截图 — 阻塞 M10/M11 [本文 §2.4]
- [ ] **P1：** Addressables 平台分级压缩 — 不阻塞 v1.0 [本文 §9.1]
- [ ] **P2：** Kenney 资源 Backup 快照 — 不阻塞 v1.0 [本文 §11 E1]
- [ ] **P2：** 评估 4K 数字画集（豪华版升级）— v1.1 [Q-01]
- [ ] **P2：** 解决 P0-001（02-v2 §13 AC-06 增补"难度上限 20"）— phase3 [本文 §15 R-01]

## 17. 评审迭代记录

| 轮 | 版本 | 时间 | 总分 | P0 | P1 | P2 | P3 | 备注 |
|---|------|------|:----:|---|---|---|---|------|
| 1 | v1.0 | 2026-06-29 | — | — | — | — | — | **本次初版:** 3 阶段美术资源台账（137.5h） / 7 预制件 × 4 状态资源清单（14h 自制） / 玩家精灵 + 出口指示（4.5h） / 19 房间美术资源映射（65h） / 6 工具链（$20） / 9 张数字画集豪华版（14h） / 4 字体授权（OFL 1.1 免费） / Addressables 分组 + Git LFS + 命名规范 / P0-001 弱依赖跟踪 |

## 18. 变更日志

| 日期 | 版本 | 变更人 | 内容 |
|------|------|--------|------|
| 2026-06-29 | v1.0 | 中书省 subagent | **ANZHONG-14 phase3 asset-list 创建:** 3 阶段美术资源台账（137.5h + $20） / 7 预制件 × 4 状态资源清单（P1-P4 Kenney 调色 + P5-P7 关键自制 14h） / 玩家精灵（2.5h）+ 出口指示（2h） / 19 房间美术资源映射（Ch1 14h / Ch2 19h / Ch3 32h = 65h） / 12 工具链（T1-T12，$20 一次性） / 9 张数字画集豪华版（Ch1 4h / Ch2 4h / Ch3 6h = 14h） / 4 字体授权（OFL 1.1 免费） / Addressables 7 分组 + Git LFS + 资源命名规范 / P0-001 弱依赖跟踪 / 8 边界条件 / 15 配置字段 / 8 关联代码模块 / 7 风险 + 3 开放问题 / 10 待办事项 P0×4 P1×3 P2×3 / 整改 AUDIT-REPORT §2.art 全部 P0 整改项 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）
**P0-001 跟踪：** OPEN — 与 art 设计**弱依赖**（仅 3 处间接引用），不阻塞 v1.0 实施