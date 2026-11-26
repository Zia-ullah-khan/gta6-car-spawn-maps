# Evidence

What backs the claims in this repository, strongest first — and, at the end, what is **not**
proven. Every figure here was re-derived from the raw bytes rather than copied from notes.

## Tier 1 — plain text, no inference required

The payload at offset `0x129000` in `common.rpf` is an uncompressed XML `CDataFileMgr__ContentDef`.
It declares, in the clear:

```xml
<Item>
  <filename>platform:/levels/americas/popcycle</filename>
  <fileType>POPSCHED_FILE</fileType>
</Item>
<Item>
  <filename>platform:/levels/americas/ai/popzonespawnerinfo</filename>
  <fileType>POPZONE_SPAWNER_INFO_FILE</fileType>
</Item>
```

So the game itself states that a popcycle file exists and where it lives. Each root `.rpf` mounts
as a directory under `platform:/`, which places it inside `levels.rpf`.

## Tier 2 — the decode, which validates itself

### Finding the file

RPF9 name tables are encrypted in this build, so the file could not be looked up by name. A zlib
payload scan of all 76 GB of `levels.rpf` produced 904,991 payloads, of which 6,982 are PSO
(`PSIN`) files. Exactly one of those has a schema field whose hash equals `joaat("PopGroup")`, and
whose root field hashes to `joaat("schedules")`:

```
levels.rpf @ 0x11BB2B8000   2,147,136 bytes   magic PSIN
```

### The keystone check

Parsing the PSO block table yields a schedule block of 137 entries and a row block of 1,334
records of 88 bytes each. Several independent things must all be correct for that to hold —
record size, the field offset of the percentage array, and the grouping of rows under schedules.
Running it:

```
popcycle PSO: levels_0011bb2b8000.bin  2,147,136 bytes  magic=PSIN
schedules: 137   rows: 1334   sum(per-schedule counts): 1334
columns checked: 1644   columns NOT summing to 100: 0
distinct category hashes: 23
```

Every one of the **1,644 columns sums to exactly 100**. A wrong record size, a wrong field offset
or a wrong schedule-to-row grouping produces noise, not 1,644 exact hundreds. This is the single
strongest piece of evidence in the project.

Reproduce it:

```python
import struct
d = open("levels_0011bb2b8000.bin", "rb").read()
blocks, off = [], 0x08
while off + 0x10 <= 0x1000:
    h, sz = struct.unpack_from("<II", d, off)
    if h in (0x70707070, 0) or sz == 0: break
    blocks.append((h, sz)); off += 0x10
cur = (off + 15) // 16 * 16
rec = {}
for h, sz in blocks:
    rec[h] = (sz, cur); cur = (cur + sz + 15) // 16 * 16
ssz, soff = rec[0xC46F7DD8]          # schedule entries, 24 bytes each
rsz, roff = rec[0x1AE5437F]          # rows, 88 bytes each
counts = [struct.unpack_from("<IIII", d, soff + i * 24)[2] for i in range(ssz // 24)]
idx = 0
for cnt in counts:
    tot = [0] * 12
    for j in range(cnt):
        v = d[roff + (idx + j) * 0x58 + 0x48:][:12]
        for k in range(12): tot[k] += v[k]
    idx += cnt
    assert all(t == 100 for t in tot), tot
print("all columns sum to 100")
```

### The hashing is verified, not assumed

A script's `VmProgram` id equals `joaat(script name)`. Checked against two independent scripts the
decompiler was able to name:

| Script | Declared program id | `joaat(name)` |
|---|---|---|
| `smashandgrab.ysc.cs` | `0x7A8EE38D` | `0x7A8EE38D` |
| `pianist.ysc.cs` | `0x9CA8F4B6` | `0x9CA8F4B6` |

Two casings are in play, and getting this split right is what made the rest readable: **schema
field names hash in PascalCase** (`Name`, `Flags`, `Tags`, `Points`, `PopGroup`, `DensityDatas`)
while **content values hash in lowercase snake_case** (`cars_muscle`, `portgellhorn`,
`malibuclub`). An earlier attempt that tried only lowercase matched nothing across 222M candidates
and wrongly concluded the hashes were salted. They are not.

### The geometry decode also validates itself

Three encodings had to be recovered from the bytes:

| Thing | Encoding |
|---|---|
| PSO pointer (8 bytes) | `u16 blockId`, `u8 flags`, `u40 offsetUnits`; byte offset = `offsetUnits × 16` |
| array, member type `0x5c` | `u32 count`, `u32 capacity`, pointer(8) |
| array, member type `0x52` | pointer(8), `u16 count`, `u16 capacity`, 4 bytes padding |
| point pool | the block whose id is `0x00000031`; 16 bytes per point (`float x`, `float y`, 8 unused) |

