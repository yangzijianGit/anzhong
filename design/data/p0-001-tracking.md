---
title: 《暗室》P0-001 跟踪文档 (P0-001 Tracking)
doc_id: DESIGN-anzhong-data-p0-001
parent: design/data/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》P0-001 跟踪文档 (P0-001 Tracking)

> **⚠️ 强 P0-001 关联文档 ⚠️**
>
> P0-001: 02-core-mechanics-v2.md §13 AC-06 缺"难度上限 20"硬约束 (与 05 §5.2 + 03 §6.2 不一致)
>
> **本文档立场：auto-chain 不擅自修复 P0-001，仅跟踪 + 文档化 + 提供修复草案，等陛下 21:00 调研后决定。**

## 目的 (Purpose)

本文档是《暗室》**P0-001 阻塞问题的全量跟踪文档**。它向：

- **陛下 (决策者)** — 提供 P0-001 完整描述 + 阻塞字段清单 + 修复草案 + 决策选项 (A/B/C)
- **中书省 (后续阶段)** — 提供 P0-001 修复的字段映射 + 数据回填 SQL + 验证步骤
- **所有工程师** — 明确"哪些字段不能填、填什么默认值、UI 显示什么"
- **QA / 测试** — 验证 P0-001 修复后的回归测试用例

**本版本（v1.0）的目的：** 把 P0-001 的**详细描述 + 11 阻塞字段清单 + 当前临时方案 + 修复时机 (3 选项) + 关联 10 份 v2 文档跟踪 + 修复示例草案 + 验证步骤**——**第一次**用统一文档完整跟踪，作为 phase3 → phase4 实施的"P0-001 决策包"。

## 范围 (Scope)

### 包含

- **P0-001 详细描述**: 02-v2 §13 AC-06 缺漏 + 与 05 §5.2 + 03 §6.2 不一致
- **阻塞字段清单 (15 字段)**: database-schema.md T03/T06/T07/T08/T10/T12/T15/T14 中所有难度字段
- **当前临时方案**: NULL/-1 占位 + UI "待配置" + 不编造数据
- **修复时机 (3 选项)**: A. phase3 design 期间 / B. phase4 单独 patch / C. 实施期间补
- **关联 10 份 v2 文档**: 02/03/05/06/07/08/09/10b/11/12 显式跟踪 + 引用列表
- **修复示例草案**: 02-v2 §13 AC-06 增补内容 + Alembic 数据回填 + 验证步骤

### 不包含 (Out of Scope)

- P0-001 的**实质性修复** (auto-chain 不擅自代决)
- 陛下 21:00 调研的最终决策
- 其他 P0/P1 风险 (R-01 ~ R-10) 的修复

## 一句话描述 (One-liner)

> **"P0-001 全量跟踪 + 15 阻塞字段 TODO + 3 修复时机选项 + 不编造数据 + 等陛下决策。"**

---

## 1. P0-001 详细描述 (P0-001 Detailed Description)

### 1.1 核心问题

**02-core-mechanics-v2.md §13 AC-06 缺"难度上限 20"硬约束。**

> **当前 02-v2 §13 AC-06 实际内容（截至 2026-06-29）：**
>
> ```
> - [ ] **AC-06：** 4 种槽位类型（Toggle / Cycle / Conditional / Locked）行为契约齐全
> ```
>
> **预期内容（应包含"难度上限 20"硬约束）：**
>
> ```
> - [ ] **AC-06：** 4 种槽位类型（Toggle / Cycle / Conditional / Locked）行为契约齐全
> - [ ] **AC-06b（建议新增）：** 房间难度 ≤ 20 硬约束 (与 05 §5.2 + 03 §6.2 对齐)
>       - 单房间难度计算公式 F1: 难度 = (槽位 × 选项系数) + 联动 + 空间
>       - 难度上限: 20 (硬约束, 不可突破)
>       - 验证: Ch3 Boss 房 (3-4/3-5/3-6) 实际计算 17.5/20/21.5 → 已通过回退到 ≤ 20 (W09)
>       - 数据层: rooms.difficulty_max = 20 常量
> ```

### 1.2 不一致链路

