# 事件模板系统 (Event Template System)

> **Status**: Designed (pending review)
> **Author**: [user + agents]
> **Last Updated**: 2026-04-23
> **Implements Pillar**: 人物是活的

## Overview

事件模板系统是《地下排练室》的**叙事内容生成基础设施**。它将叙事事件抽象为"模板 + 变量池 + 条件集 + 分支结构"四层结构，使少量手工编写的模板通过变量组合和条件判定，生成大量有差异、有灵魂的事件变体。它是叙事事件系统（#16）的数据根基——叙事事件系统负责运行时的事件调度和展示，而事件模板系统负责定义"有哪些事件模板、它们何时触发、包含什么变量、有哪些分支"。

**系统边界**：
- **包含**：事件模板的定义结构、变量池、触发条件语法、分支结构、事件类型分类
- **不包含**：事件的实时调度逻辑（属于叙事事件系统）、事件的UI展示（属于叙事事件UI）、玩家对事件的响应处理（属于叙事事件系统）、事件触发的即时数值效果（属于放置排练/关系矩阵等系统）

**为什么必须存在**：
没有事件模板系统，叙事事件系统就不知道"有哪些事件可以触发"。放置游戏的核心留存魔法是"每次回来都有新鲜事"——MVP需要5-10条核心事件，完整版需要30-50条。如果全部手工编写，内容量无法支撑长期游玩。模板化生成是唯一能兼顾"内容量"和"个性感"的方案。

---

## Player Fantasy

玩家不会直接接触"模板系统"——他们接触到的是模板系统**产生的结果**：

> 你离线了3小时，回来打开游戏。林野和大伟正在吵架——林野觉得大伟鼓点太随意，大伟觉得林野太较真。你选择"让林野冷静一下"，两天后林野主动找大伟道歉，两人的关系反而更深了。

**核心情感**：**"这个世界在我离开的时候也在运转"**。队员不是只有在你在的时候才活着——他们有自己的互动、冲突、成长。

**这不是"看故事"的幻想**——这是"参与一个正在发生的故事"的幻想。模板系统必须确保每一次"回来"都有值得期待的新变化，而且这些变化不是机械的重复。

---

## Detailed Design

### Core Rules

#### 1. 四层结构

每个事件模板由四个层级构成：

| 层级 | 作用 | 示例 |
|------|------|------|
| **模板 (Template)** | 定义事件的骨架结构——标题模板、正文模板、事件类型、默认参数 | `title: "{actor}和{target}的争执"` |
| **变量池 (Variable Pool)** | 填充模板占位符的具体内容——角色名、情感词、场景描述 | `{actor}=林野, {target}=大伟, {emotion}=不耐烦` |
| **条件集 (Condition Set)** | 控制事件何时可以被触发——状态检查、关系检查、进度检查 | `actor.mood < 30 AND relationship(actor, target) < 50` |
| **分支结构 (Branch Structure)** | 定义玩家可做的选择及每个选择的后果 | 选项A：支持林野 → 后果X；选项B：支持大伟 → 后果Y |

#### 2. 事件类型分类

所有事件模板必须属于以下类型之一：

| 类型标识 | 类型名称 | 触发场景 | MVP 数量 |
|----------|----------|----------|----------|
| `RELATIONSHIP` | 关系事件 | 角色间互动（排练、演出、日常） | 3 |
| `REHEARSAL` | 排练事件 | 排练过程中或排练后 | 2 |
| `PERFORMANCE` | 演出事件 | 演出前、中、后 | 1 |
| `DAILY` | 日常事件 | 任意时间，低门槛随机触发 | 2 |
| `MILESTONE` | 里程碑事件 | 达到特定进度节点 | 2 |

**类型约束**：每个类型的事件在视觉表现、触发频率、文本长度上有不同的默认值（见 Tuning Knobs）。

#### 3. 模板数据结构

