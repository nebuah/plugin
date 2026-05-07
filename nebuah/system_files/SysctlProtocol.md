# Nebuah System Control Protocol (Sysctl)

## Overview

The Sysctl Protocol defines how Nebuah projects are evaluated, scored, improved, and maintained over their lifecycle. It adapts the skillos-systemcontrol-plugin's governance capabilities for Nebuah's legal domain, GDrive-aware context, and hierarchical memory system.

**Invocation**: `/nebuah sysctl [mode] [project]`

**Core Principle**: The system must get better over time, but never at the cost of stability. Controlled evolution means: measure first, propose second, validate third, apply last.

## Operational Modes

| Mode | Keywords | Agent | What It Does |
|------|----------|-------|-------------|
| **AUDIT** | audit, security, scan | `skillos-systemcontrol-plugin:SecurityAuditAgent` | Security scan of agent prompts for vulnerabilities |
| **SCORE** | score, evaluate, rank | `skillos-systemcontrol-plugin:PerformanceScorecardAgent` | Score agents by trace outcomes using failure taxonomy |
| **EVOLVE** | evolve, improve, optimize | `skillos-systemcontrol-plugin:EvolutionControlAgent` | Propose controlled improvements with anti-overfitting gate |
| **PRUNE** | prune, clean, deprecate | `skillos-systemcontrol-plugin:LifecycleManagerAgent` | Identify dead/redundant agents, compact memory |
| **FULL** | full, everything, maintain | All agents in sequence | AUDIT → SCORE → EVOLVE → PRUNE |
| **HEALTH** | health, status, report | Aggregation | Lightweight audit + score + prune scan |

---

## Security Framework

### Threat Categories

| ID | Category | Severity | Description |
|----|----------|----------|-------------|
| T1 | Prompt Injection | CRITICAL | User input modifies agent behavior beyond intended scope |
| T2 | Unrestricted Execution | CRITICAL | Agent executes arbitrary code from user input |
| T3 | Path Traversal | HIGH | Agent accesses files outside its project boundary |
| T4 | Privilege Escalation | HIGH | Agent gains tool access beyond its declared capabilities |
| T5 | Sensitive Data Exposure | HIGH | Traces or outputs contain credentials, API keys, or PII |
| T6 | Dead Permissions | MEDIUM | Agent has tools it never uses (unnecessary attack surface) |
| T7 | Missing Input Validation | MEDIUM | Agent accepts input without format or content checks |
| T8 | Missing Output Sanitization | LOW | Agent outputs could contain executable content |
| T9 | Verbose Error Logging | LOW | Traces expose internal paths or stack traces |

### Audit Rules

1. **Prompt Injection Resistance**: User input must be in constrained context, not instruction-level text
2. **Minimal Tool Access**: Every tool in `tools:` must be referenced in traces or prompt
3. **Bash Sandboxing**: Bash access must constrain allowed commands
4. **File Path Boundaries**: All file ops must stay within `projects/[ProjectName]/**`
5. **Sensitive Data Handling**: No credentials, API keys, or PII in traces or outputs
6. **Output Format Contracts**: Agent must specify expected output format

### Severity Scoring Impact

- CRITICAL: -1.0 per finding
- HIGH: -0.5 per finding
- MEDIUM: -0.2 per finding
- LOW: -0.1 per finding

### Nebuah-Specific Audit Additions

- **GDrive Credential Exposure**: Check that no GDrive folder IDs or registry paths leak into outputs
- **Legal PII Protection**: Check that client names, case numbers, and privileged information are not in traces
- **Cross-Project Isolation**: Verify agents don't access other projects' data without explicit cross-project memory protocol

---

## 7-Type Failure Taxonomy

Every agent failure maps to exactly one root cause type. This drives targeted remediation.

