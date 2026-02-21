# SQL Skill Template Reference

> This is the **single source of truth** for generating `SkillResource` INSERT statements.
> Effect type docs document only what is **unique** to each type. All shared patterns live here.

---

## 1. Column Schema (copy this header once per INSERT)

```sql
INSERT INTO [dbo].[SkillResource] (
    [id],[text_id],[desc_id],[tooltip_id],[is_valid],[elemental],
    [is_passive],[is_physical_act],[is_harmful],[is_need_target],
    [is_corpse],[is_toggle],[toggle_group],[casting_type],
    [casting_level],[cast_range],[valid_range],[cost_hp],
    [cost_hp_per_skl],[cost_mp],[cost_mp_per_skl],[cost_mp_per_enhance],
    [cost_hp_per],[cost_hp_per_skl_per],[cost_mp_per],[cost_mp_per_skl_per],
    [cost_havoc],[cost_havoc_per_skl],[cost_energy],[cost_energy_per_skl],
    [cost_exp],[cost_exp_per_enhance],[cost_jp],[cost_jp_per_enhance],
    [cost_item],[cost_item_count],[cost_item_count_per_skl],[need_level],
    [need_hp],[need_mp],[need_havoc],[need_havoc_burst],[need_state_id],
    [need_state_level],[need_state_exhaust],[vf_one_hand_sword],
    [vf_two_hand_sword],[vf_double_sword],[vf_dagger],[vf_double_dagger],
    [vf_spear],[vf_axe],[vf_one_hand_axe],[vf_double_axe],
    [vf_one_hand_mace],[vf_two_hand_mace],[vf_lightbow],[vf_heavybow],
    [vf_crossbow],[vf_one_hand_staff],[vf_two_hand_staff],[vf_shield_only],
    [vf_is_not_need_weapon],[delay_cast],[delay_cast_per_skl],
    [delay_cast_mode_per_enhance],[delay_common],[delay_cooltime],
    [delay_cooltime_per_skl],[delay_cooltime_mode_per_enhance],
    [cool_time_group_id],[uf_self],[uf_party],[uf_guild],[uf_neutral],
    [uf_purple],[uf_enemy],[tf_avatar],[tf_summon],[tf_monster],
    [skill_lvup_limit],[target],[effect_type],[skill_enchant_link_id],
    [state_id],[state_level_base],[state_level_per_skl],[state_level_per_enhance],
    [state_second],[state_second_per_level],[state_second_per_enhance],
    [probability_on_hit],[probability_inc_by_slv],[hit_bonus],
    [hit_bonus_per_enhance],[percentage],[hate_mod],[hate_basic],
    [hate_per_skl],[hate_per_enhance],[critical_bonus],[critical_bonus_per_skl],
    [var1],[var2],[var3],[var4],[var5],[var6],[var7],[var8],[var9],[var10],
    [var11],[var12],[var13],[var14],[var15],[var16],[var17],[var18],[var19],
    [var20],[icon_id],[icon_file_name],[is_projectile],[projectile_speed],
    [projectile_acceleration]
)
```

---

## 2. ID Formula

| Column | Formula | Example (id=91200) |
|--------|---------|-------------------|
| `id` | Custom, must be > `91000`, increment by 1 | `91200` |
| `text_id` | `50000000 + id` | `50091200` |
| `desc_id` | Always `0` | `0` |
| `tooltip_id` | `40000000 + id` | `40091200` |
| `is_valid` | Always `1` | `1` |

---

## 3. Key Value Rules

| Rule | Value | Notes |
|------|-------|-------|
| `is_passive` | `'0'` = passive, `'1'` = active | **INVERTED** — `0` means `!is_active` in engine |
| `elemental` | `'0'`–`'6'` | `'0'`=none `'1'`=earth `'2'`=fire `'3'`=water `'4'`=wind `'5'`=light `'6'`=dark |
| Float multipliers | `1.0 = 100%` | Applies to all `var` multipliers and EF_PARAMETER_AMP bitmask slots |
| Flat damage vars | Raw units | `var4`–`var6` in damage types are raw HP values, **not** fractions |
| `target` | `1`=single `2`=region w/ target `3`=region w/o target `4`=AoE `41`=passive | See [09_Skill_Target_and_Usage.md](09_Skill_Target_and_Usage.md) |
| `casting_type` | `'0'`=uninterruptable `'1'`=interruptable | |
| `var19`, `var20` | Always `0.0000` | Never read by any handler |

