# ADR-0003: Scene Manager — Layered Scene Stack with Typed Transitions and Back Navigation

## Status

Accepted

## Date

2026-04-24

## Last Verified

2026-04-24

## Decision Makers

User + godot-specialist agent + ui-programmer agent

## Summary

The scene manager must provide fluid navigation between game screens while maintaining state and supporting "back" navigation. We decided on a 3-class scene hierarchy (Main / Sub / Overlay), a finite back-stack (max depth 5), 5 transition types with configurable timing, and a 7-state scene lifecycle. This design supports the "节奏是轻松的" pillar by making every scene change feel intentional and smooth.

## Engine Compatibility

| Field | Value |
|-------|-------|
| **Engine** | Godot 4.6 |
| **Domain** | UI / Core |
| **Knowledge Risk** | LOW — Godot's `SceneTree.change_scene_to_file()`, `CanvasLayer`, and `Tween` are well-established APIs |
| **References Consulted** | `docs/engine-reference/godot/VERSION.md` |
| **Post-Cutoff APIs Used** | None |
| **Verification Required** | Test back-stack behavior with rapid "open→back→open→back" touch input; verify transition animations complete within 1.0s on low-end Android |

## ADR Dependencies

| Field | Value |
|-------|-------|
| **Depends On** | ADR-0002 (Resource Loader — scene transitions depend on preloaded scene resources) |
| **Enables** | All UI systems (排练计划UI, 主界面UI, 叙事事件UI, 演出结果UI, 设置菜单UI) |
| **Blocks** | Any story implementing a UI screen or scene transition |
| **Ordering Note** | Scene Manager is the second system to implement after ResourceLoader. UI stories can begin once the manager + at least FADE and SLIDE transitions are functional. |

## Context

### Problem Statement

A mobile放置 game has many screens: dashboard, rehearsal planning, narrative events, performance, settings. Without centralized scene management, each screen would handle its own transitions, leading to inconsistent animations, lost state, and broken "back" behavior. We need a single system that owns all navigation.

### Current State

No scene management exists. Godot projects typically use `get_tree().change_scene_to_file()` directly, which destroys the current scene — unsuitable for our layered UI needs.

### Constraints

- **Platform**: Touch-only mobile — no keyboard "Esc" for back; must provide visible back button
- **Aesthetic**: Grunge / underground rock — transitions should feel like "flipping through a zine" not "switching apps"
- **State preservation**: Sub-scenes must retain state when covered (e.g., rehearsal plan stays visible when event pops up)
- **Memory**: Only 1 main scene + active sub-scenes kept in memory; unloaded scenes release resources

### Requirements

- Single API for all scene navigation
- Back button works correctly across all screens
- Transitions are stylized and consistent
- No scene is destroyed while player is looking at it
- Overlays can stack (max 3 deep) without breaking back navigation

## Decision

### 1. Scene Classification: 3 Types

| Type | Count (MVP) | Behavior | Example |
|------|-------------|----------|---------|
| **Main** | 1 | Never unloaded; PAUSED when covered | main_dashboard |
| **Sub** | 4 | Stacked with back navigation; UNLOADED when exited | rehearsal_planning, narrative_event, performance, boot |
| **Overlay** | 3+ | Stack on top; no back entry; closed returns to underlayer | settings_menu, event_detail, unlock_notification |

**Key rule**: Overlays do NOT enter the back stack. Closing an overlay returns to whatever was underneath, whether that's a sub-scene or the main scene.

### 2. Back Stack: Max Depth 5

The back stack tracks sub-scene navigation:

```
main_dashboard → rehearsal_planning → narrative_event
```

- **Max depth**: 5 layers (to prevent infinite nesting)
- **Overflow behavior**: Oldest non-main entry is removed
- **Back action**: Pop top entry, return to previous scene with reverse transition
- **Clear condition**: Returning to main_dashboard empties the stack

### 3. Transition Types: 5 Variants

| Type | Visual | Duration | Use Case |
|------|--------|----------|----------|
| **FADE** | Fade out (0.3s) → black (0.1s) → fade in (0.3s) | 0.7s | Main ↔ Sub |
| **SLIDE** | New slides in from right, old slides out left | 0.4s | Sub ↔ Sub |
| **POP** | Overlay bounces up from bottom/center | 0.2s | Open overlay |
| **DISSOLVE** | Pixelated shader dissolve | 0.5s | Milestone moments (first show) |
| **INSTANT** | No animation | 0s | Debug, error recovery |

**Transition formula**:
```
duration = base_duration × speed_multiplier
duration = min(duration, 1.0)  # hard cap
```

### 4. Scene Lifecycle: 7 States

```
REGISTERED → LOADING → ENTERING → ACTIVE → PAUSED → EXITING → UNLOADED
```

