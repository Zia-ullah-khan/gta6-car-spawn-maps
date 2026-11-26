# GTA VI — fastest vehicles and where a fixed one spawns

Built for route planning. Every number is read out of the shipped data; the locations are the game's own description text, quoted.

## Where this comes from

| What | Source | Nature |
|---|---|---|
| Performance figures | `common.rpf` payload `0x612000`, `CVehiclePerformanceMaps` | plain XML, 251 vehicles |
| Cross-check | `common.rpf` payload `0x616000`, `CVehicleBaseRatingsAndEstimatesMap` | plain XML, independent road/offroad estimates |
| Spawn descriptions | the game's text table | quoted verbatim |
| Spawn areas | `GRID_SML_L_VEH_<model>_UNIQUE_<area>_…` tokens in the progress dataset | 158 unique-vehicle catalog entries |

The two performance files are independent and agree on the ordering, which is the main reason to trust them.

251 vehicles have performance entries; 203 are road-capable (they have a real braking distance and turn radius — aircraft and boats have neither). 103 have a fixed spawn: 71 with a written location and 62 with a catalog area.

**Units.** `topSpeed` is unlabelled in the file. The values behave like metres per second — 76.6 for the fastest hypercar is 276 km/h — so the km/h column below is `× 3.6`. Treat the absolute numbers as an assumption; the *ordering* does not depend on it.

## The pick-up list — fastest road cars with a known fixed spawn

This is the table to route against: no traffic-spawn luck, the car is always there.

| # | Model | top (m/s) | km/h | 0–60 | Where the game says it is |
|---|---|---|---|---|---|
| 1 | `thrax` | 76.6 | 276 | 2.43 | Custom Truffade Thrax belonging to Andrew Node, found at his luxury residence on Starfish Island. |
| 2 | `emerus` | 72.3 | 260 | 2.92 | Custom, race-spec Progen Emerus with a unique livery. Discovered in a garage in Capri. |
| 3 | `vitello` | 71.4 | 257 | 2.75 | Unique Pegassi Vitello with a custom finish, discovered at Dante Fresh's mansion on Starfish Island. |
| 4 | `tempesta` | 71.0 | 256 | 2.78 | Eye-catching Pegassi Tempesta, discovered outside a restaurant in Ocean Beach. |
| 5 | `furia` | 69.2 | 249 | 2.78 | Striking leopard print Piccia Furia, discovered at a luxury residence on Starfish Island. |
| 6 | `italirsx` | 67.7 | 244 | 2.60 | catalog area: crosstown |
| 7 | `zorrusso` | 66.7 | 240 | 3.52 | One-of-a-kind Granforma Zorrusso modified to racing specifications, found at a luxury residence in Peacock Estates. |
| 8 | `coquette4` | 63.9 | 230 | 3.68 | catalog area: stockyard |
| 9 | `schutzen` | 61.6 | 222 | 3.97 | Benefactor Schutzen owned by Vice City rapper DWNPLY. Found outside a Pay 'n' Spray in Stockyard. |
| 10 | `jugular` | 61.0 | 220 | 3.68 | Race-ready Ocelot Jugular found at a wealthy residence in Belville. |
| 11 | `vigero3` | 61.0 | 219 | 4.58 | catalog area: keylento |
| 12 | `cypher` | 60.0 | 216 | 4.07 | Unique Übermacht Cypher modded for street racing with an exposed engine. Discovered in a garage in Crosstown. |
| 13 | `paragon` | 58.9 | 212 | 3.97 | Ascot Paragon with a striking rose gold finish. Discovered outside a huge property in Waning Sands. |
| 14 | `cometretro` | 57.9 | 209 | 4.55 | catalog area: capri |
| 15 | `coquette` | 57.6 | 208 | 4.47 | Promotional Invetero Coquette, found on display outside the police HQ in Capri. |
| 16 | `banshee` | 57.6 | 207 | 5.35 | Unusual Bravado Banshee, lifted and reinforced for dirt track racing by an Emma Gary. Discovered in a junkyard in Hamlet. |
| 17 | `toros` | 57.2 | 206 | 4.05 | Luxury Pegassi Toros discovered at a residence in Rialto Islands. Unmistakable, thanks to its golden finish. |
| 18 | `lectro` | 57.0 | 205 | 4.70 | Bespoke Principe Lectro with a custom livery and racing mods, discovered at Emilio's Auto Center in Rockridge. |
| 19 | `double` | 56.7 | 204 | 3.77 | catalog area: laperle |
| 20 | `coquette2` | 55.9 | 201 | 6.05 | catalog area: crosstown |
| 21 | `dominator7` | 55.2 | 199 | 5.35 | catalog area: ambrosia;kellycounty;rockridge |
| 22 | `dominator` | 54.6 | 196 | 5.73 | Vapid Dominator ASP optimized for racing with a patriotic paint job. Discovered at Ambrosia Raceway Park. |
| 23 | `fcr` | 54.6 | 196 | 4.53 | Übermacht FCR 1000, modified with a classic aviation vibe. Found at a flight school by VCIA. |
| 24 | `stallion` | 53.6 | 193 | 5.58 | Converted '68 Classique Stallion, found in a garage in Waning Sands. Modified with an exposed engine and supercharger. |
| 25 | `hermes` | 53.3 | 192 | 5.78 | '50 Albany Hermes with custom flame decals. Discovered in a shipping container on Draper Island. |

