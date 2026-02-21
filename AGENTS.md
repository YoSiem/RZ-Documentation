# Instructions for AI Agents: Generating Rappelz Skills (EF_PARAMETER_AMP)

> **Context for AI**: If a user asks you to generate new passive skills for their Rappelz server, you must strictly adhere to the rules and architectures defined in this document. Do not invent columns, and always provide valid T-SQL `INSERT` statements.

## 1. The Output Format
When asked to create new skills, you must:
1. Briefly list the names and effects of the skills you are creating.
2. Provide a single unified ````sql ```` code block containing all `INSERT INTO [dbo].[SkillResource]` statements.

### The Exact Column Schema
Every single one of these 94 columns must explicitly exist and be provided in your `INSERT` query in this exact order:

```sql
INSERT INTO [dbo].[SkillResource] (
    [id], [text_id], [desc_id], [tooltip_id], [is_valid], [elemental], 
    [is_passive], [is_physical_act], [is_harmful], [is_need_target], 
    [is_corpse], [is_toggle], [toggle_group], [casting_type], 
    [casting_level], [cast_range], [valid_range], [cost_hp], 
    [cost_hp_per_skl], [cost_mp], [cost_mp_per_skl], [cost_mp_per_enhance], 
    [cost_hp_per], [cost_hp_per_skl_per], [cost_mp_per], [cost_mp_per_skl_per], 
    [cost_havoc], [cost_havoc_per_skl], [cost_energy], [cost_energy_per_skl], 
    [cost_exp], [cost_exp_per_enhance], [cost_jp], [cost_jp_per_enhance], 
    [cost_item], [cost_item_count], [cost_item_count_per_skl], [need_level], 
    [need_hp], [need_mp], [need_havoc], [need_havoc_burst], [need_state_id], 
    [need_state_level], [need_state_exhaust], [vf_one_hand_sword], 
    [vf_two_hand_sword], [vf_double_sword], [vf_dagger], [vf_double_dagger], 
    [vf_spear], [vf_axe], [vf_one_hand_axe], [vf_double_axe], 
    [vf_one_hand_mace], [vf_two_hand_mace], [vf_lightbow], [vf_heavybow], 
    [vf_crossbow], [vf_one_hand_staff], [vf_two_hand_staff], [vf_shield_only], 
    [vf_is_not_need_weapon], [delay_cast], [delay_cast_per_skl], 
    [delay_cast_mode_per_enhance], [delay_common], [delay_cooltime], 
    [delay_cooltime_per_skl], [delay_cooltime_mode_per_enhance], 
    [cool_time_group_id], [uf_self], [uf_party], [uf_guild], [uf_neutral], 
    [uf_purple], [uf_enemy], [tf_avatar], [tf_summon], [tf_monster], 
    [skill_lvup_limit], [target], [effect_type], [skill_enchant_link_id], 
    [state_id], [state_level_base], [state_level_per_skl], [state_level_per_enhance], 
    [state_second], [state_second_per_level], [state_second_per_enhance], 
    [probability_on_hit], [probability_inc_by_slv], [hit_bonus], 
    [hit_bonus_per_enhance], [percentage], [hate_mod], [hate_basic], 
    [hate_per_skl], [hate_per_enhance], [critical_bonus], [critical_bonus_per_skl], 
    [var1], [var2], [var3], [var4], [var5], [var6], [var7], [var8], [var9], [var10], 
    [var11], [var12], [var13], [var14], [var15], [var16], [var17], [var18], [var19], 
    [var20], [icon_id], [icon_file_name], [is_projectile], [projectile_speed], 
    [projectile_acceleration]
) VALUES ( ... );
```

## 2. Best Practices & Naming Conventions
To prevent database overlap with official server skills, you must dynamically calculate the structural IDs for any new skill you generate:
- **`[id]`**: Must strictly start above **`91000`** and increment by 1 for each new skill. Make sure the user provides a starting ID if needed, otherwise assume `91000`.
- **`[text_id]`**: Must always equal **`50000000 + [id]`**. (e.g., if ID is 91000, `text_id` is 50091000).
- **`[tooltip_id]`**: Must always equal **`40000000 + [id]`**. (e.g., if ID is 91000, `tooltip_id` is 40091000).
- **`[desc_id]`**: Must always be **`0`**.
- **Icons**: Assign a realistic `[icon_id]` (e.g., `1530`, `1100`) and a suitable `[icon_file_name]` string (e.g., `'icon_skill_pas_mage_moral_culture'`, `'icon_skill_pas_mask_magic'`).