| State | Input | Update | Render | Transition Target |
|-------|-------|--------|--------|-------------------|
| REGISTERED | No | No | No | LOADING (on navigate) |
| LOADING | No | No | No | ENTERING (resources ready) |
| ENTERING | No | No | Yes | ACTIVE (enter animation done) |
| ACTIVE | Yes | Yes | Yes | PAUSED or EXITING |
| PAUSED | No | No | Yes | ACTIVE (when uncovered) |
| EXITING | No | No | Yes | UNLOADED (exit animation done) |
| UNLOADED | No | No | No | — |

**Main scene exception**: `main_dashboard` never enters UNLOADED. When a sub-scene is ACTIVE, the main scene is PAUSED (rendered but not updated).

### 5. Navigation API

```gdscript
SceneManager.navigate_to(scene_id: String, transition: String, context: Dictionary)
```

- `scene_id`: Key in scene_registry (e.g., "rehearsal_planning")
- `transition`: One of the 5 transition types
- `context`: Data passed to target scene's `_on_enter(context)` method; cleared after use

### 6. Scene Registry

All scenes are registered at startup:

```gdscript
var scene_registry = {
    "boot":           {"path": "res://scenes/boot.tscn",          "type": "sub"},
    "main_dashboard": {"path": "res://scenes/main_dashboard.tscn", "type": "main"},
    "rehearsal_planning": {"path": "res://scenes/rehearsal_planning.tscn", "type": "sub"},
    "narrative_event": {"path": "res://scenes/narrative_event.tscn", "type": "sub"},
    "performance":    {"path": "res://scenes/performance.tscn",    "type": "sub"},
    "settings_menu":  {"path": "res://scenes/settings_menu.tscn",  "type": "overlay"},
    "event_detail":   {"path": "res://scenes/event_detail.tscn",   "type": "overlay"},
    "unlock_notification": {"path": "res://scenes/unlock_notification.tscn", "type": "overlay"},
}
```

Dynamic registration is supported for DLC/mod scenes, but existing entries are immutable.

### Architecture

```
┌─────────────────────────────────────────┐
│           Scene Manager                 │
│  ┌─────────────┐    ┌──────────────┐   │
│  │ Scene Registry│   │ Back Stack   │   │
│  │ (id → path)  │    │ (max 5)      │   │
│  └──────┬──────┘    └──────────────┘   │
└─────────┼───────────────────────────────┘
          │ navigate_to()
          ▼
┌─────────────────────────────────────────┐
│         Transition Controller           │
│  ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐      │
│  │FADE │ │SLIDE│ │ POP │ │DISSOLVE     │
│  └─────┘ └─────┘ └─────┘ └─────┘      │
└─────────────────────────────────────────┘
          │
    ┌─────┴─────┐
    ▼           ▼
┌────────┐  ┌────────┐
│ Active │  │ Paused │
│ Scene  │  │ Scenes │
│ Stack  │  │ (Main) │
└────────┘  └────────┘
```

### Key Interfaces

```gdscript
class_name SceneManager

# Primary navigation
static func navigate_to(scene_id: String, transition_type: String = "FADE", context: Dictionary = {})
static func go_back()  # Pop back stack

# Query state
static func get_current_scene_id() -> String
static func get_current_overlay_count() -> int
static func is_transitioning() -> bool

# Scene registration (boot-time)
static func register_scene(id: String, path: String, type: String)

# Lifecycle callbacks (called by SceneManager, implemented by scenes)
signal scene_entered(context: Dictionary)
signal scene_exiting()
signal scene_paused()
signal scene_resumed()
```

### Implementation Guidelines

1. **Use Godot `CanvasLayer` for overlays** — they render above the current scene without disrupting its tree
2. **Sub-scenes are instanced as children of a "SceneContainer" node** — this keeps the main scene tree clean
3. **Transition animations use `Tween`** — not AnimationPlayer, to allow dynamic duration calculation
4. **During transition, input is blocked** — `TRANSITIONING` state rejects new navigation requests (queued, not dropped)
5. **Context data is passed to `_on_enter()`** and immediately cleared — scenes must copy what they need
6. **DISSOLVE transition requires a custom shader** — implement as a `ShaderMaterial` on a full-screen ColorRect

## Alternatives Considered

### Alternative 1: Godot's Built-in `change_scene_to_file()`

- **Description**: Use Godot's native scene change for everything
- **Pros**: Zero custom code, engine-optimized
- **Cons**: Destroys current scene (cannot pause main dashboard), no back stack, no transition customization
- **Estimated Effort**: 0×
- **Rejection Reason**: Unsuitable for layered UI; would require re-architecting the entire game to be a single scene with visibility toggles

### Alternative 2: Single Persistent Scene with Visibility

