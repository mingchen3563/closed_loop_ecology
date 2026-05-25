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
| 4 | **New recipe: Industrial Chemicals Synthesis** | Interactive at Chemical Refinery: 1 Carbon + 3 Water → 3 Chemicals + 2 Oxygen Tanks (items). Tanks auto-vent to 100 atmospheric O2 at Life Support |
| 5 | **New recipe: Biomass Pyrolysis** | Interactive at Chemical Refinery: 2 Biomass → 1 Carbon + 1 Char Residue (item). Residue auto-vents to 50 CO2 at CO2 Scrubber |
| 6 | Two-stage gas-byproduct pattern | Transient inventory items bridge interactive recipes (which can't expose gas outputs to the conditional UI) and non-interactive atmospheric vents |

All new recipes run at existing vanilla stations — no new visual assets, no new buildings.

## The two-stage gas pattern explained

Space Haven's "produce if storage < X" UI on interactive recipes doesn't reconcile gas outputs (Oxygen, CO2 — they have no inventory). So an interactive recipe with mixed item+gas outputs stalls in a permanent condition-mismatch.

This mod's workaround: use **transient inventory items** that get auto-vented at appropriate stations:

```
Stage A (interactive, items only)              Stage B (non-interactive, gas vent)

Chemical Refinery                              Life Support stations
  Crew runs Chemicals Synthesis                  Auto-runs when oxygen tank in storage
  1 C + 3 H2O → 3 Chemicals + 2 O2 Tank          1 O2 Tank → 50 atmospheric O2

Chemical Refinery                              CO2 Scrubber
  Crew runs Pyrolysis                            Auto-runs when char residue in storage
  2 Biomass → 1 Carbon + 1 Char Residue          1 Char Residue → 50 atmospheric CO2
```

Crew get a clean conditional UI on the interactive side; atmosphere gets the gas release as a separate auto-cycle. Items represent compressed gas / unprocessed exhaust waiting to be released.

## After install (5-crew steady state)

| Resource | Vanilla daily net | With mod |
|---|---|---|
| Carbon production | +0.10 (compost dead-end) | **+0.9 to +1.4** (composter ×6 + mining + pyrolysis) |
| Fertilizer | **-0.158** ⚠ (deficit) | **±0** ✓ (closed via chemicals synthesis) |
| Chemicals demand | 0.16/day (must import) | 0 (self-synthesized from carbon + water) |
| Water | -1.15 | -1.6 (synthesis uses extra water) |
| Atmospheric O2 | from O2 generators only | + bonus 100/cycle from chemicals synthesis |
| Atmospheric CO2 | from crew breath only | + 50/cycle from pyrolysis (recaptured by scrubber) |

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

- **Chemical Refinery** (any existing) — to pick up the new Pyrolysis and Industrial Chemicals Synthesis recipes
- **CO2 Scrubber** — to pick up the Char Residue auto-vent
- **Life Support** (any of the three variants: mid 861, 924, 61) — to pick up the Oxygen Tank auto-vent
- **Composter** — to pick up the boosted carbon retention

Newly-built stations on the same save pick up the changes automatically. This is a save-state quirk, not a mod bug.

## Compatibility

- Stacks cleanly with [drones_plus](https://github.com/mingchen3563/drones_plus) — that mod tunes bot stations; this one tunes ecology. No overlap.
- Other mods that touch composter recipes (2467, 2472, etc.) overlap — last-loaded `AttributeSet` wins.
- Mods adding new recipes to dispatcher 963 stack fine (NodeAdd is additive).
- Mods modifying mid 922 (CO2 Scrubber), 861/924/61 (Life Support) `<produces>` blocks may overlap.

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
| 7700001 | Process recipe | Industrial Chemicals Synthesis (interactive) |
| 7700010 | Process recipe | Biomass Pyrolysis (interactive) |
| 7700020 | Elementary item | Char Residue (transient inventory item) |
| 7700021 | Elementary item | Oxygen Tank (transient inventory item) |
| 7700022 | Process recipe | Char Residue → CO2 vent (non-interactive) |
| 7700023 | Process recipe | Oxygen Tank → O2 vent (non-interactive) |

All under the `77000xx` namespace anchored on modid `7700002`.

## License

MIT.

## Credits

- [spacehaven-modloader](https://github.com/Spacehaven-modding-tools/spacehaven-modloader) team for the patch system
- Bugbyte for Space Haven
- Real-world chemistry & space life-support research for design grounding
