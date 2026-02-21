# 🤖 AI Quick Start Guide - Rappelz Documentation

> **Read this first!** This document is your entry point. It tells you what to read, in what order, before starting any documentation or skill generation task.

---

## For First-Time Tasks: Read in This Order

### 1️⃣ **This File** (You are here!)
   - **Purpose**: Orientation and task dispatch
   - **Read time**: 5 minutes

### 2️⃣ **[AI-DOCUMENTATION-GUIDELINES.md](AI-DOCUMENTATION-GUIDELINES.md)**
   - **Purpose**: Learn the non-redundancy principle and cross-linking system
   - **Read time**: 10–15 minutes
   - **Why**: Prevents you from writing duplicate explanations across files

### 3️⃣ **[AGENTS.md](AGENTS.md)** - Full version
   - **Purpose**: Specific instructions for:
     - Generating EF_PARAMETER_AMP skills (effect_type = 4)
     - Investigating new effect_types
     - Creating documentation files
   - **Read time**: 20–30 minutes
   - **Why**: Contains the exact schema, formulas, and procedures

### 4️⃣ **Task-Specific Guides** (Choose one):
   - **Creating a Passive Skill?** → Read [04_Creating_Passive_Skill_EF_PARAMETER_AMP.md](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md)
   - **Investigating a New Effect Type?** → Use [SOURCE-CODE-INVESTIGATION-TEMPLATE.md](SOURCE-CODE-INVESTIGATION-TEMPLATE.md)
   - **Need Bitmask Reference?** → See [02_Standard_Bitmasks.md](02_Standard_Bitmasks.md) or [03_Extended_Bitmasks.md](03_Extended_Bitmasks.md)

---

## Task Quick Dispatcher

### ✅ "Create a new passive skill (EF_PARAMETER_AMP)"
**Time to complete**: 5–10 minutes

