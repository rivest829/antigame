# ADR-0001: Save/Load Format — JSON with Atomic Write and Version Migration Chain

## Status

Accepted

## Date

2026-04-24

## Last Verified

2026-04-24

## Decision Makers

User + game-designer agent + systems-designer agent

## Summary

The save/load system must persist player progress on mobile devices with zero data-loss tolerance. We decided on JSON as the serialization format with atomic write-via-rename, 3 rolling backups, and a chainable version-migration strategy. This balances human-debuggability, corruption resilience, and forward-compatibility without adding external dependencies.

## Engine Compatibility

| Field | Value |
|-------|-------|
| **Engine** | Godot 4.6 |
| **Domain** | Core / Persistence |
| **Knowledge Risk** | LOW — `FileAccess`, `JSON.parse`, and OS atomic-rename behavior are stable Godot APIs well within training data |
| **References Consulted** | `docs/engine-reference/godot/VERSION.md` |
| **Post-Cutoff APIs Used** | None |
| **Verification Required** | Test atomic rename on iOS Simulator and Android emulator under low-battery forced-termination scenarios |

## ADR Dependencies

| Field | Value |
|-------|-------|
| **Depends On** | None |
| **Enables** | ADR-0003 (Scene Manager — boot→dashboard flow depends on save-load), all gameplay-system stories |
| **Blocks** | Any story implementing persistent state (character stats, song proficiency, unlocks, events) |
| **Ordering Note** | Must be implemented before any system writes mutable state. Can be stubbed with in-memory-only storage for early prototyping, but atomic-write + migration must be in place before first external playtest. |

## Context

### Problem Statement

A放置 game's core promise is "set it and forget it." If a player loses progress because the app was killed mid-save, the game is fundamentally broken. We must choose a persistence format and strategy that guarantees data integrity on mobile platforms where app termination is frequent and unpredictable.

### Current State

No save system exists. The project is in pre-production, so this is a green-field decision.

### Constraints

- **Platform**: iOS / Android — file system is sandboxed, no direct SQLite unless bundled
- **Engine**: Godot 4.6 — built-in `FileAccess` and `JSON` classes available, no external DB driver by default
- **Team size**: Indie — no dedicated backend engineer for cloud sync in MVP
- **Debugging**: Need ability to inspect save files during development and QA
- **Size**: MVP save estimated at 5–50 KB — trivial for any format

### Requirements

- Zero corruption on app termination during save (atomic write)
- Automatic recovery from corrupted saves without player data loss
- Forward compatibility: old saves load in new game versions
- Human-readable for debugging and support
- No external dependencies in MVP

## Decision

### 1. Serialization Format: JSON

Use Godot's built-in `JSON.stringify()` / `JSON.parse()` for all save data. The top-level structure follows the L1/L2分层 defined in `design/gdd/save-load.md`.

**Rationale**: JSON is human-readable (critical for debugging), natively supported by Godot, and our data size (<50 KB) makes parsing speed irrelevant.

### 2. Atomic Write: Temp-File + Rename

All writes follow this sequence:

1. Serialize data to a temporary file (`save.json.tmp`)
2. `FileAccess.flush()` to ensure OS-level persistence
3. Atomically rename `save.json.tmp` → `save.json`

This guarantees that `save.json` is never in a partially-written state. If the app dies during step 1 or 2, the temp file is incomplete but `save.json` remains untouched. If the app dies during step 3, the OS guarantees the rename is atomic.

### 3. Rolling Backup Strategy: 3 Backups

On every successful save:
- `save.json.backup2` → `save.json.backup3` (discard old backup3)
- `save.json.backup1` → `save.json.backup2`
- `save.json` → `save.json.backup1`
- Then perform the atomic write to `save.json`

This creates a sliding window of 3 historical backups. Recovery attempts backup1 first, then backup2, then backup3.

### 4. Version Migration: Chainable Scripts

Each save carries `save_format_version`. Migration is performed by chained scripts:

```gdscript
func migrate(data: Dictionary, from_version: int, to_version: int) -> Dictionary:
    for v in range(from_version, to_version):
        data = call("migrate_v%d_to_v%d" % [v, v + 1], data)
    return data
```

**Rules**:
- Only "N → N+1" migrations are authored
- Cross-version jumps are handled by chaining
- Migration scripts only restructure data, never re-balance gameplay values
- If any migration step fails, fall back to backup recovery

### 5. Integrity Validation

On load, compute:

```
integrity_score = Σ(field_validity_i) / total_required_fields
```

- Pass threshold: `>= 0.8` (80%)
- Failed fields are filled with defaults, game continues with a non-blocking warning
- If integrity score < 0.8 OR JSON parse fails → attempt backup recovery

### Architecture

```
┌─────────────┐     save request      ┌─────────────┐
│  Any System  │ ───────────────────► │ SaveManager  │
│  (calls      │                      │  (singleton)  │
│   save_game) │                      └──────┬──────┘
└─────────────┘                             │
                                            ▼
                                    ┌─────────────┐
                                    │ Serialize   │
                                    │ to Dict     │
                                    └──────┬──────┘
                                           ▼
                                    ┌─────────────┐
                                    │ JSON.stringify
                                    │ write to    │
                                    │ save.json.tmp
                                    └──────┬──────┘
                                           ▼
                                    ┌─────────────┐
                                    │ FileAccess  │
                                    │ .flush()    │
                                    └──────┬──────┘
                                           ▼
                                    ┌─────────────┐
                                    │ Shift backups│
                                    │ (1→2, 2→3)   │
                                    └──────┬──────┘
                                           ▼
                                    ┌─────────────┐
                                    │ Atomic rename│
                                    │ .tmp → .json │
                                    └─────────────┘
```

### Key Interfaces

```gdscript
class_name SaveManager

# Save current game state (called by systems)
static func save_game(data: Dictionary) -> SaveResult

# Load game state (called at boot)
static func load_game() -> LoadResult

# Internal: attempt recovery from backups
static func _attempt_backup_recovery() -> LoadResult

# Internal: migrate save data across versions
static func _migrate(data: Dictionary, from: int, to: int) -> Dictionary

# Internal: validate integrity
static func _validate(data: Dictionary) -> float  # returns integrity_score
```

### Implementation Guidelines

1. **Use Godot's `UserCache` or `UserData` directory** via `OS.get_user_data_dir()` — do NOT hardcode paths
2. **Backup shifting must be synchronous** and complete BEFORE the atomic rename
3. **Migration scripts are idempotent** — running them twice on the same data produces the same result
4. **Never block the main thread** for more than 1 frame during save; JSON stringify of <50 KB is negligible but verify on low-end Android
5. **Log every save operation** with timestamp and trigger reason (auto/manual/boot) for debugging

## Alternatives Considered

### Alternative 1: SQLite

- **Description**: Use Godot's SQLite module or bundled driver for structured storage
- **Pros**: ACID guarantees, structured queries, handles schema migration natively
- **Cons**: Adds external dependency (not in Godot core), overkill for <50 KB of data, not human-readable for debugging
- **Estimated Effort**: 2× (module integration + schema management)
- **Rejection Reason**: Unnecessary complexity for MVP data size; JSON + atomic write provides sufficient durability

### Alternative 2: Binary Format (Godot's `FileStore` or custom binary)

- **Description**: Use `FileAccess.store_var()` or custom binary format
- **Pros**: Smaller file size, faster I/O, tamper-resistant (obscurity)
- **Cons**: Not human-readable, version migration is harder (binary offsets), Godot's `store_var` format can change between engine versions
- **Estimated Effort**: 1.5× (custom serialization + migration tooling)
- **Rejection Reason**: Debuggability is critical for indie development; binary format would slow down bug investigation

### Alternative 3: Cloud-First (Firebase / Play Games)