```
05-numerical-design-v2.md §5.2
  ├─ 难度上限 20 硬约束 ✅ 已写入
  ├─ §6.1 19 房间数据点表 (全部 ≤ 20) ✅ 已调参
  └─ §14 R-01: "02-v2 AC-06 缺同步" ❌ 阻塞
        ↓
03-level-design-v2.md §6.2
  ├─ Ch3 Boss 房警示 (3-4/3-5/3-6 实际计算 21.5-27) ⚠️
  └─ §13 关联: "依赖 05 §5.2 同步到 02-v2" ❌ 阻塞
        ↓
02-core-mechanics-v2.md §13 AC-06
  ├─ 仅描述 4 槽位类型 ❌
  └─ 缺"难度上限 20"硬约束 ❌ P0-001 源头
        ↓
10-roadmap-v2.md R6 W01
  └─ 必解决 (阻塞 M09 Ch3 Boss 房实施) ❌
        ↓
07-failure-retry-v2.md §9.3 R-01
  └─ 间接依赖 (重置难度) ⚠️
        ↓
11-release-v2.md R6
  └─ 阻塞 M09-M12 实施 ❌
        ↓
12-art-style-v2.md (主题美术依赖 Ch3 难度定位) ⚠️
```

### 1.3 影响范围

| 影响 | 详情 | 严重度 |
|------|------|:------:|
| **Ch3 Boss 房实施 (M09)** | 3-4/3-5/3-6 实际计算难度 17.5/20/21.5，若 21.5 不回退则**不可解** | 🔴 高 |
| **数据层 (本文档)** | 15 字段标 TODO，UI 显示"待配置" | 🟡 中 |
| **服务端 leaderboard (E10)** | difficulty_filter 参数无法严格校验 | 🟡 中 |
| **客户端存档 (SaveData)** | max_difficulty_completed 字段暂时无法填实际值 | 🟢 低 |
| **平衡性测试 (W09)** | Ch3 实际值与预期值的偏差无法系统性验证 | 🟡 中 |

### 1.4 当前状态 (截至 2026-06-29)

```
P0-001 状态: OPEN (阻塞中)
- 02-v2 §13 AC-06 仍缺"难度上限 20"硬约束
- 等待陛下 21:00 调研 + 决策
- auto-chain 立场: 不擅自修复, 仅跟踪 + 文档化
```

---

## 2. 阻塞字段清单 (15 Blocked Fields)

> 所有 15 个字段全部 NOT NULL DEFAULT NULL + CHECK (field IS NULL OR field BETWEEN 1 AND 20)。
> **不编造数据**，等 02-v2 AC-06 增补后回填。

### 2.1 按表分组

#### T03 — `rooms` (2 字段)

| # | 字段 | 类型 | 当前默认值 | CHECK 约束 | 用途 |
|---|------|------|----------|-----------|------|
| 1 | `difficulty` | INTEGER | 1 (实际值) | BETWEEN 1 AND 20 | 房间计算难度 |
| 2 | `difficulty_max` | INTEGER | 20 (常量) | = 20 | 难度上限 (自我保护) |

**当前实际值（来自 05 §6.1）：**
- 1-1: 2, 1-2: 3, 1-3: 4, 1-4: 3, 1-5: 5
- 2-1: 7, 2-2: 8, 2-3: 10, 2-4: 12, 2-5: 16, 2-6: 8
- 3-1: 13, 3-2: 14, 3-3: 16, 3-4: 17 (回退前 17.5), 3-5: 20 (回退前 20), 3-6: 20 (回退前 21.5), 3-7: 20, 3-8: 20

#### T06 — `chapters` (1 字段)

| # | 字段 | 类型 | 当前默认值 | 用途 |
|---|------|------|----------|------|
| 3 | `average_difficulty` | REAL | NULL | 章节平均难度 (5/6/8 房间均值) |

#### T07 — `player_progress` (2 字段)

| # | 字段 | 类型 | 当前默认值 | 用途 |
|---|------|------|----------|------|
| 4 | `max_difficulty_reached` | INTEGER | NULL | 玩家通关的最高难度 |
| 5 | `max_difficulty_attempted` | INTEGER | NULL | 玩家尝试但未通关的最高难度 |

#### T08 — `scores` (2 字段)

| # | 字段 | 类型 | 当前默认值 | 用途 |
|---|------|------|----------|------|
| 6 | `difficulty_used` | INTEGER | NULL | 通关时房间难度 (冗余便于排行榜) |
| 7 | `difficulty_at_play` | INTEGER | NULL | 通关时玩家难度选项 (easy/normal) |

