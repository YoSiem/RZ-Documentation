# AI Documentation Guidelines for Rappelz Skill Database

> **Purpose**: This document instructs AI agents on how to create consistent, non-redundant documentation for Rappelz skill mechanics. All AI agents working on this documentation MUST follow these guidelines.

---

## 1. Core Principles

### 1.1 Non-Redundancy and Cross-Linking
- **NEVER** repeat the same explanation twice in different MD files.
- Instead, **ALWAYS create a markdown link** to the relevant section in another file.
- Example: If `02_Standard_Bitmasks.md` explains P.Def bitmask, and you need to reference it in a new skill doc, use:
  ```markdown
  See [P.Def bitmask in Standard Bitmasks](02_Standard_Bitmasks.md#pdef)
  ```

### 1.2 Source Code Investigation
When tasked with analyzing a new `effect_type` or game mechanic:

**Primary Source Location**: `D:\Rappelz_source\program\server\Rappelz-Game-Server`

**Standard Investigation Procedure**:
1. Search for the effect handler in C++ source files (e.g., `EF_PARAMETER_AMP`, `EF_INSTANT_DAMAGE`)
2. Examine the function signature and variable interpretation
3. Document:
   - What each `var1`–`var20` parameter does
   - What value ranges are valid
   - Any bitmasks or special encoding schemes
   - Default/safe fallback values
4. Cross-reference with existing database records matching that `effect_type`

### 1.3 Documentation Structure Per Effect Type

Each new `effect_type` documentation MUST include:

| Section | Content |
|---------|---------|
| **1. Overview** | Brief description of what this effect does in-game |
| **2. C++ Source Analysis** | Reference to the source file and function name |
| **3. Variable Mapping** | Exact explanation of `var1`–`var20` |
| **4. Bitmask/Encoding Reference** | Link to or embed any special bitmask tables (avoid duplication) |
| **5. Mandatory Static Columns** | Unchanging SQL column defaults for this effect |
| **6. Practical Constraints** | Value ranges, forbidden combinations, etc. |
| **7. Example INSERT Statement** | A fully-formed, working example |

---

## 2. Investigation Protocol: Analyzing a New Effect Type

### 2.1 Step 1: Locate Source Code
```bash
# Search in the Rappelz source for the effect handler
# Example: Finding EF_INSTANT_DAMAGE
find "D:\Rappelz_source\program\server\Rappelz-Game-Server" -type f -name "*.h" -o -name "*.cpp" \
  | xargs grep -l "EF_INSTANT_DAMAGE\|effect_type.*[0-9]"
```

### 2.2 Step 2: Read and Analyze the Handler Function
- Locate the function that processes this effect (e.g., `applyPassiveSkillAmplifyEffect`)
- Document:
  - **Parameter Count**: How many `var` fields are actually used? (e.g., `var1–var18` for EF_PARAMETER_AMP)
  - **Variable Semantics**: What does each var represent?
  - **Type Casting**: Are they treated as int, float, bitmask, etc.?
  - **Validation**: Are there range checks or sanity guards?

### 2.3 Step 3: Find Existing Examples
```bash
# Search the database schema for existing skills using this effect_type
SELECT id, effect_type, var1, var2, ... FROM SkillResource WHERE effect_type = 4
```

- Reverse-engineer what the existing skills do by analyzing their var values
- Document any patterns (e.g., "All offensive damage skills use var1 as damage scaling")

### 2.4 Step 4: Document with Examples
- Create a new markdown file: `XX_Effect_Type_[NAME].md`
- Include a working example INSERT statement with realistic values
- Add comments explaining each var's role in that specific example

---

## 3. Markdown File Naming and Numbering

**Format**: `XX_[Descriptive_Name].md` where XX is a two-digit number

**Current Numbering Scheme**:
- `00–05`: Foundational architecture and references
- `06–11`: Core column categories and mechanics
- `12–20`: Individual effect_type deep dives (reserved for expansion)
- `Skills/`: Directory for individual skill documentation files

**Naming Examples**:
- ✅ `12_Effect_Type_EF_INSTANT_DAMAGE.md`
- ✅ `13_Effect_Type_EF_SUMMON_DURATION.md`
- ❌ `EF_INSTANT_DAMAGE_guide.md` (inconsistent format)

---

## 4. Linking Between Documents

### 4.1 Internal Links
**Always use relative paths**:
```markdown
# Bad (absolute path)
[See bitmasks](file:///D:/Rappelz_source/02_Standard_Bitmasks.md)

# Good (relative path)
[See bitmasks](02_Standard_Bitmasks.md#section-anchor)
```

### 4.2 Anchor Links
Create anchors for frequently-referenced sections:
```markdown
## Standard Bitmasks {#bitmask-section}

| Stat | Value |
...
```

Then link to it:
```markdown
[See bitmask values](02_Standard_Bitmasks.md#bitmask-section)
```

### 4.3 Avoid Duplication
**Bad Pattern**:
```markdown
# In 04_Creating_Passive_Skill_EF_PARAMETER_AMP.md
| Stat | Value |
| STR | 1 |
| VIT | 2 |

# In 12_Effect_Type_EF_SOME_OTHER_TYPE.md
| Stat | Value |
| STR | 1 |
| VIT | 2 |
```

