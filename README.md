# Runloop OSS

## The safest way to let AI touch your infrastructure

Runloop is an open-source execution layer for infrastructure agents. It gives developers a safe, auditable, reversible way to let AI and software operate cloud infrastructure without turning production into a trust fall.

Runloop is **not** the intelligence layer.
It is the **execution layer**.

- **Runloop runs**
- **Rivora decides**

That separation is the point.

---

## Why this exists

AI can already suggest what to do.

What teams still do **not** trust is letting AI *execute* inside real infrastructure:
- restart production services
- roll back deploys
- scale workloads
- patch misconfigurations
- rotate or update resources
- run repair workflows during incidents

The gap is not "can AI reason?"
The gap is "what is the safest way to let AI touch prod?"

That is what Runloop solves.

---

## What Runloop does

Runloop provides:
- **Plan generation** before execution
- **Explicit approval gates**
- **Scoped execution permissions**
- **Audit trails for every action**
- **Rollback hooks and verification**
- **Extensible adapters for cloud providers and platforms**
- **A CLI and SDK developers can actually use**

### Core execution loop

> Plan → Approve → Execute → Verify → Rollback (if needed)

This is the product.

---

## What Runloop does not do

Runloop is intentionally **not**:
- a predictive reliability engine
- an RCA product
- an observability platform
- a control plane for your entire cloud
- an LLM orchestration framework in disguise

Those are separate layers.

---

## Relationship to Rivora

Runloop is the open execution layer.

Rivora is the closed intelligence layer that:
- understands infrastructure context
- predicts issues
- reasons about risk
- recommends or generates plans
- learns from execution outcomes

### Boundary

- **Runloop:** "Here is the safe way to execute this plan."
- **Rivora:** "Here is what should happen, why, and how risky it is."

That separation lets you open source the hands without giving away the brain.

---

## Example workflows

### 1. Restart a failing service
```bash
runloop connect aws
runloop connect kubernetes

runloop plan service restart api   --env prod   --cluster core-us-east-1   --reason "health check failures on 3/5 replicas"

runloop apply plan-0142.json
runloop logs exec_01JX9Y1T4M7B3F
```

### 2. Scale a deployment during an incident
```bash
runloop plan deploy scale checkout   --namespace prod   --replicas 8   --reason "sustained CPU saturation > 85%"
```

### 3. Roll back a bad deploy
```bash
runloop plan deploy rollback web   --provider vercel   --target previous-stable
```

### 4. Re-run a failed GitHub Actions job
```bash
runloop plan workflow rerun api-ci   --repo acme/platform   --run-id 182771
```

---

## Example CLI output

### Plan output
```text
$ runloop plan service restart api --env prod --cluster core-us-east-1

✔ Loaded adapter: kubernetes
✔ Resolved target: deployment/api
✔ Generated plan: plan_01JX9Y1Q8JX2A

Plan Summary
────────────────────────────────────────────
Action            restart-service
Target            deployment/api
Environment       prod
Cluster           core-us-east-1
Risk Level        medium
Dry Run           true
Verification      rollout status + health checks
Rollback          reapply previous replica set
Approval Needed   yes

Steps
1. Confirm deployment exists
2. Capture current rollout state
3. Restart deployment/api
4. Wait for rollout to finish
5. Verify readiness probes
6. Verify service error rate stabilizes

Use:
  runloop approve plan_01JX9Y1Q8JX2A
  runloop apply plan_01JX9Y1Q8JX2A
```

### Apply output
```text
$ runloop apply plan_01JX9Y1Q8JX2A

✔ Approval check passed
✔ Scoped credentials verified
✔ Execution started: exec_01JX9Y1T4M7B3F

[1/6] Confirm deployment exists ................ done
[2/6] Capture current rollout state ............ done
[3/6] Restart deployment/api ................... done
[4/6] Wait for rollout to finish ............... done
[5/6] Verify readiness probes .................. done
[6/6] Verify service error rate stabilizes ..... done

Execution Result
────────────────────────────────────────────
Status            success
Duration          54s
Verification      passed
Rollback Needed   no
Logs              runloop logs exec_01JX9Y1T4M7B3F
```

