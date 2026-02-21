# Effect Types: EF_MAGIC_SINGLE_REGION_DAMAGE (261), EF_MAGIC_SPECIAL_REGION_DAMAGE (262) & EF_MAGIC_MULTIPLE_REGION_DAMAGE (263)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [EF_MAGIC_SINGLE_DAMAGE (231) — Base Reference](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md) | [EF_MAGIC_MULTIPLE_DAMAGE (232) & EF_MAGIC_DAMAGE_WITH_ABSORB_HP_MP (235)](13_Effect_Type_Magic_Damage_Variants.md)

These three effect types are the **AoE (region) counterparts** of the single-target magic damage family. They share the same base damage formula as [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md) but apply damage to **all valid enemies within a region** rather than a single target.

**Source Reference**: `Game/Skill/StructSkill.cpp`
- 261 → `SINGLE_REGION_MAGICAL_DAMAGE()`
- 262 → `SPECIAL_REGION_MAGICAL_DAMAGE()`
- 263 → `MULTIPLE_REGION_MAGICAL_DAMAGE()`

---

## ⚠️ Generation Warning — Read Before Creating Any Skill

| | 261 | 262 | 263 |
|---|---|---|---|
| **In-game effect** | Single AoE hit around target | Ground-targeted AoE hit | Multi-hit AoE |
| **target column** | `2` (region w/ target) | `3` (region w/o target) | `2` (region w/ target) |
| **is_need_target** | `'1'` | `'0'` | `'1'` |
| **var7** | UNUSED | UNUSED | Hit **count** base |
| **var8** | UNUSED | UNUSED | Hit count **per SLV** |
| **var9** | UNUSED | UNUSED | **Delay** between hits (s) |
| **valid_range** | AoE **radius** | AoE **radius** | AoE **radius** |
| **state_id** | ✅ Works normally | ✅ Works normally | ✅ Active (per hit) |

The critical difference from the single-target family is `target` and `is_need_target`. Getting those wrong silently converts a region spell into a single-target or self-cast skill.

---

## Shared Base: Damage Formula (var1–var6)

