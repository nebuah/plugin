---
id: strat_2_gdrive-integration-architecture
version: 2
hierarchy_level: 2
title: Cloud Storage Integration Architecture
trigger_goals: ["Google Drive", "integration", "cloud storage", "GDriveSync", "folder structure"]
preconditions: ["Existing local memory/strategy directory structure", "Cloud storage API access (e.g., Google Drive MCP tools)", "Root folder already exists or can be created in cloud provider"]
confidence: 0.55
success_count: 2
failure_count: 0
source_traces: ["tr_1745683680000_gd01", "tr_1745683710000_gd02", "tr_1745683800000_gd03", "tr_1745683800001_gd04", "tr_1745683800002_gd05", "tr_1745690400000_af01"]
deprecated: false
---

# Cloud Storage Integration Architecture

A systematic approach for integrating a cloud storage provider (Google Drive, OneDrive, etc.) with an existing local cognitive system. Covers folder mirroring, registry creation, specification authoring, tool mapping, memory layer updates, and configuration propagation.

## Steps
1. Load negative constraints and dream journal for context before starting
2. Search existing strategies for anything matching the integration goal to avoid redundant work
3. Explore the full codebase to understand current architecture, file count, and structure
4. Discover or create the root folder in the cloud storage provider
5. Create a mirrored folder hierarchy in cloud storage that reflects the local directory structure (projects/, system/, memory/, strategies/, level_1-4 directories)
6. Create a local registry file (e.g., gdrive_registry.json) mapping local paths to cloud folder/file IDs for deterministic lookups
7. Delegate specification creation to a dedicated architecture sub-agent -- the spec should cover registry format, lifecycle hooks, sync protocols, conflict resolution, cross-project access, and security
8. Delegate tool mapping updates to a dedicated sub-agent -- add cloud provider operations, MCP tool entries, and workflow patterns to the existing tool map
9. Delegate memory specification updates to a dedicated sub-agent -- add a Cloud Memory type, new operations, context assembly rules, and cross-project access patterns
10. Update the command handler with new commands (pull, push, sync, status, init) and add cloud-aware steps to existing workflows
11. Update all agent definitions to include cloud integration capabilities and cross-project memory query sections
12. Update root configuration (CLAUDE.md) with a new integration section, cloud commands, and additional rules

## Negative Constraints
- Do NOT create cloud folders without first checking if they already exist -- search before creating to avoid duplicates
- Do NOT hardcode cloud folder IDs in multiple files -- maintain a single registry file as the source of truth
- Do NOT skip updating the root configuration (CLAUDE.md) -- specifications that are not referenced from the entry point provide no runtime value
- Do NOT create the specification monolithically -- delegate to focused sub-agents (architecture, tool-map, memory) for separation of concerns
- Do NOT modify seed strategies in _seeds/ even if they need cloud-awareness -- extend downstream instead

## Notes
- This strategy was extracted from the first Google Drive integration with the Nebuah cognitive OS
- The integration touched 8+ files across the system: registry, specification, tool map, memory spec, command handler, 3 agent definitions, and root config
- Key architectural decision: mirror the local directory structure exactly in cloud storage for conceptual simplicity
- Key pattern: a single JSON registry file eliminates the need for repeated cloud API lookups
- The 887-line GDriveSync specification covers 7 sections and serves as the canonical reference
- Sub-agent delegation (3 agents: architect, tool-map, memory) proved effective for parallel specification work
- The existing plugin-system-audit strategy (strat_2_plugin-system-audit) was partially applicable during spec creation but was not a full match -- a dedicated specification-creation strategy is warranted
- **v2 update (dream_20260426T1700_a3f7):** This strategy was successfully reused as input for the seamless transformation project (trace af01), confirming the 12-step integration sequence is a reliable foundation. After initial integration is complete, the follow-on transformation to seamless auto-integration is captured in `strat_2_explicit-to-seamless-integration-refactor`.
