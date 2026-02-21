# Effect Type: EF_MAGIC_SINGLE_DAMAGE (231)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [Database Architecture](01_Skill_Database_Architecture.md) | [Skill Identifiers](06_Skill_Identifiers_and_Core_Flags.md)

---

## 1. Overview

`EF_MAGIC_SINGLE_DAMAGE` is the standard **single-target magical damage** effect used for offensive spells. It calculates damage based on the caster's Magic Attack (`M.Atk`) plus optional flat bonuses, and can optionally apply a debuff state on hit.

**Characteristics:**
- Single-target hit
- Scales with caster's M.Atk (modified by the `elemental` column)
- Supports both percentage-based and flat damage components
- Optionally applies a state (debuff) on hit, controlled by `state_id` and `probability_on_hit`
- Supports on-hit criticals via `critical_bonus` / `critical_bonus_per_skl`

---

## 2. C++ Source Analysis

**Source File**: `Game/Skill/StructSkill.cpp`
**Handler Function**: `StructSkill::SINGLE_MAGICAL_DAMAGE( struct StructCreature* pTarget )`

```cpp
void StructSkill::SINGLE_MAGICAL_DAMAGE( struct StructCreature* pTarget )
{
    if( !pTarget ) return;

    DamageDataType nDamage = m_pOwner->GetMagicPoint(
        (Elemental::Type)GetSkillBase()->GetElementalType(),
        GetSkillBase()->IsPhysicalSkill(),
        GetSkillBase()->IsHarmful()
    );

    nDamage *= GetVar(0) + GetVar(1) * GetRequestedSkillLevel() + GetVar(2) * GetEnhance();
    nDamage += GetVar(3) + GetVar(4) * GetRequestedSkillLevel() + GetVar(5) * GetEnhance();

    int elemental_type = GetSkillBase()->GetElementalType();

    StructCreature::_DAMAGE_INFO Damage = pTarget->DealMagicalSkillDamage(
        m_pOwner, nDamage, (Elemental::Type)elemental_type,
        GetHitBonus( GetEnhance(), m_pOwner->GetLevel() - pTarget->GetLevel() ),
        GetCriticalBonus( GetRequestedSkillLevel() )
    );

    AddSkillDamageResult( &m_vResultList, SkillResult::MAGIC_DAMAGE, elemental_type, Damage, pTarget->GetHandle() );
}
```

**Damage Formula (derived from source)**:
```
nDamage = M.Atk * (var1 + var2 * SkillLevel + var3 * Enhance)
        + (var4  + var5 * SkillLevel + var6 * Enhance)
```

**Key source-level observations:**
- `GetVar(0)` = `var1` (C++ uses 0-based index, SQL uses 1-based)
- `M.Atk` is fetched via `GetMagicPoint()` which applies elemental modifiers from the `elemental` column
- `GetHitBonus()` reads from `hit_bonus`, `hit_bonus_per_enhance`, `percentage` columns
- `GetCriticalBonus()` reads from `critical_bonus`, `critical_bonus_per_skl` columns
- State on hit is NOT in `SINGLE_MAGICAL_DAMAGE()` — it is applied by the generic skill result processor reading `state_id`, `probability_on_hit`, `state_level_base`, `state_second`, etc.

---

## 3. Variable Mapping (`var1`–`var20`)

| Variable | C++ Ref | Type | Role | Example Value |
|----------|---------|------|------|---|
| **`var1`** | `GetVar(0)` | `float` | **M.Atk multiplier — base** (1.0 = 100% M.Atk) | `1.7500` |
| **`var2`** | `GetVar(1)` | `float` | **M.Atk multiplier — per skill level** | `0.1000` |
| **`var3`** | `GetVar(2)` | `float` | **M.Atk multiplier — per enhance card level** | `0.1000` |
| **`var4`** | `GetVar(3)` | `float` | **Flat damage bonus — base** | `170.0000` |
| **`var5`** | `GetVar(4)` | `float` | **Flat damage bonus — per skill level** | `26.0000` |
| **`var6`** | `GetVar(5)` | `float` | **Flat damage bonus — per enhance card level** | `70.0000` |
| `var7–var20` | — | — | **UNUSED** — always set to `0.0000` | `0.0000` |

