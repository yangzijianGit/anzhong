---
title: 多 Agent 评审流程
doc_id: DOC-anzhong-REVIEW
parent: docs/DOC-STANDARD.md
last_updated: 2026-06-05
version: v1.0
status: draft
owner: 尚书省
---

# 《暗室》多 Agent 评审流程设计

> **目的：** 定义"尚书省交付文档 → 多 agent 并行评审 → 打回重写循环"的标准化流程
> **适用范围：** `docs/01-*.md` ~ `docs/12-*.md` 的所有产品文档
> **设计时间：** 2026-06-05
> **设计者：** 尚书省（受太子委派）

---

## 0. 流程全景图

```
                ┌─────────────────────────────────┐
                │  尚书省提交文档 v1.0             │
                │  文档路径: docs/XX-xxx.md        │
                │  状态: draft                    │
                └────────────────┬────────────────┘
                                 │
                                 ▼
                ┌─────────────────────────────────┐
                │  朝堂议政 session 启动           │
                │  4 agent 并行评审               │
                │  ┌────────┐ ┌────────┐          │
                │  │策划agent│ │技术agent│ ...     │
                │  └────────┘ └────────┘          │
                └────────────────┬────────────────┘
                                 │
                                 ▼
                ┌─────────────────────────────────┐
                │  收集评审结果 (review-result.json)│
                │  聚合 P0/P1/P2/P3 issues        │
                └────────────────┬────────────────┘
                                 │
                                 ▼
                  ┌──────────────┴──────────────┐
                  │                             │
              [无 P0/P1/P2]              [有 P0/P1/P2]
                  │                             │
                  ▼                             ▼
        ┌──────────────────┐         ┌──────────────────┐
        │  评审通过         │         │  打回尚书省       │
        │  status: reviewed │         │  附问题清单       │
        │  → 等太子批准     │         │  轮次 +1         │
        │    → approved    │         └─────────┬────────┘
        └──────────────────┘                    │
                                                ▼
                                     ┌──────────────────┐
                                     │  轮次 ≤ 3 ?       │
                                     └────┬─────────┬───┘
                                          │         │
                                       [是]      [否 + 还有 P1+]
                                          │         │
                                          ▼         ▼
                              ┌──────────────┐  ┌──────────────┐
                              │  尚书省重写   │  │ 标记卡点任务  │
                              │  v1.1        │  │ 上报太子      │
                              │  → 重新评审  │  │ 等待裁决     │
                              └──────────────┘  └──────────────┘
```

---

## 1. 评审 Agent 角色配置

### 1.1 4 个 Agent 角色

| 角色 | 关注维度 | 核心问题 |
|------|---------|---------|
| **策划 agent** | 产品视角 | "玩家会觉得好玩吗？机制清晰吗？难度合理吗？" |
| **技术 agent** | 实现视角 | "Unity 2D 能做吗？性能达标吗？代码结构合理吗？" |
| **美术 agent** | 视觉听觉视角 | "美术风格统一吗？UI 美观吗？色调协调吗？" |
| **玩家 agent** | 用户视角 | "我第一眼能看懂吗？哪里会卡？哪里会有成就感？" |

### 1.2 Agent 选型与配置

```yaml
# 每个评审 agent 启动时的标准配置
agents:
  planner:
    role: "策划agent"
    system_prompt: |
      你是《暗室》的策划总监。从产品视角评审文档。
      关注：玩法机制清晰度、用户体验、关卡平衡、机制一致性。
      你代表"老板+玩家"双重身份，评审要从"老板愿不愿意付钱做这个"和"玩家会不会买单"两个角度。
    temperature: 0.3
    max_tokens: 4000

  tech_lead:
    role: "技术agent"
    system_prompt: |
      你是 Unity 2D 高级工程师。从技术实现视角评审文档。
      关注：可实现性、技术约束、性能影响、代码可维护性。
      你代表"会不会做？做不做得完？做得好不好？"三个问题。
    temperature: 0.2
    max_tokens: 4000

  art_director:
    role: "美术agent"
    system_prompt: |
      你是独立游戏美术总监。从视觉听觉视角评审文档。
      关注：美术风格统一、UI/UX 协调、音效契合度、视觉表达力。
      你代表"玩家第一眼看到什么？玩起来视觉是否舒服？"
    temperature: 0.4
    max_tokens: 4000

  player:
    role: "玩家agent"
    system_prompt: |
      你是资深独立游戏玩家（玩过 50+ 款独立解谜游戏）。从用户视角评审文档。
      关注：上手难度、乐趣点、困惑点、情绪曲线、性价比。
      你代表"我会不会买？会不会玩到一半弃坑？会不会推荐给朋友？"
    temperature: 0.5
    max_tokens: 4000
```

