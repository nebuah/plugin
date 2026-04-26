---
id: strat_3_system-spec-creation-via-delegation
version: 1
hierarchy_level: 3
title: System Specification Creation via Agent Delegation
trigger_goals: ["specification", "system file", "architecture document", "GDriveSync", "design doc", "delegation"]
preconditions: ["The specification scope and required sections are clearly defined", "A sub-agent with domain expertise can be invoked via Task tool", "Target output directory exists"]
confidence: 0.5
success_count: 1
failure_count: 0
source_traces: ["tr_1745683800000_gd03"]
deprecated: false
---

# System Specification Creation via Agent Delegation

A tactical procedure for creating comprehensive system specification documents by delegating to a specialized sub-agent (architect agent), then writing the output to the system_files directory.

## Steps
1. Define the specification scope: list all required sections (e.g., registry, lifecycle, sync protocols, security, conflict resolution)
2. Identify or create a specialized sub-agent with the domain expertise needed (e.g., GDriveArchitectAgent for cloud sync specs)
3. Delegate specification creation to the sub-agent via Task tool, providing the scope, section requirements, and any existing patterns to follow
4. Receive the sub-agent's output and write it to the appropriate system_files location (e.g., nebuah/system_files/[Name].md)
5. Validate completeness: verify all required sections are present and cross-references to other specs are correct
6. Update CLAUDE.md or relevant configuration to reference the new specification so it is discoverable at runtime

## Negative Constraints
- Do NOT create specifications that reference tools or APIs that do not exist in the current runtime (existing constraint, severity: medium)
- Do NOT create a specification without linking it from CLAUDE.md or an agent definition -- unreferenced specs provide no runtime value
- Do NOT allow the delegated agent to invent capabilities beyond what the system actually supports

## Notes
- This strategy was extracted from the creation of GDriveSync.md (887 lines, 7 sections)
- Delegation via Task tool allows the architect agent to focus solely on specification quality without context-switching
- The partial application of strat_2_plugin-system-audit during this process improved structural consistency
- For very large specifications (500+ lines), consider breaking into multiple sub-agent calls, one per major section
