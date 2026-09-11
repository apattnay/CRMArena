# 05 — Scaling, DB, and Query Analysis

## The question

This document answers four architecture questions for the core loop:

```text
agent generates query
run SOQL/SOSL
records return
agent final answer
```

## 1. How many agents are involved?

### Per task, non-interactive
- **1 task-solving agent**
  - `ChatAgent` or `ToolCallAgent`
- **1 evaluator model**
  - used after final answer scoring

### Per task, interactive
- **1 task-solving agent**
- **1 simulated user** via `LLMUserSimulationEnv`
- **1 evaluator model**

### Important nuance
These are not multiple collaborating worker agents within the same reasoning loop. The benchmark is primarily **single-agent execution per task**, plus optional simulated user interaction and post-hoc evaluation.

## 2. What scales in parallel?

### Inside one task
Execution is effectively sequential:

```text
agent action
  → query/tool execution
  → observation return
  → next action
```

The agent does **one action at a time**.

### Across many runs
Parallelism happens at the shell-script level:
- `run_tasks_crmarena.sh`
- `run_tasks_crmarena_pro.sh`

Those scripts iterate over:
- model
- task category
- strategy

and launch Python runs in the background using `&`.

### Conclusion
- **intra-task parallelism:** no meaningful parallel query fanout
- **inter-run parallelism:** yes, via multiple background processes
- **multi-agent collaborative querying:** not implemented

## 3. How big is the DB?

Visible local DB artifacts in `local_data/`:

- `crmarena_data.db` — about **8.1 MB**
- `crmarenapro_b2b_data.db` — about **34.6 MB**
- `crmarenapro_b2c_data.db` — about **57.4 MB**

Combined size is roughly **100 MB**.

### What this means
- local database snapshots exist in the repository
- the main runtime path analyzed in code uses **Salesforce API queries** through `SalesforceConnector`
- the highlighted loop is therefore logically querying the **Salesforce org**, not directly issuing SQL against those local DB files

### What cannot be concluded from code alone
Without opening those database files directly, we cannot state with confidence:
- exact row counts
- per-table record counts
- selectivity distributions

## 4. How big is the query?

There are three useful meanings of “query size.”

### A. Query string size
The generated SOQL/SOSL strings are usually modest:
- a single `SELECT ... FROM ... WHERE ...`
- or a single `FIND {...} RETURNING ...`

Typical complexity:
- tens to a few hundred characters
- targeted filters on dates, IDs, statuses, or relationship fields

### B. Query result size
Result size can vary widely:
- single IDs
- small lists of products/articles/issues
- potentially large case/history result sets

The environment explicitly tracks observation size using:
- `info["observation_size"] = len(result)`

### C. Query logical complexity
Observed query patterns include:
- date-range filters
- `IN (...)` filters
- object relationship traversal
- `GROUP BY` and aggregation
- SOSL full-text search

## Summary table

| Question | Answer |
|---|---|
| How many agents? | Usually 1 task agent; interactive mode adds 1 simulated user; scoring uses 1 evaluator |
| Parallel agents querying DB? | Not inside one task; parallelism is across background experiment processes |
| How big is DB? | About 8.1 MB, 34.6 MB, and 57.4 MB local DB files (~100 MB total visible snapshots) |
| How big is query? | Query strings are modest; result sizes vary by task and filter breadth |

## Architecture takeaway

The scaling story of CRMArena is **batch concurrency across many independent runs**, not multi-agent cooperative database exploration within a single task.