---

## 2. 各 Agent 的评审 Checklist

### 2.1 策划 agent（产品视角）— 18 项

| # | 维度 | Checklist |
|---|------|----------|
| P-01 | 一句话描述 | ≤30 字？能在 Steam 副标题用？看完知道在玩什么？ |
| P-02 | 目标用户 | 够具体吗（年龄/经验/动机）？不是"广大玩家"？ |
| P-03 | 差异化卖点 | 对比 2+ 同类游戏？说出"我们独特在哪"？ |
| P-04 | 核心机制 | 描述清楚"输入→处理→输出"链路？玩家一秒能理解？ |
| P-05 | 机制一致性 | 和 02-core-mechanics 的机制定义一致？无矛盾？ |
| P-06 | 关卡教学 | 新机制引入是"先用后讲"还是"先讲后用"？教学顺序合理？ |
| P-07 | 难度曲线 | 渐进式？无断崖式跳跃？有"喘息点"？ |
| P-08 | 顿悟时刻 | ≥3 个？分布在各章？有"啊哈"感？ |
| P-09 | 流程完整性 | 主流程/异常流程/存档点都覆盖？ |
| P-10 | 重试机制 | 失败有兜底？不会让玩家卡住到弃坑？ |
| P-11 | 数值平衡 | 难度公式合理？有调参空间？ |
| P-12 | 体验节奏 | 玩家情绪有起伏？30 分钟内不会疲劳？ |
| P-13 | Out of Scope | 明确写了 v1.0 不做什么？ |
| P-14 | 可验证性 | 每条设计有"验收标准"？可勾选？可观测？ |
| P-15 | 风险暴露 | 风险与开放问题诚实列出？无隐藏？ |
| P-16 | 关联引用 | 链接到上游/下游文档？形成网状结构？ |
| P-17 | 命名规范 | 文档标题、章节标题、术语统一？无错别字？ |
| P-18 | 变更可追溯 | 变更日志完整？版本号一致？ |

### 2.2 技术 agent（实现视角）— 16 项

| # | 维度 | Checklist |
|---|------|----------|
| T-01 | Unity 2D 可实现性 | 所有功能在 Unity 2D + URP 中能实现？无 3D-only 需求？ |
| T-02 | 性能预算 | 切换响应 ≤ 16ms？帧率 ≥ 60 FPS？单房间槽位数 ≤ 8？ |
| T-03 | 状态机完整性 | SwitchSlot 状态机覆盖 idle/hover/active/switching/locked 五态？ |
| T-04 | 边界情况覆盖 | ≥5 条 edge case？含动画中退出、并发切换、低帧率？ |
| T-05 | 数据结构 | 房间/槽位/预制件的数据结构定义清楚？可序列化？ |
| T-06 | Tilemap 兼容性 | 地板/墙用 Tilemap？槽位用 GameObject？分层合理？ |
| T-07 | 碰撞检测 | 静态碰撞用 TileMap Collider？动态碰撞用 2D Collider？ |
| T-08 | 动画系统 | 切换动画用 Animator + DOTween？时序可控？ |
| T-09 | 存档/读档 | 状态序列化明确？JSON 还是二进制？容错机制？ |
| T-10 | 异常处理 | 断电/崩溃/存档损坏有恢复策略？ |
| T-11 | 输入系统 | 键盘 + 手柄支持？按键映射表完整？ |
| T-12 | 音频集成 | Audio Mixer 配置？动态音量规则？3D / 2D 音频？ |
| T-13 | 编辑器扩展 | 关卡编辑器是 EditorWindow？数据导出 JSON？ |
| T-14 | 平台兼容 | Windows / Mac / Linux 都能打包？WebGL 可选？ |
| T-15 | 代码可维护性 | 系统拆分模块化（Player / SwitchSlot / Room / SaveSystem）？ |
| T-16 | 第三方依赖 | 列出所有依赖（Unity Asset / SDK / 库）？授权明确？ |

### 2.3 美术 agent（视觉听觉视角）— 15 项

