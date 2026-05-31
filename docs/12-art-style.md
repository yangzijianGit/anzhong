---
title: 美术风格规范
doc_id: DOC-anzhong-12
parent: docs/README.md
last_updated: 2026-05-31
---

# 美术风格规范

## 5.1 推荐视觉风格

**参考方向：** Dead Cells × Gorogoa × 废弃设施氛围感

| 要素 | 建议 |
|------|------|
| **色调** | 低饱和度冷色调为主（深蓝灰、石墨色），槽位发光用青色/橙色点缀 |
| **整体氛围** | 废弃研究设施感，带有一点神秘感 |
| **UI风格** | 极简主义，半透明悬浮面板，无边框设计 |
| **槽位提示** | 槽位位置有微弱脉冲发光（青色），切换时发光增强 |
| **出口提示** | 连通时出口发出暖色脉冲光（橙黄） |

## 调色板

| 用途 | 色值 |
|------|------|
| 背景/深色墙 | #1A1A2E |
| 地板 | #2D2D44 |
| 实墙 | #3D3D5C |
| 槽位发光 | #00D4FF（青色） |
| 出口发光 | #FF9500（橙色） |
| 文字/UI | #E0E0E0 |

## 美术资源方案

### 最低成本路径

| 阶段 | 方案 | 成本 |
|------|------|------|
| 原型期 | Unity内置Primitives + Colored White Box | $0 |
| 过渡期 | 开放素材（Kenney.nl / OpenGameArt）2D Platformer Pack | $0 |
| 正式期 | 委托独立美术或使用付费资产包（如"Brackey's2D Pack"） | $$ |

### 推荐Unity Asset Store资源包

- Brackey's 2D Pack（高质量，免费）
- Pixel Art Platformer - Cave（$5~10）
- Dark Environment Pack（适合氛围感）

## 技术方案（Unity实现）

### 项目结构

- **渲染方案：** Unity URP + 2D Renderer（支持光照，效果优于 Built-in）
- **地图系统：** Unity Tilemap（用于基础地板/墙壁）+ 自定义房间系统（管理 SwitchSlot）
- **物理方案：** Unity 2D Physics（2D Collider，不使用 Rigidbody 动态物理，用 TileMap Collider 静态碰撞）
- **动画方案：** Unity Animator + DOTween（用于切换动画和UI反馈）

### 关卡编辑器建议

**推荐制作自定义关卡编辑器（EditorWindow）：**
- 在Unity Editor中嵌入房间编辑器
- 可视化编辑房间网格、放置槽位、配置预制件
- 一键导出为 JSON 或 ScriptableObject

**编辑器输出格式选择：JSON**
- 优势：人类可读、易于外部工具生成、可被其他平台解析
- 配合 Resources.Load<TextAsset> 加载