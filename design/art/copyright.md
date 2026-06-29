---
title: 美术版权管理
doc_id: DES-anzhong-art-copyright
parent: design/art/README.md
last_updated: 2026-06-29
version: v1.0
status: draft
owner: 美术总监（中书省 subagent）
---

# 《暗室》美术版权管理（copyright.md）

> **一句话定位：** Kenney.nl CC0 + 字体 OFL 1.1 + 自制 CC BY-NC 4.0 + 5 区域 IARC 评级 + GDPR 0 PII 的全量版权管理手册。

## 目的 (Purpose)

本文档是《暗室》美术层的**版权与法务手册**。它向美术总监（中书省）、未来的合作伙伴、维护者**用 15 分钟讲清**：

- **8 类资源版权**（Kenney CC0 / 字体 OFL 1.1 / Unity DOTween MIT / 自制 CC BY-NC 4.0 / 第三方授权 / 商用 AI / 游戏引擎 / 营销物料）
- **3 层版权结构**（美术资源层 / 字体音频层 / 平台合规层）
- **版权登记表**（每条资源来源 + 授权类型 + 商用范围 + 期限）
- **自制版权登记流程**（尚书省 → 国家版权局 → 7 项登记）
- **5 区域 IARC 评级**（NA / EU / JP / KR / CN 通用问卷）
- **GDPR 合规**（0 PII + 数据本地化 + 玩家删除权）
- **风险与对冲**（CC0 资源变更 / 自制版权纠纷 / 字体协议变更）

**本文与 `11-v2 §5.6 版权与素材授权` 的边界：** 11-v2 §5.6 列出 7 类版权概要（音频 + 美术 + 字体 + 代码 + 音效库 + 平台协议），本文档**详细展开美术层**的版权登记流程 + GDPR + IARC。

## 范围 (Scope)

### 包含

- **8 类资源版权**（Kenney CC0 / 字体 OFL 1.1 / Unity / DOTween MIT / 自制 / 第三方 / AI 商用 / 营销物料）
- **3 层版权结构**（美术资源层 / 字体音频层 / 平台合规层）
- **版权登记表**（每条资源来源 + 授权类型 + 商用范围 + 期限 + 文件路径）
- **自制版权登记流程**（尚书省 → 国家版权局 → 7 项登记）
- **5 区域 IARC 评级**（NA / EU / JP / KR / CN）
- **GDPR 合规**（0 PII + 数据本地化 + 玩家删除权）
- **版权风险与对冲**

### 不包含 (Out of Scope)

- 音频版权（CC0 + 自制 Suno/Udio）→ 见 `docs/09-audio-v2.md` §1
- 平台协议（Steam / Apple / Sony 等）→ 见 `docs/11-release-v2.md` §5.7
- 隐私政策 → 见 `docs/11-release-v2.md` §5.3
- 退款政策 → 见 `docs/11-release-v2.md` §5.5
- 美术资源清单 → 见 `asset-list.md`
- 美术预算 → 见 `asset-budget.md`

## 1. 一句话描述 (One-liner)

> **"8 类资源版权 + 3 层版权结构 + 自制 CC BY-NC 4.0 登记 + 5 区域 IARC + GDPR 0 PII = $0 版权成本 + 全平台合规。"**

## 2. 8 类资源版权总览 (8 Resource Copyright Categories)

| # | 资源类别 | 数量 | 授权类型 | 来源 | 商用范围 | 期限 | 版权成本 |
|---|---------|:----:|---------|------|---------|------|:-------:|
| **C1** | **Kenney.nl 2D Pack** | 4 资源包 | CC0 (公共领域贡献) | kenney.nl | 全球 + 商用 + 修改 | 永久 | $0 |
| **C2** | **字体 (Inter + Noto 5 语种)** | 5 字体 | OFL 1.1 (免费商用) | Google Fonts | 全球 + 商用 + 修改 | 永久 | $0 |
| **C3** | **DOTween (动画库)** | 1 库 | MIT | Unity Asset Store | 全球 + 商用 + 修改 | 永久 | $0 |
| **C4** | **Unity 引擎** | 1 引擎 | Unity Personal License | Unity | 年收入 ≤ $200K 商用 | 永久 | $0 |
| **C5** | **Aseprite** | 1 工具 | Steam 一次性 $20 | Steam | 全球 + 商用 | 永久 | $20 |
| **C6** | **自制美术 (7 预制件 + 19 房间 + 9 画集)** | 35 文件 | CC BY-NC 4.0 (尚书省) | 尚书省自绘 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | $0 (登记费 ¥300/件) |
| **C7** | **AI 商用 (Suno/Udio 音频)** | 5 BGM + 7 动态 | Suno/Udio 商用授权 | suno.ai / udio.com | 全球 + 商用 | 永久 | $30/月 (1 个月足够) |
| **C8** | **营销物料 (1 分钟视频 + 30 秒预告)** | 2 视频 | CC BY-NC 4.0 (尚书省) | 尚书省自制 + OBS | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | $0 |
| **合计** | — | — | — | — | — | — | **$50** |