| # | 维度 | Checklist |
|---|------|----------|
| A-01 | 风格一致性 | 视觉风格统一（low-poly / 像素 / 矢量）？无混杂？ |
| A-02 | 参考图 | ≥3 张参考图？标注"借鉴了 X 元素"？ |
| A-03 | 调色板 | HEX 色值完整？用途说明？出现频率？ |
| A-04 | 字体规范 | 中英文字体？字号规范？行距？颜色？ |
| A-05 | 视觉规范 | 线条粗细？对比度（WCAG AA）？光照方向？阴影规则？ |
| A-06 | HUD 布局 | 不挡游戏区？关键信息（重置/暂停）易触达？ |
| A-07 | UI 组件状态 | normal/hover/disabled/active 四态？状态可辨？ |
| A-08 | 反馈设计 | 每个操作有视觉反馈？视觉反馈明显但不喧宾夺主？ |
| A-09 | 动画节奏 | 切换动画 0.2s 合理？无卡顿？曲线流畅？ |
| A-10 | 章节氛围 | 3 章节氛围有递进（明亮→压抑→迷失）？ |
| A-11 | 资产清单 | Sprite/动画/特效清单完整？优先级清晰？ |
| A-12 | 工具链 | 制作工具（PS/Aseprite/Illustrator）明确？ |
| A-13 | 音效清单 | 时长/音量/音色/文件来源四列齐全？ |
| A-14 | BGM 清单 | 循环点/情绪标签齐全？与场景契合？ |
| A-15 | 动态音频 | 玩家位置/状态触发的音频规则明确？ |

### 2.4 玩家 agent（用户视角）— 16 项

| # | 维度 | Checklist |
|---|------|----------|
| U-01 | 一眼看懂 | 第一眼能理解游戏是什么？30 字描述能记住？ |
| U-02 | 上手难度 | 第 1 间教学清晰？5 分钟内能掌握核心操作？ |
| U-03 | 操作直观 | 按键映射符合常识（WASD 移动）？无反人类按键？ |
| U-04 | 反馈即时 | 每次操作有反馈？玩家不会怀疑"按了没反应"？ |
| U-05 | 不会卡住 | 卡住时能调出提示？不会卡到弃坑？ |
| U-06 | 进度感 | 玩家知道"我现在做到哪了"？无迷失感？ |
| U-07 | 顿悟快感 | 至少 3 个"啊哈"瞬间？分布在不同章节？ |
| U-08 | 挑战感 | 不太简单也不太难？中等玩家 30 分钟一章？ |
| U-09 | 重玩价值 | 通关后还有事做（成就/隐藏房间/挑战模式）？ |
| U-10 | 视觉舒适 | 玩 1 小时眼睛不累？对比度合适？色温合适？ |
| U-11 | 音效舒适 | 音量合适？无刺耳？BGM 不抢戏？ |
| U-12 | 情绪曲线 | 不焦虑/不无聊/不愤怒？情绪正向？ |
| U-13 | 性价比 | $4.99 值得？vs 同类价格（The Pedestrian $9.99）？ |
| U-14 | 推荐意愿 | 我会推荐给朋友吗？"这是那种应该让更多人玩到的游戏"？ |
| U-15 | 弃坑点 | 哪些设计会让玩家 10 分钟内弃坑？（教程太长？无引导？反馈不明显？） |
| U-16 | 章节记忆 | 通关后能记住"Ch2 那个隐形墙的解法"？有钩子？ |

---

## 3. 评审输出格式（JSON Schema）

### 3.1 单 Agent 评审输出

文件路径：`data/JJC-XXX/review-result-{doc_id}-{agent_role}.json`

