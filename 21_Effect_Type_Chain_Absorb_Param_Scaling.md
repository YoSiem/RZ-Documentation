# Effect Types: EF_MAGIC_CHAIN_DAMAGE (32062), EF_ABSORB_DAMAGE (32211) & EF_INC_PARAM_BASED_PARAM (32301)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md) | [EF_PARAMETER_AMP (4)](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md)

Three distinct effect types grouped for file efficiency. They share no common mechanic but are each unique enough to warrant focused documentation.

**Source Reference**: `Game/Skill/StructSkill.cpp`
- 32062 → `MAGIC_CHAIN_DAMAGE()`
- 32211 → `ABSORB_DAMAGE()`
- 32301 → `INC_PARAM_BASED_PARAM()`

---

---

# EF_MAGIC_CHAIN_DAMAGE — Type 32062

## What This Skill Does

Deals magic damage to a primary target, then **chains the hit to additional nearby enemies** in sequence. Each successive chain target receives reduced damage based on a decay multiplier. The chain does not hit the same target twice. Total chain length, decay per jump, and jump radius are all configured via `var7`–`var9`.

**Use this type for**: chain lightning, arc spells, bouncing fire, cascading elemental attacks.

## Damage Formula

The primary target receives the full 231-base magic formula:

```
nDamage_primary = M.Atk × (var1 + var2×SLV + var3×Enhance)
                + (var4  + var5×SLV + var6×Enhance)
```

Each subsequent chain hit receives:

```
nDamage_chain[n] = nDamage_chain[n-1] × var8
```

Where `n = 1` is the first bounce target (receives `nDamage_primary × var8`), `n = 2` receives that times `var8` again, and so on.

## Variables Unique to Type 32062 (var7–var9)

| Variable | Type | Role | Notes |
|----------|------|------|-------|
| **`var7`** | integer (stored as float) | **Total chain targets** (including primary) | `4.0000` = hit primary + 3 bounces. Must be ≥ 1. |
| **`var8`** | float | **Damage decay multiplier per bounce** | `0.7000` = each bounce deals 70% of the previous hit. `1.0000` = no decay. |
| **`var9`** | float | **Jump radius** (engine units) | Maximum distance the chain can jump to the next target. `8.0` = bounces to enemies within 8 units of the current target. |
| `var10`–`var20` | — | **UNUSED** | Always `0.0000`. |

**Chain damage at each step** (example: primary = 1000 damage, `var8 = 0.7`, 4 targets total):
```
Target 1 (primary): 1000
Target 2 (1st bounce): 700
Target 3 (2nd bounce): 490
Target 4 (3rd bounce): 343
```

## On-Hit State with Chain

`state_id` is applied **per-chain-hit** — each target the chain reaches independently rolls against `probability_on_hit`. This allows chain spells to spread debuffs naturally.

## Mandatory Column Defaults for Type 32062

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `32062` | Required |
| `is_passive` | `'1'` | Active skill |
| `is_harmful` | `'1'` | Damages targets |
| `is_need_target` | `'1'` | Primary target must be selected |
| `target` | `1` | `TARGET_TARGET` — primary is single target; chain is engine-handled |
| `valid_range` | `0` | Chain jump range is in `var9`, not here |
| `uf_enemy` | `'1'` | Enemy faction |
| `var7` | `≥ 1.0000` | Required — at minimum 1 target (primary only) |

## Example INSERT — "Chain Lightning" (ID: 91263)

Wind-element chain spell: hits primary target with 100% M.Atk + 60 flat, then bounces to 2 additional nearby enemies within radius 8, each dealing 70% of the previous hit. No on-hit state.

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
    91263, 50091263, 0, 40091263, 1, '4',   -- elemental='4' = Wind
    '1', '0', '1', '1',
    '0', '0', 0,
    '1', '1',
    20, 0,                                    -- valid_range=0; chain range is var9
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
    1, 32062, 0,
    -- No on-hit state
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.10,
    0, 0.00, 0.00,
    0, 0,
    -- Base damage (var1–var6)
    1.0000, 0.0600, 0.0000,
    60.0000, 8.0000, 0.0000,
    -- Chain parameters (var7–var9)
    3.0000,                    -- var7: 3 total targets (primary + 2 bounces)
    0.7000,                    -- var8: 70% damage decay per bounce
    8.0000,                    -- var9: 8-unit jump radius
    -- UNUSED (var10–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000,
    1900, 'icon_skill_active_mage_chain_lightning', 0, 0.00, 0.00
);
```

---

---

# EF_ABSORB_DAMAGE — Type 32211

## What This Skill Does

Applies a **damage absorption shield** to the target (self or ally). The shield absorbs incoming damage up to a calculated maximum. Once the shield's capacity is exhausted, subsequent damage hits the target normally. The shield amount scales with the caster's M.Atk.

**Use this type for**: magic shields, absorption barriers, pre-emptive damage sponges.

**Source Reference**: `Game/Skill/StructSkill.cpp` → `ABSORB_DAMAGE()`

## Shield Amount Formula (var1–var6)

```
nShield = M.Atk × (var1 + var2×SLV + var3×Enhance)
        + (var4  + var5×SLV + var6×Enhance)