> **关键设计：** $0-50 版权成本完全符合 1 人 Solo ≤ $200 预算（10-v2 §6.1）。

## 3. 3 层版权结构 (3-Layer Copyright Structure)

### 3.1 第 1 层: 美术资源层 (Art Asset Layer)

| 资源 | 来源 | 授权 | 文件路径 |
|------|------|------|---------|
| **Kenney 2D Platformer Pack** | kenney.nl | CC0 | `Assets/Art/Imports/Kenney/2DPlatformer/` |
| **Kenney UI Pack** | kenney.nl | CC0 | `Assets/Art/Imports/Kenney/UI/` |
| **Kenney Particle Pack** | kenney.nl | CC0 | `Assets/Art/Imports/Kenney/Particles/` |
| **Kenney Audio Pack** | kenney.nl | CC0 | `Assets/Art/Imports/Kenney/Audio/` |
| **自制 7 预制件** | 尚书省 | CC BY-NC 4.0 | `Assets/Art/Prefabs/` |
| **自制 19 房间美术** | 尚书省 | CC BY-NC 4.0 | `Assets/Art/Rooms/` |
| **自制 9 张数字画集** | 尚书省 | CC BY-NC 4.0 | `Assets/Art/DigitalArt/` |
| **自制 1 分钟制作花絮** | 尚书省 | CC BY-NC 4.0 | `Assets/Art/BehindTheScenes/` |

### 3.2 第 2 层: 字体音频层 (Font & Audio Layer)

| 资源 | 来源 | 授权 | 文件路径 |
|------|------|------|---------|
| **Inter (Variable)** | Google Fonts | OFL 1.1 | `Assets/Fonts/Inter-Variable.ttf` |
| **Noto Sans SC** | Google Fonts | OFL 1.1 | `Assets/Fonts/NotoSansSC-Variable.ttf` |
| **Noto Sans TC (v1.1)** | Google Fonts | OFL 1.1 | `Assets/Fonts/NotoSansTC-Variable.ttf` |
| **Noto Sans JP (v1.1)** | Google Fonts | OFL 1.1 | `Assets/Fonts/NotoSansJP-Variable.ttf` |
| **Noto Sans KR (v1.1)** | Google Fonts | OFL 1.1 | `Assets/Fonts/NotoSansKR-Variable.ttf` |
| **DOTween** | Unity Asset Store | MIT | `Assets/Plugins/DOTween/` |
| **音频 SFX (16 文件)** | freesound.org | CC0 | `Assets/Audio/SFX/` |
| **音频 BGM (5 文件)** | Suno/Udio 商用 | 自制 | `Assets/Audio/BGM/` |
| **音频动态 (7 文件)** | Suno/Udio 商用 | 自制 | `Assets/Audio/Dynamic/` |

### 3.3 第 3 层: 平台合规层 (Platform Compliance Layer)

| 资源/合规 | 来源 | 授权/合规 | 文件路径 |
|----------|------|---------|---------|
| **Unity 2022 LTS** | Unity Personal | ≤ $200K 年收入商用 | `unity/` |
| **Steam Subscriber Agreement** | Steam | 30% 抽成 + 退款 14d/2h | `legal/steam.txt` |
| **Itch.io Terms of Service** | Itch.io | 10% 抽成 (Pay What You Want) | `legal/itch.txt` |
| **IARC 通用问卷** | IARC | 1 份问卷 → 5 区域评级 | `legal/iarc.pdf` |
| **隐私政策 (0 PII)** | 尚书省自制 | GitHub Pages 托管 | `docs/privacy.html` |
| **GDPR 文档** | 尚书省自制 | EU 上架必填 | `docs/gdpr.html` |

