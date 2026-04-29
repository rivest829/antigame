# ADR-0002: Resource Loader — 3-Tier Cache with Async Loading and Mobile Memory Budget

## Status

Accepted

## Date

2026-04-24

## Last Verified

2026-04-24

## Decision Makers

User + godot-specialist agent + systems-designer agent

## Summary

The resource loader must eliminate loading stalls on mobile devices with limited memory. We decided on a 3-tier cache architecture (Resident / LRU / No-Cache), three loading modes (Preload / Background / On-Demand), and a 64 MB mobile memory budget with automatic eviction. This design trades memory for responsiveness while respecting the 256 MB total ceiling.

## Engine Compatibility

| Field | Value |
|-------|-------|
| **Engine** | Godot 4.6 |
| **Domain** | Core / Asset Management |
| **Knowledge Risk** | LOW — `ResourceLoader`, `ImageTexture`, and Godot's reference-counting are stable APIs within training data |
| **References Consulted** | `docs/engine-reference/godot/VERSION.md` |
| **Post-Cutoff APIs Used** | None |
| **Verification Required** | Test LRU eviction under memory pressure on low-end Android (2 GB RAM devices); verify `OS.get_static_memory_usage()` accuracy |

## ADR Dependencies

| Field | Value |
|-------|-------|
| **Depends On** | ADR-0001 (Save/Load — resource loader may need to persist cache metadata Post-MVP) |
| **Enables** | ADR-0003 (Scene Manager — scene transitions depend on preloaded resources), all UI and audio systems |
| **Blocks** | Any story loading textures, audio, or scenes |
| **Ordering Note** | Must be implemented before any system loads assets dynamically. UI systems can hardcode `preload()` initially, but must migrate to ResourceLoader before polish phase. |

## Context

### Problem Statement

Mobile players abandon apps that show loading screens or pop-in. Godot's default `preload()` blocks the main thread and `load()` can stall if the file is large. We need a system that loads resources asynchronously, caches them intelligently, and never exceeds the 256 MB mobile memory budget.

### Current State

No resource loading system exists. All assets would be loaded via Godot's built-in `preload()` or `load()`, both synchronous.

### Constraints

- **Platform**: iOS / Android — aggressive OS memory management, apps killed without warning
- **Memory ceiling**: 256 MB total (from `technical-preferences.md`)
- **Asset types**: Textures (~20-30), Audio (~10-15), Scenes (~8), Data (~5), Fonts (~2-3), Shaders (~2-3)
- **Target**: 60 fps = 16.6 ms frame budget; loading must not block frames

### Requirements

- Asynchronous loading with progress callbacks
- Zero visible pop-in for preloaded assets
- Graceful degradation (placeholder → real asset) for on-demand loads
- Memory budget enforcement with automatic eviction
- No more than 3 concurrent loads to prevent I/O thrashing

## Decision

### 1. Cache Architecture: 3 Tiers

```
L1: Resident Cache
  - Content: Core data, fonts, shaders, current scene resources
  - Strategy: Never auto-evicted
  - Management: Explicit `pin()` / `unpin()` by owning system

L2: LRU Cache
  - Content: Textures, audio
  - Strategy: Least Recently Used eviction
  - Limits: texture_cache_size_mb + audio_cache_size_mb (configurable)

L3: No Cache
  - Content: Scene files (.tscn)
  - Strategy: Loaded on demand, released when scene is unloaded
```

**Rationale**: Three tiers match three distinct asset lifecycles. Resident assets are always needed; LRU assets are reused but disposable; scenes are single-use.

### 2. Loading Modes: 3 Types

| Mode | Trigger | Blocking | Use Case |
|------|---------|----------|----------|
| **PRELOAD** | Game startup | Yes (with progress bar) | Core data, fonts, common UI |
| **BACKGROUND** | Anticipated need | No | Preloading next scene's assets while player reads text |
| **ONDEMAND** | Player action | No (with placeholder) | Texture displayed for first time |

### 3. Memory Budget: 64 MB for Cache

