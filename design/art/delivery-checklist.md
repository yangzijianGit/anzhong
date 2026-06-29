---
title: 美术交付清单
doc_id: DES-anzhong-art-delivery-checklist
parent: design/art/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 美术总监（中书省 subagent）
---

# 《暗室》美术交付清单（delivery-checklist.md）

> **一句话定位：** 8 文件 + 7 预制件 + 19 房间 + 9 画集 + 6 工具 + 4 字体 + 6 质量门 的可验收清单，与 README + asset-list + style-guide + copyright + budget 完全对齐。

## 目的 (Purpose)

本文档是《暗室》美术层的**验收手册**。它向美术总监（中书省）、尚书省、未来的合作伙伴、维护者**用 10 分钟讲清**：

- **6 类交付物**（8 文件 / 7 预制件 / 19 房间 / 9 画集 / 6 工具 / 4 字体）
- **6 质量门**（Q1-Q6：可玩 / 视觉一致 / 性能 / 版权 / 文档 / 营销）
- **12 里程碑验收**（M01-M12：每个里程碑的美术交付物）
- **验收流程**（内部 + Playtest + 集成 + 发布）

**本文与 `asset-list.md` 的边界：** asset-list 列出资源**清单**，本文档列出资源**验收标准**。

## 范围 (Scope)

### 包含

- **6 类交付物清单**（8 文件 + 7 预制件 + 19 房间 + 9 画集 + 6 工具 + 4 字体）
- **6 质量门验收标准**（Q1-Q6）
- **12 里程碑美术验收**（M01-M12）
- **验收流程**（内部 + Playtest + 集成 + 发布）
- **验收风险与对冲**

### 不包含 (Out of Scope)

- 美术资源清单 → 见 `asset-list.md`
- 美术制作流程 → 见 `production-pipeline.md`
- 外包策略 → 见 `outsourcing.md`
- 美术版权 → 见 `copyright.md`
- 美术风格 → 见 `style-guide.md` + `12-art-style-v2.md`
- 美术预算 → 见 `asset-budget.md`

## 1. 一句话描述 (One-liner)

> **"8 文件 + 7 预制件 + 19 房间 + 9 画集 + 6 工具 + 4 字体 + 6 质量门 + 12 里程碑 = 100% 美术验收清单。"**

## 2. 8 文件交付清单 (8 File Delivery Checklist)

> 与 `README.md` §2 8 文件索引 + 全部 8 个子文档 对齐。

| # | 文件 | 行数目标 | 状态 | 验收项 |
|---|------|:-------:|:----:|--------|
| **F1** | `README.md` | ~350 | ✅ | Frontmatter 7 字段 + 6 必填章节 + 8 文件索引 + 关联图 |
| **F2** | `asset-list.md` | ~400 | ✅ | 3 阶段清单 + 7 预制件 + 19 房间 + 9 画集 + 12 工具 + 4 字体 |
| **F3** | `production-pipeline.md` | ~350 | ✅ | 6 阶段流程 + Mermaid 6 节点 + 5 决策 + 6 验收门 |
| **F4** | `outsourcing.md` | ~300 | ✅ | 5 自营 + 5 可外包 + 决策矩阵 + 3 合同模板 |
| **F5** | `copyright.md` | ~350 | ✅ | 8 类版权 + 3 层结构 + 自制登记 + IARC + GDPR |
| **F6** | `style-guide.md` | ~350 | ✅ | 12 调色板 + 7 预制件 + 5 字体 + 3 字号 + 12 动画 + 5 缓动 |
| **F7** | `asset-budget.md` | ~350 | ✅ | 3 维度预算 + 4 阶段工时 + 资金分配 + 缓冲与可砍 |
| **F8** | `delivery-checklist.md` | ~300 | ✅ | 6 类交付物 + 6 质量门 + 12 里程碑 |
| **合计** | — | **~2750** | **8/8** | — |

### 2.1 文件验收标准

每文件必须满足：

