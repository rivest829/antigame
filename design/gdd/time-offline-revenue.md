# 时间管理/离线收益系统 (Time & Offline Revenue)

> **Status**: Designed (pending review)
> **Author**: [user + agents]
> **Last Updated**: 2026-04-23
> **Implements Pillar**: 成长是可见的, 节奏是轻松的

## Overview

时间管理/离线收益系统是《地下排练室》的**放置游戏心脏**，负责追踪玩家离线期间的时间流逝，并计算这段时间内角色的状态变化与成长收益。它是连接"玩家设置计划 → 离开游戏 → 回来收获"这一核心循环的桥梁——没有它，放置游戏就失去了"放置"的意义。

**系统边界**：
- **包含**：时间戳记录、离线时长计算、离线状态变化（心情衰减、精力恢复）、离线成长收益（技能、熟练度）、时间异常检测
- **不包含**：排练计划的设置界面（属于放置排练系统）、角色状态的实时更新（属于角色属性系统）、收益的视觉呈现（属于 UI 系统）、服务器时间同步（MVP 不实现）

**玩家如何感知**：
- 离线 3 小时后回来，看到"欢迎回来！乐队排练了 3 小时" → 成长感
- 看到精力从 30 恢复到 90 → "他们休息得不错"
- 看到心情从 60 降到 59 → "我不在的时候他们有点闷"
- 离线太久看到"收益已达上限" → "我应该更常回来看看"

**为什么必须存在**：
没有离线收益系统，放置游戏就变成了手动点击游戏——玩家必须一直在线才能看到成长。离线收益让玩家可以"设置计划后安心离开"，几小时后回来有明确的收获感。这是"节奏是轻松的"支柱的技术基石，也是移动端碎片化时间场景的核心适配。

## Player Fantasy

玩家应该感受到：**"即使我不在，他们也在努力。而当我回来时，我知道他们会给我惊喜。"**

你设置好排练计划，关掉手机去睡觉。第二天早上在地铁上打开游戏，你看到：
- 林野的吉他技能从 42 涨到了 45
- 《锈》的熟练度从 35% 涨到了 47%
- 大伟的精力从耗尽恢复到了 80%

你感受到的不是"系统自动给我的奖励"，而是**"这群年轻人趁你不在的时候真的在练"**。就像你知道你养的植物在你出差时会自己光合作用——它不需要你，但它因为你的安排而成长。

**核心情感时刻**：
你离线了一整天（超过 6 小时收益上限），回来时看到"收益已满"的提示。你有点小失落——"我错过了一部分成长"——但更多的是期待："下次我应该在 6 小时内回来一次。"这种**温柔的催促**是放置游戏留存的魔法。

**参考体验**：
- 《赛马娘》的离线训练——回来看到角色自己训练的成果，有陪伴感
- 《动物森友会》的跨天机制——即使你不玩，世界也在运转（但我们的离线收益更即时、更可量化）

**这不是"挂机刷怪"的幻想**——这是"我的选择在他们身上持续生效"的幻想。

## Detailed Design

### Core Rules

**1. 时间追踪基础**

系统使用**设备本地 Unix 时间戳**（秒级精度）追踪时间：

| 字段 | 说明 |
|------|------|
| `last_leave_timestamp` | 玩家上次离开游戏时的时间戳（秒） |
| `current_timestamp` | 玩家当前打开游戏时的时间戳（秒） |
| `raw_offline_seconds` | `current_timestamp - last_leave_timestamp` |
| `effective_offline_hours` | `min(offline_revenue_cap, max(0, raw_offline_seconds / 3600))` |

**关键约束**：
- 所有离线计算基于 `effective_offline_hours`，而非 `raw_offline_hours`
- `effective_offline_hours` 精确到小数点后 2 位（即分钟级精度）
- 离线时长 < 1 分钟时，视为"未离线"，跳过离线计算（防止频繁切换前后台导致的性能开销）

**2. 离线收益上限**

| 上限类型 | 值 | 说明 |
|----------|-----|------|
| `offline_revenue_cap` | 6 小时 | 成长收益（技能、熟练度）的计算上限 |
| `offline_state_cap` | 12 小时 | 状态变化（心情、精力）的计算上限 |

