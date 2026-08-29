---
name: spyderbat-agent-response-actions
description: >-
  Take live containment action on a monitored host through the Spyderbat Nano Agent — kill
  a pod, kill a process, or scan a container. Read this before any Spyderbat write that
  touches a running workload. Use only with explicit human authorization.
api: Spyderbat API
generated: '2026-08-29'
method: generated
source: openapi/spyderbat-openapi.json (all operationIds verified present)
operations:
  - OrgList
  - CanUserPerform
  - Objects
  - AgentKillPod
  - AgentKillProcess
  - AgentScanContainer
---

# Spyderbat agent response actions

These three operations do not change a record. They **kill a running pod, kill a running
process, or launch a scan on a live host**.

| Operation | Path | Consequence |
|---|---|---|
| `AgentKillPod` | `POST /api/v1/org/{orgUID}/response/agent/killpod/{podUID}` | **Irreversible** |
| `AgentKillProcess` | `POST /api/v1/org/{orgUID}/response/agent/killprocess/{processUID}` | **Irreversible** |
| `AgentScanContainer` | `POST /api/v1/org/{orgUID}/response/agent/scancontainer/{containerUID}` | Side-effecting |

## Hard rules

1. **Never call these without explicit, specific human authorization** naming the pod,
   process or container. "Contain the threat" is not authorization for a kill.
2. **There is no undo.** The Spyderbat contract exposes no restart, revive or rollback
   operation for a killed pod or process, and none is documented. Killing the wrong pod in
   a production namespace is an outage you cannot reverse through this API.
3. **There is no dry run.** Unlike `SuppressTrace`, these operations have no `preview`
   mode. The only rehearsal available is resolving the target and reading it back to the
   human first.
4. **There is no idempotency key.** If a call times out you cannot safely retry it — you
   have no way to tell a lost response from a lost request. Re-read state instead of
   re-firing, and tell the human the call is in an unknown state.

## Steps

1. **Resolve the org** — `OrgList`.
2. **Check permission first** — `CanUserPerform`
   (`POST /api/v1/rbac/capabilities/`). Do not discover a 403 by attempting a kill.
3. **Resolve and confirm the target** — `Objects`
   (`POST /api/v1/org/{orgUID}/objects/`) turns a `pod_uid` into pod name, namespace and
   node. **Read the resolved name, namespace and node back to the human and get explicit
   confirmation on that exact target.**
4. **Act** — call the operation for the confirmed target only.
5. **Verify** — re-query the workload to confirm the outcome, and report what changed.

## Rate limiting

These are the **only three operations in the entire Spyderbat contract that declare an
HTTP 429**. No limit, window, `Retry-After` or `RateLimit-*` header is published, so on a
429 you have no documented retry interval. Back off, do not tighten a retry loop, and
report the throttle to the human rather than working around it.

## Errors

- **429 too many requests** — throttled; see above.
- **403 permission denied** — the key's role does not permit response actions.
- **400** — invalid agent ID, or an invalid pipeline/agent combination.
- **404 agent action not found** — the target no longer exists; re-resolve before retrying.
