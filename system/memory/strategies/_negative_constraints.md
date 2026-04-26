# Negative Constraints

Learned anti-patterns and guardrails. These represent hard-won lessons from past failures.
The Dream Engine appends new constraints here during SWS (Slow Wave Sleep) phase.

**Severity Levels:**
- `high`: Always apply, regardless of context. Violating these caused critical failures.
- `medium`: Apply when context matches. Violating these caused significant rework.
- `low`: Consider when context matches. Violating these caused minor inconvenience.

---

### Never overwrite an entire file without reading it first
**Context:** file editing, any project
**Severity:** high
**Learned from:** seed

### Never cite a case without verifying it has not been overruled
**Context:** legal research, case law analysis
**Severity:** high
**Learned from:** seed

### Never provide legal analysis without identifying the applicable jurisdiction
**Context:** legal research, compliance, contract drafting
**Severity:** high
**Learned from:** seed

### Never list a tool in agent instructions that requires capabilities not in the agent's `tools` field
**Context:** agent definition, plugin configuration
**Severity:** medium
**Learned from:** tr_1741348860000_b3c4
**Dream ID:** dream_20260307_a4f7

### Never duplicate agent definition files across multiple directories without a single-source-of-truth mechanism
**Context:** plugin structure, agent definitions
**Severity:** medium
**Learned from:** tr_1741348860000_b3c4
**Dream ID:** dream_20260307_a4f7

### Never reference non-existent tools or APIs in specification documents
**Context:** system specifications, tool mapping
**Severity:** medium
**Learned from:** tr_1741348920000_d5e6
**Dream ID:** dream_20260307_a4f7

### Never allow multiple specification files to define the same parameter with different values
**Context:** system specifications, configuration consistency
**Severity:** medium
**Learned from:** tr_1741348920000_d5e6
**Dream ID:** dream_20260307_a4f7

### Never create a new strategy without first checking all existing strategy files on disk for the same source_traces
**Context:** dream consolidation, parallel dream sessions, strategy creation
**Severity:** medium
**Learned from:** tr_1745690400000_af01, tr_1745690460000_af02, tr_1745690520000_af03
**Dream ID:** dream_20260426_7c3e
**Detail:** Parallel dream sessions processing the same traces can create duplicate strategies with near-identical content. Before writing any new strategy file, search existing strategies for overlapping source_traces (any shared trace ID). If found, merge into the existing strategy instead of creating a new file. This was observed when three parallel dreams (7b3e, T1700_a3f7, 7c3e) all attempted to create strategies from the af01/af02/af03 traces, resulting in duplicate L2 and L3 strategy files that required post-hoc deduplication.

### Never write a cross-reference to a strategy file without verifying the referenced file still exists
**Context:** dream consolidation, strategy merging, duplicate removal
**Severity:** low
**Learned from:** tr_1745690400000_af01
**Dream ID:** dream_20260426_7c3e
**Detail:** When a parallel dream session deletes a duplicate strategy file, any other file that references the deleted duplicate will have a dangling reference. After any deduplication merge, scan all strategy files for references to the deleted ID and update them to point to the surviving file.
