---
title: 《暗室》编码规范 (Coding Standards)
doc_id: DESIGN-anzhong-implementation-coding-standards
parent: design/implementation/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》编码规范 (Coding Standards)

> **一句话定位：** C# 9 / .NET 8 / Unity 2022 LTS + .editorconfig + StyleCop + SonarQube + 注释/测试覆盖/性能预算 的端到端编码基线。

## 目的 (Purpose)

本文档是《暗室》**代码质量层**的**唯一权威基线**。它向：

- **Unity 客户端工程师** — C# 9 / .NET 8 / Unity 2022 LTS 编码规范
- **Code Reviewer** — 评审清单 (与 dev-workflow.md §3.2 对齐)
- **CI/CD** — StyleCop + .editorconfig 自动化检查
- **新加入工程师** — 30 分钟看懂"怎么命名、怎么注释、怎么测试、怎么优化"
- **架构师** — SOLID / DRY / KISS / YAGNI 原则

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的 C# / Unity / .NET 编码规范——命名约定、文件组织、注释规范、测试覆盖、性能预算、安全规范、依赖管理——**第一次**用"语言规范 + Unity 规范 + 性能预算 + 安全规范 + 工具链" 5 维度统一描述，作为 phase4 实施的"代码质量合同"。

## 范围 (Scope)

### 包含

- **C# 9 规范**：命名约定、文件组织、注释、单元测试、性能、安全
- **Unity 2022 LTS 规范**：MonoBehaviour、ScriptableObject、Asset、Prefab
- **.NET 8 规范**：NuGet、依赖注入、异步 (async/await)
- **.editorconfig + StyleCop**：自动化检查
- **性能预算**：60 FPS / 512MB / 50 DrawCall / 16ms 切换
- **安全规范**：AES-256-GCM、GDPR、SQL 注入防护

### 不包含 (Out of Scope)

- 模块规约 → 见 [`module-spec.md`](./module-spec.md)
- 测试用例 → 见 [`test-strategy.md`](./test-strategy.md)
- CI/CD → 见 [`ci-cd.md`](./ci-cd.md)
- Git Flow / Conventional Commits → 见 [`dev-workflow.md`](./dev-workflow.md)

## 一句话描述 (One-liner)

> **"C# 9 × Unity 2022 LTS × .NET 8 × StyleCop × 60 FPS，把 code style 翻译成可执行的质量基线。"**

## 1. 命名约定 (Naming Conventions)

### 1.1 标识符命名 (Identifier Naming)

| 类型 | 命名规则 | 示例 |
|------|---------|------|
| **Namespace** | PascalCase | `Anzhong.Core` |
| **Class** | PascalCase | `GlobalStateMachine` |
| **Interface** | IPascalCase | `IGameEvent` |
| **Method** | PascalCase | `TransitionTo` |
| **Property** | PascalCase | `CurrentState` |
| **Public Field** | PascalCase | `MaxSlots = 8` (常量) |
| **Private Field** | _camelCase | `_currentState` |
| **Local Variable** | camelCase | `roomId` |
| **Parameter** | camelCase | `roomId` |
| **Constant** | PascalCase | `MaxDifficulty = 20` |
| **Enum** | PascalCase (单数) | `SlotState` |
| **Enum Member** | PascalCase | `SlotState.Active` |
| **Async Method** | PascalCase + Async | `LoadAsync` |
| **Generic** | T + PascalCase | `TEvent` |

### 1.2 类名约定 (Class Naming)

| 类型 | 后缀 | 示例 |
|------|------|------|
| **Manager** | Manager | `SaveManager` |
| **Controller** | Controller | `PlayerController` |
| **Service** | Service | `TelemetryService` |
| **Factory** | Factory | `RoomFactory` |
| **Repository** | Repository | `SaveRepository` |
| **Builder** | Builder | `UrlBuilder` |
| **Validator** | Validator | `RoomValidator` |
| **Handler** | Handler | `EventHandler` |
| **Provider** | Provider | `HintProvider` |
| **MonoBehaviour** | 无强制后缀 | `SwitchSlot` |
| **ScriptableObject** | 无强制后缀 | `RoomData` |
| **Test Class** | Test | `ToggleSlotTest` |
| **Integration Test** | IntegrationTest | `RoomSwitchSlotIntegrationTest` |

