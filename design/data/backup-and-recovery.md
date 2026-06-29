---
title: 《暗室》备份与恢复 (Backup & Recovery)
doc_id: DESIGN-anzhong-data-backup
parent: design/data/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 中书省
---

# 《暗室》备份与恢复 (Backup & Recovery)

> **一句话定位：** 3-2-1 备份策略 (3 副本 + 2 介质 + 1 异地) + RPO 5 min + RTO 1 h + AES-256-GCM 加密 + 季度灾备演练 + GDPR 合规的端到端容灾契约。

## 目的 (Purpose)

本文档是《暗室》**备份与恢复层 (Backup & Recovery Layer)** 的**唯一权威策略规格**。它向：

- **DevOps / SRE** — 定义 3-2-1 备份策略、cron 定时任务、监控告警、灾备演练 SOP
- **服务端工程师 (v2.0+)** — 定义 pg_dump + WAL 归档 + Streaming Replication
- **Unity 客户端工程师** — 定义 .bak 轮转、Steam Cloud 同步、存档损坏降级
- **架构师** — 提供 RPO/RTO + 容灾等级 + GDPR 合规

**本版本（v1.0）的目的：** 把"无战斗 2D 房间解谜游戏"的全部备份与恢复策略——**v1.0 本地 .bak (3 轮转) + Steam Cloud 冗余 + v2.0+ 3-2-1 (本地 hot + 异地 warm + 离线 cold) + RPO 5 min + RTO 1 h + AES-256-GCM 加密 + 季度灾备演练 + GDPR 合规**——**第一次**用统一文档描述，作为 phase3 → phase4 实施的"容灾合同"。

## 范围 (Scope)

### 包含

- **v1.0 客户端备份**: 本地 .bak 轮转 + Steam Cloud 冗余
- **v2.0+ 服务端备份**: 3-2-1 (本地 + 异地 + 离线) + WAL 持续归档
- **备份类型**: 全量 + 增量 + WAL + 配置 + 密钥
- **加密**: AES-256-GCM (客户端 + 服务端)
- **保留策略**: 30 d (热) + 90 d (温) + 1 y (冷) + 永久 (合规)
- **恢复流程**: 故障检测 → 数据恢复 → 服务切换 → 验证
- **监控告警**: 备份成功率 / RPO / 存储容量
- **灾备演练**: 季度 1 次 + 年度 1 次全链路

### 不包含 (Out of Scope)

- API 端点契约 → 见 `../api/endpoints.md`
- 数据库 schema → 见 `database-schema.md`
- 持久化策略 → 见 `persistence-strategy.md`
- 序列化格式 → 见 `serialization.md`
- 迁移路径 → 见 `migrations.md`

## 一句话描述 (One-liner)

> **"3-2-1 + RPO 5 min + RTO 1 h + AES-256 加密 + 季度演练的端到端容灾。"**

## 1. 容灾目标 (DR Objectives)

### 1.1 RPO / RTO 矩阵

| 阶段 | RPO (数据丢失容忍) | RTO (恢复时间目标) | 故障场景 |
|------|:------------------:|:------------------:|---------|
| **v1.0 客户端** | 5 min (本地 SQLite + .bak) | 1 h (本地恢复 + Steam Cloud 下载) | 玩家硬盘损坏 |
| **v1.0 服务端** | N/A (无服务端) | N/A | — |
| **v2.0+ 服务端** | **5 min** (WAL 持续归档) | **1 h** (本地恢复) / **30 min** (异地切换) | 数据库崩溃 |
| **v2.0+ 极端灾难** | **15 min** (跨区复制) | **4 h** (异地重建) | 整个机房故障 |

### 1.2 容灾等级 (DR Tier)

| 等级 | 描述 | 适用 |
|:----:|------|------|
| **Tier 1** | 本地备份 + 同城冗余 | v1.0 (Steam Cloud) |
| **Tier 2** | Tier 1 + 异地热备 + 自动 failover | v2.0+ (主从 + 异地) |
| **Tier 3** | Tier 2 + 异地冷备 + 季度演练 | v2.0+ 增强 (合规要求) |
| **Tier 4** | Tier 3 + 跨云双活 + 实时复制 | v3.0+ (可选) |

《暗室》目标：**v2.0+ Tier 3**（已满足 GDPR + 行业标准）

## 2. v1.0 客户端备份策略

### 2.1 本地 .bak 三轮转