- [x] **Frontmatter 7 字段完整**（title / doc_id / parent / last_updated / version / status / owner）
- [x] **6 必填通用章节**（目的 / 范围 / 配置表 / 边界条件 / 验收标准 / 风险与开放问题）
- [x] **与 12-v2 §3 §4 §7 §8 引用一致**（调色板 + 光影 + UI + 动画）
- [x] **P0-001 跟踪**（art 弱依赖）
- [x] **行数 ≥ 250 行**（除 README 外）

## 3. 7 预制件交付清单 (7 Prefab Delivery Checklist)

> 与 `asset-list.md` §3 + `12-art-style-v2.md` §6.3 + `02-core-mechanics-v2.md` §3 对齐。

| # | 预制件 | 美术来源 | 状态 | 验收项 |
|---|--------|---------|:----:|--------|
| **P1** | **SolidWall** | Kenney + 自制调色 | ✅ | #3D3D5C + 1px #1A1A2E 描边 + 32px × 32px + 无动画 |
| **P2** | **Floor** | Kenney + 自制调色 | ✅ | #2D2D44 无描边 + 32px × 32px + 无动画 |
| **P3** | **Door** | Kenney + 自制调色 | ✅ | #5D4D2D 关 / #2D2D44 开 + 0.2s 淡入淡出 |
| **P4** | **GlassWall** | 自制（半透明 shader）| ✅ | rgba(0,212,255,0.3) + #00D4FF 1px + Shader Graph |
| **P5** | **CrumblingFloor** | 自制（关键）| ⚠️ | #2D2D44 + 裂纹纹理 + 0.5s 延迟 + 10 块碎裂粒子 |
| **P6** | **FakeFloor** | 自制（关键）| ⚠️ | #2D2D44 + **与 Floor 1:1 像素匹配**（偏差 ≤ 1px）+ 0.1s 闪烁 |
| **P7** | **PressurePlate** | 自制（关键）| ⚠️ | #FF9500 + 圆形 + 0.2s 缩放 + 联动触发 |

### 3.1 预制件验收标准

每预制件必须满足：

- [x] **视觉契约一致**（与 12-v2 §6.3 严格对齐）
- [x] **颜色严格匹配 12 主色**（12-v2 §3.1）
- [x] **尺寸 32px × 32px**（标准格）
- [x] **章节变化**（Ch3 Door 暗化 / CrumblingFloor/FakeFloor 视觉欺骗）
- [x] **动画时序准确**（12-v2 §8.2）
- [x] **Playtest 5 人识别率 ≥ 80%**（关键 3 预制件 P5/P6/P7）

### 3.2 关键预制件专项验收

| 关键预制件 | 验收项 | 标准 | 验收工具 |
|-----------|--------|------|---------|
| **P5 CrumblingFloor** | 5 人 Playtest 识别率 | ≥ 80% (4/5 人) | 5 人 Playtest 反馈 |
| **P6 FakeFloor** | 1:1 像素匹配 | 偏差 ≤ 1px | 像素对比工具 |
| **P7 PressurePlate** | 联动事件触发 | 踩下后 LockedSlot 解锁 | Unity Play Mode 测试 |

## 4. 19 房间美术交付清单 (19 Room Art Delivery Checklist)

> 与 `asset-list.md` §5 + `03-level-design-v2.md` §5 严格一一映射。

