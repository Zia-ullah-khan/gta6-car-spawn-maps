# GTA VI - ambient traffic mix by area, as the shipped files define it

Source: `GTA6.Xbox.v1.0.2.1-CYBERLEEK`. Every figure below is read out of the game data. Nothing is inferred from gameplay or from outside sources.

## Where the data lives

| What | File | How it was identified |
|---|---|---|
| Manifest that names the system | `common.rpf` payload @ `0x129000` | plain XML `CDataFileMgr__ContentDef`; declares `platform:/levels/americas/popcycle` with `fileType POPSCHED_FILE`, plus `ai/popzonespawnerinfo` (`POPZONE_SPAWNER_INFO_FILE`) |
| **The traffic-mix table** | `levels.rpf` @ `0x11BB2B8000`, 2,147,136 B, PSO (`PSIN`) | the only file in the entire dump whose PSO schema contains a field named `PopGroup`; its root field is `schedules` |
| Zone volumes that select a schedule | `levels.rpf` gameshape files around `0x0000F90xxxxx` | the only files that reference the 137 schedule name hashes; schema has `zoneName`, `Shape`, `Components`, `Height` |
| Parked-car / parking-zone density | `levels.rpf` @ `0x11BB00F000` | schema fields `DensityDatas`, `DensityName`, `ParkingZones`, `ZoneTags` |
| Police response per region | `data.rpf` @ `0x1AF3D000` | schema fields `DispatchRequests`, `DispatchType`, `WantedResponse`, `VehicleSets`, `RegionName` |
| Per-area liveries | `common.rpf` @ `0x449000`, `CPaletteIndexedRecipeTintsArray` | plain XML |

RPF9 name tables are encrypted in this build, so files were located structurally - a zlib-payload scan of all 76 GB of `levels.rpf` (904,991 payloads, 6,982 PSO files) - and then identified by brute-forcing the joaat hashes of their PSO schema field names. Field names hash in PascalCase (`Name`, `Flags`, `PopGroup`); content values hash in lowercase snake_case (`cars_muscle`, `portgellhorn`).

## Shape of the table

`popcycle` holds **137 traffic schedules**. Each lists 2-13 vehicle categories, and each category carries **12 one-byte percentages** - one per time-of-day slot.

All 137 x 12 columns sum to exactly **100**. That is what proves these are percentage shares of ambient traffic rather than densities or weights.

Groups per schedule: 2 groups in 1 schedules, 3 groups in 3 schedules, 4 groups in 4 schedules, 5 groups in 4 schedules, 6 groups in 3 schedules, 7 groups in 7 schedules, 8 groups in 7 schedules, 9 groups in 20 schedules, 10 groups in 24 schedules, 11 groups in 30 schedules, 12 groups in 28 schedules, 13 groups in 6 schedules

## The vehicle categories

23 distinct category hashes appear in the table; 17 names recovered, 6 still unnamed.

| Category | Schedules using it (of 137) | min share | median | max share |
|---|---|---|---|---|
| `cars_sedan` | 122 | 8% | 22% | 40% |
| `cars_suv` | 122 | 6% | 23% | 42% |
| `vans` | 120 | 1% | 8% | 23% |
| `cars_muscle` | 116 | 1% | 3% | 23% |
| `cars_pickup` | 116 | 1% | 15% | 54% |
| `cars_coupe` | 114 | 1% | 4% | 18% |
| `motorbikes` | 107 | 1% | 4% | 14% |
| `trucks` | 93 | 1% | 4% | 80% |
| `utility` | 85 | 1% | 2% | 31% |
| `cars_compact` | 78 | 1% | 2% | 11% |
| `cars_taxi` | 76 | 1% | 3% | 24% |
| `cars_sport` | 73 | 1% | 4% | 30% |
| `cars_offroad` | 62 | 1% | 2% | 30% |
| `#ed3327f7` | 13 | 24% | 43% | 72% |
| `semi` | 11 | 4% | 6% | 25% |
| `haulage` | 11 | 25% | 40% | 50% |
| `misc` | 8 | 6% | 7% | 7% |
| `#7d55604a` | 2 | 4% | 6% | 8% |
| `#77019a7c` | 1 | 4% | 4% | 4% |
| `business` | 1 | 10% | 10% | 10% |
| `#d81dfe26` | 1 | 10% | 10% | 10% |
| `#87448490` | 1 | 40% | 40% | 40% |
| `#bf5c2a05` | 1 | 30% | 30% | 30% |

`trailers` and `trailers_container` are defined in the same file but used by no schedule row. The file also defines the condition tags `clean`, `used`, `worn`, `rusty`, `tarnished`, and the fields `VehicleRecipes`, `TaxiGroup` and `ClusterData`.

Of the six unnamed hashes only `#ed3327f7` matters (13 schedules, 24-72%). A 30.7M-candidate attack returned `goosekey_haulage_leonard` - one hit against roughly 0.04 expected false positives, so it is probably genuine, but the name is odd enough to treat as tentative.

## Area archetypes - the zone with the highest share of each category

This is the direct answer to the GTA V style question. The game does do it, and these are the extremes.

