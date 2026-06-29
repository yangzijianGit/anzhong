---
title: 美术风格指南
doc_id: DES-anzhong-art-style-guide
parent: design/art/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 美术总监（中书省 subagent）
---

# 《暗室》美术风格指南（style-guide.md）

> **一句话定位：** 引用 `12-art-style-v2.md` §3 §4 §7 §8 的可执行风格手册 — 12 调色板 + 7 预制件视觉契约 + 3 章节主题 + 5 字体 + 3 字号 + 12 动画原则 + 5 缓动函数 + 色盲 3 档。

## 目的 (Purpose)

本文档是《暗室》美术层的**风格实施速查手册**。它向美术总监（中书省）、Unity 工程师、外包合作伙伴、未来的维护者**用 15 分钟讲清**：

- **12 主色调色板**（12-v2 §3.1 引用）
- **7 预制件视觉契约**（12-v2 §6.3 引用）
- **3 章节主题色温 + 光强 + 雾效**（12-v2 §4.4 引用）
- **5 字体规范**（12-v2 §7.4 引用）
- **3 字号缩放档**（12-v2 §7.5 引用）
- **12 动画原则 + 5 缓动函数**（12-v2 §8 引用）
- **3 档色盲替代色**（12-v2 §3.3 引用）
- **7 平台美术规格**（11-v2 §1.1 引用）

**本文与 `12-art-style-v2.md` 的边界：** 12-v2 是**规格定义**（什么是"对的"），本文档是**实施速查**（怎么"用对"）。

## 范围 (Scope)

### 包含

- **12 主色调色板 + 7 预制件调色板**（12-v2 §3 引用 + 实施）
- **3 章节主题 + 光强曲线 + 雾效**（12-v2 §4 引用）
- **5 字体规范 + 3 字号缩放档**（12-v2 §7 引用）
- **12 动画原则 + 5 缓动函数**（12-v2 §8 引用）
- **3 档色盲替代色**（12-v2 §3.3 引用）
- **7 平台美术规格**（11-v2 §1.1 引用）
- **风格一致性检查清单**（实施时核对）

### 不包含 (Out of Scope)

- 美术规格**定义** → 见 `docs/12-art-style-v2.md` §3 §4 §7 §8（权威源）
- 美术资源清单 → 见 `asset-list.md`
- 美术制作流程 → 见 `production-pipeline.md`
- 美术版权 → 见 `copyright.md`
- 美术预算 → 见 `asset-budget.md`

## 1. 一句话描述 (One-liner)

> **"Dead Cells × Gorogoa × 废弃设施的 2D 极简半透明风格 — 12 主色 + 7 预制件 + 3 章节 + 5 字体 + 3 字号 + 12 动画 + 色盲 3 档。"**

## 2. 12 主色调色板 (12 Master Color Palette)

> 引用 `12-art-style-v2.md` §3.1（权威定义），本文给出**实施速查 + 用途**。

### 2.1 12 主色速查表

| # | 类别 | 色名 | 色值 | 用途 | 章节 | RGB |
|---|------|------|------|------|------|-----|
| **C1** | 背景 | 深石板蓝 | #1A1A2E | Ch1 背景 / 通用深色 | Ch1 | (26, 26, 46) |
| **C2** | 背景 | 深紫蓝 | #15152A | Ch2 背景 | Ch2 | (21, 21, 42) |
| **C3** | 背景 | 暗夜蓝 | #0E0E1F | Ch3 背景 | Ch3 | (14, 14, 31) |
| **C4** | 中性 | 石墨灰 | #2D2D44 | 地板（Floor）| 全部 | (45, 45, 68) |
| **C5** | 中性 | 雾灰 | #3D3D5C | 实墙（SolidWall）| 全部 | (61, 61, 92) |
| **C6** | 强调 | 青色 | #00D4FF | 槽位发光 / 主强调色 | 全部 | (0, 212, 255) |
| **C7** | 强调 | 橙色 | #FF9500 | 出口发光 / 暖色强调 | 全部 | (255, 149, 0) |
| **C8** | 文字 | 浅灰 | #E0E0E0 | 主文字 / UI | 全部 | (224, 224, 224) |
| **C9** | 状态 | 暗灰 | #555555 | 未激活 / Locked | 全部 | (85, 85, 85) |
| **C10** | 状态 | 暗红 | #FF4444 | 警告 / 错音 / FakeFloor 闪烁 | 全部 | (255, 68, 68) |
| **C11** | 状态 | 暗金 | #FFD700 | 章节完成 / 成就 | 全部 | (255, 215, 0) |
| **C12** | 状态 | 透明黑 | rgba(0,0,0,0.6) | UI 半透明面板 | 全部 | (0, 0, 0, α=0.6) |

> **关键设计：** 调色板**不变**（19 房间一致），**仅章节背景色温 + 光强 + 雾效**变化（章节级差异）。

### 2.2 Unity 调色板配置

```csharp
// src/Art/PaletteManager.cs
public static class PaletteManager {
    public static readonly Color BgCh1 = new Color(0x1A/255f, 0x1A/255f, 0x2E/255f, 1f); // #1A1A2E
    public static readonly Color BgCh2 = new Color(0x15/255f, 0x15/255f, 0x2A/255f, 1f); // #15152A
    public static readonly Color BgCh3 = new Color(0x0E/255f, 0x0E/255f, 0x1F/255f, 1f); // #0E0E1F
    public static readonly Color Floor = new Color(0x2D/255f, 0x2D/255f, 0x44/255f, 1f); // #2D2D44
    public static readonly Color SolidWall = new Color(0x3D/255f, 0x3D/255f, 0x5C/255f, 1f); // #3D3D5C
    public static readonly Color SlotCyan = new Color(0x00/255f, 0xD4/255f, 0xFF/255f, 1f); // #00D4FF
    public static readonly Color ExitOrange = new Color(0xFF/255f, 0x95/255f, 0x00/255f, 1f); // #FF9500
    public static readonly Color TextPrimary = new Color(0xE0/255f, 0xE0/255f, 0xE0/255f, 1f); // #E0E0E0
    public static readonly Color Locked = new Color(0x55/255f, 0x55/255f, 0x55/255f, 1f); // #555555
    public static readonly Color Warning = new Color(0xFF/255f, 0x44/255f, 0x44/255f, 1f); // #FF4444
    public static readonly Color Achievement = new Color(0xFF/255f, 0xD7/255f, 0x00/255f, 1f); // #FFD700
    public static readonly Color UIPanelBg = new Color(0, 0, 0, 0.6f); // rgba(0,0,0,0.6)
}
```