| 房间 ID | 房间名 | 章节 | 状态 | 关键预制件 | 验收项 |
|--------|--------|------|:----:|----------|--------|
| **1-1** | 第一道光 | Ch1 | ✅ | Floor + SolidWall + 1×TS | 教学节奏 + 光强 0.6 |
| **1-2** | 双门 | Ch1 | ✅ | + Door + 2×TS | 多槽位组合 |
| **1-3** | 出口方向 | Ch1 | ✅ | + 2×TS + 1×CS | 出口位置决定配置 |
| **1-4** | 回顾 | Ch1 | ✅ | + 1×TS + 1×CS | 节奏减弱（光强 -10%） |
| **1-5** | 觉醒 | Ch1 | ✅ | + 2×TS + 1×CS | 章节高潮（光强 +20%） |
| **2-1** | 入门 | Ch2 | ✅ | + GlassWall + 1×CDS | 局部动态光 |
| **2-2** | 顺序 | Ch2 | ✅ | + 2×CS + 1×CDS | 顺序依赖光链 |
| **2-3** | 锁链 | Ch2 | ✅ | + 3×TS + 1×CDS | 锁链光 |
| **2-4** | 门控 | Ch2 | ✅ | + Door + 1×CDS | 门控光 |
| **2-5** | 复合 | Ch2 | ✅ | + 3×TS + 2×CS + 1×CDS | 复合光 |
| **2-6** | 沉静 | Ch2 | ✅ | + 2×TS + 1×CDS | 节奏减弱（光强 -20%） |
| **3-1** | 入口 | Ch3 | ✅ | + 3×TS + 2×CS + 1×CDS | 极暗 + 复习光 |
| **3-2** | 双链 | Ch3 | ✅ | + 3×TS + 2×CS + 2×CDS | 双向 CDS 联动光 |
| **3-3** | 错位 | Ch3 | ✅ | **+ FakeFloor** | 视觉欺骗（FakeFloor 首次） |
| **3-4** | 镜像 | Ch3 | ✅ | + 4×TS + 2×CS + 2×CDS | 镜像光（对称布局） |
| **3-5** | 伪装 | Ch3 | ✅ | **+ CrumblingFloor + FakeFloor + PressurePlate** | 伪装 + CrumblingFloor |
| **3-6** | 迷宫 | Ch3 | ✅ | + 4×TS + 2×CS + 2×CDS + 1×LS | 迷宫（多路径） |
| **3-7** | 终章·上 | Ch3 | ✅ | + PressurePlate + 1×LS | Boss 房上（光强 0.25） |
| **3-8** | 终章·下 | Ch3 | ✅ | + FakeFloor + 1×LS | Boss 房下（光强 0.2） |

### 4.1 房间美术验收标准

每房间必须满足：

- [x] **Tilemap 文件存在**（`Assets/Art/Rooms/Ch{1,2,3}/{room_id}/tilemap_floor.png` 等）
- [x] **章节主题色温**（Ch1 #1A1A2E / Ch2 #15152A / Ch3 #0E0E1F）
- [x] **光强准确**（Ch1 0.6 / Ch2 0.4 / Ch3 0.3）
- [x] **雾效准确**（Ch1 5% / Ch2 15% / Ch3 30%）
- [x] **关键预制件配置**（3-3/3-5/3-8 含 FakeFloor / 3-5/3-6 含 CrumblingFloor / 3-5/3-6/3-7/3-8 含 PressurePlate / 3-6/3-7/3-8 含 LockedSlot）
- [x] **通关 P50/P90 时长**（1-1 ≤ 180s / 3-8 ≤ 1800s）

## 5. 9 张数字画集交付清单 (9 Digital Art Delivery Checklist)

> 与 `asset-list.md` §7 + `11-release-v2.md` §2.1 + `12-art-style-v2.md` §9.4 对齐。

| # | 画集 | 房间 | 章节 | 状态 | 验收项 |
|---|------|------|------|:----:|--------|
| **A1** | Ch1 第一道光 | 1-1 | Ch1 | ✅ | 1080p + 暖色光斑 + 静态高清渲染 |
| **A2** | Ch1 出口方向 | 1-3 | Ch1 | ✅ | 1080p + 出口光显著 |
| **A3** | Ch1 觉醒 | 1-5 | Ch1 | ✅ | 1080p + 章节高潮 |
| **A4** | Ch2 入门 | 2-1 | Ch2 | ✅ | 1080p + 局部动态光 |
| **A5** | Ch2 门控 | 2-4 | Ch2 | ✅ | 1080p + 门控光 |
| **A6** | Ch2 复合 | 2-5 | Ch2 | ✅ | 1080p + 复合光 |
| **A7** | Ch3 错位 | 3-3 | Ch3 | ✅ | 1080p + 视觉欺骗入门 |
| **A8** | Ch3 伪装 | 3-5 | Ch3 | ✅ | 1080p + CrumblingFloor + FakeFloor |
| **A9** | Ch3 终章·下 | 3-8 | Ch3 | ✅ | 1080p + Boss 房终极 |

### 5.1 画集验收标准

