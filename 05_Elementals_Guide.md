# Detailed Elementals Guide

In the game engine, elements (also referred to as Elementals) govern how damage formulas compute elemental advantages, resistances, and extra damage properties. 

## Engine Element Declarations

The core elements are defined within `GameType.h` under the `Elemental::Type` namespace. These integer values are what you input into database columns like `elemental` in the `SkillBase` table or cross-reference in bitmasks.

| ID | Element Name | C++ Engine Mapping | Description & Common Usage |
| :--- | :--- | :--- | :--- |
| `0` | **None** | `TYPE_NONE` | Non-elemental / Physical. Represents raw untyped damage. |
| `1` | **Fire** | `TYPE_FIRE` | Fire Element. Common for Magician and Pyromancer classes. |
| `2` | **Water** | `TYPE_WATER` | Water Element. |
| `3` | **Wind** | `TYPE_WIND` | Wind Element. Used often for rapid strikes or evasion-based classes. |
| `4` | **Earth** | `TYPE_EARTH` | Earth Element. Typically associated with high defense and heavy hits. |
| `5` | **Light** | `TYPE_LIGHT` | Light Element. Used by Priest classes and holy abilities. |
| `6` | **Dark** | `TYPE_DARK` | Dark Element. Exploited by Warlocks and corrupt mechanics. |

## How Elementals Affect Skills

### 1. `elemental` Column in `SkillBase` 
When an active damage dealing skill is cast, the engine needs to know what element powers the attack. Placing a digit between 1 and 6 sets the skill's absolute element. If your target is weak to Fire and you cast a skill with `elemental = 1`, damage gets amplified significantly via the engine's internal affinity multipliers. 

*Note: For purely passive skills or healing properties, leave this at `0` to keep it unaffected by elemental resistances.*

### 2. Targeting Specific Elements (`target` column properties)
Occasionally, skills or buffs operate *only* on a specific creature trait. The `target` column can be set to distinct magic IDs from `201` to `207`:
- `202` = Target only Fire Elementals
- `203` = Target only Water Elementals
- `...` and so on.

### 3. Extended Bitmasks (`FLAG_ET_`) 
Besides just throwing an element onto a skill cast, elementals control advanced secondary stats managed by the **Extended Bitmasks** (see [03_Extended_Bitmasks.md](03_Extended_Bitmasks.md)). 
By using `EF_PARAMETER_AMP` or similar buffs, you can target specific elemental mechanics. Here are common applications:

- **Elemental Resistance:** Add `FLAG_ET_FIRE_RESIST` (Bitmask `2`) to a passive to reduce incoming Fire-type damage. 
- **Elemental Damage Overrides:** Adding `FLAG_ET_FIRE_DAMAGE` (Bitmask `32768`) gives your physical/auto-attacks the Fire element temporarily.
- **Additional Elemental Damage:** Adding `FLAG_ET_FIRE_ADDITIONAL_DAMAGE` (Bitmask `4194304`) provides an extra burst of Fire damage scaling independently alongside your base strike.

## Combat Affinity (Rock-Paper-Scissors)
When building custom dungeons or tuning PvP combat, keep in mind how elements interact mechanically:
- **Fire** burns **Wind**
- **Wind** erodes **Earth**
- **Earth** absorbs **Water**
- **Water** douses **Fire**
- **Light** and **Dark** oppose each other (dealing high damage to each other).