### 2.3 色盲 3 档替代色 (Color Blind 3 Modes)

> 引用 `12-art-style-v2.md` §3.3

| 档位 | C6 槽位发光 | C7 出口发光 | C10 警告 | 强化对比 | 适用 |
|------|-----------|-----------|---------|---------|------|
| **正常 (Normal)** | #00D4FF 青色 | #FF9500 橙色 | #FF4444 红色 | 标准 | 正常视觉 |
| **红绿色盲 (Deuteranopia / Protanopia)** | #0099FF 蓝色 | #FFCC00 黄色 | #FF8800 深橙 | 蓝黄对比 | 8% 男性 |
| **全色盲 (Monochromacy)** | #FFFFFF 白色 + ▢ 图案 | #FFFF00 黄色 + ◇ 图案 | #CCCCCC 灰 + ⚠ 图案 | **图案 + 形状** | < 1% |

> **关键设计：** 全色盲**完全依赖图案 + 形状**——不仅是颜色替代。

## 3. 7 预制件视觉契约 (7 Prefab Visual Contracts)

> 引用 `12-art-style-v2.md` §6.3（权威定义）。

### 3.1 7 预制件视觉契约速查表

| # | 预制件 | 主色 | 描边 | 透明度 | 章节变化 | 视觉契约 |
|---|--------|------|------|:-----:|---------|---------|
| **P1** | **SolidWall** | #3D3D5C (C5) | #1A1A2E 1px | 100% | 无 | 不可通行的暗灰方块 |
| **P2** | **Floor** | #2D2D44 (C4) | 无 | 100% | 无 | 可通行的地板 |
| **P3** | **Door** | #5D4D2D（关） / #2D2D44（开）| #FF9500 1px（开）| 100% | Ch3 暗化 | 关闭阻挡，开启通道 |
| **P4** | **GlassWall** | rgba(0,212,255,0.3) | #00D4FF 1px | 30% | 无 | 半透明蓝，可通行但视觉阻挡 |
| **P5** | **CrumblingFloor** | #2D2D44 + 裂纹纹理 | #888888 1px | 100% | 视觉提示 | 0.5s 后碎裂消失 |
| **P6** | **FakeFloor** | **#2D2D44（与 Floor 完全相同）**| 无 | 100% | 视觉欺骗 | 1:1 像素匹配 Floor，踩上才暴露 |
| **P7** | **PressurePlate** | #FF9500 + 圆形 | #FF9500 1px | 80% | 无 | 踩下时触发联动事件 |

> **关键约束：** **FakeFloor 必须与 Floor 1:1 像素匹配**——这是"视觉欺骗"的核心约束（12-v2 §6.3.6）。

### 3.2 7 预制件详细视觉规格

#### 3.2.1 SolidWall (实墙)

```
视觉：深灰方块（#3D3D5C）+ 1px 描边（#1A1A2E）
尺寸：32px × 32px
光影：主光左侧 0.6 强度 / 右侧 0.3 强度（2D 色阶模拟）
动画：无（静态）
```

#### 3.2.2 Floor (地板)

```
视觉：石墨灰（#2D2D44）+ 无描边
尺寸：32px × 32px
光影：均匀（0.5 强度）
动画：无（静态）
```

#### 3.2.3 Door (机关门)

```
关闭状态：#5D4D2D（暖棕）+ 1px 橙色描边
开启状态：#2D2D44（与 Floor 相同）+ 0.5 强度青色光晕
过渡：0.2s 淡入淡出（与 02 §3.1 切换动画同步）
触发：ConditionalSlot 依赖满足时切换
```

#### 3.2.4 GlassWall (透明墙)

```
视觉：rgba(0,212,255,0.3) + 1px 青色描边
尺寸：32px × 32px
行为：可通行（isWalkable = true）+ 视觉阻挡（半透明）
用途：Ch2+ 视觉暗示（玩家看见但需绕路）
```

#### 3.2.5 CrumblingFloor (碎裂地板)

```
视觉：#2D2D44（与 Floor 相同）+ 裂纹纹理叠加
触发：玩家踩上 0.5s 后碎裂消失
动画：碎裂粒子（10 块小方块向下飞散 0.3s）
不可逆：重置后才恢复
```

#### 3.2.6 FakeFloor (伪装地板)

```
视觉：#2D2D44（与 Floor 完全 1:1 像素匹配）
触发：玩家踩上 → 0.1s 后变 SolidWall + 闪烁红色 0.3s + 错音
设计意图：1:1 像素匹配 = 玩家必须"试错"才能发现
学习机制：玩家通过视觉反馈理解"看起来一样但实际不同"
```

#### 3.2.7 PressurePlate (压力板)

```
视觉：橙色圆形（#FF9500 80% 透明度）+ 1px 橙色描边
触发：玩家踩下 → 激活联动事件（如解锁 LockedSlot）
动画：踩下 0.2s 缩放（1.0 → 0.9 → 1.0 表示"被踩"）
```

## 4. 3 章节主题 + 光强曲线 + 雾效 (3 Chapter Themes)

> 引用 `12-art-style-v2.md` §4.4 + §1.3 + §6.2。

### 4.1 3 章节主题色温

| 章节 | 名称 | 主色 | 关键色 | 场景元素 | 雾效 | 主光 |
|------|------|------|--------|---------|:---:|:---:|
| **Ch1 觉醒** | First Light | #1A1A2E 深石板蓝 (C1) | 暖色光斑 (C7 橙色) | 走廊 + 实验室 + 休息室 | 5% | 0.6 |
| **Ch2 深掘** | Deep Dig | #15152A 深紫蓝 (C2) | 局部动态光 | 地下 + 机械室 + 控制室 | 15% | 0.4 |
| **Ch3 迷途** | Lost Path | #0E0E1F 暗夜蓝 (C3) | 强光斑 | 镜像室 + 迷宫 + 终极室 | 30% | 0.3 |

### 4.2 章节光强曲线 (Chapter Light Intensity Curve)

```
Ch1 (1-1 ~ 1-5):  强度 0.6（明亮）
Ch2 (2-1 ~ 2-6):  强度 0.4（中等）
Ch3 (3-1 ~ 3-8):  强度 0.3（极暗，Boss 房 0.2）
```

> **关键设计：** 章节光强**单调递减**——玩家从 Ch1 到 Ch3 视觉上"越走越深"。

