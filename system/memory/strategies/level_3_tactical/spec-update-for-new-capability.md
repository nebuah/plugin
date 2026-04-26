---
id: strat_3_spec-update-for-new-capability
version: 2
hierarchy_level: 3
title: Specification Update for New Capability Integration
trigger_goals: ["ClaudeCodeToolMap", "SmartMemory", "MCP tools", "operations", "capability update", "spec update", "cross-project", "design principles", "auto-bootstrap protocol", "GDriveSync", "lifecycle"]
preconditions: ["An existing system specification file needs to be extended", "The new capability is well-defined (tools, operations, or memory types)", "A sub-agent with integration expertise can be invoked"]
confidence: 0.55
success_count: 3
failure_count: 0
source_traces: ["tr_1745683800001_gd04", "tr_1745683800002_gd05", "tr_1745690520000_af03"]
deprecated: false
---

# Specification Update for New Capability Integration

A tactical procedure for extending existing system specification documents (such as ClaudeCodeToolMap.md or SmartMemory.md) with new tools, operations, memory types, or workflow patterns when a new capability is being integrated.

## Steps
1. Read the existing specification file to understand its current structure, sections, and conventions
2. Define the additions needed: new tools, operations, memory types, architecture changes, workflow patterns, and limitations
3. Delegate the update to a specialized sub-agent via Task tool (e.g., GDriveToolMapAgent for tool maps, GDriveMemoryAgent for memory specs)
4. Apply the edits to the existing specification, preserving its structure and conventions while adding new sections
5. Cross-reference the updated specification against other related specs to ensure consistency (e.g., tools in ClaudeCodeToolMap must match operations in SmartMemory)
6. Update any higher-level documents (CLAUDE.md, agent definitions) that reference the specification to reflect the new capabilities

## Negative Constraints
- Do NOT restructure or rewrite existing sections when adding new capability sections -- preserve the existing content and append
- Do NOT add tools to a tool map without also adding their operation mappings and limitation entries
- Do NOT allow parameter definitions in the updated spec to contradict definitions in other specs (existing constraint, severity: medium)
- Do NOT add a new memory type without updating the context assembly logic that consumes it

## Notes
- This strategy was derived from two parallel updates: ClaudeCodeToolMap.md (added 7 MCP tools, 4 operations, 1 workflow pattern, 3 limitations) and SmartMemory.md (added Cloud Memory type, 5 operations, cross-project access section)
- The pattern of delegating each spec update to a different specialized agent worked well for parallel execution
- When updating multiple specs for the same capability, ensure they reference each other (e.g., SmartMemory references the tools defined in ClaudeCodeToolMap)
- Success count starts at 2 because this pattern was observed independently in two traces (gd04 and gd05) with consistent positive outcomes
- **v2 merge (dream_20260426_7b3e):** This strategy also applies when transitioning a specification from explicit-operation to seamless-operation design. The variant involves: (1) changing design principles from explicit to seamless, (2) adding zero-config bootstrap principles, (3) adding an Auto-Bootstrap Protocol section with flow, idempotency, and user interaction rules, (4) updating lifecycle diagrams to reflect automatic triggers. Observed in trace af03 where GDriveSync.md was transformed from explicit-command to seamless-auto design.
