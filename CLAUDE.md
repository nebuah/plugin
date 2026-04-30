# Nebuah Kernel

You are operating within **Nebuah**, a bio-inspired cognitive operating system for legal research, literature, and legal consulting. This system brings the algorithmic rigor of hierarchical memory consolidation (proven in robotics) to Claude Code and Claude Cowork workflows, specialized for the legal domain.

## Core Architecture

Nebuah implements a **4-level cognitive hierarchy** adapted from neuroscience:

| Level | Name | Legal Equivalent | Example |
|-------|------|---------------------|---------|
| L1 | GOAL | Engagement / Case | "Handle corporate M&A due diligence" |
| L2 | ARCHITECTURE | Strategy / Framework | "Design regulatory compliance framework" |
| L3 | TACTICAL | Document / Task | "Draft non-disclosure agreement" |
| L4 | REACTIVE | Action / Query | `Search: case law on indemnification caps` |

## Before Starting Any Complex Task

1. **Check negative constraints** to avoid past mistakes:
   - Read `system/memory/strategies/_negative_constraints.md`
   - Apply ALL constraints with severity `high` unconditionally
   - Apply `medium` constraints when context matches

2. **Search for relevant strategies** before planning:
   - Use `Grep` on `system/memory/strategies/` for keywords related to the user's goal
   - If a strategy with `confidence >= 0.5` matches, APPLY its steps
   - If multiple strategies match, prefer the one with highest `success_count`

3. **Check the dream journal** for recent learnings:
   - Read `system/memory/strategies/_dream_journal.md` for the last 3 entries

4. **Auto-bootstrap Google Drive** (first run only, then no-op):
   - If `system/gdrive_registry.json` doesn't exist: search GDrive for "Nebuah" folder → use/create → build system sub-folders → write registry
   - Only ask the user if multiple root folders found (disambiguation)
   - After bootstrap, all GDrive operations are fully automatic

5. **Search cross-project memories in Google Drive** (automatic):
   - Use `mcp__claude_ai_Google_Drive__search_files` with `fullText contains '[goal keywords]'`
   - Load matching strategies with `mcp__claude_ai_Google_Drive__read_file_content`
   - Add as Priority 15 context (between constraints and local strategies)

## During Task Execution

### Trace Logging
All significant actions MUST be logged as traces. Use the following format for trace files in `system/memory/traces/`:

**File naming**: `trace_YYYY-MM-DD.md` (one file per day, append new traces)

**Trace format**:
```markdown
### Time: [ISO 8601]
**Trace ID:** tr_[timestamp]_[random]
**Level:** [1-4]
**Parent:** [parent_trace_id or null]
**Goal:** [what was being attempted]
**Strategy:** [strategy_id if one was applied, or null]
**Outcome:** [SUCCESS | FAILURE | PARTIAL | ABORTED | UNKNOWN]
**Reason:** [brief explanation of outcome]
**Duration:** [time in ms if measurable]
**Confidence:** [0.0-1.0]

**Actions:**
1. [Tool: action description] -> [result summary]
2. [Tool: action description] -> [result summary]
---
```

### Hierarchy Rules
- L1 traces wrap entire case/engagement executions
- L2 traces wrap strategic decisions and framework design phases
- L3 traces wrap individual document/analysis implementations
- L4 traces wrap individual actions and queries
- Every L2+ trace MUST reference a parent trace ID

## After Completing a Complex Task

Invoke the `DreamEngineAgent` sub-agent using the `Task` tool to consolidate learnings:

```
Task(subagent_type="nebuah:DreamEngineAgent", prompt="Run dream consolidation cycle on recent traces in system/memory/traces/")
```

This triggers the 3-phase bio-inspired consolidation:
1. **SWS (Slow Wave Sleep)**: Analyze failures, extract negative constraints
2. **REM Sleep**: Abstract successful patterns into reusable strategies
3. **Consolidation**: Write strategies to disk, update journal, prune old traces

## Memory Architecture

```
system/memory/
├── strategies/               # Long-term memory (persistent knowledge)
│   ├── level_1_epics/        # Engagement-level patterns
│   ├── level_2_architecture/ # Legal strategy & framework strategies
│   ├── level_3_tactical/     # Document/task-level tactics
│   ├── level_4_reactive/     # Action-level patterns
│   ├── _seeds/               # Bootstrap strategies (never deleted)
│   ├── _negative_constraints.md  # What NOT to do
│   └── _dream_journal.md    # Consolidation history
├── traces/                   # Short-term memory (volatile execution logs)
│   └── trace_YYYY-MM-DD.md   # Daily trace files
└── gdrive_registry.json      # Google Drive folder ID mappings

Google Drive (Nebuah/):        # Cloud memory (persistent, shared)
├── projects/[ProjectName]/
│   ├── input/                # Source documents (Google Docs, PDF, etc.)
│   ├── output/               # Generated deliverables
│   └── memory/long_term/     # Project-specific learnings
└── system/memory/strategies/ # Mirror of local strategies
```

## Strategy Format

Strategies are Markdown files with YAML frontmatter:

```yaml
---
id: strat_[level]_[slug]
version: 1
hierarchy_level: [1-4]
title: [Human-readable title]
trigger_goals: ["keyword1", "keyword2"]
preconditions: ["condition1"]
confidence: 0.5
success_count: 0
failure_count: 0
source_traces: []
deprecated: false
---
```

## Claude Cowork Integration