`var1`–`var6` work **identically** in all three types and follow the same rules as [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md#3-variable-mapping-var1var20):

```
nDamage = M.Atk × (var1 + var2×SLV + var3×Enhance)
        + (var4  + var5×SLV + var6×Enhance)
```

| Variable | Role |
|----------|------|
| `var1` | M.Atk multiplier — base |
| `var2` | M.Atk multiplier — per skill level |
| `var3` | M.Atk multiplier — per enhance |
| `var4` | Flat damage — base |
| `var5` | Flat damage — per skill level |
| `var6` | Flat damage — per enhance |

The damage calculated is applied independently to **each target hit** within the region. There is no split or reduction per target — every enemy in range takes the full formula result.

All shared columns (`elemental`, `hit_bonus`, `critical_bonus`, weapon flags, etc.) follow [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md) exactly.

---

---

# EF_MAGIC_SINGLE_REGION_DAMAGE — Type 261

## What This Skill Does

Fires the same magic damage formula as type 231, but hits **every valid enemy within a circular region around the selected target**. The caster must have a valid target to center the AoE on. The radius of the AoE is controlled by `valid_range`.

**Use this type for**: target-centered explosions, elemental bursts, nova spells.

## AoE Targeting Columns (unique to region types)

| Column | Value | Reason |
|--------|-------|--------|
| `target` | `2` | `TARGET_REGION` — region centered on selected target |
| `is_need_target` | `'1'` | A target must be selected to anchor the AoE center |
| `valid_range` | e.g. `8` | Radius of the AoE in engine units |
| `cast_range` | e.g. `20` | Maximum distance to the target from the caster |

## Variables for Type 261

| Variable | Type | Role |
|----------|------|------|
| `var1`–`var6` | float | Damage formula — identical to 231 |
| `var7`–`var20` | — | **UNUSED** — always `0.0000` |

## Key Differences from Type 231

- `effect_type` = `261` (not `231`)
- `target` = `2` instead of `1`
- `valid_range` defines the AoE radius (not just the hit cap for a single target)
- Everything else (damage formula, state, columns, flags) is identical to 231

## Example INSERT — "Arcane Explosion" (ID: 91210)

Wind-element AoE explosion centered on target. Hits all enemies within 8 units. 110% M.Atk + 80 flat, applies a slow state (9998) with 50% chance.

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
    91210, 50091210, 0, 40091210, 1, '4',   -- elemental='4' = Wind
    '1', '0', '1', '1',                      -- active, not physical, harmful, needs target
    '0', '0', 0,
    '1', '1',
    20, 8,                                    -- cast_range=20, valid_range=8 (AoE radius)
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
    1.20, 0.00, 0.00,
    0.00,
    16.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',
    '1', '1', '1',
    0,
    2, 261, 0,                                -- target=2 (REGION w/ target), effect_type=261
    -- On-hit state: slow debuff 50% chance
    9998, 0, 1.00, 0.00,
    4.00, 0.00, 0.00,
    50, 0,
    0, 0, 0,
    1.30,                                     -- hate_mod = 1.3x (high-aggro AoE)
    0, 0.00, 0.00,
    0, 0,
    -- Damage
    1.1000, 0.0700, 0.0000,   -- var1: 110% M.Atk, var2: +7%/SLV, var3: 0
    80.0000, 12.0000, 0.0000, -- var4: 80 flat, var5: +12/SLV, var6: 0
    -- UNUSED (var7–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    1100, 'icon_skill_active_mage_arcane_explosion', 0, 0.00, 0.00
);
```

---

---

# EF_MAGIC_SPECIAL_REGION_DAMAGE — Type 262

## What This Skill Does

A **ground-targeted** AoE magic damage spell. Unlike type 261, no hostile entity needs to be selected — the caster places the skill directly on a map position. All enemies within `valid_range` of that point are hit with the standard magic damage formula.

**Use this type for**: ground-target meteor strikes, manually placed AoE zones, delayed-detonation spells.

## AoE Targeting Columns (unique to type 262)

| Column | Value | Reason |
|--------|-------|--------|
| `target` | `3` | `TARGET_REGION_WITHOUT_TARGET` — ground-target, no entity required |
| `is_need_target` | `'0'` | No hostile entity must be selected |
| `valid_range` | e.g. `10` | Radius of the AoE in engine units |
| `cast_range` | e.g. `25` | Maximum distance from caster to the ground point |

> With `target = 3` and `is_need_target = '0'`, the player clicks on the ground to place the AoE center. The engine automatically hits all enemies within `valid_range` of that point.

## Variables for Type 262

| Variable | Type | Role |
|----------|------|------|
| `var1`–`var6` | float | Damage formula — identical to 231 |
| `var7`–`var20` | — | **UNUSED** — always `0.0000` |

## Key Differences from Type 261

- `effect_type` = `262` (not `261`)
- `target` = `3` instead of `2`
- `is_need_target` = `'0'` (ground-targeted — no entity needed)
- `cast_range` represents the maximum distance to the **ground point**, not to a creature
- Everything else (damage formula, state, columns, flags) is identical to 261

## Example INSERT — "Meteor Strike" (ID: 91211)

Fire-element ground-targeted AoE. Player clicks a point; all enemies within radius 10 are hit. 140% M.Atk + 120 flat, no on-hit state.

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
    91211, 50091211, 0, 40091211, 1, '2',   -- elemental='2' = Fire
    '1', '0', '1', '0',                      -- active, not physical, harmful, NO target needed
    '0', '0', 0,
    '1', '1',
    25, 10,                                   -- cast_range=25 (to ground point), valid_range=10 (AoE radius)
    0, 0,
    180, 20, -15,
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
    2.00, 0.00, 0.00,                         -- 2-second cast time
    0.00,
    20.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',
    '1', '1', '1',
    0,
    3, 262, 0,                                -- target=3 (REGION w/o target), effect_type=262
    -- No on-hit state
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.50,                                     -- hate_mod = 1.5x (major AoE = major aggro)
    0, 0.00, 0.00,
    0, 0,
    -- Damage
    1.4000, 0.1000, 0.0000,   -- var1: 140% M.Atk, var2: +10%/SLV, var3: 0
    120.0000, 18.0000, 0.0000, -- var4: 120 flat, var5: +18/SLV, var6: 0
    -- UNUSED (var7–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    1300, 'icon_skill_active_mage_meteor_strike', 0, 0.00, 0.00
);
```

---

---

# EF_MAGIC_MULTIPLE_REGION_DAMAGE — Type 263

## What This Skill Does

Combines the multi-hit mechanic of [EF_MAGIC_MULTIPLE_DAMAGE (232)](13_Effect_Type_Magic_Damage_Variants.md) with region AoE targeting. The spell strikes **all enemies in the region, multiple times in sequence**. Each hit is a full, independent damage calculation — critical chance applies per hit per target. The number of hits and the delay between them are controlled by `var7`, `var8`, and `var9`, identical to type 232.

**Use this type for**: channeled AoE storms, multi-wave explosions, rapid-fire area bursts.

## Variables Unique to Type 263 (var7–var9)

| Variable | Type | Role | Notes |
|----------|------|------|-------|
| **`var7`** | integer (stored as float) | **Hit count — base** | Must be ≥ 1. `3.0000` = 3 hits per target. |
| **`var8`** | integer (stored as float) | **Hit count — per skill level** | `1.0000` = +1 hit per SLV. Usually `0.0000`. |
| **`var9`** | float | **Delay between hits (seconds)** | `0.3000` = 300 ms gap between each wave. |
| `var10`–`var20` | — | **UNUSED** | Always `0.0000`. |

**Total hit count formula**: `hits = int(var7) + int(var8) × SLV`

> A skill at SLV 5 with `var7 = 2.0`, `var8 = 1.0` hits each target in the region **7 times**.

## AoE Targeting Columns

| Column | Value | Reason |
|--------|-------|--------|
| `target` | `2` | `TARGET_REGION` — region centered on selected target |
| `is_need_target` | `'1'` | A target must be selected to anchor the AoE center |
| `valid_range` | e.g. `8` | Radius of the AoE in engine units |

## Key Differences from Type 232

- `effect_type` = `263` (not `232`)
- `target` = `2` instead of `1` — AoE, not single-target
- `valid_range` defines the AoE radius
- `var7`–`var9` behave **identically** to type 232 (hit count + delay)
- `state_id` works the same as 232 — state is applied independently on each hit that lands on each target

## Key Differences from Type 261

- `effect_type` = `263` (not `261`)
- `var7`–`var9` are **required and meaningful** (hit count + delay)
- Multiple waves hit the entire AoE region each time

## Example INSERT — "Blizzard" (ID: 91212)

Water-element multi-hit AoE. Targets a region around selected enemy; hits all enemies 3 times with 200 ms gaps. 70% M.Atk + 40 flat per hit. Applies a chill state (9997) on each hit with 30% chance.

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
    91212, 50091212, 0, 40091212, 1, '3',   -- elemental='3' = Water
    '1', '0', '1', '1',                      -- active, not physical, harmful, needs target
    '0', '0', 0,
    '1', '1',
    20, 8,                                    -- cast_range=20, valid_range=8 (AoE radius)
    0, 0,
    200, 22, -15,
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
    1.50, 0.00, 0.00,
    0.00,
    22.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',
    '1', '1', '1',
    0,
    2, 263, 0,                                -- target=2 (REGION w/ target), effect_type=263
    -- On-hit state: chill debuff 30% chance per hit
    9997, 0, 1.00, 0.00,
    5.00, 0.00, 0.00,
    30, 0,
    0, 0, 0,
    1.40,                                     -- hate_mod = 1.4x aggro
    0, 0.00, 0.00,
    0, 0,
    -- Damage (per hit, full formula applies each wave)
    0.7000, 0.0500, 0.0000,   -- var1: 70% M.Atk, var2: +5%/SLV, var3: 0
    40.0000, 6.0000, 0.0000,  -- var4: 40 flat, var5: +6/SLV, var6: 0
    -- 263-specific: multi-hit
    3.0000, 0.0000, 0.2000,   -- var7: 3 hits, var8: no per-SLV scaling, var9: 200ms delay
    -- UNUSED (var10–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000,
    1200, 'icon_skill_active_mage_blizzard', 0, 0.00, 0.00
);
```

---

## Full Comparison: Types 261 / 262 / 263 (and their single-target siblings)

| | 231 | 261 | 262 | 263 |
|---|---|---|---|---|
| **In-game effect** | 1× single-target hit | 1× AoE hit (on-target) | 1× AoE hit (ground) | N× AoE hits |
| **target** | `1` | `2` | `3` | `2` |
| **is_need_target** | `'1'` | `'1'` | `'0'` | `'1'` |
| **Damage formula** | var1–6 | var1–6 (same) | var1–6 (same) | var1–6 (same) |
| **var7** | UNUSED | UNUSED | UNUSED | Hit count base |
| **var8** | UNUSED | UNUSED | UNUSED | Hit count per SLV |
| **var9** | UNUSED | UNUSED | UNUSED | Delay between hits |
| **var10–20** | UNUSED | UNUSED | UNUSED | UNUSED |
| **state_id** | ✅ Active | ✅ Active | ✅ Active | ✅ Active (per hit) |
| **Single-target sibling** | — | 231 | — | 232 |

---

*Source: `Game/Skill/StructSkill.cpp` · `SINGLE_REGION_MAGICAL_DAMAGE()` · `SPECIAL_REGION_MAGICAL_DAMAGE()` · `MULTIPLE_REGION_MAGICAL_DAMAGE()`*