```yaml
event_template:
  template_id: "string"           # 唯一标识，如 "rehearsal_argument"
  template_type: "enum"           # RELATIONSHIP / REHEARSAL / PERFORMANCE / DAILY / MILESTONE
  title_template: "string"        # 标题模板，含变量占位符
  body_template: "string"         # 正文模板，含变量占位符
  
  # 触发条件
  condition_set:
    conditions: []                # 条件列表（见条件语法）
    logic: "AND" | "OR"           # 条件组合逻辑
    base_chance: 0.0-1.0          # 基础触发概率（0-1）
    cooldown_hours: int           # 触发后冷却时间（小时）
    max_occurrences: int | null   # 最大触发次数（null=无限）
    priority: int                 # 优先级（高优先事件优先触发）
  
  # 变量定义
  variables: []                   # 本模板使用的变量列表
  
  # 分支定义
  branches: []                    # 玩家选择分支（见分支结构）
  
  # 元数据
  tags: []                        # 标签（用于分类和检索）
  estimated_text_length: "short" | "medium" | "long"  # 预估文本长度
```

#### 4. 变量池定义

变量分为**全局变量**（所有模板共享）和**模板私有变量**（仅本模板使用）。

**全局角色变量**（由角色属性系统提供）：

| 变量名 | 类型 | 说明 | 示例值 |
|--------|------|------|--------|
| `{actor}` | 角色引用 | 事件发起角色 | 林野、大伟 |
| `{target}` | 角色引用 | 事件目标角色 | 林野、大伟 |
| `{other_member}` | 角色引用 | 其他在场角色 | 林野、大伟 |
| `{actor_skill}` | 字符串 | actor 的主乐器 | 吉他、鼓 |
| `{target_skill}` | 字符串 | target 的主乐器 | 吉他、鼓 |

**全局情感变量**（由模板系统维护词库）：

| 变量名 | 类型 | 说明 | 示例值 |
|--------|------|------|--------|
| `{emotion_positive}` | 字符串 | 正面情感词 | 兴奋、感动、欣慰、激动 |
| `{emotion_negative}` | 字符串 | 负面情感词 | 烦躁、沮丧、焦虑、愤怒 |
| `{emotion_neutral}` | 字符串 | 中性情感词 | 困惑、疲惫、平静、专注 |

**全局场景变量**：

| 变量名 | 类型 | 说明 | 示例值 |
|--------|------|------|--------|
| `{location}` | 字符串 | 场景地点 | 排练室、校园舞台、走廊 |
| `{time_of_day}` | 字符串 | 时间段 | 深夜、午后、清晨 |
| `{weather}` | 字符串 | 天气 | 下雨、闷热、晴朗 |

**数值变量**（由其他系统提供）：

| 变量名 | 类型 | 说明 | 来源 |
|--------|------|------|------|
| `{skill_value}` | 整数 | 某项技能值 | 角色属性系统 |
| `{proficiency_pct}` | 整数 | 曲目熟练度百分比 | 曲目数据库 |
| `{relationship_score}` | 整数 | 两个角色的关系值 | 关系矩阵系统 |
| `{hours_offline}` | 整数 | 离线小时数 | 时间管理系统 |

#### 5. 条件语法

条件采用**字段-运算符-值**的三元组结构，支持组合逻辑。

**基本语法**：
```
[entity].[attribute] [operator] [value]
```

**运算符**：

| 运算符 | 说明 | 示例 |
|--------|------|------|
| `==` | 等于 | `actor.type == "技术型"` |
| `!=` | 不等于 | `target.type != "社交型"` |
| `>` `>=` `<` `<=` | 比较 | `actor.mood < 30` |
| `IN` | 包含于 | `actor.trait IN ["倔强", "敏感"]` |
| `BETWEEN` | 区间 | `relationship_score BETWEEN 30 AND 60` |

**特殊条件类型**：

| 条件类型 | 语法 | 说明 |
|----------|------|------|
| 随机条件 | `RANDOM_CHANCE(p)` | 掷骰，概率p（0-1） |
| 关系条件 | `RELATIONSHIP(a, b) [op] [value]` | 两个角色的关系值 |
| 配合技条件 | `COMBO_ACTIVE(combo_id)` | 某配合技是否激活 |
| 进度条件 | `CONTENT_UNLOCKED(content_id)` | 某内容是否已解锁 |
| 时间条件 | `HOURS_OFFLINE [op] [value]` | 离线时长 |
| 排练条件 | `REHEARSAL_HOURS(song_id) [op] [value]` | 某曲目累计排练时长 |
| 冷却条件 | `TEMPLATE_COOLDOWN(template_id)` | 某模板是否在冷却中 |
| 上限条件 | `OCCURRENCE_COUNT(template_id) [op] [value]` | 某模板已触发次数 |

**组合逻辑**：