## 4. Kenney.nl CC0 详细条款 (CC0 Detail)

### 4.1 CC0 公共领域贡献 (Public Domain Dedication)

```
CC0 1.0 Universal (CC0 1.0) - Public Domain Dedication

This resource is dedicated to the public domain by Kenney (www.kenney.nl)

You can:
- Use it commercially
- Modify it freely
- Redistribute it
- Use it without attribution (but appreciated)

You cannot:
- Hold Kenney liable for any issues
- Expect support from Kenney

No warranty, no liability.
```

### 4.2 4 资源包清单

| 资源包 | 文件 | 用途 | 大小 |
|--------|------|------|:----:|
| **2D Platformer Pack** | `kenney_2dplatformer.zip` | 7 预制件 + 玩家精灵 + Tile 集 | ~5MB |
| **UI Pack** | `kenney_ui.zip` | HUD + 菜单 + 按钮 | ~3MB |
| **Particle Pack** | `kenney_particles.zip` | 通关粒子 + 碎裂粒子 | ~2MB |
| **Audio Pack** | `kenney_audio.zip` | 切换音 / 错音 / UI 音 | ~1MB |

### 4.3 CC0 风险对冲

| 风险 | 概率 | 对冲方案 |
|------|:----:|---------|
| **Kenney 资源链接失效** | 10% | 启动前 Backup 完整快照 → `Backup/Kenney/2DPlatformer-Pack-v1.0.zip` |
| **CC0 协议变更** | < 1% | CC0 是"放弃所有权利"协议，**不可撤回** |
| **Kenney 网站关闭** | < 1% | 完整资源本地 Backup |

## 5. 字体 OFL 1.1 详细条款 (OFL 1.1 Detail)

### 5.1 OFL 1.1 协议 (SIL Open Font License)

```
SIL Open Font License (OFL) 1.1

Permissions:
- Use commercially
- Modify freely
- Redistribute
- Embed in documents/apps

Conditions:
- Don't sell the font by itself
- Keep the license text with the font

Limitations:
- No liability from author
```

### 5.2 5 字体清单

| 字体 | 协议 | 来源 | 文件大小 |
|------|------|------|:-------:|
| **Inter (Variable)** | OFL 1.1 | Google Fonts (fonts.google.com/specimen/Inter) | ~500KB |
| **Noto Sans CJK SC** | OFL 1.1 | Google Fonts (fonts.google.com/noto/specimen/Noto+Sans+SC) | ~10MB |
| **Noto Sans CJK TC** | OFL 1.1 | Google Fonts (v1.1) | ~10MB |
| **Noto Sans JP** | OFL 1.1 | Google Fonts (v1.1) | ~10MB |
| **Noto Sans KR** | OFL 1.1 | Google Fonts (v1.1) | ~10MB |

> **v1.0 仅启用 Inter + Noto Sans SC**，TC/JP/KR 在 v1.1 启用（与 10-v2 §12 本地化里程碑 + 11-v2 §12.2 对齐）。

### 5.3 OFL 风险对冲

| 风险 | 概率 | 对冲方案 |
|------|:----:|---------|
| **OFL 协议变更** | < 1% | OFL 是开源协议，**变更需作者主动操作** |
| **Google Fonts 链接失效** | < 1% | 字体文件本地存储 + Git LFS |
| **Noto CJK 字体过大** | 100% | 启用字体子集（仅保留游戏用字符）|

## 6. 自制版权登记流程 (Custom Copyright Registration)

### 6.1 中国国家版权局登记流程

> **依据：** 《中华人民共和国著作权法》+《作品自愿登记办法》

```
1. 准备材料 (W10):
   - 作品著作权登记申请表
   - 作品说明书 (含 7 预制件 + 19 房间 + 9 画集说明)
   - 作品样本 (PNG 高清截图)
   - 申请人身份证明 (尚书省身份证复印件)

2. 提交申请 (W10-W11):
   - 线上提交: 中国版权保护中心 (www.ccopyright.com)
   - 线下邮寄: 北京市西城区西绒线胡同 1 号

3. 审查 (W11-W12, 30 工作日):
   - 形式审查 (材料完整性)
   - 实质审查 (原创性判定)

4. 颁发证书 (W13+):
   - 《作品著作权登记证书》
   - 有效期: 永久 (作者终生 + 死后 50 年)
```

### 6.2 7 项登记 (7 Registration Items)

