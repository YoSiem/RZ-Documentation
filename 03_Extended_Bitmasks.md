# Extended Bitmasks - Elements and Advanced Stats

Unlike the base stats, certain skills and parameter slots (like `var7` and `var10` in `EF_PARAMETER_AMP`) utilize an entirely different bitmask architecture mapped to engine variables starting with `FLAG_ET_`.

These bits adjust elements, penetrations, and special damage types.

| Stat Name (Buffed Property) | Database Value to Input | C++ Engine Mapping (Bitset) |
| :--- | :--- | :--- |
| **None Resist** (Attr.less Res)| `1` | 1 << 0 |
| **Fire Resist** (Fire Res) | `2` | 1 << 1 |
| **Water Resist** (Water Res) | `4` | 1 << 2 |
| **Wind Resist** (Wind Res) | `8` | 1 << 3 |
| **Earth Resist** (Earth Res)| `16` | 1 << 4 |
| **Light Resist** (Light Res)| `32` | 1 << 5 |
| **Dark Resist** (Dark Res) | `64` | 1 << 6 |
| **Attack Range** (Bonus Rng)| `256` | 1 << 8 |
| **Perfect Block** | `512` | 1 << 9 |
| **Ignore Physical Defence** (PDef Ig)| `1024` | 1 << 10 |
| **Ignore Magical Defence** (MDef Ig)| `2048` | 1 << 11 |
| **Physical Penetration** (P.Pen) | `4096` | 1 << 12 |
| **Magical Penetration** (M.Pen) | `8192` | 1 << 13 |
| **None Damage** | `16384` | 1 << 14 |
| **Fire Damage** | `32768` | 1 << 15 |
| **Water Damage** | `65536` | 1 << 16 |
| **Wind Damage** | `131072` | 1 << 17 |
| **Earth Damage** | `262144` | 1 << 18 |
| **Light Damage** | `524288` | 1 << 19 |
| **Dark Damage** | `1048576` | 1 << 20 |
| **None Additional Damage** | `2097152` | 1 << 21 |
| **Fire Additional Damage** | `4194304` | 1 << 22 |
| *(Additional damage types...)* | *(...multiples of x2)* | ... |
| **Critical Damage** (Crit Power)| `268435456` | 1 << 28 |
| **HP Regen Stop** (Stops regen) | `536870912` | 1 << 29 |
| **MP Regen Stop** (Stops regen) | `1073741824` | 1 << 30 |

> [!NOTE]
> Values like `FLAG_ET_IGNORE_PHYSICAL_DEFENCE` apply scaling percentages that range between 0 and 100 inside the engine (e.g., inputting `0.1` adds 10% reduction bypass). Keep this in mind when entering values into `var8` or `var9`!
