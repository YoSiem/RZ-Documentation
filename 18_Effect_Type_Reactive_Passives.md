# Effect Types: Reactive Passives — Combat Event Triggers (10048, 10049, 10052, 10055, 10056, 10063)

> **Navigation**: [← Table of Contents](00_Table_of_Contents.md) | [States and Enhancements Reference](10_Skill_States_and_Enhancements.md) | [EF_WEAPON_MASTERY (10001)](17_Effect_Type_Weapon_Mastery.md)

These six effect types are **reactive passive skills**: they fire automatically when a specific combat event occurs, without any player action. The trigger condition is encoded directly in the `effect_type` value — the developer configures only **what happens** (the state applied or cooldown reduced) and **how often it procs** (the var columns).

**Source Reference**: `Game/Skill/StructSkill.cpp`
- 10048 → `ADD_STATE_ON_ATTACK()`
- 10049 → `ADD_STATE_BY_SELF_ON_ATTACK()`
- 10052 → `ADD_STATE_BY_SELF_ON_KILL()`
- 10055 → `ADD_STATE_ON_BEING_CRITICAL_ATTACKED()`
- 10056 → `ADD_STATE_BY_SELF_ON_BEING_CRITICAL_ATTACKED()`
- 10063 → `INC_SKILL_COOL_TIME_ON_ATTACK()`

---

## Naming Convention: "BY_SELF" vs. No Suffix

The naming pattern determines **who receives the state**:

| Pattern | State applied to |
|---------|-----------------|
| `ADD_STATE_ON_...` | The **other party** (target you attacked / attacker who hit you) |
| `ADD_STATE_BY_SELF_ON_...` | **Yourself** (the caster) |

This distinction is the single most important thing to understand before choosing between 10048/10049 or 10055/10056.

---

## Common Passive Column Defaults (All 6 Types)

These values apply to **all six** reactive passive types unless specified otherwise in each section:

| Column | Value | Notes |
|--------|-------|-------|
| `is_passive` | `'0'` | Passive skill (`0 = !is_active` in engine) |
| `is_physical_act` | `'0'` | No cast trigger |
| `is_need_target` | `'0'` | Engine provides target from combat event context |
| `is_toggle` | `'0'` | Always-on passive |
| `elemental` | `'0'` | Proc mechanics ignore element |
| All cost fields | `0` / `0.00` | Passive — no cast cost |
| `cast_range`, `valid_range` | `0` | Not applicable |
| `target` | `41` | Standard passive self-loop target |
| `delay_common` | `1.50` | Standard passive loop interval |
| All other delays | `0.00` | |
| `vf_is_not_need_weapon` | `'1'` | Procs regardless of weapon (unless you want to restrict) |
| All other vf_ | `'0'` | |

---

## Shared Variable Layout: Proc Probability (Types 10048, 10049, 10052, 10055, 10056)

For all five state-on-event types, the var columns control **only the proc probability**. The state itself is fully configured via the standard state columns.

| Variable | Type | Role |
|----------|------|------|
| **`var1`** | float | **Proc chance — base** (`1.0 = 100%` convention: `0.1500 = 15%`) |
| **`var2`** | float | **Proc chance — per skill level** (`0.0200 = +2% per SLV`) |
| **`var3`** | float | **Proc chance — per enhance** |
| `var4`–`var20` | — | **UNUSED** — always `0.0000` |

**Effective proc chance formula**: `Chance = (var1 + var2×SLV + var3×Enhance) × 100%`

> Example: `var1 = 0.1500`, `var2 = 0.0200`, SLV 5 → `(0.15 + 0.10) × 100 = 25%` proc chance.

The state itself is configured via state columns — see [States and Enhancements](10_Skill_States_and_Enhancements.md).

---

---

# EF_ADD_STATE_ON_ATTACK — Type 10048

## What This Skill Does

When the caster lands a normal attack on an enemy, there is a `var1`-based chance to apply `state_id` to **the target** (the enemy hit). Fires on every qualifying auto-attack or physical skill hit, depending on engine configuration.

**Use this type for**: on-hit debuff procs, poison-on-hit passives, blind-on-attack effects.

## Column Specifics

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `10048` | Required |
| `is_harmful` | `'1'` | State goes to enemy |
| `uf_enemy` | `'1'` | Target faction: enemies |
| `uf_self` | `'0'` | Does not affect caster |
| `tf_avatar`, `tf_summon`, `tf_monster` | Set per design | Who can be procced against |

## Example INSERT — "Venom Strike" (ID: 91250)

