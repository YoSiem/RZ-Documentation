# Effect Types: EF_ADD_HP (501) & EF_TOGGLE_AURA (701)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [States and Enhancements Reference](10_Skill_States_and_Enhancements.md) | [Skill Target and Usage](09_Skill_Target_and_Usage.md) | [Casting and Timing](07_Skill_Casting_and_Timing.md)

These two effect types are documented together as **utility** types — neither deals direct damage. They are otherwise unrelated: 501 heals HP on cast, 701 maintains a persistent area-of-effect aura via the toggle system.

**Source Reference**: `Game/Skill/StructSkill.cpp`
- 501 → `ADD_HP()`
- 701 → `TOGGLE_AURA()`

---

---

# EF_ADD_HP — Type 501

## What This Skill Does

Restores HP to the target immediately on cast. The amount healed follows the **same formula structure as the magic damage family**, but the result is applied as a heal rather than damage. The caster's M.Atk (modified by elemental) is the primary scaling stat.

**Use this type for**: direct heals, burst heals, single-target restoration spells.

**Source Reference**: `Game/Skill/StructSkill.cpp` → `ADD_HP()`

---

## Heal Formula (var1–var6)

```
nHeal = M.Atk × (var1 + var2×SLV + var3×Enhance)
      + (var4  + var5×SLV + var6×Enhance)
```