**为何区分两个上限**：
- 成长收益上限更严格（6 小时），防止离线太久一次性获得巨量收益破坏平衡
- 状态变化上限更宽松（12 小时），允许角色在较长时间离线后有明显的状态恢复/衰减
- 超过上限后，系统不再计算新的变化，角色进入"冻结"状态

**3. 离线状态变化**

离线期间，所有角色的**心情**和**精力**同时变化：

**心情衰减**：
- 公式：`new_mood = max(0, current_mood - decay_rate × effective_state_hours)`
- `decay_rate` = 0.208/小时（来自角色属性系统）
- 离线期间心情**持续衰减**，不受是否排练影响（无人陪伴的乐队成员会变闷）

**精力变化**（逐小时模拟）：
- 精力变化采用**小时粒度模拟**，而非简单公式
- 每个小时，角色要么在**排练**（消耗精力），要么在**休息**（恢复精力）
- 模拟算法见下方"离线精力循环算法"

**4. 离线成长收益**

离线期间的成长收益基于玩家**当前设置的排练计划**：

**收益类型**：
| 收益类型 | 计算来源 | 说明 |
|----------|----------|------|
| 技能成长 | 曲目数据库 `skill_growth_per_hour` × 排练效率 | 每小时增长的乐器技能 |
| 熟练度成长 | 曲目数据库 `proficiency_growth_per_hour` × 曲线修正 | 每小时增长的曲目熟练度 |

**排练效率修正**：
- 离线期间的排练效率使用**离线前最后一次计算的效率值**
- 不实时重新计算效率（避免离线期间角色状态变化导致效率波动的复杂性）
- 但心情阈值效果（低落/亢奋）在离线前已锁定，离线期间保持不变

**5. 离线精力循环算法**

这是系统的核心算法，采用**小时粒度模拟**：

```
function simulate_offline(character, offline_hours, rehearsal_plan):
    rehearsal_hours = 0
    
    for hour in 1..floor(offline_hours):
        if character.energy > rehearsal_plan.energy_cost_per_hour:
            # 排练状态
            character.energy -= rehearsal_plan.energy_cost_per_hour
            rehearsal_hours += 1
            
            # 应用成长收益
            for instrument in rehearsal_plan.instruments:
                growth = rehearsal_plan.skill_growth_per_hour[instrument]
                efficiency = rehearsal_plan.rehearsal_efficiency
                character.skills[instrument] += growth × efficiency
            
            proficiency_growth = rehearsal_plan.proficiency_growth_per_hour
            character.proficiency += proficiency_growth × efficiency
        else:
            # 休息状态
            character.energy = min(100, character.energy + recovery_rate)
    
    # 处理剩余小数小时（如 5.5 小时中的 0.5）
    remaining_fraction = offline_hours - floor(offline_hours)
    if remaining_fraction > 0 and character.energy > rehearsal_plan.energy_cost_per_hour × remaining_fraction:
        # 剩余时间足够排练
        character.energy -= rehearsal_plan.energy_cost_per_hour × remaining_fraction
        rehearsal_hours += remaining_fraction
        # 按比例应用成长收益...
    else:
        # 剩余时间休息
        character.energy = min(100, character.energy + recovery_rate × remaining_fraction)
    
    return rehearsal_hours
```

**算法说明**：
- 每个小时检查角色精力是否足够排练（`energy > energy_cost`）
- 精力足够 → 排练：消耗精力，获得成长
- 精力不足 → 休息：恢复精力，不获得成长
- 精力恢复满后，如果离线时间还有剩余，**可以继续排练**（实现多轮排练→休息→排练循环）

**6. 时间异常检测（MVP 简化版）**

| 异常类型 | 检测条件 | 处理方式 |
|----------|----------|----------|
| 时间回拨 | `current_timestamp < last_leave_timestamp` | 标记异常，使用最小离线时长（0.1 小时）计算，记录警告日志 |
| 超长离线 | `raw_offline_hours > 168`（7 天） | 标记异常，正常按上限计算，但 UI 显示"你离开太久了"提示 |
| 频繁切换 | 1 分钟内多次进入游戏 | 跳过离线计算，视为未离线 |