### 1.3 禁止命名 (Forbidden Names)

- ❌ 单字符变量 (除循环变量 `i` `j` `k`)
- ❌ 匈牙利命名法 (`strName`, `iCount`)
- ❌ 缩写（除 `Id`, `Db`, `Bgm`, `UI`, `API`）
- ❌ 数字后缀 (`Player1`, `Player2`)
- ❌ 类型名作为变量名 (`stringString`)

## 2. 文件组织 (File Organization)

### 2.1 目录结构

```
anzhong/
├── src/                          # 源代码
│   ├── Core/
│   │   ├── GlobalStateMachine.cs
│   │   ├── EventBus.cs
│   │   └── Anzhong.Core.asmdef
│   ├── SwitchSlot/
│   ├── Room/
│   └── ...
├── tests/                        # 测试
│   ├── Unit/
│   ├── Integration/
│   ├── E2E/
│   ├── Performance/
│   ├── Compatibility/
│   └── Fixtures/
├── docs/                         # GDD
├── design/                       # 设计文档
│   ├── api/
│   ├── architecture/
│   ├── art/
│   ├── data/
│   └── implementation/           # ← 本任务
├── data/                         # 数据 (JSON / SQLite)
│   ├── config/
│   ├── levels/
│   ├── levels/room-{id}.json
│   └── numerical/parameters.json
├── tools/                        # 工具脚本
│   ├── build/
│   ├── ci/
│   ├── db/
│   └── distribute/
├── .github/                      # CI/CD
│   └── workflows/
├── .editorconfig
├── .gitignore
├── .husky/
├── .lintstagedrc
├── commitlint.config.js
├── Directory.Build.props
├── Anzhong.sln
└── README.md
```

### 2.2 单文件单类 (One Class Per File)

- ✅ 一个文件一个 public class
- ❌ 禁止多 class 共享一个文件
- ✅ 文件名 = 类名

### 2.3 命名空间组织 (Namespace Organization)

```csharp
// Anzhong.Core/GlobalStateMachine.cs
namespace Anzhong.Core {
    public class GlobalStateMachine { }
}

// Anzhong.SwitchSlot/ToggleSlot.cs
namespace Anzhong.SwitchSlot {
    public class ToggleSlot : SwitchSlot { }
}
```

**规则：**
- 命名空间 = 目录路径（PascalCase）
- 一个文件一个命名空间
- `using` 放在文件顶部，命名空间内
- `using` 按字母顺序排列

## 3. C# 9 规范 (C# 9 Standards)

### 3.1 类型与变量

```csharp
// ✅ 优先使用 var (类型明显时)
var slot = new ToggleSlot(slotId: 1);

// ✅ 显式类型 (类型不明显时)
GameState newState = GameState.Playing;

// ✅ 优先使用不可变 (record / readonly)
public record OnSlotSwitch(int SlotId, int CurrentIndex) : IGameEvent;

public class RoomData {
    public string RoomId { get; init; }
    public int Difficulty { get; init; }
}

// ❌ 避免可变的 public field
public int maxSlots = 8;

// ✅ 私有字段命名 _camelCase
private int _currentState;
```

### 3.2 异步 (Async/Await)

