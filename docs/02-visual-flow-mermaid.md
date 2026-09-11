# 02 — Visual Flow (Mermaid)

## End-to-end evaluation flow

```mermaid
flowchart TD
    A[Researcher / Operator] --> B[run_tasks.py]
    B --> C[Load .env via load_dotenv]
    B --> D[Select org_type, strategy, model, task_category]
    D --> E[data/assets.py loads tasks + schema]
    D --> F{Strategy}

    F -->|react / act| G[ChatAgent + ChatEnv]
    F -->|tool_call / tool_call_flex| H[ToolCallAgent + ToolEnv]
    F -->|interactive react| I[ChatAgent + InteractiveChatEnv]

    G --> J[SalesforceConnector]
    H --> J
    I --> J

    J --> K[Authenticate to Salesforce org]
    K --> L[Agent loop starts]

    L --> M[Agent generates SOQL/SOSL or tool call]
    M --> N[Environment step executes action]
    N --> O[Salesforce returns records or error]
    O --> P[Observation returned to agent]
    P --> Q{Done?}
    Q -->|No| L
    Q -->|Yes| R[Evaluator scores answer]
    R --> S[Checkpoint JSON written]
```

## Highlighted action-observation loop

```mermaid
sequenceDiagram
    participant A as Task Agent
    participant E as Environment
    participant C as SalesforceConnector
    participant S as Salesforce Org
    participant V as Evaluator

    A->>E: generate action
    Note over A,E: one active task-solving agent per task
    E->>C: execute query / tool
    C->>S: run SOQL/SOSL
    S-->>C: records / error
    C-->>E: normalized observation
    E-->>A: observation
    loop until answer
        A->>E: next action
        E->>C: next query/tool
        C->>S: execute
        S-->>C: result
        C-->>E: observation
        E-->>A: observation
    end
    A->>E: final answer
    E->>V: score answer
```

## Parallelism model

```mermaid
flowchart LR
    A[Shell script loops over models/tasks/strategies] --> B[Background process 1]
    A --> C[Background process 2]
    A --> D[Background process N]

    B --> E[One agent loop]
    C --> F[One agent loop]
    D --> G[One agent loop]

    E --> H[Sequential query execution]
    F --> I[Sequential query execution]
    G --> J[Sequential query execution]
```

## GUI/API access subflow

```mermaid
flowchart TD
    A[Researcher wants org access] --> B{Access mode}
    B -->|GUI| C[Request GUI access by email]
    B -->|API| D[Copy .env.example to .env]
    D --> E[Set Salesforce credentials + model keys]
    E --> F[SalesforceConnector.sf_auth(org_type)]
    F --> G[simple_salesforce.Salesforce(...)]
    G --> H[Authenticated session for evaluation runtime]
```