**Steps**:
1. Read: [AGENTS.md Section 1–2](AGENTS.md#1-the-output-format)
2. Review: [04_Creating_Passive_Skill_EF_PARAMETER_AMP.md](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md) for examples
3. Reference: [02_Standard_Bitmasks.md](02_Standard_Bitmasks.md) for stat IDs
4. Generate: Full SQL INSERT with all 94 columns

**Output**: Single SQL INSERT statement in ````sql``` block

---

### ✅ "Analyze a new effect_type and create documentation"
**Time to complete**: 1–2 hours

**Steps**:
1. Read: [AI-DOCUMENTATION-GUIDELINES.md - Section 2 (Investigation Protocol)](AI-DOCUMENTATION-GUIDELINES.md#2-investigation-protocol-analyzing-a-new-effect-type)
2. Use: [SOURCE-CODE-INVESTIGATION-TEMPLATE.md](SOURCE-CODE-INVESTIGATION-TEMPLATE.md)
3. Search: `D:\Rappelz_source\program\server\Rappelz-Game-Server` for the effect handler
4. Document: Fill the template as you analyze
5. Create: New file `XX_Effect_Type_[NAME].md` following [AGENTS.md - Step 5](AGENTS.md#step-5-create-documentation-file)
6. Example: Create ONE working SQL INSERT example
7. Link: Update [00_Table_of_Contents.md](00_Table_of_Contents.md)

**Output**:
- Completed investigation template
- New documentation file (e.g., `12_Effect_Type_EF_INSTANT_DAMAGE.md`)
- Updated Table of Contents

---

### ✅ "Update or fix existing documentation"
**Time to complete**: 15–30 minutes

**Steps**:
1. Read: [AI-DOCUMENTATION-GUIDELINES.md - Section 9 (Update Process)](AI-DOCUMENTATION-GUIDELINES.md#9-update-process-for-existing-documentation)
2. Find: The file that originally documents the concept
3. Update: Only in that file (don't duplicate elsewhere)
4. Add: Note in dependent files linking to the corrected section
5. Verify: All links still work with relative paths

**Output**: Updated documentation with consistent information across all files

---

### ✅ "Create a skill of a different effect_type (not EF_PARAMETER_AMP)"
**Time to complete**: 10–30 minutes (depends on if documentation exists)

**Steps**:
1. Check: Does documentation exist for this effect_type?
   - **YES** → Go to step 3
   - **NO** → Follow "Analyze a new effect_type" task above
2. Read: The effect_type documentation file (e.g., `12_Effect_Type_[NAME].md`)
3. Review: Example SQL in that file
4. Generate: New skill following the same structure

**Output**: SQL INSERT statement for the new skill

---

## Critical Rules (Always Remember)

### ❌ DON'T:
- Write the same explanation twice in different files
- Use absolute file paths (use relative: `[filename].md`)
- Invent new SQL columns not in the 94-column schema
- Create INSERT statements without all 94 columns in exact order
- Duplicate bitmask tables (link to existing instead)
- Ignore source code validation when investigating

### ✅ DO:
- Link to existing explanations instead of repeating them
- Use relative markdown links: `[Text](filename.md#anchor)`
- Reference source files: `D:\Rappelz_source\program\server\Rappelz-Game-Server`
- Provide working, tested SQL examples
- Check [00_Table_of_Contents.md](00_Table_of_Contents.md) first for existing documentation
- Follow the naming convention: `XX_[Descriptive_Name].md`

---

## File Reference Map

```
RZ-Documentation/
│
├─ 📋 Quick Orientation (Read First)
│  ├─ AI-QUICK-START.md (you are here)
│  ├─ AI-DOCUMENTATION-GUIDELINES.md (core principles)
│  └─ 00_Table_of_Contents.md (master index)
│
├─ 📚 Architecture & References
│  ├─ 01_Skill_Database_Architecture.md
│  ├─ 02_Standard_Bitmasks.md
│  ├─ 03_Extended_Bitmasks.md
│  ├─ 05_Elementals_Guide.md
│  ├─ 06_Skill_Identifiers_and_Core_Flags.md
│  ├─ 07_Skill_Casting_and_Timing.md
│  ├─ 08_Skill_Costs_and_Requirements.md
│  ├─ 09_Skill_Target_and_Usage.md
│  ├─ 10_Skill_States_and_Enhancements.md
│  └─ 11_Skill_Combat_Modifiers_and_Variables.md
│
├─ 🎓 Tutorials & Examples
│  └─ 04_Creating_Passive_Skill_EF_PARAMETER_AMP.md
│
├─ 🤖 AI Instructions (READ THESE!)
│  ├─ AGENTS.md (master instruction file)
│  ├─ SOURCE-CODE-INVESTIGATION-TEMPLATE.md (for analyzing C++)
│  └─ AI-DOCUMENTATION-GUIDELINES.md (cross-linking & standards)
│
└─ 🛠️ Generated Content
   └─ Skills/ [individual skill docs to be created]
```

---

## Frequently Asked Questions for AI

### Q: "Should I repeat the bitmask table in my new effect_type documentation?"
**A**: No! Link to [02_Standard_Bitmasks.md](02_Standard_Bitmasks.md) or [03_Extended_Bitmasks.md](03_Extended_Bitmasks.md) instead. See [AI-DOCUMENTATION-GUIDELINES.md Section 6](AI-DOCUMENTATION-GUIDELINES.md#6-handling-bitmasks-and-encoded-values).

### Q: "I found conflicting information in existing docs and the source code. Which do I trust?"
**A**: Trust the source code first. Update the documentation to match source truth, then add a note in other files linking to the corrected section.

### Q: "How many var fields should my effect_type use?"
**A**: Check the source code for how many `GetVar()` calls the handler makes. Document exactly that many. All others default to 0 or 0.0000.

### Q: "What if I don't know what a var field does?"
**A**: Mark it as "UNUSED" or "safe default: 0.0000" and ask for clarification. Don't guess.

### Q: "Do I always need to create a new file for effect_type documentation?"
**A**: Yes. This keeps documentation modular and reusable. Use numbering `12–20` reserved for effect_types.

### Q: "What's the difference between [var2] and [var5]?"
**A**: They're typically in different "slots" of the same effect. For EF_PARAMETER_AMP: Slot 1 is var1–3 (one stat), Slot 2 is var4–6 (another stat), etc. See [04_Creating_Passive_Skill_EF_PARAMETER_AMP.md Section 4](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md#4-the-mathematics-of-variables-var1-to-var18).

### Q: "How do I format links in markdown to other files?"
**A**: Use relative paths with anchors:
```markdown
[See Standard Bitmasks](02_Standard_Bitmasks.md#bitmask-section)
[Read more in AGENTS.md](AGENTS.md#step-5-create-documentation-file)
```

### Q: "What if the user asks me to update documentation but doesn't specify a file?"
**A**: Check [00_Table_of_Contents.md](00_Table_of_Contents.md) first. If the topic exists, update the original source file. If new, create a new file following the numbering convention.

---

## Before You Start a Task

**Checklist**:
- [ ] Have I read [AI-DOCUMENTATION-GUIDELINES.md](AI-DOCUMENTATION-GUIDELINES.md)?
- [ ] Do I understand the non-redundancy principle (no duplicate explanations)?
- [ ] Have I checked [00_Table_of_Contents.md](00_Table_of_Contents.md) for existing documentation?
- [ ] Do I have the source code path memorized? (`D:\Rappelz_source\program\server\Rappelz-Game-Server`)
- [ ] Am I using relative markdown links?
- [ ] Does my SQL have all 94 columns in exact order?
- [ ] Have I tested/validated my example?

If you can check all boxes, you're ready to proceed! ✅

---

## Getting Help

If you're stuck:

1. **Understanding a concept?** → Check [01_Skill_Database_Architecture.md](01_Skill_Database_Architecture.md)
2. **Need bitmask values?** → Check [02_Standard_Bitmasks.md](02_Standard_Bitmasks.md) or [03_Extended_Bitmasks.md](03_Extended_Bitmasks.md)
3. **Investigating a new effect_type?** → Use [SOURCE-CODE-INVESTIGATION-TEMPLATE.md](SOURCE-CODE-INVESTIGATION-TEMPLATE.md)
4. **Creating documentation?** → Follow [AGENTS.md - Procedure: Investigating Effect Types](AGENTS.md#procedure-investigating-and-documenting-new-effect-types)
5. **Unsure about structure?** → Examine existing effect_type examples in [04_Creating_Passive_Skill_EF_PARAMETER_AMP.md](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md)

---

## Version Info

- **Documentation Version**: 1.0
- **Created**: 2025-02-21
- **Last Updated**: 2025-02-21
- **Target Audience**: AI Agents (Claude, GPT, etc.)
- **Approval Status**: ✅ Ready for Use

