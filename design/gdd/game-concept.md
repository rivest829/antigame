# Game Concept: 地下排练室 (Underground Rehearsal Room)

*Created: 2026-04-22*
*Status: Draft*

---

## Elevator Pitch

> 这是一款放置养成游戏，你在破旧的大学排练室里从零开始组建乐队。设置排练计划、观察队员成长、处理他们之间的化学反应——然后离开，等几个小时回来收获成果。最终从穷苦大学生走向摇滚巨星。

---

## Core Identity

| Aspect | Detail |
| ---- | ---- |
| **Genre** | 放置养成 / 角色扮演 / 叙事驱动 |
| **Platform** | 移动端 (iOS / Android) |
| **Target Audience** | 休闲到中核玩家，喜欢成长感和叙事 |
| **Player Count** | 单人 |
| **Session Length** | 5-15 分钟（收成果+处理事件+设置计划） |
| **Monetization** | 暂无（优先验证核心循环） |
| **Estimated Scope** | 小（MVP 3-4 周，完整版 8-12 周，单人开发） |
| **Comparable Titles** | 赛马娘（角色养成+放置）、疯狂骑士团（挂机成长）、偶像梦幻祭（角色关系+叙事） |

---

## Core Fantasy

你见证并参与了一群年轻人的成长——从在漏水的地下室里跑调，到在万人舞台上让全场起立。你的选择塑造了他们的命运，而他们也在塑造你。

这不是关于"成为最强"的幻想，而是关于"看着一群人从陌生到默契，从迷茫到坚定"的陪伴感。

---

## Unique Hook

像《赛马娘》的角色养成，**但**你的队员是会吵架、会恋爱、会深夜谈心的真实年轻人。乐队解散是真实可能发生的事情——而你的每一个选择都在推动或阻止这个结局。

乐队成员之间的"化学反应"不只是数值加成，它会触发真实的叙事事件：A 和 B 一起排练久了可能会产生摩擦，也可能擦出火花。这些关系变化会反过来影响排练效率和演出表现。

---

## Player Experience Analysis (MDA Framework)

### Target Aesthetics (What the player FEELS)

| Aesthetic | Priority | How We Deliver It |
| ---- | ---- | ---- |
| **Fantasy** (make-believe, role-playing) | 1 | 角色身份认同——"我是这支乐队的灵魂人物"；从穷学生到摇滚巨星的身份转变 |
| **Narrative** (drama, story arc) | 2 | 队员个人故事线、乐队整体发展弧线、化学反应触发的动态事件 |
| **Submission** (relaxation, comfort zone) | 3 | 放置机制、零操作压力、随时来随时走、没有毁灭性失败 |
| **Discovery** (exploration, secrets) | 4 | 解锁新曲目、新场地、新化学反应事件、隐藏结局 |
| **Expression** (self-expression, creativity) | 5 | 自定义排练策略、选择乐队发展方向、处理队员关系的方式 |
| **Challenge** (obstacle course, mastery) | 6 | 演出难度递增，但始终可恢复；策略性曲目编排而非操作技巧 |
| **Fellowship** (social connection) | 7 | 队员间关系系统、与外部角色（经纪人、粉丝）的互动 |
| **Sensation** (sensory pleasure) | N/A | 非核心目标；简约美术风格，不追求视听震撼 |

### Key Dynamics (Emergent player behaviors)

- 玩家会实验不同的队员组合来发现最优化学反应
- 玩家会关心特定队员的故事发展，产生情感投入
- 玩家会制定长期排练计划来为目标演出做准备
- 玩家会在离线期间期待回来看到的新变化

### Core Mechanics (Systems we build)

1. **放置排练系统** — 设置排练计划（曲目、队员、重点）→ 时间推进 → 收获技能成长和化学反应变化
2. **化学反应系统** — 队员组合产生不同化学反应（显性数值+隐性叙事事件），影响排练效率和演出表现
3. **叙事事件系统** — 基于化学反应和进度触发的动态事件，玩家选择处理方式
4. **演出判定系统** — 基于排练成果的演出表现评估，影响乐队声誉和解锁内容
5. **进度解锁系统** — 技能、曲目、场地、剧情章节的分层解锁

