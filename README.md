# Action-Specific Guardrails: From Content Safety to Tool-Execution Governance in Agentic AI Systems

> John Cheung  
> College of Professional and Continuing Education


## Manuscript PDF

[Read the full paper (PDF)](https://github.com/johncheungmk/Action-Specific-Guardrails/blob/main/AGF.pdf)

## Overview

Modern AI systems are moving from conversational retrieval-augmented generation (RAG) toward **agentic AI**: systems that can call tools, execute commands, update records, send messages, deploy services, and interact with cloud infrastructure.

Traditional AI guardrails usually focus on what the model **says**:

- harmful content detection
- prompt-injection detection
- groundedness checking
- personally identifiable information filtering
- hallucination reduction
- output validation

However, tool-using agents create another class of risk: the AI may perform an unsafe **action**, even when its text output appears harmless or compliant.

This article argues that agentic AI systems need a separate runtime control layer:

> **Content safety checks what the AI says. Action-specific guardrails check what the AI does.**

## Core Idea

An AI agent should not directly execute high-impact tool calls simply because the model proposes them.

Instead, every proposed action should be classified and mediated before execution:

```text
User request
  -> Agent or skill runtime
  -> Proposed tool/action
  -> Action policy evaluation
  -> Allow / Ask / Approve / Sandbox / Dry-run / Deny
  -> Scoped credential issuance
  -> Tool/API execution
  -> Audit trace and rollback record
```

The paper proposes an **Action Guardrails Framework (AGF)**: a runtime policy and enforcement architecture that evaluates proposed tool calls according to:

- action type
- target resource
- environment, such as local, staging, or production
- reversibility
- blast radius
- credential scope
- user and agent identity
- approval requirement
- rollback capability
- auditability

## Motivation

The paper uses the reported PocketOS incident as an illustrative case. In that incident, an AI coding agent reportedly deleted a production database volume despite written instructions that prohibited destructive behavior.

The broader lesson is not limited to one vendor, tool, or incident:

> Prompt instructions are not enforcement mechanisms.  
> A model should not hold credentials that allow actions it is not supposed to perform.

## Why RAG Guardrails Are Not Enough

RAG/content guardrails and action guardrails address different objects.

| Dimension | RAG / Content Guardrails | Action-Specific Guardrails |
|---|---|---|
| Primary object | Prompt, retrieved document, generated answer | Tool call, API call, command, workflow action |
| Main risk | Harmful output, hallucination, leakage | Unauthorized mutation, destructive action, side effect |
| Typical decision | Allow, block, redact, rewrite, cite | Allow, ask, approve, sandbox, dry-run, deny |
| Enforcement point | Before or after model generation | Before tool/API execution |
| Failure example | Ungrounded answer | Production database deletion |

A mature agentic AI platform needs both.

## Proposed Framework

The proposed AGF architecture contains:

1. **Agent or Skill Runtime**  
   Plans the task and proposes tool calls.

2. **Action Proposal Normalizer**  
   Converts shell commands, API requests, browser actions, workflow operations, and tool calls into a common action schema.

3. **Action Classifier**  
   Classifies risk using action type, resource, environment, data sensitivity, reversibility, and blast radius.

4. **Policy Engine**  
   Applies organization policy and returns a decision.

5. **Approval Gateway**  
   Presents high-risk actions to authorized humans with exact arguments and consequences.

6. **Credential Broker**  
   Issues short-lived, least-privilege credentials only after policy approval.

7. **Tool/API Adapter**  
   Executes only approved actions.

8. **Audit Trace and Rollback Manager**  
   Records action evidence and verifies rollback readiness where applicable.

## Action-Risk Taxonomy

The paper proposes a five-level action-risk taxonomy.

| Level | Category | Example | Default Decision |
|---|---|---|---|
| L0 | Read-only | Search FAQ, read logs, inspect schema | Allow |
| L1 | Local reversible | Create draft, edit local file, generate patch | Allow or ask |
| L2 | External reversible side effect | Create ticket, send test notification, restart staging service | Ask user |
| L3 | Production-impacting change | Deploy service, modify production config, update access group | Require approver |
| L4 | Destructive or irreversible | Delete database, delete volume, delete backup, wipe disk | Deny by default |

## Security Objectives

AGF is designed around five security objectives:

- **Complete mediation**: every state-changing path to a protected resource is checked before execution.
- **Least privilege**: the agent receives only the minimum authority needed for an approved action.
- **Non-bypassability**: shells, SDKs, HTTP clients, browser tools, and inherited credentials cannot bypass the action gateway.
- **Fail-closed uncertainty**: missing labels, ambiguous verbs, or unavailable policy services escalate or block high-risk actions.
- **Auditability and replay protection**: approvals and credentials are bound to exact action arguments, resource identifiers, and time windows.

## Example Policy

```yaml
version: "1.0"

default_decision: deny

risk_levels:
  L0:
    description: read_only
    default_decision: allow
  L1:
    description: local_reversible
    default_decision: allow
  L2:
    description: external_reversible
    default_decision: ask_user
  L3:
    description: production_impacting
    default_decision: require_approver
  L4:
    description: destructive_or_irreversible
    default_decision: deny

actions:
  database.select:
    risk: L0
    allowed_environments: [staging, production]
    credential_scope: read_only
    constraints:
      max_rows: 1000
      pii_redaction: true

  database.update_production:
    risk: L3
    allowed_environments: [production]
    approval: data_owner_approval
    require_backup_age_minutes_max: 60
    require_ticket: true

  volume.delete:
    risk: L4
    decision: deny

  email.create_draft:
    risk: L1
    decision: allow

  email.send_external:
    risk: L2
    approval: user_confirmation

  student_record.update:
    risk: L4
    decision: deny
```

## Example Enforcement Stack

A concrete AGF implementation may use:

- container or microVM sandboxing
- no ambient production credentials
- typed tool adapters
- identity-aware egress proxy
- API gateway with action-token validation
- short-lived credentials through STS, workload identity, managed identity, or scoped application tokens
- IAM/RBAC/ABAC enforcement at the target system
- tamper-evident audit traces
- approval records bound to signed action digests

The goal is to ensure that even if an agent proposes a forbidden action, it lacks both the route and the credential required to execute it.

## Evaluation Plan

The article proposes evaluating AGF against baselines such as:

- prompt-only guardrails
- local harness approval only
- IAM-only scoping
- detector-only agent guardrails
- commercial content guardrails without action mediation

Suggested tests include:

- seeded destructive-action tests
- prompt-injection-to-action tests
- credential-scope tests
- approval-bypass tests
- failure injection
- policy misconfiguration tests
- performance and approval-latency measurements

Suggested metrics include:

- unauthorized action block rate
- destructive action escape rate
- missed escalation rate
- false block rate
- approval latency
- privilege surface area
- action trace completeness
- rollback readiness
- policy bypass detection rate

## Application Areas

The framework is relevant to:

- AI coding agents
- enterprise AI assistants
- skill-based chatbots
- institutional student/staff support bots
- IT operations agents
- business workflow agents
- MCP/tool-connected agents
- cloud automation assistants
- email and communication agents
- systems that combine RAG with tool execution

## Repository Contents

Suggested repository structure:

```text
.
├── README.md
├── paper/
│   ├── AGF.pdf
│   └── manuscript.docx
├── examples/
│   ├── action-policy.yaml
│   ├── action-proposal-example.json
│   └── blackboard-support-skill-policy.yaml
├── schemas/
│   ├── action-proposal.schema.json
│   └── action-policy.schema.yaml
└── evaluation/
    ├── seeded-actions.md
    ├── baselines.md
    └── metrics.md
```

## Suggested Citation

```bibtex
@article{cheung2026actionguardrails,
  title   = {Action-Specific Guardrails: From Content Safety to Tool-Execution Governance in Agentic AI Systems},
  author  = {Cheung, John},
  year    = {2026},
  note    = {Manuscript submitted to F1000Research}
}
```

Update the citation after publication with the DOI and final journal metadata.

## Key Message

System prompts and skill instructions are important, but they are not sufficient protection for tool-capable agents.

For high-impact AI systems:

> Do not only ask whether the answer is safe.  
> Ask whether the proposed action is authorized.

## License

Add the appropriate license for this repository.

Suggested options:

- **CC BY 4.0** for manuscript and documentation
- **MIT License** or **Apache-2.0** for code, schemas, or example implementations

## Contact

John Cheung  
College of Professional and Continuing Education