```

The formula follows the same structure as the damage formula in [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md#3-variable-mapping-var1var20), but the result is the **maximum damage absorbed** rather than damage dealt.

| Variable | Role |
|----------|------|
| `var1` | M.Atk absorb multiplier — base |
| `var2` | M.Atk absorb multiplier — per skill level |
| `var3` | M.Atk absorb multiplier — per enhance |
| `var4` | Flat absorb capacity — base |
| `var5` | Flat absorb capacity — per skill level |
| `var6` | Flat absorb capacity — per enhance |
| `var7`–`var20` | **UNUSED** — always `0.0000` |

## State Column Interaction

The `state_id` column can be used to apply an absorb-related state (such as a visual shield indicator buff). Set `state_id = 0` for a pure absorption effect with no secondary state.

## Mandatory Column Defaults for Type 32211

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `32211` | Required |
| `is_passive` | `'1'` | Active skill |
| `is_harmful` | `'0'` | Defensive/beneficial |
| `is_need_target` | `'1'` | Target to apply the shield to |
| `target` | `1` | Single target |
| `uf_self` | `'1'` | Can shield yourself |
| `uf_party` | `'1'` | Can shield party members |
| `uf_enemy` | `'0'` | Cannot shield enemies |
| `tf_monster` | `'0'` | Cannot shield monsters |
| `hate_mod` | `≥ 1.00` | Protective spells generate some aggro |
| `var7`–`var20` | `0.0000` | UNUSED |

## Example INSERT — "Mana Shield" (ID: 91264)

Light-element shield on self or ally: absorbs `M.Atk × 2.0 + 300` flat damage. Applies a visual shield state (12540) on application.

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
    91264, 50091264, 0, 40091264, 1, '5',   -- elemental='5' = Light
    '1', '0', '0', '1',                      -- active, not physical, NOT harmful, needs target
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
    10.00, 0.00, 0.00,
    0,
    '1', '1', '1', '0',                       -- uf_self, party, guild
    '0', '0',
    '1', '1', '0',                            -- tf_avatar, tf_summon; NOT monster
    0,
    1, 32211, 0,
    -- Visual shield state (optional)
    12540, 0, 1.00, 0.00,
    10.00, 1.00, 0.00,
    100, 0,
    0, 0, 0,
    1.20,                                     -- healer/shielder still generates aggro
    0, 0.00, 0.00,
    0, 0,
    -- Shield formula
    2.0000, 0.1500, 0.0000,   -- var1: 200% M.Atk absorb base, var2: +15%/SLV
    300.0000, 40.0000, 0.0000, -- var4: 300 flat, var5: +40/SLV
    -- UNUSED (var7–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    1950, 'icon_skill_active_priest_mana_shield', 0, 0.00, 0.00
);
```

---

---

# EF_INC_PARAM_BASED_PARAM — Type 32301

## What This Skill Does

A **passive** skill that increases one stat (the destination) by a percentage of another stat (the source). This enables cross-stat synergy passives: "your P.Atk gains 30% of your STR value", "your M.Def gains 20% of your INT value", etc. The boost is dynamic — as the source stat grows, the destination stat bonus grows with it.

**Use this type for**: synergy passives, cross-stat scaling buffs, class identity passives that reward stacking a primary stat.

**Source Reference**: `Game/Skill/StructSkill.cpp` → `INC_PARAM_BASED_PARAM()`

## Variable Mapping

