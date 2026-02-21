# Effect Type: EF_WEAPON_MASTERY (10001)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [EF_PARAMETER_AMP (4) — Bitmask Reference](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md) | [Skill Costs and Requirements](08_Skill_Costs_and_Requirements.md)

`EF_WEAPON_MASTERY` is a **passive** effect that amplifies combat stats (typically P.Atk, critical rate, or attack speed) **conditionally, only while a specific weapon type is equipped**. It uses the same 6-slot bitmask variable architecture as [EF_PARAMETER_AMP (4)](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md), but the `vf_` weapon flags gate whether the bonus is active.

**Source Reference**: `Game/Skill/StructSkill.cpp` → `WEAPON_MASTERY()`

---

## How It Differs from EF_PARAMETER_AMP (4)

| | EF_PARAMETER_AMP (4) | EF_WEAPON_MASTERY (10001) |
|---|---|---|
| **Active condition** | Always active (passive) | Only when the designated weapon is equipped |
| **`vf_is_not_need_weapon`** | `'1'` (no weapon needed) | `'0'` (weapon **required**) |
| **Weapon vf_ flag** | All `'0'` | Target weapon type = `'1'` |
| **Var architecture** | 6-slot bitmask (var1–var18) | Same 6-slot bitmask |
| **`effect_type`** | `4` | `10001` |

---

## Variable Architecture (Inherited from EF_PARAMETER_AMP)

`EF_WEAPON_MASTERY` uses the **identical 6-slot bitmask layout** as EF_PARAMETER_AMP. See [04_Creating_Passive_Skill_EF_PARAMETER_AMP.md](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md) for the full bitmask tables. Summary:

Each slot = `(bitmask, base_value, per_SLV_multiplier)`:

| Slot | Vars | Bitmask table | Typical use for weapon mastery |
|------|------|---------------|-------------------------------|
| 1 | var1, var2, var3 | Standard (`128` = P.Atk, `2048` = Atk Speed, `65536` = Crit Rate) | Primary stat boost |
| 2 | var4, var5, var6 | Standard | Secondary stat boost |
| 3 | var7, var8, var9 | Extended (elemental/penetration) | Weapon penetration bonus |
| 4 | var10, var11, var12 | Extended | Elemental resist or P.Def Ignore |
| 5 | var13, var14, var15 | Standard | Tertiary stat boost |
| 6 | var16, var17, var18 | Standard | Quaternary stat boost |
| — | var19, var20 | — | **ALWAYS `0.0000`** |