**MVP 策略**：只检测、只记录警告，**不阻止收益**。防作弊优化列为 Post-MVP。

**7. 设备后台/生命周期处理**

| 场景 | 处理方式 |
|------|----------|
| 游戏进入后台 | 保存 `last_leave_timestamp` + 当前完整游戏状态 |
| 游戏回到前台 | 读取时间戳，计算离线收益，应用所有变化 |
| 设备关机/重启 | 时间戳保存在本地存储，关机不影响 |
| 游戏被卸载后重装 | 无云端存档时，视为新游戏（无离线收益） |
| iOS/Android 后台限制 | 不依赖后台任务，纯基于时间差计算 |

### States and Transitions

系统存在四种离散状态：

| 状态 | 触发条件 | 系统行为 |
|------|----------|----------|
| **在线 (Online)** | 玩家正在游戏 | 实时更新角色状态，不累积离线时长 |
| **离线 (Offline)** | 游戏进入后台或关闭 | 保存时间戳，开始累积离线时长 |
| **同步中 (Syncing)** | 游戏回到前台 | 计算离线时长，执行离线模拟算法 |
| **就绪 (Ready)** | 离线计算完成 | 显示离线收益摘要，等待玩家确认 |

**状态转换图**：

```
   游戏进入后台              游戏回到前台
  ┌─────────────┐          ┌─────────────┐
  │             │          │             │
  │    在线     │─────────►│    离线     │─────────► 同步中
  │             │          │             │  计算完成
  └─────────────┘          └─────────────┘        ┌─────────────┐
                                                  │             │
                                                  │    就绪     │
                                                  │             │
                                                  └─────────────┘
                                                       │
                                                       │ 玩家点击确认
                                                       ▼
                                                  ┌─────────────┐
                                                  │    在线     │
                                                  └─────────────┘
```

### Interactions with Other Systems

**输入（本系统接收的数据）**：

| 来源系统 | 输入数据 | 作用 |
|----------|----------|------|
| 角色属性系统 | 角色当前状态（技能、心情、精力） | 离线模拟的初始状态 |
| 曲目数据库 | 曲目排练收益参数 | 计算离线成长收益 |
| 放置排练系统 | 当前排练计划（曲目、队员、重点） | 确定离线期间的排练内容 |

**输出（本系统提供的数据）**：

| 目标系统 | 输出数据 | 作用 |
|----------|----------|------|
| 角色属性系统 | 更新后的角色状态（技能、心情、精力） | 应用离线变化 |
| 曲目数据库 | 更新后的曲目熟练度 | 应用离线成长 |
| 放置排练系统 | 离线期间的成长摘要 | 显示收益结果 |
| 通知系统 | 离线时长、收益达到上限标记 | 触发"该回来了"通知 |
| UI 系统 | 离线收益摘要数据 | 显示欢迎回来弹窗 |

## Formulas

### 1. 有效离线时长公式 (Effective Offline Hours)

The effective offline hours formula is defined as:

`effective_offline_hours = min(offline_revenue_cap, max(0, (current_timestamp - last_leave_timestamp) / 3600))`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| current_timestamp | now | int | ≥0 | 当前 Unix 时间戳（秒） |
| last_leave_timestamp | last | int | ≥0 | 上次离开时的 Unix 时间戳（秒） |
| offline_revenue_cap | cap | float | >0 | 离线收益上限（小时），默认 6 |

**Output Range:** 0 到 6 小时
**Example:** 上次离开时间戳 1713800000，当前时间戳 1713815000（差 15000 秒 ≈ 4.17 小时）→ `effective = min(6, 15000/3600) = 4.17` 小时

### 2. 离线心情衰减公式 (Offline Mood Decay)

The offline mood decay formula is defined as:

`new_mood = max(0, current_mood - decay_rate × effective_state_hours)`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| current_mood | mood | int | 0–100 | 离线前的心情值 |
| decay_rate | rate | float | 0.208 | 每小时心情衰减率（来自角色属性系统） |
| effective_state_hours | hours | float | 0–12 | 有效状态计算时长（上限 12 小时） |

