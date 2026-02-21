# Effect Types: EF_AMP_DAMAGE_BY_TARGET_STATE (32032) & EF_AMP_DAMAGE_INC_CRIT_RATE_BY_TARGET_HP_RATIO (32202)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [EF_MAGIC_SINGLE_DAMAGE (231) — Base Reference](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md)

Both types are **conditional damage amplifiers**: they deal magic damage using the standard 231 base formula, but include an additional mechanic that activates based on a **target-dependent condition**:

- **32032**: Amplifies damage when the target **has a specific state** active
- **32202**: Increases critical rate when the target is **below full HP** (scales with missing HP)

**Source Reference**: `Game/Skill/StructSkill.cpp`
- 32032 → `AMP_DAMAGE_BY_TARGET_STATE()`
- 32202 → `AMP_DAMAGE_INC_CRIT_RATE_BY_TARGET_HP_RATIO()`

---

## ⚠️ Generation Warning

| | 32032 | 32202 |
|---|---|---|
| **Condition** | Target has `var7` state ID | Target's HP is below maximum |
| **Effect when triggered** | Damage × `var8` bonus multiplier | Bonus critical rate added |
| **var7** | State ID to check (integer cast) | Crit rate bonus — base |
| **var8** | Damage multiplier when state present | Crit rate bonus — per SLV |
| **var9** | Multiplier per SLV | Crit rate bonus — per enhance |
| **state_id column** | ✅ On-hit state (normal) | ✅ On-hit state (normal) |

These types differ from type 231 only in the `var7`+ region. `var1`–`var6` are identical to 231.

---

## Shared Base: Damage Formula (var1–var6)

