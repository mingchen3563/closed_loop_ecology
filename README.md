# Closed Loop Ecology

A [Space Haven](https://store.steampowered.com/app/979110/Space_Haven/) mod that **closes the food / carbon / chemicals cycle** so peaceful playthroughs can sustain themselves without trading or hunting corpses, with stoichiometry that respects real-world chemistry.

## The problem in vanilla

Tracing the food cycle through the game data exposes three structural breaks:

1. **Composter is 87% lossy.** 1 biomass → 0.10 water + 0.017 fertilizer + 0.017 carbon. Total recovered: 0.13 of the original 1.0 mass units; the rest just vanishes from accounting. Real biochar retains 30-70%.

2. **Carbon mineable but gated at Mining skill 7.** Same tier as Noble Metals, while crew default skills are 1-3. Real carbonaceous chondrites are *the* most common asteroid type and physically soft — should be earliest tier.

3. **Chemicals → Fertilizer only from Rot.** Rot only from corpses. Peaceful builders can't make fertilizer → eventually starve unless they trade or hunt.

For a 5-crew ship in vanilla, the food loop runs at **-0.158 fertilizer/day** indefinitely.

## What this mod changes

Six surgical patches against `library/haven`:

| # | Change | Per-cycle / per-unit effect |
|---|---|---|
| 1 | Carbon mining lvl 7 → **2**, Raw Chemicals 6 → **3** | Early-game crew can mine carbon |
| 2 | Composter carbon output × **6** (recipe 2467 + 7 food-compost variants) | 1 biomass → 0.10 carbon (matches real biochar yields) |
| 3 | CO2 scrubber = exact mathematical inverse of carbon burner | Round-trip C ↔ CO2 is mass-conservative, no free-carbon exploit |
| 4 | **New recipe: Industrial Chemicals Synthesis** | Interactive at Chemical Refinery: 1 Carbon + 3 Water → 3 Chemicals |
| 5 | **New recipe: Biomass Pyrolysis** | Interactive at Chemical Refinery: 2 Biomass → 1 Carbon |

All new recipes run at existing vanilla stations — no new visual assets, no new buildings.

## Gas byproducts (intentionally omitted)

Physically, the new recipes should emit gas byproducts:
- Industrial Chemicals Synthesis (Fischer-Tropsch + electrolysis) → free O2 to atmosphere
- Biomass Pyrolysis → CO2 to atmosphere (50% of biomass mass)

The mod doesn't model these because:

1. **Interactive recipes with mixed item+gas outputs break the conditional UI.** Space Haven's "produce if storage < X" stop-condition can't reconcile gas outputs (no inventory), so the recipe stalls in a permanent condition-mismatch.

2. **The two-stage workaround** (transient inventory items auto-vented at other stations) is functional but pollutes vanilla stations like the CO2 Scrubber and Life Support with mod recipes. Stacking with other ecology mods becomes messy.

The gas byproducts are abstracted as part of station atmospheric exchange. A future version may re-introduce them via a dedicated mod station (Pyrolysis Reactor / Electrolysis Cell) that hosts both the interactive recipe and its non-interactive vent without touching vanilla stations.

## After install (5-crew steady state)

| Resource | Vanilla daily net | With mod |
|---|---|---|
| Carbon production | +0.10 (compost dead-end) | **+0.9 to +1.4** (composter ×6 + mining + pyrolysis) |
| Fertilizer | **-0.158** ⚠ (deficit) | **±0** ✓ (closed via chemicals synthesis) |
| Chemicals demand | 0.16/day (must import) | 0 (self-synthesized from carbon + water) |
| Water | -1.15 | -1.6 (synthesis uses extra water) |
| Atmospheric O2 | from O2 generators only | unchanged (vanilla O2 gen handles it) |
| Atmospheric CO2 | from crew breath only | unchanged (vanilla scrubber handles it) |

Translation: **the food/fertilizer loop closes**. Water demand goes up slightly (ice mining still required). Atmosphere is more dynamic — extra O2 and CO2 enter through industrial processes, with the scrubber/Life-Support pair handling the equilibrium.

## Real-world references

| Game change | Real-world basis |
|---|---|
| Easier carbon mining | [Carbonaceous chondrite asteroids](https://en.wikipedia.org/wiki/Carbonaceous_chondrite) — ~75% of all asteroids by type, soft and friable |
| Composter carbon retention | [Biochar](https://en.wikipedia.org/wiki/Biochar) production typically retains 30-70% of input carbon |
| CO2 scrubber = burner inverse | Stoichiometric mass conservation — no perpetual-motion exploits |
| Carbon + Water → Chemicals | [Fischer-Tropsch synthesis](https://en.wikipedia.org/wiki/Fischer%E2%80%93Tropsch_process) (industrial since 1925); water electrolysis for the oxygen byproduct |
| Biomass → Carbon | Pyrolysis / charcoal production (5000+ years of human history); ~50% carbon retention as biochar, the rest released as CO2 |

## Mismatches to reality (honest accounting)

Using the in-game CO2 generator (recipe 65) as the anchor unit-ratio (1 game-C : 20,000 game-CO2), several reactions still violate strict mass conservation:

| Reaction | Mismatch vs game-self-consistent | Note |
|---|---|---|
| Composter biomass → carbon | 2.5× too low (mod gives 0.10, should be ~0.25) | Even after ×6 boost; further buff breaks balance |
| **Crop growth** | **8× violation** — plants absorb less CO2 than they output biomass | Vanilla design choice; tweaking breaks economy |
| Burner / scrubber round-trip | 0× ✓ | Made exactly symmetric by mod |
| Pyrolysis carbon retention | 2× over-yield | Compensates for not modeling CO2 byproduct (now restored via §5) |

The biggest residual mismatch is the *crop* itself — plants in Space Haven produce biomass faster than CO2 absorption would allow. That's vanilla design and outside the mod's scope. Everything else has been brought within a factor of ~2× of stoichiometric reality.

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

> ⚠ Don't put the mod folder in the Steam Workshop directory — Steam re-validates and wipes anything that isn't a registered workshop item.

## Activating in an existing save

Existing stations on a save snapshot their `<features>` config at build time. After installing the mod you **must dismantle and rebuild**:

- **Chemical Refinery** — to pick up the new Pyrolysis and Industrial Chemicals Synthesis recipes
- **CO2 Scrubber** — to pick up the rebalanced (burner-symmetric) carbon recovery rate
- **Composter** — to pick up the boosted carbon retention

Newly-built stations on the same save pick up the changes automatically. This is a save-state quirk, not a mod bug.

## Compatibility

- Stacks cleanly with [drones_plus](https://github.com/mingchen3563/drones_plus) — that mod tunes bot stations; this one tunes ecology. No overlap.
- Other mods that touch composter recipes (2467, 2472, etc.) overlap — last-loaded `AttributeSet` wins.
- Mods adding new recipes to dispatcher 963 stack fine (NodeAdd is additive).
- Touches mid 922 (CO2 Scrubber) `<needs>` and `<products>` on recipe 66 only.

## Project layout

```
closed_loop_ecology/
├── info.xml                          # mod metadata, modid 7700002
├── patches/
│   └── haven_ecology.xml             # 6 sections, ~20 operations
└── README.md
```

### Element/recipe IDs introduced

| ID | Kind | Name |
|---|---|---|
| 7700001 | Process recipe | Industrial Chemicals Synthesis (interactive, on Chemical Refinery) |
| 7700010 | Process recipe | Biomass Pyrolysis (interactive, on Chemical Refinery) |

All under the `77000xx` namespace anchored on modid `7700002`. IDs 7700020-7700099 reserved for the future Pyrolysis Reactor station and its gas-vent recipes.

## License

MIT.

## Credits

- [spacehaven-modloader](https://github.com/Spacehaven-modding-tools/spacehaven-modloader) team for the patch system
- Bugbyte for Space Haven
- Real-world chemistry & space life-support research for design grounding
