---
name: spyderbat-triage-spydertraces
description: >-
  Triage the Spyderbat alert queue — find the highest-scoring Spydertraces in a time
  window, pull their contents, and decide which are real. Use when asked to review
  Spyderbat detections, work the queue, or investigate runtime security activity on
  Linux, container or Kubernetes workloads.
api: Spyderbat API
generated: '2026-08-29'
method: generated
source: >-
  openapi/spyderbat-openapi.json (all operationIds verified present) and
  https://docs.spyderbat.com/installation/mcp.md
operations:
  - OrgList
  - Schema
  - Validate
  - Search
  - Results
  - Objects
  - ObjectsWalkGraph
---

# Triage Spyderbat Spydertraces

A Spydertrace bundles related processes, connections and red flags into one scored causal
unit. Scores are the triage queue.

## Before you start

- Base URL: `https://api.prod.spyderbat.com/api/v1`
- Auth: `Authorization: Bearer <api-key>` on every request. The key inherits the RBAC role
  of the user who created it.
- **Resolve the org first.** Call `OrgList` (`GET /api/v1/org/`) and use a returned
  `org_uid`. Spyderbat's own docs warn that a wrong org UID returns **zero results with no
  error** — an empty result is not proof of a quiet environment until you have confirmed
  the org UID.

## Steps

1. **List organizations** — `OrgList`. Pick the `org_uid` you were asked about; if there
   is more than one and the request is ambiguous, ask rather than guessing.
2. **Check what is searchable** — `Schema`
   (`GET /api/v1/org/{orgUID}/search/schema/`) returns the schemas available to this org.
   The three named in Spyderbat's documentation are `model_spydertrace`,
   `model_connection` and `event_redflag`.
3. **Validate the query before spending a job** — `Validate`
   (`POST /api/v1/org/{orgUID}/search/validate/`), or `Parse Search Query` to see how
   Spyderbat parses it. Operators and fields are documented at
   <https://docs.spyderbat.com/reference/search/search-operators> and
   <https://docs.spyderbat.com/reference/search/search-fields>.
4. **Submit the search** — `Search` (`POST /api/v1/org/{orgUID}/search/`) against
   `model_spydertrace`, filtered on `score` and bounded by `start`/`end`. Spyderbat's own
   triage example uses `score > 50` over the last 24 hours.
5. **Collect results** — `Results` (`POST /api/v1/org/{orgUID}/search/{jobID}`). This is a
   submit-then-poll job API, not a synchronous query; poll until the job completes.
   `ObjectsStopQuery` cancels a job you no longer need.
6. **Open the top trace** — search `event_redflag` and `model_connection` for the same
   time window and identifiers to assemble the processes, connections and red flags behind
   the trace. Walk parent-child process relationships via `ppuid`.
7. **Attribute the workload** — `Objects` (`POST /api/v1/org/{orgUID}/objects/`) resolves a
   `pod_uid` into pod name, namespace and node. `ObjectsWalkGraph` walks the causal graph.

## Narrowing

Broad searches return large result sets. Narrow the time window and filter hard before
widening. Red flags carry MITRE ATT&CK technique identifiers (for example `TA0004.T1548`
for `root_shell`) — use them to group related findings.

## Errors you will hit

- **403 permission denied** — the dominant failure of this API, declared on 188 of 197
  operations. Check the key's role in this org. `CanUserPerform`
  (`POST /api/v1/rbac/capabilities/`) tests an action before you attempt it.
- **400 invalid input parameters** — the response carries a `ValidationError` naming
  `field`, `property` and the failed validation `tags`.
- **404 not found** — verify the resource UID; verify the org UID separately, because a
  wrong org returns empty rather than 404.
- There are **no 5xx responses declared** anywhere in this contract, so treat any
  server-side failure as undocumented and surface it rather than retrying blindly.

## Conventions that matter here

- No `Idempotency-Key` is supported anywhere in this API. Reads are safe to retry; see the
  response-actions skill before retrying anything that writes.
- No request-id or correlation header exists, so there is no id to quote to support.
- Pagination, where present, is `page` / `page_size` with `sort_by` and `reversed`.