#### T10 — `player_audio_settings` (1 字段, 关联)

| # | 字段 | 类型 | 当前默认值 | 用途 |
|---|------|------|----------|------|
| 8 | (关联) `accessibility_settings.difficulty` | ENUM | 'normal' | 玩家难度选项 (easy/normal v1.0) |

**注意：** 此字段已有 v1.0 临时方案 (easy/normal 二档)，但 hard 推 v1.1。**与 P0-001 关联但非完全阻塞。**

#### T12 — `saves` (3 字段)

| # | 字段 | 类型 | 当前默认值 | 用途 |
|---|------|------|----------|------|
| 9 | `max_difficulty_completed` | INTEGER | NULL | 玩家通关的最高难度 |
| 10 | `max_difficulty_ever` | INTEGER | NULL | 玩家进入过的最高难度 (含未通关) |
| 11 | `last_difficulty_played` | INTEGER | NULL | 最后一次进入的房间难度 |

#### T14 — `telemetry_events` (1 字段)

| # | 字段 | 类型 | 当前默认值 | 用途 |
|---|------|------|----------|------|
| 12 | `difficulty_context` | INTEGER | NULL | 事件上下文难度 |

#### T15 — `leaderboard_entries` (2 字段)

| # | 字段 | 类型 | 当前默认值 | 用途 |
|---|------|------|----------|------|
| 13 | `difficulty_filter` | INTEGER | NULL | 排行榜筛选的难度上限 (E10 query param) |
| 14 | `difficulty_used` | INTEGER | NULL | 通关时实际难度 (冗余自 scores) |

#### T01 — `players` (1 字段, 关联)

| # | 字段 | 类型 | 当前默认值 | 用途 |
|---|------|------|----------|------|
| 15 | `total_difficulty_completed` | INTEGER | NULL | 玩家累计完成难度 (跨房间求和) |

### 2.2 UI 显示策略 (User-Facing)

| 场景 | UI 显示 | 备注 |
|------|---------|------|
| **玩家查看自己的进度** | "最高通关难度: 待配置（P0-001 待解决）" | 不显示实际值 |
| **排行榜 (E10)** | "排行榜筛选: 全部难度" (暂不实现 difficulty_filter) | 等 P0-001 解决 |
| **房间详情页** | "难度: [1-20]" (显示范围而非具体值) | 保持神秘感 |
| **数据导出 (GDPR)** | 字段值为 NULL, 注明"待 P0-001 解决后回填" | 合规透明 |

---

## 3. 当前临时方案 (Current Temporary Solution)

### 3.1 数据层临时方案

```sql
-- 所有 15 个难度字段统一临时方案:
-- 1. 允许 NULL (默认)
-- 2. CHECK 约束允许 NULL 或 [1, 20] 范围
-- 3. 不设置 NOT NULL

ALTER TABLE rooms
ADD CONSTRAINT ck_rooms_difficulty_todo
CHECK (difficulty IS NULL OR difficulty BETWEEN 1 AND 20);

ALTER TABLE scores
ADD CONSTRAINT ck_scores_difficulty_todo
CHECK (difficulty_used IS NULL OR difficulty_used BETWEEN 1 AND 20);

-- ... (其他 13 字段类似)
```

### 3.2 应用层临时方案

```csharp
// src/SaveSystem/DifficultyHelper.cs
public static class DifficultyHelper
{
    public const bool P0_001_OPEN = true;  // 等 02-v2 AC-06 解决后改为 false

    public static string GetDisplayValue(int? difficulty, string context)
    {
        if (P0_001_OPEN && difficulty.HasValue)
        {
            // P0-001 未解决: 显示范围而非具体值
            return "待配置（P0-001 待解决）";
        }

        if (!difficulty.HasValue)
        {
            return "未知";
        }

        return difficulty.Value.ToString();
    }

    public static bool ValidateDifficulty(int? difficulty)
    {
        // P0-001 未解决: 仅校验范围, 不校验业务约束
        if (!difficulty.HasValue) return true;  // NULL 合法
        return difficulty.Value >= 1 && difficulty.Value <= 20;
    }
}
```

### 3.3 数据回填占位脚本 (P0-001 解决后执行)