```csharp
// ✅ 异步方法名加 Async 后缀
public async Task<SaveData> LoadAsync() { }

// ✅ 使用 ValueTask (热路径)
public ValueTask<RoomData> GetRoomAsync(string roomId) { }

// ✅ 异步方法返回 Task / Task<T>，不用 void
public async Task SaveAsync(SaveData data) { }  // ✅
public async void SaveAsync(SaveData data) { } // ❌

// ✅ 异步 lambda
Func<Task> handler = async () => { await Task.Delay(100); };

// ✅ 并行独立任务
await Task.WhenAll(loadAudio, loadImage, loadConfig);

// ❌ 避免 .Result / .Wait() (死锁)
var data = LoadAsync().Result;  // ❌
var data = await LoadAsync();   // ✅
```

### 3.3 异常处理

```csharp
// ✅ 自定义异常继承 AnzhongException
public class SwitchSlotException : AnzhongException {
    public SwitchSlotException(string message) : base(message) { }
}

// ✅ 抛出具体异常
throw new SwitchSlotException($"Invalid slot type: {type}");

// ✅ 不吞异常
try {
    await SaveAsync(data);
} catch (SaveException ex) {
    Logger.Error(ex, "Save failed");
    throw;  // 重新抛出
}

// ❌ 不吞异常
try {
    await SaveAsync(data);
} catch { }  // ❌

// ✅ 使用 try-catch-finally
try {
    await SaveAsync(data);
} catch (SaveException ex) {
    Logger.Error(ex, "Save failed");
} finally {
    Cleanup();
}
```

### 3.4 集合与 LINQ

```csharp
// ✅ 优先使用集合初始化器
var rooms = new List<RoomData> {
    new RoomData { RoomId = "1-1" },
    new RoomData { RoomId = "1-2" }
};

// ✅ 优先使用 LINQ (可读性)
var activeSlots = slots.Where(s => s.CurrentState == SlotState.Active).ToList();

// ✅ 避免重复枚举
var enumerable = slots.Where(s => s.IsActive);
var list = enumerable.ToList();
var count = enumerable.Count();  // ❌ 重复枚举

// ✅ 缓存
var enumerable = slots.Where(s => s.IsActive).ToList();  // 单次枚举
var list = enumerable;
var count = list.Count;
```

### 3.5 字符串

```csharp
// ✅ 优先使用内插字符串
var msg = $"Room {roomId} not found";

// ✅ 频繁拼接用 StringBuilder
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.Append($"Item {i}\n");
}

// ✅ 字符串比较用 Ordinal
if (string.Equals(a, b, StringComparison.Ordinal)) { }

// ❌ 避免 String + 拼接 (在循环中)
var msg = "Room " + roomId + " not found";  // 性能差
```

## 4. Unity 2022 LTS 规范 (Unity Standards)

### 4.1 MonoBehaviour

```csharp
// ✅ 公共字段用 [SerializeField] private (Inspector 可见 + 不可外部访问)
[SerializeField] private int _maxSlots = 8;
public int MaxSlots => _maxSlots;

// ✅ 组件引用用 [RequireComponent]
[RequireComponent(typeof(Rigidbody2D))]
public class PlayerController : MonoBehaviour { }

// ✅ Cache GetComponent
private Rigidbody2D _rigidbody;
private void Awake() {
    _rigidbody = GetComponent<Rigidbody2D>();
}

// ❌ 避免每帧 GetComponent
private void Update() {
    GetComponent<Rigidbody2D>().velocity = ...;  // ❌
}

// ✅ 使用 ProfilerMarker 标记
private static readonly ProfilerMarker _switchMarker = new("SwitchSlot.Switch");
public void Switch() {
    using (_switchMarker.Auto()) {
        // ...
    }
}
```

### 4.2 协程 vs async/await

```csharp
// ✅ Unity 协程用于帧序列
public IEnumerator FadeOutAsync(float duration) {
    float elapsed = 0;
    while (elapsed < duration) {
        canvas.alpha = 1 - (elapsed / duration);
        elapsed += Time.deltaTime;
        yield return null;
    }
}

// ✅ async/await 用于 I/O (文件 / 网络)
public async Task<SaveData> LoadAsync() {
    var data = await File.ReadAllBytesAsync(path);
    return SaveSerializer.Deserialize(data);
}

// ❌ 协程用于 I/O (会阻塞主线程)
public IEnumerator LoadCoroutine() {
    var data = File.ReadAllBytes(path);  // ❌ 阻塞
    yield return null;
}
```

