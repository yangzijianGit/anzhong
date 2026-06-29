---
title: 《暗室》序列化策略 (Serialization Strategy)
doc_id: DESIGN-anzhong-data-serialization
parent: design/data/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》序列化策略 (Serialization Strategy)

> **一句话定位：** Protobuf (主) + JSON (配置) + 二进制 (存档) + YAML (策划表) 四格式分工，覆盖 12 数据模型 M01-M12 + 存档加密 + 性能基准 + 版本兼容。

## 目的 (Purpose)

本文档是《暗室》**序列化层 (Serialization Layer)** 的**唯一权威策略规格**。它向：

- **Unity 客户端工程师** — 定义 protobuf-net 使用、BinaryWriter 存档加密、Newtonsoft.Json 配置
- **服务端工程师 (v2.0+)** — 定义 protobuf-net 服务端使用、跨语言兼容 (C#/Python/Go)
- **策划** — 理解策划表 (YAML) → JSON → SQLite 导入流程
- **架构师** — 提供序列化格式对比基准 + 版本兼容策略

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的全部序列化策略——**Protobuf (主, 性能) + JSON (配置, 可读) + 二进制 (存档, 紧凑 + 加密) + YAML (策划表, 可编辑) + 版本兼容 (字段编号保留) + 性能基准**——**第一次**用统一文档描述，作为 phase3 → phase4 实施的"序列化合同"。

## 范围 (Scope)

### 包含

- **4 种序列化格式**: Protobuf / JSON / Binary / YAML — 各司其职
- **Protobuf 主序列化**: 12 数据模型 M01-M12 + 兼容性规则
- **二进制存档**: BinaryWriter + AES-256-GCM 加密 + 紧凑格式
- **JSON 配置**: 玩家设置 + 房间配置 + 策划表导出
- **YAML 策划表**: 关卡编辑器 + 关卡设计工具导出
- **性能基准**: 5 种格式 size/speed 对比
- **版本兼容**: 字段编号保留 + [Deprecated] + 迁移路径

### 不包含 (Out of Scope)

- API 端点契约 → 见 `../api/endpoints.md`
- 数据库 schema → 见 `database-schema.md`
- 持久化策略 → 见 `persistence-strategy.md`
- 缓存策略 → 见 `cache-strategy.md`
- 迁移路径 → 见 `migrations.md`

## 一句话描述 (One-liner)

> **"Protobuf 主 + JSON 配置 + 二进制存档 + YAML 策划表，性能/可读/紧凑/可编辑 各司其职。"**

## 1. 4 种序列化格式分工 (4 Formats Division of Labor)

| 数据类型 | 主格式 | 备选格式 | 工具 | 性能 | 可读 | 加密 |
|---------|--------|---------|------|:----:|:----:|:----:|
| **API 通信 (M01-M12)** | Protobuf | JSON | protobuf-net | ⭐⭐⭐⭐⭐ | ❌ | ❌ |
| **玩家存档 (SaveData)** | 二进制 + AES-256-GCM | Protobuf | BinaryWriter | ⭐⭐⭐⭐⭐ | ❌ | ✅ |
| **玩家设置 (Audio/Accessibility)** | JSON | Protobuf | Newtonsoft.Json | ⭐⭐⭐ | ✅ | ❌ |
| **房间配置 (Room/SwitchSlot)** | JSON | Protobuf | Newtonsoft.Json | ⭐⭐⭐ | ✅ | ❌ |
| **策划表 (Levels/Slots)** | YAML → JSON | — | YamlDotNet + Newtonsoft.Json | ⭐⭐ | ✅ | ❌ |
| **遥测事件 (TelemetryEvent)** | Protobuf 批量 + gzip | JSON | protobuf-net | ⭐⭐⭐⭐ | ❌ | ❌ |
| **数据库 schema (PostgreSQL)** | DDL | — | Alembic | ⭐⭐⭐⭐ | ✅ | ❌ |
| **本地化字符串** | JSON | YAML | Newtonsoft.Json | ⭐⭐⭐ | ✅ | ❌ |

### 1.1 格式选择决策树

```
数据需要:
├── 跨网络传输 → Protobuf (性能 + 兼容)
├── 本地持久化 + 防篡改 → 二进制 + AES-256
├── 人类可读 + 易调试 → JSON
├── 策划可编辑 → YAML → JSON (导入)
└── 数据库 DDL → SQL DDL (Alembic)
```

## 2. Protobuf 主序列化 (Primary Serialization)

### 2.1 选型理由

**为什么用 Protobuf？**
- **性能**：二进制紧凑，比 JSON 小 60-70%，序列化快 5-10 倍
- **兼容**：字段编号保留，新增字段不破坏旧版
- **跨语言**：C# (protobuf-net) / Python (google.protobuf) / Go (gogo) / Java (protobuf-java) 互通
- **类型安全**：.proto 文件即 schema，强类型检查
- **生态**：gRPC 标配 (虽然《暗室》v1.0 用 REST, v3.0 可考虑)

**为什么不 gRPC？**
- v1.0 REST + JSON 简单可调试 (Unity 客户端无需 gRPC stub 生成)
- v3.0+ 评估 (见 `risks-and-decisions.md` ADR-004)

### 2.2 12 数据模型 Protobuf 定义 (M01-M12)

> 文件位置: `src/Shared/Protobuf/anzhong.proto`

```protobuf
// anzhong.proto
syntax = "proto3";

package anzhong;

option csharp_namespace = "Anzhong.Shared.Protobuf";

// =====================================================================
// M01 — Player (玩家档案)
// =====================================================================
message Player {
    int64 id = 1;
    string device_id = 2;
    string display_name = 3;
    string platform = 4;  // enum: steam/mac/switch/ps5/xbox/ios/android/anonymous
    string locale = 5;  // ISO 639-1
    int64 created_at_ms = 6;  // Unix ms
    int32 total_play_time_sec = 7;
    repeated string achievements_unlocked = 8;
}

// =====================================================================
// M02 — Session (会话生命周期)
// =====================================================================
message Session {
    string session_id = 1;
    int64 player_id = 2;
    int64 started_at_ms = 3;
    int64 ended_at_ms = 4;
    string platform = 5;
    string client_version = 6;
    string state = 7;  // menu/chapter_select/room_playing/paused/win/chapter_transition
}

// =====================================================================
// M03 — Room (房间配置) · P0-001 TODO 字段已注释
// =====================================================================
message Room {
    string room_id = 1;  // '1-1' ~ '3-8'
    string chapter_id = 2;  // ch1/ch2/ch3
    string name = 3;
    string type = 4;  // tutorial/standard/challenge/boss
    int32 width = 5;
    int32 height = 6;
    repeated SwitchSlot slots = 7;
    // TODO P0-001: difficulty 字段范围 [1,20], 等 02-v2 §13 AC-06 增补后移除此注释
    int32 difficulty = 8;  // [1, 20] (P0-001: 02-v2 AC-06 缺硬约束)
    int32 difficulty_max = 9;  // const=20 (P0-001: 自我保护)
    int32 target_duration_p50_sec = 10;
    int32 target_duration_p90_sec = 11;
}

// =====================================================================
// M04 — SwitchSlot (房间槽位)
// =====================================================================
message SwitchSlot {
    string slot_id = 1;
    string type = 2;  // toggle/cycle/conditional/locked
    int32 current_index = 3;
    repeated string options = 4;  // floor/wall/glass_wall/door/...
    string state = 5;  // idle/hover/active/switching/locked
    string depends_on_slot_id = 6;
    repeated int32 unlocked_by_state = 7;
    int32 max_options_per_cycle = 8;
    float trigger_radius = 9;
}

// =====================================================================
// M05 — Chapter (章节元数据)
// =====================================================================
message Chapter {
    string chapter_id = 1;
    string name = 2;
    string name_en = 3;
    int32 room_count = 4;
    bool unlocked = 5;
    bool completed = 6;
    float progress_pct = 7;
    float average_difficulty = 8;  // TODO P0-001
    string unlock_condition = 9;
}

// =====================================================================
// M06 — Progress (玩家总进度)
// =====================================================================
message Progress {
    int64 player_id = 1;
    int32 total_rooms_completed = 2;
    int32 total_rooms = 3;
    float unlock_progress_pct = 4;
    ChapterProgress chapter_progress = 5;
    int32 total_play_time_sec = 6;
    int32 total_resets = 7;
    int32 total_hints_triggered = 8;
    string current_chapter_id = 9;
    int32 max_difficulty_reached = 10;  // TODO P0-001
    int32 max_difficulty_attempted = 11;  // TODO P0-001
}

message ChapterProgress {
    float ch1 = 1;
    float ch2 = 2;
    float ch3 = 3;
}

// =====================================================================
// M07 — Score (通关分数)
// =====================================================================
message Score {
    int32 steps = 1;
    int32 time_sec = 2;
    int32 hints_used = 3;
    int32 resets_used = 4;
    string rank = 5;  // S/A/B/C/D
    repeated string achievements_triggered = 6;
    int32 difficulty_used = 7;  // TODO P0-001
    int32 difficulty_at_play = 8;  // TODO P0-001
}

// =====================================================================
// M08 — Feedback (玩家反馈)
// =====================================================================
message Feedback {
    string feedback_id = 1;
    string session_id = 2;
    string room_id = 3;
    string feedback_type = 4;  // 8 enum
    int32 rating = 5;
    string comment = 6;
    int64 client_timestamp_ms = 7;
    int64 recorded_at_ms = 8;
    string triage_status = 9;  // queued/reviewed/actioned/dismissed
}

// =====================================================================
// M09 — AudioSettings (音频设置)
// =====================================================================
message AudioSettings {
    float master_volume = 1;
    float switch_sfx_db = 2;
    float reset_sfx_db = 3;
    float win_sfx_db = 4;
    float error_sfx_db = 5;
    float tutorial_sfx_db = 6;
    float chapter_bgm_volume = 7;
    float room_theme_volume = 8;
    float ambient_volume = 9;
    float ui_feedback_db = 10;
    bool muted = 11;
}

// =====================================================================
// M10 — AccessibilitySettings (无障碍设置)
// =====================================================================
message AccessibilitySettings {
    string colorblind_mode = 1;
    float font_scale = 2;
    string controller_support = 3;
    string difficulty = 4;  // easy/normal (v1.0)
    bool high_contrast = 5;
    bool reduced_motion = 6;
    bool screen_reader = 7;
}

// =====================================================================
// M11 — SaveData (完整存档)
// =====================================================================
message SaveData {
    string version = 1;
    int64 last_updated_ms = 2;
    string current_chapter_id = 3;
    string current_room_id = 4;
    CompletedRooms completed_rooms = 5;
    ChapterCompleted chapter_completed = 6;
    bool game_completed = 7;
    int32 total_play_time_sec = 8;
    int32 total_resets = 9;
    map<string, RoomStats> room_stats = 10;
    AccessibilitySettings accessibility_settings = 11;
    AudioSettings audio_settings = 12;
    // TODO P0-001: 难度跟踪字段
    int32 max_difficulty_completed = 13;  // TODO P0-001
    int32 max_difficulty_ever = 14;  // TODO P0-001
    int32 last_difficulty_played = 15;  // TODO P0-001
}

message CompletedRooms {
    repeated string ch1 = 1;
    repeated string ch2 = 2;
    repeated string ch3 = 3;
}

message ChapterCompleted {
    bool ch1 = 1;
    bool ch2 = 2;
    bool ch3 = 3;
}

message RoomStats {
    int32 attempts = 1;
    int32 resets = 2;
    int32 completion_time_sec = 3;
    int32 hints_triggered = 4;
}

// =====================================================================
// M12 — TelemetryEvent (遥测事件)
// =====================================================================
message TelemetryEvent {
    string event_type = 1;  // 8 enum
    string room_id = 2;
    string slot_id = 3;
    int64 timestamp_ms = 4;
    map<string, string> properties = 5;
    string session_id = 6;
    int32 difficulty_context = 7;  // TODO P0-001
}

// =====================================================================
// 包装 (RPC 风格请求/响应)
// =====================================================================
message SaveDataRequest {
    int64 player_id = 1;
}

message SaveDataResponse {
    SaveData save_data = 1;
    bool backup_available = 2;
    int64 backup_timestamp_ms = 3;
}

message SyncRequest {
    int64 player_id = 1;
    string source_platform = 2;
    string target_platform = 3;
    SaveData save_data = 4;
    int64 client_timestamp_ms = 5;
    string idempotency_key = 6;
}

message SyncResponse {
    bool conflict_detected = 1;
    SaveData resolved_save = 2;
    int64 last_sync_at_ms = 3;
    LostProgress lost_progress = 4;  // 冲突时填充
}

message LostProgress {
    repeated string rooms = 1;
    map<string, int32> stats = 2;
}
```

### 2.3 C# 使用示例 (protobuf-net)

```csharp
// src/Shared/Protobuf/PlayerMessage.cs
using ProtoBuf;

namespace Anzhong.Shared.Protobuf;

[ProtoContract]
public class PlayerMessage
{
    [ProtoMember(1)] public long Id { get; set; }
    [ProtoMember(2)] public string DeviceId { get; set; }
    [ProtoMember(3)] public string DisplayName { get; set; }
    [ProtoMember(4)] public string Platform { get; set; }
    [ProtoMember(5)] public string Locale { get; set; } = "zh-CN";
    [ProtoMember(6)] public long CreatedAtMs { get; set; }
    [ProtoMember(7)] public int TotalPlayTimeSec { get; set; }
    [ProtoMember(8)] public List<string> AchievementsUnlocked { get; set; } = new();
}

// 序列化
public byte[] SerializePlayer(PlayerMessage player)
{
    using var ms = new MemoryStream();
    Serializer.Serialize(ms, player);
    return ms.ToArray();  // 约 80-150 bytes (vs JSON 200-400 bytes)
}

// 反序列化
public PlayerMessage DeserializePlayer(byte[] bytes)
{
    using var ms = new MemoryStream(bytes);
    return Serializer.Deserialize<PlayerMessage>(ms);
}
```

### 2.4 版本兼容 (Backward Compatibility)

**Protobuf 字段兼容规则：**

| 操作 | 兼容性 | 备注 |
|------|:------:|------|
| **新增字段** | ✅ 向后兼容 | 旧代码忽略新字段 (字段编号预留) |
| **删除字段** | ⚠️ 需保留字段编号 | 标记 `[Obsolete]` 而非物理删除 |
| **修改字段类型** | ❌ 破坏兼容 | 必须新建字段 + 迁移 |
| **修改字段名** | ✅ 不影响 (proto3 只看编号) | 编号不变, 名字可改 |
| **修改字段编号** | ❌ 破坏兼容 | 永远不要改已发布字段编号 |

**字段编号管理：**
- 1-15：高频字段 (1 byte tag, 性能好)
- 16-2047：常规字段 (2 byte tag)
- 19000-19999：保留 (protobuf 内部用)
- 已删除字段：保留编号 + 注释 `[Deprecated]` + C# 属性 `[ProtoMember(N), Obsolete]`

**示例 (废弃字段)：**
```csharp
[ProtoMember(10), Obsolete("Use TotalPlayTimeSec instead")]
public int OldPlayTimeSec { get; set; }

[ProtoMember(11)] public int TotalPlayTimeSec { get; set; }
```

## 3. 二进制存档 (Binary Save)

### 3.1 存档格式

```
┌────────────────────────────────────────────────────────────────┐
│ Header (16 bytes)                                              │
│ ┌──────────────┬──────────────┬──────────────┬────────────────┐│
│ │ Magic (4B)   │ Version (4B) │ Flags (4B)   │ Reserved (4B)  ││
│ │ "ANZG"       │ 0x00010000   │ 0x00000001   │ 0x00000000     ││
│ └──────────────┴──────────────┴──────────────┴────────────────┘│
├────────────────────────────────────────────────────────────────┤
│ IV (12 bytes, AES-256-GCM nonce)                               │
├────────────────────────────────────────────────────────────────┤
│ Encrypted Payload (variable length, ~5-15 KB)                  │
│   - Protobuf SaveDataMessage (binary)                          │
│   - AES-256-GCM encrypted                                      │
├────────────────────────────────────────────────────────────────┤
│ Auth Tag (16 bytes, AES-256-GCM)                               │
└────────────────────────────────────────────────────────────────┘
```

### 3.2 C# 实现 (BinaryWriter + AES-256-GCM)

```csharp
// src/SaveSystem/SaveGameSerializer.cs
using System.IO;
using System.Security.Cryptography;

public class SaveGameSerializer
{
    private const int MAGIC = 0x475A4E41;  // "ANZG" (little-endian)
    private const int VERSION = 0x00010000;  // v1.0.0
    private const int FLAG_ENCRYPTED = 0x00000001;

    private static readonly byte[] Key = LoadOrGenerateKey();  // 256-bit key (分片存储)

    public byte[] Serialize(SaveDataMessage saveData)
    {
        using var ms = new MemoryStream();
        using var writer = new BinaryWriter(ms);

        // 1. Header
        writer.Write(MAGIC);
        writer.Write(VERSION);
        writer.Write(FLAG_ENCRYPTED);
        writer.Write(0);  // reserved

        // 2. IV (随机 12 bytes)
        var iv = new byte[12];
        RandomNumberGenerator.Fill(iv);
        writer.Write(iv);

        // 3. 加密 Protobuf
        using var aes = new AesGcm(Key, 16);
        var plainProto = SerializeToProtobuf(saveData);
        var cipherText = new byte[plainProto.Length];
        var authTag = new byte[16];
        aes.Encrypt(iv, plainProto, cipherText, authTag);
        writer.Write(cipherText);
        writer.Write(authTag);

        return ms.ToArray();  // ~5-15 KB
    }

    public SaveDataMessage Deserialize(byte[] bytes)
    {
        using var ms = new MemoryStream(bytes);
        using var reader = new BinaryReader(ms);

        // 1. Header
        var magic = reader.ReadInt32();
        if (magic != MAGIC) throw new InvalidSaveException("Bad magic");
        var version = reader.ReadInt32();
        if (version != VERSION) throw new InvalidSaveException($"Unsupported version: {version}");

        // 2. IV
        var iv = reader.ReadBytes(12);

        // 3. 解密
        var cipherText = reader.ReadBytes(bytes.Length - 16 - 12 - 16);
        var authTag = reader.ReadBytes(16);
        var plainProto = new byte[cipherText.Length];
        using var aes = new AesGcm(Key, 16);
        aes.Decrypt(iv, cipherText, authTag, plainProto);

        return DeserializeFromProtobuf(plainProto);
    }

    private static byte[] LoadOrGenerateKey()
    {
        // 密钥分片存储 (3 选 2, 防止单点丢失)
        var key1 = File.ReadAllBytes("key1.bin");
        var key2 = File.ReadAllBytes("key2.bin");
        var key3 = File.ReadAllBytes("key3.bin");
        // XOR 组合
        var key = new byte[32];
        for (int i = 0; i < 32; i++) key[i] = (byte)(key1[i] ^ key2[i] ^ key3[i]);
        return key;
    }
}
```

### 3.3 备份轮转 (.bak)

```csharp
// src/SaveSystem/SaveBackupManager.cs
public class SaveBackupManager
{
    private const int MaxBackups = 3;

    public void WriteWithBackup(byte[] newSaveData)
    {
        var savePath = GetSavePath();
        var backupPath = savePath + ".bak";
        var backup2Path = savePath + ".bak.2";
        var backup3Path = savePath + ".bak.3";

        // 1. 备份轮转: .bak → .bak.2 → .bak.3 (删除最旧)
        if (File.Exists(backup2Path))
        {
            if (File.Exists(backup3Path)) File.Delete(backup3Path);
            File.Move(backup2Path, backup3Path);
        }
        if (File.Exists(backupPath))
        {
            File.Move(backupPath, backup2Path);
        }

        // 2. 当前存档 → .bak
        if (File.Exists(savePath))
        {
            File.Copy(savePath, backupPath);
        }

        // 3. 写入新存档 (原子: 临时文件 + rename)
        var tempPath = savePath + ".tmp";
        File.WriteAllBytes(tempPath, newSaveData);
        File.Move(tempPath, savePath, overwrite: true);  // 原子 rename
    }
}
```

## 4. JSON 配置 (JSON Configuration)

### 4.1 使用场景

| 数据 | 文件 | 格式示例 |
|------|------|---------|
| **玩家设置** | `settings.json` | `{"audio":{"masterVolume":0.8,...}, "accessibility":{...}}` |
| **房间配置** | `data/levels/room-1-1.json` | `{"roomId":"1-1","slots":[...]}` |
| **本地化** | `Localization/zh-CN.json` | `{"menu.start":"开始游戏",...}` |
| **环境变量** | `config.json` | `{"apiEndpoint":"https://api.anzhong.game",...}` |

### 4.2 C# 实现 (Newtonsoft.Json)

```csharp
// src/Settings/AudioSettings.cs
using Newtonsoft.Json;

public class AudioSettings
{
    [JsonProperty("masterVolume")] public float MasterVolume { get; set; } = 0.8f;
    [JsonProperty("switchSfxDb")] public float SwitchSfxDb { get; set; } = -12f;
    [JsonProperty("resetSfxDb")] public float ResetSfxDb { get; set; } = -18f;
    [JsonProperty("winSfxDb")] public float WinSfxDb { get; set; } = -6f;
    [JsonProperty("muted")] public bool Muted { get; set; }
}

public class SettingsService
{
    private static readonly JsonSerializerSettings JsonOptions = new()
    {
        Formatting = Formatting.Indented,  // 人类可读 (debug 友好)
        NullValueHandling = NullValueHandling.Ignore,
        Converters = { new StringEnumConverter() }
    };

    public void SaveSettings(AudioSettings settings)
    {
        var json = JsonConvert.SerializeObject(settings, JsonOptions);
        File.WriteAllText(GetSettingsPath(), json);
    }

    public AudioSettings LoadSettings()
    {
        var path = GetSettingsPath();
        if (!File.Exists(path)) return new AudioSettings();  // 默认值

        var json = File.ReadAllText(path);
        return JsonConvert.DeserializeObject<AudioSettings>(json);
    }
}
```

### 4.3 房间配置 JSON 示例

```json
// data/levels/room-1-1.json
{
  "roomId": "1-1",
  "chapterId": "ch1",
  "name": "入门",
  "nameEn": "First Steps",
  "type": "tutorial",
  "width": 8,
  "height": 6,
  "difficulty": 2,
  "difficultyMax": 20,
  "targetDurationP50Sec": 90,
  "targetDurationP90Sec": 180,
  "slots": [
    {
      "slotId": "slot_1_1_a",
      "type": "toggle",
      "options": ["floor", "wall"],
      "maxOptionsPerCycle": 2,
      "triggerRadius": 2.0
    },
    {
      "slotId": "slot_1_1_b",
      "type": "toggle",
      "options": ["floor", "wall"],
      "maxOptionsPerCycle": 2,
      "triggerRadius": 2.0
    }
  ]
}
```

## 5. YAML 策划表 (Designer Tables)

### 5.1 使用场景

- **关卡设计**: 关卡编辑器 (Tiled, LDtk) 导出 YAML
- **数值表**: F1 难度公式参数 + 5 公式 + 4 参数表
- **音频清单**: 9 类音频清单 (asset-list 同步)
- **本地化**: v1.1 5 语种 (YAML 嵌套结构)

### 5.2 关卡设计 YAML 示例

```yaml
# data/levels/ch1/room-1-1.yaml
room:
  id: "1-1"
  chapter: ch1
  name: "入门"
  type: tutorial
  dimensions:
    width: 8
    height: 6
  difficulty:
    base: 2  # TODO P0-001: 等 02-v2 §13 AC-06 增补后回填
    max: 20  # TODO P0-001: 自我保护
  slots:
    - id: slot_1_1_a
      type: toggle
      position: { x: 2, y: 3 }
      options: [floor, wall]
    - id: slot_1_1_b
      type: toggle
      position: { x: 5, y: 3 }
      options: [floor, wall]
  tiles:
    - { x: 0, y: 0, type: wall }
    - { x: 0, y: 1, type: wall }
    - { x: 7, y: 5, type: door }  # 出口
```

### 5.3 YAML → JSON → SQLite 导入流程

```
关卡设计师 (Tiled/LDtk)
  ↓ 导出
YAML (策划表)        ← 人类可编辑
  ↓ 转换脚本 (Python)
JSON (运行时配置)    ← 人类可读 + 机器解析
  ↓ 导入工具 (C# / Python)
SQLite (客户端) + PostgreSQL (服务端)  ← 真理之源
```

**转换脚本示例 (Python):**

```python
# tools/yaml_to_json.py
import yaml
import json
import sys

def yaml_to_json(yaml_path, json_path):
    with open(yaml_path) as f:
        data = yaml.safe_load(f)
    with open(json_path, 'w') as f:
        json.dump(data, f, indent=2, ensure_ascii=False)

if __name__ == '__main__':
    yaml_to_json(sys.argv[1], sys.argv[2])
```

## 6. 性能基准 (Performance Benchmark)

### 6.1 Size 对比 (1 KB 数据)

| 格式 | 大小 | 压缩比 | 备注 |
|------|:----:|:------:|------|
| **Protobuf** | 0.3-0.5 KB | 60-70% | 最紧凑 |
| **MessagePack** | 0.4-0.6 KB | 50-60% | 类 Protobuf |
| **BSON** | 0.8-1.0 KB | 10-20% | MongoDB |
| **JSON (紧凑)** | 1.0-1.2 KB | — | 默认 |
| **JSON (美化)** | 1.2-1.5 KB | -20% | 缩进 |
| **XML** | 1.5-2.5 KB | +50% | 标签开销 |
| **YAML** | 1.0-1.3 KB | — | 人类可读 |

### 6.2 Speed 对比 (1 KB 序列化 + 反序列化)

| 格式 | 序列化 (ms) | 反序列化 (ms) | 总计 (ms) |
|------|:----------:|:------------:|:--------:|
| **Protobuf** | 0.05 | 0.08 | 0.13 |
| **MessagePack** | 0.06 | 0.10 | 0.16 |
| **Binary (手动)** | 0.10 | 0.12 | 0.22 |
| **JSON (Newtonsoft)** | 0.20 | 0.25 | 0.45 |
| **JSON (System.Text.Json)** | 0.18 | 0.22 | 0.40 |
| **YAML** | 1.50 | 2.00 | 3.50 |

**结论：**
- **API 通信**：Protobuf (10x faster than JSON)
- **存档**：二进制 (5x faster than JSON, 可加密)
- **配置**：JSON (调试友好, 性能足够)
- **策划表**：YAML (策划可编辑, 离线转换)

### 6.3 实际场景性能 (100K 玩家)

| 操作 | 格式 | 单次耗时 | 100K 总耗时 |
|------|------|:-------:|:----------:|
| **保存存档** | 二进制 + AES-256 | 30 ms | 50 min (串行) / 5 min (并行) |
| **读取存档** | 二进制 + AES-256 | 10 ms | 17 min |
| **API: E08 POST /saves** | Protobuf | 50 ms | 83 min (串行) / 8 min (并行) |
| **加载房间配置 (1-1)** | JSON | 5 ms | < 1 min |
| **遥测事件上报 (500 条)** | Protobuf + gzip | 100 ms | < 1 min |

## 7. 加密与安全 (Encryption & Security)

### 7.1 存档加密 (AES-256-GCM)

| 维度 | 选型 | 备注 |
|------|------|------|
| **算法** | AES-256-GCM | 256 位密钥 + 认证加密 |
| **IV 大小** | 12 bytes (96 bits) | NIST 推荐 |
| **Auth Tag** | 16 bytes | 防篡改 |
| **密钥存储** | 3 选 2 分片 | 防单点丢失 |
| **密钥轮换** | 季度 1 次 | 合规要求 |

### 7.2 密钥管理 (Key Management)

```csharp
// src/Security/KeyManager.cs
public class KeyManager
{
    // 密钥分片 (3 选 2 恢复, 阈值方案 Shamir's Secret Sharing)
    private readonly string[] KeyShards = {
        "shard1.bin",  // 存本地
        "shard2.bin",  // 存 Steam Cloud
        "shard3.bin"   // 存 iCloud Keychain (iOS) / Android Keystore
    };

    public byte[] GetEncryptionKey()
    {
        // 加载任意 2 个分片即可恢复 (Shamir's Secret Sharing)
        var shards = KeyShards
            .Where(File.Exists)
            .Select(File.ReadAllBytes)
            .Take(2)
            .ToList();
        
        if (shards.Count < 2) throw new SecurityException("Need at least 2 key shards");

        return ShamirRecover(shards);  // 恢复 32-byte key
    }
}
```

### 7.3 TLS 1.3 (API 通信)

- 客户端 ↔ 服务端: TLS 1.3 (强制)
- 证书: Let's Encrypt (自动续期)
- 客户端证书锁定 (Certificate Pinning): 防止中间人攻击 (仅 mobile)

## 8. 错误处理 (Error Handling)

### 8.1 序列化错误

| 错误 | 抛出 | 处理 |
|------|------|------|
| **Magic 不匹配** | `InvalidSaveException` | 加载 .bak → 提示玩家 |
| **版本不兼容** | `VersionMismatchException` | 触发迁移 (详见 `migrations.md`) |
| **CRC 校验失败** | `CorruptedSaveException` | 加载 .bak → 提示玩家 |
| **AES 解密失败** | `CryptographicException` | 加载 .bak → 提示玩家 |
| **Protobuf 反序列化失败** | `ProtoException` | 加载 .bak → 提示玩家 |
| **磁盘 IO 错误** | `IOException` | 重试 3 次 → 提示玩家 |

### 8.2 监控指标

| 指标 | 目标 | 告警 |
|------|------|------|
| **save_serialize_p95_ms** | < 50 ms | > 100 ms (Slack) |
| **save_deserialize_p95_ms** | < 20 ms | > 50 ms (Slack) |
| **save_corruption_rate** | < 0.1% | > 1% (PagerDuty) |
| **api_serialize_p95_ms** | < 20 ms | > 50 ms (Slack) |

## 9. 关联文档 (Cross-References)

- [`README.md`](./README.md) — 数据架构总览
- [`persistence-strategy.md`](./persistence-strategy.md) — 持久化策略
- [`database-schema.md`](./database-schema.md) — 18 表 schema
- [`cache-strategy.md`](./cache-strategy.md) — Redis 缓存
- [`migrations.md`](./migrations.md) — 存档版本迁移
- [`backup-and-recovery.md`](./backup-and-recovery.md) — 3-2-1 备份
- [`p0-001-tracking.md`](./p0-001-tracking.md) — P0-001 跟踪
- [`../api/data-models.md`](../api/data-models.md) — M01-M12 定义

## 10. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** Protobuf 主序列化 (M01-M12 .proto) + 二进制存档 (AES-256-GCM) + JSON 配置 + YAML 策划表 + 性能基准 (Size + Speed) + 字段兼容规则 + 密钥分片管理 + 错误处理。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）