# Astrophage Crisis — Mod Skeleton (v0.1.0, Stellaris 4.4.*)

A mid/late-game crisis inspired by Project Hail Mary. A star-eating organism appears in one system and spreads along hyperlanes, dimming stars in stages until systems die. Empires race a Situation to nullify or harness it — cooperatively via a Galactic Community resolution, or solo as isolationists.

## Status: SKELETON

Every file exists and shows intent, but this has **not been validated in-game**. Expect `error.log` complaints on first launch — that is the normal workflow. Files marked TODO need their schema checked against the matching vanilla file (your best documentation is the game's own files in `Steam/steamapps/common/Stellaris/common/`).

## Installation (local dev mod)

1. Copy the `astrophage_crisis` folder to:
   `Documents/Paradox Interactive/Stellaris/mod/astrophage_crisis`
2. Create a file `astrophage_crisis.mod` in `Documents/Paradox Interactive/Stellaris/mod/` containing:

   ```
   name="Astrophage Crisis"
   path="mod/astrophage_crisis"
   supported_version="4.4.*"
   ```

3. Open the Paradox launcher → Playsets → add "Astrophage Crisis". (Note: the launcher auto-disables mods after major patches — re-enable it if it vanishes.)

## Testing loop

1. Launch, start a new game (small galaxy, few AIs = fast iteration).
2. **First stop, always:** `Documents/Paradox Interactive/Stellaris/logs/error.log`. Fix errors top-down; one syntax error can silently kill a whole file.
3. In-game console (`~` key):
   - `event astrophage.1` — force the outbreak now
   - `observe` — become observer and watch it spread
   - `event astrophage.20` — test the discovery popup
   - `research_technology tech_astrophage_harvesting` — test the harvest path
4. Reload scripts without restarting: `reload events`, `reload localization` (not everything hot-reloads; `common/` changes usually need a restart).

## File map

| File | Purpose | Confidence |
|---|---|---|
| `events/astrophage_events.txt` | Spawn, spread pulse, dimming stages, discovery, Taumoeba twist, endings | Medium — verify event scheduling pattern |
| `common/on_actions/` | Hooks crisis into game start; discovery hooks TODO | High |
| `common/scripted_effects/` | Infest / advance dimming / cleanse | Medium |
| `common/scripted_triggers/` | Valid seed & spread targets | High |
| `common/static_modifiers/` | Dimming stage penalties | Medium — verify 4.4 modifier keys |
| `common/deposits/` | Farmable bloom deposit | Medium |
| `common/strategic_resources/` | `sr_astrophage` | Medium |
| `common/situations/` | The research race, 5 stages, 3 approaches | **Low — copy vanilla schema first** |
| `common/resolutions/` | GC Joint Research Program | Low — verify category key |
| `common/technology/` | 3 event-granted techs | High |
| `common/component_templates/` | Spin drive thruster | Medium |
| `localisation/english/` | All text | High (see BOM warning below) |

## ⚠ Localization gotcha

Stellaris `.yml` localization files **must be saved as UTF-8 with BOM**. If your text shows as raw keys in-game (e.g. `astrophage.20.name`), the BOM is missing. In VS Code: click "UTF-8" in the status bar → "Save with Encoding" → "UTF-8 with BOM". The file in this skeleton already has the BOM.

## Development roadmap

**Phase 1 — Make it spread (MVP):** Get outbreak + spread pulse + dimming modifiers working and visible in observer mode. Everything else can wait.

**Phase 2 — Make it hurt:** Deposit swap on stars, colony penalties, system death state, discovery event wiring via on_actions.

**Phase 3 — Make it winnable:** Situation (copy a vanilla situation file wholesale and reshape it), tech grants, containment flag halting spread in your borders, both endings.

**Phase 4 — Make it political:** GC resolution, AI weights, isolationist balance (2–3× solo cost, exclusive Harness rights), Taumoeba twist.

**Phase 5 — Polish:** Custom icons, event art, ambient system visuals, dimmed star class, balance passes, AI response tuning (this is where the "AI ignores Gray Tempest" problem gets solved — give AI empires free progress or scripted evacuation).

## Design decisions already baked in

- Outbreak starts in **unclaimed space**, never a home system.
- Spread chance rolls per infested system per month → more infestation = faster spread = organic pressure on non-cooperators.
- Dimming is a **visible countdown** (1yr / 3yr / 6yr / 10yr stages), not an instant kill.
- Dead systems stay dead even after the cure — consequences persist.
- Harness path: slower, but yields `sr_astrophage`, farming deposits, and the Spin Drive.

## Reference reading

- Stellaris wiki → Modding portal (event modding, situations, on_actions pages)
- Vanilla files: `common/situations/`, `common/resolutions/`, `events/nanite_events.txt` (closest vanilla analog to this crisis)
- `logs/script_documentation/` folder — the game dumps every valid trigger/effect/modifier here. Ground truth for 4.4.