### 4.3 ScriptableObject 配置

```csharp
// ✅ 配置用 ScriptableObject (可热更新)
[CreateAssetMenu(fileName = "RoomData", menuName = "Anzhong/Room Data")]
public class RoomData : ScriptableObject {
    public string RoomId;
    public int Difficulty;
}

// ✅ 在 Inspector 引用
[SerializeField] private RoomData _roomData;
```

### 4.4 资源管理

```csharp
// ✅ 资源用 Addressables (热更新 + 内存管理)
public async Task<Sprite> LoadSpriteAsync(string key) {
    var handle = Addressables.LoadAssetAsync<Sprite>(key);
    await handle.Task;
    return handle.Result;
}

// ❌ Resources.Load (旧 API, 不可热更新)
var sprite = Resources.Load<Sprite>("Sprites/Player");  // ❌
```

### 4.5 生命周期 (Lifecycle)

| 方法 | 用途 | 禁用操作 |
|------|------|---------|
| **Awake** | 初始化（自身依赖） | ❌ 找其他对象 |
| **OnEnable** | 注册事件 | ❌ 找其他对象 |
| **Start** | 找其他对象 | ❌ 阻塞操作 |
| **Update** | 帧更新 | ❌ new / Find / Load |
| **FixedUpdate** | 物理更新 | ❌ 同 Update |
| **LateUpdate** | 相机跟随 | ❌ 同 Update |
| **OnDisable** | 注销事件 | ❌ 阻塞 |
| **OnDestroy** | 释放资源 | ❌ 找其他对象 |

## 5. 注释规范 (Comment Standards)

### 5.1 XML 文档注释 (Public API)

```csharp
/// <summary>
/// 切换槽位到指定方向
/// </summary>
/// <param name="direction">顺时针/逆时针</param>
/// <returns>切换是否成功</returns>
/// <exception cref="SwitchSlotException">槽位状态不允许切换</exception>
public bool Switch(SwitchDirection direction) {
    // ...
}
```

**要求：**
- 所有 public class / method / property 必须有 XML 注释
- `<summary>` ≤ 1 行
- `<param>` / `<returns>` / `<exception>` 必填
- 私有方法可选

### 5.2 行内注释 (Inline Comments)

```csharp
// ✅ 解释"为什么"而非"做什么"
// P0-001 自我保护：难度 > 20 时强制回退 (引用 data/p0-001-tracking.md §3 修复选项 C)
if (data.Difficulty > MaxDifficulty) {
    data.Difficulty = MaxDifficulty;
}

// ❌ 重复代码
// 设置 maxSlots 为 8
maxSlots = 8;  // ❌ 明显
```

### 5.3 TODO 注释

```csharp
// TODO(author): 实现 CycleSlot 边缘 case
// TODO: P0-001 等待陛下 21:00 调研后决定
// FIXME: 修复 ConditionalSlot 依赖链深度
// HACK: 临时绕过 (说明原因 + 何时移除)
```

**格式：** `<KEYWORD>(<author>): <description>`

## 6. 单元测试规范 (Unit Test Standards)

### 6.1 测试方法命名

```
[UnitOfWork]_[ScenarioUnderTest]_[ExpectedBehavior]
```

**示例：**
- `ToggleSlot_SwitchOnce_StateFlipsToActive`
- `Room_LoadRoom_DifficultyExceeds20_TriggersFallback`
- `SaveSystem_LoadSaveWithCorruptedData_FallsBackToBackup`

### 6.2 AAA 模式 (Arrange-Act-Assert)