```yaml
condition_set:
  logic: "AND"
  conditions:
    - "actor.mood < 30"
    - "target.mood < 30"
    - "RELATIONSHIP(actor, target) BETWEEN 20 AND 60"
    - "RANDOM_CHANCE(0.3)"
```

以上表示："当actor和target心情都低于30、两人关系值在20-60之间、且随机掷骰30%概率命中时，事件可能触发。"

**组合逻辑支持嵌套**：
```yaml
condition_set:
  logic: "OR"
  conditions:
    - logic: "AND"
      conditions:
        - "actor.mood < 30"
        - "target.energy < 20"
    - logic: "AND"
      conditions:
        - "actor.energy < 20"
        - "target.mood < 30"
```

以上表示："（actor心情低 AND target精力耗尽）OR（actor精力耗尽 AND target心情低）"。

#### 6. 分支结构

每个分支包含：

```yaml
branch:
  branch_id: "string"
  option_text_template: "string"     # 选项文本模板（玩家看到的按钮文字）
  option_condition: "string" | null  # 选项显示条件（null=始终显示）
  consequences: []                    # 后果列表
  next_event: "string" | null        # 连锁触发的下一个事件模板ID
  weight: int                        # 分支权重（用于随机默认选择或AI建议）
```

**后果类型**：

| 后果类型 | 参数 | 说明 |
|----------|------|------|
| `STAT_CHANGE` | `entity`, `stat`, `delta` | 改变角色状态值（如心情+10） |
| `SKILL_CHANGE` | `entity`, `skill`, `delta` | 改变角色技能值 |
| `RELATIONSHIP_CHANGE` | `entity_a`, `entity_b`, `delta` | 改变两个角色的关系值 |
| `CHEMISTRY_CHANGE` | `combo_id`, `delta` | 改变化学反应值 |
| `UNLOCK_CONTENT` | `content_id` | 解锁某内容 |
| `TRIGGER_EVENT` | `template_id`, `delay_hours` | 延迟触发另一个事件 |
| `SET_FLAG` | `flag_name`, `value` | 设置全局/角色标志 |
| `CUSTOM` | `script_ref` | 自定义脚本（预留扩展） |

**无分支事件**：`branches` 为空数组时，事件为纯叙事展示，玩家只需点击"继续"。

#### 7. 事件实例生命周期

模板被触发后，生成一个**事件实例**：

```
模板 (Template)
  ↓ 条件满足 + 变量填充
事件实例 (Event Instance)
  ↓ 玩家查看
活跃状态 (Active)
  ↓ 玩家做出选择
已处理 (Resolved)
  ↓ 后果应用
已归档 (Archived)
```

| 状态 | 说明 | 持久化 |
|------|------|--------|
| **草稿 (Draft)** | 模板生成的原始实例，等待调度 | 否（运行时） |
| **活跃 (Active)** | 已推送给玩家，等待响应 | 是 |
| **已处理 (Resolved)** | 玩家已做出选择，后果已应用 | 是 |
| **已归档 (Archived)** | 超过 max_pileup_events 限制的旧事件 | 是（仅保留摘要） |

#### 8. MVP 事件模板清单

| 模板ID | 类型 | 标题模板 | 触发条件概要 | 分支数 |
|--------|------|----------|-------------|--------|
| `rel_argument_rehearsal` | RELATIONSHIP | `{actor}和{target}的争执` | 两人同时排练、心情<30、关系20-60 | 3 |
| `rel_late_night_talk` | RELATIONSHIP | `深夜的{location}` | 离线>2小时、某角色心情<40 | 2 |
| `rel_encouragement` | RELATIONSHIP | `{actor}的鼓励` | 某角色心情<30、另一角色外向 | 2 |
| `reh_skill_plateau` | REHEARSAL | `{actor}的瓶颈` | 某技能连续4小时无成长 | 2 |
| `reh_breakthrough` | REHEARSAL | `意外的突破` | 配合技首次激活 | 1（纯叙事） |
| `perf_prestage_nerves` | PERFORMANCE | `上台前的{emotion_negative}` | 演出前、角色心情<50 | 2 |
| `daily_leak` | DAILY | `排练室的麻烦` | 随机20%、无冷却 | 2 |
| `daily_random_encounter` | DAILY | `走廊里的偶遇` | 随机15%、校园舞台已解锁 | 2 |
| `ms_first_show_invite` | MILESTONE | `来自校园的邀请` | venue_campus_stage解锁条件满足 | 1（纯叙事） |
| `ms_combo_discovery` | MILESTONE | `新的化学反应！` | 任意配合技首次触发 | 1（纯叙事） |

