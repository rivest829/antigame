# Systems Index: 地下排练室 (Underground Rehearsal Room)

> **Status**: Draft
> **Created**: 2026-04-23
> **Last Updated**: 2026-04-23
> **Source Concept**: design/gdd/game-concept.md

---

## Overview

《地下排练室》是一款移动端放置养成游戏，核心循环为"设置排练计划 → 放置等待 → 回来收获成长 + 处理叙事事件"。玩家从零开始组建乐队，通过排练提升技能、培养队员间的化学反应、触发叙事事件，最终从地下室走向摇滚巨星之路。

游戏需要21个MVP系统来验证核心循环的乐趣，外加3个Vertical Slice系统来完善体验。系统按依赖层级分为Foundation（基础数据）、Core（核心玩法）、Feature（衍生玩法）、Presentation（UI展示）和Polish（打磨）。

---

## Systems Enumeration

| # | System Name | Category | Priority | Status | Design Doc | Depends On |
|---|-------------|----------|----------|--------|------------|------------|
| 1 | 角色属性系统 (Character Stats) | Core | MVP | Designed | design/gdd/character-stats.md | — |
| 2 | 曲目数据库 (Song Database) | Core | MVP | Designed | design/gdd/song-database.md | — |
| 3 | 内容数据库 (Content Database) | Core | MVP | Designed | design/gdd/content-database.md | — |
| 4 | 事件模板系统 (Event Template) | Narrative | MVP | Designed | design/gdd/event-template.md | — |
| 5 | 时间管理/离线收益系统 (Time & Offline Revenue) | Core | MVP | Designed | design/gdd/time-offline-revenue.md | — |
| 6 | 存档/加载系统 (Save/Load) | Persistence | MVP | Designed | design/gdd/save-load.md | — |
| 7 | 场景管理器 (Scene Manager) | Core | MVP | Designed | design/gdd/scene-manager.md | — |
| 8 | 资源加载系统 (Resource Loader) | Core | MVP | Designed | design/gdd/resource-loader.md | — |
| 9 | 音频系统 (Audio System) | Audio | MVP | Designed | design/gdd/audio-system.md | — |
| 10 | 放置排练系统 (Idle Rehearsal) | Gameplay | MVP | Designed | design/gdd/idle-rehearsal.md | 角色属性, 曲目数据库, 时间管理 |
| 11 | 关系矩阵系统 (Relationship Matrix) | Gameplay | MVP | Designed | design/gdd/relationship-matrix.md | 角色属性 |
| 12 | 演出难度系统 (Performance Difficulty) | Gameplay | MVP | Designed | design/gdd/performance-difficulty.md | 内容数据库 |
| 13 | 声誉系统 (Reputation System) | Progression | MVP | Designed | design/gdd/reputation-system.md | 内容数据库 |
| 14 | 化学反应系统 (Chemistry System) | Gameplay | MVP | Designed | design/gdd/chemistry-system.md | 放置排练, 关系矩阵 |
| 15 | 演出判定系统 (Performance Judgment) | Gameplay | MVP | Designed | design/gdd/performance-judgment.md | 放置排练, 演出难度, 声誉 |
| 16 | 叙事事件系统 (Narrative Events) | Narrative | MVP | Designed | design/gdd/narrative-events.md | 事件模板, 化学反应, 关系矩阵 |
| 17 | 进度解锁系统 (Progression Unlock) | Progression | MVP | Designed | design/gdd/progression-unlock.md | 演出判定, 声誉, 内容数据库 |
| 18 | 排练计划UI (Rehearsal Planning UI) | UI | MVP | Designed | design/gdd/rehearsal-planning-ui.md | 放置排练 |
| 19 | 主界面/仪表盘UI (Main Dashboard UI) | UI | MVP | Designed | design/gdd/main-dashboard-ui.md | 放置排练, 进度解锁 |
| 20 | 叙事事件UI (Narrative Event UI) | UI | MVP | Designed | design/gdd/narrative-event-ui.md | 叙事事件 |
| 21 | 演出结果UI (Performance Result UI) | UI | MVP | Designed | design/gdd/performance-result-ui.md | 演出判定 |
| 22 | 通知系统 (Notification System) | Meta | Vertical Slice | Not Started | — | 时间管理, 进度解锁 |
| 23 | 新手引导系统 (Tutorial System) | Meta | Vertical Slice | Not Started | — | 几乎所有系统 |
| 24 | 设置菜单UI (Settings Menu UI) | UI | Vertical Slice | Not Started | — | 音频系统, 存档/加载 |