```csharp
[Test]
public void ToggleSlot_SwitchOnce_StateFlipsToActive() {
    // Arrange (准备)
    var slot = new ToggleSlot(slotId: 1);
    var expectedState = SlotState.Active;

    // Act (执行)
    slot.Switch(SwitchDirection.Clockwise);

    // Assert (断言)
    Assert.That(slot.CurrentState, Is.EqualTo(expectedState));
}
```

### 6.3 测试覆盖率 (Coverage)

| 模块类型 | 单元覆盖率 | 关键路径覆盖率 |
|---------|:----------:|:------------:|
| **P0 模块** (M01-M08) | ≥ 80% | 100% |
| **P1 模块** (M09-M13) | ≥ 70% | 100% |
| **P2 模块** (M14) | ≥ 60% | 100% |
| **跨切** (X01-X06) | ≥ 70% | 100% |
| **M12 Release** | ≥ 80% 整体 | 100% |

## 7. 性能预算 (Performance Budget)

> 详见 `docs/01-overview-v2.md` §性能预算 6 项。

| 指标 | 目标 | 验证工具 | 触发警告 |
|------|------|---------|:--------:|
| **帧率** | ≥ 60 FPS (PC) | Unity Profiler | < 50 FPS |
| **切换响应** | ≤ 16ms (1 帧) | Profiler Marker | > 16ms |
| **单房间槽位数** | ≤ 8 | 静态检查 | > 8 |
| **单场景 DrawCall** | ≤ 50 | Frame Debugger | > 50 |
| **内存峰值** | ≤ 512MB | Profiler Memory | > 480MB |
| **冷启动** | ≤ 5s 到主菜单 | 计时 | > 7s |

### 7.1 性能优化规则

```csharp
// ✅ 对象池 (避免 GC)
public class AudioSourcePool {
    private readonly Stack<AudioSource> _pool = new();
    public AudioSource Get() => _pool.Count > 0 ? _pool.Pop() : CreateNew();
    public void Return(AudioSource source) {
        source.Stop();
        _pool.Push(source);
    }
}

// ❌ 避免每帧 new
private void Update() {
    var list = new List<Transform>();  // ❌ 每帧 GC
}

// ✅ 缓存
private readonly List<Transform> _list = new();

// ❌ 避免每帧 Find
private void Update() {
    var player = GameObject.Find("Player");  // ❌
}

// ✅ Cache
[SerializeField] private Transform _player;
```

### 7.2 字符串 / 装箱优化

```csharp
// ❌ 字符串拼接（GC）
string msg = "Room " + id + " is locked";

// ✅ StringBuilder (热路径)
var sb = new StringBuilder();
sb.Append("Room ").Append(id).Append(" is locked");

// ❌ 装箱 (值类型 → object)
object boxed = 42;

// ✅ 泛型避免装箱
T value = 42;  // 泛型
```

## 8. 安全规范 (Security Standards)

### 8.1 加密 (Encryption)

```csharp
// ✅ 敏感数据 AES-256-GCM
public class AesGcmCipher {
    public byte[] Encrypt(byte[] plaintext, byte[] key, byte[] nonce);
    public byte[] Decrypt(byte[] ciphertext, byte[] key, byte[] nonce);
}

// ❌ 硬编码密钥
var key = new byte[] { 1, 2, 3, ... };  // ❌

// ✅ 密钥从 KeyStore 读取
var key = KeyStore.LoadAesKey("save-system");
```

### 8.2 输入验证 (Input Validation)

```csharp
// ✅ 验证外部输入
public void LoadRoom(string roomId) {
    if (string.IsNullOrEmpty(roomId)) throw new ArgumentException("roomId is empty");
    if (roomId.Length > 10) throw new ArgumentException("roomId too long");
    // ...
}

// ✅ 使用白名单（不是黑名单）
public bool IsValidRoomId(string roomId) {
    return Regex.IsMatch(roomId, @"^[1-3]-[1-8]$");
}
```

### 8.3 GDPR 合规