### States and Transitions

事件模板本身没有运行时状态（它是静态数据）。运行时状态属于**事件实例**：

| 实例状态 | 进入条件 | 离开条件 | 玩家可见 |
|----------|----------|----------|----------|
| **草稿** | 模板条件满足，变量填充完成 | 被叙事事件系统调度推送 | 不可见 |
| **活跃（未读）** | 推送给玩家 | 玩家打开事件界面 | 主界面红点/通知 |
| **活跃（已读未处理）** | 玩家打开事件 | 玩家做出选择 | 事件详情界面 |
| **已处理** | 玩家做出选择 | 后果应用完成 | 事件历史记录 |
| **已归档** | 超过堆积上限或玩家手动归档 | 无（终态） | 归档日志 |

```
模板条件满足
    │
    ▼
┌─────────┐    调度推送     ┌─────────────┐    玩家打开     ┌─────────────┐
│  草稿    │ ──────────────► │ 活跃(未读)   │ ────────────► │ 活跃(已读)   │
└─────────┘                 └─────────────┘               └─────────────┘
                                                                 │
                                                                 │ 玩家选择
                                                                 ▼
                                                          ┌─────────────┐
                                                          │   已处理     │
                                                          └─────────────┘
                                                                 │
                                                                 │ 后果应用
                                                                 ▼
                                                          ┌─────────────┐
                                                          │   已归档     │
                                                          └─────────────┘
```

### Interactions with Other Systems

**输入（本系统需要的数据，由叙事事件系统运行时提供）**：

| 来源系统 | 输入数据 | 作用 |
|----------|----------|------|
| 角色属性系统 | 角色技能值、状态值（心情/精力/士气） | 条件判定 |
| 关系矩阵系统 | 角色间关系值 | 关系条件判定 |
| 化学反应系统 | 配合技激活状态 | 配合技条件判定 |
| 时间管理/离线收益 | 离线时长、当前时间 | 时间条件判定 |
| 进度解锁系统 | 内容解锁状态 | 进度条件判定 |
| 曲目数据库 | 曲目熟练度 | 排练条件判定 |
| 叙事事件系统 | 模板触发请求、实例状态变更 | 运行时交互 |

**输出（本系统提供的数据）**：

| 目标系统 | 输出数据 | 作用 |
|----------|----------|------|
| 叙事事件系统 | 完整的事件模板定义 | 运行时实例化事件 |
| 存档/加载系统 | 事件实例状态、历史记录 | 持久化 |
| UI 系统 | 事件标题、正文、选项文本、后果预览 | 展示 |

**重要边界**：事件模板系统**不直接读写**任何运行时数据。所有运行时数据交互都通过叙事事件系统进行——叙事事件系统读取其他系统的数据，填充到模板变量中，执行条件判定，然后将结果返回给模板系统生成实例。

---

## Formulas

### 1. 触发概率公式 (Trigger Probability)

```
trigger_probability = base_chance × condition_multiplier × type_modifier × random_roll
```

**Where:**

`condition_multiplier = 1.0 + Σ(condition_bonus_i)`

每个满足的条件提供额外加值：

| 条件满足 | bonus |
|----------|-------|
| 状态条件（心情/精力阈值） | +0.1 |
| 关系条件 | +0.15 |
| 进度条件 | +0.2 |
| 时间条件 | +0.05 |
| 配合技条件 | +0.25 |

`type_modifier`：

| 事件类型 | modifier |
|----------|----------|
| DAILY | 1.0 |
| REHEARSAL | 0.8 |
| RELATIONSHIP | 0.7 |
| PERFORMANCE | 0.6 |
| MILESTONE | 1.0（固定触发，无随机） |

`random_roll`：0.0 到 1.0 之间的均匀随机数。

**判定规则**：`random_roll ≤ trigger_probability` 时触发。

**Example**: `rel_argument_rehearsal` 模板
- base_chance = 0.3
- 满足条件：状态条件×2（心情<30）、关系条件×1 → condition_multiplier = 1.0 + 0.1 + 0.1 + 0.15 = 1.35
- type_modifier = 0.7（RELATIONSHIP）
- trigger_probability = 0.3 × 1.35 × 0.7 = 0.2835（28.35%）
- random_roll = 0.25 → 触发成功