---

## Categories

| Category | Description | Systems in This Game |
|----------|-------------|----------------------|
| **Core** | Foundation systems everything depends on | 角色属性, 曲目数据库, 内容数据库, 时间管理, 场景管理器, 资源加载 |
| **Gameplay** | The systems that make the game fun | 放置排练, 关系矩阵, 化学反应, 演出难度, 演出判定 |
| **Progression** | How the player grows over time | 声誉, 进度解锁 |
| **Persistence** | Save state and continuity | 存档/加载 |
| **UI** | Player-facing information displays | 排练计划UI, 主界面UI, 叙事事件UI, 演出结果UI, 设置菜单UI |
| **Audio** | Sound and music systems | 音频系统 |
| **Narrative** | Story and dialogue delivery | 事件模板, 叙事事件 |
| **Meta** | Systems outside the core game loop | 通知系统, 新手引导系统 |

---

## Priority Tiers

| Tier | Definition | Target Milestone | Design Urgency |
|------|------------|------------------|----------------|
| **MVP** | Required for the core loop to function. Without these, you can't test "is this fun?" | First playable prototype (3-4 weeks) | Design FIRST |
| **Vertical Slice** | Required for one complete, polished experience. Demonstrates the full potential. | Vertical slice (6-8 weeks) | Design SECOND |
| **Full Vision** | Polish, edge cases, nice-to-haves, and content-complete features. | Alpha / Release | Design as needed |

---

## Dependency Map

Systems sorted by dependency order — design and build from top to bottom.

### Foundation Layer (no dependencies)

1. **角色属性系统** — 所有角色相关系统的根基：技能属性、性格特征、外观定义
2. **曲目数据库** — 可排练曲目的静态数据：难度、所需技能、练习收益
3. **内容数据库** — 所有可解锁内容的静态数据：场地、配合技、剧情章节
4. **事件模板系统** — 叙事事件的模板定义：条件、变量槽、分支结构
5. **时间管理/离线收益系统** — 放置游戏的心脏：实时/离线时间计算、收益公式
6. **存档/加载系统** — 进度持久化：序列化、版本兼容、云同步预留
7. **场景管理器** — 场景切换和状态保持：转场动画、状态恢复
8. **资源加载系统** — 动态资源管理：图片、音频的异步加载和缓存
9. **音频系统** — 音频播放管理：BGM切换、SFX播放、占位音频支持

### Core Layer (depends on foundation)

10. **放置排练系统** — depends on: 角色属性, 曲目数据库, 时间管理
11. **关系矩阵系统** — depends on: 角色属性
12. **演出难度系统** — depends on: 内容数据库
13. **声誉系统** — depends on: 内容数据库

### Feature Layer (depends on core)

14. **化学反应系统** — depends on: 放置排练, 关系矩阵
15. **演出判定系统** — depends on: 放置排练, 演出难度, 声誉
16. **叙事事件系统** — depends on: 事件模板, 化学反应, 关系矩阵
17. **进度解锁系统** — depends on: 演出判定, 声誉, 内容数据库

### Presentation Layer (depends on features)

18. **排练计划UI** — depends on: 放置排练
19. **主界面/仪表盘UI** — depends on: 放置排练, 进度解锁
20. **叙事事件UI** — depends on: 叙事事件
21. **演出结果UI** — depends on: 演出判定

### Polish Layer (depends on everything)

22. **通知系统** — depends on: 时间管理, 进度解锁
23. **新手引导系统** — depends on: 几乎所有系统
24. **设置菜单UI** — depends on: 音频系统, 存档/加载

---

## Recommended Design Order