```json
{
  "review_id": "REV-20260605-001-02-tech",
  "doc_id": "02-core-mechanics.md",
  "doc_version": "v1.0",
  "reviewer": "技术agent",
  "reviewer_role": "tech_lead",
  "review_time": "2026-06-05T01:30:00+08:00",
  "verdict": "fail",
  "score": 21,
  "score_breakdown": {
    "frontmatter": 4,
    "meta": 0,
    "config_table": 12,
    "edge_cases": 0,
    "acceptance_criteria": 0,
    "references": 0,
    "visual_aids": 5,
    "risks": 0,
    "changelog": 0
  },
  "p0_issues": [
    {
      "id": "ISSUE-001",
      "type": "missing",
      "location": "2.2 核心循环",
      "description": "缺 SwitchSlot 完整状态机，无法指导实现",
      "suggestion": "添加 Mermaid 状态图：空闲 → 玩家靠近 → 高亮 → 切换中 → 已激活",
      "evidence": "https://github.com/.../blob/main/docs/02-core-mechanics.md#L40"
    }
  ],
  "p1_issues": [
    {
      "id": "ISSUE-002",
      "type": "incomplete",
      "location": "2.5 切换动画",
      "description": "未指定动画曲线和中断处理"
    }
  ],
  "p2_issues": [
    {
      "id": "ISSUE-003",
      "type": "unclear",
      "location": "2.6 按键映射",
      "description": "建议添加手柄按键映射（Xbox/PS）"
    }
  ],
  "p3_issues": [
    {
      "id": "ISSUE-004",
      "type": "nice_to_have",
      "location": "整体",
      "description": "建议添加 GIF 演示切换效果"
    }
  ],
  "highlights": [
    "预制件类型表（7 种）结构清晰",
    "槽位类型表（4 种）和 02 互相引用一致"
  ],
  "summary": "核心机制的概念清晰（预制件、槽位、循环），但缺状态机、边界情况、性能约束、关联机制四大块。技术实现需要这四块才能开始编码。建议打回补充后重审。",
  "recommended_action": "打回重写"
}
```

### 3.2 评审聚合输出

文件路径：`data/JJC-XXX/review-result.json`

```json
{
  "task_id": "JJC-20260605-001",
  "doc_id": "02-core-mechanics.md",
  "doc_version": "v1.0",
  "review_round": 1,
  "max_rounds": 3,
  "review_time": "2026-06-05T01:45:00+08:00",
  "agents_reviewed": ["planner", "tech_lead", "art_director", "player"],
  "individual_results": [
    "data/JJC-XXX/review-result-02-core-mechanics-planner.json",
    "data/JJC-XXX/review-result-02-core-mechanics-tech_lead.json",
    "data/JJC-XXX/review-result-02-core-mechanics-art_director.json",
    "data/JJC-XXX/review-result-02-core-mechanics-player.json"
  ],
  "aggregated_issues": {
    "p0_count": 5,
    "p1_count": 8,
    "p2_count": 3,
    "p3_count": 2,
    "p0_unique": [...],
    "p1_unique": [...],
    "p2_unique": [...],
    "p3_unique": [...]
  },
  "verdict": "fail",
  "blocking_issues_exist": true,
  "next_action": "打回尚书省重写为 v1.1",
  "blocking_issues_list_path": "data/JJC-XXX/blocking-issues-v1.0.md"
}
```

### 3.3 问题严重度定义

| 等级 | 名称 | 含义 | 修复要求 | 停止评审条件 |
|------|------|------|---------|------------|
| **P0** | 阻断性问题 | 文档不完整到无法开始执行 | 必须修 | 必须清零 |
| **P1** | 重要问题 | 文档主体有但关键细节缺 | 必须修 | 必须清零 |
| **P2** | 一般问题 | 影响可执行性但不阻断 | 建议修 | 必须清零（**停止条件**） |
| **P3** | 建议 | 锦上添花 | 可选 | 可保留 |

> ⚠️ **停止条件 = 没有 P0/P1/P2 问题**。P3 不阻塞评审通过。

---

## 4. 打回重写循环机制

### 4.1 流程时序

```
T0  尚书省提交文档 v1.0
    ↓
T0+5min  朝堂议政 session 启动（4 agent 并行）
    ↓
T0+30min  4 agent 评审完成
    ↓
T0+32min  太子（朝堂议政主席）聚合评审结果
    ↓
    ┌────────────────┴────────────────┐
    │                                 │
T0+35min                          T0+35min
[无 P0/P1/P2]                    [有 P0/P1/P2]
评审通过                             打回尚书省
status → reviewed                  附问题清单
                                    轮次 +1
    │                                 │
    │                                 │
    ▼                                 ▼
    太子批准                      T0+35min+工作时长
    status → approved             尚书省重写 v1.1
    完成                            重新提交评审
                                       │
                                       ▼
                                  T+新一轮 评审
                                       │
                                  ...循环...
```

### 4.2 轮次控制

| 轮次 | 规则 |
|------|------|
| **第 1 轮** | 初始评审，发现所有问题 |
| **第 2 轮** | 整改后复评 |
| **第 3 轮** | 最后一次复评 |
| **第 3 轮后** | 如果仍有 P1+ 问题 → 标记"卡点任务"上报太子 |

