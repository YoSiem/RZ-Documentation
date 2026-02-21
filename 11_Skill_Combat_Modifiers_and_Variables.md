# Skill Database: Combat Modifiers and Variables

This document outlines the deepest mathematical modifiers inside the game engine, specifically managing aggro generation, hit chances, crits, and the all-important flexible variable arrays.

### Accuracy and Hit Chance
When an attack swings, the engine compares the Caster's Accuracy against the Target's Evasion/Block. Skills can bypass this natively.

* **`probability_on_hit`**
  The absolute percentage chance of landing the attached Status Effect (Buff/Debuff) if the main skill hits. Ranges from 0 to 100.
* **`probability_inc_by_slv`**
  Increases the chance of landing the status effect by this amount for every skill level upgraded.
* **`hit_bonus`**
  A flat numeric bonus added directly to the caster's underlying Accuracy strictly for this spell calculation. Essential for high-damage "Snipe" abilities so they don't easily miss.
* **`hit_bonus_per_enhance`**
  The accuracy bonus granted by having a +X enhancement card equipped.
* **`percentage`**
  Often modifies accuracy scaling calculations depending on level differences (e.g., Caster Level vs Target Level differentials).

### Threat and Aggro (Hate System)
In PvE (Player vs Environment), monsters target the player generating the most "Hate". Healing and attacking build Hate automatically, but these values can be manipulated tightly for Tanking (Knights) or avoiding damage (Assassins/Clerics).

* **`hate_mod`**
  Governs how the engine scales the Hate output.
  * `-1` = A fixed absolute value of Hate is applied, completely ignoring any raw damage the player dealt.
  * `0` = Generates absolutely no Hate whatsoever (useful for invisible buffs).
  * `1` = Hate is generated proportional to the Damage/Heal. The formula multiplies damage by `hate_mod`. So `1.5` creates 150% more threat than normal.
* **`hate_basic`**
  The raw numerical amount of baseline Threat added to the monster's aggro table, regardless of damage.
* **`hate_per_skl`** & **`hate_per_enhance`**
  Increases the generated Threat based on Skill Level and Card Enhancement level respectively.

### Critical Scaling
* **`critical_bonus`**
  A flat numerical bonus added to the caster's Critical Rate strictly during the calculation of this spell.
* **`critical_bonus_per_skl`**
  Critical bonus scaling per skill level point.

### The Flexible Variables (`var1` to `var20`)
These 20 floats act as the core mathematical data for the C++ engine to interpret. How they function relies 100% entirely on the previously defined `effect_type`.

Because there are dozens of different Effect Types (`EF_PHYSICAL_SINGLE_DAMAGE(30001)`, `EF_HEAL(61)`, `EF_PARAMETER_AMP(4)` etc.), the variables transform to mean different things based on the hook.

For example, if `effect_type` is an Area Heal:
* `var1` might be the Base Heal Value.
* `var2` might be Bonus Heal per Player Level.
* `var3` might be the Range Radius of the explosion.

But if `effect_type` is `EF_PARAMETER_AMP(4)`:
* `var1` becomes a [Bitmask](02_Standard_Bitmasks.md) flag (like `256` for M.Atk).
* `var2` becomes the flat stat point boost.
* `var3` becomes the boost multiplier per skill level.
*(Read the specifically compiled skill guides like `04_Creating_Passive_Skill_EF_PARAMETER_AMP` for precise slot layouts!)*