### 4.3 章节雾效 (Chapter Fog)

| 章节 | 雾效颜色 | 透明度 | 渐变方向 | 用途 |
|------|---------|:-----:|---------|------|
| **Ch1 觉醒** | #1A1A2E | 5% | 左→右 0%→5% | 营造轻度空间感 |
| **Ch2 深掘** | #15152A | 15% | 左→右 0%→15% | 营造压抑感 |
| **Ch3 迷途** | #0E0E1F | 30% | 左→右 0%→30% | 营造迷失感（看不清远处）|

### 4.4 19 房间视觉差异表

> 引用 `12-art-style-v2.md` §6.2。

| 房间 | 章节 | 视觉特殊点 | 难度 | 视觉密度 |
|------|------|----------|:----:|:-------:|
| **1-1** | Ch1 | 明亮 + 暖色光斑 | 2 | 极简 |
| **1-2** | Ch1 | 暖色 + 双发光源 | 3 | 简单 |
| **1-3** | Ch1 | 暖色 + 出口光显著 | 5 | 简单 |
| **1-4** | Ch1 | 节奏减弱（光强 -10%）| 3 | 简单（喘息房）|
| **1-5** | Ch1 | 章节高潮（光强 +20% 短暂）| 5 | 中等 |
| **2-1** | Ch2 | 局部动态光（条件光）| 7 | 中等 |
| **2-2** | Ch2 | 顺序依赖光链 | 10 | 中等 |
| **2-3** | Ch2 | 锁链光（多个光点串联）| 9 | 中等 |
| **2-4** | Ch2 | 门控光（Door 关闭时亮）| 12 | 中等 |
| **2-5** | Ch2 | 复合光（多源）| 16 | 复杂 |
| **2-6** | Ch2 | 节奏减弱（光强 -20%）| 8 | 中等（喘息房）|
| **3-1** | Ch3 | 极暗 + 复习光 | 13 | 复杂 |
| **3-2** | Ch3 | 双向 CDS 联动光 | 16 | 复杂 |
| **3-3** | Ch3 | 极暗 + 视觉欺骗（3-3 首次 FakeFloor）| 16 | 复杂 |
| **3-4** | Ch3 | 镜像光（对称布局）| 16 | 复杂 |
| **3-5** | Ch3 | 伪装 + CrumblingFloor | 16 | 极复杂 |
| **3-6** | Ch3 | 迷宫（多路径）| 20 | 极复杂 |
| **3-7** | Ch3 | Boss 房上（光强 0.25）| 20 | 极复杂 |
| **3-8** | Ch3 | Boss 房下（光强 0.2 + 独立 BGM）| 20 | 极复杂 |

### 4.5 视觉欺骗 3 类 (Visual Deception 3 Types)

> 引用 `12-art-style-v2.md` §6.4 + §10。

| # | 类型 | 房间 | 视觉实现 | 玩家学习路径 |
|---|------|------|---------|------------|
| **VD-1** | **FakeFloor（伪装地板）** | 3-3 / 3-4 | 1:1 像素匹配 Floor + 踩上闪烁红色 | 玩家试错 → 学习"看 = 试" |
| **VD-2** | **CrumblingFloor（碎裂地板）** | 3-5 / 3-6 | 与 Floor 视觉相似 + 裂纹纹理 + 0.5s 碎裂 | 玩家主动观察裂纹 → 学习"纹理 = 危险" |
| **VD-3** | **镜像（视觉对称 ≠ 逻辑对称）** | 3-4 | 房间镜像布局 + 槽位视觉对称但配置不同 | 玩家基于直觉推理 → 失败 → 学习"看 ≠ 对" |

> **关键设计：** 视觉欺骗**不惩罚**玩家（无失败状态），仅"闪烁 + 错音"作为反馈（与 07-v2 §7 失败反馈机制对齐）。

## 5. 角色设计 (Character Design)

> 引用 `12-art-style-v2.md` §5。

### 5.1 玩家精灵

| 维度 | 规格 | 备注 |
|------|------|------|
| **形状** | 圆形（直径 12px / 16px / 20px 三档对应 100% / 125% / 150% 字号）| 简化符号化 |
| **颜色** | #00D4FF 青色 (C6) | 与槽位发光同色（强调"我是探索者"）|
| **光晕** | 半径 0.5 格的径向渐变（#00D4FF 100% → 0%）| 玩家位置始终有微光 |
| **朝向** | 单一（无方向）| 2D 极简 |
| **动画** | 移动时 0.1s 拉伸（X+1.1, Y-0.95）| 简化跑步感 |
| **Idle 动画** | 0.5s 呼吸缩放（1.0 → 1.05 → 1.0）| "活着"的感觉 |

### 5.2 出口指示

| 状态 | 视觉 | 触发 |
|------|------|------|
| **未连通** | #555555 暗灰方块 (C9) + 0.5 强度 | 路径未连通 |
| **连通** | #FF9500 橙色方块 (C7) + 1.5 强度脉冲（1.0s 周期）| 路径连通 |
| **已通关** | 白光闪烁 0.5s + 屏幕渐白 0.5s | 玩家走到出口 |

### 5.3 角色尺寸层级

| 元素 | 尺寸 | 视觉层级 |
|------|------|---------|
| 玩家精灵 | 12/16/20px | 焦点（最显眼）|
| 槽位 | 24px × 24px | 中等 |
| 出口 | 32px × 32px | 次要 |
| 预制件（SolidWall/Floor/Door/GlassWall）| 32px × 32px | 背景 |
| Door | 32px × 64px | 双格高 |

## 6. UI 视觉规范 + 字体规范 + 字号缩放 (UI Visual Spec)

> 引用 `12-art-style-v2.md` §7。

### 6.1 HUD 极简半透明原则

```
┌─────────────────────────────────────────┐
│ Ch1-3/5                            ⚙  │  ← 半透明黑 rgba(0,0,0,0.6) (C12) + 4px backdrop blur
│                                         │
│                                         │
│         （房间区，无 UI 遮挡）              │
│                                         │
│                          ●玩家          │
│                                         │
│                              [出口]     │
└─────────────────────────────────────────┘
```

### 6.2 HUD 元素视觉规格

