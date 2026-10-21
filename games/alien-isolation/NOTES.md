# Alien: Isolation — agent notes

Game: Alien: Isolation (2014, Creative Assembly / SEGA).
Owner goal: **online co-op with friends** (splitscreen out of scope unless asked later).

## Phase 0 — Triage (2026-10-20)

### Observations (verified on this machine)

- Store/version: **Steam**, appid 214490, **buildid 4940658** (from `appmanifest_214490.acf`,
  `TargetBuildID` = `buildid` = 4940658, StateFlags 4 = fully installed, 2026-10-20).
- Install dir: `G:\SteamLibrary\steamapps\common\Alien Isolation` (owner's copy —
  read-only for us; test copies will be made per ground rule 2).
- Main exe: `AI.exe`, 21,822,464 bytes,
  SHA-256 `DDE66521798AAE8C39ED4E7A3400F561FCECFFF83619D1426B9E0CEDF5819A90`.
- Top-level layout: `AI.exe`, `DATA\`, `Extras\`, `AMD_AGS.DLL`, `ATIAGS.DLL`,
  `CRASHREPORTER32.DLL`, `STEAM_API.DLL`, `BENCHMARK_TEMPLATE.TXT`.
- Engine: Creative Assembly proprietary ("Evolution" engine; no Unreal/Unity/CryEngine
  DLLs present). 64-bit (Steam build). See engine-families.md — no sibling co-op
  framework known for this engine.
- OS: Windows 10/11 (PowerShell 5.1 environment), VS 2026 Community + C++ workload
  installed (from prior sessions, see `local/`).

### Triage answers

1. Exact game: Alien: Isolation, 2014, Steam build 4940658. ✔
2. Already done? Per `DIRECTORY.md`: one prototype (Dropper7894, announced, never
   released). Research below — **no downloadable co-op mod found** as of 2026-10-20.
3. Engine: Creative Assembly proprietary. DLL-level modding only.
4. Source available: **nothing public** — closed binary. Heavy RE expected. No decomp
   project found (searched "alien isolation decomp", GitHub topics `alien-isolation`).
5. Engine multiplayer: **none** — the game is single-player only (Survivor Mode is a
   scoring challenge mode, not netcode). No replication layer to reuse → must build a
   sync layer (host authority + state sync).
6. Shipping check: mod must ship as a separate package (DLL proxy/hooks + scripts),
   no game files. No anti-cheat, no online service in the campaign (Steam works
   overlay only) → safe to hook offline play. Must not touch Steam multiplayer of
   other games. OK.

**Verdict: Path 2 — build from zero** (nothing usable exists to revive). Difficulty:
closed binary, no MP, no scripting language shipped → "very hard" row of the triage
table. Start with research + design doc, look harder for collaborators/existing work.

## Phase 1 — Research (2026-10-20, in progress)

### Existing attempts

1. **Alien Isolation co-op (Dropper7894)** — announced 2026-06-30 with gameplay
   video; goal: turn **Survivor Mode into 2-player co-op**. No release, no repo, no
   ETA. Author runs a Discord for progress updates: https://discord.gg/UhHCPMdMh
   (joining = contacting people → needs owner's permission, ground rule 1).
   - https://bloody-disgusting.com/video-games/3957869/fan-made-co-op-mod-for-alien-isolation-announced/
     (article fetched 2026-10-20, dated June 30 2026 — confirms Discord link + goals)
   - https://www.dsogaming.com/videotrailer-news/first-look-at-upcoming-multiplayer-co-op-mod-for-alien-isolation/
   - https://shazoo.ru/2026/06/30/186490/modder-sozdaet-kooperativnyi-rezim-dlia-alien-isolation (2026-06-30)
   - Reddit r/alienisolation "A small mod I'm working on"
     (https://www.reddit.com/r/alienisolation/comments/1uhr3gr/, ~384 upvotes,
     "short demo of a multiplayer mode… still far from being a full co-op mode") —
     this IS the Dropper7894 demo post (linked from the BD article). Direct fetch
     403'd; content confirmed via article quotes.
2. **Echo Protocol** — fan-made 2-player co-op announced in r/modding
   (https://www.reddit.com/r/Modding/comments/1jyz9dq/, ~2025-03). No repo, no
   release, no recent activity found by web search. Status: **unknown, looks silent**.
3. Steam thread "has anyone played the alien isolation co-op mod"
   (https://steamcommunity.com/app/214490/discussions/0/598198356181958694/): top
   reply says the linked "co-op" is a Garry's Mod gamemode, not an A:I mod. Treat
   player chatter as low-signal here.
4. **Nucleus Co-op** (splitscreen tool, open source) supports many games; A:I support
   **unknown** — would only matter for splitscreen, currently out of scope.
5. **OpenCAGE** — https://github.com/MattFiler/OpenCAGE (MIT, 356★, 1024 commits as of
   2026-10-20, actively maintained; also a free Steam tool app 3367530; site
   opencage.co.uk). Full modding toolkit for A:I: level logic scripts ("Composites",
   XML/BML format), **behaviour trees** (Brainiac Designer editor), character/item
   configs, assets. Built on MattFiler/CinematicTools (Hattiwatti's in-process
   camera/tool injector for A:I) + x1nixmzeng/AlienBML (file format parser).
   Repo page fetched 2026-10-20. This is the strongest foundation on this engine:
   the game's logic layer is data-editable without native code, and CinematicTools
   proves DLL injection into AI.exe works.
6. **aliasIsolation** (TAA mod): https://github.com/aliasIsolation/aliasIsolation
   — another proven AI.exe injection precedent (method not yet verified).
7. Engine families: `playbook/engine-families.md` has **no entry** for Creative
   Assembly's Evolution engine → no sibling co-op framework to port. Possible
   playbook gap to note in the PR.

### Derived (not yet verified)

- No netcode, no native source, no decomp → the sm64coopdx / KeeperFX routes are
  closed; the realistic model is **host authority + state sync via injected DLL**
  (the fnv-coop / Skyrim Together family). Level logic ("Composites") is editable
  via OpenCAGE, which may carry story/quest sync at the data layer instead of the
  native layer — to evaluate in Phase 4.
- Game AI runs on behaviour trees (Brainiac Designer lineage) — AI ownership sync
  will be the hard part (see playbook/actor-ownership.md).

### Unknowns / next questions

- Does Dropper7894 publish anything (YouTube channel, GitHub, Nexus)? Owner may
  join the mod's Discord (discord.gg/UhHCPMdMh) to ask about collaboration
  (ground rule 1 — contacting people needs permission; also Discord mining requires
  playbook/discord-export.md).
- Whether the game's own save format / script tables can be driven externally.
- Legal/licensing: SEGA's stance on A:I mods (community has modded openly for years;
  no C&D known — verify before shipping).

### Checks that ran

- `appmanifest_214490.acf` read (build identity). ✔
- `Get-FileHash AI.exe -Algorithm SHA256`. ✔
- GitHub issues/PRs of cranebeaker/coop-anything searched for "alien": 0 results. ✔
- Web searches (2026-10-20): "alien isolation co-op mod", "Echo Protocol alien
  isolation", "Dropper7894 github", nexusmods multiplayer, github topics
  `alien-isolation`. Reddit direct fetches blocked (403) — noted per research.md.

## Phase 2 — Workspace (2026-10-20)

- Test copy: robocopy of the owner's install (read-only source) →
  `<workspace>\test-install\Alien-Isolation`, 389 dirs / 7,586 files / 30.17 GB,
  0 failures (2026-10-20 21:43). Copy's `AI.exe` SHA-256 **matches** the owner's
  install (DDE66521…9A90). Owner's install and saves untouched (ground rule 2).
- Owner approved: gh sign-in, test copy, "not now" for contacting Dropper7894.

### First look at the data layer (read-only, owner install)

- `DATA\` layout: 301 `.PAK` archives + loose files. Key dirs: `BEHAVIOR\`,
  `BINARY_BEHAVIOR\` (compiled behaviour trees), `ALIENCONFIGS\`, `CHR_INFO\`,
  `GLOBAL\`, `PACKAGES\`, `TEXT\`, `UI\`.
- Loose logic/config files are **both** `.XML` and `.BML` pairs (e.g.
  `GBL_ITEM.XML` 236 KB / `GBL_ITEM.BML` 143 KB, `ENGINE_SETTINGS.XML`,
  `INPUT.XML`) — consistent with OpenCAGE's AlienBML (XML source → compiled BML).
  Confirms the game's logic layer is editable at the data level.
- Native side still closed: `.BIN` (compiled behaviour), `.HKX64` (Havok anim),
  `.USM` (movies). No scripting VM docs found → native RE still needed for
  runtime sync (Phase 3+).

## Phase 3 — Understand (2026-10-20, in progress)

### Static recon on the test copy (`test-install\Alien-Isolation\AI.exe`)

- **PE: 32-bit x86** (machine 0x14C, "32 bit word machine"), link timestamp
  Tue Feb 17 2015 10:36:13, 5 sections only (`.data .rdata .reloc .rsrc .text`),
  `Dynamic base` (ASLR). No SteamStub `.bind` section. Denuvo status **unknown**
  (check `.text` entropy before any decomp plan; deferred). Verified with
  `dumpbin /headers`. (Recorded in case other write-ups assume 64-bit.)
- **Imports (dumpbin /dependents):** `steam_api.dll`, KERNEL32, USER32, **WS2_32,
  WSOCK32** (Winsock present — telemetry/LAN? to investigate), PSAPI, imagehlp,
  SETUPAPI, **dxgi, d3d11** (D3D11 renderer), XINPUT1_3, GDI32, SHELL32, ole32,
  WINMM, ADVAPI32, WLDAP32.
- **Derived:** no `dinput8.dll` import → the dinput8-proxy entry point used on
  Resident Evil/Far Cry 2 in this hub **does not apply here**. Candidate load
  vectors instead: `dxgi.dll` proxy (loaded by any D3D11 app; also gives a Present
  hook) or `steam_api.dll` proxy (classic, game must find real steam_api). 32-bit
  process ⇒ x86 DLLs, 32-bit pointer space.
- Python 3.12.10 located at `%LOCALAPPDATA%\Programs\Python\Python312\python.exe`
  (installed via winget; **not on this session's PATH** — use full path).

### Next (Phase 3)

1. Entropy check on `.text` (Denuvo?) before planning any decompile.
2. Boot smoke test of the test copy; watch `%USERPROFILE%\Documents` for new
   config/save dirs (per reverse-engineering.md, move them into the workspace).
3. Study OpenCAGE/CinematicTools source for their injection method + how composites
   are read; reuse rather than reinvent.
4. Get Ghidra headless (pinned download, ask owner) for the runtime seam map.

### Boot smoke test (2026-10-20, test copy, owner approved)

- Launch: `Start-Process AI.exe` — the started pid **exited immediately**; a second
  `AI.exe` process owned the window ("Alien: Isolation" title) ~40 s later (intro
  movies). Find: track by process name, not launcher pid.
- Runtime: process stable, ~1.2 GB working set, responsive. Steam must be running
  (steam.exe was up; direct launch worked without Steam launch-integration prompt).
- Filesystem footprint of a bare boot + graceful exit (CloseMainWindow): **nothing**
  written to Documents, `%APPDATA%`, `%LOCALAPPDATA%`, HKCU\Software (checked:
  My Games, Feral Interactive, `Creative Assembly` key — all absent before and
  after). Settings/saves presumably appear only after a settings change or
  checkpoint; revisit redirection (junction/env) when we drive real gameplay.
- Smoke test result: **boot verified on build 4940658** (QA: boot test only).

### CinematicTools injection method (read 2026-10-20, repo page)

- Now maintained at https://github.com/OpenCAGE/CinematicTools (fork of
  Hattiwatti/CinematicTools). "A DLL for Alien: Isolation which hooks various
  functionality to provide cinematic tools."
- Build: **Win32/x86**, VS2022 v143, NuGet packages: **MinHook** (hooking),
  DirectXTK, Boost. Output `Build\CT_AlienIsolation.dll`.
- Load vector: **runtime injection** via https://github.com/nefarius/Injector
  (`inject.bat` after the game is running) — *not* a proxy DLL. So the proven
  community pattern is: external injector + hooking DLL (MinHook) into the live
  32-bit process; hooks D3D11 among others.
- Derived: our Stage 1 can copy this exact pattern (x86 MinHook DLL + nefarius
  Injector or our own CreateRemoteThread loader) instead of inventing a proxy.

### Toolchain installed (2026-10-20, owner approved)

- Ghidra 12.1.4 PUBLIC → `<workspace>\tools\ghidra\ghidra_12.1.4_PUBLIC`.
  Zip sha256 (pinned; release publishes no checksum):
  `ddac49f903da9d5bac833e5cc79395098b9c33cfd3279be5f31bd00387d2d4db`
  (source: github.com/NationalSecurityAgency/ghidra release Ghidra_12.1.4_build).
- Temurin JDK 21.0.12.1 → `<workspace>\tools\jdk\jdk-21.0.12.1+1`.
  Zip sha256 **verified against Adoptium's published checksum**: `f9d6e191…d8b4e`. ✔
- `analyzeHeadless.bat` smoke-tested OK with `JAVA_HOME` set (2026-10-20).
- Zips kept in `<workspace>\coop-anything-cache`.

### Entropy check (2026-10-20, tool: `coop-anything-cache\pe_entropy.py`)

- Per-section Shannon entropy of test-copy `AI.exe`: `.text` **6.710**, `.rdata`
  5.147, `.data` 4.931, `.rsrc` 5.651, `.reloc` 5.783.
- Verdict: **no packing / no SteamStub** (packed code sits near 8.0 per triage.md).
  Plain decompilation of build 4940658 is viable. (Whether any Denuvo VM functions
  survive inline is unknown; the code section statistics say the bulk is normal
  compiled x86.)
- Ghidra 12.1.4 headless import of `AI.exe` + full analysis + DecompileAll.java
  (from cranebeaker/deus-ex-coop, read before running) launched 2026-10-20 →
  outputs `<workspace>\decomp\AI.exe.c` + `AI.exe.functions.tsv`.

### Full decompile + seam hunt (2026-10-20)

- **Analysis succeeded:** 456 s auto-analysis; **66,811 functions decompiled, 10
  failed** (9 of those "Unable to read bytes at ram:0x03030303/0x020202" —
  data-as-code false positives). Outputs: `decomp\AI.exe.c` (69.9 MB),
  `AI.exe.functions.tsv` (67,005 lines incl. header), plus
  `decomp\AI.exe.strings.txt` (62,670 strings ≥6 chars, via
  `coop-anything-cache\strings_dump.py`).
- **Engine identity: CATHODE** — RTTI class names in the exe use namespace
  `CATHODE` (e.g. `.?AVIsMultiplayerMode@CATHODE@@`). Creative Assembly's Cathode
  engine. *Triage correction: "Creative Assembly proprietary" in the directory
  should say "Creative Assembly Cathode".*
- **Script seam found:** level-logic events dispatch by string name through vtable
  calls, e.g. `FUN_00827400` sends `"SetPlayerPos"` (with floats) to the object at
  `this+200` under a critical section; siblings `"SetPlayerRotation"`,
  `"Map_SetPlayerPos"`, `"SetIsCurrentArea"`, `"ClearBreadcrumbs"`, `"show_player"`,
  `"hide_player"`, `"release_player"`, `"allow_player_to_skip"`. RTTI shows typed
  command classes: `SetPlayerHasKeycard@CATHODE`, `GetPlayerHasGatingTool@CATHODE`
  etc. → the Composite script layer has a **name→handler command table**; string
  lookup helper `FUN_008572a0` = hash(name) scanned over 20-byte records.
- **Multiplayer status: NO game netcode in this binary.** Evidence:
  - All `connect/send/recvfrom/WSAStartup` hits trace to **libcurl**
    (`curl_easy_send`, FTP/SMTP/TFTP strings) and **Havok visual debugger**
    ("Server created and will poll for new client(s) on port…", hkBase) —
    debug/HTTP only.
  - Dormant script-layer multiplayer *concepts* exist (engine heritage):
    RTTI `.?AVIsMultiplayerMode@CATHODE@@`, strings `IS_MULTIPLAYER_MODE`,
    `MULTIPLAYER_MOTIVATION`, `Multiplayer_Communications`, and a diagnostic
    "…this is okay in multiplayer, but suspicious if it triggers constantly".
  - Derived: A:I itself never shipped MP; but **Cathode as an engine family did**
    (Halo Wars 2, same engine, shipped multiplayer) — a possible reference binary
    for how sessions/replication were structured in Cathode. `engine-families.md`
    has no Cathode entry → playbook gap to propose in our PR.
- Main loop candidates: `PeekMessageA` loops at `AI.exe.c` line 873653 (engine
  pump) and 1244001; `PeekMessageW`/`PostQuitMessage` at 2452334+ (likely bundled
  third-party, e.g. crash/HTTP lib). Far Cry 2 playbook step (patch the message
  pump imports) maps onto line 873653's loop.

## Stage 1 — Load vector + pump probe (2026-10-20, TEST COPY ONLY)

**Correction to the earlier smoke-test note:** the "second AI.exe pid" behavior was
NOT a self-spawn — the test-copy exe calls `SteamAPI_RestartAppIfNecessary`
(import #A of `steam_api.dll`), which hands off to the running Steam client, and
**Steam relaunched the game from the registered real install (G:)**. So the first
smoke test actually ran the owner's real install (bare boot, verified 0 files
written — but still not the intent). Recorded here to keep the trail honest.

### Build

- `ai-coop\stage1\dxgi_proxy.cpp` + `dxgi_proxy.def` — a **dxgi.dll proxy**,
  x86 (cl /LD /O2 via vcvarsall x86), no third-party deps. Source: workspace.
- Forwards to real `C:\Windows\System32\dxgi.dll` (lazy LoadLibrary on first
  export): `CreateDXGIFactory`, `CreateDXGIFactory1`, **`CreateDXGIFactory2`**,
  `DXGIGetDebugInterface1`, `DXGIDeclareAdapterRemovalSupport`.
  - First run died `0xC0000139` "CreateDXGIFactory2 could not be located in
    C:\Windows\System32\d3d11.dll": the system 32-bit **d3d11.dll also imports
    from dxgi.dll** (its only dxgi import is `CreateDXGIFactory2`, verified with
    `dumpbin /imports C:\Windows\SysWOW64\d3d11.dll`) and it resolves against
    OUR proxy in the app dir. Lesson for any dxgi proxy: forward everything the
    in-process system consumers need, not just the game's own imports.
- On `DLL_PROCESS_ATTACH`: creates `<test dir>\stage1-probe.log`, IAT-patches
  the main module's `USER32.dll` imports (`PeekMessageA`, `PeekMessageW`,
  `PostQuitMessage`) with logging wrappers (Far Cry 2 playbook pattern — plain
  PE parsing + in-place thunk rewrite, no MinHook needed), snapshots loaded
  modules. No `LoadLibrary` work in DllMain (real dxgi deferred to first export).

### Steam handoff fix (required to stay in the test copy)

- `$env:SteamAppId = '214490'` before `Start-Process` suppresses the
  `SteamAPI_RestartAppIfNecessary` handoff. Verified: exactly 1 AI process,
  path = test copy, no G: launch. (Alternative if ever needed: a steam_api.dll
  proxy forcing that function to return FALSE.)

### Run results (assertions from `stage1-probe.log`, 2026-10-20 22:59)

| Check | Result |
|---|---|
| Probe attaches in test-copy process | ✔ attach line, correct path, pid 49788 |
| IAT patches | ✔ 3/3 "OK" |
| dxgi forwarding | ✔ `CreateDXGIFactory/1` calls `hr=00000000` (renderer initialized) |
| Message pump observed live | ✔ **PeekA=27,458 ticks** over 72.7 s uptime, log ends with detach line |
| Process count during run | ✔ exactly 1, from the test copy |
| Real install footprint | ✔ 0 files modified in G: install during whole experiment |
| Exit | ✔ graceful (CloseMainWindow), detach line written |

- Deployed artifact: `test-install\Alien-Isolation\dxgi.dll`
  (sha256 recorded in session log; rebuilt versions overwrite this copy of our
  own file — nothing of the game's is ever replaced: dxgi.dll did not exist in
  the install).
- QA level for this stage: **manual boot test with log assertions** (not yet an
  automated suite). No bots yet (playbook/automated-qa.md is Phase 6).

### Stage 2 candidates (next)

1. Hook the named-event firehose (script seam `FUN_0050e4f0` / command table) to
   log Composite commands as they fire in gameplay.
2. Locate and log the player position/entity each N frames (memory seam —
   reverse-engineering.md "find a value in memory" dance in the live process).
3. Both on top of the existing proxy; still test-copy only.

## Stage 2 — Load-vector correction + seam probe (2026-10-21)

### The dxgi-proxy load vector does not reach gameplay

- The game **re-execs itself from the registered install path**: on launch, the
  test-copy `AI.exe` calls `SteamAPI_RestartAppIfNecessary`, Steam relaunches from
  `G:\SteamLibrary\steamapps\common\Alien Isolation`, so a `dxgi.dll` dropped in
  the test copy is never loaded by the process that actually runs gameplay.
- Tried a registry redirect
  (`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\Steam App 214490\InstallLocation`)
  → **did not work**; Steam resolves the install via
  `libraryfolders.vdf` + `appmanifest_214490.acf`, not that key. Registry was
  **restored** to the `G:` path. Getting the test copy to run under Steam needs
  a second Steam library / a copied app manifest — an install-level change we did
  not make.
- **Adopted load vector (proven):** external x86 injector
  (`injector.exe`, `CreateRemoteThread` + `LoadLibraryA`) targets the already
  running `AI.exe` and loads `ai_probe.dll`. No install mutation at all; works
  against the `G:` process exactly as it would against the test copy.
  This matches the community pattern already proven for A:I (CinematicTools uses
  nefarius/Injector + a MinHook DLL — see Stage 1 notes).

### Tooling built (all small, pinned, in-repo)

`ai-coop\stage2\inject\` — `probe.cpp` (+`build_inject.bat`), `injector.cpp`
(+`build_injector.bat`), `snapshotter.cpp`/`regions.cpp`/`pospoll.cpp` (+build
scripts). All x86 via `cl` from VS 2026 Community.

### Probe seams

- **`call_method`** (RVA `0x526680`, thiscall, `ret 0Ch`) — EntityManager entity
  method dispatch. Fires **~58,000 calls / 50 s** in normal play. Arguments are
  smart pointers (caller pushes entity, entity, message). `this` is a stable
  context (e.g. `1C966F80`); entity pointers cluster in `1C59xxxx`. Useful as a
  firehose for the entity set, but **the player's transform is not among the
  entities seen here** (`this+0xC0..0xD4` is a static transform unrelated to the
  player).
- **`send_event`** (RVA `0x10E3B0`, cdecl 5 args) — hooked OK but fires
  **0 events in normal play** (only 2 call sites in the binary). Keep the hook for
  now; the `call_method` stream is the live seam that matters.
- **IAT hooks** (Stage 1, kept): `USER32.dll!PeekMessageA/W`, `PostQuitMessage`
  — 3/3 patch OK, drives graceful shutdown detection.

### Boot fix

- First injected build died `0xC0000142` at load: an uninitialized
  `CRITICAL_SECTION` was used before `InitializeCriticalSection`. Fix:
  `InitializeCriticalSection` first thing in `Init()`, a `g_csInit` guard so
  `Logf` is safe before that, and `Sleep(250)` before `MH_Initialize` (the probe
  thread is created from `DllMain`, so it must not race the loader lock).
- After the fix: `MH_Initialize`/`MH_CreateHook`/`MH_EnableHook` all `MH_OK`,
  renderer initializes, game stays healthy (~1.4 GB working set), graceful exit.

### Load-vector caveat (Defender)

- Windows Defender quarantined the injector **from the project folder** (the
  combination of `CreateRemoteThread` + `LoadLibrary` in the workspace path trips a
  heuristic). An elevated `Add-MpPreference -ExclusionPath` was attempted; the file
  was removed anyway. **Workaround:** run the injector + DLL from
  `%LOCALAPPDATA%\Temp\opencode\probe-inject` — it runs there with no
  interference. Note for anyone reproducing: build in-repo, execute from a trusted
  temp dir.

## Stage 2b — Player world position FOUND (verified)

Method: memory-diff (reverse-engineering.md "find a value in memory" dance).

1. `snapshotter.exe` dumps the private heap region `0x1B000000 + 0x04000000`
   (64 MB) from the live process.
2. Take a snapshot while standing, one after a **deliberate walk**, then a third
   pair with a **short** deliberate walk.
3. Diff 4-aligned float triples between snapshots and rank by how many addresses
   share the **same displacement vector**.

Result — four addresses moved with an identical, coherent displacement of
`(-2.75, 0.0, 1.84)` (≈3.3 units) during the short walk:

| Address | Layout | Role |
|---|---|---|
| `0x1CA467C0` | `[pos][fwd]` | authoritative copy |
| `0x1D5FB2A4` | `[pos][fwd]` | authoritative copy (bit-identical to above) |
| `0x1D5F9B44` | `[pos][fwd][pos...]` | nearby entity, Y ≈ 0.02 lower |
| `0x1D5F9B60` | `[pos][fwd]` | nearby entity, Y ≈ 0.79 lower (standing NPC) |

- **Layout:** `float3 position` immediately followed by `float3 facing`
  (a clean unit vector, e.g. `(-0.513, -0.311, 0.800)`). The position is the
  **first 3 floats** of the record.
- **Verified three ways:** (a) two walks produce coherent XZ translation with
  **Y constant**; (b) the two authoritative copies stay bit-identical; (c) live
  polling (`pospoll.exe`, and the probe's `pos[]` lines) shows the pair updating
  in real time while idle (breathing bob 6.5520 ↔ 6.5517) and jumping coherently
  during movement.
- **Live values (idle):** position `(6.552, 9.181, 11.429)`, facing
  `(-0.513, -0.311, 0.800)`.
- The probe now polls all four addresses each scan pass (`PosPollPass`,
   4 s interval) and logs `pos[i] <addr> = (x,y,z) fwd=(...)`.

### Address-mapping notes (for future RE on this binary)

- Ghidra/decomp/TSV addresses are **VAs with image base `0x400000`**
  (RVA = VA − 0x400000).
- Strings dump addresses are **file offsets**; `.text` file→RVA = +0xC00,
  `.rdata` file→VA = file + 0x401A00. `dumpbin` prints VAs.
- Runtime seam address = **ASLR base + RVA** (observed `AI.exe` base `0x00900000`;
  so `call_method` at runtime = `0x00900000 + 0x526680 = 0x00E26680`).
- The probe computes seam addresses from `GetModuleHandleA(NULL)` at runtime, so
  it does not depend on the observed base.

### Stage 2 status

- **Both seams instrumented:** event firehose (`call_method`, live) + live player
  position (`0x1CA467C0` / `0x1D5FB2A4`, verified).
- **Stage 2 verified.** Remaining for Stage 3: interpret the EntityManager to map
  entity pointers → gameplay objects, and pick the networking model (Phase 4).

## Stage 3 — EntityManager / script-command layer mapped (2026-10-21)

Static work on the decompilation (`decomp\AI.exe.c`, 66,811 functions). All
addresses below are **VAs with base 0x400000** (RVA = VA − 0x400000).

### The live seam is the Composite command system, not `send_event`

- `FUN_00926680` (`CATHODE:EntityManager::call_method`, RVA 0x526680) is the
  central dispatch: `call_method(entitySmartPtr, entitySmartPtr, messageSmartPtr)`
  (thiscall, `ret 0Ch`). It is named at runtime (`FUN_00e1bcc0("CATHODE:EntityManager::call_method")`).
- Smart-pointer layout: arg = `{ctrlBlockPtr, ...}`; the **object** is at
  `*arg + 0xC`. Refcount at `arg[1]`; vtable at `*object`.
- `FUN_00926a60` (`EntityManager::process_messages`) resolves a message's **target
  entity list** via vtable slot **`+0x118`**, then calls `call_method` per target.
  The message object uses `+0x8` as a type word (ASCII ids `0x41/0x43/0x44/0x55`
  = 'A'/'C'/'D'/'U' seen).
- `FUN_0091d9f0` is the **command-type dispatcher**: the message's type
  (`*param_1`) is compared against a table of per-command singleton globals
  (`DAT_0174d308`, `DAT_0174d310`, … `DAT_0174d4xx`), each mapping to a handler
  (`FUN_009366d0`, `FUN_00936c60`, `FUN_00938a20`, …). This is the Composite
  **name→handler command table**.
- `FUN_0050e4f0` (RVA 0x10E3B0, "send_event") builds a 3-`double` argument vector
  and dispatches via `(*msg + 0x18)(msg[2], 0)`. **It fires 0× in normal play** —
  script commands queue through the `call_method` path instead. Keep the send_event
  hook dormant; `call_method` is the live seam.

### The engine's logic layer is CATHODE script commands (840 classes)

- **840 distinct `CATHODE::<Class>::vftable` symbols** are present in the binary
  (full list saved to `coop-anything-cache\cathode_classes.txt`). These are the
  complete vocabulary of the game's logic layer — every scriptable action/condition.
- Base classes: `EntityInterface`, `EntityVariableInterface`,
  `EntityMethodInterface` (reflection over named entity methods/variables),
  `EntityState`, `EntityContainer`.
- Entity classes: `Entity`, `TemporaryEntity` (runtime-spawned),
  `ZoneableEntity` / `ZoneableEntityContainer` (streaming-zone props/NPCs),
  `PlayerHasItemEntity`, `GenericHighlightEntity`, plus `ASSAULT_JOB_ENTITY`,
  `RADIOSITY_ISLAND_ENTITY`, `SOUND_*_ENTITY`.
- **Every command class is stream-serializable:** constructors are
  `FUN_008bd950`-based (base `EntityInterface` ctor) and each class has a factory
  thunk of the form `FUN_008bd920(stream, &ID_TOKEN, ...)`. `FUN_008bd920` calls
  **vtable slot `+0x394`** = the "read command from stream" virtual. Each class
  owns a 4-byte **ID token** (`DAT_0175241x`, `DAT_017523ax`, …).
  → The script-command layer has a **generic ID + deserialize protocol**: commands
  can be written to a stream and replayed. This is the primitive a sync layer needs.

### Dormant multiplayer classes (the engine shipped MP; A:I stripped the netcode)

The frame schedule `FUN_007d6a00` registers every per-frame task by name
(`FUN_00a1a220(name, callback)`). It still contains:

| Task name | Note |
|---|---|
| `NET_AI_UPDATE` | network AI stage — present in the frame loop |
| `NETZ_DISPATCH` | network dispatch stage — present in the frame loop |
| `ENTITY_MANAGER_TICK` | per-frame entity processing (`LAB_00783640`) |
| `ENTITY_MANAGER` | `FUN_00783500` |
| `ZONE_MANAGER` | streaming (RVA hint for zone sync) |

CATHODE classes with **live vftables** for multiplayer concepts:

- `ParticipatingPlayersList`, `NetPlayerCounter`, `NumConnectedPlayers`,
  `NumPlayersOnStart`, `NumDeadPlayers`, `AllPlayersReady`
- `FilterIsLocalPlayer`, `FilterIsAPlayer`, `DespawnPlayer`
- `BroadcastTrigger`, `NetworkedTimer`, `TriggerSync`, `SyncOnAllPlayers`,
  `SyncOnFirstPlayer`, `IsMultiplayerMode`, `VariableThePlayer`

→ **Derived:** Cathode as an engine family shipped multiplayer (Halo Wars 2);
A:I retains the command/player classes but no network transport. The co-op layer
does not have to invent a state model from scratch — the *command* and *player*
vocabulary already exists, and the command stream is serializable. A:I also ships
`AI_UI_NET_*` strings (Steam leaderboard / sign-in UI), evidence the net-facing UI
scaffolding survived.

### Stage 3 conclusions (feed into Phase 4)

1. **Interception point:** `call_method` (RVA 0x526680) sees every queued script
   command, with entity + message smart-pointers. This is where a host writes
   commands into a network stream and where a client applies remote commands.
2. **Serialization:** per-command ID token + vtable `+0x394` deserialize — reuse
   rather than reinvent the wire format.
3. **Player identity:** `FilterIsLocalPlayer` / `ParticipatingPlayersList` give the
   engine's own local-vs-remote concept; the confirmed position record
   (`0x1CA467C0`, `[pos][fwd]`) is the per-frame transform to replicate.
4. **Next (Phase 4):** pick the networking model (playbook `how-coop-mods-work.md`
   / `networking-steam-spacewar.md`) — host-authoritative command relay +
   transform replication looks directly supported by the above.

### Stage 3 status

- EntityManager / Composite command layer **mapped**; dormant MP classes
  **enumerated**; serialization protocol **identified**.

### Stage 3 runtime verification (probe `ai_probe3`, 2026-10-21)

Enhanced the `call_method` hook to resolve, per call: the **target object**
(`*(a1 smartptr) + 0xC`) and the **message type** (`*(a3 smartptr) + 0x8`) and its
object pointer (`+0xC`), then histogram the type words. Observed live in gameplay:

- Message type words are **per-command-class vtable addresses**, stable and
  repeated (e.g. `0cf735e0`, `0cf73450`, `0cf73510`, `0cfa1930`, `0cf7d2a0`, …).
  **64 distinct message types** appear within ~1 s of play — a rich live command
  vocabulary, matching the static `FUN_0091d9f0` command-table finding.
- **Target objects live in a different region** (`0x18ABxxxx`, `0x1202xxxx`) from
  the message/entity control blocks (`0x1C59xxxx`), and the *target set is small and
  stable* during a moment of play (a handful of objects receive repeated calls).
- **Message objects are pooled and retargeted**: only ~16 distinct message-object
  (`msgCtrl`) addresses over thousands of calls, each carrying different type words
  at different times. → **A sync relay must key commands by captured *type* +
  payload at call time, never by message-object address.**
- Method used: inject `ai_probe3.dll` (probe build whose log name derives from the
  DLL filename, so several probe builds can co-exist in one process), read
  `msgtype NEW:` and `cm #` lines.