**上限**：trigger_probability 硬封顶至 `max_trigger_chance`（默认 0.8）。MILESTONE 类型不受上限限制。

### 2. 变量填充算法 (Variable Resolution)

```
FOR EACH placeholder IN template:
    IF placeholder is ROLE_VARIABLE:
        candidate_pool = filter(characters, placeholder.constraints)
        IF candidate_pool is empty:
            USE fallback_value
        ELSE:
            value = weighted_random_select(candidate_pool, weights)
    ELSE IF placeholder is EMOTION_VARIABLE:
        value = random_select(emotion_pool[placeholder.sentiment])
    ELSE IF placeholder is NUMERIC_VARIABLE:
        value = fetch_from_system(placeholder.source_system)
    REPLACE placeholder WITH value
```

**角色变量权重**：

```
weight(character) = base_weight × mood_bias × relationship_bias
```

- `base_weight`：角色基础权重（所有角色相等 = 1.0）
- `mood_bias`：心情越低越可能成为负面事件主角（`mood_bias = 1.0 + (50 - mood) / 100`，上限 1.5）
- `relationship_bias`：关系值差异大的角色对更可能产生冲突事件（`relationship_bias = 1.0 + |rel - 50| / 100`，上限 1.5）

**变量缺失处理**：如果某个变量无法解析（如引用了不存在的角色），使用 `fallback_value`：
- 角色变量 → 默认选第一个可用角色
- 情感变量 → 默认选中性词
- 数值变量 → 默认填 0 或最小值
- 场景变量 → 默认填"排练室"

### 3. 分支选择后果公式 (Consequence Application)

后果应用遵循**立即应用 + 延迟应用**两种模式：

**立即后果**（`delay = 0`）：
```
new_value = clamp(old_value + delta, min_cap, max_cap)
```

**延迟后果**（`delay > 0`）：
```
scheduled_time = current_time + delay_hours
at scheduled_time: apply_consequence()
```

**连锁事件触发**（`TRIGGER_EVENT` 后果）：
```
IF next_event IS NOT NULL:
    schedule_event(next_event, delay_hours)
    next_event.inherit_context = current_event.context  # 继承上下文变量
```

### 4. 事件堆积处理公式 (Event Pileup)

当活跃事件数超过 `max_active_events` 时：

```
IF active_event_count >= max_active_events:
    # 优先保留高优先级和 MILESTONE 类型事件
    sorted_events = sort(active_events, by: [priority DESC, type == MILESTONE, created_time ASC])
    events_to_archive = sorted_events[max_active_events:]
    FOR event IN events_to_archive:
        event.status = ARCHIVED
        event.summary = generate_summary(event)  # 生成一句话摘要
```

**归档摘要生成**：
```
summary = "{event.title} — {selected_branch.option_text}"
```

---

## Edge Cases

- **If 条件组合导致 trigger_probability > 1.0**：硬封顶至 `max_trigger_chance`（默认 0.8）。MILESTONE 类型不受限制，固定触发。
- **If 变量池中无可用候选**（如 `{actor}` 约束条件过滤后无角色满足）：使用 fallback 机制——角色变量默认选心情最低的角色，情感变量默认选中性情，数值变量默认填 0。并在日志中记录警告。
- **If 事件触发瞬间角色状态发生变化**（如玩家刚好在查看角色面板时离线收益结算降低了心情）：使用**触发时刻快照**——事件实例化时锁定所有参与变量的当前值，后续状态变化不影响本次事件。
- **If 同一模板的冷却期内条件再次满足**：事件不触发，记录为"冷却中跳过"。冷却期结束后重新参与概率判定。
- **If 多个模板同时满足触发条件**：按优先级排序，同优先级按类型 modifier 排序（MILESTONE > DAILY > REHEARSAL > RELATIONSHIP > PERFORMANCE），一次只触发一个事件（防止事件轰炸）。
- **If 事件堆积超过 max_active_events 且所有事件都是 MILESTONE**：保留最新的 MILESTONE，将最早的 MILESTONE 归档。MILESTONE 事件不应被丢弃——它们驱动主线进度。
- **If 分支选项的 option_condition 在玩家查看时变为 false**（如某选项要求角色精力>20，但玩家在事件堆积期间离线导致精力降到15）：选项显示为灰色不可选，hover 提示"{角色}太累了，无法选择这个选项"。
- **If 连锁事件的 next_event 模板不存在或已禁用**：连锁触发失败，记录错误日志，但当前事件正常完成。
- **If 模板中的 content_id 引用了不存在的内容**：`UNLOCK_CONTENT` 后果静默失败（不崩溃），记录警告。
- **If 事件实例化后但在玩家处理前游戏关闭**：通过存档/加载系统保存事件实例状态——活跃事件在下次启动时继续显示。

