---
id: strat_3_external-storage-integration
version: 2
hierarchy_level: 3
title: External Storage Service Integration
trigger_goals: ["gdrive", "integration", "sync", "cloud storage", "pull", "push", "external storage", "MCP", "provisioning", "folder structure"]
preconditions: ["MCP tools available for the target storage service", "Existing local memory directory structure to mirror", "CLAUDE.md and system specification files exist and are editable"]
confidence: 0.60
success_count: 3
failure_count: 0
source_traces: ["tr_1745683710000_gd02", "tr_1745683800000_gd03", "tr_1745683800001_gd04", "tr_1745683800002_gd05", "tr_1745690400000_af01", "tr_1745690460000_af02"]
deprecated: false
---

# External Storage Service Integration

A tactical procedure for integrating an external cloud storage service (e.g., Google Drive) into the Nebuah system, ensuring that folder structure, tool mappings, memory specifications, and command definitions are all updated consistently.

## Steps
1. Search for existing root folder in the external storage service using MCP discovery tools (e.g., `search_files`). Reuse if found; create only if absent.
2. Create a mirror of the local memory directory structure in the external service: `projects/`, `system/`, `memory/`, `strategies/`, and each `level_N_*` subdirectory. Record every folder ID in a local registry file (e.g., `system/gdrive_registry.json`).
3. Create a sync specification document (e.g., `GDriveSync.md`) covering: registry format, file lifecycle (push/pull/sync), conflict resolution, cross-project access rules, and security considerations. Delegate to a specialist sub-agent for thoroughness.
4. Update the tool map specification (`ClaudeCodeToolMap.md`) with all new MCP tools, operation mappings, workflow patterns, and known limitations for the storage service.
5. Update the memory specification (`SmartMemory.md`) to add the new cloud-backed memory type, its operations, context assembly rules, and cross-project access semantics.
6. Update the command handler (`nebuah.md` or equivalent) with new user-facing commands (e.g., `pull`, `push`, `sync`, `status`, `init`) and any new steps in the execution pipeline.
7. Update agent definitions (`SystemAgent.md`, `DreamEngineAgent.md`, etc.) with any new responsibilities related to the storage integration (e.g., cross-project memory queries, cloud sync in consolidation).
8. Update `CLAUDE.md` with a new integration section, command documentation, and any new rules (e.g., "always check sync status before pushing").
9. Sync agent definition copies across all locations (e.g., `.claude/agents/` and `nebuah/agents/`) to prevent drift.

## Negative Constraints
- Do NOT create folders in external storage without first searching for existing ones -- avoid duplicates
- Do NOT hardcode folder IDs inline; always store them in a centralized registry file
- Do NOT update one specification file without checking all related specifications for cross-reference consistency (tool map, memory spec, command handler, CLAUDE.md)
- Do NOT skip syncing agent definitions across locations after modifying them -- this is a known drift risk (see strat_3_agent-definition-audit)

## Notes
- This strategy was extracted from the first Google Drive integration into Nebuah (2026-04-26)
- The integration touched 6+ system files and required 4 specialized sub-agents
- Registry files (like `gdrive_registry.json`) are critical for mapping local paths to remote folder IDs -- treat them as first-class configuration
- The order of steps matters: folder structure first (Step 2), then specs (Steps 3-5), then commands and agents (Steps 6-8), then sync (Step 9). This ensures each layer builds on a stable foundation.
- When integrating a new storage backend in the future, follow this same sequence but adapt the MCP tool calls and spec sections to the specific service API
- **v2 merge (dream_20260426_a7f3_b):** MCP cloud resource provisioning detail: always use MCP search_files before create_file to avoid duplicate folders. Consider batching folder creation to reduce API round-trips. The registry file is the single source of truth for all folder-to-ID mappings.
- **v3 merge (dream_20260426_7b3e):** After the initial integration is complete, this strategy can be followed by `strat_2_explicit-to-seamless-integration-refactor` to convert explicit commands into automatic workflow steps. The seamless refactor demotes the commands added in Step 6 to "Manual Overrides" and merges integration steps into the main workflow. See also `strat_3_command-handler-seamless-refactor` for the tactical command-handler-specific pattern.