- **Description**: Store saves in cloud backend with local cache
- **Pros**: Cross-device sync, server-side backup
- **Cons**: Requires backend infrastructure, network dependency, privacy compliance overhead, external service cost
- **Estimated Effort**: 5× (backend + auth + sync logic + conflict resolution)
- **Rejection Reason**: Explicitly out of MVP scope; may be Post-MVP extension

## Consequences

### Positive

- Human-readable saves enable rapid debugging and QA data manipulation
- Atomic write + 3 backups makes data loss virtually impossible in MVP scenarios
- Chainable migration allows infinite forward compatibility without rewrite
- Zero external dependencies keeps build simple

### Negative

- JSON is not tamper-resistant — players can edit saves (accepted risk for MVP; encryption is Post-MVP)
- No cross-device sync (accepted — Post-MVP)
- Backup rotation writes 4 files per save (1 temp + 3 shifts + 1 final) = higher I/O than single-file; acceptable for <50 KB and mobile flash storage

### Neutral

- Save file size is larger than binary (~2-3×) but irrelevant at <50 KB scale
- Godot's `JSON` class does not support circular references — our data structure is a tree, so this is fine

## Risks

| Risk | Probability | Impact | Mitigation |
|------|------------|--------|-----------|
| Godot `FileAccess` rename not atomic on specific Android OEM | LOW | HIGH | Test on top 5 Android devices; if found, use `OS.execute("mv")` fallback |
| Player manually deletes all backup files | LOW | HIGH | Documented limitation; iOS sandbox prevents casual file access |
| Migration script bug corrupts valid old save | MEDIUM | HIGH | Each migration script has unit test with fixture data; never modify gameplay values |
| Save file grows unexpectedly large | LOW | MEDIUM | Monitor save size in analytics; add size cap warning in development builds |

## Performance Implications

| Metric | Before | Expected After | Budget |
|--------|--------|---------------|--------|
| Save write time | N/A | <10ms (JSON stringify + file I/O of <50 KB) | <16ms (1 frame) |
| Load time | N/A | <20ms (file read + JSON parse + validation) | <100ms (perceived instant) |
| Memory (save data) | N/A | <1 MB (in-memory Dictionary) | Negligible vs 256 MB total |
| Disk writes per save | N/A | 4 file operations | Acceptable on flash storage |

## Migration Plan

Not applicable — this is a foundational decision for a new system.

## Validation Criteria

- [ ] Unit test: save → kill app mid-write → load returns previous valid state
- [ ] Unit test: corrupt save.json → automatic recovery from backup1 succeeds
- [ ] Unit test: all backups corrupt → graceful error with "new game" option
- [ ] Unit test: save version 1 → migrate to version 3 via chained scripts → data structure correct
- [ ] Unit test: integrity score 0.85 → loads with defaults for missing fields
- [ ] Integration test: full game session (play → background → kill → relaunch) → progress intact

## GDD Requirements Addressed

| GDD Document | System | Requirement | How This ADR Satisfies It |
|-------------|--------|-------------|--------------------------|
| `design/gdd/save-load.md` | Save/Load | "JSON format with version migration, 3 rolling backups, atomic write" | Defines exact format, backup strategy, and atomic write mechanism |
| `design/gdd/save-load.md` | Save/Load | "integrity_score >= 0.8 allows load with defaults" | Specifies validation formula and fallback behavior |
| `design/gdd/save-load.md` | Save/Load | "Version migration scripts only restructure, never re-balance" | Enforced as implementation guideline |
| `design/gdd/character-stats.md` | Character Stats | "Character state must persist across sessions" | JSON structure includes character skills and status values |
| `design/gdd/time-offline-revenue.md` | Time/Offline | "last_leave_timestamp must be persisted for offline calculation" | Timestamp stored in L1 core progress |

## Related

- `design/gdd/save-load.md` — Full GDD with formulas, edge cases, and acceptance criteria
- Future ADR: Cloud sync architecture (Post-MVP)
- Future ADR: Save encryption (Post-MVP)