| Order | System | Priority | Layer | Est. Effort | Why This Order |
|-------|--------|----------|-------|-------------|----------------|
| 1 | 角色属性系统 | MVP | Foundation | M | 所有角色相关系统的根基——没有它，排练、化学反应、演出都无法运作 |
| 2 | 曲目数据库 | MVP | Foundation | S | 排练系统需要知道有哪些曲目可练 |
| 3 | 内容数据库 | MVP | Foundation | S | 解锁系统需要知道有哪些内容可解锁 |
| 4 | 事件模板系统 | MVP | Foundation | M | 叙事事件的根基——用变量模板减少手工内容量 |
| 5 | 时间管理/离线收益 | MVP | Foundation | M | 放置游戏的心脏——没有它就没有"放置" |
| 6 | 存档/加载系统 | MVP | Foundation | S | 放置进度必须持久化 |
| 7 | 场景管理器 | MVP | Foundation | S | 所有场景切换的基础 |
| 8 | 资源加载系统 | MVP | Foundation | S | 动态加载图片、音频 |
| 9 | 音频系统 | MVP | Foundation | S | MVP用占位音频，但架构要预留 |
| 10 | 放置排练系统 | MVP | Core | M | 核心循环的核心——"设置→等待→收获" |
| 11 | 关系矩阵系统 | MVP | Core | S | 化学反应和叙事事件都需要队员关系数据 |
| 12 | 演出难度系统 | MVP | Core | S | 演出判定需要知道不同场地的难度标准 |
| 13 | 声誉系统 | MVP | Core | S | 演出结果影响解锁，是成长感的重要组成部分 |
| 14 | 化学反应系统 | MVP | Feature | M | 验证"队员搭配有策略意义"——MVP只实现1-2对核心组合 |
| 15 | 演出判定系统 | MVP | Feature | S | 验证"排练投入→演出产出"的闭环 |
| 16 | 叙事事件系统 | MVP | Feature | M | MVP只需5-10条事件，验证"叙事+数值"混合体验 |
| 17 | 进度解锁系统 | MVP | Feature | S | 成长感的直接载体——每次解锁都给玩家正反馈 |
| 18 | 排练计划UI | MVP | Presentation | M | 核心交互界面——玩家最频繁使用的界面 |
| 19 | 主界面/仪表盘UI | MVP | Presentation | M | 登录后的第一印象——收成果、看状态、做决策 |
| 20 | 叙事事件UI | MVP | Presentation | M | 故事体验的窗口——要符合垃圾摇滚美学 |
| 21 | 演出结果UI | MVP | Presentation | S | 反馈闭环——让玩家看到排练投入换来的成果 |
| 22 | 通知系统 | VS | Polish | S | 放置游戏的留存核心——离线后提醒玩家回来 |
| 23 | 新手引导系统 | VS | Polish | M | 教会玩家"设置→等待→收获"的节奏 |
| 24 | 设置菜单UI | VS | Polish | S | 音量、通知偏好等 |

*Effort: S = 1 session (~30min design conversation), M = 2-3 sessions, L = 4+ sessions*

---

## Circular Dependencies

- **None found** — 依赖图是清晰的DAG（有向无环图）。

---

## High-Risk Systems

| System | Risk Type | Risk Description | Mitigation |
|--------|-----------|-----------------|------------|
| 时间管理/离线收益 | Technical | 离线收益计算准确性（时间同步、防作弊、设备休眠处理） | MVP先用简单时间差计算；后续优化防作弊 |
| 化学反应系统 | Design | 复杂度边界——太简单失去深度，太复杂让玩家困惑 | MVP只实现2-3对核心组合；通过原型验证玩家接受度 |
| 叙事事件系统 | Scope | 内容量可能超出短周期——需要模板化生成来保证事件数量 | 设计事件模板系统，用变量填充生成大量变体；MVP只需5-10条核心事件 |
| 事件模板系统 | Design | 模板化生成可能导致事件同质化，缺乏个性 | 设计丰富的变量池和分支条件；确保每个事件有独特的情境感 |

---

## Progress Tracker

| Metric | Count |
|--------|-------|
| Total systems identified | 24 |
| Design docs started | 21 |
| Design docs reviewed | 0 |
| Design docs approved | 0 |
| MVP systems designed | 21/21 |
| Vertical Slice systems designed | 0/3 |

---

## Next Steps

- [x] Design Foundation-layer systems (9/9 complete)
- [x] Design Core-layer systems: 放置排练, 关系矩阵, 演出难度, 声誉系统 (4/4 complete)
- [x] Design Feature-layer systems: 化学反应, 演出判定, 叙事事件, 进度解锁 (4/4 complete)
- [x] Design Presentation-layer systems: 排练计划UI, 主界面UI, 叙事事件UI, 演出结果UI (4/4 complete)
- [ ] Design Polish-layer systems (Vertical Slice): 通知, 新手引导, 设置菜单
- [ ] Run `/design-review` on each completed GDD
- [ ] Run `/gate-check pre-production` when all 21 MVP systems are designed
- [ ] Run `/design-review` on each completed GDD
- [ ] Run `/gate-check pre-production` when all 21 MVP systems are designed
