# Skill Database: Costs, Requirements, and Weapon Limits

This document explains the various prerequisite columns that decide whether a skill is allowed to be cast based on resource pools or equipped gear.

### Resource Costs
Skills tap into either HP, MP, or class-specific resources (Havoc, Energy) as they level up.

* **`cost_hp`** & **`cost_hp_per_skl`**
  The flat HP chunk consumed when casting the spell. `per_skl` adds to this base amount for every level beyond 1. Commonly used in sacrifice or berserker skills.
* **`cost_hp_per`** & **`cost_hp_per_skl_per`**
  The percentage (%) of Max HP consumed to cast the spell. `per_skl_per` scales this percentage by skill level.
* **`cost_mp`** & **`cost_mp_per_skl`**
  The flat MP drain (Mana) consumed during casting.
* **`cost_mp_per_enhance`**
  Additional MP drained strictly based on card enhancement (`+` levels on Alt+Y UI).
* **`cost_mp_per`** & **`cost_mp_per_skl_per`**
  The percentage (%) of Max MP consumed to cast the spell. (Often used for toggles and auras).

### Unique Class Resources & Meta-Currencies
* **`cost_havoc`** & **`cost_havoc_per_skl`**
  Resource specific to some physical classes/assassins depending on the game mechanics.
* **`cost_energy`** & **`cost_energy_per_skl`**
  Resource specific to certain other combat arts or chi-focused classes. *(In C++ parsing, these translate to variable inputs `cost_energy`)*.
* **`cost_jp`** & **`cost_jp_per_enhance`**
  Job Points consumed when upgrading/purchasing the skill or enhancing it in specific trees.
* **`cost_exp`** & **`cost_exp_per_enhance`**
  Base Experience points permanently consumed from the character's pool when casting or enhancing (rarely used outside absolute server limit breaks).
* **`cost_item`**, **`cost_item_count`**, **`cost_item_count_per_skl`**
  Some spells consume a physical inventory item *(e.g., Lapis, Shards, specific ammo)*. 
  `cost_item` is the exact item's `id`. `cost_item_count` is how many are deleted on cast.

### Condition Requirements
For a skill to physically activate, certain internal gates must be mapped correctly.
* **`need_level`**, **`need_hp`**, **`need_mp`**
  The absolute minimum amount of Base Level, Current HP, or Current MP the player must possess even to attempt clicking the skill icon.
* **`need_havoc`** & **`need_havoc_burst`**
  The specific threshold of internal 'Havoc' points required to launch the skill (burst implies it wipes the stack clean upon usage).
* **`need_state_id`**, **`need_state_level`**, **`need_state_exhaust`**
  Crucial combo mechanic. If the target or caster does not hold a specific Buff/Debuff (`need_state_id`) running at a specific `need_state_level`, the skill grays out. If `need_state_exhaust` is 1, casting the skill completely wipes and consumes that buff.

### Equipment Limitation Flags (`vf_`)
These booleans (`1` or `0`) restrict skill actions exactly to the class of weaponry held in the player's hands. Without the proper weapon, the UI notifies the player "Invalid weapon equipped."

* **`vf_one_hand_sword`** (1H Swords)
* **`vf_two_hand_sword`** (2H Swords)
* **`vf_double_sword`** (Dual Wield Swords)
* **`vf_dagger`** (Daggers)
* **`vf_double_dagger`** (Dual Daggers)
* **`vf_spear`** (Spears/Lances)
* **`vf_axe`** (Two-Hand Axes)
* **`vf_one_hand_axe`** (1H Axes)
* **`vf_double_axe`** (Dual Axes)
* **`vf_one_hand_mace`** (1H Maces)
* **`vf_two_hand_mace`** (2H Maces)
* **`vf_lightbow`** (Light Bows)
* **`vf_heavybow`** (Heavy Bows)
* **`vf_crossbow`** (Crossbows)
* **`vf_one_hand_staff`** (1H Staves/Wands)
* **`vf_two_hand_staff`** (2H Staves)
* **`vf_shield_only`** (Requires a shield actively equipped. Important for block/taunt skills).
* **`vf_is_not_need_weapon`** (Bypasses all the above. Set to `1` if the skill or passive works inherently without any gear requirement).