## 3. Mandatory Static Columns for Passive Buffs (`effect_type` = 4)
Unless the user specifies that the passive affects a specific weapon, stick to these core rigid values for the SQL insert:
- `[is_valid]`: `1`
- `[is_passive]`: `'0'` *(In the engine, `!is_active` equates 0 to passive!)*
- `[elemental]`: `'0'`
- `[is_physical_act]`, `[is_harmful]`, `[is_need_target]`, `[is_corpse]`, `[is_toggle]`: `'0'`
- All Cost fields (`[cost_hp]`, `[cost_mp]`, etc.): `0` or `0.00`
- All Need fields (`[need_level]`, `[need_hp]`, etc.): `0`
- `[cast_range]`, `[valid_range]`: `0`
- `[vf_is_not_need_weapon]`: `'1'` *(All other `vf_` weapon columns = `'0'`)*
- `[target]`: `41` *(Or `2` for caster-only bounds, but `41` safely blankets self/summon passive loops)*
- `[effect_type]`: `4`
- Delay fields: `[delay_common]` = `1.50`. All other delays = `0.00`.
- Variables `[var19]`, `[var20]`: `0.0000`.

## 4. The Mathematics of Variables (`var1` to `var18`)
Because `effect_type` is 4, you must configure the variables in 3-slot groups.
* **Slot 1 (`var1`, `var2`, `var3`)**
* **Slot 2 (`var4`, `var5`, `var6`)**
* **Slot 3 (`var7`, `var8`, `var9`)** *(Uses EXTENDED Bitmasks)*
* **Slot 4 (`var10`, `var11`, `var12`)** *(Uses EXTENDED Bitmasks)*
* **Slot 5 (`var13`, `var14`, `var15`)**
* **Slot 6 (`var16`, `var17`, `var18`)**

**Formula Check:**
**CRITICAL MATH RULE FOR VARS:**
All mathematical modifiers and multipliers in the `var` columns operate on a `1 = 100%` base float logic!
- `1.0000` = 100%
- `0.0500` = 5%
- `0.0100` = 1%

* `[var2, 5, 14, 17]` = Base Percentage/Modifier (e.g., `0.10` gives a flat 10% base boost at level 1).
* `[var3, 6, 15, 18]` = Float Level Multiplier (e.g., `0.01` means +1% per skill level).

**Crucial Exception - Extended Bitmasks:**
Slots 3 and 4 (`var7`, `var10`) pass through `ampParameter2` internally. They require an entirely different bitset table involving Elemental properties and defense logic!

### Standard Bitmasks (`var1`, `var4`, `var13`, `var16`)
| Stat | Value | Stat | Value |
|---|---|---|---|
| STR | `1` | Accuracy | `16384` |
| VIT | `2` | M.Accuracy | `32768` |
| AGI | `4` | Critical Rate | `65536` |
| DEX | `8` | Block Rate | `131072` |
| INT | `16` | Block Defense | `262144` |
| MEN | `32` | Avoid/Dodge | `524288` |
| LUK | `64` | M.Resistance | `1048576` |
| P.Atk | `128` | Max HP | `2097152` |
| M.Atk | `256` | Max MP | `4194304` |
| P.Def | `512` | HP Regen Add | `16777216` |
| M.Def | `1024` | MP Regen Add | `33554432` |
| Atk Speed | `2048` | Max Weight | `1073741824` |
| Cast Speed | `4096` | Final Dmg Red. | `2147483648` |
| Move Speed | `8192` | | |

*Note: For the above stats, you must format floats precisely (e.g. `2048.0000`). Make sure `Base Value` and `Multipliers` are 0.0000 if unused.*

### Extended Bitmasks (`var7`, `var10`)
*Use these exclusively when constructing Slot 3 or Slot 4 variables!*
| Stat | Value | Stat | Value |
|---|---|---|---|
| None Resist | `1` | Perfect Block | `512` |
| Fire Resist | `2` | P.Def Ignore | `1024` |
| Water Resist | `4` | M.Def Ignore | `2048` |
| Wind Resist | `8` | P. Penetration | `4096` |
| Earth Resist | `16` | M. Penetration | `8192` |
| Light Resist | `32` | Critical Damage | `268435456` |
| Dark Resist | `64` | Attack Range | `256` |

*Note: For properties like P.Def Ignore or Penetration, entering `0.05` in the Multiplier (`var9` or `var12`) translates strictly to a 5% bypass per skill level. Since `1 = 100%`, a value like `0.50` would be 50%.*