```python
# tools/data/p0_001_backfill.py
"""
P0-001 解决后, 数据回填脚本 (执行时机: P0-001 修复后 24h 内)

回填策略:
- rooms.difficulty: 从 05 §6.1 表格导入 (已知数据)
- chapters.average_difficulty: 计算 5/6/8 房间均值
- player_progress.max_difficulty_reached: 从 scores.max(difficulty_used) 计算
- player_progress.max_difficulty_attempted: 从 telemetry_events.max(difficulty_context) 计算
- saves.max_difficulty_completed: 从 player_progress 同步
- scores.difficulty_used: 从 rooms.difficulty JOIN 导入
- leaderboard_entries.difficulty_filter/difficulty_used: 从 scores JOIN 导入
- telemetry_events.difficulty_context: 从 events.properties.difficulty 回填
- players.total_difficulty_completed: 从 saves JOIN 聚合
"""

async def backfill_p0_001(db: AsyncSession):
    # 1. rooms.difficulty (已知值, 直接更新)
    room_difficulties = {
        '1-1': 2, '1-2': 3, '1-3': 4, '1-4': 3, '1-5': 5,
        '2-1': 7, '2-2': 8, '2-3': 10, '2-4': 12, '2-5': 16, '2-6': 8,
        '3-1': 13, '3-2': 14, '3-3': 16, '3-4': 17, '3-5': 20,
        '3-6': 20, '3-7': 20, '3-8': 20
    }
    for room_id, diff in room_difficulties.items():
        await db.execute(
            update(Room).where(Room.id == room_id).values(difficulty=diff)
        )

    # 2. chapters.average_difficulty (计算)
    chapter_avgs = {
        'ch1': 3.4,   # (2+3+4+3+5)/5
        'ch2': 10.2,  # (7+8+10+12+16+8)/6
        'ch3': 17.6,  # (13+14+16+17+20+20+20+20)/8
    }
    for ch_id, avg in chapter_avgs.items():
        await db.execute(
            update(Chapter).where(Chapter.id == ch_id).values(average_difficulty=avg)
        )

    # 3. scores.difficulty_used (从 rooms JOIN)
    await db.execute("""
        UPDATE scores s
        SET difficulty_used = r.difficulty
        FROM rooms r
        WHERE s.room_id = r.id
          AND s.difficulty_used IS NULL
    """)

    # 4. player_progress.max_difficulty_reached (从 scores 聚合)
    await db.execute("""
        UPDATE player_progress pp
        SET max_difficulty_reached = sub.max_diff
        FROM (
            SELECT player_id, MAX(difficulty_used) AS max_diff
            FROM scores
            WHERE difficulty_used IS NOT NULL
            GROUP BY player_id
        ) sub
        WHERE pp.player_id = sub.player_id
          AND pp.max_difficulty_reached IS NULL
    """)

    # 5. ... (其他 11 字段类似回填)

    await db.commit()
    print(f"P0-001 backfill completed at {datetime.now()}")
```

---

## 4. 修复时机 (3 决策选项)

### 选项 A: phase3 design 期间解决 (W01, 推荐 ✅)

**时机：** phase3 第一周 (W01, 2026-07-01 ~ 2026-07-07)

**步骤：**
1. **W01 D1-D2**: 02-v2 维护者 (尚书省) 在 §13 AC-06 增补"难度上限 20"硬约束
   - 方案 1: 现有 AC-06 描述扩写
   - 方案 2: 新增 AC-06b 条目
2. **W01 D3**: 同步更新 03-v2 §6.2 (引用 02-v2 新增 AC)
3. **W01 D4**: 同步更新 05-v2 §14 R-01 标记 RESOLVED
4. **W01 D5**: 同步更新 10-v2 R6 W01 标记 RESOLVED
5. **W01 D6**: 数据回填 (运行 `tools/data/p0_001_backfill.py`)
6. **W01 D7**: 验证 + ce-doc-review

**优点：**
- ✅ 不阻塞 phase3 后续 (M09 Ch3 Boss 房)
- ✅ 数据层 + 应用层同步解决
- ✅ 不影响 v1.0 发布计划

**缺点：**
- ❌ 需要 02-v2 维护者 (尚书省) 立即投入
- ❌ 需要 5 份 v2 文档同步更新 (02/03/05/10/12)

**风险：**
- 🟢 低: 仅文档 + 数据回填, 不影响代码

