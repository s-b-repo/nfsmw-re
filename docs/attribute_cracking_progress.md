# Attribute hash cracking progress (live)

Total attribute name hashes in `attributes.bin`: **345**
Cracked: **108** (31%)

## New cracks this session

### Round 1: From bundle strings
30 Float-typed (FOV/MASS/STEERING/Power/Life/etc.), 23 RefSpec subsystems (engine/transmission/brakes/etc.), 15 StringKey BEHAVIOR_MECHANIC_*, 9 Bool, 8 UInt32, 9 Text, 4 eDRIVE_BY.

### Round 2: Compound vocabulary
- 0x0A91596D = `InitialSpeed`
- 0x3AC8B868 = `MinRPM`
- 0x7F8EEA1A = `STIFFNESS`
- 0xA4D878EF = `MaxRPM`
- 0xC2094707 = `YAW_SPEED`
- 0xF7649E63 = `MaxSize`
- 0x2C44FF10 = `ResetTime`
- 0x3A0E4B19 = `InitialPlayerSpeed`
- 0x5F84F834 = `RESPAWN_TIME`
- 0x7585F041 = `TimeLimit`

## Still uncracked (237 attribute names)

Mix of Float (101 left), Bool (62 left), Text (30 left), UInt32 (30 left), RefSpec (15 left), StringKey (18 left).

Including the original 5 mystery hashes:
- 0xEE0011E3 (used in `RigidBody_FactoryFromAttributes_PhysicsPoolAlloc @ 0x6895a0` — likely `RigidBodyAttribute` or similar)
- 0xB5C0DAC8
- 0xDA5F19F9
- 0x360552DA
- 0x44F1273B
