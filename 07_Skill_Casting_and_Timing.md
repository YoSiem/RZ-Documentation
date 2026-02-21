# Skill Database: Casting and Timing

This document details the database variables related to how quickly a skill casts, its cooldown periods, and any projectile or range mechanics tied to the execution of the skill.

### Cooldowns and Delays
All temporal values in the Skill Database are handled as floats (e.g., `5.00` = 5 seconds) before being parsed into AR_TIMEs by the game server.

* **`delay_cast`**
  The base, fixed casting time required to channel the ability (e.g., `1.50` seconds). Does not scale with level unless `per_skl` is strictly defined.
* **`delay_cast_per_skl`**
  Modifies the casting time per skill level. E.g., `-0.05` would reduce the cast time by 0.05 seconds for every point leveled.
* **`delay_cast_mode_per_enhance`**
  Modifies the cast time specifically when the player uses Skill Enhancement cards (via Alt+Y). Represented by a multiplier. For instance: `-0.01` with a +5 card makes the skill cast 5% faster.
  *(Formula mapped mapped in `SkillBase::GetCastDelayAsFloat`)*.

* **`delay_cooltime`**
  The base cooldown of the skill. Once cast, the player must wait this many seconds (e.g., `30.00`) before it lights up again.
* **`delay_cooltime_per_skl`**
  Reduces or adds to the base cooldown depending on standard skill levels.
* **`delay_cooltime_mode_per_enhance`**
  Reduces the cooltime via Enhancement cards. Similar to the cast time card modifier.
* **`delay_common`**
  Often referred to as the GCD (Global Cooldown). Typically floats around `1.00` second for all standard abilities so they can't be spammed simultaneously.
* **`cool_time_group_id`**
  If two entirely different skills share this `cool_time_group_id`, then casting one automatically locks out the other until the highest cooldown between the two recovers. Commonly used for potion sickness or racial ultimates.

### Ranged and Projectile Logic
* **`casting_type`** & **`casting_level`**
  A structural logic hook regarding whether an interrupt occurs (e.g., if you take damage while casting this spell, does it fail?). 
  * `0` = Uninterruptable.
  * `1` = High chance of interruption depending on `casting_level` (0.2s cast delays vs hard interrupts).
* **`is_projectile`**
  Determines if casting spawns a physical projectile flying through the air toward the target (e.g., an arrow or fireball). If `1`, damage calculates only *upon impact*. If `0`, damage applies instantly upon the cast finishing (Hit-scan).
* **`projectile_speed`**
  Speed of the spawned projectile (e.g., `12.00`). Faster projectiles track and hit targets sooner.
* **`projectile_acceleration`**
  Handles whether the projectile speeds up towards the end of its flight path.