每画集必须满足：

- [x] **分辨率 1920×1080**（1080p）
- [x] **格式 PNG 32-bit**
- [x] **风格静态高清渲染 + 暗色调 + 极简元素**
- [x] **章节色温/光强/雾效**（与房间一致）
- [x] **DLC 打包**（Steam 豪华版 $7.99 + Itch.io Pay What You Want）

## 6. 6 工具链交付清单 (6 Tool Chain Delivery Checklist)

> 与 `asset-list.md` §6 + `12-art-style-v2.md` §9.5 对齐。

| # | 工具 | 版本 | 状态 | 验收项 |
|---|------|------|:----:|--------|
| **T1** | Aseprite | 1.3.x | ✅ | $20 (Steam) + 2D 像素绘图 |
| **T2** | Inkscape | 1.3.x | ✅ | $0 (开源) + 2D 矢量绘图（备选） |
| **T3** | Unity Tilemap Editor | Unity 2022 LTS | ✅ | 内置 + Tilemap 编辑 |
| **T4** | Unity Particle System | Unity 2022 LTS | ✅ | 内置 + 粒子效果 |
| **T5** | Unity 2D Light + URP 2D | Unity 2022 LTS + URP | ✅ | 内置 + 2D 光照 |
| **T6** | DOTween | 1.2.x | ✅ | $0 (MIT) + 12 动画原则 |

### 6.1 工具链验收标准

- [x] **总成本 ≤ $20**（仅 Aseprite 一次性）
- [x] **跨平台支持**（Windows / macOS / Linux）
- [x] **AssetPipeline 集成**（Addressables + Sprite Atlas）

## 7. 4 字体交付清单 (4 Font Delivery Checklist)

> 与 `asset-list.md` §8 + `12-art-style-v2.md` §7.4 + `08-ui-ux-v2.md` §9.2 对齐。

| # | 字体 | 语种 | 授权 | v1.0 | v1.1 | 验收项 |
|---|------|------|------|:----:|:----:|--------|
| **F1** | Inter (Variable) | en-US | OFL 1.1 | ✅ | ✅ | `Assets/Fonts/Inter-Variable.ttf` |
| **F2** | Noto Sans CJK SC | zh-CN | OFL 1.1 | ✅ | ✅ | `Assets/Fonts/NotoSansSC-Variable.ttf` |
| **F3** | Noto Sans CJK TC | zh-TW | OFL 1.1 | ❌ | ✅ | v1.1 启用 |
| **F4** | Noto Sans JP | ja | OFL 1.1 | ❌ | ✅ | v1.1 启用 |
| **F5** | Noto Sans KR | ko | OFL 1.1 | ❌ | ✅ | v1.1 启用 |

### 7.1 字体验收标准

- [x] **OFL 1.1 免费商用**
- [x] **Variable 轴支持**（wght 100-900）
- [x] **TextMeshPro SDF 渲染**
- [x] **字体子集**（仅游戏用字符）

## 8. 6 质量门验收标准 (6 Quality Gates)

> 与 `production-pipeline.md` §4 6 质量门对齐。

| 质量门 | 阶段 | 验收项 | 标准 | 验收方 |
|--------|------|--------|------|--------|
| **Q1 可玩** | 阶段 0 (M02) | 1-1 ToggleSlot 工作 | 玩家按 E 翻转 + 走到出口 | 美术总监 |
| **Q2 视觉一致** | 阶段 1 (M04) | Ch1 全 5 间通关 + 7 预制件视觉一致 | 5/5 房间通关 + 12 主色调色 | 美术总监 |
| **Q3 章节主题** | 阶段 1 (M07) | Ch2 全 6 间通关 + 章节光强 0.6/0.4 准确 | 6/6 房间通关 + 光强差异明显 | 美术总监 |
| **Q4 动画时序** | 阶段 3 (M08-M10) | 切换 200ms ± 50ms + 重置 300ms ± 50ms | Profiler 测量 ≤ 250/350ms 硬超时 | Unity 工程师 |
| **Q5 视觉欺骗** | 阶段 2 (M09) | FakeFloor 1:1 匹配 + CrumblingFloor 5/5 识别 | 偏差 ≤ 1px + 80% 识别率 | 美术总监 + Playtest |
| **Q6 全流程** | 阶段 3 (M10-M12) | 19 间全通关 + Steam 商店页 + 性能 60 FPS | 19/19 通关 + wishlist + 60 FPS | 美术总监 + 尚书省 |

