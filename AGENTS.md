# Instructions for AI Agents: Rappelz Skill Database Documentation & Generation

> **Context for AI**: This is the master instruction set for all AI agents working on Rappelz skill documentation and SQL generation. Before creating any new skill or documentation, read [AI-DOCUMENTATION-GUIDELINES.md](AI-DOCUMENTATION-GUIDELINES.md) to ensure consistency, avoid duplication, and maintain proper cross-linking. Do not invent columns, and always provide valid T-SQL `INSERT` statements.

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

---

# Procedure: Investigating and Documenting New Effect Types

> **⚠️ CRITICAL**: When tasked with analyzing a **new effect_type** (e.g., `EF_INSTANT_DAMAGE`, `EF_SUMMON_DURATION`), you MUST follow the procedure outlined in [AI-DOCUMENTATION-GUIDELINES.md](AI-DOCUMENTATION-GUIDELINES.md). This section provides a quick reference checklist.

## Step 1: Confirm the Effect Type ID
- User provides an effect type number or name (e.g., "effect_type = 5" or "EF_INSTANT_DAMAGE")
- Confirm you understand what this effect does in-game context
- Example: `effect_type = 4` = Passive stat amplification, `effect_type = 1` = Active instant damage

## Step 2: Locate Source Code Reference
Navigate to: `D:\Rappelz_source\program\server\Rappelz-Game-Server`

**Common file patterns to search**:
- `StructCreature.cpp` / `StructCreature.h` – Entity effect application
- `SkillEffect.cpp` / `SkillEffect.h` – Skill-specific effect logic
- `Define_Effect.h` / `Define_*.h` – Effect type definitions and constants
- `SkillResource.h` – Database column mappings

**Search command pattern**:
```bash
find "D:\Rappelz_source" -type f \( -name "*.h" -o -name "*.cpp" \) \
  | xargs grep -l "EF_PARAMETER_AMP\|effect_type.*=.*4"
```

## Step 3: Analyze the Handler Function

Once located, document:

| Aspect | What to Record |
|--------|---|
| **Function Name** | e.g., `applyPassiveSkillAmplifyEffect` |
| **Parameter Processing** | Which `var` fields does it read? (e.g., `GetVar(0)` through `GetVar(17)`) |
| **Type Interpretation** | Are vars treated as `int`, `float`, `bitmask`, `enum`, etc.? |
| **Range Validation** | Any bounds checks or sanity guards? (e.g., `if (var > 100) var = 100`) |
| **Side Effects** | Does it trigger other mechanics? (e.g., state application, cooldown reset) |

### Example Analysis Output:
```
Effect Type: 4 (EF_PARAMETER_AMP)
Handler: StructCreature::applyPassiveSkillAmplifyEffect()
Source: StructCreature.cpp:1250

Variables Used:
- var1: Bitmask for stat selection (uses Standard Bitmasks table)
- var2: Base percentage modifier (float, 1.0 = 100%)
- var3: Level scaling multiplier (float, per skill level)
- var4–var6: Second stat slot (same structure as var1–3)
- var7–var9: Extended stats (uses Extended Bitmasks table)
- var10–var12: Secondary extended slot
- var13–var18: Additional stat slots (var13/16 = bitmask, var14/17 = base, var15/18 = level scale)
- var19–var20: UNUSED (safe to set to 0)

Mandatory Columns:
- is_passive = '0' (negation: 0 means passive)
- effect_type = 4
- target = 41 (safe for self/summon)
- All costs = 0
```

## Step 4: Find and Reverse-Engineer Existing Examples
Query the database (or documentation) for existing skills using this effect_type:

```sql
SELECT id, text_id, var1, var2, var3, var4, var5, var6, ...
FROM SkillResource
WHERE effect_type = 4
LIMIT 3;
```

For each example:
- What is the skill's in-game effect?
- What stat does var1 represent?
- How are var2 and var3 combined to calculate the buff?
- Document the pattern

## Step 5: Create Documentation File
Follow the naming convention: `XX_Effect_Type_[NAME].md`