| Category | Peak share | Schedule | Full mix of that schedule (time slot 0) |
|---|---|---|---|
| `cars_pickup` | **54%** | #70 (`#df894d1d`) | cars_pickup 54%, cars_offroad 30%, motorbikes 10%, vans 6% |
| `cars_suv` | **42%** | #38 (`#720c6bba`) | cars_suv 42%, cars_pickup 28%, cars_sedan 17%, cars_compact 3%, vans 3%, cars_coupe 2%, cars_muscle 2%, cars_sport 1%, motorbikes 1%, trucks 1% |
| `cars_sedan` | **40%** | #26 (`#fcc7f868`) | cars_sedan 40%, cars_suv 27%, vans 12%, motorbikes 5%, cars_coupe 3%, cars_muscle 3%, cars_pickup 3%, cars_compact 2%, cars_taxi 2%, trucks 2%, utility 1% |
| `cars_sport` | **30%** | #21 (`#f18fdd5d`) | cars_sport 30%, cars_suv 22%, cars_sedan 20%, cars_muscle 12%, motorbikes 7%, cars_coupe 6%, cars_taxi 3% |
| `cars_offroad` | **30%** | #70 (`#df894d1d`) | cars_pickup 54%, cars_offroad 30%, motorbikes 10%, vans 6% |
| `cars_taxi` | **24%** | #31 (`#2ad14852`) | cars_taxi 24%, cars_sedan 17%, cars_suv 17%, cars_pickup 10%, motorbikes 8%, vans 7%, cars_sport 6%, cars_compact 3%, cars_muscle 2%, trucks 2%, utility 2%, cars_coupe 1%, cars_offroad 1% |
| `cars_muscle` | **23%** | #103 (`#f545c3fe`) | cars_muscle 23%, cars_suv 20%, cars_sport 19%, cars_sedan 17%, cars_taxi 6%, cars_pickup 5%, motorbikes 4%, cars_coupe 3%, cars_compact 2%, cars_offroad 1%, trucks 0%, vans 0% |
| `vans` | **23%** | #97 (`#0d3f264b`) | cars_sedan 35%, vans 23%, cars_suv 13%, cars_pickup 7%, motorbikes 7%, trucks 5%, cars_compact 4%, cars_taxi 3%, utility 2%, cars_coupe 1% |
| `cars_coupe` | **18%** | #83 (`#1a0e927e`) | cars_sedan 20%, cars_coupe 18%, cars_suv 16%, cars_muscle 12%, vans 9%, cars_sport 8%, cars_taxi 8%, motorbikes 6%, trucks 3% |
| `motorbikes` | **14%** | #98 (`#3c6a5fef`) | cars_sedan 33%, cars_suv 26%, cars_pickup 14%, motorbikes 14%, vans 5%, cars_compact 2%, cars_taxi 2%, cars_coupe 1%, cars_muscle 1%, cars_offroad 1%, trucks 1% |
| `utility` | **31%** | #28 (`#fa8d00cc`) | utility 31%, cars_suv 30%, cars_sedan 14%, cars_sport 10%, cars_coupe 5%, cars_pickup 4%, cars_muscle 3%, vans 3% |
| `cars_compact` | **11%** | #22 (`#584044f2`) | cars_sport 25%, cars_sedan 20%, cars_compact 11%, cars_pickup 9%, cars_suv 9%, cars_coupe 6%, cars_taxi 5%, motorbikes 5%, trucks 5%, vans 5% |
| `trucks` | **80%** | #113 (`#1703867f`) | trucks 80%, vans 20% |

## Time of day

126 of the 137 schedules are flat across all 12 slots. Only **11** change, and the changes are deliberate.

**Schedule #31** (`#2ad14852`)

| Category | 12 time slots |
|---|---|
| `cars_sedan` | 17 17 17 27 27 27 27 27 27 27 27 27 |
| `cars_taxi` | 24 24 24 14 14 14 14 14 14 14 14 14 |

**Schedule #32** (`#7f1a03a9`)

| Category | 12 time slots |
|---|---|
| `cars_sedan` | 16 16 16 26 26 26 26 26 26 26 26 26 |
| `cars_taxi` | 18 18 18 8 8 8 8 8 8 8 8 8 |

**Schedule #43** (`#5591f922`)

| Category | 12 time slots |
|---|---|
| `cars_sedan` | 30 26 26 26 26 26 26 26 26 30 30 30 |
| `cars_suv` | 19 15 15 15 15 15 15 15 15 19 19 19 |
| `trucks` | 0 8 8 8 8 8 8 8 8 0 0 0 |

**Schedule #77** (`#c09435b4`)

| Category | 12 time slots |
|---|---|
| `cars_sedan` | 8 8 18 18 18 18 18 18 18 18 8 8 |
| `cars_sport` | 25 25 16 16 16 16 16 16 16 16 25 25 |
| `cars_suv` | 15 15 20 20 20 20 20 20 20 20 15 15 |
| `cars_taxi` | 18 18 12 12 12 12 12 12 12 12 18 18 |

**Schedule #80** (`#938295b4`)

| Category | 12 time slots |
|---|---|
| `cars_coupe` | 10 10 5 5 5 5 5 5 5 5 10 10 |
| `cars_muscle` | 14 14 12 12 12 12 12 12 12 12 14 14 |
| `cars_pickup` | 4 4 6 6 6 6 6 6 6 5 5 5 |
| `cars_sedan` | 10 10 12 12 12 12 12 12 12 12 10 10 |
| `cars_sport` | 17 17 15 15 15 15 18 18 18 19 17 18 |
| `motorbikes` | 8 8 12 12 12 12 12 12 12 12 9 8 |
| `utility` | 5 5 5 5 5 3 3 3 3 3 3 3 |
| `vans` | 5 5 6 6 6 8 5 5 5 5 5 5 |

**Schedule #81** (`#ed1efc46`)

| Category | 12 time slots |
|---|---|
| `cars_sport` | 20 20 2 2 2 2 2 2 20 20 20 20 |
| `cars_suv` | 10 10 18 18 18 18 18 18 10 10 10 10 |
| `motorbikes` | 7 7 5 5 5 5 5 5 7 7 7 7 |
| `trucks` | 2 2 5 5 5 5 5 5 2 2 2 2 |
| `vans` | 7 7 16 16 16 16 16 16 7 7 7 7 |

**Schedule #103** (`#f545c3fe`)

