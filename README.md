# Closed Loop Ecology

A [Space Haven](https://store.steampowered.com/app/979110/Space_Haven/) mod that **closes the food / carbon / chemicals cycle** so peaceful playthroughs can sustain themselves without trading or hunting corpses.

## The problem in vanilla

Tracing the food cycle through the game data reveals three structural breaks:

1. **Composter is 87% lossy.** 1 biomass → 0.10 water + 0.017 fertilizer + 0.017 carbon. Total recovered: 0.13 units, the rest vanishes from the accounting. Real biochar retains 30-70% of input carbon.

2. **Carbon is mineable but gated at Mining level 7** — the same skill tier as Noble Metals. Crew default skill is 1-3. Carbonaceous chondrite asteroids should be early-game accessible.

3. **Chemicals (fertilizer precursor) only comes from Rot.** Rot only comes from corpses. Peaceful builds can't make chemicals → can't make fertilizer → eventually starve.

For a 5-crew ship, this manifests as a steady -0.158 fertilizer/day deficit that can only be closed by trading Chemicals or killing things for biomass.

## What this mod changes

Five surgical patches against `library/haven`:

| # | Change | Effect |
|---|---|---|
| 1 | Carbon mining lvl 7 → **2** | Early-game crew can mine carbon asteroids |
| 1 | Raw Chemicals mining lvl 6 → **3** | Similar industrial-grade material gating |
| 2 | Composter carbon output × **6** | 1 biomass → 0.10 carbon (matches biochar yields) |
| 3 | CO2 scrubber = exact inverse of carbon burner | Lossless round-trip C↔CO2 (no exploit) |
| 4 | **New recipe: Industrial Chemicals Synthesis (two-stage)** | 1 Carbon + 3 Water → 3 Chemicals + 2 Oxygen Tanks (item) → auto-vents to 100 O2 at Life Support |
| 5 | **New recipe: Biomass Pyrolysis (two-stage)** | 2 Biomass → 1 Carbon + 1 Char Residue (item) → auto-vents to 50 CO2 at CO2 scrubber |

Both new recipes run at the existing Chemical Refinery (mid 939) — no new station to build.

## After install (5-crew steady state)

| Resource | Vanilla daily net | With mod |
|---|---|---|
| Carbon production | +0.10 | **+0.9 to +1.4** |
| Fertilizer | **-0.158** ⚠ | **±0** ✓ |
| Chemicals demand | 0.16/day (import) | 0 (self-synthesized) |
| Water | -1.15 | -1.6 (more for synthesis) |
| Carbon dead-end | accumulates | actively consumed |

Translation: **the food loop closes, water demand goes up slightly** (industrial process uses water).

## Real-world references

| Game change | Real-world basis |
|---|---|
| Easier carbon mining | [Carbonaceous chondrite asteroids](https://en.wikipedia.org/wiki/Carbonaceous_chondrite) — ~75% of all asteroids by type, soft and friable |
| Composter carbon retention | [Biochar](https://en.wikipedia.org/wiki/Biochar) production typically retains 30-70% of input carbon |
| Atmospheric CO2 → Carbon | [CO2 electrolysis](https://en.wikipedia.org/wiki/Electrolysis_of_carbon_dioxide) — mirrors the burner's stoichiometry so the C↔CO2 round-trip is mass-conservative (no free carbon from looping the burner+scrubber) |
| Carbon + Water → Chemicals | [Fischer-Tropsch synthesis](https://en.wikipedia.org/wiki/Fischer%E2%80%93Tropsch_process) (industrial since 1925) |
| Biomass → Carbon | Pyrolysis / charcoal production (5000+ years of human history) |

## Requirements

- **Space Haven** v1.0.2 (patches target stable element IDs)
- **[spacehaven-modloader](https://github.com/Spacehaven-modding-tools/spacehaven-modloader)** v0.12.1+

## Install

### Option A — clone directly into the game's mods folder

```sh
cd "<SpaceHaven>/mods"
git clone https://github.com/mingchen3563/closed_loop_ecology.git
```

### Option B — workspace + junction (Windows, recommended for dev)

```powershell
git clone https://github.com/mingchen3563/closed_loop_ecology.git C:\path\to\closed_loop_ecology
cmd /c 'mklink /J "<SpaceHaven>\mods\closed_loop_ecology" "C:\path\to\closed_loop_ecology"'
```

Then launch `spacehaven-modloader`, enable **Closed Loop Ecology**, apply, launch the game.

> ⚠ Don't put the mod folder in the Steam Workshop directory — Steam wipes anything that isn't a registered workshop item.

## Compatibility

- Stacks cleanly with [drones_plus](https://github.com/mingchen3563/drones_plus) — that mod tunes bot stations; this one tunes the ecology.
- Other mods that touch the same composter recipes (2467, 2472, etc.) may overlap — last-loaded wins for `AttributeSet`.
- Mods that add new recipes to dispatcher 963 stack fine (NodeAdd is additive).

## Known gotchas

**Existing composters and Chemical Refineries on a save don't update when you install the mod.** Space Haven snapshots facility config at build time. Dismantle and rebuild affected stations to pick up the new recipes and rates.

This is a save-state quirk, not a mod bug — see [SPACEHAVEN_MODDING.md](https://github.com/mingchen3563/closed_loop_ecology/blob/main/SPACEHAVEN_MODDING.md) §9 for details.

## Project layout

```
closed_loop_ecology/
├── info.xml                          # mod metadata, modid 7700002
├── patches/
│   └── haven_ecology.xml             # all 5 sections, ~15 operations
└── README.md
```

## License

MIT.

## Credits

- [spacehaven-modloader](https://github.com/Spacehaven-modding-tools/spacehaven-modloader) team for the patch system
- Bugbyte for Space Haven
- Real-world chemistry & space science for design grounding
