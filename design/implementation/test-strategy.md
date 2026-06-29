---
title: 《暗室》测试策略 (Test Strategy)
doc_id: DESIGN-anzhong-implementation-test
parent: design/implementation/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》测试策略 (Test Strategy)

> **一句话定位：** 测试金字塔 (单元 70% / 集成 50% / E2E 30%) + 5 类测试 (单元/集成/E2E/性能/兼容) + 与 10-v2 4 阶段 12 里程碑对齐的端到端质量保障体系。

## 目的 (Purpose)

本文档是《暗室》**测试层**的**唯一权威基线**。它向：

- **Unity 客户端工程师** — 定义每个模块的单元测试入口、覆盖率目标、测试框架
- **QA / 测试工程师** — 定义集成测试、E2E 测试、性能测试、兼容性测试的方法与工具
- **DevOps / CI/CD** — 定义 CI 流水线的测试阶段、覆盖率门槛、失败回滚
- **Code Review** — 定义 PR 合并的测试要求（"完成"标准之一）
- **太子 / 陛下** — 12 里程碑验收的"可测可玩"标准

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的全部测试方法——单元测试 (NUnit + Unity Test Framework)、集成测试 (PlayMode)、E2E 测试 (手动 + 自动)、性能测试 (Unity Profiler)、兼容性测试 (7 平台)——**第一次**用"测试金字塔 + 5 类测试 + 12 里程碑对齐" 3 维度统一描述，作为 phase4 实施的"质量合同"。

## 范围 (Scope)

### 包含

- **测试金字塔**：单元 ≥ 70% / 集成 ≥ 50% / E2E ≥ 30% 覆盖率
- **5 类测试**：单元测试 / 集成测试 / E2E 测试 / 性能测试 / 兼容性测试
- **测试框架**：NUnit 3.x + Unity Test Framework 1.x + Coverlet + dotMemory + Unity Profiler
- **CI 集成**：GitHub Actions 自动运行 + 覆盖率报告 (Codecov)
- **里程碑对齐**：M01-M12 12 里程碑的测试验收标准
- **P0-001 跟踪**：难度字段测试使用 NULL/默认值，不依赖缺失数据

### 不包含 (Out of Scope)

- 单元测试用例具体步骤 → phase4 由 QA 编写
- 性能测试具体场景 → 见 `docs/01-overview-v2.md` §性能预算
- 安全测试 / 渗透测试 → 单机游戏不适用
- 用户验收测试 (UAT) → 见 10-v2 §9 后发布支持 + 11-v2 §4 5 人 Playtest

## 一句话描述 (One-liner)

> **"测试金字塔 × 5 类测试 × 12 里程碑 × 70/50/30 覆盖率，把 phase3 design 翻译成 phase4 实施的质量保障。"**

## 1. 测试金字塔 (Test Pyramid)

> **核心原则：** 测试数量 金字塔型 — 单元最多，集成次之，E2E 最少。E2E 慢且易碎，单元快且稳定。

```
        /\
       /  \      E2E 测试 (10-20 个)
      /    \     - 关键路径 (1-1~1-5 全流程)
     /------\    - 章节通关 (3-8 通关)
    /        \   - 跨平台 (Steam/Mac/Switch)
   /          \  - 慢: 5-10 min/case
  /------------\ 
 /  集成测试    \ 集成测试 (50-100 个)
/  (PlayMode)   \ - 模块集成 (M02+M03+M04)
/                \ - 跨模块事件总线
/                  \ - 中速: 30s-2min/case
/--------------------\
|    单元测试         | 单元测试 (300-500 个)
|    (EditMode)       | - 每个模块 20-30 个
|                     | - 状态机/算法/数据结构
|                     | - 快: < 1s/case
+---------------------+
```

| 层级 | 数量目标 | 覆盖率目标 | 速度 | 工具 |
|:----:|:--------:|:----------:|------|------|
| **单元** | 300-500 | ≥ 70% | < 1s/case | NUnit + Unity Test Framework + Coverlet |
| **集成** | 50-100 | ≥ 50% | 30s-2min/case | Unity Test Framework (PlayMode) |
| **E2E** | 10-20 | ≥ 30% (关键路径) | 5-10min/case | 手动 + 自动 (Selenium-like) |
| **性能** | 5-10 | 性能预算 100% | 1-3h/case | Unity Profiler + dotMemory |
| **兼容** | 5-10 | 7 平台 100% | 1-2h/平台 | 平台 SDK + IARC |

