# Source Code Investigation Template

> Use this template when analyzing C++ source code to document a new effect_type or game mechanic. Save the completed investigation as a reference document before writing final documentation.

---

## Investigation Metadata

```
Date Investigated: [YYYY-MM-DD]
Investigated by: [AI Agent / User]
Effect Type ID: [e.g., 4, 5, 10]
Effect Type Name: [e.g., EF_PARAMETER_AMP, EF_INSTANT_DAMAGE]
Game Version: Rappelz [Version if known]
```

---

## 1. Source Code Localization

### Primary Source Files
| File Path | Purpose | Status |
|-----------|---------|--------|
| `D:\Rappelz_source\program\server\Rappelz-Game-Server\...` | [Handler function location] | ✅ Found / ❌ Not Found |
| `D:\Rappelz_source\program\server\Rappelz-Game-Server\...` | [Bitmask definitions] | ✅ Found / ❌ Not Found |
| `D:\Rappelz_source\program\server\Rappelz-Game-Server\...` | [Example usage in skill] | ✅ Found / ❌ Not Found |

### Key Source Locations Found:
```cpp
// File: [filename.cpp / .h]
// Line: [line number range]
// Function: [function name]
```

---

## 2. Handler Function Analysis

### Function Signature
```cpp
[Exact C++ function signature]
[Return type, parameters, etc.]
```

### Function Purpose
[One-paragraph explanation of what this function does]

### Parameter Processing

#### Investigated Variables
List all variables that the function actually reads/processes:

| Variable | Source (var#) | Type | Read As | Default | Notes |
|----------|---|---|---|---|---|
| [var name] | var1 | bitmask | `GetVar(0)` | 0 | Used for stat selection |
| [var name] | var2 | float | `GetVar(1)` | 0.0 | Base percentage modifier |
| ... | ... | ... | ... | ... | ... |

#### Unprocessed Variables
```
Variables var19 and var20:
- Status: UNUSED
- Safe default: 0 or 0.0000
- Rationale: Function does not call GetVar(18) or GetVar(19)
```

---

## 3. Variable Interpretation Details

### Bitmask Analysis (if applicable)

#### Standard Bitmask (ampParameter):
```cpp
// From source code, extracted bitmask values:
#define FLAG_STR        1
#define FLAG_VIT        2
#define FLAG_AGI        4
...
```

**Interpretation**:
- Bitmask type: Standard bit flags
- Combined via: Bitwise OR (`|`)
- Example: `var1 = 1 | 2` means "STR AND VIT"
- Reference: [Standard Bitmasks](02_Standard_Bitmasks.md)

#### Extended Bitmask (ampParameter2):
```cpp
// From source code, if applicable:
#define FLAG_ET_FIRE_RESIST    2
#define FLAG_ET_P_DEF_IGNORE   1024
...
```

**Interpretation**:
- Used by: [which var slots] (e.g., var7–9, var10–12)
- Combined via: Bitwise OR
- Reference: [Extended Bitmasks](03_Extended_Bitmasks.md)

### Float Interpretation

#### Percentage / Modifier Logic
```cpp
// From source code:
float baseValue = GetVar(1);
float levelMultiplier = GetVar(2);
float total = baseValue + (levelMultiplier * skillLevel);
```

**Interpretation**:
- `1.0` = 100% (100% of weapon damage, 100% modifier, etc.)
- `0.5` = 50% (50% damage, 50% stat increase, etc.)
- `0.01` = 1% (per skill level increment)
- Example: If `var2 = 0.10` and skill is level 5, bonus = 0.10 * 5 = 0.50 (50%)

### Enum / Flag Interpretation

```cpp
// From source code:
enum DamageType {
    PHYSICAL = 0,
    MAGICAL = 1,
    HYBRID = 2
};

DamageType dmgType = (DamageType)GetVar(0);
```

**Interpretation**:
- var1 must be: 0, 1, or 2
- Invalid values: If not specified, behavior is [undefined/default to PHYSICAL/etc.]
- Range: 0–2

---

## 4. Validation & Range Checking

### Discovered Range Constraints

```cpp
// From source code analysis:
if (var1 < 0 || var1 > 2147483648) var1 = 0;  // Example range check
if (var2 < 0.0) var2 = 0.0;                   // Floor constraint
if (var3 > 10.0) var3 = 10.0;                 // Ceiling constraint
```

### Documented Constraints
| Variable | Min | Max | Invalid Behavior | Safe Default |
|----------|-----|-----|---|---|
| var1 | [0 or int.min] | [2147483647 or int.max] | [Error/Default] | [0 or safe value] |
| var2 | [0.0 or float.min] | [max float or specified] | [Clamp/Ignore] | [0.0 or 0.0000] |
| ... | ... | ... | ... | ... |

---

## 5. Side Effects & State Changes

### Does this effect trigger additional mechanics?

- [ ] State Application (`state_id`)
- [ ] Cooldown Triggers (`delay_cooltime`)
- [ ] Damage Calculation Hooks
- [ ] Buff Duration Updates
- [ ] Threat/Aggro Changes
- [ ] Other: [specify]

### Details on Each:
```
[If state application is triggered]
- Source: [function call, line number]
- State applied from: [state_id column, var field, or hardcoded value]
- Duration: [seconds, from state_second, or calculated]

[If cooldown triggers]
- Cooldown group: [cool_time_group_id]
- Duration: [from delay_cooltime, var field, or formula]
```

---

## 6. Reverse-Engineered Examples from Database

### Existing Skill 1
```sql
SELECT * FROM SkillResource WHERE id = [example_id] AND effect_type = [type];

-- Results:
-- var1: [value]    -- Explanation: [stat bitmask]
-- var2: [value]    -- Explanation: [base buff]
-- var3: [value]    -- Explanation: [level scaling]
-- ... [all vars shown] ...
```

**In-Game Effect**: [What does this skill do for players?]

**Analysis**: [How do the var values produce this effect?]

### Existing Skill 2
[Same structure as above]

---

## 7. Mandatory Static Columns (for this effect_type)

```sql
-- Columns that DO NOT change for skills using this effect_type:
[is_valid]:           [1 or '0']
[is_passive]:         ['0' or '1']
[elemental]:          ['0' or specific value]
[is_physical_act]:    ['0' or '1']
[is_harmful]:         ['0' or '1']
[is_need_target]:     ['0' or '1']
[is_toggle]:          ['0' or '1']
[effect_type]:        [ID number]
[target]:             [value] -- Explanation: [why this target value]
[cost_hp]:            [0 or constraint]
[cost_mp]:            [0 or constraint]
... [all mandatory columns listed]
```

### Rationale
[Explain why these columns are fixed for this effect_type, based on source code logic]

---

## 8. Practical Constraints & Edge Cases

### Known Limitations
- [ ] Maximum number of stat slots: [e.g., 6]
- [ ] Cannot combine physical + magical: [True/False]
- [ ] Incompatible with certain states: [List]
- [ ] Weapon restrictions apply: [True/False]
- [ ] Other: [specify]

### Edge Cases Discovered
```
Case 1: What happens if var1 = 0 (no bitmask)?
Response: [Does it error, apply nothing, or use default stat?]

Case 2: What happens if var2 and var3 are both 0?
Response: [Is the buff 0, or does it apply 1.0 as default?]

Case 3: [Other edge case]
Response: [Behavior]
```

---

## 9. Contradictions or Ambiguities Found

### Unresolved Questions
- [ ] Question 1: [Description]
  - Source: [Function/file where contradiction found]
  - Possible explanation: [A or B?]
  - Resolution: [Pending user clarification / Assumed A]

- [ ] Question 2: [Description]
  - Pending: [User to review source or test in-game]

---

## 10. Summary & Recommendations

### Effect Type Overview
**ID**: [Number]
**Name**: [Name]
**Category**: [Passive Buff / Active Damage / State Effect / Other]
**Var Count**: [e.g., Uses var1–18, ignores var19–20]
**Complexity**: [Low / Medium / High]

### Documentation Recommended File Name
```
12_Effect_Type_[EF_NAME].md
OR
13_Effect_Type_[EF_NAME].md
(Use next available number from 12–20 range)
```

### Pre-Draft Checklist
Before writing final documentation, confirm:
- [ ] All var fields analyzed
- [ ] Bitmask tables confirmed (link to existing or note as new)
- [ ] Mandatory columns identified
- [ ] At least 2 existing examples reverse-engineered
- [ ] Safe default values determined
- [ ] Edge cases documented
- [ ] Source file paths verified and functional

### Next Steps
1. User reviews this investigation
2. Proceed to write: [12_Effect_Type_[NAME].md](AGENTS.md#step-5-create-documentation-file)
3. Include one brand-new example INSERT statement
4. Link to this investigation as reference in final documentation

---

## Appendix: Raw Source Code Excerpts

### Full Function Code
```cpp
[Paste full function code from source]
```

### Related Bitmask Definitions
```cpp
[Paste all #define FLAG_* relevant to this effect]
```

### Database Schema Reference
```sql
-- From SkillResource table:
-- Columns relevant to this effect_type
[id] INT,
[effect_type] INT,
[var1] FLOAT, [var2] FLOAT, ... [var20] FLOAT,
[state_id] INT,
[is_passive] CHAR(1),
...
```

---

## Investigation Sign-Off

```
Investigation Status: COMPLETE / IN PROGRESS / BLOCKED
Completed by: [Name]
Date: [YYYY-MM-DD]
Ready for Documentation: YES / NO
Blockers (if any): [List]
```