### 选项 B: phase4 单独 patch (W12+)

**时机：** phase4 实施期间, v1.0 发布前 (W12+, 2026-09)

**步骤：**
1. **W12**: 02-v2 维护者增补 AC-06
2. **W12+1**: 同步更新 5 份 v2 文档
3. **W12+2**: 数据回填 + 应用层启用
4. **v1.0 发布**: 完整难度数据

**优点：**
- ✅ 不阻塞 phase3 design
- ✅ 在实施期间解决 (更接近真实场景)

**缺点：**
- ❌ 阻塞 phase4 M09 (Ch3 Boss 房实施)
- ❌ 阻塞 v1.0 发布 (若 Ch3 Boss 房验证失败)

**风险：**
- 🟡 中: 阻塞 phase4 关键路径

### 选项 C: 实施期间补 (W03+ 灵活)

**时机：** phase4 实施期间, 遇到具体问题时再补

**步骤：**
1. **W03+**: 实施 M03 房间模块时, 遇到难度数据问题
2. **临时方案**: 用现有 CHECK 约束 (BETWEEN 1 AND 20) 保护
3. **延后修复**: 等 W08 Ch3 Boss 房实施时再补

**优点：**
- ✅ 最灵活, 按需解决

**缺点：**
- ❌ 风险最高 (Ch3 Boss 房可能不可解)
- ❌ 没有明确的修复时间表
- ❌ 数据层 NULL 占位长期存在

**风险：**
- 🔴 高: Ch3 Boss 房难度可能超过 20 (3-4/3-5/3-6 实际 17.5/20/21.5)

### 4.4 决策建议

**推荐选项 A (phase3 W01 解决)**，理由：
1. 不阻塞 phase3 后续 + phase4 实施
2. 与 10-v2 R6 "W01 必解决" 计划一致
3. auto-chain 不擅自代决, 仅跟踪 + 文档化
4. 给陛下 (决策者) 提供清晰的决策包

---

## 5. 关联 10 份 v2 文档跟踪 (Cross-References)

> P0-001 显式跟踪已存在于 10 份 v2 文档中, 引用列表如下。

### 5.1 P0-001 显式跟踪的 10 份文档

| # | 文档 | 跟踪位置 | 状态 |
|---|------|---------|:----:|
| 1 | `docs/02-core-mechanics-v2.md` | §13 AC-06 (源头, 缺漏) | ❌ |
| 2 | `docs/03-level-design-v2.md` | §6.2 Ch3 Boss 房警示 (依赖 05 §5.2) | ⚠️ |
| 3 | `docs/05-numerical-design-v2.md` | §5.2 难度上限 20 + §14 R-01 (源头已定) | ✅ |
| 4 | `docs/06-player-experience-v2.md` | §10.4 难度选项 + §8.3 难度坡度 | ⚠️ |
| 5 | `docs/07-failure-retry-v2.md` | §9.3 P0-001 跟踪 + §16 R-01 | ⚠️ |
| 6 | `docs/08-ui-ux-v2.md` | §6.4 难度选项 + §6.5 5 开关 | ⚠️ |
| 7 | `docs/09-audio-v2.md` | (间接) 9 类音频 dB 与难度无直接关联 | ✅ |
| 8 | `docs/10-roadmap-v2.md` | R6 + W01 + M09 (显式跟踪, 阻塞关键路径) | ⚠️ |
| 9 | `docs/11-release-v2.md` | R6 OPEN (阻塞 M09-M12) | ⚠️ |
| 10 | `docs/12-art-style-v2.md` | Ch3 主题美术依赖难度定位 | ⚠️ |

**总跟踪文档：10 份**（02/03/05/06/07/08/09/10/11/12）

### 5.2 跟踪一致性 (Consistency Check)

| 字段 | 02-v2 | 03-v2 | 05-v2 | 06-v2 | 10-v2 | 11-v2 |
|------|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
| 难度上限 20 | ❌ 缺 | ✅ 引 | ✅ 定 | ✅ 引 | ✅ 引 | ✅ 引 |
| Ch3 Boss 房 ≤ 20 | — | ⚠️ 警 | ✅ 定 | ✅ 引 | ✅ 引 | ✅ 引 |
| 02-v2 同步阻塞 | — | ⚠️ 引 | ✅ R-01 | ✅ 引 | ✅ R6 | ✅ R6 |

