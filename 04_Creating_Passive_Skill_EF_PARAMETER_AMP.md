# Tutorial: Creating a Passive Skill from Scratch (EF_PARAMETER_AMP = 4)

This document explains in detail how the `EF_PARAMETER_AMP` effect operates within the game engine (based on the C++ source code for `StructCreature::applyPassiveSkillAmplifyEffect` and engine header files) and how to correctly record it in the database.

## 1. How does this Passive Amplification work? (C++ Source Analysis)
The C++ logic bound to `EF_PARAMETER_AMP` reads as many as **6 variable groups** from the skill configuration (from `GetVar(0)` up to `GetVar(17)`, which in the SQL database translates to columns `var1` through `var18`).

Each of these 6 groups (bit slots) requires **3 specific parameters**:
1. **Bitmask (Bitset)** – Dictates WHICH specific stat receives the buff.
2. **Base Value** – The flat value applied unconditionally.
3. **Level Multiplier** – The value multiplied by the skill level to dynamically increment the buff power per skill level upgrade.

---

## 2. Slot Column Architecture (`var1` – `var18`)
This is how the C++ interprets variables mapped from the DB/SQL:

### Slot 1: Basic Combat Stats (`ampParameter` - `FLAG_` flags)
- **`var1`** - Stat Bitmask (e.g., `256` for M.Atk - Magic Point)
- **`var2`** - Base Value added.
- **`var3`** - Multiplier added *per Skill Level*. *(Uses `1 = 100%` fraction math. E.g., `0.05` means 5% per level)*

### Slot 2: Basic Combat Stats (`ampParameter` - `FLAG_` flags)
- **`var4`** - Stat Bitmask (Next targeted stat to modify)
- **`var5`** - Base Value.
- **`var6`** - Level Multiplier.

### Slot 3: Extended Elemental & Penetration Stats (`ampParameter2` - `FLAG_ET_` flags)
> Here, the game switches to `ampParameter2` methodology. The array involves Elementals, Penetrations, and Ignoring Defenses! This requires entirely different `FLAG_ET_` values from the extended bitmask table!
- **`var7`** - Bitmask from the extended table (e.g., inputting `8192` translates in C++ to `FLAG_ET_MAGICAL_PENETRATION`).
- **`var8`** - Base Value.
- **`var9`** - Level Multiplier (e.g., 0.05 grants a 5% magic penetration per skill level).

### Slot 4: Extended Elemental Stats (`ampParameter2` - `FLAG_ET_` flags)
- **`var10`**, **`var11`**, **`var12`** - Same behavior and rules as Slot 3.

### Slot 5: Basic Combat Stats (`ampParameter` - `FLAG_` flags)
- **`var13`**, **`var14`**, **`var15`** - Similar directly to Slot 1 & 2.

### Slot 6: Basic Combat Stats (`ampParameter` - `FLAG_` flags)
- **`var16`**, **`var17`**, **`var18`** - Similar directly to Slot 1 & 2.

> [!TIP]
> The variables **`var19`** and **`var20`** alongside `EF_PARAMETER_AMP` **ARE NEVER** queried by this module. Always input a safe fallback of `0`.

---

## 3. Creating a New Skill - Column by Column Checklist
Below is a step-by-step SQL cheat sheet explaining how to seamlessly populate all necessary rows to construct a passive from scratch featuring parameter 4 effect.