| Type | Name | Weight | Indicators | Remediation |
|------|------|--------|------------|-------------|
| 1 | INSTRUCTION_AMBIGUITY | 0.3 | Agent misinterprets vague instructions | Add 1 sentence clarifying the decision point |
| 2 | MISSING_TOOL | 0.5 | Agent cannot complete task, lacks required tool | Add tool to `tools:` in YAML frontmatter |
| 3 | PATTERN_MISMATCH | 0.4 | Wrong workflow pattern selected | Select better pattern from memory |
| 4 | MISSING_RECOVERY | 0.6 | Agent fails with unhandled error | Add recovery rule for the specific error |
| 5 | CONTEXT_OVERLOAD | 0.7 | Agent loses track in long interactions | Add checkpointing guidance |
| 6 | STALE_ASSUMPTION | 0.5 | References outdated paths/APIs/facts | Update the stale assumption |
| 7 | UNDERSPECIFIED_OUTPUT | 0.3 | Wrong format or missing fields | Tighten output format contract |

---

## Composite Scoring Formula

### Per-Agent Score (0.0 to 1.0)

```
success_rate = completed_traces / total_traces

failure_severity = SUM(failure_type_weight * count_of_type) / total_failures
  (0.0 if no failures)

output_quality = traces_with_valid_output_format / total_traces
  (1.0 if no format spec exists)

efficiency = 1.0 - (unused_tools_count * 0.05)
  (capped at 0.5 minimum)

composite = (success_rate * 0.40)
          + ((1.0 - failure_severity) * 0.30)
          + (output_quality * 0.20)
          + (efficiency * 0.10)
```

### Tier Classification

| Tier | Score Range | Meaning | Action |
|------|------------|---------|--------|
| **S** | >= 0.90 | Excellent | Template extraction candidate — save as reusable agent pattern |
| **A** | 0.70 - 0.89 | Good | No action needed |
| **B** | 0.50 - 0.69 | Needs Improvement | Evolution candidate |
| **C** | < 0.50 | Underperforming | Evolution or pruning candidate |

### Scoring Edge Cases

- **New agents (< 3 traces)**: Score as "INSUFFICIENT DATA" — don't classify
- **External dependency failures**: Discount failures caused by upstream agents
- **Partial completions**: Count as 0.5 for success_rate

---

## Anti-Overfitting Gate

### The Test

For every proposed change (strategy or agent improvement):

```
"If the specific tasks that caused these failures
 were removed from the workload entirely,
 would this change still make the agent/strategy better?"

YES → The change is generically beneficial → APPROVE
NO  → The change is a rubric hack → REJECT
```

### Examples for Legal Domain

| Proposed Change | Test Result | Verdict |
|----------------|-------------|---------|
| "Add timeout recovery for Bash commands" | YES — timeouts happen in any task | APPROVE |
| "Add knowledge about Delaware corporate law" | NO — only helps Delaware cases | REJECT |
| "Specify output must include citation format" | YES — citations matter in all legal work | APPROVE |
| "Handle the case where opposing counsel's brief is missing" | DEPENDS — if it's a common pattern: APPROVE; if case-specific: REJECT |

### Boundary Cases

- If uncertain, lean toward REJECT — false negatives are safer than overfitting
- Domain-specific knowledge additions fail the gate unless the agent is explicitly domain-scoped
- Cross-cutting improvements (error handling, format validation, recovery rules) usually pass

---

## Minimal-Surface Edit Preference

Ranked from least invasive to most invasive:

| Rank | Edit Type | Description | Risk |
|------|-----------|-------------|------|
| 1 | Instruction Clarification | Add 1 sentence to disambiguate a decision point | Lowest |
| 2 | Tool Addition | Add a tool to the YAML `tools:` list | Low |
| 3 | Recovery Rule | Add a fallback clause for a specific error type | Low-Med |
| 4 | Output Contract | Tighten the output format specification | Medium |
| 5 | Path/Assumption Update | Fix stale path, API endpoint, or fact | Medium |
| 6 | Checkpoint Guidance | Add progress tracking for long workflows | Higher |

**Rules**:
- NEVER rewrite entire agent specs — prefer 5-line patches
- One change per proposal — enables clear attribution
- If 3+ changes needed, consider pruning and replacement instead
- Always show the exact diff (old_string → new_string)

---

## Evolution Pipeline

```
PROPOSED → VALIDATED → STAGED → APPROVED → APPLIED → VERIFIED
```

1. **PROPOSED**: Agent identifies a change based on trace analysis
2. **VALIDATED**: Change passes anti-overfitting gate
3. **STAGED**: Written to `evolution_proposals.md` for review
4. **APPROVED**: Human explicitly approves
5. **APPLIED**: Change made to agent's markdown file
6. **VERIFIED**: Subsequent executions confirm improved scores

