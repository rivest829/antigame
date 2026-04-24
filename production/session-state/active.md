# Session State

## Current Task
- **Epic**: Game Design
- **Feature**: 关系矩阵系统 GDD
- **Task**: 已完成 — 所有 11 节已写入并批准

## Progress Checklist
- [x] Brainstorm — game concept created
- [x] Setup Engine — Godot 4.6 + GDScript configured
- [x] Art Bible — sections 1-4 completed
- [x] Map Systems — 24 systems enumerated
- [x] Design System — character-stats.md GDD completed
- [x] Design 曲目数据库 GDD — completed
- [x] Design 内容数据库 GDD — completed
- [x] Design 时间管理离线收益 GDD — completed
- [x] Design 事件模板系统 GDD — completed
- [x] Design 存档加载系统 GDD — completed
- [x] Design 场景管理器 GDD — completed
- [x] Design 资源加载系统 GDD — completed
- [x] Design 音频系统 GDD — completed
- [x] Design 放置排练系统 GDD — completed

## Progress Checklist
- [x] Brainstorm — game concept created
- [x] Setup Engine — Godot 4.6 + GDScript configured
- [x] Art Bible — sections 1-4 completed (Visual Identity, Mood, Shape, Color)
- [x] Map Systems — 24 systems enumerated, dependencies mapped, priorities assigned
- [x] Design System — character-stats.md GDD completed (11 sections)
- [x] Design next system — Song Database (completed)
- [x] Design 曲目数据库 GDD — all sections completed
- [x] Design 内容数据库 GDD — all sections completed
- [x] Design 时间管理离线收益 GDD — all sections completed
- [x] Design 事件模板系统 GDD — all sections completed
- [x] Design 存档加载系统 GDD — all sections completed
- [x] Design 场景管理器 GDD — all sections completed
- [x] Design 资源加载系统 GDD — all sections completed
- [x] Design 音频系统 GDD — all sections completed

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

## Foundation Layer Complete
All 9 Foundation layer systems are now designed:
1. 角色属性系统 (Character Stats)
2. 曲目数据库 (Song Database)
3. 内容数据库 (Content Database)
4. 事件模板系统 (Event Template)
5. 时间管理/离线收益 (Time & Offline Revenue)
6. 存档/加载系统 (Save/Load)
7. 场景管理器 (Scene Manager)
8. 资源加载系统 (Resource Loader)
9. 音频系统 (Audio System)

## Files Being Worked On
- `design/gdd/game-concept.md` — game concept document
- `design/gdd/systems-index.md` — systems enumeration and dependencies
- `design/gdd/character-stats.md` — first GDD (complete)
- `design/gdd/song-database.md` — song database GDD (complete)
- `design/gdd/content-database.md` — content database GDD (complete)
- `design/gdd/time-offline-revenue.md` — time/offline revenue GDD (complete)
- `design/gdd/event-template.md` — event template system GDD (complete)
- `design/gdd/save-load.md` — save/load system GDD (complete)
- `design/gdd/scene-manager.md` — scene manager GDD (complete)
- `design/gdd/resource-loader.md` — resource loader GDD (complete)
- `design/gdd/audio-system.md` — audio system GDD (complete)
- `design/art/art-bible.md` — art bible sections 1-4
- `design/registry/entities.yaml` — entity registry (populated)

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

## Next Step
Foundation 层全部完成！现在可以进入 Core 层设计。

Design the next system in the design order:
1. `/design-system 放置排练` (Idle Rehearsal — Core, 核心循环, 最重要的系统)
2. `/design-system 关系矩阵` (Relationship Matrix — Core, 简单)
3. `/design-system 演出难度` (Performance Difficulty — Core, 简单)
4. `/design-system 声誉系统` (Reputation System — Core, 简单)
