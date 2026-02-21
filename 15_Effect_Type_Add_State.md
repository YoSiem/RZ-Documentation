# Effect Types: EF_ADD_STATE (301), EF_ADD_REGION_STATE (302) & EF_ADD_STATE_TO_CASTER_AND_TARGET (311)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [States and Enhancements Reference](10_Skill_States_and_Enhancements.md) | [Skill Target and Usage](09_Skill_Target_and_Usage.md)

These three effect types are the **pure state application** family. They apply a buff or debuff (a `state_id` entry) to target(s) without dealing any damage. The entire mechanic is driven by the **state columns**, not the var columns — `var1`–`var20` are unused.

**Source Reference**: `Game/Skill/StructSkill.cpp`
- 301 → `ADD_STATE()`
- 302 → `ADD_REGION_STATE()`
- 311 → `ADD_STATE_TO_CASTER_AND_TARGET()`

---

## ⚠️ Generation Warning — Read Before Creating Any Skill

| | 301 | 302 | 311 |
|---|---|---|---|
| **Applies to** | Selected target | All enemies/allies in region | Selected target **and** caster |
| **target** | `1` (single target) | `2` (region w/ target) | `1` (single target) |
| **is_need_target** | `'1'` | `'1'` | `'1'` |
| **valid_range** | Ignored (`0`) | AoE radius | Ignored (`0`) |
| **var1–var20** | ❌ ALL UNUSED | ❌ ALL UNUSED | ❌ ALL UNUSED |
| **Primary config** | State columns | State columns | State columns |

> There is no damage formula in these types. Do **not** fill `var1`–`var6` expecting a damage effect — the engine ignores them entirely. Set all vars to `0.0000`.

---

## The State Columns — Primary Configuration

All three types are fully configured through the state columns. See [States and Enhancements](10_Skill_States_and_Enhancements.md) for the full reference. The key columns are:

| Column | Role | Example |
|--------|------|---------|
| `state_id` | ID of the buff/debuff to apply | `12500` |
| `state_level_base` | Base level of the state | `0` |
| `state_level_per_skl` | State level scaling per skill level | `1.00` |
| `state_level_per_enhance` | State level scaling per enhance | `0.00` |
| `state_second` | Duration in engine ticks (seconds) | `30.00` |
| `state_second_per_level` | Duration increase per skill level | `2.00` |
| `state_second_per_enhance` | Duration increase per enhance | `0.00` |
| `probability_on_hit` | Chance (%) to apply the state | `100` (buff) or `<100` (debuff) |
| `probability_inc_by_slv` | Chance increase per skill level | `0` |

**Rule of thumb:**
- For **buffs** (`is_harmful = '0'`): `probability_on_hit = 100` — always succeeds
- For **debuffs** (`is_harmful = '1'`): `probability_on_hit < 100` — resisted by target stats

---

---

# EF_ADD_STATE — Type 301

## What This Skill Does

Applies a single state (`state_id`) to the selected target. No damage is dealt. This is the generic skill type for:
- Targeted **buffs** on allies (attack speed up, defense up, regeneration)
- Targeted **debuffs** on enemies (poison, slow, blind, silence)
- Any skill whose sole effect is applying a persistent status

## Mandatory Column Defaults for Type 301

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `301` | Required |
| `is_passive` | `'1'` | Active skill |
| `is_harmful` | `'0'` buff / `'1'` debuff | Drives faction filtering |
| `is_need_target` | `'1'` | A target is always required |
| `is_physical_act` | `'0'` | Magic-based application |
| `target` | `1` | `TARGET_TARGET` — single entity |
| `uf_enemy` | `'1'` | For debuffs only |
| `uf_self` / `uf_party` | `'1'` | For buffs on allies |
| `uf_enemy` + `uf_self` | Never both `'1'` | A skill cannot be both buff and debuff |
| `var1`–`var20` | `0.0000` | UNUSED — always zero |

## Example INSERT — "Hex" (ID: 91220)

Dark-element debuff skill: applies curse state (12505) to a single enemy for 20 + 2 per SLV seconds. 75% base chance.

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
    91220, 50091220, 0, 40091220, 1, '6',   -- elemental='6' = Dark
    '1', '0', '1', '1',                      -- active, not physical, harmful (debuff), needs target
    '0', '0', 0,
    '1', '1',
    20, 0,                                    -- cast_range=20, valid_range=0 (no AoE)
    0, 0,
    80, 8, 0,
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
    12.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',                                 -- uf_enemy=1 (debuff targets enemies only)
    '1', '1', '1',
    0,
    1, 301, 0,                                -- target=1 (single), effect_type=301
    -- State: curse debuff
    12505, 0, 1.00, 0.00,                     -- state_id=12505, level scales with SLV
    20.00, 2.00, 0.00,                        -- 20 seconds + 2 per SLV
    75, 2,                                    -- 75% base chance, +2% per SLV
    0, 0, 0,
    1.00,
    0, 0.00, 0.00,
    0, 0,
    -- ALL vars UNUSED for type 301
    0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000,
    1050, 'icon_skill_active_mage_hex', 0, 0.00, 0.00
);
```

---

---

# EF_ADD_REGION_STATE — Type 302

## What This Skill Does

Identical to type 301, but applies the state to **all valid entities within a circular region** centered on the selected target. Every entity hit by the AoE independently rolls against `probability_on_hit`.

**Use this type for**: AoE debuffs, group buff (allied targets), area silence, mass slow.

## AoE Targeting Columns (differs from 301)

| Column | Value | Reason |
|--------|-------|--------|
| `target` | `2` | `TARGET_REGION` — centered on selected target |
| `is_need_target` | `'1'` | Target entity anchors the AoE center |
| `valid_range` | e.g. `10` | Radius of the AoE in engine units |

## Key Differences from Type 301

- `effect_type` = `302` (not `301`)
- `target` = `2` instead of `1`
- `valid_range` defines the AoE radius — must be > 0
- State columns and vars are identical to 301

## Example INSERT — "Plague Cloud" (ID: 91221)

Earth-element AoE debuff: applies poison state (12506) to all enemies in radius 10. 60% chance per target.

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
    91221, 50091221, 0, 40091221, 1, '1',   -- elemental='1' = Earth
    '1', '0', '1', '1',
    '0', '0', 0,
    '1', '1',
    20, 10,                                   -- cast_range=20, valid_range=10 (AoE radius)
    0, 0,
    100, 10, 0,
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
    15.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',
    '1', '1', '1',
    0,
    2, 302, 0,                                -- target=2 (REGION w/ target), effect_type=302
    -- State: poison debuff
    12506, 0, 1.00, 0.00,
    15.00, 1.00, 0.00,
    60, 2,                                    -- 60% chance, +2% per SLV
    0, 0, 0,
    1.20,                                     -- AoE debuff generates moderate aggro
    0, 0.00, 0.00,
    0, 0,
    -- ALL vars UNUSED for type 302
    0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000,
    1060, 'icon_skill_active_mage_plague_cloud', 0, 0.00, 0.00
);
```

