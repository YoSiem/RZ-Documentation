# Database Architecture - Skills

When creating a new skill in the database tables (e.g., `SkillBase` or RDB/SQL files), dozens of columns need to be populated. Because there are so many columns, this guide is split into multiple detailed documents categorizing each part of a skill's structure.

Below is the master list of all columns in the Skill database table. Click the category headers to read the detailed mechanics of those specific columns.

### [1. Identifiers and Core Flags](06_Skill_Identifiers_and_Core_Flags.md)
* **`id`** - Unique identity number for the skill.
* **`text_id`** - String ID for the long description.
* **`desc_id`** - Secondary String ID.
* **`tooltip_id`** - String ID for the Skill Name.
* **`is_valid`** - Toggle to enable or disable the skill (`1` or `0`).
* **`elemental`** - Assigns element type. (See [Detailed Elementals Guide](05_Elementals_Guide.md)).
* **`is_passive`** - `0` = Passive (`!is_active`), `1` = Active.
* **`is_physical_act`** - `1` = Physical Attack, `0` = Magic/Buff.
* **`is_harmful`** - `1` = Negative buff or attack, `0` = Positive spell.
* **`is_toggle`** - `1` = Sustained MP aura.
* **`toggle_group`** - Groups toggles to prevent stacking.
* **`effect_type`** - The core C++ logic switch hook (e.g. `4` for `EF_PARAMETER_AMP`).
* **`icon_id` & `icon_file_name`** - UI graphics.

### [2. Casting and Timing](07_Skill_Casting_and_Timing.md)
* **`casting_type`** - Interrupt mechanics.
* **`casting_level`** - Interruption scaling.
* **`delay_cast`** - Base casting time.
* **`delay_cast_per_skl`** - Cast time increment per level.
* **`delay_cast_mode_per_enhance`** - Enhance card casting speed modifier.
* **`delay_common`** - Global Cooldown (GCD).
* **`delay_cooltime`** - Cooldown before reuse.
* **`delay_cooltime_per_skl`** - Cooldown scaling per level.
* **`delay_cooltime_mode_per_enhance`** - Cooldown reduction via enhance cards.
* **`cool_time_group_id`** - Shared cooldown group ID across distinct skills.
* **`is_projectile`** - `1` = Spawns flying projectile (calculates hit on impact).
* **`projectile_speed`** - Speed of the projectile.
* **`projectile_acceleration`** - Acceleration curve of the projectile.

### [3. Costs, Requirements, and Weapon limits](08_Skill_Costs_and_Requirements.md)
* **`cost_hp` / `cost_hp_per_skl` / `cost_hp_per` / `cost_hp_per_skl_per`** - Flat and % Max HP costs.
* **`cost_mp` / `cost_mp_per_skl` / `cost_mp_per` / `cost_mp_per_skl_per` / `cost_mp_per_enhance`** - Flat and % MP costs.
* **`cost_havoc` / `cost_havoc_per_skl` / `cost_energy` / `cost_energy_per_skl`** - Specific class combat point consumption.
* **`cost_exp` / `cost_exp_per_enhance` / `cost_jp` / `cost_jp_per_enhance`** - EXP and Job Points cost for using/upgrading.
* **`cost_item` / `cost_item_count` / `cost_item_count_per_skl`** - Item ID and quantities consumed on cast.
* **`need_level` / `need_hp` / `need_mp` / `need_havoc` / `need_havoc_burst`** - Minimum thresholds needed to attempt casting.
* **`need_state_id` / `need_state_level` / `need_state_exhaust`** - Requires a specific buff/debuff to be active on target/caster.
* **`vf_...` Weapon Flags (`vf_dagger`, `vf_two_hand_sword`, etc.)** - Limits usage uniquely to equipped weapon classes.
* **`vf_shield_only`** - Requires a shield.
* **`vf_is_not_need_weapon`** - Bypasses all weapon restrictions.

### [4. Target and Usage Logic](09_Skill_Target_and_Usage.md)
* **`is_need_target`** - `1` = Requires targeting, `0` = Auto/Self.
* **`is_corpse`** - `1` = Can only cast on dead targets (Resurrection).
* **`target`** - Defines regional structure (e.g., `0`=Target, `1`=AoE, `2`=Self).
* **`cast_range` / `valid_range`** - Max distance to initiate and finish the cast.
* **`uf_self` / `uf_party` / `uf_guild` / `uf_neutral` / `uf_purple` / `uf_enemy`** - Faction logic (is the entity allowed to be hit by this).
* **`tf_avatar` / `tf_summon` / `tf_monster`** - Entity type restrictions (`1` = Allowed).

### [5. States and Enhancements](10_Skill_States_and_Enhancements.md)
* **`skill_enchant_link_id`** - String table link for Alt+Y Enhancement Cards.
* **`skill_lvup_limit`** - Leveling restrictions via skill window.
* **`state_id`** - The Status Effect ID applied by the skill.
* **`state_level_base` / `state_level_per_skl` / `state_level_per_enhance`** - Modifies the Rank/Level of the Buff applied.
* **`state_second` / `state_second_per_level` / `state_second_per_enhance`** - Dictates duration of buff/debuff in seconds.

### [6. Combat Modifiers and Variables](11_Skill_Combat_Modifiers_and_Variables.md)
* **`probability_on_hit` / `probability_inc_by_slv`** - Absolute percentage chance to apply the `state_id` effect.
* **`hit_bonus` / `hit_bonus_per_enhance` / `percentage`** - Flat accuracy injection into combat formula logic.
* **`hate_mod` / `hate_basic` / `hate_per_skl` / `hate_per_enhance`** - Modulates Threat/Aggro generation towards monsters.
* **`critical_bonus` / `critical_bonus_per_skl`** - Flat injection to Critical Rate chance.
* **`var1` through `var20`** - The primary mathematical floats interpreted dynamically by `effect_type`. (e.g., Damage scaling, Bitmasks, Radius ranges, etc.).