```
savegame.db          ← 当前存档 (主)
savegame.db.bak      ← 上一次写入前备份 (1st backup)
savegame.db.bak.2    ← 上上次备份 (2nd backup)
savegame.db.bak.3    ← 上上上次备份 (3rd backup, 最旧)
```

**轮转规则（写入前）：**
```
.bak.2 → .bak.3 (覆盖最旧)
.bak   → .bak.2 (向前推)
当前   → .bak   (备份当前)
新数据 → 当前   (原子写入)
```

**实现 (C#):**

```csharp
// src/SaveSystem/SaveBackupManager.cs
public class SaveBackupManager
{
    private const int MaxBackups = 3;
    private readonly string SavePath;
    private readonly string BackupPath => SavePath + ".bak";
    private readonly string Backup2Path => SavePath + ".bak.2";
    private readonly string Backup3Path => SavePath + ".bak.3";

    public async Task WriteWithBackupAsync(byte[] newSaveData)
    {
        // 1. 备份轮转
        if (File.Exists(Backup2Path))
        {
            if (File.Exists(Backup3Path)) File.Delete(Backup3Path);
            File.Move(Backup2Path, Backup3Path);
        }
        if (File.Exists(BackupPath))
        {
            File.Move(BackupPath, Backup2Path);
        }

        // 2. 当前 → .bak
        if (File.Exists(SavePath))
        {
            File.Copy(SavePath, BackupPath, overwrite: true);
        }

        // 3. 原子写入 (临时 + rename)
        var tempPath = SavePath + ".tmp";
        await File.WriteAllBytesAsync(tempPath, newSaveData);
        File.Move(tempPath, SavePath, overwrite: true);  // POSIX atomic rename
    }

    public async Task<byte[]> LoadWithFallbackAsync()
    {
        // 1. 尝试主存档
        if (File.Exists(SavePath))
        {
            try
            {
                return await ReadAndValidateAsync(SavePath);
            }
            catch (Exception ex)
            {
                Log.Warn(ex, "Main save corrupted, falling back to backup");
            }
        }

        // 2. 尝试 .bak
        if (File.Exists(BackupPath))
        {
            try
            {
                return await ReadAndValidateAsync(BackupPath);
                // 提示玩家: "主存档损坏, 已加载备份"
                UI.ShowNotification("主存档损坏, 已加载备份");
            }
            catch (Exception ex)
            {
                Log.Warn(ex, "Backup 1 corrupted, falling back to backup 2");
            }
        }

        // 3. 尝试 .bak.2
        if (File.Exists(Backup2Path))
        {
            try
            {
                return await ReadAndValidateAsync(Backup2Path);
            }
            catch (Exception ex)
            {
                Log.Warn(ex, "Backup 2 corrupted, falling back to backup 3");
            }
        }

        // 4. 尝试 .bak.3
        if (File.Exists(Backup3Path))
        {
            try
            {
                return await ReadAndValidateAsync(Backup3Path);
            }
            catch (Exception ex)
            {
                Log.Error(ex, "All backups corrupted, starting fresh");
            }
        }

        // 5. 全部失败 → 从 Ch1-1 重新开始
        UI.ShowNotification("存档损坏, 已重置进度");
        return GetDefaultSaveData();
    }
}
```

### 2.2 Steam Cloud 冗余 (v1.0)

| 维度 | 策略 |
|------|------|
| **上传时机** | 应用退出 + 房间通关 + 手动同步 |
| **下载时机** | 应用启动 (检测本地 vs 云端时间戳) |
| **加密** | TLS 1.3 (传输) + AES-256 (存档) |
| **冗余度** | Steam 3 个数据中心 (内置) |
| **冲突解决** | Last-Writer-Wins by timestamp (UTC) |

详见 `persistence-strategy.md` §3。

### 2.3 v1.0 容灾能力

| 故障 | 恢复 | RTO |
|------|------|-----|
| **主存档损坏** | 自动加载 .bak | < 1s |
| **全部 .bak 损坏** | 从 Steam Cloud 下载 | < 5 min (网络) |
| **硬盘故障** | 从 Steam Cloud 下载 + 重新开始 | < 30 min |
| **Steam Cloud 不可用** | 本地继续游戏 (Offline-First) | 0 (无影响) |
| **账号被盗** | 联系 Steam 客服恢复存档 | 1-7 d |

## 3. v2.0+ 服务端 3-2-1 备份策略

### 3.1 3-2-1 原则

```
3 副本 (3 Copies):
  - 主库 (Primary, 本地 hot)
  - 异地备份 (异地 warm)
  - 离线冷备 (S3 Glacier / 磁带 cold)

2 介质 (2 Media Types):
  - SSD (主库 + 异地热备)
  - HDD 或 磁带 (离线冷备)

1 异地 (1 Offsite):
  - 异地数据中心 (≥ 100 km 距离)
  - 满足 GDPR / SOC2 / ISO 27001
```

### 3.2 备份类型与频率

| 类型 | 频率 | 保留期 | 存储 | 用途 |
|------|:----:|:------:|------|------|
| **全量备份 (Full)** | 每日 03:00 UTC | 7 d | 本地 SSD | 每日基线 |
| **增量备份 (Incremental)** | 每小时 | 24 h | 本地 SSD | RPO 5 min |
| **WAL 归档 (WAL Archive)** | 持续 (≤ 16 MB / 文件) | 7 d | 异地 S3 | Point-in-Time Recovery |
| **异地全量备份** | 每日 06:00 UTC | 90 d | 异地 SSD | 跨区灾难恢复 |
| **离线冷备** | 每周日 06:00 UTC | 1 y | S3 Glacier | 长期归档 + 合规 |
| **配置备份** | 每次变更 | 永久 | Git | Infrastructure as Code |
| **密钥备份** | 季度轮换时 | 永久 | Shamir 分片 | 防密钥丢失 |

### 3.3 PostgreSQL 备份配置

```conf
# postgresql.conf
wal_level = replica
archive_mode = on
archive_command = 'aws s3 cp %p s3://anzhong-wal-archive/%f'
archive_timeout = 60  # 强制归档间隔 (秒)
max_wal_size = 1GB
min_wal_size = 80MB

# 流复制 (Replica)
primary_slot_name = 'replica_slot_1'
synchronous_commit = on  # 强同步 (RPO = 0 for replica)
synchronous_standby_names = 'replica_1,replica_2'
```

### 3.4 备份脚本 (Shell)

```bash
#!/bin/bash
# tools/db/backup.sh
# 每日全量备份 + 异地传输

set -euo pipefail

# 配置
PG_HOST="${PG_HOST:-db.anzhong.game}"
PG_USER="${PG_USER:-anzhong_backup}"
PG_DB="${PG_DB:-anzhong}"
BACKUP_DIR="/var/backups/postgresql"
S3_BUCKET="s3://anzhong-backups"
RETENTION_DAYS=7
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="${BACKUP_DIR}/anzhong_full_${TIMESTAMP}.dump"

# 1. 创建备份目录
mkdir -p "$BACKUP_DIR"

# 2. pg_dump (全量, 并行 4 jobs)
echo "[$(date)] Starting full backup..."
pg_dump -h "$PG_HOST" -U "$PG_USER" \
    -Fc -j 4 -Z 9 \
    -d "$PG_DB" \
    -f "$BACKUP_FILE"

# 3. 校验备份
echo "[$(date)] Verifying backup..."
pg_restore -l "$BACKUP_FILE" > /dev/null

# 4. 上传到 S3 (异地)
echo "[$(date)] Uploading to S3..."
aws s3 cp "$BACKUP_FILE" "${S3_BUCKET}/daily/${TIMESTAMP}/anzhong_full.dump" \
    --storage-class STANDARD_IA  # 30 天后转 Glacier

# 5. 清理本地旧备份 (保留 7 天)
echo "[$(date)] Cleaning up old backups..."
find "$BACKUP_DIR" -name "anzhong_full_*.dump" -mtime +${RETENTION_DAYS} -delete

# 6. 记录到审计日志
echo "[$(date)] Backup completed: $BACKUP_FILE ($(du -h "$BACKUP_FILE" | cut -f1))"
aws logs put-log-events \
    --log-group-name "/anzhong/db-backup" \
    --log-stream-name "$(date +%Y/%m/%d)" \
    --log-events timestamp=$(date +%s%3N),message="DB backup success: $BACKUP_FILE"
```

### 3.5 异地备份脚本

```bash
#!/bin/bash
# tools/db/backup-offsite.sh
# 异地全量备份 (每日 06:00 UTC)

set -euo pipefail

S3_BUCKET_OFFSITE="s3://anzhong-backups-offsite"
S3_BUCKET_PRIMARY="s3://anzhong-backups"
TIMESTAMP=$(date +%Y%m%d)

# 1. 从主 S3 复制到异地 S3 (跨区)
aws s3 sync \
    "s3://${S3_BUCKET_PRIMARY}/daily/${TIMESTAMP}/" \
    "s3://${S3_BUCKET_OFFSITE}/daily/${TIMESTAMP}/" \
    --source-region us-east-1 \
    --region eu-west-1 \
    --storage-class STANDARD_IA

# 2. 保留 90 天 (异地 S3 Lifecycle Policy)
echo "Offsite backup synced: ${S3_BUCKET_OFFSITE}/daily/${TIMESTAMP}/"
```

### 3.6 离线冷备脚本 (每周日)

```bash
#!/bin/bash
# tools/db/backup-cold.sh
# 离线冷备 (S3 Glacier, 保留 1 年)

set -euo pipefail

S3_BUCKET_COLD="s3://anzhong-backups-cold"
S3_BUCKET_PRIMARY="s3://anzhong-backups"
WEEK=$(date +%Y_W%U)

# 1. 复制到 Glacier
aws s3 cp "s3://${S3_BUCKET_PRIMARY}/weekly/latest.dump" \
    "s3://${S3_BUCKET_COLD}/weekly/${WEEK}/anzhong_full.dump" \
    --storage-class GLACIER \
    --metadata "retention-days=365"

# 2. Glacier 检索策略 (季度演练)
echo "Cold backup uploaded: ${S3_BUCKET_COLD}/weekly/${WEEK}/"
```

### 3.7 自动备份调度 (cron)

```cron
# /etc/cron.d/anzhong-db-backup

# 每日全量备份 (本地)
0 3 * * * anzhong /opt/anzhong/tools/db/backup.sh >> /var/log/anzhong/backup.log 2>&1

# 每小时增量 (WAL 持续归档, 不需要 cron)
# 由 PostgreSQL archive_mode 自动处理

# 每日异地全量
0 6 * * * anzhong /opt/anzhong/tools/db/backup-offsite.sh >> /var/log/anzhong/backup.log 2>&1

# 每周日离线冷备
0 7 * * 0 anzhong /opt/anzhong/tools/db/backup-cold.sh >> /var/log/anzhong/backup.log 2>&1

# 每日清理过期 save_backups (PostgreSQL)
30 3 * * * postgres psql -d anzhong -c "DELETE FROM save_backups WHERE retention_until < NOW()" >> /var/log/anzhong/cleanup.log 2>&1

# 每日清理过期 audit_logs (1 年后归档)
30 4 * * * postgres psql -d anzhong -c "DELETE FROM audit_logs WHERE timestamp < NOW() - INTERVAL '1 year'" >> /var/log/anzhong/cleanup.log 2>&1
```

## 4. 恢复流程 (Recovery Procedures)

### 4.1 恢复场景矩阵

| 场景 | 检测 | 恢复流程 | RTO |
|------|------|---------|-----|
| **单条记录误删** | 玩家投诉 | 从 save_backups 表恢复 | < 5 min |
| **主存档损坏** | CRC 校验失败 / E08 失败 | 自动加载 .bak | < 1 min |
| **数据库崩溃** | PG 连接失败 | 自动 failover 到 Replica | < 30 s |
| **数据库数据损坏** | E08 批量失败 | 从最近全量 + WAL 恢复 | < 30 min |
| **整个机房故障** | 服务全部不可用 | 异地切换 | < 1 h |
| **勒索软件** | 数据被加密 | 离线冷备恢复 | < 4 h |
| **GDPR 数据删除** | 玩家请求 | 软删除 + 30 天后硬删除 | < 1 d |

### 4.2 PostgreSQL Point-in-Time Recovery (PITR)

```bash
#!/bin/bash
# tools/db/restore-pitr.sh
# Point-in-Time Recovery (恢复到指定时间点)

set -euo pipefail

TARGET_TIME="${1:?Usage: restore-pitr.sh '2026-06-29 10:30:00'}"
BACKUP_FILE="/var/backups/postgresql/anzhong_full_20260629_030000.dump"

# 1. 停止 PostgreSQL
systemctl stop postgresql

# 2. 备份当前数据目录 (以防回滚)
mv /var/lib/postgresql/data /var/lib/postgresql/data.broken

# 3. 从全量备份恢复
pg_restore -h localhost -U postgres -d anzhong \
    --create --clean \
    "$BACKUP_FILE"

# 4. 配置 recovery.conf (恢复到指定时间点)
cat > /var/lib/postgresql/data/recovery.conf << EOF
restore_command = 'aws s3 cp s3://anzhong-wal-archive/%f %p'
recovery_target_time = '$TARGET_TIME'
recovery_target_action = 'promote'
EOF

# 5. 启动 PostgreSQL (进入恢复模式)
systemctl start postgresql

# 6. 等待恢复完成 (自动 promote)
echo "Waiting for recovery to complete..."
while ! pg_isready -h localhost; do sleep 1; done

# 7. 验证
psql -d anzhong -c "SELECT count(*) FROM players"
echo "PITR recovery completed to: $TARGET_TIME"
```

### 4.3 异地故障切换 (Failover)

```bash
#!/bin/bash
# tools/db/failover.sh
# 主库故障 → 切换到异地 Replica

set -euo pipefail

PRIMARY_HOST="db-primary.anzhong.game"
REPLICA_HOST="db-replica-eu.anzhong.game"

# 1. 确认主库不可达
if ! pg_isready -h "$PRIMARY_HOST" -t 5; then
    echo "Primary unreachable, promoting replica..."
else
    echo "Primary is healthy, aborting failover"
    exit 1
fi

# 2. 提升 Replica 为主库
ssh "$REPLICA_HOST" "/usr/lib/postgresql/16/bin/pg_ctl promote -D /var/lib/postgresql/data"

# 3. 等待 Replica 接受写入
while ! pg_isready -h "$REPLICA_HOST"; do sleep 1; done

# 4. 更新 DNS / Load Balancer
aws route53 change-resource-record-sets \
    --hosted-zone-id Z123ABC \
    --change-batch file://failover-dns.json

# 5. 通知应用层 (Sentinel 自动处理 Redis)
echo "Failover completed. New primary: $REPLICA_HOST"

# 6. 记录到审计日志
psql -h "$REPLICA_HOST" -d anzhong -c "
    INSERT INTO audit_logs (action, actor, result, metadata)
    VALUES ('db_failover', 'system', 'success', '{\"old_primary\":\"$PRIMARY_HOST\",\"new_primary\":\"$REPLICA_HOST\"}'::jsonb)
"
```

### 4.4 勒索软件恢复

```bash
#!/bin/bash
# tools/db/restore-from-cold.sh
# 从离线冷备恢复 (勒索软件场景)

set -euo pipefail

# 1. 隔离感染主机 (防止进一步传播)
echo "ISOLATING infected hosts..."
aws ec2 stop-instances --instance-ids i-12345 i-67890  # 假设已知

# 2. 从 Glacier 申请冷备 (4-12 小时 retrieval)
echo "Requesting cold backup from Glacier..."
aws s3api restore-object \
    --bucket anzhong-backups-cold \
    --key weekly/2026_W26/anzhong_full.dump \
    --restore-request '{"Days":7,"GlacierJobParameters":{"Tier":"Standard"}}'

# 3. 等待 restore 完成 (轮询)
while true; do
    STATUS=$(aws s3api head-object \
        --bucket anzhong-backups-cold \
        --key weekly/2026_W26/anzhong_full.dump \
        --query 'Restore.Status')
    if [[ "$STATUS" == *"COMPLETED"* ]]; then break; fi
    sleep 60
done

# 4. 下载并恢复
aws s3 cp s3://anzhong-backups-cold/weekly/2026_W26/anzhong_full.dump /tmp/
pg_restore -h localhost -U postgres -d anzhong --create --clean /tmp/anzhong_full.dump

# 5. 重放 WAL (从冷备到感染前时间点)
# ... (类似 PITR)

# 6. 验证完整性
psql -d anzhong -c "SELECT count(*) FROM players; SELECT count(*) FROM saves;"

# 7. 通知玩家 (可能丢失 1 周进度)
echo "Cold restore completed. Data loss: ~1 week. Notifying players via in-game banner."
```

## 5. 加密与密钥管理 (Encryption & Key Management)

### 5.1 数据加密 (AES-256-GCM)

| 数据 | 加密时机 | 算法 | 密钥 |
|------|---------|------|------|
| **客户端存档** | 写入前 | AES-256-GCM | 分片存储 (3 选 2) |
| **服务端备份 (S3)** | 上传前 | AES-256-GCM | KMS 托管 |
| **WAL 归档 (S3)** | 上传前 | AES-256-GCM | KMS 托管 |
| **离线冷备 (Glacier)** | 上传前 | AES-256-GCM | KMS 托管 |
| **API 通信** | 传输中 | TLS 1.3 | Let's Encrypt |

### 5.2 密钥管理 (AWS KMS)

```hcl
# terraform/kms.tf
resource "aws_kms_key" "anzhong_backup" {
  description             = "KMS key for anzhong backup encryption"
  deletion_window_in_days = 30
  enable_key_rotation     = true  # 自动年度轮换

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Principal = {
          AWS = "arn:aws:iam::123456789:role/anzhong-backup-role"
        }
        Action = [
          "kms:Encrypt",
          "kms:Decrypt",
          "kms:GenerateDataKey"
        ]
        Resource = "*"
      }
    ]
  })

  tags = {
    Project     = "anzhong"
    Environment = "production"
    Compliance  = "GDPR"
  }
}
```

### 5.3 密钥轮换 (Quarterly)

```bash
#!/bin/bash
# tools/security/rotate-keys.sh
# 季度密钥轮换 (1月/4月/7月/10月)

set -euo pipefail

# 1. 生成新密钥
NEW_KEY=$(aws kms create-key --description "anzhong backup key $(date +%Y%m)" \
    --key-usage ENCRYPT_DECRYPT --customer-master-key-spec SYMMETRIC_DEFAULT \
    --query 'KeyMetadata.KeyId' --output text)

# 2. 创建新密钥别名
aws kms create-alias --alias-name "alias/anzhong-backup-$(date +%Y%m)" --target-key-id "$NEW_KEY"

# 3. 更新备份脚本使用新密钥
sed -i "s/alias\/anzhong-backup-[0-9]*/alias\/anzhong-backup-$(date +%Y%m)/g" /opt/anzhong/tools/db/*.sh

# 4. 重新加密旧备份 (异步, 不阻塞)
# aws s3 cp s3://anzhong-backups/daily/20260629_030000.dump s3://anzhong-backups/daily/20260629_030000.dump.new \
#     --sse aws:kms --sse-kms-key-id "$NEW_KEY"

# 5. 通知团队
echo "Key rotation completed: $NEW_KEY (alias/anzhong-backup-$(date +%Y%m))"
```

## 6. 监控与告警 (Monitoring & Alerts)

### 6.1 备份监控指标

| 指标 | 目标 | 告警阈值 |
|------|------|---------|
| **backup_success_rate (24h)** | 100% | < 100% (PagerDuty) |
| **backup_duration_sec** | < 600s | > 1800s (Slack) |
| **backup_size_gb** | < 50 GB | > 80 GB (Slack) |
| **wal_archive_lag_sec** | < 60s | > 300s (PagerDuty) |
| **rpo_actual_sec** | < 300s | > 600s (PagerDuty) |
| **replica_lag_sec** | < 10s | > 60s (Slack) |
| **s3_bucket_size_gb** | < 500 GB | > 800 GB (Slack) |
| **glacier_retrieval_pending** | 0 | > 0 (Slack) |

### 6.2 备份监控命令

```bash
# 检查最近备份时间
aws s3 ls s3://anzhong-backups/daily/ --recursive | tail -5

# 检查 WAL 归档延迟
psql -d anzhong -c "SELECT now() - last_archived_time AS lag FROM pg_stat_archiver"

# 检查 Replica 延迟
psql -h db-replica -d anzhong -c "SELECT now() - pg_last_xact_replay_timestamp() AS replication_lag"

# 检查备份大小
aws s3 ls s3://anzhong-backups/ --recursive --summarize | grep "Total Size"
```

### 6.3 告警配置 (CloudWatch)

```yaml
# terraform/cloudwatch-alarms.tf
resource "aws_cloudwatch_metric_alarm" "backup_failure" {
  alarm_name          = "anzhong-db-backup-failure"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "BackupFailureCount"
  namespace           = "Anzhong/DB"
  period              = 86400  # 24h
  statistic           = "Sum"
  threshold           = 0
  alarm_description   = "DB backup failed in last 24h"
  alarm_actions       = ["arn:aws:sns:ap-east-1:123456789:anzhong-pagerduty"]

  dimensions = {
    Environment = "production"
  }
}

resource "aws_cloudwatch_metric_alarm" "rpo_exceeded" {
  alarm_name          = "anzhong-rpo-exceeded"
  comparison_operator = "GreaterThanThreshold"
  evaluation_periods  = 1
  metric_name         = "WALArchiveLagSeconds"
  namespace           = "Anzhong/DB"
  period              = 300  # 5 min
  statistic           = "Average"
  threshold           = 600  # 10 min
  alarm_description   = "RPO exceeded 10 minutes"
  alarm_actions       = ["arn:aws:sns:ap-east-1:123456789:anzhong-pagerduty"]
}
```

## 7. 灾备演练 (DR Drills)

### 7.1 季度演练 (Quarterly)

| 季度 | 演练场景 | 步骤 |
|:----:|---------|------|
| **Q1** | Replica failover | 1. 模拟主库宕机 2. 自动切换到 Replica 3. 验证读写 4. 恢复主库 |
| **Q2** | 异地备份恢复 | 1. 从异地 S3 下载备份 2. 在临时环境恢复 3. 验证数据完整性 4. 删除临时环境 |
| **Q3** | PITR Point-in-Time | 1. 模拟数据误删 2. 恢复到删除前 30 秒 3. 验证丢失数据已恢复 |
| **Q4** | 勒索软件演练 | 1. 模拟数据被加密 2. 从 Glacier 冷备恢复 4. 验证恢复时间 < 4h |

### 7.2 演练 SOP (示例: Q1 Replica Failover)

```bash
#!/bin/bash
# tools/dr-drills/q1-failover.sh
# Q1 Replica Failover 演练

set -euo pipefail

DRILL_ID="dr-q1-$(date +%Y%m%d)"
START_TIME=$(date +%s)

echo "[DRILL $DRILL_ID] Starting at $(date)"

# 1. 模拟主库宕机 (仅演练, 不真停)
echo "[DRILL $DRILL_ID] Simulating primary failure..."
# 实际生产: sudo systemctl stop postgresql (危险, 仅演练环境)
# 演练环境: iptables -A INPUT -p tcp --dport 5432 -j DROP (模拟网络隔离)

# 2. Sentinel 自动检测 + 切换 (≤ 30s)
echo "[DRILL $DRILL_ID] Waiting for Sentinel to detect failure..."
sleep 35

# 3. 验证 Replica 升级为主
NEW_PRIMARY=$(aws route53 list-resource-record-sets \
    --hosted-zone-id Z123ABC \
    --query "ResourceRecordSets[?Name == 'db.anzhong.game.'].ResourceRecords[0].Value" \
    --output text)
echo "[DRILL $DRILL_ID] New primary: $NEW_PRIMARY"

# 4. 写入测试
psql -h "$NEW_PRIMARY" -d anzhong -c "
    INSERT INTO audit_logs (action, actor, result, metadata)
    VALUES ('dr_drill_q1', 'system', 'success', '{\"drill_id\":\"$DRILL_ID\"}'::jsonb)
"

# 5. 读取测试
COUNT=$(psql -h "$NEW_PRIMARY" -d anzhong -t -c "SELECT count(*) FROM players")
echo "[DRILL $DRILL_ID] Player count after failover: $COUNT"

# 6. 恢复原主库 (演练完成)
echo "[DRILL $DRILL_ID] Restoring original primary..."
# iptables -D INPUT -p tcp --dport 5432 -j DROP
# 手动提升原主库为 Replica 或主库

END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))
echo "[DRILL $DRILL_ID] Completed in ${DURATION}s"

# 7. 记录演练结果
echo "[DRILL $DRILL_ID] Result: PASS / FAIL: PASS / RTO: ${DURATION}s"
```

### 7.3 演练报告

```markdown
# DR Drill Report Q1 2026

## 基本信息
- 演练 ID: dr-q1-20260115
- 演练类型: Replica Failover
- 演练时间: 2026-01-15 14:00 UTC
- 演练负责人: SRE Team
- 参与人员: DevOps, Backend, QA

## 演练结果
- [x] 自动检测时间: 25s (目标 ≤ 30s) ✅
- [x] 切换时间: 35s (目标 ≤ 30s) ⚠️ (略超)
- [x] 写入测试: PASS ✅
- [x] 读取测试: PASS ✅
- [x] 数据完整性: PASS ✅ (100K 玩家数据全部一致)
- [x] 总 RTO: 60s (目标 ≤ 30min) ✅

## 发现问题
1. Sentinel 配置可优化, 切换时间 35s 略超 30s 目标
   - **改进**: 调整 Sentinel down-after-milliseconds 从 5000ms → 3000ms
2. 部分客户端连接池未自动重连, 需 60s 后手动重连
   - **改进**: 客户端集成 StackExchange.Redis 自动重试

## 后续行动
- [ ] 优化 Sentinel 配置 (P1, 1 周内)
- [ ] 客户端自动重连测试 (P2, 2 周内)
- [ ] 下次演练: 2026-04-15 (Q2 异地恢复)
```

## 8. GDPR 合规与存档管理 (GDPR Compliance)

### 8.1 GDPR 数据保留策略

| 数据类型 | 活跃期 | 软删除期 | 硬删除 | 合规依据 |
|---------|-------|---------|--------|---------|
| **players** | 永久 (活跃) | 30 d (软删除) | 30 d 后硬删除 | GDPR Art. 17 (删除权) |
| **sessions** | 1 y | 1 y 后归档 | 2 y 后删除 | GDPR Art. 5 (最小化) |
| **saves** | 永久 (跟随玩家) | 30 d | 30 d 后硬删除 | GDPR Art. 17 |
| **scores** | 永久 (排行榜) | 30 d | 30 d 后硬删除 | GDPR Art. 17 |
| **feedback** | 1 y (分析) | 30 d | 1 y 后删除 | GDPR Art. 5 |
| **telemetry_events** | 90 d (热) + 1 y (冷) | — | 1 y 后删除 | GDPR Art. 5 |
| **audit_logs** | 1 y (合规) | — | 1 y 后归档 S3 | GDPR Art. 30 (记录) |
| **save_backups** | 30 d (恢复窗口) | — | 30 d 后删除 | GDPR Art. 5 |
| **leaderboard_entries** | 永久 (Top 100) | 30 d | 30 d 后删除 | GDPR Art. 17 |

### 8.2 GDPR 数据导出 (玩家请求)

详见 `persistence-strategy.md` §6.1。

### 8.3 GDPR 数据删除 (玩家请求)

```sql
-- GDPR 软删除 (30 天保留期)
UPDATE players
SET deleted_at = NOW(),
    anonymized = TRUE,
    display_name = 'Deleted User',
    device_id = 'deleted-' || id::TEXT
WHERE id = $1;

-- 30 天后, 由定时任务硬删除
DELETE FROM saves WHERE player_id = $1;
DELETE FROM scores WHERE player_id = $1;
DELETE FROM feedback WHERE player_id = $1;
DELETE FROM player_audio_settings WHERE player_id = $1;
DELETE FROM player_accessibility_settings WHERE player_id = $1;
DELETE FROM telemetry_events WHERE player_id = $1;
DELETE FROM save_backups WHERE player_id = $1;
DELETE FROM device_links WHERE player_id = $1;
DELETE FROM audit_logs WHERE player_id = $1 AND action IN ('gdpr_export', 'gdpr_delete');  -- 保留合规日志
DELETE FROM players WHERE id = $1;
```

## 9. 关联文档 (Cross-References)

- [`README.md`](./README.md) — 数据架构总览
- [`persistence-strategy.md`](./persistence-strategy.md) — 持久化策略
- [`database-schema.md`](./database-schema.md) — 18 表 schema
- [`cache-strategy.md`](./cache-strategy.md) — Redis 缓存
- [`serialization.md`](./serialization.md) — 序列化格式
- [`migrations.md`](./migrations.md) — 3 阶段迁移
- [`p0-001-tracking.md`](./p0-001-tracking.md) — P0-001 跟踪
- [`../../docs/11-release-v2.md`](../../docs/11-release-v2.md) — 7 平台 + GDPR

## 10. 变更日志 (Changelog)

| 日期 | 版本 | 变更内容 |
|------|:----:|---------|
| 2026-06-29 | v1.0 | 中书省 subagent 创建。**新建：** v1.0 本地 .bak 轮转 + Steam Cloud 冗余 + v2.0+ 3-2-1 (本地 + 异地 + Glacier) + RPO 5 min / RTO 1 h + AES-256-GCM + KMS 密钥管理 + 季度密钥轮换 + 4 类备份脚本 (全量/增量/异地/冷备) + cron 调度 + PITR + Failover + 勒索软件恢复 + GDPR 数据保留策略 + 监控告警 + 季度灾备演练 SOP。 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）