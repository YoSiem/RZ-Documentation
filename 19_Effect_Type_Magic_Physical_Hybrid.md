# Effect Type: EF_MAGIC_SINGLE_DAMAGE_WITH_PHYSICAL_DAMAGE (32021)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [EF_MAGIC_SINGLE_DAMAGE (231) — Base Reference](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md) | [Magic Damage Variants (232 / 235)](13_Effect_Type_Magic_Damage_Variants.md)

`EF_MAGIC_SINGLE_DAMAGE_WITH_PHYSICAL_DAMAGE` is a single-target skill that deals **two independent damage hits** in one cast: a **magical damage hit** (using the caster's M.Atk) and a **physical damage hit** (using the caster's P.Atk). Each hit is calculated separately and checks the target's corresponding defense (M.Def for the magic hit, P.Def for the physical hit).

This type is listed in the [Related Effect Types](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md#11-related-effect-types) section of the base 231 document.

**Source Reference**: `Game/Skill/StructSkill.cpp` → `SINGLE_MAGICAL_DAMAGE_WITH_PHYSICAL()`

---

## ⚠️ Generation Warning — var Split

| Vars | Component | Formula stat |
|------|-----------|-------------|
| `var1`–`var6` | **Magic damage** | M.Atk (same as type 231) |
| `var7`–`var12` | **Physical damage** | P.Atk |
| `var13`–`var20` | UNUSED | — |

Mixing vars between components produces either broken magic damage or zero physical damage. Always populate both blocks independently.

---

## Magic Damage Formula (var1–var6)

Identical to [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md#3-variable-mapping-var1var20):

```
nMagicDmg = M.Atk × (var1 + var2×SLV + var3×Enhance)
           + (var4  + var5×SLV + var6×Enhance)
```

| Variable | Role |
|----------|------|
| `var1` | M.Atk multiplier — base |
| `var2` | M.Atk multiplier — per skill level |
| `var3` | M.Atk multiplier — per enhance |
| `var4` | Flat magic damage — base |
| `var5` | Flat magic damage — per skill level |
| `var6` | Flat magic damage — per enhance |

---

## Physical Damage Formula (var7–var12)

Mirrors the magic formula structure but drives `DealPhysicalSkillDamage()`:

```
nPhysDmg = P.Atk × (var7 + var8×SLV + var9×Enhance)
          + (var10 + var11×SLV + var12×Enhance)
```

| Variable | Role |
|----------|------|
| `var7` | P.Atk multiplier — base |
| `var8` | P.Atk multiplier — per skill level |
| `var9` | P.Atk multiplier — per enhance |
| `var10` | Flat physical damage — base |
| `var11` | Flat physical damage — per skill level |
| `var12` | Flat physical damage — per enhance |
| `var13`–`var20` | UNUSED — always `0.0000` |

> The physical hit uses **P.Def** on the target's side. It does **not** use the `elemental` column — elemental modifiers apply only to the magic component.

---

## Elemental Column

The `elemental` column controls the magic component only. The physical component is always non-elemental. See [Elementals Guide](05_Elementals_Guide.md) for values.

---

## On-Hit State

`state_id` and `probability_on_hit` work the same as in type 231. The state fires once per cast (not once per hit component) — it is not applied twice.

---

## Mandatory Column Defaults for Type 32021

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `32021` | Required |
| `is_passive` | `'1'` | Active skill |
| `is_harmful` | `'1'` | Damages the target |
| `is_need_target` | `'1'` | Single-target — requires a target |
| `is_physical_act` | `'0'` | Casting is magical; physical component is engine-handled |
| `target` | `1` | `TARGET_TARGET` — single target |
| `uf_enemy` | `'1'` | Enemies only |
| `vf_is_not_need_weapon` | `'1'` | Magic cast — no weapon required (adjust if restricted) |
| `var13`–`var20` | `0.0000` | UNUSED |

---

## Example INSERT — "Runic Strike" (ID: 91260)

Earth-element skill: deals M.Atk × 1.1 + 80 flat magic damage AND P.Atk × 0.6 + 50 flat physical damage in one cast.

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
    91260, 50091260, 0, 40091260, 1, '1',   -- elemental='1' = Earth (magic component)
    '1', '0', '1', '1',
    '0', '0', 0,
    '1', '1',
    20, 0,
    0, 0,
    140, 15, -10,
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
    14.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',
    '1', '1', '1',
    0,
    1, 32021, 0,
    -- No on-hit state
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.20,
    0, 0.00, 0.00,
    0, 0,
    -- Magic component (var1–var6)
    1.1000, 0.0700, 0.0000,    -- M.Atk × (1.10 + 0.07/SLV)
    80.0000, 10.0000, 0.0000,  -- + (80 + 10/SLV) flat magic
    -- Physical component (var7–var12)
    0.6000, 0.0400, 0.0000,    -- P.Atk × (0.60 + 0.04/SLV)
    50.0000, 8.0000, 0.0000,   -- + (50 + 8/SLV) flat physical
    -- UNUSED (var13–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    1750, 'icon_skill_active_mage_runic_strike', 0, 0.00, 0.00
);
```

---

## Full Comparison: Types 231 / 32021

| | 231 | 32021 |
|---|---|---|
| **Damage hit(s)** | 1× magic | 1× magic + 1× physical |
| **var1–var6** | Magic formula | Magic formula (identical) |
| **var7–var12** | UNUSED | Physical formula (P.Atk-based) |
| **var13–var20** | UNUSED | UNUSED |
| **elemental** | Applies to magic hit | Applies to magic hit only |
| **state_id** | ✅ Active (one proc) | ✅ Active (one proc, not doubled) |
| **P.Def interaction** | None (magic bypasses P.Def) | Physical hit checks target P.Def |

---

## Practical Constraints

- **Both var blocks are independent**: scaling `var1` does not affect the physical output, and scaling `var7` does not affect the magic output. Balance each component separately.
- **`var7` must not be `0.0`** if you intend a physical component. A zero P.Atk multiplier produces a zero physical hit unless `var10`+ provide flat physical damage.
- **Physical hit is not elemental**: do not expect the `elemental` column to reduce enemy physical resistance. Only M.Def and P.Def reduction mechanics apply to their respective hits.
- **On-hit state fires once**: `state_id` is applied once per cast, regardless of the two damage components. It is not rolled twice.

---

*Source: `Game/Skill/StructSkill.cpp` · `SINGLE_MAGICAL_DAMAGE_WITH_PHYSICAL()`*
