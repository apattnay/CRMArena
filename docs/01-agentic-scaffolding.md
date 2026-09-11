# 01 — Agentic Scaffolding

## Purpose

This document maps the runtime architecture of the CRMArena evaluation framework.

The system is best understood as an **agent evaluation harness** for Salesforce CRM tasks. It is not a multi-agent orchestration framework in the usual production sense; instead, it runs a **single task-solving agent per task**, optionally with a **simulated user** and a separate **evaluator model**.

## Runtime actors

### 1. Researcher / operator
Responsible for:
- configuring `.env`
- selecting model, strategy, org type
- launching batch scripts or `run_tasks.py`

### 2. Orchestrator
File:
- `run_tasks.py`

Responsibilities:
- parse CLI args
- choose dataset and schema by `org_type`
- choose runtime strategy
- instantiate environment and agent
- loop through tasks
- checkpoint results to JSON

### 3. Data loader
File:
- `crm_sandbox/data/assets.py`

Responsibilities:
- load benchmark datasets from Hugging Face
- expose original, B2B, B2C, and interactive task splits
- expose corresponding Salesforce object schemas

### 4. Task agent
Files:
- `crm_sandbox/agents/chat_agent.py`
- `crm_sandbox/agents/tool_call_agent.py`

Variants:
- `ChatAgent` for `react` and `act`
- `ToolCallAgent` for `tool_call` and `tool_call_flex`

Responsibilities:
- consume task query + schema + metadata
- decide next action
- either emit SOQL/SOSL execution requests or structured tool calls
- produce final answer

### 5. Environment
File:
- `crm_sandbox/env/env.py`

Variants:
- `ChatEnv`
- `ToolEnv`
- `InteractiveChatEnv`

Responsibilities:
- mediate between agent and Salesforce
- track action trajectory
- return observations
- terminate and score tasks

### 6. Salesforce connector
File:
- `crm_sandbox/env/connect_sandbox.py`

Responsibilities:
- read credentials from environment variables
- select auth by `org_type`
- connect through `simple_salesforce`
- run SOQL/SOSL
- normalize result/error payloads

### 7. Simulated user (interactive only)
File:
- `crm_sandbox/env/users.py`

Class:
- `LLMUserSimulationEnv`

Responsibilities:
- reveal task information gradually
- respond to clarification questions
- end the dialogue when the task has been satisfied

### 8. Evaluator
File:
- `crm_sandbox/env/env.py`

Responsibilities:
- parse final answers into task-specific entities
- compute exact/fuzzy/privacy-based scores
- compare agent answer to benchmark answer

## Layered architecture

```text
LAYER 1  Access / Setup
  README.md, .env.example, GUI/API org access

LAYER 2  Experiment Orchestration
  run_tasks.py, run_tasks_crmarena.sh, run_tasks_crmarena_pro.sh

LAYER 3  Benchmark Data
  crm_sandbox/data/assets.py

LAYER 4  Agent Runtime
  ChatAgent / ToolCallAgent

LAYER 5  Interaction Environment
  ChatEnv / ToolEnv / InteractiveChatEnv

LAYER 6  CRM Access
  SalesforceConnector

LAYER 7  Task World
  Salesforce org objects and records

LAYER 8  Evaluation
  Evaluator + metrics
```

## Core execution pattern

```text
researcher configures run
        ↓
run_tasks.py selects tasks/schema/strategy
        ↓
agent + environment instantiated
        ↓
SalesforceConnector authenticates to org
        ↓
agent generates query or tool call
        ↓
environment executes query/tool
        ↓
records returned as observation
        ↓
agent produces final answer
        ↓
evaluator scores output
        ↓
results written to checkpoint JSON
```

## Key architectural conclusion

CRMArena is architected around **single-agent task execution**, with optional **interactive user simulation** and a distinct **post-hoc evaluator**. Parallelism mainly comes from **multiple batch processes launched by shell scripts**, not from multiple agents collaborating inside one task.
