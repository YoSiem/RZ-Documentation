# Standard Bitmasks - Base and Combat Stats

To easily configure skill parameters, variables can use standard bitmasks mapped to engine variables directly from the `StructMisc.h` file. You can **ADD/SUM** these numbers together if you want to bundle multiple boosts into a single variable property! For instance, `128` (P.Atk) + `256` (M.Atk) = `384` inputted into a `var` slot will grant both P.Atk and M.Atk.

Certain `EF_PARAMETER_AMP` slots (like `var1`, `var4`, `var13`, and `var16`) accept these bitmasks exclusively to boost specific stats.

| Stat Name (Buffed Property) | Database Value to Input | C++ Engine Mapping (Bitset) | 
| :--- | :--- | :--- |
| **STR** (Strength) | `1` | 1 << 0 |
| **VIT** (Vitality) | `2` | 1 << 1 |
| **AGI** (Agility) | `4` | 1 << 2 |
| **DEX** (Dexterity) | `8` | 1 << 3 |
| **INT** (Intelligence) | `16` | 1 << 4 |
| **MEN** (Mentality) | `32` | 1 << 5 |
| **LUK** (Luck) | `64` | 1 << 6 |
| **Attack Point** (P.Atk) | `128` | 1 << 7 |
| **Magic Point** (M.Atk) | `256` | 1 << 8 |
| **Defence** (P.Def) | `512` | 1 << 9 |
| **Magic Defence** (M.Def) | `1024` | 1 << 10 |
| **Attack Speed** | `2048` | 1 << 11 |
| **Cast Speed** | `4096` | 1 << 12 |
| **Move Speed** | `8192` | 1 << 13 |
| **Accuracy** (Physical) | `16384` | 1 << 14 |
| **Magic Accuracy** (Magical)| `32768` | 1 << 15 |
| **Critical Rate** | `65536` | 1 << 16 |
| **Block Rate** (Physical) | `131072` | 1 << 17 |
| **Block Defence** | `262144` | 1 << 18 |
| **Avoid** (Physical Dodge) | `524288` | 1 << 19 |
| **Magic Resistance** (M.Dodge)| `1048576` | 1 << 20 |
| **Max HP** | `2097152` | 1 << 21 |
| **Max MP** | `4194304` | 1 << 22 |
| **Max SP** | `8388608` | 1 << 23 |
| **HP Regen Add** (Flat Regen)| `16777216` | 1 << 24 |
| **MP Regen Add** (Flat Regen)| `33554432` | 1 << 25 |
| **HP Regen Ratio** (% Regen) | `134217728` | 1 << 27 |
| **MP Regen Ratio** (% Regen) | `268435456` | 1 << 28 |
| **Final DMG Increase** (%) | `536870912` | 1 << 29 |
| **Max Weight** | `1073741824` | 1 << 30 |
| **Final DMG Reduction** (%) | `2147483648` | 1 << 31 |