From the 256 MB total mobile budget:
- 64 MB allocated to ResourceLoader (textures + audio caches)
- 192 MB reserved for game logic, Godot engine overhead, and OS buffers

**Eviction triggers**:
- **Warning** (192 MB = 75%): Clear LRU cache
- **Critical** (224 MB = 87.5%): Force release all non-resident resources

### 4. Concurrent Loading: Max 3

Only 3 resources load simultaneously. Excess requests enter a priority queue sorted by:

```
sort_score = priority_value × 1000 - time_in_queue_ms × 0.001
```

Priority values: CRITICAL(0) > HIGH(1) > NORMAL(2) > LOW(3) > IDLE(4)

### 5. Placeholder System

While loading, show a stylistic placeholder that matches the grunge aesthetic:
- **Texture**: Dirty-yellow block (#8B7355) with hand-drawn "loading..." text
- **Audio**: Silence (no error)
- **Scene**: Extended transition animation
- **Data**: Hardcoded defaults, refresh when loaded

Placeholders crossfade to real assets over 0.2 seconds when loading completes.

### Architecture

```
                    ┌─────────────────┐
                    │  Request Queue   │
                    │ (priority sorted)│
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
        ┌─────────┐    ┌─────────┐    ┌─────────┐
        │ Loader 1 │    │ Loader 2 │    │ Loader 3 │  (max 3 concurrent)
        └────┬────┘    └────┬────┘    └────┬────┘
             │              │              │
             └──────────────┼──────────────┘
                            ▼
                    ┌─────────────┐
                    │  Loaded     │
                    │  Resource   │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌─────────┐  ┌─────────┐  ┌─────────┐
        │ Resident │  │  LRU    │  │ No Cache│
        │  Cache   │  │  Cache  │  │ (scenes)│
        │ (pinned) │  │(evictable)│  │(release)│
        └─────────┘  └─────────┘  └─────────┘
```

### Key Interfaces

```gdscript
class_name ResourceLoaderManager

# Request async load
static func load_request(path: String, priority: LoadPriority, callback: Callable)

# Check cache
static func is_cached(path: String) -> bool
static func get_cached(path: String) -> Resource

# Batch preload
static func preload_batch(paths: Array[String], progress_callback: Callable)

# Cache management
static func pin(path: String)           # Move to Resident
static func unpin(path: String)         # Return to LRU or evict
static func clear_lru_cache()
static func clear_all_caches()          # Emergency: keep only Resident

# Memory monitoring
static func get_memory_usage_mb() -> float
static func get_cache_usage_mb() -> float
```

### Implementation Guidelines

1. **Use Godot's `ResourceLoader.load_threaded_request()`** for true async loading on supported platforms; fall back to coroutine-based async on others
2. **LRU eviction must check reference count** — never evict a resource currently in use
3. **Memory check runs every 30 seconds** via `_on_memory_check()` timer; also triggered immediately after any load completes
4. **Placeholder assets are preloaded at boot** and pinned in Resident cache so they are always available
5. **Scene resources are never cached** — scenes are instantiated, then the PackedScene is freed

## Alternatives Considered

### Alternative 1: Single Global Cache

- **Description**: One cache for all resources with a single size limit
- **Pros**: Simpler implementation, one eviction policy
- **Cons**: Cannot distinguish "always needed" from "temporarily needed"; critical assets could be evicted
- **Estimated Effort**: 0.7×
- **Rejection Reason**: Risk of evicting fonts or core data; tiered cache is safer for mobile

### Alternative 2: Godot's Built-in `ResourceLoader` Cache

- **Description**: Rely on Godot's internal resource cache (`ResourceLoader.has_cached()`)
- **Pros**: Zero custom code, engine-managed
- **Cons**: No memory budget control, no LRU eviction, no async loading, no priority queue
- **Estimated Effort**: 0× (use built-in)
- **Rejection Reason**: Godot's cache grows unbounded; on mobile this leads to OOM kills

### Alternative 3: Addressables-style Asset Bundles

- **Description**: Group assets into bundles, load/unload bundles as units
- **Pros**: Clean lifecycle management, natural fit for level-based games
- **Cons**: Overkill for our scene count (~8 scenes); adds bundle build complexity
- **Estimated Effort**: 2×
- **Rejection Reason**: Our scene count is small; per-resource caching is simpler and sufficient

## Consequences

### Positive

- Players never see loading stalls for preloaded assets
- Memory budget enforcement prevents OOM crashes on low-end devices
- Placeholder system maintains aesthetic consistency during load
- Priority queue ensures critical assets load first

### Negative

- LRU eviction may cause re-load of recently-used assets if memory is tight
- Placeholder system requires art assets (dirty-yellow blocks, hand-drawn text)
- 30-second memory polling is reactive, not proactive
- Reference-count checking adds overhead to every cache access

### Neutral

- Total code complexity is moderate (~500-800 lines GDScript)
- Requires coordination with Scene Manager for background preloading

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| `OS.get_static_memory_usage()` inaccurate on Android | MEDIUM | MEDIUM | Add safety margin; test on physical devices |
| Texture pop-in on low-end devices | MEDIUM | HIGH | Increase preload scope for critical textures; use lower-res placeholders |
| LRU thrashing (load→evict→reload loop) | LOW | MEDIUM | Monitor cache hit rate; increase cache size if hit rate < 80% |
| Background loading interferes with frame time | LOW | HIGH | Cap background loader to 1 concurrent; defer to idle frames |

## Performance Implications

| Metric | Before | Expected After | Budget |
|--------|--------|---------------|--------|
| Frame time during load | 16ms+ stall | 0ms (async) | < 16.6ms |
| Memory usage | Unbounded | < 64 MB cache + Resident | < 256 MB total |
| Scene switch time | 1-3s | 0.3-0.7s (preloaded) | < 1s perceived |
| Cache hit rate | N/A | > 85% (target) | > 80% minimum |

## Migration Plan

Not applicable — foundational decision.

**Adoption path for existing code**:
1. Phase 1: All `preload()` calls remain; ResourceLoader wraps them
2. Phase 2: Migrate texture/audio loads to `load_request()`
3. Phase 3: Scene Manager integrates background preloading
4. Phase 4: Remove direct `preload()` usage entirely

## Validation Criteria

- [ ] Unit test: load 10 textures concurrently → max 3 active, rest queued
- [ ] Unit test: memory usage exceeds 192 MB → LRU cache auto-clears
- [ ] Unit test: pinned resource not evicted when LRU cleared
- [ ] Integration test: switch scenes 20 times → no OOM, cache hit rate > 80%
- [ ] Performance test: background loading during gameplay → frame time < 17ms
- [ ] Visual test: placeholder → asset transition is smooth, no flicker

## GDD Requirements Addressed

| GDD Document | System | Requirement | How This ADR Satisfies It |
|-------------|--------|-------------|--------------------------|
| `design/gdd/resource-loader.md` | Resource Loader | "3-tier cache (Resident/LRU/No-Cache)" | Defines exact tier structure and eviction policies |
| `design/gdd/resource-loader.md` | Resource Loader | "3 load modes: PRELOAD, BACKGROUND, ONDEMAND" | Specifies trigger conditions and blocking behavior |
| `design/gdd/resource-loader.md` | Resource Loader | "64 MB cache budget, 256 MB total" | Allocates budget and defines memory thresholds |
| `design/gdd/resource-loader.md` | Resource Loader | "Max 3 concurrent loads" | Enforces concurrency limit in loader pool |
| `design/gdd/scene-manager.md` | Scene Manager | "Scene resources preloaded before transition" | Background loading mode enables this |
| `design/gdd/audio-system.md` | Audio System | "Audio resources loaded asynchronously" | ONDEMAND mode for audio with silent placeholder |

## Related

- `design/gdd/resource-loader.md` — Full GDD with formulas and acceptance criteria
- ADR-0003 (Scene Manager) — Depends on ResourceLoader for background scene preloading
- Future ADR: Texture compression format decision (affects cache size calculations)