**Starfish Island is the single best stop on the map.** Three of the top five — `thrax`, `vitello` and `furia` — sit at luxury residences there, and `thrax` is the fastest vehicle in the game that isn't an aircraft.

## Fastest overall, fixed spawn or not

For when you'll take whatever you can find.

| Model | top (m/s) | km/h | 0–60 | brake 60–0 | road? | fixed spawn? |
|---|---|---|---|---|---|---|
| `lazer` | 116.0 | 417 | 1.80 | 0.0 | no | no |
| `thrax` | 76.6 | 276 | 2.43 | 16.8 | yes | yes |
| `emerus` | 72.3 | 260 | 2.92 | 17.1 | yes | yes |
| `pulsebike` | 71.8 | 258 | 4.13 | 17.7 | yes | no |
| `vitello` | 71.4 | 257 | 2.75 | 18.8 | yes | yes |
| `tempesta` | 71.0 | 256 | 2.78 | 17.8 | yes | yes |
| `gauntlet4` | 69.5 | 250 | 3.32 | 21.0 | yes | no |
| `furia` | 69.2 | 249 | 2.78 | 19.4 | yes | yes |
| `comet7` | 68.2 | 245 | 3.12 | 19.5 | yes | no |
| `valkyrie` | 67.7 | 244 | 4.98 | 0.0 | no | no |
| `italirsx` | 67.7 | 244 | 2.60 | 18.1 | yes | yes |
| `buzzard` | 66.9 | 241 | 7.23 | 0.0 | no | no |
| `zorrusso` | 66.7 | 240 | 3.52 | 18.2 | yes | yes |
| `mallard` | 64.3 | 232 | 0.73 | 0.0 | no | yes |
| `infernus2` | 64.2 | 231 | 3.93 | 20.1 | yes | no |
| `neon` | 64.0 | 230 | 3.02 | 20.9 | yes | no |
| `coquette4` | 63.9 | 230 | 3.68 | 19.3 | yes | yes |
| `mambagt` | 62.6 | 226 | 3.95 | 23.6 | yes | no |
| `sfb1000` | 61.9 | 223 | 3.42 | 17.5 | yes | no |
| `raiden` | 61.8 | 222 | 3.77 | 22.6 | yes | no |

Entries with `road? = no` are aircraft and boats: `lazer` at 116.0 is the jet, `valkyrie` and `buzzard` are helicopters. They have no braking distance or turn radius in the data.

## Best acceleration

Top speed rarely matters on a short city leg. Sorted by 0–60 among road cars:

| Model | 0–60 | top (m/s) | fixed spawn? |
|---|---|---|---|
| `thrax` | 2.43 | 76.6 | yes |
| `italirsx` | 2.60 | 67.7 | yes |
| `vitello` | 2.75 | 71.4 | yes |
| `tempesta` | 2.78 | 71.0 | yes |
| `furia` | 2.78 | 69.2 | yes |
| `emerus` | 2.92 | 72.3 | yes |
| `neon` | 3.02 | 64.0 | no |
| `comet7` | 3.12 | 68.2 | no |
| `gauntlet4` | 3.32 | 69.5 | no |
| `sfb1000` | 3.42 | 61.9 | no |
| `zorrusso` | 3.52 | 66.7 | yes |
| `coquette4` | 3.68 | 63.9 | yes |

## Fixed spawns grouped by area

If the route already passes through an area, this is the best car waiting there.

| Area | Best car there | top (m/s) | others |
|---|---|---|---|
| `airport` | `fcr` | 54.6 | — |
| `ambrosia` | `dominator7` | 55.2 | `gauntlet5`, `penumbraff`, `picador`, `caracara4x4`, `zombieb` |
| `bayside` | `cypher` | 60.0 | `dominatorclassic` |
| `belville` | `baller2` | 52.2 | `zion2` |
| `capri` | `cometretro` | 57.9 | `penumbra`, `bobcatxl`, `caracara4x4`, `sungrazer` |
| `crosstown` | `italirsx` | 67.7 | `cypher`, `coquette2` |
| `downtown` | `cavalcade3` | 49.4 | — |
| `frogswater` | `picador` | 51.7 | — |
| `gellhorn` | `cliffhanger` | 46.3 | `bandito` |
| `grassrivers` | `sovereign107` | 51.3 | `bison900`, `enduro`, `sanchez`, `boxville` |
| `grayson` | `gauntlet5` | 53.2 | — |
| `hamlet` | `banshee` | 57.6 | `bandito` |
| `kellycounty` | `emerus` | 72.3 | `dominator7`, `elegy`, `sovereign107`, `manchez` |
| `keylento` | `vigero3` | 61.0 | `ruiner`, `akane`, `woodland`, `hustler`, `yougaxl` |
| `laperle` | `double` | 56.7 | — |
| `leaflinks` | `toros` | 57.2 | `jubilee` |
| `leonardcounty` | `sanchez` | 40.8 | `declasse500` |
| `lummoxcounty` | `zombieb` | 44.8 | — |
| `millport` | `yougaxl` | 35.1 | — |
| `peacockbay` | `cavalcade3` | 49.4 | — |
| `portgellhorn` | `blista2` | 47.8 | `yosemite1500`, `caracara4x4`, `marshall` |
| `rockridge` | `dominator7` | 55.2 | — |
| `southside` | `esskey` | 52.7 | — |
| `stockyard` | `coquette4` | 63.9 | — |
| `tequesta` | `tempesta` | 71.0 | `schafters` |
| `waningsands` | `paragon` | 58.9 | `stallion` |
| `washingtonbeach` | `baller8` | 53.2 | `zombieb` |
| `watsonbay` | `phoenix` | 46.1 | `youga2`, `blazer4`, `journey` |

## Caveats

- `topSpeed` units are inferred, not stated in the file. Rankings are unaffected.
- The area column comes from the catalog token (`…_UNIQUE_<area>_…`) and is only as precise as the area name. The description column is the game's own text and is more specific.
- A description tells you the place in words, not coordinates. The exact parking spot would need the `VEHICLEGEN_OVERVIEW_FILE`, which I have not decoded.
- 80 written locations were matched to models by looking for the model name inside the description. A handful of models share a word with ordinary dialogue; obvious false matches were filtered by requiring a "found/discovered … in/at/outside" pattern, but spot-check anything surprising.
- Availability over the story is not modelled here. A car existing at a spot does not mean it is reachable at that point in a run.

Full table including non-road vehicles: `gta6_vehicle_performance.csv`.