---

## Player Motivation Profile

### Primary Psychological Needs Served

| Need | How This Game Satisfies It | Strength |
| ---- | ---- | ---- |
| **Autonomy** (freedom, meaningful choice) | 选择练什么、和谁练、接什么演出、怎么处理队员冲突；每个选择都有倾向性后果 | 核心 |
| **Competence** (mastery, skill growth) | 看着数值一天天上涨；攻克越来越大的演出；发现最优排练策略 | 核心 |
| **Relatedness** (connection, belonging) | 和队员建立情感连接；经历他们的故事；感受团队从陌生到默契 | 核心 |

### Player Type Appeal (Bartle Taxonomy)

- [x] **Achievers** (goal completion, collection, progression) — How: 技能数值成长、曲目收集、场地解锁、声誉提升
- [x] **Explorers** (discovery, understanding systems, finding secrets) — How: 发现新的化学反应组合、解锁隐藏剧情、探索不同决策路径
- [x] **Socializers** (relationships, cooperation, community) — How: 队员关系系统、叙事事件中的情感互动、对角色产生情感连接
- [ ] **Killers/Competitors** (domination, PvP, leaderboards) — How: 不适用；本游戏明确排除竞技元素

### Flow State Design

- **Onboarding curve**: 前10分钟——创建乐队、认识第一个队员、完成第一次排练设置、回来收成果。重点传达"设置→等待→收获"的核心节奏。
- **Difficulty scaling**: 演出难度随乐队成长线性提升；始终保持在"努力就能过"的区间，失败是故事挫折而非游戏结束。
- **Feedback clarity**: 每次登录都有明确的数值变化、新解锁内容、新触发事件。成长是可见的。
- **Recovery from failure**: 演出失败不会重置进度，而是触发叙事事件（"观众嘘声让我们更团结了"或"A 陷入了自我怀疑"），玩家可以选择如何应对。

---

## Core Loop

### Moment-to-Moment (30 seconds)

核心动作：**设置排练计划**

1. 选择今天要排练的曲目
2. 分配参与排练的队员（1-4人）
3. 设定排练重点（技巧熟练度 / 团队默契 / 特定技能）
4. 确认，进度条开始推进

这是玩家最频繁做的事——设置后离开，几小时后回来验收。零操作压力，成长感清晰。

### Short-Term (5-15 minutes)

一次"排练回合"的典型流程：

1. 登录，收上次排练成果（技能提升、化学反应变化）
2. 查看触发的叙事事件（A和B吵架了、C练会了新技巧、收到演出邀约）
3. 处理事件（选择如何回应——每个选择有倾向性后果，但包含随机惊喜）
4. 设置下一次排练计划（选曲目、分配队员、设定重点）
5. 放置等待，或消耗资源加速

**"再来一次"触发点**：每次回来都有新东西——数值涨了、解锁了新事件、队员关系有了微妙变化。

### Session-Level (30-120 minutes)

一次完整的游戏会话：

1. 登录收累积的成果
2. 处理堆积的叙事事件和决策
3. 规划接下来几次排练
4. 可能触发阶段性目标（如"下周有校园演出"——需要紧急特训）
5. **自然停止点**：设置好下一轮排练计划后，放心离开

**不玩时的钩子**：想着"过几个小时回来，看看队员们练得怎么样了"——放置游戏的核心留存魔法。

### Long-Term Progression

- **技能维度**：个人乐器技能、乐队整体默契度、曲目熟练度
- **解锁维度**：新曲目、新演出场地、新配合技、新剧情章节
- **关系维度**：队员关系网络、外部关系（经纪人、粉丝、对手乐队）
- **终极目标**：从地下室排练室 → 校园演出 → 本地Livehouse → 音乐节 → 摇滚巨星
- **完成标志**：登上最大舞台，或者选择保持地下乐队的纯粹（多结局）