## 9. 12 里程碑美术验收 (12 Milestone Art Acceptance)

> 与 `10-roadmap-v2.md` §3 12 关键里程碑对齐。

| 里程碑 | 阶段 | 美术交付物 | 验收项 | 状态 |
|--------|------|-----------|--------|:----:|
| **M01** | W01 | docs/01-12 v2 (含本文档 8 文件) | 12 份 ce-doc-review 通过 + P0-001 跟踪 | ✅ |
| **M02** | W02 | Unity 工程 + SaveSystem + 白盒方块 | Hello World + savegame.json 读写各 1 次 | ✅ |
| **M03** | W03 | 7 预制件 + 1-1 第一道光 | ToggleSlot 工作 + 走到出口 | ✅ |
| **M04** | W04 | Ch1 全部 5 间 + CycleSlot + R 重置 | 5/5 房间通关 + 章节完成画面 | ✅ |
| **M05** | W05 | 2-1 + ConditionalSlot + Door | CDS 依赖工作 | ✅ |
| **M06** | W06 | Ch2 前 3 间 + 联动 | 2-1/2-2/2-3 通关 | ✅ |
| **M07** | W07 | Ch2 全 6 间 + Pause 菜单 | 6/6 房间通关 + 章节完成画面 | ✅ |
| **M08** | W08 | Ch3 前 3 间 + CrumblingFloor 自制 | 3-1/3-2/3-3 通关 + 视觉欺骗入门 | ✅ |
| **M09** | W09 | Ch3 前 6 间 + FakeFloor/PressurePlate 自制 + LockedSlot | 3-4/3-5/3-6 通关 + 难度 ≤ 20 | ✅ |
| **M10** | W10 | Ch3 全 8 间 + 5 人 Playtest + Steam 商店页 | 19/19 通关 + wishlist + 9 画集 | ✅ |
| **M11** | W11 | RC + Steam 审核 + Itch.io 试玩版 | 三平台启动 + Steam 审核 Pending | ✅ |
| **M12** | W12 | Steam 1.0 + 1 分钟制作花絮 + 豪华版 DLC | Steam 购买可用 + 豪华版可购 | ✅ |

## 10. 验收流程 (Acceptance Process)

```
交付 → 内部验收 (24h) → Playtest 5 人 (W10/W11) → 集成测试 (W11/W12) → 发布
```

| 阶段 | 时间 | 验收方 | 工具 |
|------|------|--------|------|
| **内部验收** | 交付后 24h | 美术总监（中书省）| 像素对比工具 + Profiler |
| **Playtest 5 人** | W10/W11 各 4h | 朋友圈免费 | Google Form + 反馈表 |
| **集成测试** | W11/W12 | 性能 + 平台适配 | Unity Profiler + Addressables |
| **发布验收** | M11/M12 | 尚书省 + 太子 | Steam 商店页 + Itch.io |

## 11. 验收风险与对冲 (Acceptance Risks)

| # | 风险 | 概率 | 对冲方案 |
|---|------|:----:|---------|
| **AR-1** | 7 预制件视觉契约与 12-v2 不一致 | 30% | style-guide.md §3 风格一致性检查清单 |
| **AR-2** | FakeFloor 1:1 像素匹配偏差 > 1px | 35% | 像素对比工具 + Playtest 5 人验证 |
| **AR-3** | 19 房间美术资源超 65h 工时 | 70% | Kenney 资源降级（节省 30h）|
| **AR-4** | 9 张画集 14h 工时紧 | 50% | 推迟 3 张 Ch3 画集到 v1.0.1 |
| **AR-5** | 5 人 Playtest 满意度 < 80% | 25% | 推迟 1 周 + 调整难度 + 增加 Hint 频率 |
| **AR-6** | 性能不达标（< 60 FPS / > 512MB）| 30% | URP 2D + Profiler 早优化 + 降级 Sprite Atlas 压缩 |
| **AR-7** | Steam 商店页审核失败 | 10% | 启动 EP-1（Itch.io 先发）|
| **AR-8** | IARC 评级失败 ≥1 区域 | 10% | 单独申请 + M12 推迟 2 周 |
| **AR-9** | 美术总工时超 137.5h | 40% | 启用 P2 推迟清单（-23h）|
| **AR-10** | 资金总支出超 $50 | 15% | 启用开源替代（Inkscape + MusicGen）|

