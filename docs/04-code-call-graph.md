# 04 — Code Call Graph

## Top-down call graph from `run_tasks.py`

```text
run_tasks.py
└── run()
    ├── choose TASKS_NATURAL + SCHEMA from crm_sandbox/data/assets.py
    ├── filter selected_tasks by task_category
    ├── load existing checkpoint JSON if reuse_results
    ├── choose runtime strategy
    │   ├── ChatEnv(...) for react/act non-interactive
    │   ├── InteractiveChatEnv(...) for react interactive
    │   └── ToolEnv(...) for tool_call / tool_call_flex
    ├── for each task
    │   ├── choose agent_type = internal/external
    │   ├── instantiate ChatAgent(...) or ToolCallAgent(...)
    │   ├── agent.act(env, idx)
    │   ├── collect reward, trajectory, usage
    │   └── append result to checkpoint JSON
    └── print start/end timestamps
```

## Chat-agent path

```text
ChatAgent.act(env, index)
├── env.reset(task_index=index)
├── self.reset({query, metadata})
├── while current_agent_turn < max_turns
│   ├── litellm.completion(...)
│   ├── message_action_parser(...)
│   ├── env.step(action)
│   ├── append assistant/user observation messages
│   └── break when done
└── return reward
```

## Tool-calling path

```text
ToolCallAgent.act(env, index)
├── env.reset(task_index=index)
├── self.reset({query, metadata})
├── for each turn
│   ├── chat_completion_request(..., tools=self.tools)
│   ├── message_action_parser(...)
│   ├── env.step(action)
│   ├── append tool observations
│   └── break when done
└── return reward
```

## Environment path

### ChatEnv

```text
ChatEnv.step(action)
├── if action == execute
│   ├── SalesforceConnector.run_query(...)
│   └── return observation
├── if action == respond
│   ├── calculate_reward()
│   └── done = True
└── return observation, reward, done, info
```

### InteractiveChatEnv

```text
InteractiveChatEnv.reset(task_index)
├── ChatEnv.reset(task_index)
└── LLMUserSimulationEnv.reset(instruction, persona)

InteractiveChatEnv.step(action)
├── if execute
│   └── SalesforceConnector.run_query(...)
├── if respond
│   ├── LLMUserSimulationEnv.step(...)
│   ├── if ###STOP###
│   │   └── calculate_reward()
│   └── else continue dialogue
└── return observation, reward, done, info
```

### ToolEnv

```text
ToolEnv.step(action)
├── lookup tool in tools_dict
├── if action == respond
│   └── calculate_reward()
└── else invoke tool(**arguments, sf_connector=self.sf_connector)
```

## Connector path

```text
SalesforceConnector.__init__(org_type)
├── sf_auth(org_type)
└── simple_salesforce.Salesforce(...)

SalesforceConnector.run_query(query)
├── detect SOQL vs SOSL
├── preprocess query if needed
├── call sf.query_all(...) or sf.search(...)
├── normalize rows
└── return (result, status)
```

## Evaluator path

```text
Evaluator.evaluate(proposed_answer, gt_answer, reward_metric, task_name, action_trajectory)
├── exact_match
│   ├── direct compare or parse_answers(...)
├── fuzzy_match
│   └── get_all_metrics(...)
└── privacy_rejection
    └── compute_privacy_confidential_awareness_score(...)
```

## Key observation

The architecture is a classic **agent → environment → connector → data source → environment → agent** loop, with scoring attached after the final response.