→ **Verdict:** the `call_method` seam yields every queued script command with a
resolvable type and target — the interception point Phase 4 assumes is real.

## Phase 4 — Networking model chosen (2026-10-21)

Per `playbook/networking.md`. A:I has **no live UDP netcode to tunnel** — the
net-layer strings and `NET_*` frame stages exist, but there is no transport (see
Stage 3). So section 4 (local UDP proxy) does **not** apply; this is section 1
(sync models) work.

### Chosen model (playbook defaults, adapted to the Stage 3 findings)

- **Player avatars: client-authoritative, snapshot-interpolated on peers.**
  Each player's own avatar (position + facing) is simulated locally and replicated
  by the host to the others. The confirmed per-frame transform record
  (`0x1CA467C0`, `[float3 pos][float3 fwd]`) is the datum; send it unreliable
  (stale positions are worthless — playbook §1.2), quantized per §1.2.
- **Script commands / story state: host-authoritative relay.** The whole Composite
  command layer is serializable (per-class ID token + vtable `+0x394` deserialize,
  Stage 3), so the host captures commands at `call_method` (RVA 0x526680), writes
  them to a stream, and clients replay them. This is the playbook's
  "quest and story flags — the highest-value hook" (§1.3) in this engine's own
  vocabulary.
- **NPC/enemy ownership:** one owner per NPC with transfer, per
  `playbook/actor-ownership.md`. Deferred to build; the entity set is visible via
  the `call_method` firehose and `ZoneableEntity`.