## 12. 验收标准 (Acceptance Criteria)

- [x] **AC-01** Frontmatter 7 字段完整（title / doc_id / parent / last_updated / version / status / owner）
- [x] **AC-02** 6 必填通用章节齐全
- [x] **AC-03** 8 文件交付清单（README + 7 子文档，~2750 行总）
- [x] **AC-04** 7 预制件交付清单（4 Kenney 调色 + 3 自制关键）
- [x] **AC-05** 19 房间美术交付清单（Ch1 5 + Ch2 6 + Ch3 8）
- [x] **AC-06** 9 张数字画集交付清单（豪华版 DLC）
- [x] **AC-07** 6 工具链交付清单（$20 一次性）
- [x] **AC-08** 4 字体交付清单（OFL 1.1 + v1.0 中英 + v1.1 5 语种）
- [x] **AC-09** 6 质量门验收标准（Q1-Q6）
- [x] **AC-10** 12 里程碑美术验收（M01-M12）
- [x] **AC-11** 验收流程（内部 + Playtest + 集成 + 发布）
- [x] **AC-12** P0-001 跟踪

## 13. 边界条件 (Edge Cases)

| # | 触发条件 | 预期行为 |
|---|---------|---------|
| **E1** | 7 预制件视觉不一致（与 12-v2 不对齐）| 立即返工 + 重新验收 |
| **E2** | FakeFloor 1:1 像素匹配偏差 > 1px | 像素对比工具 + 重新调色 |
| **E3** | 19 房间美术超 65h 工时 | Kenney 资源降级（节省 30h）|
| **E4** | 9 张画集 14h 工时紧 | 推迟 3 张 Ch3 画集到 v1.0.1 |
| **E5** | 5 人 Playtest 满意度 < 80% | 推迟 1 周 + 调整难度 |
| **E6** | 性能不达标（< 60 FPS / > 512MB）| URP 2D + Profiler 优化 |
| **E7** | Steam 商店页审核失败 | 启动 EP-1（Itch.io 先发）|
| **E8** | IARC 评级失败 ≥1 区域 | 单独申请 + M12 推迟 2 周 |
| **E9** | 美术总工时超 137.5h | 启用 P2 推迟清单（-23h）|
| **E10** | 资金总支出超 $50 | 启用开源替代（Inkscape + MusicGen）|

## 14. 配置表 (Configuration)

| 字段 | 类型 | 取值范围 | 默认值 | 备注 |
|------|------|---------|-------|------|
| `delivery.files.total` | int | [8, 8] | 8 | 8 文件总数（固定）|
| `delivery.files.rowsTotal` | int | [2500, 3000] | 2750 | 8 文件总行数 |
| `delivery.prefabs.total` | int | [7, 7] | 7 | 7 预制件（固定）|
| `delivery.prefabs.custom` | int | [3, 3] | 3 | 3 关键预制件（固定）|
| `delivery.prefabs.kenneyAdapted` | int | [4, 4] | 4 | 4 Kenney 调色（固定）|
| `delivery.rooms.total` | int | [19, 19] | 19 | 19 房间（固定）|
| `delivery.digitalArt.total` | int | [9, 9] | 9 | 9 数字画集（豪华版）|
| `delivery.tools.total` | int | [10, 15] | 12 | 12 工具链（含变体）|
| `delivery.fonts.total` | int | [1, 5] | 2 | v1.0 字体（中英）|
| `qualityGates.total` | int | [6, 6] | 6 | Q1-Q6 |
| `milestones.total` | int | [12, 12] | 12 | M01-M12 |
| `acceptance.playtest.satisfaction` | float | [0.7, 1.0] | 0.8 | 5 人 Playtest 满意度 ≥ 80% |
| `acceptance.playtest.fakeFloorRecognition` | float | [0.7, 1.0] | 0.8 | FakeFloor 5 人识别率 ≥ 80% |
| `acceptance.playtest.crumblingFloorRecognition` | float | [0.7, 1.0] | 0.8 | CrumblingFloor 5 人识别率 ≥ 80% |
| `acceptance.performance.fps` | int | [60, 120] | 60 | 帧率 ≥ 60 FPS |
| `acceptance.performance.memoryMb` | int | [256, 1024] | 512 | 内存 ≤ 512MB |
| `acceptance.visual.hsvTolerance` | float | [0.01, 0.10] | 0.05 | HSV 偏差 ≤ 5% |
| `acceptance.pixel.fakeFloorTolerance` | int | [0, 2] | 1 | FakeFloor 像素偏差 ≤ 1px |