**Structure** (from [AI-DOCUMENTATION-GUIDELINES.md](AI-DOCUMENTATION-GUIDELINES.md#documentation-structure-per-effect-type)):
1. Overview of the effect
2. C++ source file reference
3. Variable mapping (each var's purpose, type, expected range)
4. Bitmask/encoding reference (link to existing or explain new)
5. Mandatory static SQL columns
6. Practical constraints
7. Complete working example INSERT

### Example Skeleton:
```markdown
# Effect Type: EF_INSTANT_DAMAGE (5)

## Overview
Applies instantaneous damage to a target, optionally triggering on-hit effects.

## C++ Source
- **File**: `SkillEffect.cpp`
- **Function**: `SkillEffect::applyInstantDamage()`
- **Lines**: 2150–2200

## Variable Mapping
- **var1**: Damage type flag (0 = Physical, 1 = Magical, 2 = Hybrid)
- **var2**: Base damage modifier (float, 1.0 = 100% weapon damage)
- **var3**: Skill level scaling (float, added per level)
- ...

## Mandatory Columns
- `effect_type`: 5
- `is_physical_act`: (1 if var1=0, 0 if var1=1)
- `is_harmful`: 1
- `state_id`: [optional – if on-hit buff]
- ...

## Example: Fireball Spell (ID: 91100)
...full INSERT statement...
```

## Step 6: Link to Existing Documentation

**Use relative markdown links to avoid duplication**:
```markdown
For stat bitmask values, see [Standard Bitmasks](02_Standard_Bitmasks.md#bitmask-section).
For extended stat encoding, see [Extended Bitmasks](03_Extended_Bitmasks.md).
For column definitions, see [Database Architecture](01_Skill_Database_Architecture.md).
```

## Step 7: Provide a Working Example

At minimum, include ONE realistic skill using this effect_type:

**Requirements**:
- All 94 SQL columns present in exact order
- Realistic values (no placeholder text like `[var1_value]`)
- Valid integer and float types
- SQL syntax is correct and ready to execute
- Inline comments explaining key var values

### Template Example:
```sql
-- Skill Name: [Skill Description]
-- Effect: [Brief description]
-- Key Variables:
--   var1 = [explanation] (value: N)
--   var2 = [explanation] (value: N)

INSERT INTO [dbo].[SkillResource] (
    [id], [text_id], [desc_id], [tooltip_id], [is_valid], [elemental],
    ... [all 94 columns] ...
) VALUES (
    91100, 50091100, 0, 40091100, 1, '0',  -- ID starts at 91000 range
    ... [all 94 values] ...
    -- Key vars: var1=1 (magical dmg), var2=2.5 (250% base), var3=0.1 (10% per level)
    1.0000, 2.5000, 0.1000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, ...
);
```

## Step 8: Update Table of Contents
After creating a new effect_type documentation file:

1. Add entry to `00_Table_of_Contents.md`
2. Follow numbering: `12–20` are reserved for effect_type deep dives
3. Provide one-line description
4. Use relative link

**Example**:
```markdown
### [12. Effect Type: EF_INSTANT_DAMAGE](12_Effect_Type_EF_INSTANT_DAMAGE.md)
Damage-dealing effects with customizable damage type (physical/magical) and scaling.
```

---

# Quick Reference: Mandatory Columns by Effect Type

> **For use when creating skills**: The columns below MUST match the effect_type. Refer to this table before generating SQL.

| Effect Type | ID | effect_type | is_physical_act | is_harmful | Typical var Count | Key Points |
|---|---|---|---|---|---|---|
| EF_PARAMETER_AMP | 4 | `4` | `'0'` | `'0'` | var1–18 | Passive buff, bitmask-based |
| EF_INSTANT_DAMAGE | 1 | `1` | (depends) | `'1'` | var1–10 | Single/AoE damage, immediate |
| EF_SUMMON_DURATION | ? | `?` | `'0'` | `'0'` | var1–6 | Summon properties, duration |
| [More to be documented] | – | – | – | – | – | See `1X_Effect_Type_*.md` |

> **Note**: This table is a quick reference. For detailed analysis of each effect type, consult the dedicated documentation file (e.g., `04_Creating_Passive_Skill_EF_PARAMETER_AMP.md`).

---

# Procedure Checklist: Before Submitting Documentation

- [ ] Source code file path and function name documented
- [ ] All `var1`–`var20` explained or marked "UNUSED"
- [ ] Safe default/fallback values specified
- [ ] At least ONE complete, syntax-valid INSERT statement provided
- [ ] No information duplicated across files (use links instead)
- [ ] All relative links tested and functional
- [ ] Bitmask tables either linked to existing docs or justified as new
- [ ] SQL column order matches [AGENTS.md](AGENTS.md) schema (94 columns)
- [ ] File named according to convention: `XX_Effect_Type_[NAME].md`
- [ ] Table of Contents updated if file is new
- [ ] Follows [AI-DOCUMENTATION-GUIDELINES.md](AI-DOCUMENTATION-GUIDELINES.md) best practices