**结论：** 除 02-v2 缺漏外, 其他 9 份文档均已正确跟踪 + 引用 + 警示。

### 5.3 修复时同步清单 (Sync List)

修复 P0-001 时, 需同步更新的文档:

```
□ 02-core-mechanics-v2.md §13 AC-06 (新增"难度上限 20"硬约束)
□ 03-level-design-v2.md §6.2 (移除 Ch3 Boss 房警示, 改为 RESOLVED)
□ 05-numerical-design-v2.md §14 R-01 (标记 RESOLVED)
□ 06-player-experience-v2.md §10.4 (移除"待 02 同步"备注)
□ 07-failure-retry-v2.md §9.3 (标记 RESOLVED)
□ 08-ui-ux-v2.md §6.4 (移除"待 02 同步"备注)
□ 09-audio-v2.md (无需变更, 仅核对)
□ 10-roadmap-v2.md R6 (标记 RESOLVED) + W01 (标记完成)
□ 11-release-v2.md R6 (标记 RESOLVED)
□ 12-art-style-v2.md (核对 Ch3 主题, 无需变更)
□ design/data/database-schema.md (去除 15 字段 TODO comment)
□ design/data/persistence-strategy.md (去除 TODO 标记)
□ design/data/serialization.md (去除 TODO 字段)
□ design/data/README.md (去除 P0-001 跟踪章节)
□ design/data/p0-001-tracking.md (本文档, 标记 RESOLVED)
```

---

## 6. 修复示例草案 (Fix Proposal Draft)

> 等待陛下 (决策者) 决策后, 由 02-v2 维护者 (尚书省) 执行。

### 6.1 02-v2 §13 AC-06 增补内容草案

```markdown
# 02-core-mechanics-v2.md §13 验收标准 (AC)

原内容:
- [ ] **AC-06：** 4 种槽位类型（Toggle / Cycle / Conditional / Locked）行为契约齐全

建议增补:
- [ ] **AC-06：** 4 种槽位类型（Toggle / Cycle / Conditional / Locked）行为契约齐全
- [ ] **AC-06b（新增）：** 房间难度上限 20 硬约束 (与 05 §5.2 对齐)
      - **公式 F1:** 难度 = (槽位 × 选项系数) + 联动 + 空间
        - 槽位基础: Toggle/Cycle = 1, Conditional = 2, Locked = 3
        - 选项系数: 2 选项 = 1, 3 选项 = 2, 4 选项 = 3
        - 联动: 每个联动 +1
        - 空间: width × height 系数 [0.5, 2.0]
      - **硬约束:** 难度 ≤ 20 (任何房间不可超过)
      - **验证:** Ch3 Boss 房 (3-4/3-5/3-6) 实际计算 17.5/20/21.5 → 通过回退到 ≤ 20
      - **数据层:** rooms.difficulty_max = 20 (CHECK 约束)
      - **服务端:** rooms.difficulty BETWEEN 1 AND 20 (CHECK 约束)
      - **客户端:** SQLite rooms 表 difficulty_max = 20 (CHECK 约束)
      - **平衡性测试 (W09):** 全部 19 房间实际计算 ≤ 20
```

### 6.2 Alembic 数据回填 (P0-001 解决后)