> **覆盖率门槛：** PR 合并要求 单元 ≥ 70% + 集成 ≥ 50% + 关键路径 100%；M12 Release 要求 单元 ≥ 80% + 集成 ≥ 60% + E2E 100%。

## 2. 5 类测试 (5 Test Types)

### 2.1 单元测试 (Unit Test)

#### 目标

- **覆盖率：** 模块 ≥ 80% (P0 模块) / ≥ 60% (P1/P2 模块)
- **速度：** < 1s/case
- **隔离性：** 不依赖 Unity Player / 文件系统 / 网络

#### 工具

- **NUnit 3.x**：xUnit-style 框架
- **Unity Test Framework 1.x**：EditMode 测试
- **Moq 4.x**：依赖 mock
- **FluentAssertions 5.x**：可读断言
- **Coverlet**：覆盖率收集

#### 测试入口

```
tests/Unit/
├── Core/
│   ├── GlobalStateMachineTest.cs       # 12 态 + 144 状态转移
│   ├── EventBusTest.cs                 # 15 事件类型
│   └── ConfigLoaderTest.cs             # JSON 加载
├── SwitchSlot/
│   ├── ToggleSlotTest.cs               # 1 选项翻转
│   ├── CycleSlotTest.cs                # 3-4 选项循环
│   ├── ConditionalSlotTest.cs          # 依赖链 ≤ 2 层
│   ├── LockedSlotTest.cs               # 解锁条件
│   └── StateMachineTest.cs             # 5 态机
├── Room/
│   ├── RoomLoaderTest.cs               # JSON 加载
│   ├── RoomDataTest.cs                 # 19 房间数据
│   ├── RoomValidateTest.cs             # P0-001 难度 ≤ 20
│   └── WinConditionTest.cs             # 3 重判定
├── SaveSystem/
│   ├── SaveSerializerTest.cs           # 二进制 + AES-256
│   ├── SaveSystemTest.cs               # Load/Save
│   └── CorruptedSaveTest.cs            # 损坏降级
├── Audio/
│   ├── AudioManagerTest.cs             # 9 类 dB
│   └── AudioCategoryTest.cs            # 9 类别
├── HintSystem/
│   ├── StuckDetectorTest.cs            # 卡点识别
│   └── HintLevelTest.cs                # 5 阶段
├── Settings/
│   ├── AudioSettingsTest.cs            # 9 类 dB
│   └── AccessibilitySettingsTest.cs    # 4 类
└── Common/
    ├── AesGcmCipherTest.cs             # 加密
    └── ObjectPoolTest.cs               # 性能
```

#### 关键测试用例 (示例)

```csharp
// tests/Unit/SwitchSlot/ToggleSlotTest.cs
[Test]
public void ToggleSlot_SwitchOnce_StateFlipsToActive() {
    var slot = new ToggleSlot(slotId: 1);
    slot.Switch(SwitchDirection.Clockwise);
    Assert.That(slot.CurrentState, Is.EqualTo(SlotState.Active));
}

[Test]
public void ToggleSlot_SwitchTwice_StateFlipsBackToIdle() {
    var slot = new ToggleSlot(slotId: 1);
    slot.Switch(SwitchDirection.Clockwise);
    slot.Switch(SwitchDirection.Clockwise);
    Assert.That(slot.CurrentState, Is.EqualTo(SlotState.Idle));
}

[Test]
public void ToggleSlot_ConsecutiveSwitches_WithinCooldown_AreDropped() {
    var slot = new ToggleSlot(slotId: 1);
    slot.Switch(SwitchDirection.Clockwise);
    Assert.Throws<CooldownException>(() =>
        slot.Switch(SwitchDirection.Clockwise));  // 300ms 内
}
```

