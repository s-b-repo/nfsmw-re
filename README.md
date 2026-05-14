# NFS Most Wanted (2005) — Reverse Engineering Notes

Reverse engineering documentation for `speed.exe` from Need for Speed: Most Wanted (2005, PC).

Build analyzed: PE32 i386, MSVC 7.10, image base `0x00400000`, 26,316 functions, **6,511 named (24.7%)** as of 2026-05-14. Captured live under Wine 11.7 against the Magipack v5 repack (with WidescreenFix, ExtraOptions, HDReflections, HUDAdapter ASI mods + DSOAL DSOUND wrapper).

No game binaries are redistributed — bring your own copy.

## Repo layout

```
docs/                        Main reference docs (markdown + JSON)
├── ARCHITECTURE.md          Canonical subsystem map
├── PROGRESS.md              Wave-by-wave RE progress log
├── ANTI_RE_AND_PATTERNS.md  Defensive techniques + coding patterns + corrections
├── DEBUGGER_PLAN.md         How the Wine + Ghidra GDB-remote workflow was set up
├── renames.csv              Full export of 6,511 named functions (addr, name, xref_count)
├── attribute_hashes.md      Cracked StringHash32 attribute names
├── attribute_cracking_progress.md
├── GHIDRA_MCP_*.md          Reference docs for the Ghidra MCP plugin tools used
├── sdk_addrs.json           181 hardcoded addresses extracted from NFSPluginSDK
├── sdk_enums.json           65 enums extracted from NFSPluginSDK
├── sdk_structs.json         245 structs extracted from NFSPluginSDK
├── nfsplugin_sdk_mw05/      Mirror of berkayylmao's BSD-3 NFSPluginSDK headers
└── extracted_strings/       Raw string dumps from key bundles (FrontB, GlobalB, InGameB, gameplay)

notes/                       Persistent subsystem notes (40 files)
├── MEMORY.md                Index of all notes
├── project_*.md             Subsystem deep-dives (physics, AI, audio, cutscenes, etc.)
├── reference_*.md           Pointers to external resources (SDK, MCP docs)
└── feedback_*.md            RE workflow lessons learned

speed_renames.txt            Earlier renames snapshot (2026-05-08, 1,313 names)
launch-debug.sh              Wine + winedbg --gdb launcher (port 38002)
launch-then-attach.sh        Alternative: launch then attach by PID
```

## Subsystems mapped

Engine internals (EAGL family):
- **Physics** — worker-thread integrator; pvehicle dispatch chain; AggregatePhysicsState @ 0x694160 (leaf state collector)
- **Render pipeline** — D3D9 boot + per-frame pass order (Clear → ShadowMap → World → HUD → PostProcess → Present)
- **Animation runtime** — EAGL4Anim + TickableBus
- **Allocator** — 16-arena heap → CreateObjectPool → 50+ named slot pools
- **World streamer** — region-grid; AI path graph
- **Particle system** — 3 pools (Particle 1024×0x70, Emitter 500×0x90, EmitterGroup 200×0x80)
- **Audio** — NFSMixMaster + 4 entity classes
- **Input** — DInput8 → action enum → AI/PlayerCar same controller
- **Cutscenes** — On2 VP6 + MAD audio in custom `.vp6` chunks (not Bink)

Gameplay:
- **AI architecture** — AIGoals (strategic) + AIActions (tactical) hierarchy
- **AI racer** — no drafting, no AI nitrous; rubber-band speed-mult only
- **Cop AI + pursuit** — goal-stack model; ground/heli pursuit dispatch
- **Race rules** — mode enum 0–10 (Drift confirmed absent)
- **GameBreaker** (the engine's name for SpeedBreaker) — dt scaling site
- **Career + milestones** — event-broadcast model
- **Damage** — data-driven via mission flags
- **Customization** — vinyls + parts (body/engine/exhaust/suspension/brakes)

Frontend / meta:
- **FE engine** — screen-stack manager keyed on `.fng` asset names
- **HUD** — 28 named widgets as passive FNG-tree nodes
- **EA Trax** — interactive score + licensed tracks (streamed via EAXS_StreamManager)
- **Network / multiplayer** — host-authoritative GameSpy + LAN
- **Replay / Joylog** — Joylog (deterministic capture) + CarReplay (per-vehicle state)
- **Save / load** — MD5-hashed profile; ~63KB layout
- **Script natives** — ~150 C++ natives bound for the script VM
- **Script VM** — **vanilla Lua 5.0.2** (confirmed by error strings)

Formats:
- **JDLZ v0x02** decompressor RE'd (`tools/nfsmw_bun_reader/jdlz_nfsmw.py` byte-perfect)
- **bChunk** hash = Bob Jenkins 1996 mix3 (seed `0xABCDEF00`), NOT DJB2
- **attributes.bin** schema: 16-byte rows from `0x18000`; 7/9 type hashes cracked

See `docs/ARCHITECTURE.md` for the canonical write-up and `notes/MEMORY.md` for the index of per-subsystem notes.

## Tooling used

- **Ghidra** with a custom MCP server (177 tools) for AI-driven analysis
- **Wine 11.7** + **winedbg --gdb** for live debugging
- **Ghidra MCP** + **Claude Code** for parallelized RE waves
- **berkayylmao's NFSPluginSDK** (BSD-3) — mirrored locally for offline reference

## Credits

- berkayylmao — [NFSPluginSDK](https://github.com/berkayylmao/NFSPluginSDK) (BSD-3), the type-name/address source for 120 functions + 16 globals applied in Ghidra
- ThirteenAG, Lyall — WidescreenFix, ExtraOptions, HUDAdapter (the mod surface that informed several anchors)

## License

Documentation: CC-BY-SA-4.0. Code snippets / scripts: MIT. Mirrored NFSPluginSDK headers retain berkayylmao's BSD-3 license (see `docs/nfsplugin_sdk_mw05/LICENSE.berkayylmao`).

This repository contains no copyrighted EA assets, no decompiled source, and no game binaries — only original analysis notes, function-name/address tables, and references to publicly available SDKs.