| Category | 12 time slots |
|---|---|
| `cars_muscle` | 23 23 23 18 8 8 8 8 8 8 18 23 |
| `cars_offroad` | 1 1 1 4 4 4 4 4 4 4 4 1 |
| `cars_sedan` | 17 17 17 27 27 27 27 27 27 27 27 17 |
| `cars_sport` | 19 19 19 4 4 4 4 4 4 4 4 19 |
| `cars_suv` | 20 20 20 27 27 27 27 27 27 27 27 20 |
| `trucks` | 0 0 0 0 3 3 3 3 3 3 0 0 |
| `vans` | 0 0 0 0 7 7 7 7 7 7 0 0 |

**Schedule #104** (`#1aaebcee`)

| Category | 12 time slots |
|---|---|
| `cars_sport` | 14 14 14 2 2 2 2 2 2 14 14 14 |
| `cars_taxi` | 21 21 21 14 14 14 14 14 14 21 21 21 |
| `trucks` | 0 0 0 6 6 6 6 6 6 0 0 0 |
| `utility` | 0 0 0 2 2 2 2 2 2 0 0 0 |
| `vans` | 0 0 0 11 11 11 11 11 11 0 0 0 |

**Schedule #105** (`#221ffe81`)

| Category | 12 time slots |
|---|---|
| `cars_sport` | 14 14 14 2 2 2 2 2 2 14 14 14 |
| `cars_taxi` | 21 21 21 14 14 14 14 14 14 21 21 21 |
| `trucks` | 0 0 0 6 6 6 6 6 6 0 0 0 |
| `utility` | 0 0 0 2 2 2 2 2 2 0 0 0 |
| `vans` | 0 0 0 11 11 11 11 11 11 0 0 0 |

**Schedule #120** (`#745bf860`)

| Category | 12 time slots |
|---|---|
| `cars_sedan` | 20 20 13 13 20 20 20 20 20 20 20 20 |
| `cars_sport` | 10 10 6 6 6 6 6 6 6 6 10 10 |
| `cars_suv` | 39 39 32 32 39 39 39 39 39 39 39 39 |
| `trucks` | 0 0 4 4 0 0 0 0 0 0 0 0 |
| `utility` | 0 0 3 3 0 0 0 0 0 0 0 0 |
| `vans` | 6 6 17 17 10 10 10 10 10 10 6 6 |

**Schedule #136** (`#e5f4831b`)

| Category | 12 time slots |
|---|---|
| `semi` | 4 4 3 3 3 3 3 3 3 3 4 4 |
| `#ed3327f7` | 36 36 27 27 27 27 27 27 27 27 36 36 |
| `#87448490` | 30 30 40 40 40 40 40 40 40 40 30 30 |