---

## Dependencies

**上游依赖**：
| 系统 | 依赖数据 | 必要性 |
|------|----------|--------|
| 角色属性系统 | 角色状态值、技能值、性格特征 | 软依赖 — 条件判定和变量填充需要，但模板系统本身不直接读取 |
| 关系矩阵系统 | 角色间关系值 | 软依赖 — 关系条件需要 |
| 曲目数据库 | 曲目熟练度 | 软依赖 — 排练条件需要 |
| 时间管理/离线收益 | 离线时长 | 软依赖 — 时间条件需要 |
| 进度解锁系统 | 内容解锁状态 | 软依赖 — 进度条件需要 |
| 化学反应系统 | 配合技状态 | 软依赖 — 配合技条件需要 |

> 注：所有上游依赖都是**软依赖**。事件模板系统作为纯数据定义层，其模板定义可以独立于运行时系统存在。叙事事件系统负责在运行时桥接这些依赖。

**下游被依赖**：
| 系统 | 依赖数据 | 数据流向 |
|------|----------|----------|
| 叙事事件系统 | 所有模板定义、变量池、条件语法、分支结构 | Event Template → 叙事事件运行时 |
| 存档/加载系统 | 事件实例状态格式 | Event Template 定义状态结构 |
| UI 系统 | 事件展示的数据结构 | Event Template 定义文本模板和分支选项格式 |
| 内容数据库 | 剧情章节的 linked_events 引用模板 ID | Content DB → Event Template（反向引用） |

---

## Tuning Knobs

| 参数名 | 当前值 | 安全范围 | 过低效果 | 过高效果 | 关联 |
|--------|--------|----------|----------|----------|------|
| `max_trigger_chance` | 0.8 | 0.5–1.0 | 事件触发太少，世界感觉"死寂" | 事件触发过于频繁，玩家疲劳 | 叙事事件 |
| `base_chance_range` | 0.1–0.5 | 0.05–0.8 | 模板几乎不触发 | 模板频繁触发，缺乏稀缺感 | 叙事事件 |
| `condition_bonus_per_condition` | +0.1 | 0.05–0.3 | 多条件模板与单条件模板概率差异小 | 多条件模板概率过高 | 概率公式 |
| `cooldown_hours_default` | 24 | 4–72 | 同一事件频繁重复 | 事件种类轮换过慢 | 叙事事件 |
| `max_active_events` | 5 | 3–10 | 玩家可能错过重要事件 | 事件堆积过多，处理压力大 | UI/叙事 |
| `max_occurrences_default` | null | 1–10 | 无限制 | 事件过早耗尽 | 内容量 |
| `daily_type_modifier` | 1.0 | 0.5–1.5 | 日常事件触发太少 | 日常事件淹没重要事件 | 事件类型 |
| `relationship_type_modifier` | 0.7 | 0.3–1.0 | 关系事件太少，角色缺乏互动 | 关系事件过多 | 事件类型 |
| `milestone_forced_trigger` | true | true/false | 里程碑事件可能错过 | 无影响 | 主线进度 |
| `variable_fallback_enabled` | true | true/false | 变量缺失导致事件无法生成 | 无影响 | 稳定性 |
| `event_pileup_priority_weight` | 10 | 5–20 | 低优先级事件容易被积压 | 高优先级事件垄断队列 | 队列管理 |
| `archive_summary_max_length` | 50 | 20–100 | 归档摘要太短，信息不足 | 归档摘要太长，占用空间 | 存档 |

---

## Visual/Audio Requirements

事件模板系统本身**不定义视觉资产**，但定义了事件类型的视觉分类：