**Math convention**: `1 = 100%` throughout. `0.0500 = 5%` per SLV. See [AGENTS.md — Variable Math](AGENTS.md#4-the-mathematics-of-variables-var1-to-var18) for the full explanation.

---

## Weapon Restriction via vf_ Flags

The `vf_` columns gate which weapon activates this mastery. **Set exactly one group to `'1'`** and set `vf_is_not_need_weapon = '0'`.

| vf_ Column | Weapon Type |
|------------|-------------|
| `vf_one_hand_sword` | One-handed sword |
| `vf_two_hand_sword` | Two-handed sword |
| `vf_double_sword` | Dual swords |
| `vf_dagger` | Dagger |
| `vf_double_dagger` | Dual daggers |
| `vf_spear` | Spear |
| `vf_axe` | Two-handed axe |
| `vf_one_hand_axe` | One-handed axe |
| `vf_double_axe` | Dual axes |
| `vf_one_hand_mace` | One-handed mace |
| `vf_two_hand_mace` | Two-handed mace |
| `vf_lightbow` | Light bow |
| `vf_heavybow` | Heavy bow |
| `vf_crossbow` | Crossbow |
| `vf_one_hand_staff` | One-handed staff |
| `vf_two_hand_staff` | Two-handed staff |
| `vf_shield_only` | Shield (off-hand) |

> To apply mastery to multiple weapon types in the same category (e.g., both one-hand and two-hand sword), set both corresponding flags to `'1'`. For full weapon class masteries covering all subtypes, set all relevant vf_ flags.

---

## Mandatory Column Defaults for Type 10001

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `10001` | Required |
| `is_passive` | `'0'` | Passive skill (`0 = !is_active` in engine) |
| `is_harmful` | `'0'` | Self-buff |
| `is_physical_act` | `'0'` | No physical trigger |
| `is_need_target` | `'0'` | No target needed |
| `is_toggle` | `'0'` | Always-on passive, not a toggle |
| `elemental` | `'0'` | Passives generally have no element |
| All cost fields | `0` / `0.00` | Passive — no cast cost |
| `cast_range`, `valid_range` | `0` | Not applicable |
| `target` | `41` | Standard passive self-loop target |
| `delay_common` | `1.50` | Standard passive loop interval |
| All other delays | `0.00` | |
| `vf_is_not_need_weapon` | `'0'` | **Required** — weapon IS needed |
| Designated weapon vf_ | `'1'` | **Required** — the mastered weapon type |
| All other vf_ | `'0'` | |
| `var19`, `var20` | `0.0000` | Always zero |

---

## Example INSERT — "Two-Handed Sword Mastery" (ID: 91240)

Passive that grants +10% P.Atk base (+5%/SLV) and +3% Critical Rate (+1%/SLV) when a two-handed sword is equipped.

- Slot 1 (var1–var3): `P.Atk` bitmask `128`, base `0.10`, +`0.05`/SLV
- Slot 2 (var4–var6): `Critical Rate` bitmask `65536`, base `0.03`, +`0.01`/SLV
- Slots 3–6: unused

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
    91240, 50091240, 0, 40091240, 1, '0',
    '0', '0', '0', '0',       -- passive, not physical, not harmful, no target
    '0', '0', 0, '0',
    '0', 0, 0,
    0, 0, 0, 0, 0,
    0.00, 0.00, 0.00, 0.00,
    0, 0, 0.00, 0.00,
    0, 0, 0, 0,
    0, 0, 0,
    0, 0, 0, 0, 0, 0, 0,
    0,
    -- Weapon flags: two-handed sword only
    '0', '1', '0', '0', '0',  -- vf_two_hand_sword = '1'
    '0', '0', '0', '0',
    '0', '0', '0', '0',
    '0', '0', '0', '0',
    '0',                       -- vf_is_not_need_weapon = '0' (weapon IS required)
    0.00, 0.00, 0.00,
    1.50,                      -- delay_common = 1.50 (standard passive loop)
    0.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '0',
    '0', '0', '0',
    0,
    41, 10001, 0,              -- target=41, effect_type=10001
    -- No state effect
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.00,
    0, 0.00, 0.00,
    0, 0,
    -- Slot 1: P.Atk boost (bitmask 128, 10% base, +5%/SLV)
    128.0000, 0.1000, 0.0500,
    -- Slot 2: Critical Rate boost (bitmask 65536, 3% base, +1%/SLV)
    65536.0000, 0.0300, 0.0100,
    -- Slots 3–6: UNUSED
    0.0000, 0.0000, 0.0000,    -- Slot 3 (extended)
    0.0000, 0.0000, 0.0000,    -- Slot 4 (extended)
    0.0000, 0.0000, 0.0000,    -- Slot 5
    0.0000, 0.0000, 0.0000,    -- Slot 6
    0.0000, 0.0000,            -- var19, var20: always 0
    1600, 'icon_skill_pas_warrior_sword_mastery', 0, 0.00, 0.00
);
```

---

## Practical Constraints

- **`vf_is_not_need_weapon` must be `'0'`** — if set to `'1'`, the weapon restriction is bypassed and the mastery becomes an unconditional passive (equivalent to EF_PARAMETER_AMP). Use EF_PARAMETER_AMP for unconditional stat passives.
- **Exactly one or more `vf_` flags must be `'1'`** — leaving all vf_ flags at `'0'` with `vf_is_not_need_weapon = '0'` will produce a passive that is never active (no weapon can satisfy the condition).
- **Bitmask math is identical to EF_PARAMETER_AMP** — including the `1 = 100%` convention and the separation between standard (`var1`, `var4`, `var13`, `var16`) and extended (`var7`, `var10`) bitmask slots. Always cross-reference [04_Creating_Passive_Skill_EF_PARAMETER_AMP.md](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md) for bitmask values.
- **`var19` and `var20` must always be `0.0000`** — they are never read by this handler.

---

*Source: `Game/Skill/StructSkill.cpp` · `WEAPON_MASTERY()` · Bitmask tables: `SkillBase.h`*
