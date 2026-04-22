# EPIC-4: Diagnostics, Proc Alert & Correctness

**Status:** Done (ongoing bugfix runway)
**Version range:** v0.3.0 → current
**Commits:** `48e6914` → `272c3fe`

## Goal

Give the addon the diagnostic infrastructure needed to identify and fix wrong-icon bugs without a live debugger; add a dedicated proc-alert icon slot for visual urgency; fix talent override icon resolution (Defile, etc.); stop log buffer flooding; and eliminate unnecessary CPU use between fights.

## Business Value

Without a persistent log, diagnosing "why did it show the wrong icon?" requires reproducing the bug live with a developer attached — impossible during a raid. The session log lets a player type `/hkl log 20` after a wipe and hand the output to a developer. This makes the feedback loop from bug report to fix a matter of hours, not sessions.

The talent override fix directly addresses the #1 reported correctness issue for Blood DK (and likely other talent-heavy specs): the icon showing the base spell instead of what the player will actually cast.

## Stories

| ID | Title | Status |
|----|-------|--------|
| 4.1 | Proc Alert, Edit Mode, Session Log & Locale | Done |
| 4.2 | Talent Override Icon Resolution & Log Spam Fix | Done |
| 4.3 | Poll Loop Tied to Combat State | Done |

## Key Technical Decisions

- **ADR-8:** Poll loop lifetime now tied to `PLAYER_REGEN_ENABLED/DISABLED` — zero OnUpdate overhead between fights
- **ADR-9:** Talent override resolution via `GetActionInfo(realSlotID)` in pcall; `OVERRIDE` DLog tag records every substitution
- **ADR-12:** Session log ring buffer in SavedVariables — 500 entries, `table.remove(t,1)` on overflow; previous session saved to `lastSessionLog` on `/reload`
- Change-detection guards (`lastLogSuggID`, `lastSlotSpellID`, `lastSkippedAlertID`) prevent high-frequency events from flooding the 500-entry buffer
- `keybindCache[spellID]` stores last-known-good keybind to avoid redundant `GetBindingKey` calls per frame

## Acceptance Criteria (Epic-Level)

- [x] Proc-alert icon slot appears when the suggested spell has an active overlay glow
- [x] Proc-alert frame independently positionable; lockable to main display
- [x] Edit Mode integration: display frame appears in WoW's layout editor
- [x] `DLog(tag, msg)` logs timestamped entries to `HekiLightDB.sessionLog`
- [x] Ring buffer capped at 500 entries; oldest removed on overflow
- [x] `/hkl log [N]` prints last N entries from current or previous session log
- [x] Talent-overridden spells (Defile, etc.) display the correct override icon and name
- [x] Every override substitution logged with `OVERRIDE` tag
- [x] `SKIPPED` log entry fires once per proc activation, not every poll tick
- [x] Poll loop stops on `PLAYER_REGEN_ENABLED`, restarts on `PLAYER_REGEN_DISABLED`
- [x] `Locale.lua` loaded first via TOC; identity-fallback metatable for untranslated keys
- [x] ptBR translation block in place

## Open Bugs (at Epic close)

| Bug | Details | Priority |
|-----|---------|----------|
| Secondary slot dedup broken by override | `primaryID` is override ID; `GetRotationSpells()` returns base IDs; `sid ~= primaryID` misses match | High |
| `IsActionInRange` not pcall-guarded | `C_ActionBar.IsActionInRange(rslot)` at line ~1743 lacks pcall — could taint if called on an unexpected slot type | Medium |
| `GetRealSlot` for secondary spells | Secondary spells from `GetRotationSpells()` are base IDs; `FindSpellActionButtons(baseID)` may fail if slot indexed under override ID | Medium |

## Next Suggested Epic

**EPIC-5: Correctness Hardening** — fix the three open bugs above; add override-aware deduplication; make the secondary slot pipeline fully consistent with the primary slot's override resolution.