| Variable | Type | Role | Notes |
|----------|------|------|-------|
| **`var1`** | float (bitmask) | **Destination stat** — stat to boost | Use standard bitmask values (e.g., `128.0000` = P.Atk). See [AGENTS.md — Standard Bitmasks](AGENTS.md#standard-bitmasks-var1-var4-var13-var16). |
| **`var2`** | float (bitmask) | **Source stat** — stat to scale from | Use standard bitmask values (e.g., `1.0000` = STR). |
| **`var3`** | float | **Conversion ratio — base** | Fraction of source stat added to destination. `0.3000 = 30%`. Uses `1 = 100%` convention. |
| **`var4`** | float | **Conversion ratio — per SLV** | `0.0200 = +2%` per SLV. |
| **`var5`** | float | **Conversion ratio — per enhance** | |
| `var6`–`var20` | — | **UNUSED** | Always `0.0000`. |

**Effective conversion formula**:
```
bonus = SourceStat × (var3 + var4×SLV + var5×Enhance)
```

This is a passive — the engine continuously applies the bonus. It does not deal damage or cast a state.

> Standard bitmask table for `var1` and `var2`: see [AGENTS.md — Standard Bitmasks](AGENTS.md#standard-bitmasks-var1-var4-var13-var16). Extended bitmasks (elemental/penetration stats) are NOT supported in `var1`/`var2` for this type — only standard stat bitmasks apply.

## Mandatory Column Defaults for Type 32301

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `32301` | Required |
| `is_passive` | `'0'` | **Passive** skill (`0 = !is_active`) |
| `is_harmful` | `'0'` | Self-buff |
| `is_physical_act` | `'0'` | No cast |
| `is_need_target` | `'0'` | No target |
| `elemental` | `'0'` | Passives have no element |
| All cost fields | `0` / `0.00` | Passive — no cost |
| `target` | `41` | Standard passive loop target |
| `delay_common` | `1.50` | Standard passive loop interval |
| All other delays | `0.00` | |
| `vf_is_not_need_weapon` | `'1'` | No weapon required (adjust if restricted) |
| `var6`–`var20` | `0.0000` | UNUSED |

## Example INSERT — "Warrior's Resolve" (ID: 91265)

Passive: adds 25% of STR (bitmask `1`) to P.Atk (bitmask `128`). Conversion increases by +2% per SLV.

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
    91265, 50091265, 0, 40091265, 1, '0',
    '0', '0', '0', '0',       -- passive
    '0', '0', 0, '0',
    '0', 0, 0,
    0, 0, 0, 0, 0,
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
    0.00, 0.00, 0.00,
    1.50,
    0.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '0',
    '0', '0', '0',
    0,
    41, 32301, 0,
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.00,
    0, 0.00, 0.00,
    0, 0,
    -- Cross-stat scaling
    128.0000,                  -- var1: Destination = P.Atk (bitmask 128)
    1.0000,                    -- var2: Source = STR (bitmask 1)
    0.2500,                    -- var3: 25% of STR added to P.Atk base
    0.0200,                    -- var4: +2% conversion per SLV
    0.0000,                    -- var5: no enhance scaling
    -- UNUSED (var6–var20)
    0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000,
    2000, 'icon_skill_pas_warrior_resolve', 0, 0.00, 0.00
);
```

---

## Quick Reference: All Three Types

| | 32062 | 32211 | 32301 |
|---|---|---|---|
| **Category** | Active damage | Active defensive | Passive stat |
| **is_passive** | `'1'` | `'1'` | `'0'` |
| **Primary mechanic** | Magic chain bounce | Damage absorb shield | Cross-stat conversion |
| **Formula base** | 231 magic damage | 231 formula → shield amount | Source stat × ratio |
| **var1–var6** | Magic damage | Shield amount | var1/var2: bitmasks; var3–var5: ratio |
| **var7** | Chain count (int) | UNUSED | UNUSED |
| **var8** | Decay per bounce | UNUSED | UNUSED |
| **var9** | Jump radius | UNUSED | UNUSED |
| **state_id** | Per-chain-hit | Optional visual state | UNUSED |
| **target** | `1` (single) | `1` (single) | `41` (passive) |
| **delay_common** | `0.00` | `0.00` | `1.50` |

---

## Practical Constraints

### EF_MAGIC_CHAIN_DAMAGE (32062)
- **`var7 = 1.0000`** produces a skill that only hits the primary target with no chain — effectively identical to type 231 (but slower to process). Always set `var7 ≥ 2.0000` for a real chain.
- **`var8 = 1.0000`** means no damage reduction per bounce — every target takes identical damage. This can be extremely powerful in dense groups; balance carefully.
- **`var9` must be > 0**: a zero jump radius prevents the chain from finding any bounce target. The chain silently terminates after the primary hit.

### EF_ABSORB_DAMAGE (32211)
- **`var1` must not be `0.0`** if M.Atk scaling is intended. A zero multiplier produces a flat-only shield that ignores the caster's stats.
- **The `state_id` is not required**: use it to track the visual/buff indicator for the shield. The engine does not require it to calculate the absorption — it is purely cosmetic/informational.
- **The shield is not permanent**: if `state_id` controls the shield's visual, ensure `state_second` matches the intended shield lifespan. The absorb mechanic itself may have its own timeout separate from the state duration — verify against source if the shield should expire before it is fully exhausted.

### EF_INC_PARAM_BASED_PARAM (32301)
- **`var2` (source stat) must be a standard bitmask**: extended bitmasks (elemental, penetration) are not valid sources for this type. Only stats in the standard bitmask table apply.
- **Circular references are a logic error**: never set `var1 == var2` (boosting a stat based on itself). The result is engine-dependent and likely undefined.
- **This stacks with EF_PARAMETER_AMP**: both passives can be active simultaneously. A character could have EF_PARAMETER_AMP boosting P.Atk by a flat % AND EF_INC_PARAM_BASED_PARAM adding P.Atk from STR. Design with both in mind to avoid double-stacking imbalance.

---

*Source: `Game/Skill/StructSkill.cpp` · `MAGIC_CHAIN_DAMAGE()` · `ABSORB_DAMAGE()` · `INC_PARAM_BASED_PARAM()`*