| 事件类型 | 推荐主色调 | 图标风格 | 文本长度 |
|----------|-----------|----------|----------|
| RELATIONSHIP | 铁锈红 (#B7410E) | 两个重叠的人形剪影 | medium (80-150字) |
| REHEARSAL | 脏黄 (#D4A843) | 乐器轮廓（吉他+鼓槌） | medium (80-150字) |
| PERFORMANCE | 荧光粉 (#FF6B9D) | 舞台聚光灯 | short (50-80字) |
| DAILY | 水泥灰 (#8C8C8C) | 日历/时钟图标 | short (50-80字) |
| MILESTONE | 金/白 (#F0E6D2) | 星形/奖章 | long (150-250字) |

**文本长度约束**：模板定义中的 `estimated_text_length` 字段指导叙事内容作者控制文本量。放置游戏的玩家平均每次会话阅读时间有限——MILESTONE 事件可以长一些（里程碑时刻值得投入），DAILY 事件必须短（日常琐事不应占用太多时间）。

---

## UI Requirements

事件模板系统**不直接定义 UI**，但定义了 UI 展示所需的数据结构：

**事件卡片数据结构**（UI 系统消费）：
```
EventCardData:
  - event_id: string
  - title: string (已填充变量)
  - body: string (已填充变量)
  - type: enum (事件类型，决定颜色和图标)
  - priority: int (决定排序)
  - is_read: bool
  - branches: BranchOption[]
    - option_text: string
    - is_available: bool
    - unavailable_reason: string | null
  - timestamp: datetime
```

**通知数据结构**：
```
EventNotification:
  - event_id: string
  - title_preview: string (标题前20字)
  - type: enum
  - badge_count: int
```

---

## Acceptance Criteria

- **GIVEN** `rel_argument_rehearsal` 模板定义了条件 `actor.mood < 30 AND target.mood < 30`，**WHEN** 林野心情25、大伟心情20，**THEN** 状态条件判定为 `true`
- **GIVEN** `rel_argument_rehearsal` 的 base_chance = 0.3，满足2个状态条件和1个关系条件，**WHEN** 计算 trigger_probability，**THEN** 结果为 `0.3 × 1.35 × 0.7 = 0.2835`
- **GIVEN** trigger_probability = 0.2835，**WHEN** random_roll = 0.25，**THEN** 事件触发（0.25 ≤ 0.2835）
- **GIVEN** trigger_probability = 0.2835，**WHEN** random_roll = 0.5，**THEN** 事件不触发（0.5 > 0.2835）
- **GIVEN** 模板中 `{actor}` 变量约束为"技术型"角色，**WHEN** 林野（技术型）在场，**THEN** `{actor}` 填充为"林野"
- **GIVEN** 模板中 `{target}` 变量约束过滤后无角色满足，**WHEN** 填充变量，**THEN** 使用 fallback（选择心情最低的角色）并记录警告
- **GIVEN** `daily_leak` 模板冷却期为24小时，**WHEN** 模板触发后12小时内条件再次满足，**THEN** 事件不触发，状态为"冷却中"
- **GIVEN** 活跃事件数为5（已达 max_active_events），**WHEN** 新事件触发，**THEN** 最早的一个非MILESTONE事件被归档
- **GIVEN** `reh_breakthrough` 模板为纯叙事事件（branches 为空），**WHEN** 玩家查看事件，**THEN** 只显示"继续"按钮，无选择分支
- **GIVEN** 某分支选项的 option_condition 为 `actor.energy > 20`，**WHEN** actor 精力 = 15，**THEN** 选项显示为灰色不可选，提示"{actor}太累了"
- **GIVEN** 事件实例状态为"活跃（未读）"，**WHEN** 游戏关闭并重新启动，**THEN** 事件状态保持"活跃（未读）"
- **GIVEN** MILESTONE 类型事件满足触发条件，**WHEN** 概率判定，**THEN** 固定触发（random_roll 不参与判定）

---

## Open Questions

- **Q1**: 模板变量池是否需要支持多语言（中文/英文/日文）？MVP 只支持中文，但数据结构是否需要预留 i18n 字段？
- **Q2**: 条件语法是否需要支持更复杂的表达式（如数学运算 `actor.skill_guitar + actor.skill_drum > 50`）？当前设计仅支持简单比较和逻辑组合。
- **Q3**: 模板系统是否需要支持"事件链"的完整定义（不只是 next_event，而是完整的多事件序列）？MVP 的连锁机制可能已足够。
- **Q4**: 玩家对同一事件的多次体验是否需要差异化（如第二次看到类似争执时，文本稍有不同）？这需要模板支持"次数感知变量"。
- **Q5**: 事件模板的版本管理如何处理？如果后续更新修改了某模板的后果，已处理的历史事件是否回溯更新？