### 4.3 卡点升级规则

当满足以下**任一**条件时，标记为"卡点任务"并上报太子：

1. **3 轮评审后仍有 P1+ 问题**（即问题修不完）
2. **跨轮次出现反复矛盾**（如第 1 轮说 A 重要要修，第 2 轮修完后第 3 轮说 A 反而是多余的）
3. **Agent 之间对核心机制有原则性分歧**（如策划说要"无失败"，玩家说要"必须有惩罚"）
4. **超过 5 条 P0 问题**（文档严重不达标，应直接判定不通过）

太子收到卡点报告后：
- 可裁决（按太子意见调整评审标准）
- 可拆分（将文档拆成多份，分别评审）
- 可终止（判定文档不再投入精力）

### 4.4 重写规则（给尚书省）

打回重写时，尚书省必须：

1. **逐条回应**问题清单中的每一条（P0/P1 必须回应，P2 建议回应，P3 可不回应）
2. **变更日志**记录整改内容
3. **版本号**+0.1（如 v1.0 → v1.1）
4. **同模块复用**——前一轮的"亮点"保留，不重写
5. **不引入新 P0**——新整改不应带来新的 P0 问题（避免"按下葫芦起了瓢"）

### 4.5 评分变化追踪

每次重写后，尚书省需在文档中记录评分变化：

```markdown
## 评审迭代记录

| 轮次 | 版本 | 评审时间 | 总分 | P0 | P1 | P2 | P3 | 备注 |
|------|------|----------|------|----|----|----|----|------|
| 1 | v1.0 | 2026-06-05 | 21 | 5 | 8 | 3 | 2 | 初版 |
| 2 | v1.1 | 2026-06-07 | 65 | 0 | 3 | 5 | 2 | 补状态机+边界 |
| 3 | v1.2 | 2026-06-09 | 85 | 0 | 0 | 2 | 4 | 通过 |
```

---

## 5. 自动化的实现思路

### 5.1 朝堂议政 Session 启动

```python
# scripts/review_doc.py
import json
import subprocess
from pathlib import Path

def review_document(doc_id: str, task_id: str, round_num: int = 1):
    """
    启动朝堂议政 session，4 agent 并行评审指定文档
    """
    # 1. 加载文档
    doc_path = Path(f"docs/{doc_id}")
    doc_content = doc_path.read_text()
    doc_version = extract_version(doc_content)  # 从 frontmatter 读取
    
    # 2. 启动 4 个 agent 并行评审
    agents = ["planner", "tech_lead", "art_director", "player"]
    results = {}
    for agent in agents:
        result = run_agent_review(
            agent=agent,
            doc_id=doc_id,
            doc_content=doc_content,
            doc_version=doc_version
        )
        results[agent] = result
        # 保存单 agent 结果
        result_path = Path(f"data/{task_id}/review-result-{doc_id}-{agent}.json")
        result_path.parent.mkdir(parents=True, exist_ok=True)
        result_path.write_text(json.dumps(result, indent=2, ensure_ascii=False))
    
    # 3. 聚合结果
    aggregated = aggregate_results(doc_id, doc_version, results, round_num)
    aggregated_path = Path(f"data/{task_id}/review-result.json")
    aggregated_path.write_text(json.dumps(aggregated, indent=2, ensure_ascii=False))
    
    # 4. 判定 verdict
    if aggregated["blocking_issues_exist"]:
        return {"verdict": "fail", "next_action": "打回重写"}
    else:
        return {"verdict": "pass", "next_action": "太子批准 → approved"}


def run_agent_review(agent: str, doc_id: str, doc_content: str, doc_version: str) -> dict:
    """
    调用单个 agent 评审文档，返回结构化 JSON 结果
    """
    checklist = load_checklist(agent)  # 加载该 agent 的 checklist
    prompt = build_prompt(agent, doc_id, doc_content, doc_version, checklist)
    response = call_claude(prompt, agent_config[agent])
    return parse_review_response(response, agent)


def aggregate_results(doc_id, doc_version, results, round_num):
    """
    聚合 4 个 agent 的评审结果
    """
    all_p0, all_p1, all_p2, all_p3 = [], [], [], []
    for agent, result in results.items():
        all_p0.extend(result["p0_issues"])
        all_p1.extend(result["p1_issues"])
        all_p2.extend(result["p2_issues"])
        all_p3.extend(result["p3_issues"])
    
    blocking = len(all_p0) > 0 or len(all_p1) > 0 or len(all_p2) > 0
    
    return {
        "doc_id": doc_id,
        "doc_version": doc_version,
        "review_round": round_num,
        "agents_reviewed": list(results.keys()),
        "aggregated_issues": {
            "p0_count": len(all_p0),
            "p1_count": len(all_p1),
            "p2_count": len(all_p2),
            "p3_count": len(all_p3)
        },
        "blocking_issues_exist": blocking,
        "verdict": "fail" if blocking else "pass",
        "next_action": "打回重写" if blocking else "太子批准"
    }
```

