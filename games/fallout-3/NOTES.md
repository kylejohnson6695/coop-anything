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

## Session 1b: extending fnv-coop to TTW (owner chose shared quest progress)

### Observations (ran here, 2026-10-24)

- fnv-coop @ commit of 2026-10-23: `python -I tools/test_all.py --quick` → ALL SUITES PASS on Linux
  (C++ and game-dependent suites SKIP, as documented).
- fnv-coop names records as `Plugin.esm:LOCALID` and resolves runtime FormIDs from the live mod
  list (`plugin/dialogue/engine/engine_xnvse.cpp`); quest rules are keyed by quest EditorID
  (`docs/quest-policies/rules.json`). Neither assumes a fixed load order at runtime.
- Hard-coded New Vegas plugin lists: `tools/questdata/extract.py` `PLUGINS`, and
  `tools/questdata/loadorder.json` (used by test tooling for console FormIDs).
- `tools/questdata/extract.py`, `effects.py`, `classify.py` run unchanged on **vanilla** Fallout 3
  GOTY ESMs (`--plugins Fallout3.esm,Anchorage.esm,ThePitt.esm,BrokenSteel.esm,PointLookout.esm,Zeta.esm`):
  294 quests (136 with stages/objectives), 1,343 stages, 28,606 INFOs, 8,490 result scripts, ~8 s.
  New Vegas has rules for 483 quests, for scale.
- Story-quest risk tiers (vanilla FO3 + DLC, 136 story quests): T0 12, T1 15, T2 16, T3 56, T4 37.
  Riskiest are the Vault 101 childhood (CG02, CG03, CG04), Tranquility Lane (MQ04), Wasteland
  Survival Guide (MS03), The Replicated Man (MS08), and the DLC openers.
- 17 FO3 quest EditorIDs also exist in New Vegas' rules (leftovers in FalloutNV.esm):
  AchievementQuest, AudioHolotapes, BH01, CG00, CG02, CGTutorial, ControlRumble, Doctors,
  FFradioSignals02, Followers, FollowersHireRL3, Generic, HD00JukeboxMusic, HD00RobotsDialog,
  RadioTenpenny, RadioVault101PA, WaterBeggars.
- Generated data lives in `local/fo3-questdata/` (game-derived, never committed).

### Derived

- TTW support is mostly a data and rules job, not an engine port: same exe (FalloutNV.exe
  1.4.0.525), same xNVSE plugin, plugin-qualified IDs.
- EditorID keys are ambiguous under TTW for the 17 clashing names (unless TTW renamed them):
  rules need a plugin-qualified key or a TTW-specific rules file.

### Guesses (unverified)

- The TTW installer rewrites Fallout3.esm and TaleOfTwoWastelands.esm overrides many FO3 quests,
  so the real counts and tiers will differ from vanilla. Re-run on TTW's generated ESMs.
- TTW scripts use JIP LN NVSE functions that fnv-coop's hooks may not capture; check
  `vocab_gaps.md` on TTW data.
- `SetQuestVar` tops the "not in vocabulary" list (5,345 scripts); likely the parser's name for
  `set Quest.var to x`. Need the same report on New Vegas data to compare.

### Plan

1. Owner installs TTW per the official guide (needed for real data and every in-game step).
2. Stage TTW's ESMs read-only, re-run the questdata pipeline with a TTW plugin list
   (FNV + DLC, Fallout3.esm + DLC, TaleOfTwoWastelands.esm, YUPTTW.esm).
3. fnv-coop changes, as a branch/PR to fnv-coop: make the plugin list and load order a profile
   (`fnv` / `ttw`), plugin-qualified rule keys for clashes, `rules.ttw.json`.
4. Auto-generate draft rules for FO3 quests with the existing coverage tools; hand-curate the
   T4 set (Vault 101 childhood first: it runs before anyone leaves the vault).
5. In-game proofs on the owner's PC (two game copies, the existing bridge): needs MSVC build
   and a shell on the owner's PC; not possible from the cloud workspace alone.
6. Give back: TTW notes into fnv-coop docs, Fallout 3 directory entry + write-up here.

### Unknown

- Whether fnv-coop's maintainer wants TTW in the same repo or a fork; ask in a claim issue.
- Whether TTW keeps FO3 EditorIDs unchanged.

## Session 2 (2026-10-27): TTW installed on owner's PC

- Observed: TTW 3.4 installer completed from Steam FO3 GOTY 1.7.0.4 + FNV 1.4.0.525 (resolves the
  "Unknown" above: the Steam 1.7.0.4 build works with the TTW 3.4 installer). Output ≈17 GB.
- Observed: MO2 2.5.2-ML1.5 TTW instance, load order of 18 masters matches the installer's list.
- Note for fnv-coop testing: FalloutNV.exe is now 4GB-patched (sha256 changes); test installs
  must be copies, never the owner's install.
- Next (plan step 2): stage TTW's generated ESMs read-only and re-run questdata with the TTW
  plugin list.
