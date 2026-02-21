# Skill Database: Identifiers and Core Flags

This document explains the foundational database columns that give a skill its identity, text descriptions, and core active/passive behaviors.

### Basic Identifiers
* **`id`**
  The unique identification number for the skill (e.g., `90006`). This is how the engine, items, and NPCs refer to this specific skill.
* **`text_id`**
  The string ID (from `StringTable` resources) representing the prolonged description text of the skill. For example `50090006`.
* **`desc_id`**
  Often used for secondary brief descriptions or sub-tooltips in certain client versions. Usually maps to a string ID.
* **`tooltip_id`**
  The string ID (from `StringTable`) representing the skill's Name that appears in bold when hovered over in the UI. For example `40090006`.

### Core Activation & Types
* **`is_valid`**
  Determines if the game server should load and process this skill.
  `1` = Valid and active.
  `0` = Disabled, ignored by the engine (useful for deprecating old skills).
* **`elemental`**
  Assigns a specific element to the skill's hit processing.
  `0` = None/Physical, `1` = Fire, `2` = Water, `3` = Wind, `4` = Earth, `5` = Light, `6` = Dark. 
  *(For more details, see `05_Elementals_Guide.md`)*.
* **`is_passive`**
  Determines if the skill is always active without casting. 
  `0` = Passive (`!is_active` in the engine).
  `1` = Active (Requires casting).
* **`is_physical_act`**
  Determines the damage and hit-calculation type.
  `1` = Physical Attack (scales with P.Atk, uses weapon strike animations).
  `0` = Magical / Spell (scales with M.Atk or works as a non-damaging buff/heal).
* **`is_harmful`**
  Flags the skill as a positive or negative effect.
  `1` = Harmful (Attacks, Debuffs, DoTs). This allows the skill to engage combat and be resisted.
  `0` = Helpful (Heals, Buffs). Can be cast on allies without initiating combat.
* **`is_toggle`** & **`toggle_group`**
  Sets up sustained "Aura" style skills that drain MP over time.
  `is_toggle` = `1` turns this feature on.
  `toggle_group` determines which aura group it belongs to (casting an aura from the same group will turn off the previous one to prevent stacking).

### Visuals and Execution Logic
* **`effect_type`**
  The most critical mechanical switch. This dictates exactly which C++ function handles the skill's numbers. For example, `4` triggers the `EF_PARAMETER_AMP` calculations, `61` triggers `EF_HEAL`, etc.
* **`icon_id`** & **`icon_file_name`**
  These map to the client-side UI graphics. `icon_file_name` is the physical texture file (e.g., `icon_skill_pas_mask_magic`), while `icon_id` is an internal UI indexing value.