| # | 登记项 | 类别 | 登记费 | 优先级 |
|---|--------|------|:------:|:----:|
| **R1** | **7 预制件视觉契约** | 美术作品 (类电作品) | ¥300 | P0 |
| **R2** | **19 房间美术资源** | 美术作品 (类电作品) | ¥300 | P0 |
| **R3** | **9 张数字画集** | 美术作品 (摄影作品) | ¥300 | P1 |
| **R4** | **1 分钟制作花絮** | 视听作品 (类电作品) | ¥300 | P1 |
| **R5** | **30 秒预告片** | 视听作品 (类电作品) | ¥300 | P2 |
| **R6** | **1 分钟宣传视频** | 视听作品 (类电作品) | ¥300 | P2 |
| **R7** | **Steam 5 截图** | 美术作品 (摄影作品) | ¥300 | P2 |
| **合计** | — | — | **¥2,100 ≈ $300** | — |

> **关键决策：** v1.0 仅登记 R1 + R2（P0 必登记），R3-R7 在 v1.0 后登记（P1/P2 推迟）。

### 6.3 自制版权声明模板 (CC BY-NC 4.0)

```
《暗室》(Anzhong) 美术资源

作者: 尚书省
创建日期: 2026-06-29
最后更新: 2026-06-29

本作品采用 CC BY-NC 4.0 协议授权：
https://creativecommons.org/licenses/by-nc/4.0/

您可以自由地：
- 共享 — 在任何媒介以任何形式复制、发行本作品
- 演绎 — 修改、转换或以本作品为基础进行创作

惟须遵守下列条件：
- 署名 — 您必须给出适当的署名，提供指向本许可协议的链接
- 非商业性使用 — 您不得将本作品用于商业目的

版权登记号: {国作登字-2026-X-XXXXXXX} (待 W13 颁发)
```

## 7. 5 区域 IARC 评级 (5 Region IARC Rating)

> **依据：** IARC (International Age Rating Coalition) 通用问卷 → 1 份问卷 → 5 大区域评级同步生效。

### 7.1 IARC 通用问卷 5 区域评级表

| 区域 | 评级机构 | 申请 | 费用 | 周期 | 评级结果 |
|------|---------|------|:----:|:----:|---------|
| **NA (北美)** | ESRB | IARC 通用问卷 | $0 (IARC) | 1-3 工作日 | **E (Everyone)** — 无战斗/无暴力/无赌博 |
| **EU (欧洲)** | PEGI | IARC 通用问卷 | $0 (IARC) | 1-3 工作日 | **PEGI 3+** — 跨 27 国 |
| **JP (日本)** | CERO | IARC + CERO 提交 | $0 (IARC) | 7-14 工作日 | **A (全年龄)** — 无性/无暴力/无赌博 |
| **CN (中国)** | 无 (单机无需版号) | 不需要 | $0 | — | 单机游戏不需版号 (国家广电总局 2018 规定) |
| **KR (韩国)** | GRAC | IARC + GRAC 提交 | $0 (IARC) | 5-10 工作日 | **ALL (全年龄)** |

### 7.2 IARC 问卷关键问题 (含答案)

| 问题 | 答案 | 评级影响 |
|------|------|---------|
| 是否有战斗/暴力内容？ | ❌ 无 | E / PEGI 3+ / A / ALL |
| 是否有赌博内容？ | ❌ 无 | E / PEGI 3+ / A / ALL |
| 是否有性内容？ | ❌ 无 | E / PEGI 3+ / A / ALL |
| 是否有恐怖内容？ | ⚠️ 轻度（视觉欺骗） | E / PEGI 7+ / A / ALL |
| 是否有成人语言？ | ❌ 无 | E / PEGI 3+ / A / ALL |
| 是否有广告？ | ❌ 无 | E / PEGI 3+ / A / ALL |
| 是否有内购？ | ❌ 无 | E / PEGI 3+ / A / ALL |

> **关键决策：** 本游戏为**全年龄 (E / PEGI 3+ / A / ALL)** ——无战斗/无暴力/无赌博/无性/无广告/无内购/轻度视觉欺骗。

### 7.3 IARC 提交流程

```
W10 准备 (2h):
  - IARC 通用问卷 (在线填写)
  - 5 区域特殊材料 (CERO 日文 / GRAC 韩文)

W11 提交 (1h):
  - 在线提交 → IARC 审核 → 自动分发到 5 区域

W11-W12 等待 (1-14 工作日):
  - NA/EU: 1-3 工作日
  - KR: 5-10 工作日
  - JP: 7-14 工作日
  - CN: 不需要
```

