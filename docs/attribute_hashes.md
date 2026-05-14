# NFSMW attribute hash registry

The `attributes.bin` file (VPAK, 689,728 bytes) holds the gameplay attribute schema. Starting at file offset `0x18000`, every 16 bytes is one attribute row:

```
[+0x00..+0x03]  u32  attribute_name_hash  (Jenkins mix3, seed 0xABCDEF00)
[+0x04..+0x07]  u32  type_hash             (one of the EA::Reflection::* / Attrib::* types below)
[+0x08..+0x09]  u16  ?
[+0x0a..+0x0b]  u16  ?
[+0x0c..+0x0d]  u16  ?
[+0x0e..+0x0f]  u16  type/flags
```

Total attributes: **345** rows. Cracked: **98** (28%).

## Type hashes

EA's reflection system + their custom Attrib namespace types:

| Hash | Name | Count |
|------|------|-------|
| 0x3C16EC5E | EA::Reflection::Float        | 137 attributes |
| 0xA3F0C234 | EA::Reflection::Text         | 39 attributes |
| 0x939992BB | EA::Reflection::UInt32       | 38 attributes |
| 0x064BEC37 | EA::Reflection::Bool         | 71 attributes |
| 0x2B936EB7 | Attrib::RefSpec              | 38 attributes |
| 0xA502A824 | Attrib::StringKey            | 33 attributes |
| 0xDB9D3A16 | eDRIVE_BY_TYPE               | 5 attributes |
| 0x349D3A16 | (uncracked)                  | ? |
| 0x934A36EC | (uncracked)                  | ? |

Other reflection types (referenced as values, not used as schema-row types):
0x5763DA41 = EA::Reflection::Int32, 0x6F27B5BC = Int8, 0x671ECBE2 = UInt8, 0xE51A99C1 = UInt16,
0x34FDE6BB = Attrib::Types::Vector4, 0xA1E54784 = Attrib::Types::Matrix, 0xD680287D = Attrib::Types::Vector3.

## Float (30/137 cracked)

Camera/Rendering: FOV, Radius, Width, HEIGHT, ANGLE, MinSize, Life, Power, Rotation, distance, LAG, TOD, ColourBloomIntensity, BlackBloomIntensity, DetailMapIntensity, Desaturation
Vehicle/Physics: MASS, STEERING, AxlePair
Game logic: LeaderSupport, SpeedHighway, DamageScaleRecord, CopCountRecord, Priority

## RefSpec (23/38 cracked) — Subsystem references

These are the major vehicle/engine subsystem records: **engine, transmission, brakes, tires, nos, induction, chassis, rigidbodyspecs, engineaudio, damagespecs, chopperspecs, junkman, aivehicle, frontend, acceltrans, Trailer, EffectLinkageRecord**.

This is a complete vehicle data model:
- **engine** + **transmission** + **acceltrans** (acceleration transmission)
- **chassis** + **rigidbodyspecs** + **damagespecs**
- **brakes** + **tires** + **nos** + **induction**
- **engineaudio** for audio
- **junkman** (the meta-tier system that stacks performance bonuses)
- **chopperspecs** (helicopter)
- **Trailer** (truck trailers)
- **aivehicle** (AI-only data)
- **frontend** (FE preview)

## StringKey (15/33 cracked) — Behavior mechanic registry

The 9 BEHAVIOR_MECHANIC_* keys define the entire vehicle-component system:

```
BEHAVIOR_MECHANIC_SUSPENSION
BEHAVIOR_MECHANIC_EFFECTS
BEHAVIOR_MECHANIC_RESET
BEHAVIOR_MECHANIC_DAMAGE
BEHAVIOR_MECHANIC_RIGIDBODY
BEHAVIOR_MECHANIC_ENGINE
BEHAVIOR_MECHANIC_DRAW
BEHAVIOR_MECHANIC_INPUT
BEHAVIOR_MECHANIC_AUDIO
```

Plus: DESCRIPTION, CLASS, MODEL, StitchCollisionVol, STICH_COLLISION_TYPE.

These are the 9 "behaviors" attached to every pvehicle subobject (matches the pvehicle inventory at 0x008add1c).

## UInt32 (8/38 cracked)

CarID, COST_TO_STATE, EffectLinkageRecord (and 5 type-self-refs).

## Bool (9/71 cracked)

CatchUp (rubber-band on/off), Directional, Tranny, Persistent.

## Text (9/39 cracked)

Name, EventID, MilestoneName, Region, Barriers, ZoneType, ParticleTextureRecord.

## 5 mystery hashes (still uncracked, all Float-typed)

`0xEE0011E3, 0xB5C0DAC8, 0xDA5F19F9, 0x360552DA, 0x44F1273B`

These attribute name strings appear nowhere in any shipped bundle (BUN/LZC/BIN/EXE/DLL). They're likely attribute names that were used at compile time only — strings stripped from the release build but the hashes survive in `attributes.bin`.

## How to use

To find the type and offset of any attribute name:
1. Hash the name with bChunk (Jenkins mix3, seed `0xABCDEF00`).
2. Search `attributes.bin` for the hash as little-endian u32.
3. Read the 16-byte row containing it; the second u32 is the type hash.

Example: `bChunk("MASS")` = `0x4A56503D`. Searching attributes.bin finds the entry; type = `0x3C16EC5E` (Float). Confirms MASS is a Float-typed attribute.

## Cracking script

`/tmp/crack_floats.py` and the heredoc in this session — extract strings from all bundles + speed.exe + DLLs (~6.9M unique strings), generate name variants, hash against the attribute table. Per-type recovery rates:
- RefSpec: 60% (subsystem ids tend to appear in code as data-section strings)
- StringKey: 45%
- Float: 22%
- Text: 23%
- UInt32: 21%
- Bool: 13%