```csharp
// ✅ 玩家可导出/删除数据
public async Task<byte[]> ExportAllDataAsync() { }
public async Task DeleteAllDataAsync() { }

// ❌ 不收集 PII (姓名、邮箱、IP)
public string PlayerName { get; set; }  // ❌ 单机游戏不需要

// ✅ 仅本地存档 (无服务器)
public class SaveData {
    public string PlayerId { get; set; }  // 设备本地 UUID
    public DateTime CreatedAt { get; set; }
}
```

## 9. 依赖管理 (Dependency Management)

### 9.1 NuGet 包 (Package Manager)

```xml
<!-- Anzhong.Core.csproj -->
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <LangVersion>9.0</LangVersion>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Unity.Mathematics" Version="1.3.1" />
    <PackageReference Include="Unity.Collections" Version="2.1.4" />
  </ItemGroup>
</Project>
```

### 9.2 依赖原则

- ✅ 优先使用 Unity 内置包 (URP / Input System / Addressables)
- ❌ 避免外部 SDK（DOTween 除外, MIT 许可）
- ❌ 避免 GPL / LGPL 协议
- ✅ 每个依赖必须有 License 审查
- ✅ 重大版本升级前评估 ABI 兼容性

### 9.3 禁止依赖 (Forbidden Dependencies)

| 库 | 原因 |
|----|------|
| **FMOD** | 商业许可 (1 人 indie 负担不起) |
| **Wwise** | 商业许可 |
| **PlayFab** | v1.0 无服务器 (Phase 4+ 评估) |
| **GameAnalytics** | v1.0 不收集数据 |
| **Any GPL library** | 协议冲突 |

## 10. 工具链配置 (Tooling)

### 10.1 .editorconfig

```ini
# .editorconfig
root = true

[*]
indent_style = space
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true

[*.{cs}]
indent_size = 4
csharp_style_var_for_built_in_types = false:warning
csharp_style_var_when_type_is_apparent = true:suggestion
csharp_style_var_elsewhere = true:suggestion
csharp_new_line_before_open_brace = all
csharp_new_line_before_else = true
csharp_new_line_before_catch = true
csharp_new_line_before_finally = true

[*.{md}]
trim_trailing_whitespace = false

[*.{json,yml,yaml}]
indent_size = 2
```

### 10.2 stylecop.json

```json
{
  "settings": {
    "documentationRules": {
      "documentInternalElements": false,
      "documentPrivateElements": false
    },
    "orderingRules": {
      "elementOrder": [
        "constant", "field", "constructor", "property", "delegate",
        "event", "method", "struct", "class", "interface", "enum"
      ]
    },
    "namingRules": {
      "allowCommonHungarianPrefixes": ["Id", "Db"]
    }
  }
}
```

### 10.3 Directory.Build.props

```xml
<Project>
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <LangVersion>9.0</LangVersion>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <TreatWarningsAsErrors>false</TreatWarningsAsErrors>
    <NoWarn>CS1591</NoWarn>
    <Company>Anzhong Studio</Company>
    <Product>Anzhong (《暗室》)</Product>
    <Authors>1 人 Solo</Authors>
    <Copyright>Copyright (c) 2026 Anzhong Studio</Copyright>
  </PropertyGroup>
</Project>
```

## 11. P0-001 跟踪 (P0-001 Tracking)

> **强 P0-001 跟踪：** 02-v2 §13 AC-06 缺"难度上限 20"。

**编码期策略：**

1. **静态保护**：`RoomData.Difficulty` 字段用 `int?` (nullable) + XML 注释引用 `data/p0-001-tracking.md`
2. **运行时校验**：`RoomLoader.Validate()` 难度 > 20 → 警告 + 强制回退
3. **不修复**：不修改 02-v2 §13 AC-06（auto-chain 不擅自）
4. **不编造**：使用 05-v2 §6.1 实际值（1-1: 2, ..., 3-6: 20）
5. **常量**：`MaxDifficulty = 20` (P0-001 自我保护)

