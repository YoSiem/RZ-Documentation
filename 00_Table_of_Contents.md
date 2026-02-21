# Rappelz Documentation (RZ-Documentation)

Welcome to the main content creation and modification documentation. Below is the categorization of the game engine's various systems. Click on the links to navigate to the specific section.

> **🤖 For AI Agents**: Start with [AI-QUICK-START.md](AI-QUICK-START.md) to understand the documentation structure and find your task-specific guide.

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

## 4. Effect Type Deep Dives
- [EF_MAGIC_SINGLE_DAMAGE (231)](12_Effect_Type_EF_MAGIC_SINGLE_DAMAGE.md) - Standard single-target magical damage: M.Atk multiplier + flat bonus, optional on-hit debuff, elemental support.
- [EF_MAGIC_MULTIPLE_DAMAGE (232) & EF_MAGIC_DAMAGE_WITH_ABSORB_HP_MP (235)](13_Effect_Type_Magic_Damage_Variants.md) - Multi-hit magic damage (232) and life/mana drain damage (235). Both extend the base 231 formula.
- [EF_MAGIC_SINGLE_REGION_DAMAGE (261), EF_MAGIC_SPECIAL_REGION_DAMAGE (262) & EF_MAGIC_MULTIPLE_REGION_DAMAGE (263)](14_Effect_Type_Magic_Region_Damage.md) - AoE variants: target-centered AoE (261), ground-targeted AoE (262), and multi-hit AoE (263). All extend the base 231 formula.
- [EF_ADD_STATE (301), EF_ADD_REGION_STATE (302) & EF_ADD_STATE_TO_CASTER_AND_TARGET (311)](15_Effect_Type_Add_State.md) - Pure state application: single-target (301), AoE (302), and simultaneous caster+target (311). No damage — driven entirely by state columns.
- [EF_ADD_HP (501) & EF_TOGGLE_AURA (701)](16_Effect_Type_Add_HP_and_Toggle_Aura.md) - Utility types: instant HP heal with M.Atk-based formula (501), and persistent pulsing area aura via toggle system (701).
- [EF_WEAPON_MASTERY (10001)](17_Effect_Type_Weapon_Mastery.md) - Passive stat amplification (same bitmask architecture as EF_PARAMETER_AMP) gated by equipped weapon type via vf_ flags.
- [Reactive Passives: EF_ADD_STATE_ON_ATTACK (10048), EF_ADD_STATE_BY_SELF_ON_ATTACK (10049), EF_ADD_STATE_BY_SELF_ON_KILL (10052), EF_ADD_STATE_ON_BEING_CRITICAL_ATTACKED (10055), EF_ADD_STATE_BY_SELF_ON_BEING_CRITICAL_ATTACKED (10056), EF_INC_SKILL_COOL_TIME_ON_ATTACK (10063)](18_Effect_Type_Reactive_Passives.md) - Combat-event triggered passives: on-attack state procs, on-kill buffs, on-being-critted counters, and attack-based cooldown reduction.
- [EF_MAGIC_SINGLE_DAMAGE_WITH_PHYSICAL_DAMAGE (32021)](19_Effect_Type_Magic_Physical_Hybrid.md) - Hybrid active skill dealing independent magic (var1–var6) + physical (var7–var12) hits. Physical component checks P.Def; elemental applies to magic only.
- [EF_AMP_DAMAGE_BY_TARGET_STATE (32032) & EF_AMP_DAMAGE_INC_CRIT_RATE_BY_TARGET_HP_RATIO (32202)](20_Effect_Type_Conditional_Damage_Amp.md) - Conditional damage amplifiers extending 231: 32032 deals bonus damage if target has a specific state; 32202 increases crit rate based on target's missing HP.
- [EF_MAGIC_CHAIN_DAMAGE (32062), EF_ABSORB_DAMAGE (32211) & EF_INC_PARAM_BASED_PARAM (32301)](21_Effect_Type_Chain_Absorb_Param_Scaling.md) - Chain magic bouncing between enemies (32062), active damage-absorb shield (32211), and passive cross-stat conversion (32301).

## 5. AI Instructions and Workflows
### 🚀 Entry Point
- **[AI Quick Start Guide](AI-QUICK-START.md)** - **READ THIS FIRST!** Task dispatcher and orientation guide for AI agents. Choose your task and follow the recommended reading order.

### 📖 Core Documentation
- [AI Documentation Guidelines for Rappelz Skill Database](AI-DOCUMENTATION-GUIDELINES.md) - Core principles for creating consistent, non-redundant documentation with proper cross-linking.
- [Instructions for AI Agents: Skill Generation and Effect Type Documentation](AGENTS.md) - Detailed procedures for generating SQL skills, investigating new effect_types, and documenting game mechanics from source code.
- [Source Code Investigation Template](SOURCE-CODE-INVESTIGATION-TEMPLATE.md) - Structured template for analyzing C++ source code and documenting effect handlers, variables, and constraints before writing final documentation.