### 5.2 朝堂议政 Session 的 Prompt 模板

```markdown
# 朝堂议政 · 文档评审

## 上下文
- 任务 ID: JJC-{task_id}
- 文档 ID: {doc_id}
- 文档版本: {doc_version}
- 评审轮次: {round_num} / 3
- 你的角色: {agent_role}

## 文档内容
```markdown
{doc_content}
```

## 你的评审 Checklist
{checklist_for_this_agent}

## 输出要求
请按以下 JSON 格式输出你的评审结果：

```json
{
  "verdict": "pass | fail | conditional",
  "score": <0-100>,
  "p0_issues": [{"id", "type", "location", "description", "suggestion"}],
  "p1_issues": [...],
  "p2_issues": [...],
  "p3_issues": [...],
  "highlights": [...],
  "summary": "...",
  "recommended_action": "通过 | 打回重写 | 条件通过"
}
```

## 评审原则
1. **诚实** — 发现问题就指出来，不要"放水"
2. **可执行** — 每条 issue 都要有"具体改什么"建议
3. **可观测** — 验收标准要能勾选、能验证
4. **分级准确** — 阻断性问题打 P0，重要的打 P1，建议的打 P2，锦上添花打 P3
```

### 5.3 尚书省自动重写流程

```python
# scripts/auto_rewrite.py
def auto_rewrite(doc_id, review_result_path, prev_version):
    """
    根据 review-result.json 自动重写文档
    """
    review = load_json(review_result_path)
    
    # 1. 解析所有 P0/P1/P2 issues
    blocking_issues = review["p0_unique"] + review["p1_unique"] + review["p2_unique"]
    
    # 2. 加载原文档
    doc_path = Path(f"docs/{doc_id}")
    doc = doc_path.read_text()
    
    # 3. 对每个 issue 调用 agent 生成修复
    for issue in blocking_issues:
        fix_prompt = build_fix_prompt(issue, doc)
        fix = call_claude(fix_prompt, role="editor")
        doc = apply_fix(doc, issue["location"], fix)
    
    # 4. 升级版本号
    new_version = bump_version(prev_version, "+0.1")
    doc = update_frontmatter(doc, version=new_version)
    
    # 5. 加变更日志
    doc = add_changelog_entry(doc, prev_version, new_version, blocking_issues)
    
    # 6. 写回
    doc_path.write_text(doc)
    
    return new_version
```

### 5.4 完整工作流（伪代码）

```python
# scripts/review_workflow.py
def review_workflow(task_id, doc_id, max_rounds=3):
    """
    完整的多 agent 评审工作流
    """
    round_num = 1
    while round_num <= max_rounds:
        print(f"\n=== 第 {round_num} 轮评审 ===\n")
        
        # 1. 评审
        result = review_document(doc_id, task_id, round_num)
        print(f"verdict: {result['verdict']}")
        print(f"next_action: {result['next_action']}")
        
        # 2. 判定
        if result["verdict"] == "pass":
            print("✅ 评审通过")
            mark_as_approved(doc_id)
            return {"status": "approved", "round": round_num}
        
        # 3. 打回重写
        print(f"❌ 打回重写（第 {round_num} 轮）")
        review_path = f"data/{task_id}/review-result.json"
        prev_version = extract_version(doc_id)
        new_version = auto_rewrite(doc_id, review_path, prev_version)
        print(f"已重写: {prev_version} → {new_version}")
        
        # 4. 检查轮次
        if round_num >= max_rounds:
            print(f"⚠️ 已达 {max_rounds} 轮上限，标记卡点任务")
            mark_as_blocked(task_id, doc_id, round_num)
            notify_taizi(f"任务 {task_id} 文档 {doc_id} 评审卡点，请裁决")
            return {"status": "blocked", "round": round_num}
        
        round_num += 1
```