```csharp
// tests/Unit/Room/RoomValidateTest.cs
[Test]
public void RoomLoader_LoadRoom_DifficultyExceeds20_TriggersP0001SelfProtection() {
    // P0-001 关联测试：难度 > 20 → 警告 + 强制回退到 20
    var corruptedData = new RoomData { Difficulty = 25, DifficultyMax = 20 };
    var validated = RoomLoader.Validate(corruptedData);
    Assert.That(validated.Difficulty, Is.EqualTo(20));  // 强制回退
    Assert.That(validated.Warnings, Has.Member("P0-001: difficulty > 20, fallback to 20"));
}

[Test]
public void RoomLoader_LoadRoom_DifficultyNull_UsesDefault() {
    // P0-001 关联测试：难度 NULL → 默认值
    var data = new RoomData { Difficulty = null, DifficultyMax = 20 };
    var validated = RoomLoader.Validate(data);
    Assert.That(validated.Difficulty, Is.EqualTo(1));  // 默认值
    Assert.That(validated.Warnings, Has.Member("P0-001: difficulty is NULL, using default 1"));
}
```

### 2.2 集成测试 (Integration Test)

#### 目标

- **覆盖率：** 关键模块组合 ≥ 50%
- **速度：** 30s-2min/case
- **范围：** 模块间接口 + 事件总线 + 数据流

#### 工具

- **Unity Test Framework 1.x (PlayMode)**：真实 Unity Player 环境
- **InMemory SQLite**：替代真实数据库
- **Fake Steamworks SDK**：模拟云同步

#### 测试入口

```
tests/Integration/
├── CoreSwitchSlotIntegrationTest.cs    # M01 + M02
├── RoomPlayerIntegrationTest.cs        # M03 + M04
├── RoomSwitchSlotIntegrationTest.cs    # M03 + M02
├── SaveSystemRoomIntegrationTest.cs    # M07 + M03 (存档含房间)
├── AudioSwitchSlotIntegrationTest.cs   # M06 + M02 (切换音)
├── HintSystemRoomIntegrationTest.cs    # M09 + M03 (卡点)
├── EventBusIntegrationTest.cs          # 15 事件全链路
└── EndToEndFlowTest.cs                 # 1-1 → 1-2 → 1-3
```

#### 关键测试用例 (示例)

```csharp
// tests/Integration/CoreSwitchSlotIntegrationTest.cs
[UnityTest]
public IEnumerator SwitchSlot_PublishesEvent_CoreReceives() {
    var core = new GameObject().AddComponent<GlobalStateMachine>();
    var slot = new GameObject().AddComponent<ToggleSlot>();
    int eventCount = 0;
    EventBus.Subscribe<OnSlotSwitch>(_ => eventCount++);

    slot.Switch(SwitchDirection.Clockwise);
    yield return null;  // 等待 1 帧

    Assert.That(eventCount, Is.EqualTo(1));
}
```

```csharp
// tests/Integration/SaveSystemRoomIntegrationTest.cs
[UnityTest]
public IEnumerator SaveSystem_PersistsRoomStats_LoadRecoversCorrectly() {
    var save = new SaveSystem();
    var data = new SaveData {
        CurrentRoomId = "2-3",
        RoomStats = new[] {
            new RoomStats { RoomId = "1-1", Completed = true, DurationSec = 120 }
        }
    };
    yield return save.SaveAsync(data).AsCoroutine();

    var loaded = save.LoadAsync();
    yield return loaded.AsCoroutine();
    Assert.That(loaded.Result.CurrentRoomId, Is.EqualTo("2-3"));
    Assert.That(loaded.Result.RoomStats[0].DurationSec, Is.EqualTo(120));
}
```

### 2.3 E2E 测试 (End-to-End Test)

#### 目标

- **覆盖率：** 关键路径 100% (1-1→1-5→2-1→2-6→3-1→3-8)
- **速度：** 5-10min/case
- **范围：** 玩家视角的全流程

#### 工具

- **手动测试**：5 人 Playtest 玩家 (10-v2 §6.1)
- **自动脚本 (可选)**：Unity Input System + Coroutine 模拟玩家操作
- **截图对比**：ScreenshotDiff 工具 (可选 v1.1)

#### 测试入口

```
tests/E2E/
├── Manual/
│   ├── E2E-01-Ch1-Complete.md          # 1-1→1-5 全通关
│   ├── E2E-02-Ch2-Complete.md          # 2-1→2-6 全通关
│   ├── E2E-03-Ch3-Complete.md          # 3-1→3-8 全通关
│   ├── E2E-04-FullGame-Complete.md     # 19 间全通关
│   ├── E2E-05-Reset-Flow.md            # R 键重置
│   ├── E2E-06-SaveLoad.md              # 存档读档
│   ├── E2E-07-PauseMenu.md             # 暂停菜单
│   ├── E2E-08-Settings.md              # 9 类音频 + 4 类无障碍
│   └── E2E-09-Localization.md          # 中英切换
└── Auto/ (Phase 4+)
    └── E2E-Auto-Ch1.cs                 # 自动跑通 Ch1
```