**Good Pattern**:
```markdown
# In 04_Creating_Passive_Skill_EF_PARAMETER_AMP.md
[See Standard Bitmasks for stat encoding](02_Standard_Bitmasks.md)

# In 12_Effect_Type_EF_SOME_OTHER_TYPE.md
This effect also uses the same bitmask encoding.
[See Standard Bitmasks](02_Standard_Bitmasks.md)
```

---

## 5. Creating Skill Examples

### 5.1 Required Components
Every effect_type example MUST include:

1. **Skill Description** (1–3 sentences)
   ```markdown
   **Flame Burst** - An offensive spell that deals fire damage to a single target.
   ```

2. **Effect Mechanism** (What the vars control)
   ```markdown
   - `var1` = Damage scaling multiplier
   - `var2` = Fire resistance penetration
   ```

3. **Complete INSERT Statement**
   - All 94 columns in exact order
   - Realistic values
   - Proper SQL formatting

4. **Inline Comments** (optional but recommended)
   ```sql
   INSERT INTO [dbo].[SkillResource] (
       [var1], [var2], ...
   ) VALUES (
       2.5,    0.15,  -- var1 = 250% base damage, var2 = 15% fire penetration
       ...
   );
   ```

### 5.2 Example Template
```markdown
### Example: [Skill Name] (ID: [START_ID])
- **Effect**: [Brief description]
- **Key Variables**:
  - `var1` = [explanation]
  - `var2` = [explanation]
  - ...

\`\`\`sql
INSERT INTO [dbo].[SkillResource] (
    [id], [text_id], [desc_id], [tooltip_id],
    ...columns...
) VALUES (
    [id_value], [text_id_value], ...values...
);
\`\`\`
```

---

## 6. Handling Bitmasks and Encoded Values

### 6.1 When to Embed vs. When to Link
- **Embed** a bitmask table only if it's **unique to that effect_type** or **not documented elsewhere**
- **Link** to an existing bitmask table in `02_Standard_Bitmasks.md` or `03_Extended_Bitmasks.md`

### 6.2 Bitmask Analysis Example
If analyzing a new effect that uses an unknown bitmask:

1. Search source code for the bitmask definition:
   ```
   #define FLAG_ET_FIRE_RESIST 2
   #define FLAG_ET_WATER_RESIST 4
   ...
   ```

2. Extract and organize all values into a table

3. If this bitmask is **reused by multiple effects**, move it to a central file (02 or 03)

4. If it's **unique to one effect**, document it in that effect's file

---

## 7. Quality Checklist Before Finalizing

Before marking a new effect_type documentation as complete:

- [ ] Source code file path documented
- [ ] All `var1`–`var20` explained (or explicitly marked as "unused")
- [ ] Safe default values specified
- [ ] At least one complete example INSERT statement
- [ ] No information duplicated (use links instead)
- [ ] All links are relative paths and functional
- [ ] SQL syntax is valid and tested (if possible)
- [ ] Bitmask tables reference existing docs or are justified as new
- [ ] File follows naming convention `XX_[Name].md`
- [ ] Numbering does not conflict with existing files

---

## 8. AI Agent Responsibilities

When assigned to create or update documentation:

1. **Read this file first** – Understand the non-redundancy principle
2. **Investigate the source** – Use the path `D:\Rappelz_source\program\server\Rappelz-Game-Server`
3. **Cross-check existing docs** – Ensure you're not duplicating explanations
4. **Link appropriately** – Use relative markdown links
5. **Provide working examples** – Validate INSERT syntax
6. **Verify column order** – All 94 columns must be in exact SQL order
7. **Update Table of Contents** – If adding new files, update `00_Table_of_Contents.md`

---

## 9. Update Process for Existing Documentation

If you discover a mistake or improvement opportunity in existing docs:

1. **Do not duplicate** the corrected information
2. **Update the source** (the file where it's originally documented)
3. **Add a note** in dependent files pointing to the corrected section:
   ```markdown
   > **Note**: This was corrected in [Primary Documentation](XX_Name.md#anchor)
   ```
4. **Update related links** if paths have changed

---

## 10. Table of Contents Management

The `00_Table_of_Contents.md` file MUST be updated whenever:
- A new effect_type documentation file is added
- A file is renamed or moved
- Sections are significantly reorganized

**Procedure**:
1. Add new entry in numerical order
2. Include a brief one-line description
3. Ensure link is relative and functional

---

## Questions for Clarification

If an AI agent encounters ambiguity:

1. **Unknown bitmask value?** → Search source code or ask user for clarification
2. **Conflicting info in existing docs?** → Cross-check against source code and prioritize source truth
3. **Missing var documentation?** → Assume safe default (0 or 0.0000) and document as "unused"
4. **New column not in schema?** → Verify against actual SQL schema; do not invent columns

---

## Approved for AI Agent Usage

This document is the **canonical reference** for all documentation generation tasks related to Rappelz skill database.

**Date Created**: 2025-02-21
**Last Updated**: 2025-02-21
**Version**: 1.0
