# Skill Database: States and Enhancements

This document explores how abilities apply ongoing Buffs and Debuffs (known as States) and how they link to card Enhancement mechanics.

### State Applications (Buffs & Debuffs)
When casting a skill like a heal over time, an attack speed buff, or a poison dot, the engine utilizes the secondary `StateBase` structure.

* **`state_id`**
  The exact ID of the status effect applied by the skill. This ID must correspond to an entry in the State/Buff tables (not the SkillBase ID). The mechanics of what this buff actually does are defined strictly within the State system. For simple nuke spells with `EFFECT_TYPE` = `231`, this stays at `0`.
* **`state_level_base`**
  The foundational level of the buff cast (e.g., Level 1 Poison).
* **`state_level_per_skl`**
  As the player levels up the skill, does the applied Buff level up alongside it? For example, setting this to `1.0` means a Level 5 ability casts a Level 5 Buff. `0.20` means a Level 5 skill casts a Level 1 buff.
* **`state_level_per_enhance`**
  Does enhancing the skill via cards increase the Buff's potency layer? `1.0` means a +5 card casts a buff 5 levels higher than standard.

### Duration
* **`state_second`**
  The flat number of seconds the Buff or Debuff stays active on the entity.
* **`state_second_per_level`** & **`state_second_per_enhance`**
  Similar to the level scale, these floats allow you to incrementally add seconds to a buff's duration based on the caster's skill level or enhancement card tier.

### Enhancement System (`Skill Cards`)
* **`skill_enchant_link_id`**
  This links the skill structurally to the UI enhancement window (Alt+Y cards). The parameter supplied here binds to the localization Strings determining what the card displays. By placing `0`, the game engine assumes the skill has no valid card drop and cannot be upgraded.
* **`skill_lvup_limit`**
  Determines specific limitations on when/how the skill points can be distributed, often capping basic leveling or restricting it completely to external item effects.