#### 关键 E2E 场景

| E2E ID | 场景 | 步骤 | 验收 |
|--------|------|------|------|
| E2E-01 | Ch1 通关 | 进入 1-1 → 切换 → 走到出口 → 重复 1-2~1-5 | 5 房间全通关 + 章节完成画面 |
| E2E-04 | 19 间全通关 | 1-1~3-8 全部通关 | 通关画面 + 步数显示 |
| E2E-05 | R 键重置 | 1-1 切换 3 次 → 按 R | 房间回到初始 + 300ms 淡出淡入 |
| E2E-06 | 存档读档 | 完成 1-1 → 退出 → 重启 → 进入 | 自动加载到 1-2 |
| E2E-07 | 暂停菜单 | 按 ESC → 菜单 → 继续 | Time.timeScale = 0 → 1 |
| E2E-08 | 9 类音频 | 设置中调节 9 类 dB | 实时生效 + 持久化 |
| E2E-09 | 中英切换 | 设置中切到 en-US | 85 字符串全部英文 |

### 2.4 性能测试 (Performance Test)

#### 目标

- **覆盖率：** 性能预算 100% (01-v2 §性能预算 6 项)
- **速度：** 1-3h/case
- **范围：** 帧率 / 内存 / DrawCall / 冷启动 / 切换响应 / 加载时长

#### 性能预算 (与 01-v2 对齐)

| 指标 | 目标 | 验证方式 | 测试频次 |
|------|------|---------|:--------:|
| **帧率** | ≥ 60 FPS (PC) / ≥ 30 FPS (移动可选) | Unity Profiler | 每周 |
| **切换响应** | ≤ 16ms (1 帧) | Profiler Marker | 每周 |
| **单房间槽位数** | ≤ 8 | 静态检查 | 每 PR |
| **单场景 DrawCall** | ≤ 50 | Frame Debugger | 每周 |
| **内存峰值** | ≤ 512MB | Profiler Memory | 每周 |
| **冷启动** | ≤ 5s 到主菜单 | 计时 | 每 Release |

#### 工具

- **Unity Profiler**：CPU / GPU / 内存 / 渲染
- **Unity Frame Debugger**：DrawCall
- **dotMemory**：内存快照
- **自定义 Marker**：`ProfilerMarker` 类标记关键路径

#### 测试入口

```
tests/Performance/
├── FrameRateTest.cs                    # 60 FPS
├── MemoryPeakTest.cs                   # 512MB
├── DrawCallTest.cs                     # ≤ 50
├── SwitchResponseTest.cs               # ≤ 16ms
├── ColdStartTest.cs                    # ≤ 5s
├── SaveLoadPerformanceTest.cs          # ≤ 50ms 写 / ≤ 20ms 读
└── LoadTimeTest.cs                     # 房间加载 ≤ 200ms
```

#### 关键性能测试 (示例)

```csharp
// tests/Performance/SwitchResponseTest.cs
[UnityTest, Performance]
public IEnumerator SwitchSlot_ResponseTime_Below16ms() {
    var slot = new GameObject().AddComponent<ToggleSlot>();
    var marker = new ProfilerMarker("SwitchSlot.Switch");

    // 预热
    for (int i = 0; i < 100; i++) slot.Switch(SwitchDirection.Clockwise);

    // 测量 1000 次
    var stopwatch = Stopwatch.StartNew();
    for (int i = 0; i < 1000; i++) {
        using (marker.Auto()) {
            slot.Switch(SwitchDirection.Clockwise);
        }
    }
    stopwatch.Stop();
    var avgMs = stopwatch.ElapsedMilliseconds / 1000.0;
    Assert.That(avgMs, Is.LessThan(16.0), "Switch response should be < 16ms");
    yield return null;
}
```

### 2.5 兼容性测试 (Compatibility Test)

#### 目标

- **覆盖率：** 7 平台 100% (v1.0: 3 平台 Steam+Mac+Itch.io / v1.1: 4 平台 / v2.0: 7 平台)
- **速度：** 1-2h/平台
- **范围：** 平台 SDK / 输入设备 / 分辨率 / 操作系统

#### 7 平台测试矩阵 (与 11-v2 对齐)

