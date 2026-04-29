# Session State

## Current Task
- **Epic**: Game Design
- **Feature**: MVP 系统 GDD 补全
- **Task**: 已完成 — 全部 21 个 MVP 系统 GDD 已写入

## Previous Task
- **Feature**: 演出判定系统 GDD
- **Status**: 已完成

## Progress Checklist
- [x] Brainstorm — game concept created
- [x] Setup Engine — Godot 4.6 + GDScript configured
- [x] Art Bible — sections 1-4 completed
- [x] Map Systems — 24 systems enumerated
- [x] Design Foundation Layer (9/9) — all GDDs completed
- [x] Design Core Layer (4/4) — all GDDs completed
- [x] Design Feature Layer (4/4) — all GDDs completed
- [x] Design Presentation Layer (4/4) — all GDDs completed
- [ ] Design Polish Layer (Vertical Slice, 0/3) — 通知系统、新手引导、设置菜单UI
- [ ] Run `/design-review` on completed GDDs
- [ ] Run `/gate-check pre-production`

## Foundation Layer (9/9 Complete)
1. 角色属性系统 (Character Stats)
2. 曲目数据库 (Song Database)
3. 内容数据库 (Content Database)
4. 事件模板系统 (Event Template)
5. 时间管理/离线收益 (Time & Offline Revenue)
6. 存档/加载系统 (Save/Load)
7. 场景管理器 (Scene Manager)
8. 资源加载系统 (Resource Loader)
9. 音频系统 (Audio System)

## Core Layer (4/4 Complete)
10. 放置排练系统 (Idle Rehearsal)
11. 关系矩阵系统 (Relationship Matrix)
12. 演出难度系统 (Performance Difficulty)
13. 声誉系统 (Reputation System)

## Feature Layer (4/4 Complete)
14. 化学反应系统 (Chemistry System)
15. 演出判定系统 (Performance Judgment)
16. 叙事事件系统 (Narrative Events)
17. 进度解锁系统 (Progression Unlock)

## Presentation Layer (4/4 Complete)
18. 排练计划UI (Rehearsal Planning UI)
19. 主界面/仪表盘UI (Main Dashboard UI)
20. 叙事事件UI (Narrative Event UI)
21. 演出结果UI (Performance Result UI)

## Key Decisions Made
- Engine: Godot 4.6 + GDScript
- Visual Direction: Grunge / Underground Rock (垃圾摇滚地下美学)
- Character Stats: 乐器技能 + 性格特征 + 状态值（心情/精力/士气）
- MVP Characters: 林野（技术型吉他手）+ 大伟（社交型鼓手）
- Social mechanic: 社交提升心情但消耗精力（双刃剑）
- Song Database: 5 MVP songs, proficiency 0-100%, 3 curve types
- Content Database: 1 venue + 1 combo + 2 chapters for MVP
- Time/Offline: 6-hour growth cap, 12-hour status cap, hourly energy simulation
- Event Template: 4-layer structure (Template/Variable/Condition/Branch), 5 event types, 10 MVP templates
- Save/Load: JSON format, version migration, 3 rolling backups, atomic write, integrity score >= 0.8
- Scene Manager: 1 main scene + 4 sub scenes + 3+ overlays, 5 transition types, back stack depth 5
- Resource Loader: 3-tier cache (Resident/LRU/No-Cache), 3 load modes (Preload/Background/Ondemand), 64MB budget
- Audio System: 4-layer Bus (Master/BGM/SFX/Ambient+UI), MVP placeholder audio, 5 BGM tracks, 13 SFX, fade in/out
- Idle Rehearsal: 4 focuses (Balanced/Technical/Chemistry/EnergySave), hourly growth, chemistry trigger 15%
- Relationship Matrix: 0-100 score, compatibility 0.5-1.5, chemistry-driven narrative
- Performance Difficulty: 0-100 score, venue modifier, readiness penalty, morale factor
- Reputation System: 0-200 points, 5 tiers, soft cap at 150, audience expectation 1.0-1.3
- Chemistry System: 3 tiers (Basic/Core/Peak), streak multiplier, relationship threshold
- Performance Judgment: 4 dimensions (Technique/Synergy/Stage/Audience), 5 ratings, failure protection
- Narrative Events: runtime event scheduler, max 3 events/session, pileup handling, delayed consequences
- Progression Unlock: 6 condition types, chain unlocks, batch notifications, retroactive check
- Rehearsal Planning UI: 3-zone layout, drag-and-drop, auto-assign, real-time preview
- Main Dashboard UI: offline reward animation, character cards, rehearsal status, red dot notifications
- Narrative Event UI: typewriter effect, paper card design, burn animation, branch options
- Performance Result UI: recall animation, score scroll, rating reveal, reward display, feedback text