Passive proc: 15% base chance (+2%/SLV) to apply poison (state 12530) on any attack.

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
    91250, 50091250, 0, 40091250, 1, '0',
    '0', '0', '1', '0',        -- passive, not physical, harmful (debuff on enemy), no target
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
    '1',                        -- vf_is_not_need_weapon = '1'
    0.00, 0.00, 0.00,
    1.50,
    0.00, 0.00, 0.00,
    0,
    '0', '0', '0', '0',
    '0', '1',                   -- uf_enemy = '1'
    '1', '1', '1',
    0,
    41, 10048, 0,
    -- State: poison applied on proc
    12530, 0, 1.00, 0.00,
    8.00, 1.00, 0.00,
    100, 0,                     -- probability_on_hit=100 (proc chance handled by var1, not this column)
    0, 0, 0,
    1.00,
    0, 0.00, 0.00,
    0, 0,
    -- Proc probability
    0.1500, 0.0200, 0.0000,    -- var1: 15% base, var2: +2%/SLV, var3: 0
    -- UNUSED (var4–var20)
    0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000,
    1650, 'icon_skill_pas_rogue_venom_strike', 0, 0.00, 0.00
);
```

> **Note on `probability_on_hit`**: For reactive passives, the proc roll is handled internally via `var1`. Set `probability_on_hit = 100` in the state columns (full application once the proc fires), as these two systems operate independently.

---

---

# EF_ADD_STATE_BY_SELF_ON_ATTACK — Type 10049

## What This Skill Does

Identical trigger to type 10048 (landing a normal attack), but the state is applied to **the caster** rather than the target.

**Use this type for**: bloodlust/berserk procs on attack, temporary attack speed buffs triggered by hitting, rage stacks.

## Column Specifics

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `10049` | Required |
| `is_harmful` | `'0'` | State goes to caster (buff) |
| `uf_self` | `'1'` | Caster faction |
| `uf_enemy` | `'0'` | Does not affect the target |

## Key Difference from Type 10048

Only two columns differ: `effect_type = 10049`, `is_harmful = '0'`, `uf_self = '1'`, `uf_enemy = '0'`. All var layout and state columns are identical.

---

---

# EF_ADD_STATE_BY_SELF_ON_KILL — Type 10052

## What This Skill Does

Fires when the caster **kills** an enemy (delivers the killing blow). On proc, applies `state_id` to **the caster**. The kill event fires once per kill regardless of how many enemies are nearby.

**Use this type for**: kill-streak buffs, soul-harvest mechanics, execute-triggered empowerment.

## Column Specifics

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `10052` | Required |
| `is_harmful` | `'0'` | Self-buff on kill |
| `uf_self` | `'1'` | |
| `uf_enemy` | `'0'` | |

> With `var1 = 1.0000`, the state is guaranteed to apply on every kill. Lower values add RNG to the mechanic.

---

---

# EF_ADD_STATE_ON_BEING_CRITICAL_ATTACKED — Type 10055

## What This Skill Does

Fires when the caster **receives a critical hit** from an enemy. On proc, applies `state_id` to **the attacker** (the enemy who landed the crit). This is a counter-reactive debuff.

**Use this type for**: thorns-style critical counter-procs, "blind my attacker when critted" defenses.

## Column Specifics

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `10055` | Required |
| `is_harmful` | `'1'` | Debuff applied to the attacker |
| `uf_enemy` | `'1'` | Target is the enemy who critted you |
| `uf_self` | `'0'` | Does not affect caster |

---

---

# EF_ADD_STATE_BY_SELF_ON_BEING_CRITICAL_ATTACKED — Type 10056

## What This Skill Does

Same trigger as type 10055 (receiving a critical hit), but applies the state to **the caster** instead of the attacker. This is a defensive reactive proc — being critted activates a personal shield or counter-buff.

**Use this type for**: "tank stance" procs on being critted, emergency shields, counter-buff mechanics.

## Column Specifics

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `10056` | Required |
| `is_harmful` | `'0'` | Self-buff triggered by incoming crit |
| `uf_self` | `'1'` | |
| `uf_enemy` | `'0'` | |

---

---

# EF_INC_SKILL_COOL_TIME_ON_ATTACK — Type 10063

## What This Skill Does

When the caster lands a normal attack, reduces the remaining cooldown of a designated skill (or skill group) by a flat amount in seconds. This enables "attack to reduce cooldown" mechanics — hitting enemies charges a skill faster.

**Use this type for**: CDR-on-hit passives, charge-by-attacking mechanics, attack-gated skill resets.

**Source Reference**: `Game/Skill/StructSkill.cpp` → `INC_SKILL_COOL_TIME_ON_ATTACK()`

## Variable Mapping (Differs from Other Reactive Types)

| Variable | Type | Role |
|----------|------|------|
| **`var1`** | float | **Cooldown reduction per attack hit — base** (in seconds; `0.5000 = 0.5s`) |
| **`var2`** | float | **Reduction per skill level** (`0.1000 = +0.1s per SLV`) |
| **`var3`** | float | **Reduction per enhance** |
| `var4`–`var20` | — | **UNUSED** — always `0.0000` |

**Effective reduction per hit**: `Reduction = var1 + var2×SLV + var3×Enhance` seconds

## Target Skill Reference

The skill whose cooldown is reduced is identified by the **`cool_time_group_id`** column. The engine reduces the remaining cooldown of all skills sharing that group ID.

| Column | Role |
|--------|------|
| `cool_time_group_id` | ID of the cooldown group to accelerate. Set to the group ID of the skill you want to charge. Set to `0` for no specific target (may affect all skills — verify per source). |

> **Important**: This type does **not** use the state columns (`state_id`, `probability_on_hit`, etc.). Set all state columns to `0`.

## Column Specifics

| Column | Value | Notes |
|--------|-------|-------|
| `effect_type` | `10063` | Required |
| `is_harmful` | `'0'` | Self-benefit (reduces your own cooldown) |
| `uf_self` | `'1'` | |
| `uf_enemy` | `'0'` | |
| `cool_time_group_id` | target group | The skill group whose CDR is reduced |
| `state_id` | `0` | Unused |
| `probability_on_hit` | `0` | Unused |

## Example INSERT — "Swift Blade" (ID: 91251)

Passive: each attack reduces the cooldown of skills in group `5` by 0.5s base (+0.1s/SLV).

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
    91251, 50091251, 0, 40091251, 1, '0',
    '0', '0', '0', '0',
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
    5,                          -- cool_time_group_id = 5 (target skill group)
    '1', '0', '0', '0',         -- uf_self = '1'
    '0', '0',
    '0', '0', '0',
    0,
    41, 10063, 0,
    -- State columns all zeroed (unused for type 10063)
    0, 0, 0.00, 0.00,
    0.00, 0.00, 0.00,
    0, 0,
    0, 0, 0,
    1.00,
    0, 0.00, 0.00,
    0, 0,
    -- CDR per attack
    0.5000, 0.1000, 0.0000,    -- var1: 0.5s CDR base, var2: +0.1s/SLV, var3: 0
    -- UNUSED (var4–var20)
    0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000, 0.0000,
    0.0000, 0.0000, 0.0000,
    1700, 'icon_skill_pas_warrior_swift_blade', 0, 0.00, 0.00
);
```