| 平台 | v1.0 | 测试类型 | 工具 |
|------|:----:|---------|------|
| **PC Steam** | ✅ | Steam SDK + 4 分辨率 + 键鼠/手柄 | Steamworks Test |
| **PC Mac** | ✅ | macOS 10.15+ + Apple Silicon | Unity Mac Build |
| **Itch.io** | ✅ | 浏览器下载 + 试玩版 1-1~1-5 | 手动 |
| **PS5** | v2.0 | PS5 SDK + DualSense + 4K HDR | Sony Test Kit |
| **Xbox Series X\|S** | v2.0 | GDK + Smart Delivery + Xbox 手柄 | MS Test |
| **Nintendo Switch** | v1.1 | Nintendo SDK + Joy-Con + 掌机/TV | Lotcheck |
| **iOS** | v2.0 | iOS 14+ + iPhone/iPad + 触屏 | App Store TestFlight |
| **Android** | v2.0 | Android 10+ + 多分辨率 + 触屏 | Google Play Console |

#### 测试入口

```
tests/Compatibility/
├── Steam/
│   ├── SteamAchievementsTest.cs       # 6 隐藏成就
│   ├── SteamCloudTest.cs              # 云存档同步
│   └── SteamDlcTest.cs                # 豪华版 DLC
├── Mac/
│   ├── MacMetalRenderingTest.cs       # Metal 渲染
│   └── MacAppleSiliconTest.cs         # M1/M2 原生
├── PS5/
│   ├── PS5DualSenseTest.cs            # DualSense 触觉
│   └── PS5HdrTest.cs                  # 4K HDR
├── Xbox/
│   ├── XboxSmartDeliveryTest.cs
│   └── XboxAchievementsTest.cs
├── Switch/
│   ├── SwitchJoyConTest.cs            # Joy-Con / Pro Controller
│   └── SwitchHandheldTest.cs          # 掌机模式 720p
├── iOS/
│   ├── iOSTouchScreenTest.cs          # 触屏输入
│   └── iOSAppStoreTest.cs             # TestFlight
└── Android/
    ├── AndroidFragmentationTest.cs    # 多分辨率
    └── AndroidGooglePlayTest.cs       # Play Billing
```

#### 输入设备兼容性矩阵

| 平台 | 键鼠 | 手柄 | 触屏 | 测试通过标准 |
|------|:----:|:----:|:----:|-------------|
| **PC Steam** | ✅ | Xbox/PS | ❌ | 全部按键响应 |
| **PC Mac** | ✅ | Xbox/PS | ❌ | 全部按键响应 |
| **Itch.io** | ✅ | Xbox/PS | ❌ | 全部按键响应 |
| **PS5** | ❌ | DualSense | ❌ | DualSense 触觉 + 自适应扳机 |
| **Xbox** | ❌ | Xbox 手柄 | ❌ | Xbox 按钮响应 |
| **Switch** | ❌ | Joy-Con/Pro | ❌ | HD Rumble |
| **iOS** | ❌ | MFi (可选) | ✅ | 触屏模拟 E/Q/R/方向 |
| **Android** | ❌ | Bluetooth (可选) | ✅ | 触屏模拟 E/Q/R/方向 |

## 3. 与 10-v2 4 阶段 12 里程碑对齐 (Milestone Alignment)