The check that confirms all of them at once is contiguity of the point pool:

```
shape  outerStartIdx  outerPts  innerRings
  0      0              6         1
  1      12             97        36
  2      277            11        4
  3      305            5         2
```

Shape 0's outer ring occupies point indices 0–5 and its single inner ring 6–11, so shape 1 must
start at 12 — and it does. The pool is consumed contiguously with no gaps across every shape in
the file. A wrong stride or a wrong pointer decode would not land on 12.

### Counts, re-confirmed

```
zone polygons: 200   distinct schedules with geometry: 132 of 137
named places: 199    region_*: 113
places on a specific zone (<5 km²): 141    on a broad fallback: 58
```

## Tier 3 — corroboration that was never an input

These fall out of the decoded data and match independent expectation. They were not fed in, and
they are the reason to trust the place-to-schedule join:

- `region_downtown` is the taxi peak of the entire map (24%, against a 3% median).
- `region_capri` and `region_decordistrict` are the sports-car peaks (30% and 27%, against 4%).
- `region_frogswater`, in the swamp, is 54% pickups and 30% off-road.
- `trucks` and `utility` sit at exactly 0% in the night slots and appear only by day.

## Vehicle performance and fixed spawns

This part needed no reverse engineering at all — both sources ship as plain XML.

`CVehiclePerformanceMaps` (`common.rpf` payload `0x612000`) gives 251 vehicles with `topSpeed`,
`accelZeroTwenty`, `accelZeroSixty`, `brakingSixtyZero`, `turnRadius` and `maxGforceLateral`.
`CVehicleBaseRatingsAndEstimatesMap` (payload `0x616000`) is a second, independent file with
road and offroad estimates for 231 of them. **The two agree on the ordering** — `thrax` is first
in both, followed by the same group — which is the main reason to trust either.

**Road-capable is derived, not assumed.** Aircraft and boats have `brakingSixtyZero` and
`turnRadius` of exactly 0 in the performance file, so the 203 road vehicles separate out of the
data rather than from a hand-written list. `lazer` (the jet), `valkyrie` and `buzzard` fall out
correctly.

**Spawn locations are quoted, not paraphrased.** They are the game's own description strings,
matched to internal model names. The matching has a real failure mode: a first pass that looked
for the model name anywhere in the text attached a bird description — *"a small wading bird…
the double black bands around its breast"* — to the model `double`. The rule was tightened to
require the model name to appear **capitalised**, as a proper name, plus an exclusion list for
wildlife and weapon descriptions. That dropped 76 non-vehicle strings and left 138 descriptions
across 78 models. Anything surprising in that column is still worth a spot-check.

Areas come from `GRID_SML_L_VEH_<model>_UNIQUE_<area>_…` tokens in the progress dataset (158
unique-vehicle catalog entries), matched against the gazetteer.

## What is NOT proven

- **Schedule names are hashed.** Exactly one was ever recovered, `rialto_islands_region_14`. A zone
  is therefore tied to a place by *geometry*, never by name. If the containment model is wrong, the
  labels are wrong.
- **"Smallest containing polygon wins" is a model, not a fact from the files.** It matches how zone
  priority normally works in this engine family, but the data does not state it.
- **58 of 199 places** resolve only to a broad 5–56 km² county or highway zone. Those assignments
  are weak; the atlas marks them `R` and the specific ones `S`.
- **5 of 137 schedules** have no polygon at all.
- **6 of 23 category hashes are uncracked.** For `#ed3327f7` a 30.7M-candidate attack returned
  `goosekey_haulage_leonard`, one hit against roughly 0.04 expected false positives. A follow-up
  search for sibling names in the same pattern found none, which weakens it. Treat it as
  unconfirmed.
- **The 12 slots have no stated clock hours.** Describing slots as "night" or "day" is read off the
  pattern in the data, not taken from the file.
- **`topSpeed` units are inferred.** The field is unlabelled. The values behave like metres per
  second (76.6 for the fastest hypercar is 276 km/h), so the km/h figures are `× 3.6`. The
  *ordering* does not depend on this.
- **Spawn descriptions give a place in words, not coordinates.** Exact parking spots would need
  the `VEHICLEGEN_OVERVIEW_FILE` named in the content manifest, which is not decoded here.
- **Story availability is not modelled.** A vehicle existing at a spot does not mean it is
  reachable at that point in a playthrough.
- Hash cracking has a false-positive floor. Any single match from a very large candidate space is
  probabilistic. Where that matters it is flagged above rather than presented as settled.