This is structurally identical to the damage formula in [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md#2-c-source-analysis). The `M.Atk` here is the **healer's M.Atk** — for magical healing classes, their magic power directly scales their heal output.

| Variable | C++ Ref | Type | Role |
|----------|---------|------|------|
| **`var1`** | `GetVar(0)` | `float` | M.Atk heal multiplier — base |
| **`var2`** | `GetVar(1)` | `float` | M.Atk heal multiplier — per skill level |
| **`var3`** | `GetVar(2)` | `float` | M.Atk heal multiplier — per enhance |
| **`var4`** | `GetVar(3)` | `float` | Flat HP restore — base |
| **`var5`** | `GetVar(4)` | `float` | Flat HP restore — per skill level |
| **`var6`** | `GetVar(5)` | `float` | Flat HP restore — per enhance |
| `var7`–`var20` | — | — | **UNUSED** — always `0.0000` |

---

## On-Heal State (Optional)

Type 501 supports an optional **state applied on heal** using the same state columns as the magic damage types. This allows heal-over-time (HoT) effects or regen buffs to be triggered by the cast.

| Column | Role | Notes |
|--------|------|-------|
| `state_id` | State applied after healing | `0` for no secondary effect |
| `probability_on_hit` | Chance (%) to apply the state | Use `100` for guaranteed |
| `state_second` | Duration | As normal |

> Set `state_id = 0` and `probability_on_hit = 0` for a pure instant heal with no secondary state.

---

## Mandatory Column Defaults for Type 501

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `501` | Required |
| `is_passive` | `'1'` | Active skill |
| `is_harmful` | `'0'` | Heal — never harmful |
| `is_physical_act` | `'0'` | Magical healing |
| `is_need_target` | `'1'` | Requires a target to heal |
| `target` | `1` | `TARGET_TARGET` — single target heal |
| `uf_self` | `'1'` | Can always heal yourself |
| `uf_party` | `'1'` | Can heal party members |
| `uf_enemy` | `'0'` | Cannot heal enemies |
| `tf_avatar` | `'1'` | Heal player characters |
| `tf_summon` | `'1'` | Can heal pets/summons (if intended) |
| `tf_monster` | `'0'` | Cannot heal monsters |
| `hate_mod` | `≥ 1.00` | Healing generates aggro — set appropriately |
| `var7`–`var20` | `0.0000` | UNUSED |

> **Aggro note**: Heals generate threat on all enemies currently fighting the healed target. `hate_mod = 1.50` is a reasonable starting point for a strong heal. Leave `hate_basic`, `hate_per_skl`, `hate_per_enhance` at `0` unless the skill is designed as a taunt.

---

## Example INSERT — "Holy Light" (ID: 91230)

Light-element single-target heal: restores `M.Atk × (1.20 + 0.08×SLV)` plus `200 + 30×SLV` flat HP. No secondary state.

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
    91230, 50091230, 0, 40091230, 1, '5',   -- elemental='5' = Light
    '1', '0', '0', '1',                      -- active, not physical, NOT harmful, needs target
    '0', '0', 0,
    '1', '1',
    20, 0,                                    -- cast_range=20, valid_range=0 (single target)
    0, 0,
    100, 12, -8,
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
    8.00, 0.00, 0.00,
    0,
    '1', '1', '1', '0',                       -- uf_self=1, uf_party=1, uf_guild=1
    '0', '0',                                 -- uf_enemy=0
    '1', '1', '0',                            -- tf_avatar=1, tf_summon=1, tf_monster=0
    0,
    1, 501, 0,                                -- target=1 (single), effect_type=501
    -- No secondary state
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.50,                                     -- hate_mod = 1.5x (heals generate significant aggro)
    0, 0.00, 0.00,
    0, 0,
    -- Heal formula
    1.2000, 0.0800, 0.0000,   -- var1: 120% M.Atk, var2: +8%/SLV, var3: 0
    200.0000, 30.0000, 0.0000, -- var4: 200 flat HP, var5: +30/SLV, var6: 0
    -- UNUSED (var7–var20)
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    1400, 'icon_skill_active_priest_holy_light', 0, 0.00, 0.00
);
```

---

---

# EF_TOGGLE_AURA — Type 701

## What This Skill Does

Creates a **persistent, player-controlled aura** around the caster. While active, the aura continuously applies (or maintains) `state_id` on all valid entities within `valid_range` of the caster. The player can toggle the aura on or off manually; it stays active until deactivated, the caster dies, or the caster logs out.

**Use this type for**: passive auras, persistent group buffs, environmental damage zones, leader buffs that pulse to nearby allies.

**Source Reference**: `Game/Skill/StructSkill.cpp` → `TOGGLE_AURA()`

---

## Toggle Mechanics

Type 701 uses the toggle system:

| Column | Value | Notes |
|--------|-------|-------|
| `is_toggle` | `'1'` | **Required** — activates the toggle skill loop |
| `toggle_group` | `> 0` | Group ID; only one aura per group can be active at a time. Prevents stacking the same aura twice or two auras of the same group. |
| `delay_cast` | e.g. `0.00` | Most auras activate instantly. Set a non-zero value only if there is a winding-up animation. |
| `delay_cooltime` | e.g. `1.00` | Short cooldown prevents instant re-toggle abuse. |

> **`toggle_group` is critical**: Two aura skills sharing the same `toggle_group` value will deactivate each other when cast. Use different group IDs for independent auras.

---

## Aura Range and Targeting

| Column | Value | Notes |
|--------|-------|-------|
| `target` | `4` | `TARGET_AOE` — general area around the caster |
| `is_need_target` | `'0'` | No entity is selected; the aura radiates from the caster's position |
| `valid_range` | e.g. `15` | Radius of the aura in engine units |
| `cast_range` | `0` | Irrelevant for self-centered auras; set to `0` |

---

## Variable Mapping for Type 701

| Variable | Type | Role |
|----------|------|------|
| **`var1`** | `float` | **Tick interval** — how often (in seconds) the aura re-applies the state to entities in range. `2.0000` = state is refreshed every 2 seconds on all valid targets. |
| `var2`–`var20` | — | **UNUSED** — always `0.0000` |

> If `var1 = 0.0000`, the aura applies the state only once on entry and does not re-pulse. For most persistent auras, set `var1` to the desired refresh interval (e.g., `3.0000`).

---

## State Configuration for Type 701

The aura applies `state_id` to each entity in range on every tick (per `var1`). The state columns follow the same rules as [States and Enhancements](10_Skill_States_and_Enhancements.md):

| Column | Role |
|--------|------|
| `state_id` | The buff/debuff the aura applies |
| `state_level_base` / `state_level_per_skl` | Level of the state applied per tick |
| `state_second` | Duration of the state; should be ≥ `var1` to keep it active continuously |
| `probability_on_hit` | Chance per tick to apply the state (use `100` for guaranteed auras) |

> **Duration rule**: Set `state_second` to at least `var1 + buffer` (e.g., `var1 = 2.0`, `state_second = 4.0`) so the state does not expire between ticks.

---

## Mandatory Column Defaults for Type 701

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `701` | Required |
| `is_toggle` | `'1'` | **Required** |
| `toggle_group` | `≥ 1` | Unique per aura group |
| `is_passive` | `'1'` | Active skill (player must activate it) |
| `is_harmful` | `'0'` buff aura / `'1'` debuff aura | |
| `is_need_target` | `'0'` | Self-centered, no target needed |
| `target` | `4` | `TARGET_AOE` |
| `valid_range` | aura radius | |
| `cast_range` | `0` | |
| `uf_self` | `'1'` if aura affects caster too | |
| `uf_party` | `'1'` for ally auras | |
| `uf_enemy` | `'1'` for hostile auras | |
| `var1` | tick interval (seconds) | Only meaningful variable |
| `var2`–`var20` | `0.0000` | UNUSED |

---

## Example INSERT — "Guardian Aura" (ID: 91231)

Light-element ally aura: pulses every 3 seconds, applying a defense buff (state 12520) at level 1 per SLV to all nearby party members within radius 15. State lasts 5 seconds per pulse (longer than the tick — continuous coverage guaranteed).

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
    91231, 50091231, 0, 40091231, 1, '5',   -- elemental='5' = Light
    '1', '0', '0', '0',                      -- active, not physical, NOT harmful, NO target needed
    '0', '1', 10,                             -- is_toggle='1', toggle_group=10
    '0', '0',                                 -- casting_type=0 (uninterruptable), casting_level=0
    0, 15,                                    -- cast_range=0 (self-centered), valid_range=15 (aura radius)
    0, 0,
    50, 5, -3,                                -- low MP cost while active
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
    0.00, 0.00, 0.00,                         -- instant toggle activation
    0.00,
    1.00, 0.00, 0.00,                         -- short cooldown (prevents instant re-toggle)
    0,
    '1', '1', '1', '0',                       -- uf_self=1, uf_party=1, uf_guild=1
    '0', '0',                                 -- uf_enemy=0
    '1', '1', '0',                            -- tf_avatar=1, tf_summon=1, tf_monster=0
    0,
    4, 701, 0,                                -- target=4 (AOE), effect_type=701
    -- Aura state: defense buff pulsed every tick
    12520, 0, 1.00, 0.00,                     -- state_id=12520, level=SLV
    5.00, 0.00, 0.00,                         -- 5-second duration per pulse (> tick interval of 3s)
    100, 0,                                   -- 100% chance (guaranteed buff)
    0, 0, 0,
    1.00,
    0, 0.00, 0.00,
    0, 0,
    -- var1 = tick interval
    3.0000,                                   -- var1: aura pulses every 3 seconds
    -- UNUSED (var2–var20)
    0.0000, 0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000,
    1500, 'icon_skill_active_support_guardian_aura', 0, 0.00, 0.00
);
```

