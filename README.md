# CRMArena Architecture Notes

This repository contains architecture notes and flow mappings for the upstream `SalesforceAIResearch/CRMArena` project, focused on the agentic evaluation loop, GUI/API access path, scaling behavior, and code call graph.

## Docs

- `docs/01-agentic-scaffolding.md` — high-level agentic scaffolding and runtime layers
- `docs/02-visual-flow-mermaid.md` — Mermaid flowcharts for the end-to-end runtime
- `docs/03-gui-login-flow.md` — GUI login / API access mapping
- `docs/04-code-call-graph.md` — call graph from `run_tasks.py` down into agents/env/connector
- `docs/05-scaling-db-query-analysis.md` — analysis of agent count, concurrency, DB artifacts, and query characteristics