Both types use the standard magic damage base. See [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md#3-variable-mapping-var1var20) for the full reference:

```
nDamage = M.Atk × (var1 + var2×SLV + var3×Enhance)
        + (var4  + var5×SLV + var6×Enhance)
```

---

---

# EF_AMP_DAMAGE_BY_TARGET_STATE — Type 32032

## What This Skill Does

Deals standard single-target magic damage (231 formula). If the target currently has the state specified in `var7` active, the damage is multiplied by an additional factor derived from `var8`/`var9`. Useful for "execute a debuffed target for bonus damage" combos.

**Use this type for**: state-synergy damage skills, "if poisoned deal +50% damage" combo finishers, elemental synergy attacks.

## Variables Unique to Type 32032 (var7–var9)

| Variable | Type | Role | Notes |
|----------|------|------|-------|
| **`var7`** | integer (stored as float) | **Required state ID on target** | E.g., `12530.0000` = checks if state 12530 is active on target. `0.0000` disables the check. |
| **`var8`** | float | **Damage multiplier bonus when state active** | `1.5000` = deal 150% of normal damage (50% bonus). `1.0000` = no bonus. |
| **`var9`** | float | **Multiplier increase per SLV** | `0.0500` = +5% bonus per SLV. |
| `var10`–`var20` | — | **UNUSED** | Always `0.0000`. |

**Damage when state is present**:
```
nDamage_bonus = nDamage × (var8 + var9×SLV)
```

**Damage when state is absent**: standard formula result — no penalty, no bonus.

> If `var7 = 0.0000`, the engine disables the state check entirely and the bonus is never applied. Use `0.0000` only if you intend a pure 231-equivalent skill.

## Key Differences from Type 231

- `effect_type` = `32032`
- `var7` = the state ID to check (NOT UNUSED)
- `var8`/`var9` = the conditional bonus multiplier
- Everything else (damage formula, state columns, flags) is identical to 231

## Example INSERT — "Venomous Burst" (ID: 91261)

Dark-element spell: deals standard magic damage. If the target has poison state (12530), damage is multiplied by 1.5 (+5%/SLV). Applies a different debuff on hit (12531, 70% chance).

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
    91261, 50091261, 0, 40091261, 1, '6',   -- elemental='6' = Dark
    '1', '0', '1', '1',
    '0', '0', 0,
    '1', '1',
    20, 0,
    0, 0,
    120, 12, -8,
    0.00, 0.00, 0.00, 0.00,
    0, 0, 0.00, 0.00,
    0, 0, 0, 0,
    0, 0, 0,
    0, 0, 0, 0, 0, 0, 0,
    0,
    '0', '0', '0', '0', '0',
    '0', '0', '0', '0',
    '0', '0', '0', '0',
    '0', '0', '0', '0',
    '1',
    0.80, 0.00, 0.00,
    0.00,
    12.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',
    '1', '1', '1',
    0,
    1, 32032, 0,
    -- On-hit state: different debuff, 70% chance
    12531, 0, 1.00, 0.00,
    6.00, 0.00, 0.00,
    70, 0,
    0, 0, 0,
    1.20,
    0, 0.00, 0.00,
    0, 0,
    -- Base damage (same as 231 formula)
    1.2000, 0.0800, 0.0000,   -- var1: 120% M.Atk, var2: +8%/SLV
    100.0000, 15.0000, 0.0000, -- var4: 100 flat, var5: +15/SLV
    -- 32032-specific: conditional state bonus
    12530.0000,                -- var7: check for poison state ID 12530
    1.5000,                    -- var8: 1.5× damage multiplier when state active
    0.0500,                    -- var9: +5% multiplier per SLV
    -- UNUSED (var10–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000,
    1800, 'icon_skill_active_mage_venomous_burst', 0, 0.00, 0.00
);
```

---

---

# EF_AMP_DAMAGE_INC_CRIT_RATE_BY_TARGET_HP_RATIO — Type 32202

## What This Skill Does

Deals standard single-target magic damage (231 formula), but adds a **bonus critical rate** that scales with how much HP the target is missing. The lower the target's HP, the higher the chance of the hit being a critical. This creates an "execute" dynamic — weakened targets are more likely to receive a finishing critical blow.

**Use this type for**: execute spells, low-HP finishers, predator-style crit scaling.

**Critical rate bonus formula**:
```
bonus_crit_rate = (var7 + var8×SLV + var9×Enhance) × (1 − target_HP_ratio)
```
Where `target_HP_ratio = target_current_HP / target_max_HP` (1.0 at full HP, 0.0 at death).

At full HP: `bonus_crit_rate = 0` (no bonus).
At 50% HP: bonus = `(var7 + …) × 0.5`.
At 1% HP: bonus ≈ `var7 + …` (near maximum).

## Variables Unique to Type 32202 (var7–var9)

| Variable | Type | Role | Notes |
|----------|------|------|-------|
| **`var7`** | float | **Max bonus crit rate** (at 0 HP) | `0.5000 = 50%` additional crit chance when target is nearly dead. Uses `1 = 100%` convention. |
| **`var8`** | float | **Max bonus — per SLV** | `0.0500 = +5%` max crit bonus per SLV. |
| **`var9`** | float | **Max bonus — per enhance** | |
| `var10`–`var20` | — | **UNUSED** | Always `0.0000`. |

> The bonus crit rate is added to the skill's existing `critical_bonus`. At full HP it has zero effect — there is no penalty to hitting healthy targets.

## Key Differences from Type 231

- `effect_type` = `32202`
- `var7`–`var9` define the max crit rate bonus (HP-ratio-scaled)
- The `critical_bonus` column stacks additively with the `var7`-derived bonus
- Everything else (damage formula, state, flags) is identical to 231

## Example INSERT — "Death's Mark" (ID: 91262)

Fire-element execute spell: 130% M.Atk base damage. Gains up to +40% critical rate bonus (+5%/SLV) against targets near death.

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
    91262, 50091262, 0, 40091262, 1, '2',   -- elemental='2' = Fire
    '1', '0', '1', '1',
    '0', '0', 0,
    '1', '1',
    20, 0,
    0, 0,
    130, 14, -10,
    0.00, 0.00, 0.00, 0.00,
    0, 0, 0.00, 0.00,
    0, 0, 0, 0,
    0, 0, 0,
    0, 0, 0, 0, 0, 0, 0,
    0,
    '0', '0', '0', '0', '0',
    '0', '0', '0', '0',
    '0', '0', '0', '0',
    '0', '0', '0', '0',
    '1',
    0.80, 0.00, 0.00,
    0.00,
    14.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',
    '1', '1', '1',
    0,
    1, 32202, 0,
    -- No on-hit state
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.20,
    0, 0.00, 0.00,
    0, 0,                      -- critical_bonus = 0 (var7 handles dynamic crit scaling)
    -- Base damage (same as 231 formula)
    1.3000, 0.0900, 0.0000,
    90.0000, 12.0000, 0.0000,
    -- 32202-specific: HP-ratio crit rate bonus
    0.4000,                    -- var7: max +40% crit rate (at 0 HP)
    0.0500,                    -- var8: +5% max bonus per SLV
    0.0000,                    -- var9: no enhance scaling
    -- UNUSED (var10–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000,
    1850, 'icon_skill_active_mage_deaths_mark', 0, 0.00, 0.00
);
```

---

## Full Comparison: Types 231 / 32032 / 32202

| | 231 | 32032 | 32202 |
|---|---|---|---|
| **Base damage** | M.Atk × var1–6 | M.Atk × var1–6 (same) | M.Atk × var1–6 (same) |
| **Conditional mechanic** | None | State check on target | HP ratio check on target |
| **var7** | UNUSED | Required state ID (integer) | Max bonus crit rate (float) |
| **var8** | UNUSED | Damage multiplier if state present | Bonus crit per SLV |
| **var9** | UNUSED | Multiplier per SLV | Bonus crit per enhance |
| **var10–20** | UNUSED | UNUSED | UNUSED |
| **state_id column** | ✅ Active | ✅ Active (independent of var7) | ✅ Active |
| **Bonus fires when** | — | Target has state var7 | Target HP < max |

---

## Practical Constraints

### EF_AMP_DAMAGE_BY_TARGET_STATE (32032)
- **`var8` must be ≥ `1.0000`** if the intent is a damage bonus. A value of `1.0000` means no bonus (×1 = same damage). Values below 1.0 would be a damage penalty if the state is present — only use intentionally.
- **`var7` is the state ID, not the `state_id` column**: the `state_id` column controls what the *skill applies on hit* (separate from what it *checks on the target*). These two state IDs are completely independent.
- **State must already be on the target**: this skill does not apply the required state itself. Use a preceding debuff skill (type 301 or 231 with `state_id`) to apply the state first.

### EF_AMP_DAMAGE_INC_CRIT_RATE_BY_TARGET_HP_RATIO (32202)
- **`var7 = 1.0000` (100% max bonus) is not a cap**: the bonus crit stacks with `critical_bonus`. Set carefully to avoid unintentional guaranteed-crit against low-HP targets.
- **No effect at full HP**: this type is not useful for skills intended to be impactful at all HP thresholds — use standard `critical_bonus` for flat crit rate boosts.

---

*Source: `Game/Skill/StructSkill.cpp` · `AMP_DAMAGE_BY_TARGET_STATE()` · `AMP_DAMAGE_INC_CRIT_RATE_BY_TARGET_HP_RATIO()`*