### Evidence Requirements

- Minimum 3 traces showing the same failure pattern
- Failure classified by the 7-type taxonomy
- Root cause traceable to specific agent prompt section
- Impact estimate: how many traces would this fix?

---

## Agent Lifecycle

### States

```
CREATED → ACTIVE → [EVOLVING] → DEPRECATED → DELETED
```

### Deprecation Criteria

| Criterion | Threshold |
|-----------|-----------|
| Zero execution | Created > 7 days ago, 0 traces |
| Total failure | >= 5 traces, 100% failure rate |
| Superseded | Another agent handles same tasks with higher score |
| Stale | Not used in 30+ days AND score < 0.5 |
| Redundant | Capabilities are strict subset of another agent |

### Cascading Prune-and-Verify Protocol

Before deleting any agent, follow these steps (NEVER skip):

1. **Inventory target**: Identify agent file and location
2. **Reference scan**: Grep ALL references across the project
3. **Classify references**: Routing-Critical | Runtime-Affecting | Documentation-Only
4. **Plan updates**: Draft edits for routing-critical and runtime-affecting references
5. **Execute**: Apply reference updates FIRST, then delete agent file
6. **Verify**: Re-grep for deleted agent name — should return 0 results

### Agent Merge Protocol

Merge when two agents share > 60% capabilities:
1. Compare YAML frontmatter and prompts side-by-side
2. Identify unique value from each
3. Draft unified spec combining best elements
4. After approval: create merged agent, deprecate both originals

---

## Memory Compaction

### Short-Term (Traces)

| Condition | Action |
|-----------|--------|
| Duplicate traces (same agent, task, outcome within 24h) | Merge into single representative |
| Traces older than 30 days, already consolidated | Archive |
| Orphaned traces (parent references deleted trace) | Reparent to root or archive |
| Pending traces (never completed) | Mark abandoned, archive |

### Long-Term (Strategies)

| Condition | Action |
|-----------|--------|
| Duplicate strategies (> 80% trigger_goals overlap) | Merge, sum success_counts |
| Contradictory strategies (same trigger, opposite advice) | Keep higher confidence |
| Stale strategies (confidence < 0.3, no recent traces) | Archive |
| Orphaned templates (reference nonexistent agents) | Archive or update references |

### Compaction Rules

- Never delete long-term memory without archival
- Always recount after compaction (never trust incremental counts)
- Log compaction as a trace for Dream Engine
- Preserve high-confidence strategies (>= 0.8) regardless of age

---

## GDrive Integration

All sysctl reports MUST be uploaded to Google Drive:

1. Create `output/sysctl/` folder in the project's GDrive if it doesn't exist
2. Upload each report with `mimeType: "text/plain"` and `disableConversionToGoogleType: true`
3. Reports include: `audit_report_YYYYMMDD.md`, `scorecard_YYYYMMDD.md`, `evolution_proposals_YYYYMMDD.md`, `prune_candidates_YYYYMMDD.md`, `health_report_YYYYMMDD.md`
4. The sysctl execution report MUST include a GDrive section confirming uploads

---

## Constraints

- **NC-1**: Never rewrite entire specs during self-improvement
- **NC-2**: Never apply improvements without anti-overfitting gate
- **NC-5**: Never apply improvements directly (staged queue mandatory)
- **NC-6**: Never fabricate failure evidence
- **NC-7**: Never re-propose rejected improvements without new evidence
- **NC-8**: Never prune files without cascading reference updates
- **NC-9**: Always classify references (routing-critical vs doc-debt)
- **NC-10**: Never let registries drift from canonical definitions
- **NC-13**: Always grep for deleted paths after operations
- **NC-21**: Never compute totals by increments — recount from source

## Integration with Dream Engine

Sysctl operations generate traces for dream consolidation:
- **S-tier agents**: Dream Engine extracts their patterns as reusable templates
- **Successful evolution**: Dream Engine learns which edit types work for which failure types
- **Failed evolution → pruning**: Dream Engine learns when to abandon vs improve
- **Memory compaction**: Dream Engine learns which strategies persist vs ephemeral
- **Anti-overfitting gate results**: Dream Engine refines its own strategy creation to avoid overfitting
