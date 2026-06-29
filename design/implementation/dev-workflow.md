---
title: 《暗室》开发者工作流 (Developer Workflow)
doc_id: DESIGN-anzhong-implementation-dev-workflow
parent: design/implementation/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》开发者工作流 (Developer Workflow)

> **一句话定位：** Git Flow (main/develop/feature) + Conventional Commits + Code Review (≥ 1 人) + Pre-commit Hooks + AGENTS.md 三省六部协作 的端到端开发者协议。

## 目的 (Purpose)

本文档是《暗室》**开发者工作流层**的**唯一权威基线**。它向：

- **新加入工程师** — 30 分钟看懂"怎么 clone、怎么改、怎么 commit、怎么 push、怎么 PR、怎么 review、怎么 merge"
- **Code Reviewer** — 评审清单 + 通过标准
- **太子 / 尚书省** — 治理规则，与 `AGENTS.md` 三省六部协作对齐
- **CI/CD** — Pre-commit Hooks + Branch 保护规则
- **历史追溯** — Conventional Commits + 变更日志自动生成

**本版本（v1.0）的目的：** 把"1 人 Solo 为主 + 未来扩展"的 Git Flow + Conventional Commits + Code Review + Pre-commit Hooks + 跨角色协作——**第一次**用"分支策略 + 提交规范 + 评审清单 + 治理规则" 4 维度统一描述，作为 phase4 实施的"开发者协议"。

## 范围 (Scope)

### 包含