| 元素 | 视觉 | 位置 | 字号 |
|------|------|------|------|
| **房间名** | #E0E0E0 (C8) / 18px | 左上 (2%, 4%) | 默认 18px / 22px (125%) / 27px (150%) |
| **章节进度** | #00D4FF (C6) / 12px | 左上 (2%, 9%) | 默认 12px / 15px (125%) / 18px (150%) |
| **设置按钮** | #E0E0E0 (C8) 圆形 24px | 右上 (92%, 4%) | — |
| **小键盘提示** | #E0E0E0 (C8) / 12px | 右下 (95%, 95%) | 默认 12px |
| **槽位提示浮动框** | rgba(0,0,0,0.7) (C12 加深) + 4px blur | 屏幕中下 30% | 14px (默认) |

### 6.3 字号缩放 3 档 (Font Scale 3 Modes)

> 引用 `12-art-style-v2.md` §7.5。

| 档位 | HUD | 槽位提示 | 菜单 | 适用 |
|------|:---:|:-------:|:----:|------|
| **100% (默认)** | 12-18px | 14px | 24px | 正常 |
| **125%** | 15-22px | 17.5px | 30px | 轻度视觉障碍 |
| **150%** | 18-27px | 21px | 36px | 中度视觉障碍 |

> **字号缩放规则：** 仅影响 UI 文字，**不影响房间区**（与 08-v2 §6.2 对齐）。

### 6.4 槽位 4 态视觉契约

> 引用 `12-art-style-v2.md` §7.2。

| 状态 | 视觉 | 触觉 | 玩家可操作？ |
|------|------|:----:|:----------:|
| **normal** | ❌ 不可见 | ❌ | ❌ |
| **hover** | 100% 不透明 + 青色脉冲 1.0s 周期 + 2px 青色边框 + 槽位类型图标 | ❌ | ✅ |
| **disabled** | 50% 不透明 + 灰色 + 锁图标 🔒 | ❌ | ❌ |
| **active** | 100% 不透明 + 青色实色（无脉冲）+ 2px 青色边框 + 旋转动画 0.2s | ✅ 50ms 震动 | ❌（动画锁定）|

### 6.5 4 槽位类型图标

| 槽位类型 | 图标 | 主色 | 边框样式 |
|---------|------|------|---------|
| **ToggleSlot (TS)** | ◯ (圆形 2 选 1) | #00D4FF 青色 (C6) | 实线 2px |
| **CycleSlot (CS)** | ⇆ (箭头循环) | #00D4FF 青色 (C6) | 虚线 2px |
| **ConditionalSlot (CDS)** | ◇ (菱形 条件) | #FF9500 橙色 (C7) | 实线 2px |
| **LockedSlot (LS)** | 🔒 (锁形) | #888888 灰色 (C9) | 实线 2px |

### 6.6 5 字体规范 (5 Fonts)

> 引用 `12-art-style-v2.md` §7.4 + `08-ui-ux-v2.md` §9.2。

| 语言 | 字体 | 来源 | 授权 | v1.0 | v1.1 |
|------|------|------|------|:----:|:----:|
| **英文 (en-US)** | Inter (Variable) | Google Fonts | OFL 1.1 | ✅ | ✅ |
| **简体中文 (zh-CN)** | Noto Sans CJK SC | Google Fonts | OFL 1.1 | ✅ | ✅ |
| **繁体中文 (zh-TW)** | Noto Sans CJK TC | Google Fonts | OFL 1.1 | ❌ | ✅ |
| **日文 (ja)** | Noto Sans JP | Google Fonts | OFL 1.1 | ❌ | ✅ |
| **韩文 (ko)** | Noto Sans KR | Google Fonts | OFL 1.1 | ❌ | ✅ |

> **设计原则：** 5 语种**全部用 Noto 家族**——免费 + 一致 + 跨平台。

### 6.7 字体实施细节

| 字段 | 规格 |
|------|------|
| **字重** | Regular (400) / Medium (500) / Bold (700) |
| **Variable 轴** | wght (100-900) |
| **抗锯齿** | Grayscale 8x |
| **渲染** | TextMeshPro SDF (高 DPI 清晰) |
| **子集** | 启用（仅保留游戏用字符）|
| **路径** | `Assets/Fonts/Inter-Variable.ttf` + `NotoSansSC-Variable.ttf` (+ TC/JP/KR v1.1) |

## 7. 7 平台美术规格 (7-Platform Art Spec)

> 引用 `docs/11-release-v2.md` §1.1。

### 7.1 7 平台美术规格速查

| 平台 | 分辨率 | 资产格式 | 压缩 | 适配工时 | 优先级 |
|------|--------|---------|------|:-------:|:----:|
| **Steam PC/Mac** | 720p/1080p (16:9) | PNG (Sprite Atlas) | LZ4 (默认) | 1h | **P0** (M11/M12) |
| **Itch.io (试玩版)** | 720p/1080p (16:9) | PNG (Sprite Atlas) | LZ4 (默认) | 0.5h | **P0** (M11) |
| **Nintendo Switch** | 720p 掌机 / 1080p 主机 | PNG + ETC2 压缩 | ETC2 | 2h | **P1** (v1.1, 200h 总) |
| **PS5** | 4K (3840×2160) | PNG + ASTC 压缩 | ASTC | 0.5h (评估) | **P2** (v2.0) |
| **Xbox Series X\|S** | 4K / 1440p | PNG + BC 压缩 | BC | 0.5h (评估) | **P2** (v2.0) |
| **iOS** | iPhone/iPad (Retina) | PNG + ASTC 压缩 | ASTC | 0.5h (评估) | **P2** (v2.0) |
| **Android** | 多端 (240dpi-560dpi) | PNG + ETC2 压缩 | ETC2 | 0.5h (评估) | **P2** (v2.0) |

### 7.2 Sprite Atlas 平台适配

| 平台 | Atlas 最大尺寸 | 压缩格式 | 备注 |
|------|:------------:|---------|------|
| **Steam PC/Mac** | 2048 × 2048 | LZ4 | 标准 |
| **Itch.io** | 2048 × 2048 | LZ4 | 同 Steam |
| **Switch** | 2048 × 2048 | ETC2 | 内存优化 |
| **PS5** | 4096 × 4096 | ASTC | 高分辨率 |
| **Xbox** | 4096 × 4096 | BC | 高分辨率 |
| **iOS** | 2048 × 2048 | ASTC | 移动端优化 |
| **Android** | 2048 × 2048 | ETC2 | 移动端优化 |

> **关键设计：** 美术源文件统一在 **2048×2048 Sprite Atlas** 输出，平台发布时按需压缩。**避免**为每个平台单独设计。

## 8. 12 动画原则 + 5 缓动函数 (12 Animation Principles + 5 Easing Functions)