- **Transport:** Steam relay via `ISteamNetworkingMessages` (AppID 480 Spacewar for
  the prototype) per `networking-steam-spacewar.md`, **fallback** Tailscale/direct-IP.
  Keep direct-IP working (easiest to test). A:I already imports `steam_api.dll`,
  so the Steam path is native to the process.
- **Testing:** mock steam DLL + two local instances, `clumsy` for bad-network
  profiles (§5). Reuse `tools/` sync-testing utilities at Phase 6.

### Playbook gaps found (to propose in the PR)

1. **`playbook/engine-families.md` has no Cathode entry.** Cathode (Creative
   Assembly) shipped multiplayer in Halo Wars 2; A:I retains the command/player
   class vocabulary and a serializable command stream. A "Cathode" row would let
   the next agent skip the rediscovery: *games:* Alien: Isolation (gap), Halo Wars 2
   (official MP); *what carries over:* the Composite command layer (840 classes,
   per-class ID + `+0x394` deserialize), `EntityManager::call_method` as the
   dispatch seam, the dormant `ParticipatingPlayersList`/`FilterIsLocalPlayer`
   classes, and the `NET_AI_UPDATE`/`NETZ_DISPATCH` frame stages.
2. **`data/directory.yaml` engine entry for Alien: Isolation should read "Cathode"**,
   not "Creative Assembly proprietary" (Phase 3 correction).

### Phase 4 status

- Model + transport **chosen and grounded in the Stage 3 findings**. Next:
  Phase 5 build — start with the avatar replication loop (position record) and the
  `call_method` command relay, tested without the game first (fake world), then in
  two local instances.