```csharp
/// <summary>
/// 房间难度（1-20）
/// </summary>
/// <remarks>
/// P0-001 关联：02-v2 §13 AC-06 缺"难度上限 20"硬约束。
/// 实施期使用 05-v2 §6.1 实际值；超过 20 时 <see cref="RoomLoader.Validate"/> 强制回退。
/// 修复时机：等陛下 21:00 调研后决定 (3 修复选项: phase3 design 期间 / phase4 单独 patch / 实施期间补)。
/// 详见: ../../design/data/p0-001-tracking.md
/// </remarks>
[Range(1, 20)]
public int? Difficulty { get; init; }

/// <summary>
/// 难度上限（自我保护常量）
/// </summary>
public const int MaxDifficulty = 20;
```

## 12. 关联文档 (Cross-References)

### 上游 (本文档依赖)

- [`../README.md`](../README.md) — 实施期总览
- [`./dev-workflow.md`](./dev-workflow.md) — Code Review 检查项
- [`./module-spec.md`](./module-spec.md) — 模块规约
- [`./test-strategy.md`](./test-strategy.md) — 测试覆盖率
- [`../../docs/01-overview-v2.md`](../../docs/01-overview-v2.md) §性能预算
- [`../../docs/02-core-mechanics-v2.md`](../../docs/02-core-mechanics-v2.md) §4 I/O Spec
- [`../architecture/tech-stack.md`](../architecture/tech-stack.md) — C# 9 / .NET 8
- [`../data/p0-001-tracking.md`](../data/p0-001-tracking.md) — 强 P0-001 跟踪

### 下游 (本文档被依赖)

- `.editorconfig` 自动化检查
- `stylecop.json` StyleCop 配置
- `Directory.Build.props` 项目级配置
- `tools/ci/lint_csharp.py` C# Lint 脚本

## 13. 验收标准 (Acceptance Criteria)

- [x] **AC-01** Frontmatter 7 字段完整
- [x] **AC-02** 命名约定 (12 类标识符 + 后缀 + 禁止命名)
- [x] **AC-03** 文件组织 (目录结构 + 单文件单类 + 命名空间)
- [x] **AC-04** C# 9 规范 (类型/异步/异常/集合/字符串)
- [x] **AC-05** Unity 2022 LTS 规范 (MonoBehaviour/协程/ScriptableObject/资源/生命周期)
- [x] **AC-06** 注释规范 (XML 文档 + 行内 + TODO 格式)
- [x] **AC-07** 单元测试规范 (AAA + 命名 + 覆盖率 70/80%)
- [x] **AC-08** 性能预算 (6 项 + 对象池 + 字符串优化)
- [x] **AC-09** 安全规范 (AES-256 + 输入验证 + GDPR)
- [x] **AC-10** 依赖管理 (NuGet + 原则 + 禁止依赖)
- [x] **AC-11** 工具链配置 (.editorconfig + stylecop.json + Directory.Build.props)
- [x] **AC-12** **强 P0-001 跟踪** — RoomData.Difficulty 静态保护 + Validate() 运行时校验
- [x] **AC-13** 文档总行数 ≥ 500 行 (实际 ~600 行)

## 14. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent (ANZHONG-16) 创建。**新建**：命名约定 (12 类) + 文件组织 + C# 9 规范 (类型/异步/异常/集合/字符串) + Unity 2022 LTS 规范 (MonoBehaviour/协程/ScriptableObject/资源/生命周期) + 注释规范 + 单元测试 (AAA + 命名 + 覆盖率 70/80%) + 性能预算 (6 项 + 对象池) + 安全规范 (AES-256 + GDPR) + 依赖管理 (NuGet + 禁止) + 工具链配置 (.editorconfig + stylecop.json + Directory.Build.props) + **强 P0-001 跟踪** (RoomData.Difficulty 静态保护 + Validate() 运行时校验)。**全链 16/16 收官 🏆** 第 6/8 文件。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft (等待 ce-doc-review 评审)
**全链状态：** 16/16 收官 🏆