- **Git Flow**：main / develop / feature/* / hotfix/* / release/* 5 种分支
- **Conventional Commits**：feat/fix/docs/refactor/test/chore/perf 7 种类型
- **Code Review**：PR 模板 + 评审清单 + 通过标准 + 审批权限
- **Pre-commit Hooks**：Husky + lint-staged + commitlint
- **AGENTS.md 三省六部**：中书省 / 门下省 / 尚书省 + 吏户礼兵刑工 6 部
- **冲突解决**：merge conflict 处理流程

### 不包含 (Out of Scope)

- 编码规范 (StyleCop / .editorconfig) → 见 [`coding-standards.md`](./coding-standards.md)
- CI/CD pipeline → 见 [`ci-cd.md`](./ci-cd.md)
- 测试用例 → 见 [`test-strategy.md`](./test-strategy.md)
- 文档评审 → 见 `ce-doc-review`

## 一句话描述 (One-liner)

> **"Git Flow × Conventional Commits × Code Review × 三省六部，1 人 Solo + 未来扩展的统一开发者协议。"**

## 1. Git Flow 分支策略 (Git Flow Branch Strategy)

### 1.1 5 种分支

| 分支 | 用途 | 生命周期 | 保护 |
|------|------|---------|:----:|
| **main** | 稳定代码 (= 已发布 / Release) | 永久 | ✅ 受保护 |
| **develop** | 开发主线 (集成 feature) | 永久 | ✅ 受保护 |
| **feature/* | 新功能开发 | 短 (1-7 天) | ❌ |
| **hotfix/* | 紧急修复 (main 分支) | 极短 (≤ 24h) | ❌ |
| **release/* | 发布准备 (v1.0.0-rc.1) | 短 (1-3 天) | ❌ |

### 1.2 分支命名规范

| 类型 | 模式 | 示例 |
|------|------|------|
| **Feature** | `feature/<scope>-<desc>` | `feature/save-system-aes-encryption` |
| **Hotfix** | `hotfix/<version>-<desc>` | `hotfix/v1.0.1-crash-on-room-3-6` |
| **Release** | `release/<version>` | `release/v1.0.0-rc.1` |
| **Bug** | `bugfix/<id>-<desc>` | `bugfix/BUG-042-conditional-slot` |
| **Doc** | `docs/<scope>-<desc>` | `docs/update-readme` |

**Scope 取值：**

- `core` / `switchslot` / `room` / `player` / `ui` / `audio` / `savesystem` / `input` / `hint` / `telemetry` / `localization` / `settings` / `asset` / `devops` / `ci`
- `docs` / `tests` / `tools`

### 1.3 分支工作流

```
main (v1.0.0)
  │
  ├── hotfix/v1.0.1-crash-on-room-3-6 (from main)
  │     ↓ PR → main
  │
develop (W01-W12 持续集成)
  │
  ├── feature/save-system-aes (from develop)
  │     ↓ PR → develop
  │
  ├── release/v1.0.0-rc.1 (from develop)
  │     ↓ 测试 + Bug 修复
  │     ↓ PR → main + develop
  │
main (v1.0.0-rc.1 → v1.0.0 → v1.0.1)
```

### 1.4 分支保护规则 (Branch Protection)

| 规则 | main | develop |
|------|:----:|:-------:|
| **Require PR** | ✅ | ✅ |
| **Require 1 approval** | ✅ | ✅ |
| **Require status checks (CI pass)** | ✅ | ✅ |
| **Require linear history** | ✅ | ❌ |
| **No force push** | ✅ | ✅ |
| **No deletion** | ✅ | ✅ |

## 2. Conventional Commits 规范

### 2.1 7 种提交类型

| 类型 | 用途 | 示例 |
|------|------|------|
| **feat** | 新功能 | `feat(switchslot): add CycleSlot with 4 options` |
| **fix** | Bug 修复 | `fix(savesystem): resolve corrupted save fallback` |
| **docs** | 文档变更 | `docs(readme): update 7-platform matrix` |
| **refactor** | 重构（无功能变化） | `refactor(core): extract event bus from GameLoop` |
| **test** | 测试 | `test(room): add P0-001 difficulty validation tests` |
| **chore** | 杂项 | `chore(deps): bump Unity to 2022.3.20f1` |
| **perf** | 性能优化 | `perf(audio): implement AudioSource pool` |

### 2.2 提交消息格式

```
<type>(<scope>): <subject>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

**Subject 规则：**
- ≤ 50 字符
- 首字母小写
- 不加句号
- 命令式（"add" 而非 "added"）

**Body 规则：**
- 解释"什么" + "为什么"，不解释"如何"
- 多行，每行 ≤ 72 字符
- 与 Subject 间隔 1 个空行

**Footer 规则：**
- 引用 Issue：`Refs: #123` / `Closes: #456`
- BREAKING CHANGE：`BREAKING CHANGE: <description>`
- Co-authored-by：`Co-authored-by: Name <email>`

### 2.3 提交消息示例

#### feat 示例

```
feat(switchslot): add CycleSlot with 4 options

Implement CycleSlot class supporting 3-4 option rotation
for Ch1 room 1-2 (double door puzzle).

- Add CycleSlot.cs (3-4 option rotation)
- Update SwitchSlotStateMachine to handle Cycle state
- Add unit tests (3-4 options, clockwise/counter-clockwise)
- Update I/O Spec with Cycle slot visual feedback

Refs: #42
Refs: 02-core-mechanics-v2.md §3.1
```

#### fix 示例

```
fix(savesystem): resolve corrupted save fallback

Corrupted savegame.json would crash SaveSystem.LoadAsync.
Now falls back to .bak → fresh start, with user notification.

- Add try-catch around BinaryFormatter.Deserialize
- Add backup load retry (3 backups)
- Add UI notification "存档损坏，从头开始"
- Add unit test CorruptedSaveTest

Closes: #128
```

#### docs 示例

```
docs(design): create implementation design document - phase3 final

Create 8 files in design/implementation/:
- README.md (overview + 14 modules + full-chain wrap-up)
- module-spec.md (14 modules + 6 cross-cutting)
- test-strategy.md (test pyramid + 5 types + P0-001)
- ci-cd.md (GitHub Actions + 7 platforms)
- dev-workflow.md (Git Flow + Conventional Commits)
- coding-standards.md (C# 9 / Unity 2022 LTS / .NET 8)
- deployment-runbook.md (7 platforms + rollback + monitoring)
- risks-mitigation.md (15 P0 + P0-001 tracking)

Refs: ANZHONG-16
Refs: design/data/p0-001-tracking.md (P0-001 强关联)
```

## 3. Code Review 规范

### 3.1 PR 模板 (Pull Request Template)

```markdown
## 描述 (Description)
<!-- 简述变更内容 -->

## 关联 Issue / 任务 (Related Issue / Task)
<!-- 关联的 GitHub Issue / 任务 ID -->
- Closes: #
- Refs: #

## 关联文档 (Related Documents)
<!-- 关联的 GDD / design 文档 -->
- docs/XX-v2.md §
- design/XX/README.md

## 变更类型 (Type of Change)
- [ ] feat (新功能)
- [ ] fix (Bug 修复)
- [ ] docs (文档)
- [ ] refactor (重构)
- [ ] test (测试)
- [ ] perf (性能)
- [ ] chore (杂项)
- [ ] BREAKING CHANGE (不兼容变更)

## 测试 (Testing)
- [ ] 单元测试通过 (覆盖率 ≥ 70%)
- [ ] 集成测试通过 (PlayMode)
- [ ] 手动测试通过 (描述步骤)
- [ ] 性能预算未退化 (60 FPS / 512MB)

## 评审清单 (Review Checklist)
- [ ] 代码符合 `coding-standards.md`
- [ ] 无 P0/P1 Bug
- [ ] 公共 API 有单元测试
- [ ] 文档已更新 (GDD / design)
- [ ] 无 PII / 敏感信息泄露
- [ ] 无 force push / rebase main
- [ ] 提交消息符合 Conventional Commits

## 截图 / 视频 (Screenshots / Videos)
<!-- 如有视觉变更，请附截图或视频 -->

## P0-001 检查 (P0-001 Check)
- [ ] 难度字段未编造数据
- [ ] Room.validate() / Level.validate() 自我保护
- [ ] UI 显示"待配置"如适用
- [ ] 引用 design/data/p0-001-tracking.md
```

### 3.2 评审清单 (Review Checklist)

| 维度 | 检查项 | 严重度 |
|------|--------|:------:|
| **正确性** | 代码逻辑正确 + 边界条件处理 | P0 |
| **可读性** | 命名清晰 + 注释充分 + 函数 ≤ 50 行 | P1 |
| **可维护性** | 无重复代码 + 单一职责 + 依赖倒置 | P1 |
| **性能** | 60 FPS / 512MB / 50 DrawCall 不退化 | P0 |
| **安全** | 无 SQL 注入 / XSS / 硬编码密钥 | P0 |
| **测试** | 单元测试覆盖率 ≥ 70% + 关键路径 100% | P0 |
| **文档** | GDD / design / README 同步更新 | P1 |
| **P0-001** | 难度字段不编造 + 自我保护 + 引用 tracking | P0 |
| **Conventional Commits** | 消息格式正确 + scope 准确 | P2 |
| **AGENTS.md** | 跨角色协作 (三省六部) 已通知 | P2 |

### 3.3 评审通过标准 (Approval Criteria)

- **至少 1 个 Approval** （中书省 / 尚书省 / 门下省 任一）
- **所有 P0 检查项通过**
- **CI 全绿** (Lint / Unit Test / Integration Test / Build)
- **Codecov 覆盖率** 不下降 > 5%
- **无 P0/P1 Bug 标记**
- **无 force push 痕迹** (`git log --merges` 检查)

### 3.4 审批权限 (Approval Authority)

| 角色 | 可审批 | 不可审批 |
|------|--------|----------|
| **太子** | 全部 | — |
| **中书省** | 全部 + 自审 (需门下省复核) | 门下省专属 |
| **门下省** | 全部 + 复核中书省 | 尚书省专属 |
| **尚书省** | 实施层 (code/CI) | 战略层 (路线图/预算) |
| **吏部** | 人员 + 权限 | — |
| **户部** | 预算 + 资产 | — |
| **礼部** | 文档 + 命名 | — |
| **兵部** | 安全 + 应急 | — |
| **刑部** | 合规 + 审计 | — |
| **工部** | 工具 + 平台 | — |

## 4. Pre-commit Hooks

### 4.1 Husky 配置 (`.husky/`)

```bash
# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

# Lint staged files
npx lint-staged

# Markdown Lint
npx markdownlint '**/*.md' --ignore node_modules

