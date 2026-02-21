# Rappelz Documentation (RZ-Documentation)

Welcome to the main content creation and modification documentation. Below is the categorization of the game engine's various systems. Click on the links to navigate to the specific section.

## 1. Architecture and Database
- [Skill Database Architecture (Master List)](01_Skill_Database_Architecture.md) - The main index for skill columns.
- [Skill Identifiers and Core Flags](06_Skill_Identifiers_and_Core_Flags.md) - `id`, `elemental`, `is_passive`, `effect_type`.
- [Skill Casting and Timing](07_Skill_Casting_and_Timing.md) - Cooldowns, cast limits, and projectile logic.
- [Skill Costs, Requirements, and Weapon Limits](08_Skill_Costs_and_Requirements.md) - Resources, prerequisites, and `vf_...` equipment flags.
- [Skill Target and Usage Conditions](09_Skill_Target_and_Usage.md) - AoE/Target limits and `tf_`/`uf_` combat faction flags.
- [Skill States and Enhancements](10_Skill_States_and_Enhancements.md) - Enduring buff IDs and Card enhancement levels.
- [Skill Combat Modifiers and Variables](11_Skill_Combat_Modifiers_and_Variables.md) - C++ variable interpreters, Hate (aggro) scaling, and Base Hit/Crit bonuses.

## 2. Bitmasks, Elements, and Parameters
- [Standard Bitmasks (Base Stats and Combat)](02_Standard_Bitmasks.md) - Basic flags (e.g., P.Atk, M.Atk, Attack Speed, Max HP).
- [Extended Bitmasks (ET_ and Elementals)](03_Extended_Bitmasks.md) - Extended flags (e.g., Elemental Resistances, Penetration, Defense Ignore).
- [Detailed Elementals Guide](05_Elementals_Guide.md) - Core engine overview of Elemental attributes and target mechanics.

## 3. Creating Skills from Scratch (Tutorials)
- [Complex Passive Skill: EF_PARAMETER_AMP (Effect 4)](04_Creating_Passive_Skill_EF_PARAMETER_AMP.md) - A step-by-step guide to building a powerful passive skill that manipulates secondary stats, elementals, and penetrations.

## 4. Workflows and Generators
- [Instructions for AI Agents: Generating Rappelz Skills](AGENTS.md) - Best practices and strict mapping rules for prompting an AI to generate skill `INSERT` SQL queries.