---

---

# EF_ADD_STATE_TO_CASTER_AND_TARGET — Type 311

## What This Skill Does

Applies the **same `state_id`** to both the **selected target** and the **caster** in a single cast. Both entities receive the same state level, duration, and probability roll. No damage is dealt.

**Use this type for**: blood pact skills (both parties share a buff/penalty), solidarity buffs, mirrored effects.

> The engine applies the state to the target first via the standard probability check, then unconditionally applies the same state to the caster. Verify source behavior if you need different probabilities per side.

## Key Differences from Type 301

- `effect_type` = `311` (not `301`)
- The caster is affected in addition to the target — no extra column configuration needed
- `uf_self` does **not** need to be set to `'1'` to affect the caster (the handler does this implicitly)
- `target`, `is_need_target`, and state columns work identically to 301
- All vars are UNUSED — identical to 301

## When Not to Use Type 311

- If caster and target need **different** states: use two separate skills or a combo skill instead
- If only the caster should be affected (self-buff): use `target = 41`, `uf_self = '1'`, `effect_type = 301`
- If only the target should be affected: use type 301

## Example INSERT — "Blood Oath" (ID: 91222)

No-element skill that applies a bond state (12510) to both caster and target for 60 seconds. The state is always applied (100% chance).

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
    91222, 50091222, 0, 40091222, 1, '0',   -- elemental='0' = None
    '1', '0', '0', '1',                      -- active, not physical, NOT harmful (buff), needs target
    '0', '0', 0,
    '1', '1',
    15, 0,                                    -- cast_range=15, valid_range=0
    0, 0,
    90, 10, 0,
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
    0.50, 0.00, 0.00,
    0.00,
    30.00, 0.00, 0.00,
    0,
    '0', '1', '1', '0',                       -- uf_party='1', uf_guild='1' (target allies)
    '0', '0',                                 -- uf_enemy=0 (buff skill)
    '1', '0', '0',                            -- tf_avatar=1 only (player-to-player bond)
    0,
    1, 311, 0,                                -- target=1 (single), effect_type=311
    -- State: bond state applied to BOTH caster and target
    12510, 0, 1.00, 0.00,
    60.00, 5.00, 0.00,                        -- 60 seconds + 5 per SLV
    100, 0,                                   -- 100% chance (guaranteed buff)
    0, 0, 0,
    1.00,
    0, 0.00, 0.00,
    0, 0,
    -- ALL vars UNUSED for type 311
    0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000,
    1070, 'icon_skill_active_support_blood_oath', 0, 0.00, 0.00
);
```

---

## Full Comparison: Types 301 / 302 / 311

| | 301 | 302 | 311 |
|---|---|---|---|
| **Applies to** | Target only | All in region | Target + caster |
| **target** | `1` | `2` | `1` |
| **is_need_target** | `'1'` | `'1'` | `'1'` |
| **valid_range** | `0` | AoE radius | `0` |
| **var1–var20** | UNUSED | UNUSED | UNUSED |
| **Damage dealt** | None | None | None |
| **state_id scope** | Target | Each entity in region | Target AND caster |
| **probability roll** | Per cast | Per entity in region | Per cast (target), then always on caster |

---

## Practical Constraints

- **Never set `var1`–`var20` to non-zero values** for these types. The handlers do not read them; non-zero values only create confusion and maintenance risk.
- **`is_harmful` drives faction targeting**: a buff skill (`is_harmful = '0'`) with `uf_enemy = '1'` is a logic error — the engine will still allow casting on enemies, potentially gifting them a buff.
- **`elemental` is mostly irrelevant** for pure state application (no damage to amplify), but set to `'0'` unless the state's lore is element-specific and you want the column to be self-documenting.
- **`hit_bonus` and `critical_bonus`** are meaningless for state-only skills — set all to `0`.
- **Type 311 and `uf_self`**: The caster is affected by the engine implicitly. You do not need `uf_self = '1'` to apply the state to the caster via type 311.

---

*Source: `Game/Skill/StructSkill.cpp` · `ADD_STATE()` · `ADD_REGION_STATE()` · `ADD_STATE_TO_CASTER_AND_TARGET()`*
