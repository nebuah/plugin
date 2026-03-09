# Memory Trace Manager — Nebuah Edition

Coordinates the lifecycle of hierarchical memory traces within the Nebuah cognitive architecture. This is the specification that all agents must follow when creating, linking, and managing traces.

## Trace Schema

Every trace entry MUST contain these fields:

```yaml
trace_id: string          # Unique: tr_[unix_ms]_[4hex]
hierarchy_level: 1-4       # L1=GOAL, L2=ARCHITECTURE, L3=TACTICAL, L4=REACTIVE
parent_trace_id: string?   # null for root traces, parent ID for children
timestamp: string          # ISO 8601 (e.g., 2026-03-06T14:30:22.000Z)
goal: string               # What was being attempted
strategy_id: string?       # ID of applied strategy (null if none)
outcome: enum              # SUCCESS | FAILURE | PARTIAL | ABORTED | UNKNOWN
outcome_reason: string     # Brief explanation of the outcome
duration_ms: number?       # Execution time (null if not measurable)
confidence: number         # 0.0-1.0 estimated quality
actions: ActionEntry[]     # List of actions taken
```

### ActionEntry Schema

```yaml
timestamp: string          # ISO 8601
tool: string               # Tool name (Read, Write, Bash, Task, Grep, etc.)
description: string        # What the action did
result_summary: string     # Brief outcome ("found 3 precedents", "draft completed", "error: missing citation")
```

## Trace Hierarchy

Traces form a tree structure via `parent_trace_id`:

```
L1 GOAL: "Handle corporate M&A due diligence"
├── L2 ARCHITECTURE: "Design due diligence framework"
│   ├── L3 TACTICAL: "Review target company contracts"
│   │   ├── L4 REACTIVE: Read vendor-agreement-2024.pdf
│   │   └── L4 REACTIVE: Flag non-standard indemnification clause
│   └── L3 TACTICAL: "Analyze regulatory compliance"
│       └── L4 REACTIVE: Search SEC filing requirements
└── L2 ARCHITECTURE: "Assess litigation exposure"
    └── L3 TACTICAL: "Review pending lawsuits"
        └── L4 REACTIVE: Read case-docket-2025-cv-1234.pdf
```

## Trace File Format (Markdown)

**Location**: `system/memory/traces/trace_YYYY-MM-DD.md`
**Mode**: Append-only (one file per day)

```markdown
### Time: 2026-03-06T14:30:22.000Z
**Trace ID:** tr_1709736622000_a3f2
**Level:** 3
**Parent:** tr_1709736000000_b1c4
**Goal:** Review vendor contracts for non-standard terms
**Strategy:** strat_3_contract-review
**Outcome:** SUCCESS
**Reason:** 12 contracts reviewed, 3 flagged with non-standard indemnification
**Duration:** 45000
**Confidence:** 0.85

**Actions:**
1. [Read: contracts/vendor/] -> Loaded 12 vendor agreements
2. [Grep: "indemnif|limitation of liability"] -> Found clauses in 8 contracts
3. [Read: flagged contracts] -> Identified 3 with non-standard terms
4. [Write: output/due-diligence-flags.md] -> Documented flagged clauses with analysis
---
```

## Trace Linking Patterns

### Sequential (Task B follows Task A)
```
Trace A (outcome: SUCCESS) → Trace B (parent: A.id)
```

### Hierarchical (Task B is subtask of Task A)
```
Trace A (L2) → Trace B (L3, parent: A.id)
```

### Dependency (Task B requires output from Task A)
```
Trace A (outcome: SUCCESS, output: due-diligence-flags.md)
Trace B (parent: A.parent, actions include Read: due-diligence-flags.md)
```

## Trace Lifecycle

### Phase 1: Active Execution
- Traces created in real-time during task execution
- Written to daily trace files
- Actions appended as they complete

### Phase 2: Dream Consolidation
- DreamEngineAgent reads traces from `system/memory/traces/`
- Groups into sequences by parent-child relationships
- Analyzes for strategies (successes) and constraints (failures)
- Writes results to `system/memory/strategies/`

### Phase 3: Pruning
- Trace files older than 7 days are deleted by the DreamEngine
- Strategies persist indefinitely in `system/memory/strategies/`
- Dream journal preserves a summary of what was learned

## Outcome Aggregation

When a parent trace has multiple children:

| Children Outcomes | Parent Outcome |
|-------------------|----------------|
| All SUCCESS | SUCCESS |
| All FAILURE | FAILURE |
| Mix (any SUCCESS + any FAILURE) | PARTIAL |
| Any ABORTED | ABORTED |
| All UNKNOWN | UNKNOWN |

## Confidence Calibration Guidelines

| Scenario | Confidence |
|----------|------------|
| Clean execution, thorough analysis, all citations verified | 0.9-1.0 |
| Completed with 1-2 minor adjustments | 0.7-0.8 |
| Completed after significant rework | 0.5-0.6 |
| Barely completed, many workarounds | 0.3-0.4 |
| Mostly failed, trivial progress only | 0.1-0.2 |

## Action Compression

For traces with many repetitive actions, compress using run-length notation:

**Instead of**:
```
1. [Read: contracts/vendor/agreement-001.pdf] -> Standard terms
2. [Read: contracts/vendor/agreement-002.pdf] -> Standard terms
3. [Read: contracts/vendor/agreement-003.pdf] -> Non-standard indemnification
...
```

**Write**:
```
1. [Read: scanned 12 vendor agreements in contracts/vendor/] -> 3 flagged with non-standard terms
```

## Tool-to-Action Mapping

| Claude Code Tool | Nebuah Action Category | Trace Level |
|------------------|------------------------|-------------|
| Read, Glob, Grep | Discovery / Analysis | L4 |
| Write, Edit | Creation / Modification | L3-L4 |
| Bash (document processing) | Execution | L3-L4 |
| Bash (file operations) | Execution | L2-L3 |
| Task (agent delegation) | Orchestration | L1-L2 |
| WebSearch, WebFetch | Legal Research | L2-L3 |