**Output Range:** 0 到 100
**Example:** 当前心情 60，离线 6 小时 → `new_mood = max(0, 60 - 0.208 × 6) = max(0, 60 - 1.25) = 59`

### 3. 离线精力恢复公式 (Offline Energy Recovery — Rest State)

The offline energy recovery formula during rest is defined as:

`new_energy = min(100, current_energy + recovery_rate × rest_hours)`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| current_energy | energy | int | 0–100 | 休息前的精力值 |
| recovery_rate | rate | float | 20.0 | 每小时精力恢复量（来自角色属性系统） |
| rest_hours | hours | float | ≥0 | 休息小时数 |

**Output Range:** 0 到 100
**Example:** 精力 10，休息 3 小时 → `new_energy = min(100, 10 + 20 × 3) = min(100, 70) = 70`

### 4. 离线技能成长公式 (Offline Skill Growth)

The offline skill growth formula is defined as:

`skill_growth_total = Σ(skill_growth_per_hour × rehearsal_efficiency)`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| skill_growth_per_hour | growth | float | 1–10 | 曲目定义的基础技能成长/小时 |
| rehearsal_efficiency | efficiency | float | 0.0–1.56 | 离线前锁定的排练效率（来自角色属性系统） |
| rehearsal_hours | hours | float | 0–6 | 实际排练小时数（由精力循环算法确定） |

**Output Range:** 0 到 ~60（6 小时 × 最高效率 1.56 × 最高成长 8/hr）
**Example:** 曲目成长 4/hr，效率 1.2，排练 3 小时 → `skill_growth = 4 × 1.2 × 3 = 14.4`（向下取整 = 14）

### 5. 离线熟练度成长公式 (Offline Proficiency Growth)

The offline proficiency growth formula is defined as:

`proficiency_growth_total = Σ(base_proficiency_rate × curve_multiplier × rehearsal_efficiency)`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| base_proficiency_rate | base | float | 1.0–10.0 | 曲目定义的基础熟练度成长率（%/小时） |
| curve_multiplier | curve | float | 0.5–1.3 | 熟练度曲线修正（来自曲目数据库） |
| rehearsal_efficiency | efficiency | float | 0.0–1.56 | 离线前锁定的排练效率 |
| rehearsal_hours | hours | float | 0–6 | 实际排练小时数 |

**Output Range:** 0% 到 ~100%
**Example:** 基础率 4%/hr，曲线修正 0.9（递减曲线），效率 1.0，排练 2 小时 → `proficiency_growth = 4 × 0.9 × 1.0 × 2 = 7.2%`

### 6. 精力消耗公式 (Energy Cost During Rehearsal)

The energy cost formula during offline rehearsal is defined as:

`energy_consumed = energy_cost_per_hour × rehearsal_hours`

**Variables:**
| Variable | Symbol | Type | Range | Description |
|----------|--------|------|-------|-------------|
| energy_cost_per_hour | cost | int | 5–20 | 每小时排练消耗的精力 |
| rehearsal_hours | hours | float | 0–6 | 实际排练小时数 |

**Note:** `energy_cost_per_hour` 由放置排练系统定义。MVP 固定为 10/小时。

## Edge Cases