---

## 6. 评审日历（建议节奏）

### 6.1 12 个文档的评审顺序

按"先核心后周边"原则：

| 阶段 | 文档 | 评审时间 | 优先级 |
|------|------|---------|--------|
| **第 1 周** | 01-overview, 02-core-mechanics | T+1~2 天 | P0（核心玩法） |
| **第 1 周** | 03-level-design, 04-gameplay-flow | T+3~5 天 | P0（关卡+流程） |
| **第 2 周** | 05-numerical, 06-player-experience | T+6~9 天 | P1（数值+体验） |
| **第 2 周** | 07-failure-retry, 08-ui-ux | T+10~12 天 | P1（重试+UI） |
| **第 3 周** | 09-audio, 12-art-style | T+13~16 天 | P1（美术+音频） |
| **第 3 周** | 10-roadmap, 11-release | T+17~19 天 | P2（路线图+发布） |

### 6.2 每天评审节奏

```
09:00 - 尚书省提交文档
09:30 - 朝堂议政启动，4 agent 并行
10:30 - 评审完成
11:00 - 太子聚合 + 判定
11:30 - 尚书省收到反馈
14:00 - 尚书省重写完成
15:00 - 触发下一轮评审
```

---

## 7. 评审质量保证

### 7.1 评审本身的审计

为避免"评审放水"，引入**交叉评审机制**：

- 每个 agent 的评审结果，由太子（人类）抽样 10% 复审
- 如果发现某 agent 经常给高分（>80% 通过率）但实际文档质量差 → 调整该 agent 的 prompt
- 如果发现某 agent 经常给低分（<20% 通过率）但实际文档质量好 → 检查是否是"过度严格"

### 7.2 Prompt 迭代

每月回顾一次 4 个 agent 的 prompt，根据：
- 误报率（P0 问题被回退）
- 漏报率（评审通过后用户报告问题）
- 评分一致性（不同 agent 对同一文档的评分相关性）

迭代 prompt，提高评审质量。

### 7.3 评审档案

所有评审结果永久保存到 `data/JJC-XXX/`，形成"评审档案"：
- 6 个月后可回溯"为什么这个设计决策是这么做的"
- 1 年后可分析"评审 agent 的判断是否准确"
- 2 年后可作为"产品质量预测模型"的训练数据

---

## 8. 特殊场景

### 8.1 紧急发布（绕过评审）

如果项目紧急（如 Steam Festival 截稿），可以：
1. 太子授权"conditional pass"
2. 文档状态 `status: reviewed`（非 approved）
3. 必须在发布后 1 周内补完评审

### 8.2 大版本变更

当 v1.0 → v2.0 时：
1. 必须重跑全部 12 个文档的评审
2. 不允许"v1.0 通过了所以 v2.0 也通过"
3. 重新计算评分

### 8.3 文档废弃

当文档被新文档替代时：
1. 旧文档 `status: deprecated`
2. 新文档继承旧文档的"亮点"和"评分"
3. 废弃文档仍保留在 git 历史中（不删）

---

## 9. 流程检查清单（自检用）

文档评审前自检：

- [ ] 文档已通过 `DOC-STANDARD.md` 的自审（每项必填字段已勾）
- [ ] 文档已 `git commit` 并 push
- [ ] 文档版本号 ≥ v1.0
- [ ] 文档状态为 `draft`（等待评审）
- [ ] 上一轮评审的 blocking issues 已修复

文档评审后自检：

- [ ] 4 个 agent 评审结果都写入 `data/JJC-XXX/`
- [ ] 聚合结果写入 `data/JJC-XXX/review-result.json`
- [ ] 判定 verdict（pass/fail/conditional）
- [ ] 如打回，blocking issues 清单已发给尚书省
- [ ] 尚书省收到反馈并启动重写

---

## 10. 变更日志

| 日期 | 版本 | 变更人 | 内容 |
|------|------|--------|------|
| 2026-06-05 | v1.0 | 尚书省 | 初版评审流程设计（4 agent 角色 + checklist + 自动化） |

---

**最后更新：** 2026-06-05
**文档版本：** v1.0
**状态：** draft
**关联文档：** [DOC-STANDARD.md](./DOC-STANDARD.md) / [AUDIT-REPORT.md](./AUDIT-REPORT.md)