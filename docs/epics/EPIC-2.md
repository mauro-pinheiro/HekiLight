# EPIC-2: Multi-Suggestion Queue

**Status:** Done
**Version range:** v0.1.0 → v0.1.3
**Commits:** `f09711c` → `8743a99`

## Goal

Extend the single-icon display into a configurable strip of up to 5 suggestion slots, showing the rotation queue ahead — with correct cooldown greying, spell deduplication, per-character ignored spell filtering, and keybind labels on every slot.

## Business Value

Experienced players plan keystrokes several GCDs ahead. Seeing only the next ability provides no planning horizon. A strip of 3–5 upcoming spells lets players pre-position fingers, anticipate resource spenders, and time cooldowns without reading a separate rotation guide. This is the feature that makes the overlay useful for skilled play, not just learning.

## Stories

| ID | Title | Status |
|----|-------|--------|
| 2.1 | Multi-Suggestion Icon Strip | Done |
| 2.2 | Ignored Spells & Per-Character Storage | Done |
| 2.3 | Keybind Labels on Secondary Slots | Done |

## Key Technical Decisions

- **ADR-5:** Secret-number pcall trick for cooldown detection — `duration == 0` comparison inside pcall; error means on-CD. `GetSpellBaseCooldown` (>1500ms) gates the real-CD tracking table to exclude GCD-only spells
- **ADR-10:** Pre-allocated `queueCache` — module-level table repopulated in-place; zero per-frame allocation from the queue pipeline
- **ADR-11:** `ignoredSpells` and `classDefaultsApplied` live in `HekiLightDBChar` (per-character), not `HekiLightDB` (account-wide)

## Acceptance Criteria (Epic-Level)

- [x] Up to 5 secondary icon slots configurable via `/hkl suggestions <N>` and settings panel
- [x] Secondary slots show correct textures from `C_AssistedCombat.GetRotationSpells()`
- [x] Spells on real cooldown (>1500ms base CD) rendered greyed at lower priority
- [x] GCD does not trigger false-positive greying on secondary slots
- [x] Primary suggestion is deduplicated from the secondary strip
- [x] Keybind text overlaid on each secondary slot icon
- [x] Per-character ignored spells list filters secondary suggestions
- [x] Class maintenance spells (Revive Pet, Raise Ally, etc.) auto-ignored on first load
- [x] Zero per-frame table allocation from the queue pipeline

## Open Bug (at Epic close)

Secondary slot deduplication uses `sid ~= primaryID` where `primaryID` may be an override spellID (e.g., Defile `152280`), but `GetRotationSpells()` returns base IDs (Death and Decay `43265`). The comparison misses the match — the same ability can appear in both primary and a secondary slot simultaneously.
