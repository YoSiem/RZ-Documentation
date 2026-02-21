# Effect Types: EF_MAGIC_MULTIPLE_DAMAGE (232) & EF_MAGIC_DAMAGE_WITH_ABSORB_HP_MP (235)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [EF_MAGIC_SINGLE_DAMAGE (231) — Base Reference](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md)

These two effect types are documented together because they share the same **base damage formula** as [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md). Beyond the shared foundation, they are **completely different mechanics** and must never be confused when generating skills.

---

## ⚠️ Generation Warning — Read Before Creating Any Skill

| | 232 | 235 |
|---|---|---|
| **var7** | Hit **count** (integer, e.g. `3.0000`) | HP **absorb ratio** (float, e.g. `0.2500`) |
| **var8** | Hit count **per SLV** | HP absorb **per SLV** |
| **var9** | **Delay** between hits in seconds | HP absorb **per enhance** |
| **var10** | UNUSED | MP **absorb ratio** (float) |
| **state_id** | ✅ Works normally | ❌ Ignored by engine |

These variables are completely different. Getting them wrong produces either a non-functional skill or a skill with broken behavior.

---

## Shared Base: Damage Formula (var1–var6)

`var1`–`var6` work **identically** in both types and follow the same rules as [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md#3-variable-mapping-var1var20):

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

All other shared columns (`elemental`, `hit_bonus`, `critical_bonus`, weapon flags, faction flags, etc.) follow [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md) exactly.

---

---

# EF_MAGIC_MULTIPLE_DAMAGE — Type 232

## What This Skill Does

Fires the same magic damage formula as type 231, but **hits the target multiple times in sequence**. Each hit is a full, independent damage calculation — critical chance applies to each hit separately. The number of hits and the delay between them are controlled by `var7`, `var8`, and `var9`.

**Use this type for**: rapid-fire spells, multi-projectile skills, channeled magic bursts.

**Source Reference**: `Game/Skill/StructSkill.cpp` → `MULTIPLE_MAGICAL_DAMAGE()`

## Variables Unique to Type 232 (var7–var9)

| Variable | Type | Role | Notes |
|----------|------|------|-------|
| **`var7`** | integer (stored as float) | **Hit count — base** | Must be ≥ 1. `3.0000` = hits 3 times. |
| **`var8`** | integer (stored as float) | **Hit count — per skill level** | `1.0000` = +1 extra hit per SLV. Usually `0.0000`. |
| **`var9`** | float | **Delay between hits (seconds)** | Engine multiplies by 100 internally. `0.2000` = 200 ms gap. |
| `var10`–`var20` | — | **UNUSED** | Always `0.0000`. |

**Total hit count formula**: `hits = int(var7) + int(var8) × SLV`

> A skill at SLV 5 with `var7 = 2.0`, `var8 = 1.0` hits **7 times**.
> At `var9 = 0.3`, there is a 300 ms gap between each hit.

## Key Differences from Type 231

- `effect_type` = `232` (not `231`)
- `var7`–`var9` are **required and meaningful** (hit count + delay)
- `state_id` works the same as in 231 — state is applied independently on each hit that lands
- Everything else (damage formula, columns, flags) is identical to 231

## Example INSERT — "Storm Volley" (ID: 91201)

Fire-element spell that strikes 3 times, each hit at 80% M.Atk + 60 flat, 200 ms between hits. No on-hit state.

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
    91201, 50091201, 0, 40091201, 1, '2',
    '1', '0', '1', '1',
    '0', '0', 0,
    '1', '1',
    20, 20,
    0, 0,
    150, 15, -10,
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
    1.00, 0.00, 0.00,
    0.00,
    18.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',
    '1', '1', '1',
    0,
    1, 232, 0,
    -- No on-hit state
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.10,
    0, 0.00, 0.00,
    0, 0,
    -- Damage (shared formula, same as 231)
    0.8000, 0.0500, 0.0000,   -- var1: 80% M.Atk, var2: +5%/SLV, var3: 0
    60.0000, 8.0000, 0.0000,  -- var4: 60 flat, var5: +8/SLV, var6: 0
    -- 232-specific: multi-hit
    3.0000, 0.0000, 0.2000,   -- var7: 3 hits, var8: no per-SLV scaling, var9: 200ms delay
    -- UNUSED (var10–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000,
    1530, 'icon_skill_active_mage_storm_volley', 0, 0.00, 0.00
);
```

---

---

# EF_MAGIC_DAMAGE_WITH_ABSORB_HP_MP — Type 235

## What This Skill Does

Deals single-target magic damage (same formula as 231), then immediately **converts a fraction of the damage dealt into HP and/or MP for the caster**. Both HP and MP absorb are independent and optional.

**Use this type for**: life-drain spells, soul-steal skills, mana-siphon attacks.

**Source Reference**: `Game/Skill/StructSkill.cpp` → `SINGLE_MAGICAL_DAMAGE_WITH_ABSORB()`

## ⚠️ State-on-Hit Is Disabled for This Type

Type 235 is **explicitly excluded** from the engine's state-on-hit processing. Even if `state_id` is set to a valid value, the engine will ignore it. Always use:

```sql
[state_id] = 0,
[probability_on_hit] = 0,
[probability_inc_by_slv] = 0
```

## Variables Unique to Type 235 (var7–var12)

| Variable | Type | Role | Notes |
|----------|------|------|-------|
| **`var7`** | float | **HP absorb — base ratio** | Fraction of damage returned as HP. `0.2000` = 20%. |
| **`var8`** | float | **HP absorb — per skill level** | `0.0100` = +1% per SLV. |
| **`var9`** | float | **HP absorb — per enhance** | Usually `0.0000`. |
| **`var10`** | float | **MP absorb — base ratio** | Fraction of damage returned as MP. `0.1000` = 10%. |
| **`var11`** | float | **MP absorb — per skill level** | `0.0050` = +0.5% per SLV. |
| **`var12`** | float | **MP absorb — per enhance** | Usually `0.0000`. |
| `var13`–`var20` | — | **UNUSED** | Always `0.0000`. |

**Absorb formulas**:
```
HP restored = Damage dealt × (var7 + var8×SLV + var9×Enhance)
MP restored = Damage dealt × (var10 + var11×SLV + var12×Enhance)
```

To use only HP absorb (no MP), set `var10`–`var12 = 0.0000`. To use only MP absorb, set `var7`–`var9 = 0.0000`.

## Key Differences from Type 231

- `effect_type` = `235` (not `231`)
- `var7`–`var12` are **required and meaningful** (absorb ratios)
- `state_id` = `0` always — on-hit state is disabled for this type
- Everything else (damage formula, columns, flags) is identical to 231

## Example INSERT — "Soul Drain" (ID: 91202)

Dark-element single-target drain: 120% M.Atk base damage, returns 25% as HP and 10% as MP.

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
    91202, 50091202, 0, 40091202, 1, '6',   -- elemental='6' = Dark
    '1', '0', '1', '1',
    '0', '0', 0,
    '1', '1',
    20, 20,
    0, 0,
    110, 12, -8,
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
    12.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',
    '1', '1', '1',
    0,
    1, 235, 0,
    -- State disabled for type 235 — always zero
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.00,
    0, 0.00, 0.00,
    0, 0,
    -- Damage (shared formula, same as 231)
    1.2000, 0.0800, 0.0000,   -- var1: 120% M.Atk, var2: +8%/SLV, var3: 0
    50.0000, 10.0000, 0.0000, -- var4: 50 flat, var5: +10/SLV, var6: 0
    -- 235-specific: absorb ratios
    0.2500, 0.0050, 0.0000,   -- var7: 25% HP return, var8: +0.5%/SLV, var9: 0
    0.1000, 0.0025, 0.0000,   -- var10: 10% MP return, var11: +0.25%/SLV, var12: 0
    -- UNUSED (var13–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    1530, 'icon_skill_active_mage_soul_drain', 0, 0.00, 0.00
);
```

---

## Full Comparison: Types 231 / 232 / 235

| | 231 | 232 | 235 |
|---|---|---|---|
| **In-game effect** | 1× magic hit | N× magic hits in sequence | 1× magic hit + HP/MP return |
| **Damage formula** | var1–6 | var1–6 (same) | var1–6 (same) |
| **var7** | UNUSED | Hit count base | HP absorb ratio |
| **var8** | UNUSED | Hit count per SLV | HP absorb per SLV |
| **var9** | UNUSED | Delay between hits (s) | HP absorb per enhance |
| **var10** | UNUSED | UNUSED | MP absorb ratio |
| **var11** | UNUSED | UNUSED | MP absorb per SLV |
| **var12** | UNUSED | UNUSED | MP absorb per enhance |
| **var13–20** | UNUSED | UNUSED | UNUSED |
| **state_id** | ✅ Active | ✅ Active (per hit) | ❌ Disabled |
| **effect_type value** | `231` | `232` | `235` |