---

## 4. Compact VALUES Format

All INSERT examples in this documentation use the **compact VALUES** format below.
The column list from section 1 is always assumed — do not repeat it in examples.

```sql
-- [Skill Name] (ID: XXXXX) | effect_type=NNN | element=X
-- Brief description of what it does
INSERT INTO [dbo].[SkillResource] VALUES (
/*id  text_id  desc  tooltip  valid  elem*/  91200,50091200,0,40091200, 1,'3',
/*pasv phys harm need corp togg grp*/        '1','0','1','1','0','0',0,
/*cast_type cast_lvl cast_range valid_range*/ '1','1', 20,20,
/*cost_hp/skl  cost_mp/skl/enh*/             0,0, 120,12,-10,
/*cost_%hp/%hp_skl/%mp/%mp_skl*/             0.00,0.00,0.00,0.00,
/*havoc/skl energy/skl*/                     0,0,0.00,0.00,
/*exp/enh jp/enh*/                           0,0,0,0,
/*item count/skl*/                           0,0,0,
/*need: lvl hp mp havoc burst state_id lvl exhaust*/ 0,0,0,0,0,0,0,0,
/*vf: 1hsw 2hsw dbl dag dbldag spear axe 1haxe dblaxe 1hmace 2hmace lbow hbow xbow 1hstf 2hstf shld NOT_NEED*/
    '0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','1',
/*delay: cast/skl/enh  common*/              0.80,0.00,0.00, 0.00,
/*delay: cooltime/skl/enh  group_id*/        15.00,0.00,0.00, 0,
/*uf: self party guild neutral purple enemy*/'0','0','0','0','0','1',
/*tf: avatar summon monster*/                '1','1','1',
/*skill_lvup_limit  target  effect_type  enchant_link*/ 0, 1,231,0,
/*state: id  lvl_base  /skl  /enh*/         9999,0,1.00,0.00,
/*state: second  /lvl  /enh*/               6.00,0.00,0.00,
/*prob_on_hit  inc_by_slv*/                  70,0,
/*hit_bonus  /enh  percentage*/              0,0,0,
/*hate: mod  basic  /skl  /enh*/             1.20,0,0.00,0.00,
/*critical: bonus  /skl*/                    0,0,
/*var1-6*/  1.5000,0.0800,0.0000, 100.0000,20.0000,0.0000,
/*var7-20*/ 0.0000,0.0000,0.0000,0.0000, 0.0000,0.0000,0.0000,0.0000,
            0.0000,0.0000,0.0000,0.0000, 0.0000,0.0000,
/*icon  file  projectile speed accel*/       1100,'icon_skill_active_mage_frost_shard',0,0.00,0.00
);
```

---

## 5. Standard Presets

Copy the relevant preset, then override only the columns that differ for your skill.

### PRESET — Active Offensive (harmful, single target, magic)
```sql
VALUES (
/*id text desc tooltip valid elem*/   {id},50{id},0,40{id}, 1,'{elem}',
/*pasv phys harm need corp togg grp*/ '1','0','1','1','0','0',0,
/*cast_type lvl range valid*/          '1','1', {cast_range},0,
/*cost_hp/skl*/                        0,0,
/*cost_mp/skl/enh*/                    {mp},{mp_skl},0,
/*cost_%hp/skl/%mp/skl*/               0.00,0.00,0.00,0.00,
/*havoc/skl energy/skl*/               0,0,0.00,0.00,
/*exp/enh jp/enh*/                     0,0,0,0,
/*item/count/skl*/                     0,0,0,
/*need: lvl..exhaust (8 zeros)*/       0,0,0,0,0,0,0,0,
/*vf (17×'0', then not_need='1')*/     '0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','1',
/*delay cast/skl/enh common*/          {cast_time},0.00,0.00, 0.00,
/*delay cooltime/skl/enh group*/       {cooldown},0.00,0.00, 0,
/*uf*/                                 '0','0','0','0','0','1',
/*tf*/                                 '1','1','1',
/*lvup_limit target eff_type enchant*/ 0,1,{effect_type},0,
/*state: id lvl/skl/enh*/             {state_id},0,{state_lvl_skl},0.00,
/*state: sec/lvl/enh*/                {state_sec},0.00,0.00,
/*prob inc*/                           {prob},0,
/*hit_bonus /enh pct*/                 0,0,0,
/*hate: mod basic /skl /enh*/          {hate_mod},0,0.00,0.00,
/*crit: bonus /skl*/                   0,0,
/*var1-6*/  {v1},{v2},{v3}, {v4},{v5},{v6},
/*var7-20*/ 0.0000,0.0000,0.0000,0.0000, 0.0000,0.0000,0.0000,0.0000,
            0.0000,0.0000,0.0000,0.0000, 0.0000,0.0000,
/*icon file proj spd accel*/           {icon_id},'{icon_file}',0,0.00,0.00
);
```