# StyleCop (C#)
python tools/ci/lint_csharp.py --staged
```

```bash
# .husky/commit-msg
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx --no-install commitlint --edit "$1"
```

### 4.2 lint-staged 配置 (`.lintstagedrc`)

```json
{
  "*.cs": [
    "dotnet format --verify-no-changes",
    "python tools/ci/lint_csharp.py --file"
  ],
  "*.md": [
    "npx markdownlint --fix",
    "npx cspell --no-progress"
  ],
  "*.{yml,yaml}": [
    "npx yamllint"
  ]
}
```

### 4.3 commitlint 配置 (`commitlint.config.js`)

```javascript
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'type-enum': [2, 'always', [
      'feat', 'fix', 'docs', 'refactor', 'test', 'chore', 'perf',
      'revert', 'build', 'ci'
    ]],
    'scope-enum': [2, 'always', [
      'core', 'switchslot', 'room', 'player', 'ui', 'audio',
      'savesystem', 'input', 'hint', 'telemetry', 'localization',
      'settings', 'asset', 'devops', 'ci', 'docs', 'tests', 'tools',
      'deps', 'release'
    ]],
    'subject-max-length': [2, 'always', 50],
    'body-max-line-length': [2, 'always', 72]
  }
};
```

## 5. AGENTS.md 三省六部协作

> 详见 `AGENTS.md` (中书省) + `/Users/yzj/.openclaw/workspace/AGENTS.md` (尚书省/门下省)

### 5.1 三省 (Three Departments)

| 部门 | 职责 | 协作 |
|------|------|------|
| **中书省** | 文档起草 (GDD / design) | 起草后交门下省复核 |
| **门下省** | 复核 + 封驳 | 复核后交尚书省执行 |
| **尚书省** | 实施 (code / CI / release) | 执行后向门下省汇报 |

### 5.2 六部 (Six Ministries)

| 部 | 职责 | 实施期关联 |
|----|------|----------|
| **吏部** | 人事 + 权限 | 开发者权限 + Approval 角色 |
| **户部** | 财务 + 资产 | 预算 + Secrets + 7 平台费用 |
| **礼部** | 礼仪 + 命名 | 命名规范 + 文档 + 提交规范 |
| **兵部** | 国防 + 应急 | 安全 + 灾备 + 应急计划 |
| **刑部** | 司法 + 合规 | 合规 + GDPR + 审计 + License |
| **工部** | 工程 + 工具 | 工具链 + CI/CD + 构建脚本 |

### 5.3 协作流程 (Workflow)

```
1. 中书省起草 (GDD / design)
   ↓ 提交 PR (待复核)