## Files Being Worked On
- `design/gdd/game-concept.md` — game concept document
- `design/gdd/systems-index.md` — systems enumeration and dependencies (updated: 21/21 MVP designed)
- `design/gdd/character-stats.md` — character stats GDD
- `design/gdd/song-database.md` — song database GDD
- `design/gdd/content-database.md` — content database GDD
- `design/gdd/time-offline-revenue.md` — time/offline revenue GDD
- `design/gdd/event-template.md` — event template system GDD
- `design/gdd/save-load.md` — save/load system GDD
- `design/gdd/scene-manager.md` — scene manager GDD
- `design/gdd/resource-loader.md` — resource loader GDD
- `design/gdd/audio-system.md` — audio system GDD
- `design/gdd/idle-rehearsal.md` — idle rehearsal system GDD
- `design/gdd/relationship-matrix.md` — relationship matrix GDD
- `design/gdd/performance-difficulty.md` — performance difficulty GDD
- `design/gdd/reputation-system.md` — reputation system GDD
- `design/gdd/chemistry-system.md` — chemistry system GDD
- `design/gdd/performance-judgment.md` — performance judgment GDD
- `design/gdd/narrative-events.md` — narrative events system GDD
- `design/gdd/progression-unlock.md` — progression unlock system GDD
- `design/gdd/rehearsal-planning-ui.md` — rehearsal planning UI GDD
- `design/gdd/main-dashboard-ui.md` — main dashboard UI GDD
- `design/gdd/narrative-event-ui.md` — narrative event UI GDD
- `design/gdd/performance-result-ui.md` — performance result UI GDD
- `design/art/art-bible.md` — art bible sections 1-4

## Open Questions
- 是否需要第四项状态值？
- 角色外观自定义是否属于本系统？
- 数据结构是否需要预留4+角色扩展？
- 事件模板是否需要预留 i18n 字段？
- 条件语法是否需要支持数学表达式？
- 存档是否需要导出/导入功能用于调试？
- 云同步接口预留到什么程度？
- 场景是否需要预加载？
- DISSOLVE 转场效果 MVP 是否实现？
- 纹理压缩格式如何选择？
- 是否需要实现资源的引用计数自动释放？
- MVP 音频全部使用免版税资源还是原创合成？
- 是否需要动态 BGM 分层系统？
- 事件触发频率 0.3/hour 是否需要 playtest 验证？
- 解锁通知批量显示阈值 5 是否合适？
- 排练计划UI的化学反应概率预览是否应该显示具体数值？
- 主界面背景的7天连续登录变化是否增加开发量？
- 叙事事件UI的打字机效果是否需要支持语速变化？
- 演出结果UI的回想动画是否在MVP中实现？

## Next Step
全部 21 个 MVP 系统 GDD 已完成！接下来可选方向：

1. **设计审查**：运行 `/design-review` 和 `/review-all-gdds` 对已完成GDD进行交叉审查
2. **阶段门检查**：运行 `/gate-check pre-production` 验证是否具备进入生产期的条件
3. **Vertical Slice 系统设计**：继续设计剩余的 3 个 Vertical Slice 系统（通知系统、新手引导、设置菜单UI）
4. **引擎初始化**：运行 `/setup-engine` 初始化 Godot 4.6 项目
5. **架构决策**：为尚未覆盖的系统补充 ADR（当前只有 3 个 ADR，覆盖不足）