**Override for no state**: `state_id=0, state_level_base=0, state_level_per_skl=0.00, state_level_per_enhance=0.00, state_second=0.00, state_second_per_level=0.00, state_second_per_enhance=0.00, probability_on_hit=0, probability_inc_by_slv=0`

---

### PRESET — Active Support (non-harmful, targeted buff/heal/shield)
```sql
VALUES (
/*id text desc tooltip valid elem*/   {id},50{id},0,40{id}, 1,'{elem}',
/*pasv phys harm need corp togg grp*/ '1','0','0','1','0','0',0,
/*cast_type lvl range valid*/          '1','1', {cast_range},0,
/*cost_hp/skl*/                        0,0,
/*cost_mp/skl/enh*/                    {mp},{mp_skl},0,
/*cost_%hp/skl/%mp/skl*/               0.00,0.00,0.00,0.00,
/*havoc/skl energy/skl*/               0,0,0.00,0.00,
/*exp/enh jp/enh*/                     0,0,0,0,
/*item/count/skl*/                     0,0,0,
/*need: lvl..exhaust*/                 0,0,0,0,0,0,0,0,
/*vf*/                                 '0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','1',
/*delay cast/skl/enh common*/          {cast_time},0.00,0.00, 0.00,
/*delay cooltime/skl/enh group*/       {cooldown},0.00,0.00, 0,
/*uf: self party guild neutral purple enemy*/ '1','1','1','0','0','0',
/*tf: avatar summon monster*/          '1','1','0',
/*lvup target eff enchant*/            0,1,{effect_type},0,
/*state id/lvl/skl/enh*/              {state_id},0,{state_lvl_skl},0.00,
/*state sec/lvl/enh*/                 {state_sec},0.00,0.00,
/*prob inc*/                           100,0,
/*hit_bonus /enh pct*/                 0,0,0,
/*hate mod basic /skl /enh*/           {hate_mod},0,0.00,0.00,
/*crit bonus /skl*/                    0,0,
/*var1-6*/  {v1},{v2},{v3}, {v4},{v5},{v6},
/*var7-20*/ 0.0000,0.0000,0.0000,0.0000, 0.0000,0.0000,0.0000,0.0000,
            0.0000,0.0000,0.0000,0.0000, 0.0000,0.0000,
/*icon file proj spd accel*/           {icon_id},'{icon_file}',0,0.00,0.00
);
```

---

### PRESET — Passive (always-on, EF_PARAMETER_AMP style or weapon mastery)
```sql
VALUES (
/*id text desc tooltip valid elem*/   {id},50{id},0,40{id}, 1,'0',
/*pasv phys harm need corp togg grp*/ '0','0','0','0','0','0',0,
/*cast_type lvl range valid*/          '0','0', 0,0,
/*all costs (12 values)*/              0,0,0,0,0, 0.00,0.00,0.00,0.00, 0,0,0.00,0.00,
/*exp/enh jp/enh item/count/skl*/      0,0,0,0, 0,0,0,
/*need: 8 zeros*/                      0,0,0,0,0,0,0,0,
/*vf: (adjust for weapon mastery)*/    '0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','1',
/*delay cast/skl/enh*/                 0.00,0.00,0.00,
/*delay common*/                       1.50,
/*delay cooltime/skl/enh group*/       0.00,0.00,0.00, 0,
/*uf: all zeros*/                      '0','0','0','0','0','0',
/*tf: all zeros*/                      '0','0','0',
/*lvup target eff enchant*/            0,41,{effect_type},0,
/*state: all zeros*/                   0,0,0.00,0.00, 0.00,0.00,0.00, 0,0,
/*hit_bonus /enh pct*/                 0,0,0,
/*hate: 1.00 zeros*/                   1.00,0,0.00,0.00,
/*crit: zeros*/                        0,0,
/*var1-18: bitmask slots*/             {v1},{v2},{v3}, {v4},{v5},{v6}, {v7},{v8},{v9}, {v10},{v11},{v12},
                                       {v13},{v14},{v15}, {v16},{v17},{v18},
/*var19-20: always 0*/                 0.0000,0.0000,
/*icon file proj spd accel*/           {icon_id},'{icon_file}',0,0.00,0.00
);
```