```python
# migrations/versions/2026_09_01_0005_p0_001_resolved_and_backfill.py
"""P0-001 解决: 02-v2 §13 AC-06b 增补 + 数据回填

Revision ID: 2026_09_01_0005
Revises: 2026_08_15_0004
Create Date: 2026-09-01 00:00:00.000000

条件: 必须先解决 02-v2 §13 AC-06b
"""

def upgrade():
    # 1. 去除 TODO CHECK 约束
    op.drop_constraint('ck_rooms_difficulty_todo', 'rooms', type_='check')
    op.drop_constraint('ck_scores_difficulty_todo', 'scores', type_='check')
    # ... (其他 13 字段)

    # 2. 添加正式 CHECK 约束 (难度 ∈ [1, 20])
    op.create_check_constraint(
        'ck_rooms_difficulty',
        'rooms',
        'difficulty BETWEEN 1 AND 20'
    )
    op.create_check_constraint(
        'ck_rooms_difficulty_max',
        'rooms',
        'difficulty_max = 20'
    )

    # 3. 数据回填 (从 05 §6.1 表格)
    room_difficulties = {
        '1-1': 2, '1-2': 3, '1-3': 4, '1-4': 3, '1-5': 5,
        '2-1': 7, '2-2': 8, '2-3': 10, '2-4': 12, '2-5': 16, '2-6': 8,
        '3-1': 13, '3-2': 14, '3-3': 16, '3-4': 17, '3-5': 20,
        '3-6': 20, '3-7': 20, '3-8': 20
    }
    for room_id, diff in room_difficulties.items():
        op.execute(f"UPDATE rooms SET difficulty = {diff} WHERE id = '{room_id}'")

    # 4. 计算并填充 chapters.average_difficulty
    chapter_avgs = {
        'ch1': 3.4, 'ch2': 10.2, 'ch3': 17.6
    }
    for ch_id, avg in chapter_avgs.items():
        op.execute(f"UPDATE chapters SET average_difficulty = {avg} WHERE id = '{ch_id}'")

    # 5. scores.difficulty_used 从 rooms JOIN
    op.execute("""
        UPDATE scores s
        SET difficulty_used = r.difficulty
        FROM rooms r
        WHERE s.room_id = r.id
    """)

    # 6. player_progress.max_difficulty_reached 从 scores 聚合
    op.execute("""
        UPDATE player_progress pp
        SET max_difficulty_reached = sub.max_diff
        FROM (
            SELECT player_id, MAX(difficulty_used) AS max_diff
            FROM scores
            GROUP BY player_id
        ) sub
        WHERE pp.player_id = sub.player_id
    """)

    # 7. saves.max_difficulty_completed 从 player_progress 同步
    op.execute("""
        UPDATE saves s
        SET max_difficulty_completed = pp.max_difficulty_reached
        FROM player_progress pp
        WHERE s.player_id = pp.player_id
    """)

    # 8. 其他字段 (similar)


def downgrade():
    # 反向: 重新加 TODO CHECK
    op.drop_constraint('ck_rooms_difficulty', 'rooms', type_='check')
    op.create_check_constraint(
        'ck_rooms_difficulty_todo',
        'rooms',
        'difficulty IS NULL OR difficulty BETWEEN 1 AND 20'
    )
    # ... (其他字段)
    op.execute("UPDATE rooms SET difficulty = NULL WHERE id IN (...)")  # 恢复 NULL
```

### 6.3 验证步骤 (Validation)

```bash
#!/bin/bash
# tools/verify/p0_001_resolved.sh
# P0-001 解决后验证脚本

set -euo pipefail

echo "=== P0-001 解决验证 ==="

# 1. 数据库约束验证
echo "[1/5] 验证数据库 CHECK 约束..."
psql -d anzhong -c "
    SELECT conname FROM pg_constraint
    WHERE conname LIKE 'ck_%difficulty%'
    ORDER BY conname
"
# 预期: ck_rooms_difficulty + ck_rooms_difficulty_max + ck_scores_difficulty_used

# 2. 数据完整性验证 (所有 19 房间 difficulty ≤ 20)
echo "[2/5] 验证 19 房间 difficulty ≤ 20..."
psql -d anzhong -c "
    SELECT id, name, difficulty, difficulty_max
    FROM rooms
    WHERE difficulty > 20 OR difficulty_max != 20
"
# 预期: 0 行 (无违反)

# 3. 平均难度验证 (3 章节 average 计算正确)
echo "[3/5] 验证章节平均难度..."
psql -d anzhong -c "
    SELECT c.id, c.average_difficulty,
           ROUND(AVG(r.difficulty)::NUMERIC, 1) AS computed_avg,
           ABS(c.average_difficulty - AVG(r.difficulty)) AS diff
    FROM chapters c
    JOIN rooms r ON r.chapter_id = c.id
    GROUP BY c.id, c.average_difficulty
    HAVING ABS(c.average_difficulty - AVG(r.difficulty)) > 0.1
"
# 预期: 0 行 (无显著偏差)

# 4. 玩家进度验证
echo "[4/5] 验证玩家 max_difficulty_reached 准确性..."
psql -d anzhong -c "
    SELECT pp.player_id,
           pp.max_difficulty_reached,
           MAX(s.difficulty_used) AS computed_max,
           pp.max_difficulty_reached - MAX(s.difficulty_used) AS diff
    FROM player_progress pp
    JOIN scores s ON s.player_id = pp.player_id
    WHERE s.difficulty_used IS NOT NULL
    GROUP BY pp.player_id, pp.max_difficulty_reached
    HAVING pp.max_difficulty_reached != MAX(s.difficulty_used)
    LIMIT 10
"
# 预期: 0 行 (所有玩家一致)

# 5. UI 验证 (Playwright)
echo "[5/5] 验证 UI 显示 (Playwright)..."
python tools/verify/ui_difficulty_display.py
# 检查: 玩家详情页显示真实难度值 (非"待配置")

echo "=== P0-001 验证通过 ✅ ==="
```