- **If 设备时间回拨（current_timestamp < last_leave_timestamp）**：标记时间异常，使用最小离线时长 0.1 小时计算收益，心情和精力变化也按 0.1 小时计算。UI 显示"检测到时间异常，收益已保护性计算"。
- **If 离线时长 < 1 分钟**：视为未离线，跳过所有离线计算，直接进入在线状态。防止频繁切换前后台导致的性能开销。
- **If 离线时长 > 12 小时（状态上限）**：心情和精力的变化只计算前 12 小时，超过部分角色进入"冻结"状态。成长收益仍只计算前 6 小时。
- **If 所有角色的精力在离线开始时都为 0**：无人可以排练，离线成长收益为 0。所有角色进入休息状态，精力恢复、心情衰减正常计算。
- **If 排练计划中的曲目在离线期间被删除或修改**：使用离线前保存的排练计划快照计算，不受后续修改影响。
- **If 游戏在离线计算过程中被强制关闭（如设备没电）**：已计算的部分收益已保存到本地状态，未计算的部分在下次打开游戏时继续计算。
- **If 角色在离线期间达到精力 100% 且还有剩余离线时间**：角色继续排练（精力从 100 开始消耗），实现多轮"排练→休息→排练"循环。
- **If 玩家跨时区旅行（设备时区改变）**：Unix 时间戳不受时区影响，离线计算不受影响。时区改变只影响时间显示格式。
- **If 夏令时切换导致时间跳变（1 小时）**：Unix 时间戳连续，不受影响。系统只关心时间差，不关心本地时间显示。
- **If 玩家手动修改设备时间后进入游戏**：由时间异常检测处理。如果 current_timestamp < last_leave_timestamp → 回拨检测。如果离线时长异常大 → 超长离线检测。
- **If 没有设置排练计划就离线**：离线成长收益为 0。角色只进行状态变化（心情衰减、精力恢复）。

## Dependencies

**上游依赖**：
| 系统 | 依赖数据 | 必要性 |
|------|----------|--------|
| 角色属性系统 | 角色当前状态、decay_rate、recovery_rate | 硬依赖 — 离线模拟的初始状态和参数 |
| 曲目数据库 | 曲目成长参数 | 硬依赖 — 离线成长收益计算 |
| 放置排练系统 | 排练计划设置 | 硬依赖 — 确定离线期间排练内容 |

**下游被依赖**：
| 系统 | 依赖数据 | 数据流向 |
|------|----------|----------|
| 角色属性系统 | 更新后的状态 | Time → 角色状态更新 |
| 曲目数据库 | 更新后的熟练度 | Time → 曲目熟练度更新 |
| 放置排练系统 | 离线收益摘要 | Time → 收益展示 |
| 通知系统 | 离线时长、收益上限标记 | Time → 通知触发 |
| UI 系统 | 离线摘要数据 | Time → 欢迎弹窗 |

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 过低效果 | 过高效果 | 关联系统 |
|--------|--------|----------|----------|----------|----------|
| `offline_revenue_cap` | 6 小时 | 4–12 | 玩家被迫过于频繁登录，产生压力感 | 离线收益过高，破坏放置节奏平衡 | 放置排练 |
| `offline_state_cap` | 12 小时 | 8–24 | 状态变化限制过严，长离线后角色状态不真实 | 状态变化过大，长离线后心情永远为 0 | 角色属性 |
| `energy_cost_per_hour` | 10 | 5–20 | 角色几乎不消耗精力，可无限排练 | 角色快速耗尽精力，离线收益大幅减少 | 放置排练 |
| `min_offline_minutes` | 1 | 0–5 | 频繁切换前后台触发离线计算，性能开销 | 短暂离开不计为离线，玩家感到"少了点什么" | UI |
| `time_manipulation_tolerance_hours` | 168 | 72–720 | 过于敏感，正常长时间离线被误判 | 过于宽松，时间篡改检测失效 | 安全 |
| `efficiency_lock_mode` | 锁定离线前效率 | — | — | — | 放置排练 |

## Visual/Audio Requirements

- **离线收益结算弹窗**：像一张被雨水打湿后重新展开的演出传单，上面手写记录着离线期间的所有变化。数字像被墨水刚刚写上去一样，有轻微的晕染效果。
- **时间流逝动画**：弹窗顶部有一个简单的时钟动画，指针从上次离开的时间转动到当前时间，持续 1-1.5 秒。指针是生锈的金属质感。
- **收益数字跳动**：技能成长、熟练度成长的数字以"打字机"效果逐个出现，配合轻微的纸张翻动音效。
- **精力恢复可视化**：精力条从离线前的值以渐变动画恢复到新值，颜色从灰→黄→绿渐变。
- **"收益已满"提示**：当离线时长超过 6 小时，显示一枚生锈的徽章，上面写着"6h MAX"，暗示玩家"你回来晚了，但他们已经尽力了"。
- **音效**：欢迎回来的吉他扫弦（轻快但略带慵懒，像刚睡醒的乐队），收益数字跳动时的轻微鼓点。