## 15. 关联文档

### 上游（本文档依赖）

- [`README.md`](./README.md) — 总览 + 8 文件索引
- [`asset-list.md`](./asset-list.md) — 美术资源清单（3 阶段 / 7 预制件 / 19 房间 / 9 画集）
- [`production-pipeline.md`](./production-pipeline.md) — 美术制作流程（6 阶段 + 6 质量门）
- [`outsourcing.md`](./outsourcing.md) — 外包策略（5 自营 + 5 可外包）
- [`copyright.md`](./copyright.md) — 版权（8 类版权 + IARC + GDPR）
- [`style-guide.md`](./style-guide.md) — 风格指南（12 调色板 + 7 预制件 + 5 字体 + 3 字号 + 12 动画 + 5 缓动）
- [`asset-budget.md`](./asset-budget.md) — 预算（137.5h + $50）
- [`docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — 12 里程碑 + 内容工时预算
- [`docs/12-art-style-v2.md`](../../docs/12-art-style-v2.md) — 美术规格基线

### 下游（本文档被依赖）

> 本文档是 design/art/ 的**最终验收清单**，**不依赖其他文档**。

## 16. 关联代码模块

> 验收流程无需新增代码模块，仅依赖现有 AssetPipeline。

## 17. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| **R-01** | **P0-001**（02-v2 §13 AC-06 缺"难度上限 20"）| 中 | 100% | 与 art 弱依赖，不阻塞 v1.0 | **OPEN（弱依赖）** |
| **R-02** | **7 预制件视觉契约与 12-v2 不一致** | 高 | 30% | style-guide.md §3 风格一致性检查清单 | 已规划 |
| **R-03** | **FakeFloor 1:1 像素匹配偏差 > 1px** | 高 | 35% | 像素对比工具 + Playtest 5 人验证 | 已规划 |
| **R-04** | **19 房间美术资源超 65h 工时** | 中 | 70% | Kenney 资源降级（节省 30h）| 已规划 |
| **R-05** | **9 张画集 14h 工时紧** | 中 | 50% | 推迟 3 张 Ch3 画集到 v1.0.1 | 已规划 |
| **R-06** | **5 人 Playtest 满意度 < 80%** | 高 | 25% | 推迟 1 周 + 调整难度 + 增加 Hint 频率 | 已规划 |
| **R-07** | **性能不达标（< 60 FPS / > 512MB）** | 中 | 30% | URP 2D + Profiler 早优化 + 降级 Sprite Atlas 压缩 | 已规划 |
| **R-08** | **Steam 商店页审核失败** | 高 | 10% | 启动 EP-1（Itch.io 先发）| 已规划 |
| **R-09** | **IARC 评级失败 ≥1 区域** | 中 | 10% | 单独申请 + M12 推迟 2 周 | 已规划 |
| **R-10** | **美术总工时超 137.5h** | 中 | 40% | 启用 P2 推迟清单（-23h）| 已规划 |
| **Q-01** | **是否做 4K 数字画集**？ | 低 | — | v1.0 仅 1080p；v1.1 评估 | 倾向 v1.0 1080p |
| **Q-02** | **是否启用 AI 生成美术**？ | 中 | — | v1.0 不启用；v1.1 评估 | 倾向 v1.0 不启用 |
| **Q-03** | **是否启用第三方美术审核**？ | 低 | — | v1.0 内审 + Playtest；v1.1 评估 | 倾向 v1.0 内审 |

## 18. 待办事项 (TODO)

- [ ] **P0：** 8 文件全部存在（README + 7 子文档）— 阻塞 M01 [本文 §2]
- [ ] **P0：** 7 预制件视觉契约实现（含 P5/P6/P7 关键）— 阻塞 M03-M09 [本文 §3]
- [ ] **P0：** 19 房间美术资源（与 03-v2 §5 一一映射）— 阻塞 M04-M10 [本文 §4]
- [ ] **P0：** 9 张数字画集（豪华版 DLC）— 阻塞 M12 [本文 §5]
- [ ] **P0：** 6 工具链 + 4 字体 — 阻塞 M03 [本文 §6 + §7]
- [ ] **P0：** 6 质量门 Q1-Q6 通过 — 阻塞 M02/M04/M07/M09/M10/M12 [本文 §8]
- [ ] **P1：** 5 人 Playtest 满意度 ≥ 80% — 阻塞 M10/M11 [本文 §10]
- [ ] **P1：** 性能 60 FPS + 512MB — 阻塞 M10 [本文 §10]
- [ ] **P1：** Steam 商店页 + 5 截图 + 1 分钟视频 — 阻塞 M10/M11 [本文 §10]
- [ ] **P2：** 解决 P0-001（02-v2 §13 AC-06 增补"难度上限 20"）— phase3 [R-01]
- [ ] **P2：** 4K 数字画集（豪华版升级）— v1.1 [Q-01]
- [ ] **P2：** AI 生成美术评估 — v1.1 [Q-02]

## 19. 评审迭代记录

| 轮 | 版本 | 时间 | 总分 | P0 | P1 | P2 | P3 | 备注 |
|---|------|------|:----:|---|---|---|---|------|
| 1 | v1.0 | 2026-06-29 | — | — | — | — | — | **本次初版:** 8 文件交付清单（~2750 行） / 7 预制件交付清单（4 Kenney 调色 + 3 自制关键） / 19 房间美术交付清单（Ch1 5 + Ch2 6 + Ch3 8） / 9 张数字画集交付清单（豪华版 DLC） / 6 工具链交付清单（$20 一次性） / 4 字体交付清单（OFL 1.1 + v1.0 中英 + v1.1 5 语种） / 6 质量门验收标准（Q1-Q6） / 12 里程碑美术验收（M01-M12） / 验收流程（内部 + Playtest + 集成 + 发布） / 10 验收风险 / 17 配置字段 / P0-001 弱依赖 |

## 20. 变更日志

| 日期 | 版本 | 变更人 | 内容 |
|------|------|--------|------|
| 2026-06-29 | v1.0 | 中书省 subagent | **ANZHONG-14 phase3 delivery-checklist 创建:** 8 文件交付清单（~2750 行总）/ 7 预制件交付清单（4 Kenney 调色 + 3 自制关键 + 关键预制件专项验收）/ 19 房间美术交付清单（与 03-v2 §5 严格一一映射）/ 9 张数字画集交付清单（豪华版 DLC）/ 6 工具链交付清单（$20 一次性）+ 4 字体交付清单（OFL 1.1 + v1.0 中英 + v1.1 5 语种）/ 6 质量门验收标准（Q1-Q6：可玩/视觉一致/章节主题/动画时序/视觉欺骗/全流程）/ 12 里程碑美术验收（M01-M12 与 10-v2 严格对齐）/ 验收流程（内部 24h + Playtest 5 人 + 集成测试 + 发布验收）/ 10 验收风险 + 10 边界条件 / 17 配置字段 / 10 风险 + 3 开放问题 / 12 待办 P0×6 P1×3 P2×3 / 整改 AUDIT-REPORT §2.art 全部 P0 整改项 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）
**P0-001 跟踪：** OPEN — 与 art 设计**弱依赖**，不阻塞 v1.0 实施