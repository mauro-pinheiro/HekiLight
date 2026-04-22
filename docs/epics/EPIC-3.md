# EPIC-3: Settings, Polish & Player Customization

**Status:** Done
**Version range:** v0.1.4 → v0.2.1
**Commits:** `daadbf9` → `cadc6b5`

## Goal

Make HekiLight fully self-configurable in-game without requiring the player to edit Lua or reload for every preference change. Cover font customization, color picking, visibility fine-tuning, and a collapsible settings panel that scales gracefully as the option count grows.

## Business Value

An overlay addon that requires Lua edits to change icon size or keybind color will not be adopted outside a developer audience. CurseForge players expect to configure addons entirely in-game. This epic makes HekiLight a first-class citizen alongside WeakAuras and Details in terms of in-game configurability — every visual and behavioral property is point-and-click adjustable.

## Stories

| ID | Title | Status |
|----|-------|--------|
| 3.1 | Collapsible Settings Panel | Done |
| 3.2 | Keybind Font/Color + Always Show Mode | Done |
| 3.3 | Defensive Floater (built and reverted) | Reverted |

## Key Technical Decisions

- `OptionsSliderTemplate` is deprecated in Midnight 12.0 — all sliders built manually with `BackdropTemplate` + `FontString` value display
- `Settings.RegisterCanvasLayoutCategory` / `RegisterCanvasLayoutSubcategory` for panel registration — the only supported path in Midnight
- Color picker uses Blizzard's `ColorPickerFrame`; no third-party color picker library
- Scrollbar auto-hide was attempted and reverted — no stable hook in Midnight 12.0's scroll frame API
- **ADR-6:** `C_CooldownViewer` is not safe to read from addon Lua during combat — the defensive floater was permanently reverted

## Acceptance Criteria (Epic-Level)

- [x] All settings configurable from Game Menu → Interface → AddOns → HekiLight
- [x] Settings panel uses collapsible sections, all collapsed by default
- [x] Icon size, scale, slot count, spacing, poll rate configurable via sliders
- [x] Keybind font size, color, outline style, anchor corner all configurable
- [x] Color picker available for keybind text color
- [x] Show mode toggle: "always" (whenever RA active) vs "active" (combat/target required)
- [x] All hard-stop toggles exposed in settings (dead, vehicle, cinematic)
- [x] Minimap button on/off; minimap angle adjustable by drag
- [x] All settings persist account-wide via `HekiLightDB`
- [x] All settings also accessible via `/hkl` slash commands without reload

## Shipped to CurseForge

v0.1.0 was the first CurseForge release (`2761bfb`). CurseForge release workflow added at `aa9a183`.
