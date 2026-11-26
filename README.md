# GTA VI car spawn maps

Which categories of car the game spawns in each area of Leonida, read out of GTA VI's own
`popcycle` table and the zone volumes that select it.

**[Open the interactive atlas →](https://zia-ullah-khan.github.io/gta6-car-spawn-maps/)**

Pick a vehicle category, scrub the time-of-day slider, click any area for its full mix.

## What this is

GTA VI's equivalent of GTA V's `popcycle.dat` is declared in the game's content manifest as
`platform:/levels/americas/popcycle` with `fileType POPSCHED_FILE`. It holds **137 traffic
schedules**. Each one lists 2–13 vehicle categories, and each category carries **12 one-byte
percentages** — one per time-of-day slot. Every one of the 137 × 12 columns sums to exactly 100,
which is what confirms these are percentage shares of ambient traffic rather than densities.

The schedule names are stored as hashes and only one was ever recovered, but the zone *geometry*
is not hashed. Walking the gameshape zone volumes resolved **132 of the 137 schedules to real
world polygons**, and a second pass over gameshape node names produced a gazetteer of
**199 named places, 113 of them `region_*`**. Each place takes the smallest schedule zone that
contains it, which is how zone priority works.

## The categories

`cars_sedan`, `cars_suv`, `cars_coupe`, `cars_compact`, `cars_muscle`, `cars_sport`,
`cars_pickup`, `cars_offroad`, `cars_taxi`, `motorbikes`, `vans`, `trucks`, `utility`, `haulage`,
`semi`, `business`, `misc`. Three category hashes were never cracked and keep their hex names.

## A few things it shows

| Area | Traffic mix (slot 0) |
|---|---|
| `downtown` | **taxis 24%**, sedans 17, SUVs 17 — the taxi peak of the map, against a 3% median |
| `capri` | **sports 30%**, SUVs 22, sedans 20, muscle 12 — the richest traffic on the map |
| `littlecuba` | SUVs 24, **vans 16**, taxis 10, **compacts 9**, and no sports cars at all |
| `frogswater` | **pickups 54%, off-road 30%** — the most lopsided mix in the game |
| `grayson` | **SUVs 42%** |
| `crosstown` | **sedans 40%** |

126 of the 137 schedules are flat across the day. The 11 that aren't are deliberate: taxis drop by
about half after the first three slots, `trucks` and `utility` sit at 0% at night, and the
sports/muscle share spikes at night in four city schedules.

## Files

| Path | What it is |
|---|---|
| `index.html` | the atlas — one self-contained file, all data inlined, no build step |
| `data/leonida_traffic_data.json` | the same data standalone: zone polygons, per-area mixes, world extent |
| `data/gta6_traffic_schedules.csv` | all 137 schedules × category × 12 slots |
| `data/gta6_area_traffic_mix.csv` | per named area: coordinates, governing schedule, mix |
| `data/gta6_traffic_zones_geometry.csv` | the raw zone polygons |
| `METHOD.md` | how the files were found and the formats decoded |

### JSON shape

```
{ extent: [minX, minY, maxX, maxY],
  areas: [{ n, x, y, si, sh, za, sp, p: [[x,y]…], m: [[category, 12 slots | one number]…] }],
  zones: [{ si, sh, a, p, m }] }
```

`n` internal name · `si`/`sh` schedule index and hash · `za` governing zone in km² ·
`sp` 1 for a specific zone, 0 for a regional fallback · `p` polygon. A mix value is a single
number when it is flat across the day and a 12-element array when it varies.

World coordinates span x −11135…3884, y −8646…10066. North is up; one unit is about a metre.

## Caveats

- 141 of the 199 places land inside a specific zone under 5 km²; those assignments are solid. The
  other 58 fall through to a broad county or highway zone. The atlas marks these `S` and `R`.
- 5 of the 137 schedules have no polygon yet.
- The 12 slots are the file's own time-of-day columns. The file does not record which clock hours
  they cover, so they are numbered rather than named.
- Everything here is read out of the shipped data. Nothing is inferred from gameplay or from
  outside sources.

## Licence and ownership

This repository contains derived data and analysis only — no game assets, code or binaries. All
names, place names and game data are the property of Rockstar Games. Published for research and
documentation; it is not affiliated with or endorsed by Rockstar Games or Take-Two Interactive.