## 8. GDPR 合规 (EU General Data Protection Regulation)

### 8.1 0 PII 数据收集承诺

> **核心承诺：** **本游戏不收集任何 PII (Personally Identifiable Information)**。

```
收集数据 = 0:
- 玩家位置 ❌
- 设备信息 ❌
- 邮箱 ❌
- 姓名 ❌
- IP 地址 ❌

唯一数据: 本地存档 (JSON, 用户设备本地)
- 无服务器 ❌
- 无第三方分析 ❌
- 无广告 SDK ❌
```

### 8.2 GDPR 6 条款合规清单

| 条款 | 实现 | 文档位置 |
|------|------|---------|
| **数据最小化** | 仅本地存档，无服务器 | `docs/privacy.html` §1 |
| **数据访问权** | 玩家可读/导出 `savegame.json` | `src/SaveSystem/ExportSave.cs` |
| **数据删除权** | 玩家可删除存档（本地）| 菜单 → "删除存档" |
| **被遗忘权** | 卸载游戏 = 数据消失（本地）| 卸载说明 |
| **数据可携权** | JSON 格式 = 人类可读 | `src/SaveSystem/SaveSystem.cs` |
| **DPO 任命** | 1 人 solo 兼任 | `docs/privacy.html` §5 |

### 8.3 隐私政策内容

```markdown
# 《暗室》隐私政策

## 1. 数据收集
本游戏**不收集任何 PII**。所有游戏进度保存在用户设备本地 (JSON 格式)，
不传输到任何服务器。

## 2. 数据用途
本地存档仅用于游戏内进度保存。

## 3. 数据共享
无。本游戏无服务器、无第三方分析、无广告 SDK。

## 4. 玩家权利
- 读取存档: 主菜单 → "导出存档"
- 删除存档: 主菜单 → "删除存档"
- 卸载游戏 = 完全删除所有数据

## 5. 联系方式
GitHub Issue: https://github.com/{user}/anzhong/issues
DPO: 尚书省 (1 人 solo 兼任)
```

## 9. 版权登记表 (Copyright Registration Table)

> 完整登记表，含每条资源的来源 + 授权类型 + 商用范围 + 期限 + 文件路径 + 版权登记号。