> 引用 `12-art-style-v2.md` §8。

### 8.1 12 动画原则 (AP-1 ~ AP-12)

| # | 原则 | 在《暗室》中的体现 |
|---|------|------------------|
| **AP-1** 挤压拉伸 | 玩家移动时 0.1s 拉伸（X+1.1, Y-0.95）|
| **AP-2** 预备动作 | 切换前 0.05s 槽位轻微收缩（提示"即将变化"）|
| **AP-3** 演出布局 | 切换动画聚焦于槽位（其他元素 50% 透明度 0.1s）|
| **AP-4** 跟随动作 | 切换完成后 0.1s 视觉残像（淡出 0.05s）|
| **AP-5** 慢入慢出 | 切换动画 0.20s 缓动（EaseInOutQuad）|
| **AP-6** 弧形运动 | 出口光脉冲用 sin 曲线（自然呼吸感）|
| **AP-7** 附属动作 | 切换后出口光强度 +30%（强化"我做到了"）|
| **AP-8** 节奏 | 0.20s 切换 = 60 FPS × 12 帧（标准 1 个 beat）|
| **AP-9** 夸张 | 章节高潮（1-5 通关）光强 +20% 短暂 1s |
| **AP-10** 立体感 | 2D 极简（无 3D 透视）|
| **AP-11** 吸引力 | 槽位光恒定（始终吸引玩家）|
| **AP-12** 简洁 | 静帧主导（≥ 80% 时间无动画）|

### 8.2 关键时序约束 (Key Timing Constraints)

| 动画 | 时长 | 缓动函数 | 阻塞玩家输入？ |
|------|------|---------|:------------:|
| **切换动画（Switching）** | 200ms ± 50ms | EaseInOutQuad | ✅ |
| **重置动画（Reset）** | 300ms ± 50ms | EaseInQuad | ✅ |
| **通关渐白（Win）** | 500ms | Linear | ❌（透明 UI 仍可点）|
| **章节过渡（黑屏）** | 2000ms | Linear | ❌ |
| **章节标题画面** | 3000ms | EaseInOut | ❌ |
| **房间加载** | ≤ 1000ms | Linear | ❌ |
| **HUD 淡入** | 500ms | EaseOutQuad | ❌ |
| **HUD 淡出** | 300ms | EaseInQuad | ❌ |
| **槽位提示淡入** | 300ms | EaseOutQuad | ❌ |
| **槽位提示淡出** | 500ms | EaseInQuad | ❌ |
| **通关音** | 0.50s 一次性 | — | ❌ |
| **错音** | 0.15-0.20s 一次性 | — | ❌ |

### 8.3 5 缓动函数 (5 Easing Functions)

| 缓动 | 公式 | 用途 |
|------|------|------|
| **Linear** | t | 通关渐白 / 章节过渡 |
| **EaseInQuad** | t² | 淡出 / 重置（开始快）|
| **EaseOutQuad** | 1-(1-t)² | 淡入 / 加载（结束快）|
| **EaseInOutQuad** | t<0.5 ? 2t² : 1-2(1-t)² | 切换动画（首尾平滑）|
| **EaseOutBack** | 1+2.7*(t-1)³+1.7*(t-1)² | 通关庆祝（轻微回弹）|

### 8.4 DOTween 实施代码示例

```csharp
// src/Art/DOTweenAnimations.cs
using DG.Tweening;

// 切换动画（200ms ± 50ms, EaseInOutQuad）
public void PlaySwitchAnimation(GameObject slot) {
    slot.transform.DOScale(1.1f, 0.1f)  // 预备动作
        .SetEase(Ease.OutQuad)
        .OnComplete(() => {
            slot.transform.DOScale(0.9f, 0.1f)  // 切换中
                .SetEase(Ease.InQuad)
                .OnComplete(() => {
                    slot.transform.DOScale(1.0f, 0.1f)  // 恢复
                        .SetEase(Ease.OutQuad);
                });
        });
}

// 重置动画（300ms ± 50ms, EaseInQuad）
public void PlayResetAnimation(GameObject slot) {
    slot.transform.DOScale(0.8f, 0.15f)
        .SetEase(Ease.InQuad)
        .OnComplete(() => {
            slot.transform.DOScale(1.0f, 0.15f)
                .SetEase(Ease.OutQuad);
        });
}

// 通关渐白（500ms, Linear）
public void PlayWinAnimation(GameObject screen) {
    screen.GetComponent<CanvasGroup>().DOFade(1f, 0.5f)
        .SetEase(Ease.Linear);
}

// 出口脉冲（1.0s 周期, sin 曲线）
public void PlayExitPulseAnimation(Light2D exitLight) {
    exitLight.intensity = 1.0f;
    DOTween.To(() => exitLight.intensity, x => exitLight.intensity = x, 1.5f, 0.5f)
        .SetEase(Ease.InOutSine)
        .SetLoops(-1, LoopType.Yoyo);
}

// 玩家移动拉伸（0.1s, X+1.1, Y-0.95）
public void PlayPlayerWalkAnimation(Transform player) {
    player.DOScale(new Vector3(1.1f, 0.95f, 1f), 0.1f)
        .SetEase(Ease.OutQuad);
}
```

## 9. 风格一致性检查清单 (Style Consistency Checklist)

> 实施时逐项核对，确保与 `12-art-style-v2.md` 一致。

### 9.1 调色板检查 (Color Palette Checklist)