**For Weapon Mastery** (`vf_is_not_need_weapon = '0'`): replace `'1'` at position 18 in the vf_ block with `'0'`, and set the appropriate weapon flag to `'1'`.

---

### PRESET — Toggle Aura
```sql
VALUES (
/*id text desc tooltip valid elem*/   {id},50{id},0,40{id}, 1,'{elem}',
/*pasv phys harm need corp togg grp*/ '1','0','{harm}','0','0','1',{toggle_group},
/*cast_type lvl range valid*/          '0','0', 0,{aura_radius},
/*cost_mp/skl/enh (reduced for toggle)*/ 0,0, {mp},{mp_skl},0,
/*cost_%hp/skl/%mp/skl*/               0.00,0.00,0.00,0.00,
/*havoc/skl energy/skl*/               0,0,0.00,0.00,
/*exp/enh jp/enh item/count/skl*/      0,0,0,0, 0,0,0,
/*need: 8 zeros*/                      0,0,0,0,0,0,0,0,
/*vf*/                                 '0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','1',
/*delay cast/skl/enh common*/          0.00,0.00,0.00, 0.00,
/*delay cooltime/skl/enh group*/       1.00,0.00,0.00, 0,
/*uf: self party guild neutral purple enemy*/ '1','1','1','0','0','0',
/*tf: avatar summon, NOT monster*/     '1','1','0',
/*lvup target eff enchant*/            0,4,{effect_type},0,
/*state id/lvl/skl/enh*/              {state_id},0,{state_lvl_skl},0.00,
/*state sec/lvl/enh*/                 {state_sec},0.00,0.00,
/*prob inc*/                           100,0,
/*hit_bonus /enh pct*/                 0,0,0,
/*hate: mod*/                          1.00,0,0.00,0.00,
/*crit: zeros*/                        0,0,
/*var1 = tick interval, rest unused*/  {tick_interval},0.0000,0.0000,0.0000,0.0000,0.0000,
/*var7-20*/                            0.0000,0.0000,0.0000,0.0000, 0.0000,0.0000,0.0000,0.0000,
                                       0.0000,0.0000,0.0000,0.0000, 0.0000,0.0000,
/*icon file proj spd accel*/           {icon_id},'{icon_file}',0,0.00,0.00
);
```

---

### PRESET — Reactive Passive (10000+ on-event triggers)
```sql
VALUES (
/*id text desc tooltip valid elem*/   {id},50{id},0,40{id}, 1,'0',
/*pasv phys harm need corp togg grp*/ '0','0','{harm}','0','0','0',0,
/*cast_type lvl range valid*/          '0','0', 0,0,
/*all costs: zeros*/                   0,0,0,0,0, 0.00,0.00,0.00,0.00, 0,0,0.00,0.00, 0,0,0,0, 0,0,0,
/*need: 8 zeros*/                      0,0,0,0,0,0,0,0,
/*vf*/                                 '0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','0','1',
/*delay cast/skl/enh*/                 0.00,0.00,0.00,
/*delay common*/                       1.50,
/*delay cooltime/skl/enh group*/       0.00,0.00,0.00, {cool_time_group_id},
/*uf*/                                 {uf_self},'{uf_party}','0','0','0',{uf_enemy},
/*tf: avatar summon monster*/          '1','1','1',
/*lvup target eff enchant*/            0,41,{effect_type},0,
/*state id/lvl/skl/enh*/              {state_id},0,{state_lvl_skl},0.00,
/*state sec/lvl/enh*/                 {state_sec},0.00,0.00,
/*prob_on_hit=100 (proc handled by var1)*/ 100,0,
/*hit_bonus /enh pct*/                 0,0,0,
/*hate: 1.00*/                         1.00,0,0.00,0.00,
/*crit: zeros*/                        0,0,
/*var1=proc%, var2=/skl, var3=/enh, rest=0*/
                                       {proc_chance},0.0000,0.0000,
/*var4-20*/                            0.0000,0.0000,0.0000, 0.0000,0.0000,0.0000,0.0000,
                                       0.0000,0.0000,0.0000,0.0000, 0.0000,0.0000,0.0000,0.0000,
                                       0.0000,0.0000,
/*icon file proj spd accel*/           {icon_id},'{icon_file}',0,0.00,0.00
);
```