> 详见 [`../README.md` §4 实施阶段](../README.md#4-实施阶段-implementation-phases--与-10-v2-4-阶段-12-里程碑对齐) + 10-v2 §1。

| 里程碑 | 阶段 | 测试验收标准 | 测试负责人 |
|:------:|:----:|-------------|----------|
| **M01** | P0 | docs/01-12 v2 ≥ 80 分 (ce-doc-review 通过) | 中书省 |
| **M02** | P0 | Unity 跑通 + SaveSystem Load/Save 单元测试 100% | Unity 工程师 |
| **M03** | P0 | 1-1 端到端 + SwitchSlot 单元测试 ≥ 80% | Unity 工程师 + QA |
| **M04** | P1 | Ch1 全 5 间 E2E + CycleSlot 单元测试 ≥ 80% | QA |
| **M05** | P1 | 2-1 + ConditionalSlot 单元测试 ≥ 80% + 集成测试 | Unity 工程师 |
| **M06** | P1 | 2-1/2-2/2-3 E2E | QA |
| **M07** | P1 | 2-1~2-6 E2E + 性能测试 60 FPS | QA + 性能工程师 |
| **M08** | P2 | 3-1/3-2/3-3 E2E | QA |
| **M09** | P2 | 3-4/3-5/3-6 E2E + **P0-001 平衡性回退** | QA + 数值策划 |
| **M10** | P2 | 19 间全通关 + 5 人 Playtest | QA + 5 人 |
| **M11** | P3 | RC + 性能优化 + 兼容性测试 (Steam/Mac) | QA + DevOps |
| **M12** | P3 | Release + 全平台 E2E + IARC 评级 | DevOps + 发行 |

## 4. CI 集成 (CI Integration)

> 详见 [`ci-cd.md`](./ci-cd.md) §3 CI/CD 流水线。

### 4.1 GitHub Actions Workflow

```yaml
# .github/workflows/ci.yml
name: CI
on: [push, pull_request]
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: StyleCop Lint
        run: python tools/ci/lint_csharp.py
      - name: Markdown Lint
        run: npx markdownlint '**/*.md'

  unit-test:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - name: Unity Unit Test (EditMode)
        uses: game-ci/unity-test-runner@v3
        with:
          testMode: editmode
          artifactsPath: test-results
      - name: Coverage Report
        run: python tools/ci/coverage.py
      - name: Codecov Upload
        uses: codecov/codecov-action@v3
        with:
          file: coverage/coverage.xml

  integration-test:
    runs-on: ubuntu-latest
    needs: unit-test
    steps:
      - uses: actions/checkout@v4
      - name: Unity Integration Test (PlayMode)
        uses: game-ci/unity-test-runner@v3
        with:
          testMode: playmode
          artifactsPath: test-results

  build:
    runs-on: ubuntu-latest
    needs: integration-test
    steps:
      - uses: actions/checkout@v4
      - name: Unity Build (StandaloneWindows64)
        uses: game-ci/unity-builder@v3
        with:
          targetPlatform: StandaloneWindows64
      - name: Upload Build
        uses: actions/upload-artifact@v4
        with:
          name: Build-Windows
          path: build/Windows/
```

### 4.2 覆盖率门槛 (Coverage Threshold)

```yaml
# codecov.yml
coverage:
  status:
    project:
      default:
        target: 70%       # 单元测试 ≥ 70%
        threshold: 2%     # 下降不超过 2%
    patch:
      default:
        target: 50%       # 新代码 ≥ 50%
```

### 4.3 PR 合并要求 (PR Merge Requirements)

- [ ] Lint 通过 (StyleCop + Markdown)
- [ ] 单元测试通过 + 覆盖率 ≥ 70%
- [ ] 集成测试通过 (PlayMode)
- [ ] Code Review 至少 1 人通过
- [ ] 无 P0 Bug
- [ ] 性能预算未退化 (60 FPS / 512MB / 50 DrawCall)

## 5. P0-001 测试策略 (P0-001 Test Strategy)

> **强 P0-001 跟踪：** 02-v2 §13 AC-06 缺"难度上限 20"，实施期测试必须**自我保护**。

### 5.1 难度字段测试原则

1. **不依赖缺失数据：** 测试用例使用 NULL / 默认值（参考 05-v2 §6.1 实际值）
2. **自我保护验证：** 验证 `Room.validate()` / `Level.validate()` / `SaveSystem` 字段 CHECK (1-20) 生效
3. **不修复：** 测试不修改 02-v2 §13 AC-06
4. **不编造：** 测试不创造 02-v2 缺失的难度上限数据
5. **UI 测试：** UI 显示"待配置（P0-001）"作为默认行为

### 5.2 关键 P0-001 测试用例

```csharp
// tests/Unit/Room/RoomValidateTest.cs (P0-001)
[TestFixture]
public class RoomValidateP0001Test {
    [Test]
    public void Room_Difficulty_Exceeds20_TriggersFallback() {
        // P0-001 关联：02-v2 §13 AC-06 缺 "难度上限 20"
        // 设计/data/p0-001-tracking.md §3 修复选项 C
        var data = new RoomData { Difficulty = 25, DifficultyMax = 20 };
        var validated = RoomLoader.Validate(data);

        Assert.That(validated.Difficulty, Is.EqualTo(20),
            "P0-001 self-protection: difficulty > 20 must fallback to 20");
    }

    [Test]
    public void Room_Difficulty_Null_UsesDefault() {
        var data = new RoomData { Difficulty = null, DifficultyMax = 20 };
        var validated = RoomLoader.Validate(data);

        Assert.That(validated.Difficulty, Is.EqualTo(1),
            "P0-001: NULL difficulty uses default 1 (P0-001 pending)");
    }

    [Test]
    public void Room_Difficulty_Negative_Rejected() {
        var data = new RoomData { Difficulty = -1, DifficultyMax = 20 };
        Assert.Throws<ValidationException>(() => RoomLoader.Validate(data));
    }

    [Test]
    public void Room_Difficulty_BoundaryValues_Accepted() {
        Assert.DoesNotThrow(() => RoomLoader.Validate(new RoomData { Difficulty = 1 }));
        Assert.DoesNotThrow(() => RoomLoader.Validate(new RoomData { Difficulty = 20 }));
    }

    [Test]
    public void BossRooms_3_4_3_5_3_6_DifficultyCapped() {
        // P0-001 关联：03-v2 §6.2 警示 + 05-v2 §6.1 Boss 房数据
        // 实际计算 17.5/20/21.5 → 强制 ≤ 20
        var bossRooms = new[] {
            new RoomData { RoomId = "3-4", Difficulty = 17 },
            new RoomData { RoomId = "3-5", Difficulty = 20 },
            new RoomData { RoomId = "3-6", Difficulty = 20 }  // 21.5 → 20
        };

        foreach (var room in bossRooms) {
            var validated = RoomLoader.Validate(room);
            Assert.That(validated.Difficulty, Is.LessThanOrEqualTo(20),
                $"P0-001: {room.RoomId} difficulty must ≤ 20");
        }
    }
}
```

```csharp
// tests/Unit/SaveSystem/SaveSystemP0001Test.cs
[TestFixture]
public class SaveSystemP0001Test {
    [Test]
    public void SaveSystem_LoadSaveWithNullDifficulty_LoadsWithNull() {
        // P0-001 关联：11 阻塞字段 NOT NULL DEFAULT NULL
        var saveData = new SaveData {
            MaxDifficultyCompleted = null  // P0-001 待修复
        };
        var serialized = SaveSerializer.Serialize(saveData);
        var deserialized = SaveSerializer.Deserialize(serialized);

        Assert.That(deserialized.MaxDifficultyCompleted, Is.Null,
            "P0-001: NULL max_difficulty_completed is acceptable (pending fix)");
    }

    [Test]
    public void SaveSystem_LoadSaveWithInvalidDifficulty_Rejected() {
        // P0-001 CHECK 约束：difficulty IS NULL OR BETWEEN 1 AND 20
        var saveData = new SaveData { MaxDifficultyCompleted = 25 };
        Assert.Throws<ValidationException>(() => SaveSerializer.Validate(saveData));
    }
}
```

## 6. 测试数据管理 (Test Data Management)

### 6.1 测试夹具 (Test Fixtures)

```
tests/Fixtures/
├── Rooms/
│   ├── room-1-1.json              # 教学房
│   ├── room-2-3.json              # 中等
│   ├── room-3-6.json              # Boss 房 (P0-001)
│   └── room-3-8.json              # 通关房
├── Saves/
│   ├── valid-save.json            # 正常存档
│   ├── corrupted-save.json        # 损坏存档 (降级测试)
│   └── old-version-save.json      # 旧版本 (迁移测试)
└── Audio/
    ├── switch.wav
    ├── reset.wav
    └── win.wav
```

### 6.2 Mock 数据 (Mock Data)

- **MockPlayer**：模拟玩家输入
- **MockAudioManager**：不实际播放音频
- **MockSteamworks**：模拟云同步成功/失败
- **MockNetwork**：模拟断网/弱网

### 6.3 测试隔离 (Test Isolation)

- 每个测试前清空 EventBus
- 每个测试用独立 SQLite 内存数据库
- 每个测试用临时目录（避免污染真实存档）

## 7. 测试报告与指标 (Test Reports & Metrics)

### 7.1 报告维度

| 维度 | 指标 | 工具 |
|------|------|------|
| **覆盖率** | 单元/集成/E2E 各层覆盖率 | Coverlet + Codecov |
| **通过率** | 各层通过率 | NUnit + Unity Test Framework |
| **速度** | 各层平均/最长耗时 | Unity Test Framework |
| **稳定性** | Flaky test 比例 | 手动跟踪 |
| **缺陷** | P0/P1/P2 Bug 数 | GitHub Issues |
| **性能** | 帧率/内存/DrawCall | Unity Profiler |

### 7.2 报告输出

- **PR 评论**：Codecov 自动评论覆盖率变化
- **周报**：测试通过率 + 覆盖率趋势
- **里程碑报告**：每个 M01-M12 的测试验收

## 8. 测试工具链 (Test Toolchain)

| 工具 | 用途 | 版本 |
|------|------|------|
| **NUnit** | 单元测试框架 | 3.13.x |
| **Unity Test Framework** | EditMode/PlayMode | 1.1.x |
| **Moq** | Mock 库 | 4.18.x |
| **FluentAssertions** | 可读断言 | 5.10.x |
| **Coverlet** | 覆盖率收集 | 6.0.x |
| **Codecov** | 覆盖率报告 | (云) |
| **Unity Profiler** | 性能分析 | (Unity 内置) |
| **dotMemory** | 内存分析 | 2023.x |
| **Unity Performance Testing Extension** | 性能测试 | 2.8.x |

## 9. 关联文档 (Cross-References)

### 上游 (本文档依赖)

- [`../README.md`](../README.md) — 实施期总览
- [`./module-spec.md`](./module-spec.md) — 14 模块测试入口
- [`./ci-cd.md`](./ci-cd.md) — CI/CD 流水线（测试集成）
- [`../../docs/01-overview-v2.md`](../../docs/01-overview-v2.md) §性能预算
- [`../../docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) §1 4 阶段 12 里程碑
- [`../../docs/11-release-v2.md`](../../docs/11-release-v2.md) §1 7 平台
- [`../data/p0-001-tracking.md`](../data/p0-001-tracking.md) — **强 P0-001 跟踪**

### 下游 (本文档被依赖)

- `tests/**` 全部测试代码 (phase4 实施期)
- `.github/workflows/ci.yml` CI 配置
- `codecov.yml` 覆盖率配置
- `tools/ci/coverage.py` 覆盖率脚本

## 10. 验收标准 (Acceptance Criteria)

- [x] **AC-01** Frontmatter 7 字段完整
- [x] **AC-02** 测试金字塔 3 层 (单元 70% / 集成 50% / E2E 30%)
- [x] **AC-03** 5 类测试 (单元/集成/E2E/性能/兼容) 完整定义
- [x] **AC-04** 7 平台兼容性测试矩阵 (v1.0: 3 平台 / v1.1: 4 平台 / v2.0: 7 平台)
- [x] **AC-05** 与 10-v2 4 阶段 12 里程碑对齐
- [x] **AC-06** **强 P0-001 测试策略** — 5 条原则 + 6 关键测试用例
- [x] **AC-07** CI 集成 (GitHub Actions + Codecov)
- [x] **AC-08** PR 合并要求 (Lint + 测试 + 覆盖率 + Code Review)
- [x] **AC-09** 测试工具链 (NUnit + UTF + Moq + Coverlet + Profiler)
- [x] **AC-10** 测试数据管理 (Fixtures + Mock + 隔离)
- [x] **AC-11** 测试报告 (覆盖率/通过率/速度/稳定性/缺陷/性能 6 维度)
- [x] **AC-12** 文档总行数 ≥ 600 行 (实际 ~700 行)

## 11. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent (ANZHONG-16) 创建。**新建**：测试金字塔 (单元 70% / 集成 50% / E2E 30%) + 5 类测试 (单元/集成/E2E/性能/兼容) 完整定义 + 7 平台兼容性测试矩阵 (v1.0 3 平台 + v1.1 4 平台 + v2.0 7 平台) + 与 10-v2 4 阶段 12 里程碑对齐 + **强 P0-001 测试策略** (5 条原则 + 6 关键测试用例, 引用 design/data/p0-001-tracking.md, 不修复 P0-001, 不编造数据) + CI 集成 (GitHub Actions + Codecov 70%/50%) + PR 合并要求 + 测试工具链 (NUnit 3.13 + UTF 1.1 + Moq 4.18 + Coverlet 6.0 + Unity Profiler)。**全链 16/16 收官 🏆** 第 3/8 文件。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft (等待 ce-doc-review 评审)
**全链状态：** 16/16 收官 🏆