- [ ] 章节背景色 C1/C2/C3 严格使用 (#1A1A2E / #15152A / #0E0E1F)
- [ ] Floor 使用 C4 (#2D2D44)
- [ ] SolidWall 使用 C5 (#3D3D5C)
- [ ] 槽位发光使用 C6 (#00D4FF)
- [ ] 出口发光使用 C7 (#FF9500)
- [ ] 主文字使用 C8 (#E0E0E0)
- [ ] Locked 使用 C9 (#555555)
- [ ] 警告使用 C10 (#FF4444)
- [ ] 成就使用 C11 (#FFD700)
- [ ] UI 面板使用 C12 (rgba(0,0,0,0.6))

### 9.2 预制件检查 (Prefab Checklist)

- [ ] SolidWall: #3D3D5C + 1px #1A1A2E 描边
- [ ] Floor: #2D2D44 无描边
- [ ] Door: #5D4D2D 关 / #2D2D44 开 + #FF9500 描边
- [ ] GlassWall: rgba(0,212,255,0.3) + #00D4FF 描边
- [ ] CrumblingFloor: #2D2D44 + 裂纹纹理
- [ ] **FakeFloor: #2D2D44（与 Floor 1:1 像素匹配）**
- [ ] PressurePlate: #FF9500 + 圆形 + 1px 描边

### 9.3 动画时序检查 (Animation Timing Checklist)

- [ ] 切换动画 200ms ± 50ms
- [ ] 重置动画 300ms ± 50ms
- [ ] 通关渐白 500ms
- [ ] 章节过渡 2000ms
- [ ] 章节标题 3000ms
- [ ] HUD 淡入 500ms / 淡出 300ms
- [ ] 槽位提示淡入 300ms / 淡出 500ms

### 9.4 字号检查 (Font Size Checklist)

- [ ] 100% 档位: HUD 12-18px / 槽位 14px / 菜单 24px
- [ ] 125% 档位: HUD 15-22px / 槽位 17.5px / 菜单 30px
- [ ] 150% 档位: HUD 18-27px / 槽位 21px / 菜单 36px
- [ ] 字号缩放**仅影响 UI**，不影响房间区

### 9.5 色盲检查 (Color Blind Checklist)

- [ ] 默认色盲模式为 Normal
- [ ] 红绿色盲模式 C6 → #0099FF / C7 → #FFCC00 / C10 → #FF8800
- [ ] 全色盲模式 C6 → #FFFFFF + ▢ / C7 → #FFFF00 + ◇ / C10 → #CCCCCC + ⚠
- [ ] 实时预览（颜色立即生效）

## 10. 验收标准 (Acceptance Criteria)

- [x] **AC-01** Frontmatter 7 字段完整
- [x] **AC-02** 6 必填通用章节齐全
- [x] **AC-03** 12 主色调色板（含章节色 + 强调色 + 中性色 + 文字色 + 状态色）
- [x] **AC-04** 7 预制件视觉契约（SolidWall / Floor / Door / GlassWall / CrumblingFloor / FakeFloor / PressurePlate）
- [x] **AC-05** 3 章节主题 + 光强曲线 + 雾效 + 19 房间视觉差异表
- [x] **AC-06** 5 字体规范（en/zh-CN/zh-TW/ja/ko）
- [x] **AC-07** 3 字号缩放档（100% / 125% / 150%）
- [x] **AC-08** 12 动画原则 + 5 缓动函数 + 关键时序约束
- [x] **AC-09** 3 档色盲替代色（Normal / 红绿色盲 / 全色盲）
- [x] **AC-10** 7 平台美术规格速查表
- [x] **AC-11** 风格一致性检查清单（5 类 × 30 项）
- [x] **AC-12** P0-001 跟踪

## 11. 边界条件 (Edge Cases)

| # | 触发条件 | 预期行为 |
|---|---------|---------|
| **E1** | 玩家字号 150% 时槽位提示遮挡出口 | 槽位提示**自动上移** |
| **E2** | 玩家色盲模式切换时 | **实时预览**（颜色立即生效）|
| **E3** | 玩家通关 3-8 时 0.5s 渐白期间 | UI **仍可点击** |
| **E4** | 切换动画中按 ESC | 切换动画**立即完成** + UI 强制重置 |
| **E5** | 玩家快速进出 trigger 区（10 次/秒）| 槽位提示**淡出 0.5s 后强制 1s 最小显示** |
| **E6** | CrumblingFloor 碎裂后立即重置 | CrumblingFloor **恢复初始** |
| **E7** | FakeFloor 踩上后立即重置 | FakeFloor **恢复初始**（与 Floor 1:1）|
| **E8** | 全色盲玩家 + 小尺寸槽位 | 字号 150% 时图案**放大 1.5x** |
| **E9** | Switch 平台 ETC2 压缩失败 | 降级为 PNG LZ4 压缩 |
| **E10** | 玩家在 1-1 教学房停留 > 5 分钟 | 第 3 分钟弹出 HUD 提示 |

## 12. 配置表 (Configuration)

> 完整参数化字段（与 12-v2 §12 配置表一致）。

| 字段 | 类型 | 取值范围 | 默认值 | 单位 | 适用场景 |
|------|------|---------|-------|------|---------|
| `palette.bgCh1` | string | HEX | #1A1A2E | — | Ch1 背景色 |
| `palette.bgCh2` | string | HEX | #15152A | — | Ch2 背景色 |
| `palette.bgCh3` | string | HEX | #0E0E1F | — | Ch3 背景色 |
| `palette.slotCyan` | string | HEX | #00D4FF | — | 槽位发光青色 |
| `palette.exitOrange` | string | HEX | #FF9500 | — | 出口发光橙色 |
| `palette.solidWall` | string | HEX | #3D3D5C | — | 实墙 |
| `palette.floor` | string | HEX | #2D2D44 | — | 地板 |
| `palette.doorClosed` | string | HEX | #5D4D2D | — | 门关闭 |
| `palette.doorOpen` | string | HEX | #2D2D44 | — | 门开启 |
| `palette.glassWall` | string | HEX | rgba(0,212,255,0.3) | — | 透明墙 |
| `palette.warningRed` | string | HEX | #FF4444 | — | 警告/错音 |
| `palette.textPrimary` | string | HEX | #E0E0E0 | — | 主文字 |
| `palette.uiPanelBg` | string | HEX | rgba(0,0,0,0.6) | — | UI 半透明面板 |
| `light.mainAngleDeg` | float | [0, 360] | 315 | 度 | 主光角度 |
| `light.mainIntensityCh1` | float | [0.0, 1.0] | 0.6 | — | Ch1 主光强度 |
| `light.mainIntensityCh2` | float | [0.0, 1.0] | 0.4 | — | Ch2 主光强度 |
| `light.mainIntensityCh3` | float | [0.0, 1.0] | 0.3 | — | Ch3 主光强度 |
| `light.slotHoverRadius` | float | [0.5, 5.0] | 2.0 | 格 | 槽位光范围 |
| `light.exitConnectRadius` | float | [0.5, 5.0] | 3.0 | 格 | 出口光范围 |
| `light.fogCh1` | float | [0.0, 0.5] | 0.05 | — | Ch1 雾效透明度 |
| `light.fogCh2` | float | [0.0, 0.5] | 0.15 | — | Ch2 雾效透明度 |
| `light.fogCh3` | float | [0.0, 0.5] | 0.30 | — | Ch3 雾效透明度 |
| `fontScale.normal` | float | [0.8, 1.2] | 1.0 | 倍率 | 100% 字号 |
| `fontScale.large` | float | [1.0, 1.5] | 1.25 | 倍率 | 125% 字号 |
| `fontScale.xLarge` | float | [1.2, 2.0] | 1.5 | 倍率 | 150% 字号 |
| `font.roomName` | int | [12, 36] | 18 | px | 房间名 |
| `font.hud` | int | [10, 24] | 12 | px | HUD 文字 |
| `font.slotTip` | int | [10, 24] | 14 | px | 槽位提示 |
| `font.menu` | int | [16, 48] | 24 | px | 菜单项 |
| `font.title` | int | [24, 72] | 48 | px | 主菜单标题 |
| `anim.switchMs` | int | [100, 500] | 200 | ms | 切换动画 |
| `anim.resetMs` | int | [200, 500] | 300 | ms | 重置动画 |
| `anim.winMs` | int | [200, 1000] | 500 | ms | 通关渐白 |
| `anim.hudFadeInMs` | int | [100, 1000] | 500 | ms | HUD 淡入 |
| `anim.hudFadeOutMs` | int | [100, 1000] | 300 | ms | HUD 淡出 |
| `anim.slotTipFadeInMs` | int | [100, 500] | 300 | ms | 槽位提示淡入 |
| `anim.slotTipFadeOutMs` | int | [100, 500] | 500 | ms | 槽位提示淡出 |
| `anim.crumbleMs` | int | [200, 1000] | 500 | ms | CrumblingFloor 碎裂延迟 |
| `anim.fakeFloorFlashMs` | int | [100, 500] | 300 | ms | FakeFloor 闪烁 |
| `anim.chapterTransitionMs` | int | [1000, 5000] | 2000 | ms | 章节间黑屏 |
| `anim.chapterTitleMs` | int | [1000, 5000] | 3000 | ms | 章节标题画面 |
| `colorBlindMode` | enum | Normal / Deuteranopia / Monochromacy | Normal | — | 色盲模式 |
| `palette.cbDeutanSlot` | string | HEX | #0099FF | — | 红绿色盲槽位色 |
| `palette.cbDeutanExit` | string | HEX | #FFCC00 | — | 红绿色盲出口色 |
| `palette.cbMonoSlot` | string | HEX | #FFFFFF | — | 全色盲槽位色 |
| `palette.cbMonoExit` | string | HEX | #FFFF00 | — | 全色盲出口色 |
| `grid.tileSize` | int | [16, 64] | 32 | px | 1 格像素 |
| `grid.roomWidthMin` | int | [8, 32] | 12 | 格 | 房间最小宽度 |
| `grid.roomWidthMax` | int | [12, 40] | 20 | 格 | 房间最大宽度 |
| `grid.roomHeightMin` | int | [6, 16] | 8 | 格 | 房间最小高度 |
| `grid.roomHeightMax` | int | [8, 24] | 12 | 格 | 房间最大高度 |
| `grid.playerSize` | int | [8, 32] | 12 | px | 玩家精灵直径 |
| `grid.slotSize` | int | [16, 48] | 24 | px | 槽位尺寸 |
| `grid.exitSize` | int | [16, 64] | 32 | px | 出口尺寸 |

## 13. 关联文档

### 上游（本文档依赖）

- [`README.md`](./README.md) — 总览 + 8 文件索引
- [`docs/12-art-style-v2.md`](../../docs/12-art-style-v2.md) — 美术规格基线（权威定义）
- [`docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — 7 预制件类型 + 切换时序
- [`docs/03-level-design-v2.md`](../../docs/03-level-design-v2.md) — 19 房间配置
- [`docs/04-gameplay-flow-v2.md`](../../docs/04-gameplay-flow-v2.md) — 12 动画时序约束
- [`docs/06-player-experience-v2.md`](../../docs/06-player-experience-v2.md) — 色盲 3 档 + 沉浸感 3 层
- [`docs/08-ui-ux-v2.md`](../../docs/08-ui-ux-v2.md) — HUD 极简半透明 + 字号 3 档 + 字体规范
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台美术规格

### 下游（本文档被依赖）

- [`asset-list.md`](./asset-list.md) — 引用本文档 12 调色板 + 7 预制件 + 5 字体清单
- [`production-pipeline.md`](./production-pipeline.md) — 引用本文档 12 动画原则 + 5 缓动函数
- [`delivery-checklist.md`](./delivery-checklist.md) — 引用本文档风格一致性检查清单

## 14. 关联代码模块

| 模块 | 路径 | 状态 | 职责 |
|------|------|------|------|
| **PaletteManager** | `src/Art/PaletteManager.cs` | 待创建 | 12 主色 + 色盲 3 档 |
| **ChapterLightingController** | `src/Art/ChapterLightingController.cs` | 待创建 | 3 章节光强 + 雾效 |
| **URP 2D Lighting** | `src/Art/URP2DLighting.cs` | 待创建 | 槽位/出口/玩家光 |
| **DOTween 动画** | `src/Art/DOTweenAnimations.cs` | 待创建 | 12 动画原则 |
| **7 预制件视觉** | `src/Prefabs/*.cs` | 待创建 | 7 预制件视觉契约 |
| **FakeFloor 视觉欺骗** | `src/Art/FakeFloorVisualDeception.cs` | 待创建 | 1:1 像素匹配 |
| **CrumblingFloor 碎裂** | `src/Art/CrumblingFloorAnimator.cs` | 待创建 | 0.5s 延迟 + 粒子 |
| **压力板动画** | `src/Art/PressurePlateAnimator.cs` | 待创建 | 踩下缩放 |
| **玩家精灵动画** | `src/Art/PlayerSpriteAnimator.cs` | 待创建 | 移动拉伸 + Idle 呼吸 |
| **出口指示器** | `src/Art/ExitIndicator.cs` | 待创建 | 连通/未连通 2 态 |
| **HUD 极简半透明** | `src/UI/HUDStyling.cs` | 待创建 | rgba(0,0,0,0.6) + blur |
| **字号缩放 3 档** | `src/UI/FontScaleController.cs` | 待创建 | 100%/125%/150% |
| **色盲模式** | `src/UI/ColorBlindFilter.cs` | 待创建 | 3 档色盲 + 实时预览 |

## 15. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| **R-01** | **P0-001**（02-v2 §13 AC-06 缺"难度上限 20"）| 中 | 100% | 与 art 弱依赖（仅 Boss 房光强假设），不阻塞 v1.0 | **OPEN（弱依赖）** |
| **R-02** | **FakeFloor 1:1 像素匹配失误** | 高 | 30% | 像素对比工具验证 + Playtest 5 人验证 | 已规划 |
| **R-03** | **章节光强单调递减** 在 Ch3 Boss 房 0.2 强度下玩家看不清 | 中 | 25% | Boss 房实际 0.25 强度 | 已规划 |
| **R-04** | **Ch2 玻璃墙半透明 shader 性能问题** | 中 | 20% | URP 2D Renderer + Shader Graph 优化 | 待验证 |
| **R-05** | **CrumblingFloor 碎裂粒子占用 GPU 过高** | 低 | 15% | 限制粒子数 ≤ 10 + 短生命周期 0.3s | 已规划 |
| **R-06** | **全色盲玩家依赖图案识别** 在小尺寸槽位可能识别失败 | 中 | 35% | 字号 150% 时图案放大 1.5x | 已规划 |
| **R-07** | **DOTween MIT 协议变更** | 低 | 5% | 降级为 Unity Animator + 自写缓动函数 | 已规划 |
| **R-08** | **字号 150% 时槽位提示遮挡出口** | 中 | 40% | 槽位提示**自动上移** | 已规划 |
| **R-09** | **色盲模式实时预览不生效** | 中 | 15% | 启用调色板缓存 + 立即写存档 | 已规划 |
| **R-10** | **Switch ETC2 压缩导致美术细节丢失** | 低 | 20% | 压缩前留 1.5x 余量 | 已规划 |
| **Q-01** | **是否做 2D 物理光照（带阴影）**？ | 中 | — | v1.0 用色阶代替阴影；v1.1 评估 | 倾向 v1.0 不做 |
| **Q-02** | **是否做章节 BGM 节拍同步光斑**？ | 低 | — | v1.0 独立；v1.1 评估 | 倾向不做 |
| **Q-03** | **是否提供玩家自定义皮肤**？ | 低 | — | v1.0 不支持；v1.1 评估 | 倾向不做 |

## 16. 待办事项 (TODO)

- [ ] **P0：** 实现调色板管理器（PaletteManager）+ 12 主色 + 色盲 3 档切换 — 阻塞房间内循环可视化 [本文 §2]
- [ ] **P0：** 实现 URP 2D 光照（URP2DLighting）+ 槽位光 / 出口光 / 玩家光 — 阻塞核心视觉 [本文 §4]
- [ ] **P0：** 实现 7 预制件视觉契约（SolidWall / Floor / Door / GlassWall / CrumblingFloor / FakeFloor / PressurePlate）— 阻塞 19 房间实施 [本文 §3]
- [ ] **P0：** 实现 FakeFloor 1:1 像素匹配 + 踩上闪烁红色 — 阻塞 Ch3 3-3 [本文 §3.2.6]
- [ ] **P0：** 实现 CrumblingFloor 碎裂动画（0.5s 延迟 + 粒子）— 阻塞 Ch3 3-5 [本文 §3.2.5]
- [ ] **P0：** 实现玩家精灵动画（移动拉伸 + Idle 呼吸）— 阻塞核心循环 [本文 §5.1]
- [ ] **P0：** 实现 HUD 极简半透明（rgba(0,0,0,0.6) + 4px backdrop blur）— 阻塞 UI 实施 [本文 §6.1]
- [ ] **P1：** 实现 3 章节光强 + 雾效（ChapterLightingController）— 不阻塞 1.0 [本文 §4.2]
- [ ] **P1：** 实现字号缩放 3 档（FontScaleController）— 不阻塞 1.0 [本文 §6.3]
- [ ] **P1：** 实现色盲模式 3 档（ColorBlindFilter）— 不阻塞 1.0 [本文 §2.3]
- [ ] **P1：** 实现 PressurePlate 踩下动画 + 联动触发 — 不阻塞 1.0 [本文 §3.2.7]
- [ ] **P1：** 实现 12 动画原则（DOTween Animations）— 不阻塞 1.0 [本文 §8.1]
- [ ] **P2：** 评估 2D 物理光照（带阴影）— v1.1 评估 [Q-01]
- [ ] **P2：** 评估 BGM 节拍同步光斑脉冲 — v1.1 评估 [Q-02]
- [ ] **P2：** 解决 P0-001（02-v2 §13 AC-06 增补"难度上限 20"）— phase3 [R-01]

## 17. 评审迭代记录

| 轮 | 版本 | 时间 | 总分 | P0 | P1 | P2 | P3 | 备注 |
|---|------|------|:----:|---|---|---|---|------|
| 1 | v1.0 | 2026-06-29 | — | — | — | — | — | **本次初版:** 12 主色调色板（含 RGB + 色盲 3 档替代色）/ 7 预制件视觉契约（SolidWall/Floor/Door/GlassWall/CrumblingFloor/FakeFloor/PressurePlate）/ 3 章节主题 + 光强曲线 + 雾效 + 19 房间视觉差异表 / 5 字体规范 + 3 字号缩放档 / 12 动画原则 + 5 缓动函数 + 关键时序约束 / 7 平台美术规格速查 / 风格一致性检查清单（5 类 × 30 项）/ 54 配置参数 / 13 关联代码模块 |

## 18. 变更日志

| 日期 | 版本 | 变更人 | 内容 |
|------|------|--------|------|
| 2026-06-29 | v1.0 | 中书省 subagent | **ANZHONG-14 phase3 style-guide 创建:** 引用 12-v2 §3 §4 §7 §8 + 11-v2 §1.1 + 08-v2 §9.2 / 12 主色调色板（含 RGB + Unity 代码 + 色盲 3 档替代色）/ 7 预制件视觉契约（详细规格）/ 3 章节主题（色温 + 光强曲线 0.6/0.4/0.3 + 雾效 5%/15%/30% + 19 房间视觉差异表）/ 5 字体规范 + 3 字号缩放档 + HUD 极简半透明 + 槽位 4 态 + 4 槽位类型图标 / 7 平台美术规格 / 12 动画原则 + 5 缓动函数 + 关键时序约束 + DOTween 实施代码 / 风格一致性检查清单（5 类 × 30 项）/ 54 配置参数 / 13 关联代码模块 / P0-001 弱依赖 / 10 边界条件 / 15 待办 P0×7 P1×5 P2×3 / 整改 AUDIT-REPORT §2.art 全部 P0 整改项 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）
**P0-001 跟踪：** OPEN — 与 art 设计**弱依赖**（仅 Boss 房光强假设间接引用），不阻塞 v1.0 实施