### Retention Hooks

- **Curiosity**: "下次回来会看到什么新事件？A和B的关系怎么样了？下一个场地什么时候解锁？"
- **Investment**: "我已经培养了这支乐队三个月，想知道他们的结局。"
- **Social**: 队员故事产生的情感连接——"想回去看看他们最近怎么样。"
- **Mastery**: 发现更优的排练策略、解锁隐藏化学反应组合。

---

## Game Pillars

### Pillar 1: 成长是可见的

每一次登录都必须能看到变化——无论是数值涨了、解锁了新内容、还是触发了新事件。

*Design test*: 当我们在争论"要不要把这个成长藏得更深"时，这个支柱说：**不，让玩家看到每一次进步**。

### Pillar 2: 人物是活的

队员不只是数值容器。他们有性格、有故事、会开心会生气，玩家的选择会真实地影响他们。

*Design test*: 当我们在争论"为了简化，把队员做成纯技能模板"时，这个支柱说：**不，每个人物都要有灵魂**。

### Pillar 3: 节奏是轻松的

失败存在，但不会是灾难性的。演出搞砸了、队员吵架了——这些都是故事的一部分。玩家总能找到办法继续，不会有"存档丢失"或"必须重开"的毁灭性惩罚。压力是叙事性的（"下一场演出不能搞砸"），不是系统性的（"输了就清零"）。

*Design test*: 当我们在争论"要不要加入'乐队解散'的永久失败状态"时，这个支柱说：**不，但允许演出失败、关系恶化这些可恢复的剧情挫折**。

### Pillar 4: 选择是有意义的

每个决策都有方向和倾向性的后果，但具体结果包含不可预测的元素。选择让A和B一起排练**大多**会提升默契，但偶尔也会触发意外事件（比如A突然表白）。这意味着"有意义"不等于"完全可控"——就像真实生活中，你做了最好的选择，但结果仍有变数。

*Design test*: 当我们在争论"所有决策结果是否必须是确定性的"时，这个支柱说：**不，允许惊喜和意外，但要确保大方向是玩家选择塑造的**。

### Anti-Pillars (What This Game Is NOT)

- **NOT 节奏游戏**: 不会加入任何需要手速或节奏的实时操作，因为那会违背"节奏是轻松的"支柱。
- **NOT 竞技游戏**: 没有PvP、没有排行榜压力、没有"别人比你强"的焦虑，因为那会违背"节奏是轻松的"支柱。
- **NOT 硬核模拟**: 不需要真实的音乐理论知识、不需要理解乐理，因为那会提高门槛，违背所有支柱。
- **NOT 无限刷怪**: 不会用"刷材料-升级-刷更强的材料"这种空洞循环来填充时间，因为那会违背"人物是活的"和"选择是有意义的"支柱。

---

## Inspiration and References

| Reference | What We Take From It | What We Do Differently | Why It Matters |
| ---- | ---- | ---- | ---- |
| 赛马娘 | 角色养成+放置节奏；成长感直接可见 | 我们的"队员"是有真实人际关系的乐队成员，不是偶像竞争 | 验证了放置养成+角色情感的商业模式 |
| 疯狂骑士团 | 极简挂机体验；随时来随时走 | 加入叙事和人际关系深度 | 验证了休闲放置类在移动端的受众规模 |
| 不休的乌拉拉 | 组队社交+自动战斗+持续解锁 | 把"组队"转化为乐队内部关系系统 | 验证了低操作+高成长反馈的留存模型 |
| 偶像梦幻祭 | 角色剧情+关系系统 | 更写实的大学生活背景，非偶像工业设定 | 验证了叙事驱动+角色养成的组合 |
| 星露谷物语 | 日常节奏+关系系统 | 聚焦音乐而非农场 | 验证了日常模拟+关系养成的长期吸引力 |