Readable patterns: taxis drop by roughly half after the first three slots (#31, #32, #104, #105); `trucks` and `utility` sit at 0% in the night slots and only appear by day (#43, #103, #104, #105, #120); and the sport/muscle share spikes at night in #77, #80, #81 and #103 - 20-25% sports cars at night against 2-4% during the day.

## The freight corridors

Schedules #124-#136 are a distinct block with no sedans or SUVs at all: `haulage` 25-50%, `semi` 4-25%, the unnamed `#ed3327f7` at 24-72%, plus a little `misc`, `motorbikes` and `cars_pickup`. These are the highway/freight profiles.

## What is still missing

The 137 schedule names are stored as joaat hashes and only **one** was recovered: `rialto_islands_region_14`. That fixes the convention as `<area>_region_<n>`, so the schedules genuinely are per named area - but the other 136 base names are not in any plain-text list in the dump. I tried 15,904 place-name bases harvested from the 36,888-entry text table, crossed with `_region_`, `_area_`, `_zone_`, `_sector_`, `_district_`, `_popzone_`, `_sched_`, `_block_` and `_tile_` and indices 0-200, with no further hits.

So the mix tables are complete and exact, but attaching each one to a neighbourhood by name needs either the schedule-name strings or a pass that pulls the zone polygon coordinates out of the gameshape files and places them on the map. The polygons are present (`Shape`, `Components`, `Height` per node), so that is the next step if the mapping is wanted.

## Area-specific vehicle sets that ARE fully named

These come from the livery/recipe table in `common.rpf`, which ships as plain XML, so the area names survive intact. This is the part of which-vehicles-where that needs no hash cracking at all.

### Police vehicles by jurisdiction (137 liveries)

| Area / force | Vehicles |
|---|---|
| `AMBROSIA` | bison1800, burrito, lowline, lurepredator, trailerbox |
| `BELVILLE` | buffalo6, burrito |
| `BELVILLE_1_STAR` | landstalker2 |
| `CAPRI` | landstalker2, riot, scorcher |
| `CAPRIPOLICE` | buffalo6 |
| `CAPRIPOLICE_1_STAR` | sovereign107 |
| `CAPRI_1_STAR` | gauntlet4 |
| `HAMLET` | granger, stanierg2 |
| `HAMLET_3_STAR` | stanierg2 |
| `HIGHWAYPATROL` | buffalo6, montagne |
| `KELLYCOUNTY` | bison1800, buzzard, montagne, rhode, stockade, valkyrie |
| `KELLYCOUNTY_4_STAR` | montagne |
| `KEYLENTO` | lowline, montagne, rhode |
| `LEONARDCOUNTY` | bison1800, blast, buzzard, lowline, montagne, polmav |
| `LEONARDCOUNTY_3_STAR` | stanierg2 |
| `LUMMOXCOUNTY` | bison1800, buzzard, stanierg2 |
| `LUMMOXCOUNTYK9` | granger |
| `LUMMOXCOUNTY_5_STAR` | bison1800 |
| `MAIN` | bison1800, bison900, blast, buffalo6, burrito, buzzard, coastguard, gauntlet4, granger, landstalker2, lowline, lurepredator, montagne, patriot3, rhode, riot, scorcher, sovereign107, stanierg2, stockade, valkyrie, verus |
| `MARIANACOUNTY` | bison1800, buzzard, granger, lowline, lurepredator, polmav |
| `MARIANACOUNTY_1_STAR` | bison1800 |
| `MARIANACOUNTY_3_STAR` | bison1800, lowline |
| `MARIANACOUNTY_4_STAR` | bison1800, granger |
| `MARIANACOUNTY_5_STAR` | bison1800 |
| `NATIONALCOASTGUARD` | bison1800, montagne, rhode |
| `NOOSE` | patriot3, polmav, valkyrie |
| `PORTGELLHORN` | bison1800, blast, lurepredator, stanierg2 |
| `TURRET` | patriot3 |
| `UNMARKED` | gauntlet4 |
| `VCPD` | buzzard, lowline, lurepredator, montagne, sovereign107, stanierg2 |
| `VCPD_1_STAR` | lowline, montagne, stanierg2 |
| `VCPD_2_STAR` | lowline, stanierg2 |
| `VCPD_3_STAR` | lowline, stanierg2 |
| `VCPD_4_STAR` | lowline, stanierg2 |
| `VICEBEACH` | bison1800, blast, buffalo6, rhode, scorcher, sovereign107, verus |
| `VICEBEACH_1_STAR` | gauntlet4 |
| `VICECITY` | buffalo6 |
| `VICECITY_1_STAR` | buffalo6 |
| `VICECITY_2_STAR` | buffalo6 |
| `VICECITY_3_STAR` | buffalo6 |
| `VICECITY_4_STAR` | buffalo6 |
| `VICEDALE` | bison1800, buffalo6, buzzard, coastguard, granger, landstalker2, lurepredator, montagne, polmav, riot, sovereign107, stockade, valkyrie |
| `VICEDALE_1_STAR` | buffalo6 |
| `VICEDALE_2_STAR` | buffalo6 |
| `VICEDALE_3_STAR` | buffalo6 |
| `VICEDALE_4_STAR` | landstalker2 |
| `WANING_SANDS` | buffalo6 |
| `WATSONBAY` | bison900, blast, lurepredator |
| `WATSONBAY_3_STAR` | bison900 |

### Taxi fleets by company or area (31 liveries)

| Area / force | Vehicles |
|---|---|
| `AMBERCABCO` | minivan, stanierg2, vivanite |
| `ARCHAVENUE` | minivan, stanierg2 |
| `BELVILLE` | minivan, stanierg2 |
| `CLEAN` | allander, granger, minivan, stanierg2 |
| `GOODTIME` | minivan, vivanite |
| `GOODTIMETAXI` | stanierg2 |
| `KAUFMAN` | vivanite |
| `KAUFMANCAB` | minivan, stanierg2 |
| `KEYLENTO` | minivan, stanierg2 |
| `MAIN` | vivanite |
| `PARADISE` | minivan, vivanite |
| `PARADISETAXI` | stanierg2 |
| `PORTGELLHORN` | stanierg2 |
| `USED` | allander, granger, minivan, stanierg2 |
| `VICEAIRPORT` | minivan, stanierg2 |
| `VICEINTERNATIONALAIRPORT` | vivanite |

### Gang vehicle sets (55 liveries)

| Area / force | Vehicles |
|---|---|
| `FINALCHAPTER_MAIN` | avarus, nightblade2, sovereign107, zombieb |
| `GUARDIA_MAIN` | animus, chino, montagne |
| `KINGFISHERS_MAIN` | airgator, bison3800, contender, lurepredator, verus |
| `KINGFISHER_MAIN` | bombora |
| `KOLT_MAIN` | bison, bison900, blazer4, caracara4x4, dominator7, dorado, impaler5, montagnexl, ruiner, sanchez, stanier, stanierg2, tulip2, walton, yosemite1500 |
| `M7_MAIN` | impaler5, impaler6, picador |
| `POLILLA_NEGRA_MAIN` | baller8 |
| `POLLIA_MAIN` | clarion |
| `PTT_MAIN` | blazer4, chavosxs, cypher, intruder, manchez, sanchez, tailgater2, willard |
| `S4S_MAIN` | cavalcade3, impaler5, impaler6, meritlx, pmp700, stanier, stanierg2, tulip2, willard |
| `STYLING_FINALCHAPTER` | global |
| `STYLING_FINALCHAPTER_BIKE` | global |
| `STYLING_GUARDIA` | global |
| `STYLING_KOLT` | global |
| `STYLING_PTT` | global |

### Ambient style sets (the visual subcultures)

| Style | Vehicles |
|---|---|
| `DONK` | asterope2, chavosxs, impaler6, lasso, manana, meritlx, pmp700, primo, willard |
| `DONK_BASE` | asterope2, chavosxs, impaler6, lasso, meritlx, pmp700, primo, willard |
| `DRAG` | buccaneer, declasse500, dominator, dominator7, dominatorclassic, gauntlet4, gauntlet5, phoenix, picador, ruiner, slamvan, stallion, tulip2, vamos, yosemite |
| `HIGHEND` | carbonizzare, comet7, coquette, cypher, furia, growler, infernus2, toros, vitello |
| `HIHGEND` | coquette4 |
| `HOTROD` | hustler |
| `LOWRIDER` | buccaneer, impaler6, sungrazer |
| `LOWRIDER_BASE` | chino, manana |
| `MUDPARK` | blazer4, outlaw |
| `MUSCLE` | buccaneer, dominator, dominator7, gauntlet4, gauntlet5, phoenix, picador, ruiner, stallion, tulip, tulip2, vamos |
| `OFFROAD` | asterope2, bison, bison1800, bison3800, bison4, bison900, blast, bobcatxl, bogan, caracara4x4, contender, crawler, dorado, futo, grouse, guardiantx2, hellion, intruder, marshall, mesa, montagnepu, montagnexl, phoenix, primo, rancher, rebel, reclaimer, reclaimerclassic, sandking_4dr, sandkingxt, stanierg2, sultan, walton, yosemite1500 |
| `OFFROAD_BASE` | bison, bison1800, bison3800, bison900, bobcatxl, caracara4x4, contender, kamacho, montagnepu, rancher, rebel, reclaimer, riata, sandking_4dr, sandkingxt, walton, yosemite1500 |
| `OFFROAD_NOBEDMODS` | marshall |
| `OFFROAD_NOROOFMODS` | caracara4x4, marshall |
| `OFFROAD_TRUCKBED_MODS` | caracara4x4 |
| `RATROD` | hustler |
| `ROOF_ATTACHMENTS` | allander, animus, astron, baller2, baller8, bogan, cavalcade3, cypher, dorado, drafter, dubsta, feroci, granger, granger3600lx, greta, growler, hellion, intruder, landstalker2, lowline, mesa, montagne, montagnexl, neon, penumbra, reblagts, schafter3, schafters, sugoi, sultan, tailgater2, toros, vstr, xls |
| `TOWING` | aleutian, astron, baller2, bison, bison1800, bison3800, bison900, blast, bobcatxl, bogan, contender, dorado, granger, gresleyxt, hellion, mesa, montagnepu, montagnexl, reclaimer, sandkingxt, walton, yosemite1500 |
| `TOWING_GOOSENECK` | bison3800, bobcatxl, sandking_4dr |
| `TRACK` | buffalo6, cypher, dominator7, vstr |
| `TUNER` | akane, baller2, buffalo, buffalo6, carbonizzare, chavosxs, comet7, coquette4, cypher, dominator9, drafter, elegy, fr36, furia, gauntlet4, growler, intruder, italirsx, jugular, penumbra, penumbraff, schafter3, schafters, sugoi, sultan, tailgater2, tempesta, vectre, vigero3, vitello, vstr |
| `TUNER_BASE` | astron, baller8, comet7, coquette4, furia, growler, italirsx, neon, paragon, tempesta, vitello, zorrusso |

The full 137-schedule table is in `gta6_traffic_schedules.csv` beside this file: one row per schedule and category, with all 12 time slots.

---

# Part 2 - the zones placed on the map

The schedule names are hashed, but the zone *geometry* is not. Each traffic schedule is selected by a gameshape zone volume, and those volumes carry plain float coordinates. Walking them gives the mapping without ever needing the schedule names.

## How the geometry was read

Three encodings had to be worked out from the bytes, none of them documented anywhere:

| Thing | Encoding |
|---|---|
| PSO pointer (8 bytes) | `u16 blockId`, `u8 flags`, `u40 offsetUnits`; byte offset = `offsetUnits * 16` |
| array, member type `0x5c` | `u32 count`, `u32 capacity`, pointer(8) |
| array, member type `0x52` | pointer(8), `u16 count`, `u16 capacity`, 4 bytes padding |
| point pool | the block whose id is `0x00000031`; 16 bytes per point (`float x`, `float y`, 8 unused) |

A node (`#b55501c9`: `Name`, `Shape`, `Components`) points at a shape (`#fc24fa58`: outer ring, inner rings, base Z, `Height`) and at a list of components through a pointer block (`0x00000007`). One of those components carries the popcycle schedule hash in a nested struct.

Sweeping all 200 zone volumes out of 1,706 gameshape files resolved **132 of the 137 schedules** to real world polygons.

## The world

Zone coordinates span **x -11135 to 3884** and **y -8646 to 10066** (about 15 km by 19 km). Zones are extruded volumes: most sit at base Z -100 with height 1100, city ones at -30 with height 1500.

## Naming the places

The zone volumes are not named after places, but other gameshape nodes are. Brute-forcing node name hashes against place names harvested from the game's own text table turned up the convention `region_<area>` (plus `area_<x>_<nn>`, `<x>_volume`, `freeway_<nn>`, `city_<nn>`, `<colour>_route`). That yielded **199 named polygons, 113 of them `region_*`** - a real gazetteer with coordinates.

Each named place is then assigned the **smallest** schedule zone whose polygon contains it, which is how zone priority works: specific beats broad.

- 141 places land inside a specific zone (under 5 km²) - these assignments are solid.
- 58 fall through to a broad county or highway zone (5-56 km²). For those the answer is really "the regional schedule applies here", either because the place genuinely has no zone of its own or because its zone is one of the five schedules I could not resolve.

## Named areas and the traffic that spawns in them

Top categories at time slot 0. `S` marks a specific zone, `R` a broad regional fallback.

| Area | Sched | Zone km² | | Traffic mix (slot 0) |
|---|---|---|---|---|
| `abismokey` | #58 | 1.20 | S | cars_sedan 22%, cars_pickup 16%, vans 12%, cars_suv 10%, cars_offroad 8% |
| `ambrosia` | #8 | 0.49 | S | cars_pickup 36%, cars_suv 25%, cars_sedan 19%, trucks 8%, utility 5% |
| `ambrosiaeast` | #127 | 7.38 | R | haulage 50%, #ed3327f7 32%, #7d55604a 8%, misc 7%, cars_pickup 3% |
| `balsero` | #49 | 0.49 | S | cars_pickup 32%, vans 22%, cars_sedan 16%, cars_suv 11%, trucks 10% |
| `bayside` | #92 | 1.26 | S | cars_sport 26%, cars_suv 23%, cars_pickup 18%, cars_sedan 15%, cars_coupe 7% |
| `belpoint` | #135 | 33.97 | R | haulage 40%, #ed3327f7 36%, motorbikes 10%, misc 7%, semi 4% |
| `belville` | #126 | 10.87 | R | #ed3327f7 45%, haulage 40%, misc 7%, semi 5%, cars_pickup 3% |
| `belvilleeast` | #126 | 10.87 | R | #ed3327f7 45%, haulage 40%, misc 7%, semi 5%, cars_pickup 3% |
| `belvillenorth` | #127 | 7.38 | R | haulage 50%, #ed3327f7 32%, #7d55604a 8%, misc 7%, cars_pickup 3% |
| `belvilletown` | #13 | 0.46 | S | cars_suv 30%, cars_sedan 21%, cars_pickup 15%, vans 8%, motorbikes 7% |
| `bobcatkey` | #135 | 33.97 | R | haulage 40%, #ed3327f7 36%, motorbikes 10%, misc 7%, semi 4% |
| `bonanza` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `buckskinswamp` | #130 | 12.09 | R | haulage 40%, #ed3327f7 32%, motorbikes 10%, misc 7%, #7d55604a 4% |
| `capri` | #21 | 0.04 | S | cars_sport 30%, cars_suv 22%, cars_sedan 20%, cars_muscle 12%, motorbikes 7% |
| `capriisland` | #15 | 0.35 | S | cars_suv 27%, cars_sport 25%, cars_sedan 21%, cars_coupe 14%, cars_muscle 6% |
| `catalankey` | #25 | 0.88 | S | cars_suv 30%, cars_sedan 22%, cars_pickup 12%, cars_coupe 10%, vans 7% |
| `corsairkey` | #56 | 1.58 | S | cars_pickup 19%, cars_sedan 18%, cars_muscle 12%, cars_suv 11%, cars_coupe 8% |
| `crosstown` | #26 | 0.14 | S | cars_sedan 40%, cars_suv 27%, vans 12%, motorbikes 5%, cars_coupe 3% |
| `crudeislands` | #128 | 19.50 | R | #ed3327f7 64%, haulage 25%, semi 11% |
| `daltonisland` | #28 | 0.92 | S | utility 31%, cars_suv 30%, cars_sedan 14%, cars_sport 10%, cars_coupe 5% |
| `decordistrict` | #29 | 0.19 | S | cars_suv 30%, cars_sport 27%, cars_sedan 17%, cars_coupe 8%, motorbikes 5% |
| `downtown` | #31 | 0.39 | S | cars_taxi 24%, cars_sedan 17%, cars_suv 17%, cars_pickup 10%, motorbikes 8% |
| `draperisland` | #86 | 4.03 | S | cars_sedan 22%, cars_pickup 20%, cars_suv 17%, cars_coupe 11%, cars_muscle 10% |
| `eastbigpine` | #74 | 5.97 | R | cars_suv 32%, cars_pickup 29%, cars_sedan 20%, vans 10%, cars_coupe 2% |
| `eastbuckskinswamp` | #130 | 12.09 | R | haulage 40%, #ed3327f7 32%, motorbikes 10%, misc 7%, #7d55604a 4% |
| `eastlummoxcounty` | #75 | 1.46 | S | cars_suv 35%, cars_pickup 30%, cars_sedan 18%, motorbikes 4%, vans 4% |
| `eastpelicanvalley` | #74 | 5.97 | R | cars_suv 32%, cars_pickup 29%, cars_sedan 20%, vans 10%, cars_coupe 2% |
| `eastwatsonbay` | #74 | 5.97 | R | cars_suv 32%, cars_pickup 29%, cars_sedan 20%, vans 10%, cars_coupe 2% |
| `emeraldsprings` | #35 | 2.59 | S | cars_pickup 40%, cars_suv 26%, cars_sedan 19%, vans 7%, cars_muscle 3% |
| `featherstoneislands` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `framptonislands` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `frogswater` | #70 | 0.74 | S | cars_pickup 54%, cars_offroad 30%, motorbikes 10%, vans 6% |
| `gellhorn` | #128 | 19.50 | R | #ed3327f7 64%, haulage 25%, semi 11% |
| `glorianakey` | #24 | 1.26 | S | cars_sedan 25%, cars_suv 23%, cars_pickup 16%, vans 12%, cars_coupe 4% |
| `grassrivers` | #123 | 0.37 | S | cars_pickup 32%, cars_suv 26%, cars_sedan 24%, utility 5%, cars_compact 3% |
| `grayson` | #38 | 0.46 | S | cars_suv 42%, cars_pickup 28%, cars_sedan 17%, cars_compact 3%, vans 3% |
| `hamlet` | #41 | 0.15 | S | cars_sedan 36%, cars_pickup 31%, cars_suv 21%, cars_muscle 8%, cars_coupe 4% |
| `hydeisland` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `kalaga` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `kelly_county` | #91 | 2.60 | S | cars_suv 30%, cars_sedan 28%, cars_pickup 15%, vans 10%, cars_taxi 5% |
| `keylento` | #57 | 0.38 | S | cars_pickup 35%, cars_sedan 25%, cars_coupe 10%, cars_muscle 8%, cars_suv 8% |
| `laperle` | #60 | 0.15 | S | cars_sedan 32%, cars_suv 27%, cars_taxi 11%, vans 10%, cars_pickup 5% |
| `leaflinks` | #116 | 0.66 | S | cars_sedan 28%, cars_suv 26%, cars_sport 15%, cars_coupe 8%, cars_muscle 7% |
| `leonard_county` | #126 | 10.87 | R | #ed3327f7 45%, haulage 40%, misc 7%, semi 5%, cars_pickup 3% |
| `leonida` | #130 | 12.09 | R | haulage 40%, #ed3327f7 32%, motorbikes 10%, misc 7%, #7d55604a 4% |
| `littlecuba` | #66 | 0.12 | S | cars_suv 24%, vans 16%, cars_sedan 13%, cars_taxi 10%, cars_compact 9% |
| `littlehardinisland` | #125 | 7.07 | R | #ed3327f7 45%, haulage 40%, misc 7%, semi 5%, cars_pickup 3% |
| `lummox_county` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `manifestcanyon` | #86 | 4.03 | S | cars_sedan 22%, cars_pickup 20%, cars_suv 17%, cars_coupe 11%, cars_muscle 10% |
| `mariana_county` | #135 | 33.97 | R | haulage 40%, #ed3327f7 36%, motorbikes 10%, misc 7%, semi 4% |
| `maroonkey` | #73 | 55.83 | R | cars_pickup 29%, cars_suv 28%, cars_sedan 20%, vans 10%, trucks 4% |
| `milezeroairstation` | #73 | 55.83 | R | cars_pickup 29%, cars_suv 28%, cars_sedan 20%, vans 10%, trucks 4% |
| `millport` | #75 | 1.46 | S | cars_suv 35%, cars_pickup 30%, cars_sedan 18%, motorbikes 4%, vans 4% |
| `millportharbor` | #76 | 0.67 | S | cars_pickup 34%, cars_sedan 28%, cars_suv 20%, vans 6%, cars_compact 2% |
| `moreau` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `mounthailescountryclub` | #126 | 10.87 | R | #ed3327f7 45%, haulage 40%, misc 7%, semi 5%, cars_pickup 3% |
| `northbobcatkey` | #135 | 33.97 | R | haulage 40%, #ed3327f7 36%, motorbikes 10%, misc 7%, semi 4% |
| `northfrogswater` | #70 | 0.74 | S | cars_pickup 54%, cars_offroad 30%, motorbikes 10%, vans 6% |
| `northgellhorn` | #88 | 0.67 | S | cars_suv 24%, cars_pickup 22%, vans 20%, cars_sedan 13%, cars_coupe 4% |
| `northkalaga` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `northkellycounty` | #130 | 12.09 | R | haulage 40%, #ed3327f7 32%, motorbikes 10%, misc 7%, #7d55604a 4% |
| `northkeylento` | #55 | 0.27 | S | cars_pickup 29%, cars_sedan 17%, cars_suv 14%, vans 9%, cars_sport 6% |
| `northleonardcounty` | #125 | 7.07 | R | #ed3327f7 45%, haulage 40%, misc 7%, semi 5%, cars_pickup 3% |
| `northmarianacounty` | #73 | 0.61 | S | cars_pickup 29%, cars_suv 28%, cars_sedan 20%, vans 10%, trucks 4% |
| `northmoreau` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `northwatsonbay` | #123 | 0.37 | S | cars_pickup 32%, cars_suv 26%, cars_sedan 24%, utility 5%, cars_compact 3% |
| `oceanbeach` | #80 | 0.24 | S | cars_sport 17%, cars_muscle 14%, cars_suv 14%, cars_taxi 13%, cars_coupe 10% |
| `peacockbay` | #106 | 1.00 | S | cars_suv 27%, cars_sedan 18%, cars_taxi 14%, cars_pickup 12%, vans 10% |
| `peacockestates` | #110 | 0.33 | S | cars_suv 31%, cars_sedan 29%, cars_coupe 12%, cars_sport 10%, cars_pickup 8% |
| `permitkeys` | #73 | 55.83 | R | cars_pickup 29%, cars_suv 28%, cars_sedan 20%, vans 10%, trucks 4% |
| `portgellhorn` | #86 | 4.03 | S | cars_sedan 22%, cars_pickup 20%, cars_suv 17%, cars_coupe 11%, cars_muscle 10% |
| `portvc` | #112 | 0.64 | S | cars_sedan 24%, cars_suv 24%, cars_taxi 10%, trucks 10%, vans 8% |
| `redappleorchard` | #86 | 4.03 | S | cars_sedan 22%, cars_pickup 20%, cars_suv 17%, cars_coupe 11%, cars_muscle 10% |
| `refreshments_07` | #16 | 0.11 | S | cars_sedan 22%, cars_muscle 20%, cars_suv 15%, cars_sport 11%, cars_offroad 6% |
| `refreshments_08` | #16 | 0.11 | S | cars_sedan 22%, cars_muscle 20%, cars_suv 15%, cars_sport 11%, cars_offroad 6% |
| `riquezaskeys` | #73 | 55.83 | R | cars_pickup 29%, cars_suv 28%, cars_sedan 20%, vans 10%, trucks 4% |
| `rockridge` | #95 | 0.21 | S | cars_sedan 34%, cars_suv 33%, vans 9%, motorbikes 7%, cars_pickup 5% |
| `rushwoodlogging` | #76 | 0.67 | S | cars_pickup 34%, cars_sedan 28%, cars_suv 20%, vans 6%, cars_compact 2% |
| `sabal` | #114 | 0.93 | S | cars_pickup 36%, cars_suv 22%, cars_sedan 18%, vans 15%, utility 4% |
| `saintattawayislands` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `salton` | #2 | 0.29 | S | cars_sedan 28%, cars_suv 24%, cars_taxi 15%, vans 12%, cars_muscle 5% |
| `sandbars` | #128 | 19.50 | R | #ed3327f7 64%, haulage 25%, semi 11% |
| `shoelesskeys` | #73 | 55.83 | R | cars_pickup 29%, cars_suv 28%, cars_sedan 20%, vans 10%, trucks 4% |
| `southambrosia` | #8 | 0.49 | S | cars_pickup 36%, cars_suv 25%, cars_sedan 19%, trucks 8%, utility 5% |
| `southkalaga` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `southkellycounty` | #51 | 0.38 | S | cars_pickup 36%, cars_suv 26%, cars_sedan 19%, vans 7%, cars_muscle 3% |
| `southleonardcounty` | #127 | 7.38 | R | haulage 50%, #ed3327f7 32%, #7d55604a 8%, misc 7%, cars_pickup 3% |
| `southpelicanvalley` | #74 | 5.97 | R | cars_suv 32%, cars_pickup 29%, cars_sedan 20%, vans 10%, cars_coupe 2% |
| `southside` | #100 | 0.10 | S | cars_suv 26%, cars_sedan 22%, cars_pickup 15%, vans 14%, trucks 11% |
| `southvicedalecounty` | #111 | 0.95 | S | cars_sedan 28%, cars_pickup 24%, cars_suv 21%, vans 7%, cars_muscle 6% |
| `southwestlummoxcounty` | #38 | 0.41 | S | cars_suv 42%, cars_pickup 28%, cars_sedan 17%, cars_compact 3%, vans 3% |
| `southwind` | #36 | 0.27 | S | cars_suv 28%, cars_pickup 26%, cars_sedan 17%, cars_muscle 6%, vans 6% |
| `starfishisland` | #93 | 0.38 | S | cars_suv 36%, cars_sport 27%, cars_sedan 18%, cars_coupe 8%, cars_pickup 6% |
| `stockyard` | #103 | 0.11 | S | cars_muscle 23%, cars_suv 20%, cars_sport 19%, cars_sedan 17%, cars_taxi 6% |
| `tequesta` | #104 | 0.61 | S | cars_suv 23%, cars_sedan 22%, cars_taxi 21%, cars_sport 14%, motorbikes 11% |
| `tequestaretreat` | #109 | 0.33 | S | cars_suv 31%, cars_sedan 29%, cars_coupe 13%, cars_sport 6%, cars_muscle 5% |
| `vcia` | #1 | 1.23 | S | cars_sedan 20%, vans 20%, cars_pickup 18%, cars_suv 16%, trucks 10% |
| `vice_beach` | #81 | 0.59 | S | cars_sedan 21%, cars_sport 20%, cars_suv 10%, cars_coupe 8%, cars_muscle 7% |
| `vicecity` | #33 | 0.07 | S | cars_sedan 32%, cars_suv 20%, cars_sport 14%, cars_coupe 10%, cars_taxi 6% |
| `vicedale_county` | #33 | 0.38 | S | cars_sedan 32%, cars_suv 20%, cars_sport 14%, cars_coupe 10%, cars_taxi 6% |
| `vinewoodbeach` | #116 | 1.33 | S | cars_sedan 28%, cars_suv 26%, cars_sport 15%, cars_coupe 8%, cars_muscle 7% |
| `waningsands` | #114 | 0.93 | S | cars_pickup 36%, cars_suv 22%, cars_sedan 18%, vans 15%, utility 4% |
| `waningsandscentral` | #115 | 0.91 | S | cars_sedan 30%, cars_suv 25%, cars_pickup 14%, cars_compact 6%, cars_coupe 6% |
| `washingtonbeach` | #119 | 0.16 | S | cars_sedan 26%, cars_suv 23%, cars_pickup 13%, cars_coupe 7%, cars_muscle 7% |
| `watsonbay` | #122 | 0.24 | S | cars_pickup 30%, cars_suv 23%, utility 19%, cars_sedan 13%, cars_coupe 3% |
| `waxmanisland` | #94 | 0.66 | S | cars_pickup 24%, cars_suv 18%, vans 17%, cars_sedan 12%, cars_sport 7% |
| `westbuckskinswamp` | #130 | 12.09 | R | haulage 40%, #ed3327f7 32%, motorbikes 10%, misc 7%, #7d55604a 4% |
| `westkalaga` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `westmoreau` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |
| `westvicedalecounty` | #111 | 0.28 | S | cars_sedan 28%, cars_pickup 24%, cars_suv 21%, vans 7%, cars_muscle 6% |
| `wildhogairfield` | #128 | 19.50 | R | #ed3327f7 64%, haulage 25%, semi 11% |
| `wildhogpoint` | #128 | 19.50 | R | #ed3327f7 64%, haulage 25%, semi 11% |
| `zephyrislands` | #124 | 33.64 | R | #ed3327f7 54%, haulage 34%, semi 6%, misc 6% |

Other named polygons (airports, prisons, malls, islands, freeway corridors and the rest) are in `gta6_area_traffic_mix.csv` alongside these.

## What the map says

A few things fall straight out of the numbers once the places are attached:

- **Downtown runs on taxis.** `region_downtown` is the taxi peak of the whole map: 24% at night, 14% by day, against a 3% median everywhere else.
- **Money shows up as sports cars.** `region_capri` 30%, `region_decordistrict` 27%, `region_bayside` 26%, `region_capriisland` 25% - all against a 4% median. Capri is the richest traffic on the map.
- **Poorer and denser means vans and compacts.** `region_littlecuba` is 24% SUV, 16% vans, 10% taxi and 9% compacts, with no sports cars at all.
- **The swamp is pickups and off-road.** `region_frogswater` and `region_northfrogswater` run 54% pickups and 30% off-road - the single most lopsided mix in the game.
- **Farm country is pickups.** `region_emeraldsprings` 40%, `region_ambrosia` 36%, `region_grassrivers` 32%, `region_balsero` 32%.
- **Suburbs are SUVs.** `region_grayson` 42%, `region_eastlummoxcounty` and `region_millport` 35%, `region_belvilletown` 30%.
- **Plain sedans rule the mid-city.** `region_crosstown` 40%, `region_hamlet` 36%, `region_laperle` 32%.

So yes: GTA VI varies traffic by neighbourhood the way GTA V did, and it does it with 137 schedules over a 113-region gazetteer rather than GTA V's handful of popcycle groups.