---
### Example AI Output Generation
If prompted to "Create a defensive passive starting at ID 92000 that boosts P.Def and VIT", you must generate a full block exactly like this:

**1. Vitality Guardian (ID: 92000)**
- **Effect:** Passively increases P.Def by +5% per skill level and VIT by +2% per skill level.

```sql
INSERT INTO [dbo].[SkillResource] (
    [id], [text_id], [desc_id], [tooltip_id], [is_valid], [elemental], 
    [is_passive], [is_physical_act], [is_harmful], [is_need_target], 
    [is_corpse], [is_toggle], [toggle_group], [casting_type], 
    [casting_level], [cast_range], [valid_range], [cost_hp], 
    [cost_hp_per_skl], [cost_mp], [cost_mp_per_skl], [cost_mp_per_enhance], 
    [cost_hp_per], [cost_hp_per_skl_per], [cost_mp_per], [cost_mp_per_skl_per], 
    [cost_havoc], [cost_havoc_per_skl], [cost_energy], [cost_energy_per_skl], 
    [cost_exp], [cost_exp_per_enhance], [cost_jp], [cost_jp_per_enhance], 
    [cost_item], [cost_item_count], [cost_item_count_per_skl], [need_level], 
    [need_hp], [need_mp], [need_havoc], [need_havoc_burst], [need_state_id], 
    [need_state_level], [need_state_exhaust], [vf_one_hand_sword], 
    [vf_two_hand_sword], [vf_double_sword], [vf_dagger], [vf_double_dagger], 
    [vf_spear], [vf_axe], [vf_one_hand_axe], [vf_double_axe], 
    [vf_one_hand_mace], [vf_two_hand_mace], [vf_lightbow], [vf_heavybow], 
    [vf_crossbow], [vf_one_hand_staff], [vf_two_hand_staff], [vf_shield_only], 
    [vf_is_not_need_weapon], [delay_cast], [delay_cast_per_skl], 
    [delay_cast_mode_per_enhance], [delay_common], [delay_cooltime], 
    [delay_cooltime_per_skl], [delay_cooltime_mode_per_enhance], 
    [cool_time_group_id], [uf_self], [uf_party], [uf_guild], [uf_neutral], 
    [uf_purple], [uf_enemy], [tf_avatar], [tf_summon], [tf_monster], 
    [skill_lvup_limit], [target], [effect_type], [skill_enchant_link_id], 
    [state_id], [state_level_base], [state_level_per_skl], [state_level_per_enhance], 
    [state_second], [state_second_per_level], [state_second_per_enhance], 
    [probability_on_hit], [probability_inc_by_slv], [hit_bonus], 
    [hit_bonus_per_enhance], [percentage], [hate_mod], [hate_basic], 
    [hate_per_skl], [hate_per_enhance], [critical_bonus], [critical_bonus_per_skl], 
    [var1], [var2], [var3], [var4], [var5], [var6], [var7], [var8], [var9], [var10], 
    [var11], [var12], [var13], [var14], [var15], [var16], [var17], [var18], [var19], 
    [var20], [icon_id], [icon_file_name], [is_projectile], [projectile_speed], 
    [projectile_acceleration]
) VALUES (
    92000, 50092000, 0, 40092000, 1, '0', 
    '0', '0', '0', '0', 
    '0', '0', 0, '0', 
    '0', 0, 0, 0, 
    0, 0, 0, 0, 
    0.00, 0.00, 0.00, 0.00, 
    0, 0, 0.00, 0.00, 
    0, 0, 0, 0, 
    0, 0, 0, 0, 
    0, 0, 0, 0, 0, 
    0, 0, '0', 
    '0', '0', '0', '0', 
    '0', '0', '0', '0', 
    '0', '0', '0', '0', 
    '0', '0', '0', '0', 
    '1', 0.00, 0.00, 
    0.00, 1.50, 0.00, 
    0.00, 0.00, 
    0, '0', '0', '0', '0', 
    '0', '0', '0', '0', '0', 
    0, 41, 4, 0, 
    0, 0, 0.00, 0.00, 
    0.00, 0.00, 0.00, 
    0, 0, 0, 
    0, 0, 0.00, 0, 
    0.00, 0.00, 0, 0, 
    512.0000, 0.0000, 0.0500, 2.0000, 0.0000, 0.0200, 0.0000, 0.0000, 0.0000, 0.0000, 
    0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 
    0.0000, 1530, 'icon_skill_pas_mage_moral_culture', 0, 0.00, 
    0.00
);
```