---

## Full Comparison: Types 501 / 701

| | 501 | 701 |
|---|---|---|
| **Effect** | Instant HP restore on cast | Persistent area buff/debuff pulsed on tick |
| **is_toggle** | `'0'` | `'1'` |
| **toggle_group** | `0` | `≥ 1` (required) |
| **target** | `1` (single target) | `4` (AoE around caster) |
| **is_need_target** | `'1'` | `'0'` |
| **valid_range** | `0` (no AoE) | Aura radius |
| **Primary vars** | `var1`–`var6` (heal formula) | `var1` only (tick interval) |
| **var2–var20** | UNUSED | UNUSED |
| **state_id** | Optional (post-heal buff) | Required (the aura's effect) |
| **hate_mod** | Set above 1.00 (heals generate aggro) | `1.00` for buffs |
| **is_harmful** | `'0'` always | `'0'` buff / `'1'` hostile aura |

---

## Practical Constraints

### EF_ADD_HP (501)
- **`var1` must never be `0.0`** unless you intend a flat-only heal. A zero multiplier ignores the caster's M.Atk entirely.
- **`is_harmful` must always be `'0'`**: the engine will not apply a heal if the skill is flagged as harmful.
- **`elemental` matters**: the Light (`'5'`) element typically amplifies heals for light-specced characters. Use `'0'` for non-elemental heals.
- **`tf_monster = '0'`**: never allow players to heal enemy monsters — set this to `'0'` always for heal skills.

### EF_TOGGLE_AURA (701)
- **`toggle_group` must be non-zero**: a value of `0` disables the mutual-exclusion system and allows multiple auras of the same group to stack simultaneously.
- **`state_second` ≥ `var1 + 1`**: if the state expires before the next pulse, targets will briefly lose the buff. Always set duration longer than the tick interval.
- **`var1 = 0.0000`**: the aura fires once on activation and never re-pulses. Only use this if the intended behavior is a one-time area trigger (not a true pulsing aura).
- **`is_passive = '1'`** (not `'0'`): auras are active skills activated by the player — they are not automatic passives. `is_passive = '0'` would flag the engine to treat this as an automatic passive, which conflicts with the toggle system.

---

*Source: `Game/Skill/StructSkill.cpp` · `ADD_HP()` · `TOGGLE_AURA()`*