**Non-game inspirations**:
- 《海盗电台》(电影) — 地下音乐场景的浪漫与混乱
- 《Nana》(动漫) — 乐队成员之间的复杂关系
- 真实乐队故事 — Oasis、The Beatles 等乐队的分合史
- 大学地下室文化 — 穷但有梦想的青春叙事

---

## Target Player Profile

| Attribute | Detail |
| ---- | ---- |
| **Age range** | 18-35 |
| **Gaming experience** | 休闲到中核 |
| **Time availability** | 每天数次、每次5-15分钟的碎片化时间；偏好可以随时中断的游戏 |
| **Platform preference** | 移动端为主 |
| **Current games they play** | 疯狂骑士团、不休的乌拉拉、赛马娘、偶像梦幻祭、星露谷物语 |
| **What they're looking for** | 成长感（看着角色变强）、叙事沉浸（关心角色命运）、低压力（不需要技巧或手速） |
| **What would turn them away** | 需要反复练习的操作、失败惩罚过重、角色没有灵魂纯数值、强制社交或竞技 |

---

## Technical Considerations

| Consideration | Assessment |
| ---- | ---- |
| **Recommended Engine** | Godot 4 — 开源免费、2D支持优秀、移动端导出成熟、适合独立开发者短周期开发 |
| **Key Technical Challenges** | 1. 离线收益计算（放置核心）；2. 叙事事件的条件触发系统；3. 化学反应的组合爆炸管理 |
| **Art Style** | 2D 简约/像素风 — 低美术门槛，角色立绘+简单场景即可 |
| **Art Pipeline Complexity** | 低到中 — 角色头像/立绘 + 排练室场景 + UI 面板 |
| **Audio Needs** | 中 — 需要背景音乐（不同风格）和简单音效；MVP 可用占位音频 |
| **Networking** | 无 — 纯单机体验 |
| **Content Volume** | MVP: 1-2 角色、3-5 曲目、1-2 场地、5-10 事件。完整版: 4 角色、8-12 曲目、5-8 场地、30-50 事件 |
| **Procedural Systems** | 叙事事件模板化生成 — 用变量填充生成大量变体事件，减少手工内容量 |

---

## Risks and Open Questions

### Design Risks
- 核心循环（设置→等待→收获）在缺乏足够事件池时可能变得单调
- 化学反应系统如果过于复杂会让玩家感到困惑，过于简单则失去深度
- 放置节奏与叙事深度的平衡：玩家可能跳过文本只看数值

### Technical Risks
- 离线收益计算的准确性（时间同步、防作弊）
- 叙事事件系统的条件复杂度可能超出短周期可控范围
- 移动端性能：Godot 在低端安卓设备上的表现需要验证

### Market Risks
- 放置养成类在移动端竞争激烈（已有成熟产品占据市场）
- 乐队题材相比偶像/赛马等题材受众面更窄
- 叙事驱动游戏的本地化成本较高（如果想出海）

### Scope Risks
- 叙事事件的内容量可能在短周期内不足，需要依赖模板化生成
- 化学反应系统的组合数量随角色增加呈指数增长
- MVP 到完整版的内容扩展需要持续投入

### Open Questions
- **化学反应系统的复杂度边界在哪里？** → MVP 中只实现 2-3 对核心组合，通过原型验证
- **放置节奏下玩家愿意读多少文本？** → MVP 中测试不同长度的事件描述对留存的影响
- **离线多久的收益是合理的？** → 参考同类放置游戏，MVP 中用 4-8 小时上限测试

---

## MVP Definition

**Core hypothesis**: 玩家会发现"设置排练计划 → 放置等待 → 回来收获成长"的核心循环在移动端放置场景下具有持续的吸引力，尤其是当成长伴随着角色叙事时。