**Note**: For type 10063 (CDR on attack), `state_id=0` and `var1` = seconds reduced per hit (not proc%).

---

## 6. Common Overrides Cheat Sheet

| Situation | Columns to change |
|-----------|------------------|
| AoE around target | `target=2`, `valid_range={radius}` |
| Ground-targeted AoE | `target=3`, `is_need_target='0'` |
| Multi-hit (type 232/263) | `var7=hit_count`, `var8=hit_count_per_skl`, `var9=delay_secs` |
| Life drain (type 235) | `state_id=0`, `var7=hp_ratio`, `var10=mp_ratio` |
| Weapon-restricted active | Set `vf_{weapon}='1'`, `vf_is_not_need_weapon='0'` |
| Physical damage component | `var7-var12` = P.Atk formula (type 32021) |
| Chain bounce (type 32062) | `var7=chain_count`, `var8=decay_mult`, `var9=jump_radius` |
| Interruptable cast | `casting_type='1'`, `casting_level='1'` |
| Projectile | `is_projectile=1`, `projectile_speed={n}`, `projectile_acceleration={n}` |
| State required to cast | `need_state_id={id}`, `need_state_level={n}`, `need_state_exhaust=1` |
| HP cost instead of MP | Use `cost_hp` / `cost_hp_per_skl` instead; set `cost_mp=0` |

---

## 7. hate_mod Quick Guide

| Skill type | Suggested `hate_mod` |
|-----------|---------------------|
| Weak nuke / utility | `1.00` |
| Standard nuke | `1.20` |
| Strong nuke / AoE | `1.30`–`1.50` |
| Heal | `1.50` (heals generate aggro) |
| Absorb shield | `1.20` |
| Pure buff (no damage) | `1.00` |
| Passive | `1.00` |

---

## 8. State Columns Quick Reference

| Column | Role | Typical value |
|--------|------|--------------|
| `state_id` | State to apply | ID from StateBase table, or `0` for none |
| `state_level_base` | Base level of state | `0` |
| `state_level_per_skl` | Level scaling | `1.00` (1 level per SLV) |
| `state_level_per_enhance` | Level per enhance | `0.00` |
| `state_second` | Duration (seconds) | e.g. `8.00` |
| `state_second_per_level` | Duration per SLV | e.g. `1.00` |
| `state_second_per_enhance` | Duration per enhance | `0.00` |
| `probability_on_hit` | Chance % to apply | `100` for buffs, `<100` for debuffs |
| `probability_inc_by_slv` | Chance gain per SLV | `0`–`5` |

> Full reference: [10_Skill_States_and_Enhancements.md](10_Skill_States_and_Enhancements.md)

---

## 9. Damage Formula (types 231 family)

```
nDamage = M.Atk × (var1 + var2×SLV + var3×Enhance)
        + (var4  + var5×SLV + var6×Enhance)
```

| var | Role |
|-----|------|
| var1 | M.Atk multiplier base (`1.0 = 100% M.Atk`) |
| var2 | M.Atk multiplier per SLV |
| var3 | M.Atk multiplier per enhance |
| var4 | Flat damage base (raw HP units) |
| var5 | Flat damage per SLV |
| var6 | Flat damage per enhance |

Applies identically to: 231, 232, 235, 261, 262, 263, 32021 (magic component), 32032, 32062, 32202 (var1–var6 portion), and EF_ADD_HP 501 (as heal amount).

---

*This document is the canonical source for SQL generation. All effect type docs reference it instead of repeating these patterns.*
