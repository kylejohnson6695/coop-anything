# Fallout 3 — working notes

Session 1 (2026-10-24). Phase 0 (triage) and a first pass of phase 1 (research).

## Game identity

- Title: Fallout 3 (2008), engine Gamebryo.
- Store: Steam (Fallout 3 GOTY + Fallout: New Vegas, all DLC). OS: Windows x64.
- `Fallout3.exe` FileVersion 1.7.0.4 (Steam's GFWL-free build), SHA-256
  `c3f97c2255fa041a851c17cf372d69aaadd8694e2dc4230ba556001bbfbd2f3e`.
- `FalloutNV.exe` FileVersion 1.4.0.525 (matches fnv-coop's tested version), SHA-256
  `3a87f92f011e5dc9179ddf733cf08be2b39ea6e5b7a8a9e3a9a72dafcc1b104d`.
- On the co-op route the FNV executable runs everything (TTW puts FO3 content in FNV); the
  FO3 install only feeds the TTW installer its data files.

## Observations (sourced)

- Directory entry: VaultMP, abandoned, repo archived 2023-04, last commit 2021-06, QA none.
  https://github.com/foxtacles/vaultmp
- No open coop-anything issues or PRs mention Fallout 3 (issue list checked via web, 2026-10-24).
- NV:MP has an official co-op guide for running **Tale of Two Wastelands** (the full Fallout 3
  game and DLC ported into the New Vegas engine) on NV:MP: requires FO3 + all DLC, FNV + all DLC,
  TTW via the official guide, MO2, the latest NV:MP release, and a fixed 18-entry plugin order.
  Known issue: starting-vault NPCs are not synced; outside the vault NPCs and enemies are.
  https://nuwiki.nv-mp.com/wiki/Hosting_TTW
- NV:MP co-op: up to 10 players, free, Steam and GOG; quest progress is **not** shared (each
  player has their own save). https://steamcommunity.com/sharedfiles/filedetails/?id=3292884725
- "NVMP COOP Companion and Server Patch": author reports finishing the FO3 main quest in TTW with
  one other player; the Purification mission can get stuck in MP (workaround: finish it in true
  single-player). https://www.nexusmods.com/newvegas/mods/97189
- fnv-coop (story/quest sync for FNV) targets FNV 1.4.0.525 and New Vegas story quests; no TTW
  support mentioned in its docs. https://github.com/cranebeaker/fnv-coop

## Derived

- Best route to Fallout 3 co-op is not the FO3 executable at all: TTW + NV:MP gives players,
  NPCs and combat sync today. The missing piece is quest/story sync, which is exactly what
  fnv-coop builds for FNV; extending its per-quest rules to TTW's FO3 quests is the natural
  Path 1/extend project.

## Guesses (unverified)

- fnv-coop's effect-capture approach should work on TTW quests since they run in the FNV
  engine, but TTW quest scripts are different data; untested.

## Unknown

- Current NV:MP release number and TTW version to pin.
- Whether the TTW installer accepts Steam FO3 1.7.0.4 without issue: the official guide says
  English Steam/GOG/Epic copies are supported (https://thebestoftimes.moddinglinked.com/intro.html);
  untested here.

## Session 1 outcome

- Owner chose the play-now route (TTW + NV:MP, no shared quests). Agent installed nothing;
  owner installs following the official guides.
- NV:MP co-op hosting: local story server on port 27015, Radmin VPN to avoid port forwarding,
  everyone needs identical mods and load order. https://nuwiki.nv-mp.com/wiki/Hosting_CO:OP
