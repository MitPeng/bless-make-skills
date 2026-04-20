---
name: "bless-make"
description: "Creates or updates Dota Super Mid bless content end-to-end. Invoke when user asks to make, modify, or troubleshoot a bless (generic or hero-specific)."
---

# Bless Make

Turn a bless requirement into complete, repo-ready implementation for Dota Super Mid.

## When To Invoke

Use this skill when the user asks to:
- create a new bless
- adjust an existing bless's logic or values
- add bless modifier behavior
- wire bless localization, icon, or audio assets
- debug bless behavior or recraft/init flow

## Required Inputs

- Bless ID (for example `10001`)
- Scope: generic bless or hero-specific bless
- Expected gameplay effect and trigger conditions
- Any required numbers (weight, quality, special values, attrs)

If inputs are incomplete, ask focused clarification questions before coding.

## Execution Workflow

1. Validate names first (mandatory)
- Check ability/modifier naming against `game/scripts/src/shared/share_dota2_declarations.d.ts`.
- If required names cannot be confirmed, stop and ask the user before continuing.

2. Confirm config source of truth
- Bless table source is `excels/server/blessAbility.xlsx` (compiled to `game/scripts/npc/server/blessAbility.txt`).
- For AI coding tasks, implement code and localization changes in repo files; do not manually edit generated files.

3. Implement bless class
- Location: `game/scripts/src/modules/bless/bless_details/`.
- Naming: `bless_{id}.ts`.
- Base class:
  - Generic bless: `BlessBase`
  - Hero-specific bless: `BlessBase_Hero`
- Lifecycle methods to use as needed: `_OnInit`, `_OnCreated`, `_OnRemove`, `_OnRecraftStart`, `_OnRecraftEnd`.

4. Implement modifier class
- Location: `game/scripts/src/modifiers/bless_modifiers/`.
- Naming: `sl_modifier_bless_{id}.ts`.
- Register with `@registerModifier("modifiers/bless_modifiers/sl_modifier_bless_{id}")`.
- Base class choice:
  - `SLModifierBase` for server-only logic
  - `sl_modifier_transmitter_data` if client sync or dynamic tooltip data is needed

5. Apply core implementation rules
- For permanent/static buff on init, use:
  - `this._ListenForAssignHero(hero => this._AddStaticModifier(hero, modifier, {...}), { check_alive: true })`
- Do not directly add long-term buff in `_OnInit()` through immediate hero fetch.
- Prefer KV `fix_ability_special_values` for static native ability changes.
- Use `_SetDynamicAbilityAmp()` only when runtime-dynamic behavior is required.
- Keep values in configs (`special_values`, `attrs`), avoid hardcoded constants.

6. Update localization CSVs only
- Files:
  - `game/resource/bless_name.csv`
  - `game/resource/bless_desc.csv`
  - `game/resource/bless_desc_short.csv`
  - `game/resource/sl_modifiers.csv`
- Rules:
  - Never modify CSV header row.
  - Only append new row or edit the specific target row.
  - Use comma-separated columns; quote any field containing commas.
  - Include all required languages.

7. Optional assets
- Icon path: `game/resource/flash3/images/spellicons/buff/bless/{id}.png`.
- Optional sound declaration in `content/soundevents/bless.vsndevts` with naming:
  - `bless_{id}`
  - `bless_{id}_{suffix}`

8. Validate and handoff
- Run diagnostics on touched files and fix straightforward issues.
- Provide changed file list and concise verification guidance.
- Include recommended debug commands:
  - `addb {id}`
  - `delb {id}`
  - `recb {id}`
  - `loop_blesses`

## Quick Checklist

- Declaration names validated before coding
- Correct base classes selected
- Init-time permanent buff follows assign-hero listener pattern
- Config-driven values used (no accidental hardcoding)
- Localization rows complete and header untouched
- Optional audio/icon naming correct
- Diagnostics clean for edited files

## Project Rules (Hard Constraints)

- Always verify native skill names and modifiers in `game/scripts/src/shared/share_dota2_declarations.d.ts` before implementation.
- If a required native name is not found, stop and notify the user before taking the next step.
- When calling `AddSLModifier`, always pass params inside `modifierTable`; otherwise modifier `OnCreated` params may be nil.
- Never modify localization CSV header row (first line). Only edit target rows or append allowed rows.
- If a localization token uses suffixed IDs (for example `bless_100363a`), place it directly below its base ID (`bless_100363`), not at file end.
- If a CSV field includes English commas, wrap that field in double quotes to avoid column shift.
- Never run `npm run lint` in this repository.

## Project Rules (Engineering Conventions)

- If bless logic changes only behavior, prioritize code changes and avoid touching localization unless user explicitly requests text updates.
- For client-visible dynamic modifier values or tooltip sync, prefer `sl_modifier_transmitter_data` with `_ApplyParams`.
- For invisibility that should not break on attack/cast, use `sl_modifier_invisible_non_break`.
- For gold number popups, use `SLModules.ClientData.PushNumberData` as standard effect.
- For tree-destruction triggers, use `ModifierFunction.ON_TREE_CUT_DOWN` and verify `event.unit === this.GetParent()`.