- **Description**: One massive scene with all UI panels as hidden nodes; toggle visibility for "navigation"
- **Pros**: Instant switching, no loading, simple state persistence
- **Cons**: Loads all scenes into memory simultaneously (violates 256 MB budget), no transition animations between "scenes"
- **Estimated Effort**: 0.5×
- **Rejection Reason**: Memory-unfriendly; our 8 scenes with textures would exceed mobile budget

### Alternative 3: State Machine + Page Router (Web-style)

- **Description**: Treat scenes as "pages" in a router with URL-like state
- **Pros**: Familiar pattern for web developers, deep-linking support
- **Cons**: Overkill for 8 scenes; adds unnecessary abstraction; Godot's node tree doesn't map well to URL routing
- **Estimated Effort**: 1.5×
- **Rejection Reason**: Mobile game with 8 scenes doesn't need routing complexity

## Consequences

### Positive

- Consistent transitions across all screens strengthen game identity
- Back stack provides intuitive mobile navigation
- State preservation means players never lose context when switching screens
- Overlay system enables rich UI (settings, notifications) without destroying underlying gameplay

### Negative

- DISSOLVE transition requires custom shader development
- Back stack management adds complexity to edge cases (e.g., what happens if an event fires while a sub-scene is open)
- 5 transition types mean 5 animation implementations to maintain
- Scene lifecycle states must be respected by every scene script (discipline cost)

### Neutral

- Main scene always in memory adds ~20-30 MB baseline usage
- Scene registry is static at runtime (except DLC registration)

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Back stack gets corrupted by rapid input | MEDIUM | HIGH | Debounce back button (0.3s cooldown); queue transitions |
| Scene fails to load (missing file) | LOW | HIGH | Validate all registry paths at boot; show error overlay |
| Memory leak: exited scenes not freed | MEDIUM | HIGH | Unit test: navigate through all scenes → verify only main + current in tree |
| DISSOLVE shader performance on low-end | MEDIUM | MEDIUM | Provide fallback to FADE; benchmark on Mali-G52 |
| Context data causes circular references | LOW | MEDIUM | Context is Dictionary (value types only); document "no nodes in context" |

## Performance Implications

| Metric | Before | Expected After | Budget |
|--------|--------|---------------|--------|
| Scene switch time | Instant (no transition) | 0.2–0.7s | < 1.0s |
| Memory (active scenes) | 1 scene | 1 main + 1 sub + 3 overlays max | < 100 MB |
| Frame time during transition | 0 ms | < 2 ms (Tween overhead) | < 16.6 ms |
| Back stack operations | N/A | O(1) | Negligible |

## Migration Plan

Not applicable — foundational decision.

**Adoption path**:
1. Phase 1: Implement registry + FADE transition + back stack (enables first playable)
2. Phase 2: Add SLIDE + POP transitions
3. Phase 3: Add DISSOLVE shader transition
4. Phase 4: Integrate ResourceLoader for background scene preloading

## Validation Criteria

- [ ] Unit test: navigate A → B → C → back → back → A, verify stack empties
- [ ] Unit test: push 6 sub-scenes → oldest auto-removed, stack depth stays 5
- [ ] Unit test: open 3 overlays → close 3 → return to original sub-scene
- [ ] Unit test: transition-in-progress → new navigate request → queued, not dropped
- [ ] Visual test: all 5 transitions complete within budget timing
- [ ] Memory test: navigate through all scenes → only main + current in tree
- [ ] Integration test: rapid back-tap on mobile → no crash, no state corruption

## GDD Requirements Addressed

| GDD Document | System | Requirement | How This ADR Satisfies It |
|-------------|--------|-------------|--------------------------|
| `design/gdd/scene-manager.md` | Scene Manager | "1 main + 4 sub + 3+ overlay scenes" | Defines exact scene classification and counts |
| `design/gdd/scene-manager.md` | Scene Manager | "Back stack depth 5" | Enforces max depth with overflow behavior |
| `design/gdd/scene-manager.md` | Scene Manager | "5 transition types: FADE, SLIDE, POP, DISSOLVE, INSTANT" | Specifies visual, timing, and use case for each |
| `design/gdd/scene-manager.md` | Scene Manager | "Scene lifecycle: 7 states" | Maps each state to input/update/render behavior |
| `design/gdd/scene-manager.md` | Scene Manager | "Context data passed to target scene" | Defines API and lifecycle (cleared after use) |
| `design/gdd/save-load.md` | Save/Load | "Auto-save on scene transition" | Scene Manager emits signal before transition |
| `design/gdd/resource-loader.md` | Resource Loader | "Preload scene resources before transition" | Scene Manager triggers background load |

## Related

- `design/gdd/scene-manager.md` — Full GDD with formulas and acceptance criteria
- ADR-0002 (Resource Loader) — Scene Manager depends on it for background preloading
- Future ADR: DISSOLVE shader implementation details