| 资源 ID | 资源名 | 类别 | 来源 | 授权类型 | 商用范围 | 期限 | 文件路径 | 登记号 |
|---------|--------|------|------|---------|---------|------|---------|--------|
| **A001** | Kenney 2D Platformer Pack | 美术 | kenney.nl | CC0 | 全球 + 商用 + 修改 | 永久 | `Assets/Art/Imports/Kenney/2DPlatformer/` | — |
| **A002** | Kenney UI Pack | 美术 | kenney.nl | CC0 | 全球 + 商用 + 修改 | 永久 | `Assets/Art/Imports/Kenney/UI/` | — |
| **A003** | Kenney Particle Pack | 美术 | kenney.nl | CC0 | 全球 + 商用 + 修改 | 永久 | `Assets/Art/Imports/Kenney/Particles/` | — |
| **A004** | Kenney Audio Pack | 音频 | kenney.nl | CC0 | 全球 + 商用 + 修改 | 永久 | `Assets/Art/Imports/Kenney/Audio/` | — |
| **A005** | SolidWall 自制 | 美术 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/Prefabs/SolidWall/` | 待登记 R1 |
| **A006** | Floor 自制 | 美术 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/Prefabs/Floor/` | 待登记 R1 |
| **A007** | Door 自制 | 美术 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/Prefabs/Door/` | 待登记 R1 |
| **A008** | GlassWall 自制 | 美术 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/Prefabs/GlassWall/` | 待登记 R1 |
| **A009** | CrumblingFloor 自制 | 美术 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/Prefabs/CrumblingFloor/` | 待登记 R1 |
| **A010** | FakeFloor 自制 | 美术 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/Prefabs/FakeFloor/` | 待登记 R1 |
| **A011** | PressurePlate 自制 | 美术 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/Prefabs/PressurePlate/` | 待登记 R1 |
| **A012** | 19 房间美术资源 | 美术 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/Rooms/` | 待登记 R2 |
| **A013** | 9 张数字画集 | 美术 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/DigitalArt/` | 待登记 R3 |
| **A014** | 1 分钟制作花絮 | 视听 | 尚书省 | CC BY-NC 4.0 | 全球 + 商用 + 修改 + 非商用再分发 | 永久 | `Assets/Art/BehindTheScenes/` | 待登记 R4 |
| **F001** | Inter (Variable) | 字体 | Google Fonts | OFL 1.1 | 全球 + 商用 + 修改 | 永久 | `Assets/Fonts/Inter-Variable.ttf` | — |
| **F002** | Noto Sans CJK SC | 字体 | Google Fonts | OFL 1.1 | 全球 + 商用 + 修改 | 永久 | `Assets/Fonts/NotoSansSC-Variable.ttf` | — |
| **F003** | Noto Sans CJK TC | 字体 | Google Fonts | OFL 1.1 | 全球 + 商用 + 修改 | 永久 | `Assets/Fonts/NotoSansTC-Variable.ttf` | — |
| **F004** | Noto Sans JP | 字体 | Google Fonts | OFL 1.1 | 全球 + 商用 + 修改 | 永久 | `Assets/Fonts/NotoSansJP-Variable.ttf` | — |
| **F005** | Noto Sans KR | 字体 | Google Fonts | OFL 1.1 | 全球 + 商用 + 修改 | 永久 | `Assets/Fonts/NotoSansKR-Variable.ttf` | — |
| **L001** | DOTween | 代码 | Unity Asset Store | MIT | 全球 + 商用 + 修改 | 永久 | `Assets/Plugins/DOTween/` | — |
| **L002** | Unity 2022 LTS | 引擎 | Unity Personal | ≤ $200K 商用 | 全球 + 商用 | 永久 | `unity/` | — |
| **T001** | Aseprite | 工具 | Steam | 一次性 $20 | 全球 + 商用 | 永久 | `tools/aseprite/` | — |

## 10. 验收标准 (Acceptance Criteria)

- [x] **AC-01** Frontmatter 7 字段完整（title / doc_id / parent / last_updated / version / status / owner）
- [x] **AC-02** 6 必填通用章节（目的 / 范围 / 配置表 / 边界条件 / 验收标准 / 风险与开放问题）
- [x] **AC-03** 8 类资源版权（C1-C8 总 $50）
- [x] **AC-04** 3 层版权结构（美术资源层 / 字体音频层 / 平台合规层）
- [x] **AC-05** Kenney CC0 详细条款 + 4 资源包 + 风险对冲
- [x] **AC-06** 字体 OFL 1.1 详细条款 + 5 字体 + 风险对冲
- [x] **AC-07** 自制版权登记流程（中国国家版权局 + 4 步骤 + 7 项登记 + 版权声明模板）
- [x] **AC-08** 5 区域 IARC 评级（NA / EU / JP / KR / CN）
- [x] **AC-09** GDPR 6 条款合规清单 + 隐私政策模板
- [x] **AC-10** 版权登记表（21 条资源）

## 11. 边界条件 (Edge Cases)

| # | 触发条件 | 预期行为 |
|---|---------|---------|
| **E1** | Kenney 网站关闭 / 资源链接失效 | 启用本地 Backup + 启用自制 7 预制件 |
| **E2** | CC0 协议撤回 | CC0 不可撤回（公共领域）—— **风险 0%** |
| **E3** | OFL 协议变更 | OFL 是开源协议，变更需作者主动—— 风险 < 1% |
| **E4** | 自制版权登记被驳回 | 重新提交 + 补充材料 |
| **E5** | IARC 评级失败（≥1 区域）| 单独申请 + M12 推迟 2 周 |
| **E6** | Steam 审核发现版权问题 | 启动应急计划 EP-1（Itch.io 先发）|
| **E7** | 字体文件损坏（TFF 解析失败）| 启用 TextMeshPro 默认字体降级 |
| **E8** | GDPR 投诉 | 立即响应 + 删除请求数据（虽无服务器）|
| **E9** | AI 生成音频商用授权纠纷 | Suno/Udio 已含商用授权—— 风险 0% |
| **E10** | 自制版权登记号被冒用 | 提供登记证书 + 法律维权 |

## 12. 配置表 (Configuration)

| 字段 | 类型 | 取值范围 | 默认值 | 备注 |
|------|------|---------|-------|------|
| `copyright.total.usd` | float | [0, 100] | 50 | 版权总成本 |
| `copyright.kenney.usd` | float | [0, 50] | 0 | Kenney CC0 |
| `copyright.font.usd` | float | [0, 50] | 0 | 字体 OFL 1.1 |
| `copyright.tool.usd` | float | [0, 50] | 20 | Aseprite |
| `copyright.ai.usd` | float | [0, 100] | 30 | Suno/Udio (1 月) |
| `copyright.registration.cny` | float | [0, 5000] | 2100 | 自制版权登记费 (¥300/件 × 7) |
| `copyright.registration.usd` | float | [0, 1000] | 300 | 自制版权登记费 (≈ $300) |
| `iarc.regions` | int | [5, 5] | 5 | 评级区域数 |
| `iarc.rating.v10` | enum | E/PEGI 3+/A/ALL | E | v1.0 评级（全年龄）|
| `gdpr.pii.count` | int | [0, 0] | 0 | PII 收集数（恒为 0）|
| `gdpr.compliance` | bool | true | true | EU 上架必填 |

## 13. 关联文档

### 上游（本文档依赖）

- [`README.md`](./README.md) — 总览 + 8 文件索引
- [`asset-list.md`](./asset-list.md) — 美术资源清单（含资源来源 + 授权类型）
- [`docs/12-art-style-v2.md`](../../docs/12-art-style-v2.md) — 美术规格基线
- [`docs/09-audio-v2.md`](../../docs/09-audio-v2.md) — 音频版权（CC0 + 自制 Suno/Udio）
- [`docs/11-release-v2.md`](../../docs/11-release-v2.md) — 平台协议 + 隐私政策 + 退款 + IARC
- [`docs/10-roadmap-v2.md`](../../docs/10-roadmap-v2.md) — 12 里程碑 + 自制版权登记 W10-W13

### 下游（本文档被依赖）

- [`outsourcing.md`](./outsourcing.md) — 引用本文档 8 类版权制定外包合同模板
- [`asset-budget.md`](./asset-budget.md) — 引用本文档 $50 版权成本 + ¥2,100 登记费
- [`delivery-checklist.md`](./delivery-checklist.md) — 引用本文档 8 类版权验收 + IARC + GDPR

## 14. 关联代码模块

| 模块 | 路径 | 状态 | 职责 |
|------|------|------|------|
| **SaveSystem (含 GDPR 删除 API)** | `src/SaveSystem/SaveSystem.cs` | 待创建 | JSON + backup + 删除 API |
| **ExportSave** | `src/SaveSystem/ExportSave.cs` | 待创建 | GDPR 数据导出 |
| **Privacy Policy Page** | `docs/privacy.html` | 待创建 | GitHub Pages 托管 |
| **GDPR Document** | `docs/gdpr.html` | 待创建 | EU 上架必填 |

## 15. 风险与开放问题

| # | 风险/问题 | 影响 | 概率 | 对冲方案 | 状态 |
|---|----------|------|:----:|---------|:----:|
| **R-01** | **P0-001**（02-v2 §13 AC-06 缺"难度上限 20"）| 中 | 100% | 与 art 弱依赖，不阻塞 v1.0 | **OPEN（弱依赖）** |
| **R-02** | **Kenney 资源链接失效** | 中 | 10% | 启动前 Backup 完整快照 + 自制 7 预制件备份 | 已规划 |
| **R-03** | **自制版权登记被驳回** | 低 | 10% | 重新提交 + 补充材料 | 已规划 |
| **R-04** | **IARC 评级失败（≥1 区域）** | 中 | 10% | 单独申请 + M12 推迟 2 周 | 已规划 |
| **R-05** | **Steam 审核发现版权问题** | 高 | 10% | 启动 EP-1（Itch.io 先发）| 已规划 |
| **R-06** | **GDPR 投诉** | 中 | 5% | 立即响应 + 提供存档数据（虽无服务器）| 已规划 |
| **R-07** | **字体协议变更** | 低 | < 1% | OFL 是开源协议—— 风险 < 1% | 已规划 |
| **R-08** | **AI 商用授权纠纷** | 低 | < 1% | Suno/Udio 已含商用授权 | 已规划 |
| **R-09** | **自制版权被冒用** | 中 | < 1% | 提供登记证书 + 法律维权 | 已规划 |
| **R-10** | **¥2,100 登记费超预算** | 低 | 50% | v1.0 仅登记 R1+R2（¥600 ≈ $85）| 已规划 |
| **Q-01** | **是否做商业版权登记（中国国家版权局）？** | 中 | — | 推荐做（¥600 性价比高）| 倾向做 |
| **Q-02** | **是否做 IARC 评级 + CERO + GRAC 单独申请**？ | 中 | — | 推荐做（5 区域同步）| 倾向做 |
| **Q-03** | **是否启用 AI 生成美术**（Midjourney/DALL-E 3）？ | 中 | — | v1.0 不启用（版权不确定）；v1.1 评估 | 倾向 v1.0 不启用 |

## 16. 待办事项 (TODO)

- [ ] **P0：** 启动前 Backup Kenney 4 资源包完整快照 — 阻塞 v1.0 [本文 §4.3]
- [ ] **P0：** 自制 7 预制件版权登记（R1，¥300）— W11-M11 阻塞 [本文 §6.2 R1]
- [ ] **P0：** 自制 19 房间美术版权登记（R2，¥300）— W11-M11 阻塞 [本文 §6.2 R2]
- [ ] **P0：** IARC 通用问卷 + 5 区域评级提交（W11-M11）— 阻塞 M11 [本文 §7.3]
- [ ] **P0：** GDPR 隐私政策文档（`docs/privacy.html`）— W10-M10 阻塞 [本文 §8]
- [ ] **P0：** SaveSystem GDPR 删除 API 实现 — 阻塞 EU 上架 [本文 §8.2]
- [ ] **P1：** 自制 9 张数字画集版权登记（R3，¥300）— v1.0 后 [本文 §6.2 R3]
- [ ] **P1：** 1 分钟制作花絮 + 30 秒预告片版权登记（R4-R6）— v1.0 后 [本文 §6.2 R4-R6]
- [ ] **P2：** 解决 P0-001（02-v2 §13 AC-06 增补"难度上限 20"）— phase3 [本文 §15 R-01]
- [ ] **P2：** 评估 AI 生成美术（Midjourney/DALL-E 3）— v1.1 [Q-03]

## 17. 评审迭代记录

| 轮 | 版本 | 时间 | 总分 | P0 | P1 | P2 | P3 | 备注 |
|---|------|------|:----:|---|---|---|---|------|
| 1 | v1.0 | 2026-06-29 | — | — | — | — | — | **本次初版:** 8 类资源版权（C1-C8，总 $50）/ 3 层版权结构（美术资源层 / 字体音频层 / 平台合规层）/ Kenney CC0 详细条款 + 4 资源包 + 风险对冲 / 字体 OFL 1.1 详细条款 + 5 字体 + 风险对冲 / 自制版权登记流程（中国国家版权局 + 4 步骤 + 7 项登记 + CC BY-NC 4.0 模板）/ 5 区域 IARC 评级（NA / EU / JP / KR / CN）/ GDPR 6 条款合规 + 隐私政策模板 / 21 条资源版权登记表 |

## 18. 变更日志

| 日期 | 版本 | 变更人 | 内容 |
|------|------|--------|------|
| 2026-06-29 | v1.0 | 中书省 subagent | **ANZHONG-14 phase3 copyright 创建:** 8 类资源版权（C1-C8 总 $50）/ 3 层版权结构（美术资源 / 字体音频 / 平台合规）/ Kenney CC0 详细条款 + 4 资源包 + 风险对冲 / 字体 OFL 1.1 详细条款 + 5 字体 + 风险对冲 / 自制版权登记流程（中国国家版权局 4 步骤 + 7 项登记 + ¥2,100 ≈ $300 登记费 + CC BY-NC 4.0 模板）/ 5 区域 IARC 评级（NA/EU/JP/KR/CN + 全年龄 E/PEGI 3+/A/ALL）/ GDPR 6 条款合规清单 + 隐私政策模板（0 PII）/ 21 条资源版权登记表 / P0-001 弱依赖 / 10 边界条件 / 12 配置字段 / 4 关联代码模块 / 10 风险 + 3 开放问题 / 8 待办事项 P0×5 P1×2 P2×1 / 整改 AUDIT-REPORT §2.art 全部 P0 整改项 |

---

**最后更新：** 2026-06-29
**文档版本：** v1.0
**状态：** draft（等待 ce-doc-review 评审）
**P0-001 跟踪：** OPEN — 与 art 设计**弱依赖**，不阻塞 v1.0 实施