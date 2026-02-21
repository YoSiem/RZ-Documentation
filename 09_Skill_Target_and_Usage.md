# Skill Database: Target and Usage Conditions

This document covers skill targeting mechanics, range limitations, and what faction or combat entities the skill is conditionally allowed to strike.

### Core Targeting Logic
These columns decide if the skill requires an explicitly clicked target or if it triggers automatically.

* **`is_need_target`**
  Does the user need an entity targeted to cast this?
  `1` = Yes, a target is mandatory.
  `0` = No. Great for self-buffs, AoE explosions centered on self, and passive skills.
* **`is_corpse`**
  `1` = Skill can ONLY be used on a deceased player/creature. Most resurrection or corpse-explosion skills require this.
* **`target`**
  This dictates the exact structural entity condition of the spell. (Mapped by `TARGET_` in `SkillBase.h`).
  * `0` = Any Miscellaneous target
  * `1` = Specifically the selected target
  * `2` = Region including current target
  * `3` = Region without target (ground target)
  * `4` = General Area of Effect (AoE)
  * `21` = Party
  * `22` = Guild

### Combat Distance Limitations
* **`cast_range`**
  The maximum distance the caster can stand from the target before the game engine forces their character to run closer.
* **`valid_range`**
  The absolute maximum distance the skill will register as "hitting". If a targeted player runs further away than `valid_range` *while* the caster is mid-cast, the spell fails or misses. For simple buffs or passives, leave both at `0`.

### Faction Logic (`uf_`)
These parameters dictate *who* is considered a valid target when the combat reticle is displayed. They are binary toggles (`1` for allowed, `0` for forbidden).
* **`uf_self`** - Can I cast this on myself? (Mandatory for self-buffs).
* **`uf_party`** - Can I cast this on players within my party group?
* **`uf_guild`** - Can I cast this on players within my guild?
* **`uf_neutral`** - Can I cast this on neutral unflagged entities (like random players)?
* **`uf_purple`** - Can I cast this on players flagged as aggressive/criminal (Purple name)?
* **`uf_enemy`** - Can I cast this on hostile monsters or opposing players in PK/Sieges?

### Entity Distinctions (`tf_`)
Even if the faction allows it, you can restrict skills to specifically hit monsters, pets, or players exclusively.
* **`tf_avatar`**
  `1` = Allowed to hit/target Player Characters (Avatars).
* **`tf_summon`**
  `1` = Allowed to hit/target Player Pets or Summons.
* **`tf_monster`**
  `1` = Allowed to hit/target PvE Monsters and NPCs.