---

## Full Comparison: All 6 Reactive Passive Types

| | 10048 | 10049 | 10052 | 10055 | 10056 | 10063 |
|---|---|---|---|---|---|---|
| **Trigger** | You land attack | You land attack | You kill enemy | You receive crit | You receive crit | You land attack |
| **State target** | Your target (enemy) | Yourself | Yourself | Your attacker (enemy) | Yourself | — (CDR effect) |
| **is_harmful** | `'1'` | `'0'` | `'0'` | `'1'` | `'0'` | `'0'` |
| **uf_enemy** | `'1'` | `'0'` | `'0'` | `'1'` | `'0'` | `'0'` |
| **uf_self** | `'0'` | `'1'` | `'1'` | `'0'` | `'1'` | `'1'` |
| **var1** | Proc % base | Proc % base | Proc % base | Proc % base | Proc % base | CDR seconds base |
| **var2** | Proc % per SLV | Proc % per SLV | Proc % per SLV | Proc % per SLV | Proc % per SLV | CDR per SLV |
| **var3** | Proc % per enh | Proc % per enh | Proc % per enh | Proc % per enh | Proc % per enh | CDR per enh |
| **state_id** | Required | Required | Required | Required | Required | `0` (unused) |
| **cool_time_group_id** | `0` | `0` | `0` | `0` | `0` | Target group ID |

---

## Practical Constraints

- **`var1` uses `1 = 100%` float convention**: `0.1500 = 15%`, `1.0000 = 100%` (guaranteed proc). This is consistent with EF_PARAMETER_AMP math, not the integer percentage of `probability_on_hit`.
- **`probability_on_hit` is not the proc rate**: For reactive passives, the proc chance is exclusively `var1`. The `probability_on_hit` column controls only the state-application chance *after* a proc fires — set it to `100` for guaranteed state application once the proc triggers.
- **State duration matters for kill/crit types**: For 10052 and 10056, the state must last long enough to be meaningful since the trigger is infrequent. Use generous `state_second` values.
- **Type 10063 ignores all state columns**: Do not set `state_id` for type 10063 — the handler does not read them and any non-zero value is dead data.
- **`cool_time_group_id` for type 10063**: Must match the `cool_time_group_id` assigned to the skill(s) you want to charge. See [Casting and Timing](07_Skill_Casting_and_Timing.md#cooldowns-and-delays) for the `cool_time_group_id` reference.

---

*Source: `Game/Skill/StructSkill.cpp` · `ADD_STATE_ON_ATTACK()` · `ADD_STATE_BY_SELF_ON_ATTACK()` · `ADD_STATE_BY_SELF_ON_KILL()` · `ADD_STATE_ON_BEING_CRITICAL_ATTACKED()` · `ADD_STATE_BY_SELF_ON_BEING_CRITICAL_ATTACKED()` · `INC_SKILL_COOL_TIME_ON_ATTACK()`*