### Variable Semantics

**Percentage block (`var1`, `var2`, `var3`)**:
- `var1` is the base multiplier on M.Atk. Setting `var1 = 1.0` deals 100% of M.Atk. `var1 = 2.0` deals 200%.
- `var2` adds to the multiplier per skill level. At SLV 5 with `var2 = 0.10`, the multiplier is `var1 + 5*0.10`.
- `var3` adds per enhance card. Useful for cards that increase damage.

**Flat block (`var4`, `var5`, `var6`)**:
- `var4` is a flat damage constant added after the M.Atk multiplication.
- `var5` adds to this flat bonus per skill level.
- `var6` adds per enhance card.
- These are in raw damage units, **not** fractions of M.Atk. Use `0.0` if you want purely percentage-based damage.

> [!IMPORTANT]
> Unlike `EF_PARAMETER_AMP` variables, these floats do NOT use the `1 = 100%` convention for flat damage (`var4`–`var6`). The multiplier group (`var1`–`var3`) does use it. See [AGENTS.md - Formula Math](AGENTS.md#4-the-mathematics-of-variables-var1-to-var18) for comparison.

---

## 4. Elemental Type

The `elemental` column directly controls which element is used for damage calculation:

| Value | Element |
|-------|---------|
| `'0'` | None (neutral) |
| `'1'` | Earth |
| `'2'` | Fire |
| `'3'` | Water |
| `'4'` | Wind |
| `'5'` | Light |
| `'6'` | Dark |

For full elemental details, see [Elementals Guide](05_Elementals_Guide.md).

---

## 5. On-Hit State (Debuff/Status Effect)

`EF_MAGIC_SINGLE_DAMAGE` supports applying a **state on hit** via the standard state columns. This is processed **outside** the handler function, by the generic skill result system.

| Column | Role | Example |
|--------|------|---------|
| `state_id` | ID of the state/debuff to apply | `13037` |
| `state_level_base` | Base level of the state | `0` |
| `state_level_per_skl` | Level of state increases per skill level | `1.00` |
| `state_level_per_enhance` | Level increase per enhance | `0.00` |
| `state_second` | Duration of the state (in engine ticks) | `6.00` |
| `state_second_per_level` | Duration increase per skill level | `0.00` |
| `state_second_per_enhance` | Duration increase per enhance | `0.00` |
| `probability_on_hit` | Chance in % to apply the state (base) | `100` |
| `probability_inc_by_slv` | Chance increase per skill level | `0` |

For full documentation of these columns, see [States and Enhancements](10_Skill_States_and_Enhancements.md).

> Set `state_id = 0` and `probability_on_hit = 0` for a pure damage spell with no state effect.

---

## 6. Hit Accuracy and Critical Bonus

These columns inject directly into the damage formula via `GetHitBonus()` and `GetCriticalBonus()`:

```cpp
// From SkillBase.h:
int GetHitBonus(int enhance, int level_diff) const {
    return hit_bonus + enhance * hit_bonus_per_enhance + level_diff * percentage;
}
int GetCriticalBonus(int skill_lv) const {
    return critical_bonus + critical_bonus_per_skl * skill_lv;
}
```

| Column | Role |
|--------|------|
| `hit_bonus` | Flat accuracy bonus added to hit calculation |
| `hit_bonus_per_enhance` | Accuracy per enhance card level |
| `percentage` | Accuracy scaled by level difference (caster - target) |
| `critical_bonus` | Flat critical rate bonus (e.g., `26995` = max cap) |
| `critical_bonus_per_skl` | Critical rate bonus per skill level |

For the full column reference, see [Combat Modifiers and Variables](11_Skill_Combat_Modifiers_and_Variables.md).

---

## 7. Mandatory Column Defaults for `effect_type = 231`

| Column | Value | Reason |
|--------|-------|--------|
| `effect_type` | `231` | Required — activates `SINGLE_MAGICAL_DAMAGE` handler |
| `is_passive` | `'1'` | Active skill — not a passive |
| `is_harmful` | `'1'` | Damages the target |
| `is_need_target` | `'1'` | Requires a hostile target |
| `is_physical_act` | `'0'` | Magic damage — not physical |
| `is_corpse` | `'0'` | Cannot target corpses |
| `is_toggle` | `'0'` | Not a toggle skill |
| `uf_enemy` | `'1'` | Only targets enemies |
| `uf_self` / `uf_party` / etc. | `'0'` | Not friendly |
| `target` | `1` | `TARGET_TARGET` — single target |
| `vf_is_not_need_weapon` | `'1'` | Magic casts ignore weapon type (adjust if needed) |
| `var7`–`var20` | `0.0000` | Unused — always zero |

---

## 8. Practical Constraints and Notes

- **`var1` must never be `0.0`** unless you intend pure flat damage. A zero multiplier means all M.Atk is ignored.
- **`elemental` is critical**: The element here determines both which elemental resistance the target uses and which elemental attack bonus the caster gains. `'0'` (None) bypasses all elemental interactions.
- **`cost_mp_per_enhance`** accepts **negative values** — this allows enhance cards to *reduce* the mana cost of the skill (as seen in the reverse-engineered example).
- **`is_projectile`** can be set to `1` for projectile-based spells (fireballs, arrows of light, etc.), with `projectile_speed` and `projectile_acceleration` controlling the trajectory. Hit is calculated on impact.
- **`hate_mod`** (aggro multiplier) should be set above `1.00` for high-damage spells to increase their aggro.

---

## 9. Reverse-Engineered Example

**Skill ID**: `90082` | **Elemental**: Fire | **State**: Applies debuff `13037`

```
Skill Damage at SLV 10, Enhance 0, with M.Atk = 1000:
  nDamage = 1000 * (1.75 + 0.10*10 + 0.10*0)  +  (170 + 26*10 + 70*0)
           = 1000 * 2.75                         +  430
           = 2750 + 430 = 3180 raw damage

With Enhance 5:
  nDamage = 1000 * (1.75 + 0.10*10 + 0.10*5)  +  (170 + 26*10 + 70*5)
           = 1000 * 3.25                         +  780
           = 3250 + 780 = 4030 raw damage
```

---

## 10. Example INSERT — AI-Generated Skill

**Skill Name**: "Frost Shard" — A Water-element single-target spell that slows the target.

- **ID**: `91200`
- **Element**: Water (`'3'`)
- **Damage**: 150% M.Atk base + 8% per level, 100 flat + 20 per level
- **State**: Applies slow debuff (state_id = 9999) for 6 ticks, 70% chance
- **Cost**: 120 MP base, +12 per level
- **Cooldown**: 15 seconds
- **Cast Time**: 0.8 seconds

```sql
-- Frost Shard (ID: 91200)
-- EF_MAGIC_SINGLE_DAMAGE (231) — Water element, applies slow on hit
-- Damage: M.Atk * (1.50 + 0.08*SLV) + (100 + 20*SLV)
-- State: state_id=9999, 70% chance, 6 tick duration

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
    -- Identity
    91200, 50091200, 0, 40091200,
    -- Core flags
    1, '3',          -- is_valid=1, elemental=Water
    '1', '0', '1', '1',  -- active, not physical, harmful, needs target
    '0', '0', 0,     -- no corpse, no toggle, toggle_group=0
    -- Casting
    '1', '1',        -- casting_type=1 (interruptible), casting_level=1
    20, 20,          -- cast_range=20, valid_range=20
    -- Costs
    0, 0,            -- cost_hp, cost_hp_per_skl
    120, 12, -10,    -- cost_mp=120, per_skl=+12, per_enhance=-10 (enhance reduces cost)
    0.00, 0.00, 0.00, 0.00,  -- hp/mp percent costs
    0, 0, 0.00, 0.00,        -- havoc, energy costs
    0, 0, 0, 0,              -- exp, jp costs
    0, 0, 0,                 -- item costs
    -- Requirements
    0, 0, 0, 0, 0, 0, 0,     -- need_level, hp, mp, havoc, havoc_burst, state_id, state_level
    0,                        -- need_state_exhaust
    -- Weapon flags (magic — no weapon required)
    '0', '0', '0', '0', '0',
    '0', '0', '0', '0',
    '0', '0', '0', '0',
    '0', '0', '0', '0',
    '1',                      -- vf_is_not_need_weapon = 1
    -- Timing
    0.80, 0.00, 0.00,         -- delay_cast=0.8s, no per_skl/enhance
    0.00,                     -- delay_common (GCD) = 0
    15.00, 0.00, 0.00,        -- cooltime=15s, no scaling
    0,                        -- cool_time_group_id
    -- Faction flags
    '0', '0', '0', '0',       -- uf_self/party/guild/neutral = 0
    '0', '1',                 -- uf_purple=0, uf_enemy=1
    '1', '1', '1',            -- tf_avatar, tf_summon, tf_monster = 1
    -- Skill config
    0,                        -- skill_lvup_limit
    1, 231, 0,                -- target=1 (single), effect_type=231, enchant_link=0
    -- State (slow debuff on hit)
    9999, 0, 1.00, 0.00,      -- state_id=9999, base_lv=0, +1 per SLV, no enhance
    6.00, 0.00, 0.00,         -- 6 ticks duration, no scaling
    70, 0,                    -- 70% chance, no per-SLV increase
    -- Accuracy / Critical
    0, 0, 0,                  -- hit_bonus, per_enhance, percentage
    1.20,                     -- hate_mod = 1.2x aggro
    0, 0.00, 0.00,            -- hate_basic, per_skl, per_enhance
    0, 0,                     -- critical_bonus, per_skl (no crit bonus)
    -- Variables
    1.5000, 0.0800, 0.0000,   -- var1=150% M.Atk base, var2=+8% per SLV, var3=0 (no enhance scaling)
    100.0000, 20.0000, 0.0000, -- var4=100 flat, var5=+20 per SLV, var6=0
    0.0000, 0.0000, 0.0000, 0.0000,  -- var7-10: UNUSED
    0.0000, 0.0000, 0.0000, 0.0000,  -- var11-14: UNUSED
    0.0000, 0.0000, 0.0000, 0.0000,  -- var15-18: UNUSED
    0.0000,                          -- var19: UNUSED
    0.0000,                          -- var20: UNUSED
    -- Icon
    1100, 'icon_skill_active_mage_frost_shard', 0, 0.00, 0.00
);
```

---

## 11. Related Effect Types

| Effect Type | ID | Difference from EF_MAGIC_SINGLE_DAMAGE |
|---|---|---|
| `EF_MAGIC_SINGLE_DAMAGE` | `231` | Standard version (this document) |
| `EF_MAGIC_SINGLE_DAMAGE_ADD_RANDOM_STATE` | `239` | Same handler + applies a random state |
| `EF_MAGIC_SINGLE_DAMAGE_OR_DEATH` | `234` | Same handler + instant kill chance |
| `EF_MAGIC_SINGLE_DAMAGE_BY_CONSUMING_TARGETS_STATE` | `240` | Requires and consumes target state |
| `EF_MAGIC_SINGLE_DAMAGE_WITH_PHYSICAL_DAMAGE` | `32021` | Same handler + adds physical damage component (`var7`–`var11` active) |
| `EF_MAGIC_SINGLE_DAMAGE_T1_OLD` | `201` | Legacy — migrate to `231` |
| `EF_MAGIC_SINGLE_DAMAGE_T2_OLD` | `203` | Legacy — migrate to `231` |

> The old T1/T2 types (`201`, `203`) are marked for replacement in the source:
> `// migrate to 231`

---

*Source: `Game/Skill/StructSkill.cpp` · `StructSkill::SINGLE_MAGICAL_DAMAGE()` · `Game/Resource/SkillBase.h`*
