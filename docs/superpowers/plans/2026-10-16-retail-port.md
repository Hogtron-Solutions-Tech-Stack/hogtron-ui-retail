# Retail port plan (2026-10-16)

Goal: every HogTron UI module working on WoW Retail (Midnight 12.x), the same suite players get on Forever.
Non-goals for this plan: Retail-only new features, a CurseForge release before the in-game pass, changing Forever.

Why it is tractable: Forever runs on a Midnight-era client. Secret values, no restricted snippets, no combat log,
`C_DamageMeter`, `C_UnitAuras` private auras are the same rules on Retail. The engine code ports; the **content**
(Classic dungeons, spell ranks, trainers, quest data) and the **Blizzard frame names** the skin hooks are what change.

Confidence labels: *likely* = read the code; nothing below has been seen in a Retail client yet.

## Phase 0 - bootstrap (this PR)
`## Interface: 120105,120107` on every addon, `Compat.isRetail`, Forever's CurseForge id removed, `dev/link.ps1`
defaults to `_retail_`, docs say Retail. Exit: lint 0, tests green.

## Phase 1 - first in-game load (needs a person in game)
`dev/link.ps1`, full restart, then paste: `/hh errors`, `/hh apicheck`, `/hh svinfo`, and `SavedVariables/HogTronUI.lua`.
Every error found here becomes a test + fix. This decides the order of everything below.

## Phase 2 - engine modules (likely small: same API rules, Retail frame names)
| Module | Lines | Expected work |
|---|---|---|
| HogTronUI (core) | ~4.3k | `ClassData` (Retail classes/specs: Evoker, Monk, DH, DK), `Setup` pages, `EditMode` vs Retail Edit Mode |
| HogHeals_Frames | ~4.9k | Click-cast defaults for Retail healers (add Monk, Evoker); dispel types per spec; LibHealComm stays off (Classic-only) |
| HogTron_Units | ~1.2k | Retail `PlayerFrame`/`TargetFrame` layout (Dragonflight+ frame tree) |
| HogTron_Bars | ~1.3k | LibActionButton already Retail-aware; check stance/pet/vehicle/dragonriding bars |
| HogTron_Plates | ~2.8k | Retail nameplate tree; quest-mob icons via Retail quest objectives |
| HogTron_Meter | ~0.9k | `C_DamageMeter` is native Retail - likely works as is |
| HogTron_Chat | ~0.5k | likely works as is |
| HogTron_Skin | ~2.8k | Largest risk: Retail `MainMenuBar`, bags (combined bags), `MicroMenu`, tooltips differ. Re-run `/hh skindiag` |

## Phase 3 - content modules (rewrite the data, keep the UI)
| Module | Lines | Classic content to replace |
|---|---|---|
| HogTron_HUD | ~2.3k | Drop five-second rule, mana pacing, rank advisor (no ranks on Retail) behind `isRetail`; castbar/swing/XP stay |
| HogTron_Training | ~0.4k | Retail trainers don't sell spells: repurpose (talent/new-ability preview) or disable on Retail |
| HogTron_Atlas | ~4.4k | Dungeon guide data -> Retail dungeons/M+ season; Blizzard's `EJ_*` / `C_EncounterJournal` may supply most of it |
| HogTron_Journal | ~1.3k | Healer notes per boss for current Retail dungeons (dispel / shield / kick) |
| HogTron_Quests | ~5k | Quest data + tracker vs Retail `ObjectiveTrackerFrame`, world quests, campaign quests |
| HogTron_Awards | ~2k | Mostly engine; check addon-message limits in Retail instances/encounters |

## Order
Phase 1 first (it re-ranks the rest). Then one PR per module, smallest-risk first: Meter, Chat, Units, Bars,
Frames, Plates, Skin, HUD, Training, Awards, Atlas, Journal, Quests. Each PR: tests + lint output + what to check in game.

## Keeping both repos in step
Forever fixes to engine code: `git fetch forever && git cherry-pick -x <sha>`. Keep folder names, namespaces and
`HogTronUIDB` identical so picks apply cleanly; put Retail-only behaviour behind `Compat.isRetail`, not in renamed files.

## Needs a human
- In-game Retail test passes (Phase 1, and every module PR).
- A new CurseForge (and Wago) project for Retail + `CF_API_TOKEN` repo secret, before the first `v*` tag.
- Sean: heads-up that the Retail fork exists (co-owner of the codebase).
