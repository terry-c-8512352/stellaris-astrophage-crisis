# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Stellaris mod (Paradox script, targeting Stellaris 4.4.*): a mid/late-game crisis inspired by Project Hail Mary. A star-eating organism spreads along hyperlanes, dimming stars in stages, while empires race a Situation to nullify or harness it. Currently a **skeleton** — every file exists and shows intent, but it has not been validated in-game. Many blocks are marked TODO pending schema verification against vanilla files.

## Development workflow (no build/lint/tests)

There is no compiler or test suite. The workflow is:

1. Copy/symlink `astrophage_crisis/` into `Documents/Paradox Interactive/Stellaris/mod/astrophage_crisis` and put `astrophage_crisis.mod` in `Documents/Paradox Interactive/Stellaris/mod/`. Enable via the Paradox launcher playset.
2. Launch a new game (small galaxy, few AIs) — then **always check `Documents/Paradox Interactive/Stellaris/logs/error.log` first**. Fix errors top-down; one syntax error can silently kill an entire file.
3. Test via the in-game console (`~`):
   - `event astrophage.1` — force the outbreak
   - `observe` — watch the spread as observer
   - `event astrophage.20` — discovery popup
   - `research_technology tech_astrophage_harvesting` — harvest path
4. `reload events` / `reload localization` hot-reload; changes under `common/` usually need a full restart.

Ground truth for valid triggers/effects/modifiers in 4.4 is the game's own dump at `Documents/.../logs/script_documentation/` and vanilla files in `Steam/steamapps/common/Stellaris/common/`. Do not trust 3.x-era tutorials — 4.x renamed pop/job modifiers and reworked pops to be workforce-based.

## Architecture: the crisis flow

The mod is a chain of events, scripted effects/triggers, and flags spanning several files. The core loop:

1. `common/on_actions/` hooks `on_game_start` → fires `astrophage.0` (setup) → schedules `astrophage.1` (outbreak).
2. `astrophage.1` picks a seed system using scripted trigger `is_valid_astrophage_seed` (unclaimed space, never home systems) and applies scripted effect `astrophage_infest_system`, then starts `astrophage.2`.
3. `astrophage.2` is a **self-rescheduling 30-day pulse** (`event = { id = astrophage.2 days = 30 }` inside its own `immediate`). Each pulse, every infested system advances its dimming clock (`astrophage_advance_dimming`) and rolls a 25% chance to infect a hyperlane neighbor (`is_valid_astrophage_target`). If this self-scheduling pattern fails validation, the documented fallback is `on_yearly_pulse` (see the TODO in `common/on_actions/`).
4. Dimming progress is a per-system variable (`astrophage_dim_progress`, +1/pulse). Thresholds 12/36/72/120 fire events `astrophage.10–13`, which swap static modifiers `astrophage_dimming_1/2/3` and set star flags `astrophage_dim_1/2/3` / `astrophage_dead_system`.
5. Discovery (`astrophage.20`) starts the per-empire Situation `astrophage_research_situation` (5 stages, 3 approaches: fund/observe/harness). Stage 4 fires the Taumoeba twist (`astrophage.40`); completion fires an ending — `astrophage.50` (Nullified, galaxy-wide cleanse) or `astrophage.51` (Harnessed, exclusive `sr_astrophage` farming).
6. The GC resolution (`resolution_astrophage_research_program`) sets `astrophage_gc_program_active`, which the situation's `monthly_progress` reads to double member progress.

### Cross-file contracts (change these consistently everywhere)

- **Global flags:** `astrophage_mod_active`, `astrophage_outbreak_started`, `astrophage_gc_alerted`, `astrophage_gc_program_active`, `astrophage_crisis_resolved`
- **Star flags:** `astrophage_infested`, `astrophage_dim_0/1/2/3`, `astrophage_dead_system`
- **Country flags:** `astrophage_discovered` (planned: `astrophage_containment_active`)
- **Event ID plan** (documented at the top of `events/astrophage_events.txt`): 0=setup, 1=outbreak, 2=spread pulse, 10–13=dimming stages, 20=discovery, 30=GC alert, 40=Taumoeba, 50/51=endings
- **Tech chain:** `tech_astrophage_containment` → `tech_astrophage_harvesting` (gates the `d_astrophage_bloom` deposit's production trigger) → `tech_astrophage_spin_drive` (prerequisite of the `ASTROPHAGE_SPIN_DRIVE_1` component). All are event-granted (`weight = 0`).
- Cleansing (`astrophage_cleanse_system`) intentionally does **not** revive dead systems — consequences persist post-cure.

### Design decisions already baked in (don't undo casually)

- Outbreak starts in unclaimed space, never a home system.
- Spread chance rolls per infested system per month — more infestation = faster spread.
- Dimming is a visible countdown (~1/3/6/10 year stages), not an instant kill.
- Isolationist balance: full solo cost, but exclusive Harness rights.

Roadmap phases 1–5 (spread → hurt → winnable → political → polish) are in `astrophage_crisis/README.md`.

## Gotchas and conventions

- **Localisation files must be UTF-8 with BOM** (`localisation/english/*.yml`, note British spelling of the directory). If in-game text shows raw keys, the BOM was lost. Keys use format `l_english:` then one space indent, and `key: "text"`.
- The two `.mod` files serve different roles: root `astrophage_crisis.mod` is the launcher pointer; `astrophage_crisis/descriptor.mod` carries version/tags. Keep `supported_version` in sync in both.
- Paradox script uses tabs for indentation and `#` comments; files under `common/` are keyed definitions, `events/` holds `event`/`country_event` blocks with a `namespace` line.
- Confidence levels per file are tracked in the README's file map — `common/situations/` is the least certain (copy a vanilla situation file wholesale and reshape it before trusting the current schema).
- The closest vanilla analog for the crisis mechanics is `events/nanite_events.txt` (Gray Goo).