When running in **Claude Cowork** (background agents):
- The **SystemAgent** (Cortex) handles user-facing orchestration
- The **DreamEngineAgent** (Hippocampus) can run asynchronously in background to consolidate memory while you work
- The **MemoryAnalysisAgent** handles real-time trace logging

## Execution Modes

### Default Multi-Agent Mode (Triad Decomposition)

Every `/nebuah [goal]` invocation **always creates at least 3 agents** along these concern axes:

| Agent Role | Responsibility |
|---|---|
| **Research Agent** | Core legal research (case law, statutes, doctrinal analysis) |
| **Quality Agent** | Compliance review, citation verification, factual accuracy |
| **Integration Agent** | Document formatting, deliverable assembly, client-ready output |

For complex goals, additional agents are created beyond the minimum 3. After execution, **one dream cycle per agent** runs in parallel (minimum 3), each filtered by that agent's keywords.

### Loop Mode (Recurring Dreams)

`/nebuah loop` generates `/loop` commands for the user to copy-paste (the plugin cannot invoke `/loop` directly since it's a built-in CLI command):

- `/nebuah loop` → outputs: `/loop 1h /nebuah dream`
- `/nebuah loop [keywords]` → outputs: `/loop 1h /nebuah dream [keywords]`
- `/nebuah loop [interval] [keywords]` → outputs: `/loop [interval] /nebuah dream [keywords]`
- `/nebuah loop stop` → outputs cancellation instructions

Loop commands are session-scoped (max 3 days) and stop when the session ends or Ctrl+C is pressed.

## Google Drive Integration (MANDATORY — Never Optional)

Nebuah integrates with Google Drive as a **MANDATORY requirement**. Every file produced by the system MUST be saved to Google Drive. The user types `/nebuah [goal]` and all GDrive operations happen transparently — no setup commands, no manual sync. **GDrive is NOT optional** — if the MCP tools are available, they MUST be used.

### Enforcement Rules
- **NEVER silently skip** GDrive uploads. If `system/gdrive_registry.json` is missing, bootstrap it immediately — do not skip.
- **ALWAYS use `disableConversionToGoogleType: true`** when uploading markdown, strategy, or any text file. Use `mimeType: "text/plain"` (not `"text/markdown"`). This prevents Google from auto-converting files and corrupting YAML frontmatter.
- **NEVER report task success** without confirming GDrive uploads completed. The execution report MUST include GDrive status.
- **If an upload fails**, retry once. If it still fails, log the failure explicitly — never hide it.

### How It Works
1. **Auto-bootstrap** (first run only): Detects or creates the Nebuah root folder in GDrive. Only asks the user if disambiguation is needed.
2. **Project creation**: MUST create GDrive project folders (input/, output/, memory/) and download any input documents.
3. **Output upload**: MUST upload ALL deliverables to GDrive after production. Non-optional.
4. **Memory sync**: MUST upload ALL strategies/constraints/journal to GDrive after dream consolidation. Non-optional.
5. **Cross-project memory**: Searches GDrive for strategies from past projects during planning.

### Manual Override Commands (Advanced)
- `/nebuah gdrive pull [project]` — Force download input documents
- `/nebuah gdrive push [project]` — Force upload outputs + memories
- `/nebuah gdrive sync` — Force bidirectional system memory sync
- `/nebuah gdrive status` — Show sync state

### Strategy Upload Rule
Always upload strategies with `disableConversionToGoogleType: true` to preserve YAML frontmatter.

## Reference Specifications

Detailed specifications for system internals are in `nebuah/system_files/`:
- **SmartMemory.md**: Memory hierarchy, retrieval algorithm, and context assembly
- **MemoryTraceManager.md**: Trace schema, lifecycle, and hierarchy rules
- **QueryMemoryTool.md**: Strategy scoring algorithm and query patterns
- **ClaudeCodeToolMap.md**: Tool-to-operation mapping and workflow patterns
- **GDriveSync.md**: Google Drive integration, sync protocols, and cross-project memory access

Consult these when implementing complex memory operations or debugging the consolidation pipeline.

## Important Rules

1. NEVER delete seed strategies in `_seeds/` - they are the bootstrap knowledge
2. ALWAYS log traces for tasks that take more than 2 tool calls
3. PREFER applying existing strategies over improvising from scratch
4. UPDATE strategy confidence after each application (success: +0.05, failure: -0.1)
5. DEPRECATE strategies where `failure_count > success_count * 2`
6. **MANDATORY**: SYNC ALL outputs and strategies to Google Drive — never skip, never silently fail. If registry is missing, bootstrap it. If upload fails, retry once and report.
7. NEVER upload traces or agent definitions to Google Drive (sensitive data)
8. NEVER modify cross-project memories in GDrive (read-only access)
9. AUTO-BOOTSTRAP GDrive on first run — never require manual setup commands
10. NEVER ask the user more than ONE question about GDrive setup (only for root folder disambiguation)
11. **MANDATORY**: ALWAYS use `mimeType: "text/plain"` with `disableConversionToGoogleType: true` when uploading ANY file to GDrive. Never use `text/markdown` without this flag. Google auto-converts markdown to Google Docs and corrupts YAML frontmatter.
12. **MANDATORY**: A task is NOT complete until its outputs are confirmed uploaded to Google Drive. The execution report MUST show GDrive sync results. Zero GDrive uploads when local files exist = FAILURE.
