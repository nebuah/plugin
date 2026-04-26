---
id: strat_3_command-handler-seamless-refactor
version: 2
hierarchy_level: 3
title: Command Handler Seamless Refactor
trigger_goals: ["command handler", "nebuah.md", "seamless", "Step 0", "auto-bootstrap", "merge steps", "manual overrides", "workflow transformation", "auto-integration"]
preconditions: ["An existing command handler with explicit integration-specific commands", "The command handler has paired steps (e.g., Step N and Step N.5) for the integration", "The integration is mature enough to be automated (at least one successful manual execution)"]
confidence: 0.55
success_count: 2
failure_count: 0
source_traces: ["tr_1745690460000_af02"]
deprecated: false
---

# Command Handler Seamless Refactor

A tactical procedure for transforming a command handler file from an explicit-command model to a seamless auto-integration model. This involves demoting explicit commands to manual overrides, adding a Step 0 auto-bootstrap phase, merging paired workflow steps, and updating report templates.

## Steps
1. Read the full command handler file to understand its current structure, routing logic, and all integration-specific sections
2. Change the explicit command section header from its current name (e.g., "GOOGLE DRIVE COMMANDS") to "Manual Overrides" and add a note to each command that it runs automatically and is only needed for manual control
3. Simplify routing logic: remove any special-case routing for the integration (e.g., remove "For non-dream, non-gdrive goals" and simplify to "For non-dream goals") since the integration is now automatic for all goals
4. Add Step 0: AUTO-BOOTSTRAP before the first workflow step. This step should: (a) search for existing resources in the external service, (b) create any missing resources, (c) register resource IDs in the local registry, (d) be fully idempotent (safe to run on every invocation)
5. Merge paired steps: for each pair of steps where one is the core workflow and the other is the integration add-on (e.g., Step 3 + Step 3.5), combine them into a single unified step with a descriptive name that covers both concerns (e.g., "CREATE PROJECT STRUCTURE (Local + GDrive)")
6. Update the report/output template at the end of the workflow to include an automatic status section for the integration (e.g., "Google Drive (Automatic)" showing sync status)
7. Verify that the merged steps handle partial failures gracefully: a cloud failure should not prevent the local operation from completing
8. Review the final command handler to ensure no orphaned references to removed steps or commands remain

## Negative Constraints
- Do NOT delete the explicit commands -- demote them to manual overrides so experienced users can still invoke them directly when needed
- Do NOT merge steps if the combined step would have more than 7 sub-actions -- split into logically grouped steps instead
- Do NOT add Step 0 without ensuring it is idempotent -- repeated invocations must not create duplicate resources
- Do NOT remove routing logic for other command types (e.g., dream commands) when simplifying the integration routing
- Do NOT forget to update the report template -- users need visibility into what the auto-integration did
- Do NOT merge steps that have incompatible error handling requirements (e.g., if cloud upload failure should abort but local output failure should not)
- Do NOT change step numbering in a way that breaks references in other files (agent definitions, specifications, CLAUDE.md) -- verify cross-references before renumbering

## Notes
- This strategy was extracted from the transformation of `nebuah/commands/nebuah.md` during the GDrive seamless integration refactor
- 7 discrete edits were applied to the command handler in a specific order: (1) header rename, (2) routing simplification, (3) Step 0 addition, (4-6) three step merges, (7) report template update
- The Step 0 auto-bootstrap pattern uses a search-before-create approach: first check if the required external resources already exist, then create only what is missing, then register everything in the local registry
- Step merging reduced the total step count in the workflow, making it easier for both humans and agents to follow the execution flow
- The "Manual Overrides" framing is important: it preserves backward compatibility while clearly communicating that manual invocation is no longer the primary path
- **v2 merge (dream_20260426T1700_a3f7):** Consolidated duplicate strategy `strat_3_command-handler-seamless-transformation` into this file. Added unique negative constraint about step-numbering cross-reference safety. Increased success_count (1->2) and confidence (0.50->0.55) based on corroborating evidence from the parallel dream session. Both dream sessions independently derived the same 7-step procedure, confirming pattern stability.
