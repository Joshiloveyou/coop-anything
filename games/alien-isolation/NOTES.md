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