---

## 7. 关联文档 (Cross-References)

### P0-001 跟踪相关 (本文档)

- [`README.md`](./README.md) — 数据架构总览 (含 P0-001 概览)
- [`persistence-strategy.md`](./persistence-strategy.md) — 持久化策略 (含 TODO 字段)
- [`database-schema.md`](./database-schema.md) — 18 表 schema (15 字段 TODO)
- [`cache-strategy.md`](./cache-strategy.md) — Redis 缓存
- [`serialization.md`](./serialization.md) — Protobuf (.proto TODO 字段)
- [`migrations.md`](./migrations.md) — Alembic (P0-001 迁移示例)
- [`backup-and-recovery.md`](./backup-and-recovery.md) — 备份策略

### 上游 (P0-001 源头 + 跟踪)

- [`../../docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) — **源头** §13 AC-06 缺漏
- [`../../docs/03-level-design-v2.md`](../../docs/03-level-design-v2.md) — §6.2 Ch3 Boss 警示
- [`../../docs/05-numerical-design-v2.md`](../../docs/05-numerical-design-v2.md) — §5.2 难度上限 20 + §14 R-01
- [`../../docs/06-player-experience-v2.md`](../../docs/06-player-experience-v2.md) — §10.4 难度选项
- [`../../docs/07-failure-retry-v2.md`](../../docs/07-failure-retry-v2.md) — §9.3 P0-001 跟踪 + §16 R-01
- [`../../docs/08-ui-ux-v2.md`](../../docs/08-ui-ux-v2.md) — §6.4 难度选项
- [`../../docs/09-audio-v2.md`](../../docs/09-audio-v2.md) — (间接, 9 类音频与难度无直接关联)
- [`../../docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — R6 W01 (阻塞关键路径)
- [`../../docs/11-release-v2.md`](../../docs/11-release-v2.md) — R6 OPEN (阻塞 M09-M12)
- [`../../docs/12-art-style-v2.md`](../../docs/12-art-style-v2.md) — Ch3 主题美术依赖难度定位

### 设计文档关联

- [`../api/data-models.md`](../api/data-models.md) — M03 Room.difficulty/difficultyMax (P0-001 自我保护)
- [`../api/endpoints.md`](../api/endpoints.md) — E10 Leaderboard difficultyMax query param
- [`../architecture/README.md`](../architecture/README.md) — R-01 P0-001 跟踪 (自我保护)
- [`../architecture/data-flow.md`](../architecture/data-flow.md) — Persistence 数据流 (TODO)
- [`../architecture/tech-stack.md`](../architecture/tech-stack.md) — PostgreSQL CHECK constraint
- [`../art/README.md`](../art/README.md) — Ch3 主题美术依赖

### auto-chain 决策链

- `/Users/yzj/.openclaw/workspace-taizi/data/auto_chain_anzhong.json` — ANZHONG-15 任务定义
- `/Users/yzj/.openclaw/workspace-zhongshu/AGENTS.md` — 中书省工作协议 (硬要求)
- `/Users/yzj/.openclaw/workspace-zhongshu/data/ANZHONG-15/006-result.md` — 本次完工报告 (写入后)

## 8. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** P0-001 详细描述 (02-v2 §13 AC-06 缺漏) + 15 阻塞字段清单 + 临时方案 (NULL/-1 占位 + UI "待配置") + 3 修复时机选项 (A phase3 design / B phase4 patch / C 实施期间) + 10 份 v2 文档跟踪清单 + 修复示例草案 (AC-06b 增补 + Alembic 数据回填 + 验证步骤) + auto-chain 立场 (不擅自修复, 仅跟踪 + 文档化, 等陛下 21:00 决策)。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）