**Required for MVP**:
1. 放置排练系统：设置计划（曲目+队员+重点）→ 时间推进 → 验收成果
2. 1-2 个乐队成员，每人有 3-4 项技能属性
3. 基础化学反应系统：1-2 对核心组合，可见的数值影响
4. 5-10 条叙事事件，展示角色个性和关系动态
5. 1 个演出场地，验证"排练成果 → 演出判定 → 声誉/解锁"的闭环
6. 离线收益计算（基础版）

**Explicitly NOT in MVP** (defer to later):
- 多结局系统
- 复杂的队员关系网络图
- 外部角色（经纪人、粉丝）系统
- 加速/跳过等付费机制
- 音效和背景音乐（可用占位音频）

### Scope Tiers

| Tier | Content | Features | Timeline |
| ---- | ---- | ---- | ---- |
| **MVP** | 1-2 角色、3-5 曲目、1 场地、5-10 事件 | 核心排练循环 + 基础化学反应 + 演出判定 + 离线收益 | 3-4 周 |
| **Vertical Slice** | 2-3 角色、5-8 曲目、2 场地、15-20 事件 | MVP + 完整化学反应系统 + 队员关系面板 + 更多演出类型 | 6-8 周 |
| **Alpha** | 3-4 角色、8-10 曲目、3-5 场地、25-35 事件 | 所有核心功能 + 占位美术 + 基础音效 | 10-12 周 |
| **Full Vision** | 4 角色、12 曲目、8 场地、50 事件 + 多结局 | 完整功能 + 全部美术 + 完整音效 + 本地化 | 16-20 周 |

---

## Visual Identity Anchor

**Selected visual direction**: Grunge / Underground Rock (垃圾摇滚地下美学)

**One-line visual rule**: "像从垃圾堆里捡来的、被雨水泡过又晒干的演出传单——你几乎看不清上面的字，但你感受得到那股不服输的劲儿。"

**Supporting visual principles**:

1. **垃圾不是隐喻，是字面意思**
   - 污渍、划痕、咖啡渍、胶带痕迹、手写字体、复印件的噪点——这不是瑕疵，是生活的痕迹
   - Design test: 如果一个UI元素看起来像是刚从印刷厂出来的完美成品，那它就不对

2. **反叛是默认状态**
   - 每一帧都在说"去你的规则"。倾斜的构图、截断的文字、故意的不对齐、反传统的配色——视觉本身就是一种态度
   - Design test: 如果画面看起来"平衡"或"和谐"，那就打破它

3. **不完美是最高标准**
   - 完美是敌人，polish 是犯罪。队员有黑眼圈、衣服皱巴巴、姿势僵硬——因为真实的人就是这样
   - Design test: 如果你看到某个元素想"这里应该再修一下"——停下，它就是对的

**Color philosophy**: 脏黄/烟黄为主色（像被烟熏黄的墙壁），铁锈红为强调色（像生锈的管道和干涸的血），烟灰/水泥灰为阴影色。演出时刻用刺眼荧光色（粉/绿）从黑暗中爆发。整体低饱和，关键跳脱元素高饱和——日常生活是"脏"的，梦想时刻是"刺眼"的。

---

## Next Steps

- [ ] Run `/setup-engine` to configure Godot 4 and populate version-aware reference docs
- [ ] Run `/art-bible` to create the visual identity specification — do this BEFORE writing GDDs
- [ ] Use `/design-review design/gdd/game-concept.md` to validate concept completeness
- [ ] Decompose the concept into individual systems with `/map-systems`
- [ ] Author per-system GDDs with `/design-system [system-name]`
- [ ] Plan the technical architecture with `/create-architecture`
- [ ] Record key architectural decisions with `/architecture-decision (×N)`
- [ ] Validate readiness to advance with `/gate-check pre-production`
- [ ] Prototype the riskiest system with `/prototype rehearsal-loop`
- [ ] Run `/playtest-report` after the prototype
- [ ] Plan the first sprint with `/sprint-plan new`
