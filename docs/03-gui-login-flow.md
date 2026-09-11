# 03 — GUI Login Flow

## Context

The upstream repository README distinguishes between two access paths:

1. **GUI access** to the Salesforce org
2. **API access** used by the benchmark runtime

The runtime path used in code is the **API path** through `SalesforceConnector` in `crm_sandbox/env/connect_sandbox.py`.

## GUI login flow

```text
Researcher wants org access
        ↓
Determine if GUI access is needed
        ↓
README states public GUI access is no longer directly available
        ↓
Researcher emails required details
  - First Name
  - Last Name
  - Email
  - Which org(s) they want access to
        ↓
Salesforce / repo maintainers grant access manually
        ↓
Researcher logs in through the Salesforce GUI
```

## API login flow used by code

```text
Copy .env.example to .env
        ↓
Provide org credentials / provider API keys
        ↓
run_tasks.py calls load_dotenv()
        ↓
Environment creates SalesforceConnector(org_type=...)
        ↓
SalesforceConnector.sf_auth(org_type) selects credentials
        ↓
simple_salesforce.Salesforce(username, password, security_token)
        ↓
Authenticated API session
        ↓
Agent queries org data through SOQL/SOSL
```

## Important distinction

The GUI login screenshot belongs to the **human access/setup surface**, not to the inner agent loop.

That means the screenshot should be placed in architecture diagrams as:
- a **pre-runtime access step**, or
- an **optional manual inspection path**

and not as part of:
- `agent generates query`
- `run SOQL/SOSL`
- `records return`
- `agent final answer`

## Recommended placement in architecture visuals

```text
Human / Researcher lane:
  GUI access request / login
  API credential setup
  launch experiments

System lane:
  agent runtime
  environment mediation
  SalesforceConnector
  evaluator
```

## Files involved

- `README.md`
- `.env.example`
- `crm_sandbox/env/connect_sandbox.py`
- `run_tasks.py`