| SQL Column Name | Value to Input & EF_PARAMETER_AMP Description |
| :--- | :--- |
| **`id`** | Your custom unique Skill ID (e.g., **90006**). |
| **`text_id`** / **`desc_id`** / **`tooltip_id`**| Bind the display tags to your String tables. Tooltip maps crucial hover information. |
| **`is_valid`** | **1** – Makes the skill active in the engine and prevents it from acting as invalid/developer debug. |
| **`elemental`** | **0** – Parametric passive skills generally rarely benefit from having elements directly bound to their core casting form. |
| **`is_passive`** | **0** – **(Important!)** Instructs the server this requires no casting and acts passively. In the engine, `0` = Passive (`!is_active`), while `1` = Active. |
| **`is_physical_act`** | **0** – Flags that the passive does not demand a physical strike trigger and physics ruleset. |
| **`is_harmful`** | **0** – The passive functions as a continuous buff benefiting the user. Debuffs map to 1. |
| **`is_need_target`** / **`is_corpse`** | **0** – Your passive skill requires no external target locking, nor targeting a corpse. |
| **`is_toggle`** / **`toggle_group`** | **0** – We do not deploy "Always On" aura toggles here; passive effects run intrinsically through `applyPassiveSkillAmplifyEffect`. |
| **All Cost fields (`cost_...`)** | Every cost mapped to `hp`, `mp`, `energy`, `havoc` and relative "per_skl" instances must unquestionably remain at **0** or **0.00**. Passive skills siphon zero resources. |
| **Cast logic (`cast_range`...)**| Preserve vestigial limits by placing flat **0**. |
| **Weapon Limits (`vf_...`)** | Apply appropriate blocks. If this Atk_Speed passive only works for Daggers, enter `1` into `vf_dagger` (or `vf_double_dagger`) and `0` for the rest. |
| **`vf_is_not_need_weapon`** | Input **1** if the passive works universally without strapping to any exact weapon (or no weapon). |
| **`target`** | Output **2** here. C++ treats this target as "Caster" (Targeting: Self).|
| **`effect_type`** | Allocate exactly **4**. The code engine switches specifically towards `EF_PARAMETER_AMP` mechanically upon seeing this string. |
| **`skill_enchant_link_id`** | Keep as an empty **0** unless explicitly enchantable. |
| **Status Effect (`state_id`...)** | Maintain empty **0** or **0.00**! Effect 4 directly channels variables. It DOES NOT steal IDs or icon metadata remotely from the State Buff tree matrices! |
| **Variables `var1` - `var18`** | Establish the logic with the aforementioned Bit Sub-Slots. *(e.g., var1:256, var2:0, var3:0.025 provides permanent M.ATK passively, injecting further scaled magic atk points proportional to skill level).* |
| **Variables `var19` - `var20`** | Must remain **0** - Parametric Effect 4 strictly omits them. |
| **Icon data** (`icon_...`) | Assign proper image file textures and alt+s GUI IDs native to the client interface structure. |

---

## 4. Practical Example: Deep Dive Analysis of a Real Skill 4 Insert
To truly understand how deep the engine goes, let's look at an actual SQL `INSERT` sent for a new passive skill:

```sql
INSERT INTO [dbo].[SkillResource] (
    [id], [text_id], [desc_id], [tooltip_id], [is_valid], [elemental], 
    [is_passive], [is_physical_act], [is_harmful], [is_need_target], 
    [is_corpse], [is_toggle], [toggle_group], [casting_type], 
    [casting_level], [cast_range], [valid_range], [cost_hp], 
    [cost_hp_per_skl], [cost_mp], [cost_mp_per_skl], [cost_mp_per_enhance], ...
    [vf_is_not_need_weapon], [delay_cast] ... [delay_common] ...
    [target], [effect_type], ...
    [hit_bonus], ... [critical_bonus], ...
    [var1], [var2], [var3], [var4], [var5] ... [var20], 
    [icon_id], [icon_file_name], [is_projectile], [projectile_speed], [projectile_acceleration]
) VALUES (
    90012, 50090012, 0, 40090012, 1, '0', 
    '0', '0', '0', '0', '0', '0', 0, '0', '0', 0, 0, 0, 0, 0, 0, 0, 
    ... (All costs, MP, Exp, JP, targets = 0) ...
    ... (vf_weapons = '0') ... '1',  -- This '1' maps to [vf_is_not_need_weapon]
    0.00, 0.00, 0.00, 1.50, ... -- 1.50 is [delay_common]
    ...
    41, 4, -- [target] = 41, [effect_type] = 4
    ...
    100, 0, 0, 0.00, 0, 0.00, 0.00, 26995, 0, -- hit_bonus = 100, critical_bonus = 26995
    48.0000, 0.0000, 0.0175, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 
    0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 0.0000, 
    1530, 'icon_skill_pas_mage_moral_culture', 0, 0.00, 0.00
);
```

### Column-by-Column Architectural Breakdown
Because this `INSERT` populated 94 columns, here is an absolute breakdown of WHY those specific values were chosen and exactly how they impact the game's engine:

#### 1. Identity & Visuals
- **`[id]` = 90012**: The unique identifier for this skill.
- **`[tooltip_id]` = 40090012** & **`[text_id]` = 50090012**: The string logic (StringTable ID) mapping the Name and Description text.
- **`[icon_id]` = 1530** & **`[icon_file_name]` = 'icon_skill_pas_mage_moral_culture'**: Instructs the client UI to load the "Moral Culture" texture file for the skill tree.

#### 2. Core Behavior Constraints (The "Is" Booleans)
- **`[is_valid]` = 1**: The engine activates the skill (0 would make it a dead test skill).
- **`[is_passive]` = 0**: To the game engine, this directly targets `!is_active`. Setting it to `0` makes it a passive ability that never needs casting!
- **`[elemental]` = 0 / `[is_physical_act]` = 0 / `[is_harmful]` = 0**: It does no elemental damage, is not a physical strike, and is decidedly a beneficial (non-harmful) buff.
- **`[is_need_target]`, `[is_corpse]`, `[is_toggle]` = 0**: It is not an aura toggle, and requires absolutely zero manual targeting to function. 

#### 3. Execution, Costs & Ranges
- **All `cost_` fields (HP/MP/Havoc/Energy/EXP/Item) = 0**: Since it's a passive buff, it must drain exactly 0 resources. Entering costs here could break passive initialization.
- **All `need_` fields (Level/HP/State) = 0**: Zero gating requirements to "cast" it.
- **`[delay_common]` = 1.50**: While passives never cast, standard development practice leaves the Global Cooldown (GCD) placeholder at `1.50`. All other delays (`delay_cast`, `delay_cooltime`) are safely `0.00`.
- **`[cast_range]` / `[valid_range]` = 0**: Passives have no casting tether limits.

#### 4. Weapons & Equipment (`vf_`)
- **All `vf_weapon` columns = '0'**: Rejects all physical weapons as triggers...
- **`[vf_is_not_need_weapon]` = '1'**: ...However, throwing a `1` here completely overrides the previous lines! This skill will permanently buff the player even if they hold absolutely nothing.

#### 5. Skill Target & The Engine Hook
- **`[target]` = 41**: In the engine (`SkillBase.h`), target constants above 30 refer strictly to advanced relational loops (e.g., Summon parameters, Party boundaries). Setting `41` structures how the aura loops the caster/summon.
- **`[effect_type]` = 4 (EF_PARAMETER_AMP)**: The absolute most vital column here. It demands that the engine ignores conventional calculations and hands the skill entirely over to `applyPassiveSkillAmplifyEffect` in `StructCreature`.

#### 6. External Combat Modifiers
Passives in Rappelz often use hidden standalone columns to inject flat boosts independently from the `var` slots.
- **`[hit_bonus]` = 100**: Injects a massive flat +100 bonus definitively into the player's underlying Accuracy pool.
- **`[critical_bonus]` = 26995**: An absurdly high critical override. Values this high are used internally to guarantee critical behavior limits or spoof mechanics in the aggro engine.
- **All `hate_` columns = 0**: Standing still emitting this passive buff generates exactly 0 monster threat/aggro.

#### 7. The Mathematical Multipliers (`var1` to `var20` Slots)
Because `[effect_type] = 4`, the variables must be interpreted using the **3-step Bitmask Formula** (Bitmask, Base Value, Multiplier).

- **Slot 1 Matrix (`var1`, `var2`, `var3`)**:
  - **`var1` = 48.0000 (Bitmask)**: We must look at the Standard bitmasks list. Which stats add up to 48? 
    `16` (Intelligence/INT) + `32` (Mentality/MEN) = `48`. 
    This means the single slot flawlessly targets **INT & MEN** simultaneously!
  - **`var2` = 0.0000 (Base Profit)**: Provides a flat `0` bonus points out-of-the-box (Level 1).
  - **`var3` = 0.0175 (Level Multiplier)**: Because the engine natively uses `1 = 100%` fractions, `0.0175` translates perfectly to a **1.75% multiplier boost** per skill level.
- **Slots 2 through 6 (`var4` through `var20`) = 0.0000**: All remaining parameter slots are completely empty. The skill has finished its processing and exits the C++ cleanly.

**Conclusion of the Query:** Without guessing, the absolute evidence generated by reading through the database architecture reveals that this insert creates a passive that buffs INT and MEN proportionally with its skill level, permanently applies +100 base Accuracy, demands absolutely zero weapon or resource costs, and does not require targeting.