2. 门下省复核 (评审清单 + 6 维度自查)
   ↓ 通过 / 封驳
3. 尚书省执行 (code / CI / release)
   ↓ 提交 PR (待 merge)
4. 门下省复核 code
   ↓ 通过
5. Merge to develop / main
   ↓
6. 吏部更新人员 / 礼部更新文档 / 户部更新预算 / 工部更新工具 / 兵部更新应急 / 刑部更新合规
```

## 6. 冲突解决 (Conflict Resolution)

### 6.1 Merge Conflict 处理

```bash
# 1. 拉取最新 develop
git checkout develop
git pull origin develop

# 2. 切到 feature 分支
git checkout feature/save-system-aes

# 3. rebase develop (推荐) 或 merge
git rebase develop
# 或
git merge develop

# 4. 解决冲突
# 打开冲突文件，手动编辑
# 完成后:
git add .
git rebase --continue
# 或
git commit

# 5. 推送
git push origin feature/save-system-aes --force-with-lease
```

### 6.2 冲突解决原则

| 原则 | 说明 |
|------|------|
| **沟通优先** | 冲突前先与对方沟通 |
| **保持功能** | 保留双方功能，不丢失 |
| **小步提交** | 频繁 commit + 频繁 rebase |
| **避免大改** | feature 分支 ≤ 7 天 |
| **测试覆盖** | 解决冲突后必须跑测试 |

## 7. 关联文档 (Cross-References)

### 上游 (本文档依赖)

- [`../README.md`](../README.md) — 实施期总览
- [`./coding-standards.md`](./coding-standards.md) — 编码规范（Code Review 检查项）
- [`./ci-cd.md`](./ci-cd.md) — CI/CD 流水线（分支保护）
- [`./test-strategy.md`](./test-strategy.md) — 测试策略（PR 合并要求）
- `AGENTS.md` (中书省) — 三省六部协作
- `docs/git-flow.md` (尚书省) — Git Flow 详细

### 下游 (本文档被依赖)

- `.git/hooks/` Git Hooks
- `.github/pull_request_template.md` PR 模板
- `.github/CODEOWNERS` 审批权限
- `.husky/` Husky Hooks
- `.lintstagedrc` lint-staged 配置
- `commitlint.config.js` commitlint 配置

## 8. 验收标准 (Acceptance Criteria)

- [x] **AC-01** Frontmatter 7 字段完整
- [x] **AC-02** Git Flow 5 种分支 (main/develop/feature/hotfix/release)
- [x] **AC-03** Conventional Commits 7 类型 + 格式规范
- [x] **AC-04** Code Review 清单 + 模板 + 审批权限
- [x] **AC-05** Pre-commit Hooks (Husky + lint-staged + commitlint)
- [x] **AC-06** AGENTS.md 三省六部协作流程
- [x] **AC-07** 冲突解决流程
- [x] **AC-08** **强 P0-001 跟踪** — Code Review 清单含 P0-001 项
- [x] **AC-09** 分支保护规则
- [x] **AC-10** 文档总行数 ≥ 400 行 (实际 ~500 行)

## 9. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent (ANZHONG-16) 创建。**新建**：Git Flow 5 分支 (main/develop/feature/hotfix/release) + 分支保护 + Conventional Commits 7 类型 (feat/fix/docs/refactor/test/chore/perf) + 提交消息格式 + Code Review 模板 + 10 维度评审清单 + 审批权限 (三省六部) + Pre-commit Hooks (Husky + lint-staged + commitlint) + 三省六部协作流程 + 冲突解决 + **强 P0-001 跟踪** (Code Review 清单含 P0-001 项)。**全链 16/16 收官 🏆** 第 5/8 文件。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft (等待 ce-doc-review 评审)
**全链状态：** 16/16 收官 🏆