### Failure + rollback output
```text
$ runloop apply plan_01JX9Y2M2Q21C

✖ Verification failed: latency remained above threshold after rollout
→ Starting rollback sequence

[rollback 1/2] Reapply previous replica set .... done
[rollback 2/2] Verify service recovery ......... done

Execution Result
────────────────────────────────────────────
Status            rolled_back
Cause             post-action verification failed
Rollback          successful
Next Step         inspect recent deploy + config drift
```

---

## Why not just use Terraform?

Terraform is excellent for declarative provisioning.
Runloop is for **operational execution**, especially during live system workflows.

Terraform is strongest for:
- desired infrastructure state
- provisioning and change management
- repeatable infra definitions

Runloop is strongest for:
- safe operational actions
- incident-driven steps
- approvals + verification + rollback
- cloud and platform actions that are procedural, not just declarative

---

## Why not just use scripts?

Scripts are powerful, but they are usually:
- ad hoc
- inconsistent
- hard to audit
- permission-heavy
- unsafe by default
- difficult to standardize across teams

Runloop turns scripts into:
- typed actions
- reviewed plans
- scoped execution
- repeatable workflows
- logs and rollback semantics

---

## Why not just use generic agent frameworks?

Most generic agent frameworks optimize for:
- tool orchestration
- multi-step reasoning
- LLM loops
- broad automation

Runloop optimizes for:
- explicit infrastructure actions
- safe execution semantics
- dry-run and approval boundaries
- production-grade traceability

It is much narrower on purpose.

---

## Developer experience goals

Runloop should feel:
- clear
- boring in the best way
- safe by default
- composable
- easy to extend

The product experience should make a developer think:

> "I would actually let this near production because I can see exactly what it is going to do."

---

## Monorepo structure

```text
runloop/
├─ apps/
│  └─ cli/
├─ packages/
│  ├─ core/
│  ├─ sdk/
│  ├─ types/
│  ├─ policy/
│  ├─ logger/
│  ├─ crust-cli/
│  └─ ui-formatters/
├─ adapters/
│  ├─ aws/
│  ├─ kubernetes/
│  ├─ vercel/
│  └─ github/
├─ examples/
│  ├─ restart-service/
│  ├─ rollback-vercel-deploy/
│  └─ rerun-github-actions/
├─ docs/
│  ├─ workflows/
│  ├─ comparison/
│  └─ show-hn/
└─ .github/
```

---

## CrustJS CLI direction

Runloop should use **CrustJS** for the CLI because:
- it aligns with your preference for a modern TS CLI developer experience
- it supports a clean command structure
- it helps keep the CLI ergonomic, typed, and composable
- it fits the OSS-dev-tool vibe better than a bloated framework stack

---

## Launch wedge

The best initial wedge is not "AI runs your cloud."

It is:

> **Runloop is the safest way to let AI touch your infrastructure.**

That is sharper, more believable, and more installable.

---

## Suggested first adapters

1. **Kubernetes**
   - restart deployment
   - scale deployment
   - get rollout status
   - cordon/drain node
2. **AWS**
   - ECS service restart/update
   - Lambda config adjustment
   - RDS safety checks
3. **Vercel**
   - list deploys
   - roll back deploy
   - inspect deploy metadata
4. **GitHub**
   - rerun workflow
   - fetch workflow status
   - open incident follow-up issue

---

## Suggested first milestone

### Milestone 1 — Trustworthy execution core
- typed action model
- plan/apply/logs flow
- dry-run support
- approval system
- execution logs
- rollback contract
- kubernetes adapter
- GitHub adapter

That is enough for a real dev-tool launch.

---

## License and community direction

A permissive license makes the most sense if the goal is distribution and trust.
The moat should not be the runtime.
The moat should be the intelligence layer built around it.

---

## Status

This repo package is a launch-grade design/spec bundle for the OSS layer.
It is meant to help define:
- product scope
- architecture
- DX
- launch story
- repo scaffolding
- examples