## UI Requirements

- **欢迎回来弹窗（Welcome Back Modal）**：游戏回到前台时的全屏/半屏弹窗，显示：
  - 离线时长（"你离开了 X 小时 Y 分钟"）
  - 如果超过 6 小时，显示"收益已达上限"提示
  - 每个角色的变化摘要：技能成长（+X）、精力变化（+Y/-Y）、心情变化（-Z）
  - 曲目熟练度变化：每首排练曲目的熟练度涨幅
  - 总排练小时数（"乐队总共排练了 X 小时"）
  - 一个"确认"按钮，点击后进入主界面

- **收益详情展开**：点击角色头像或曲目名称可展开详细计算过程（供好奇的玩家查看）。

- **离线状态指示器**：主界面角落显示一个小时钟图标，hover 时显示"距离收益上限还有 X 小时"。

- **通知触发条件**：当离线时长接近或达到 6 小时上限时（如 5 小时），触发系统通知："乐队已经排练了 5 小时，快回来看看成果！"

## Acceptance Criteria

- **GIVEN** 上次离开时间戳 1713800000，当前时间戳 1713815000（差 4.17 小时），**WHEN** 计算有效离线时长，**THEN** 结果为 4.17 小时
- **GIVEN** 上次离开时间戳 1713800000，当前时间戳 1713900000（差 27.8 小时），**WHEN** 计算有效离线时长，**THEN** 成长收益按 6 小时上限计算，状态变化按 12 小时上限计算
- **GIVEN** 角色精力 50，每小时消耗 10，离线 6 小时，**WHEN** 执行精力循环模拟，**THEN** 前 5 小时排练（精力从 50→0），第 6 小时休息（精力从 0→20），总排练 5 小时
- **GIVEN** 角色精力 100，每小时消耗 10，离线 6 小时，**WHEN** 执行精力循环模拟，**THEN** 前 10 小时排练到耗尽（但受离线上限 6 小时限制），实际排练 6 小时
- **GIVEN** 当前时间戳 < 上次离开时间戳，**WHEN** 检测时间异常，**THEN** 标记异常，按 0.1 小时计算收益
- **GIVEN** 离线时长 < 1 分钟，**WHEN** 计算离线收益，**THEN** 跳过离线计算，视为未离线
- **GIVEN** 心情 60，离线 6 小时，decay_rate=0.208，**WHEN** 计算新心情，**THEN** 新心情 = max(0, 60 - 0.208 × 6) = 59
- **GIVEN** 没有设置排练计划，**WHEN** 离线 3 小时后回来，**THEN** 成长收益为 0，但状态变化（心情衰减、精力恢复）正常计算
- **GIVEN** 所有角色精力为 0，**WHEN** 离线 4 小时，**THEN** 成长收益为 0，所有角色休息恢复精力
- **GIVEN** 排练计划包含《锈》（成长 4/hr），效率 1.2，实际排练 3 小时，**WHEN** 计算技能成长，**THEN** 吉他技能增长 4 × 1.2 × 3 = 14.4（向下取整 14）

## Open Questions

- **Q1**: 是否需要服务器时间同步？MVP 用本地时间简单，但存在被篡改风险。服务器时间需要后端支持，增加复杂度。
- **Q2**: 离线期间是否可能触发叙事事件？例如"你不在时，A 和 B 吵了一架"。这会增加沉浸感，但需要叙事事件系统支持离线触发。
- **Q3**: 精力循环算法的复杂度是否过高？逐小时模拟最精确，但可以用简单公式近似（可排练小时数 = 初始精力 / 消耗 + 恢复后额外小时数）。MVP 后需要性能评估。
- **Q4**: 跨天登录的特殊处理：如果玩家每天晚上睡 8 小时，连续 7 天每天登录一次——这种使用模式是否在 6 小时上限下体验良好？
- **Q5**: 是否需要在后台限制严格的设备（如 iOS）上测试离线计算的准确性？某些设备可能在后台被强制终止，导致 last_leave_timestamp 未及时保存。
