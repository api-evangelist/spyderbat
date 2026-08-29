---
name: spyderbat-suppress-noisy-detections
description: >-
  Suppress a benign Spyderbat Spydertrace so it stops filling the queue — preview the
  suppression policy and its scope first, then apply it. Use when a detection is confirmed
  benign (a health probe, a known job, expected operator activity) and should be tuned out.
api: Spyderbat API
generated: '2026-08-29'
method: generated
source: >-
  openapi/spyderbat-openapi.json (all operationIds verified present) and
  https://docs.spyderbat.com/installation/mcp.md
operations:
  - OrgList
  - Search
  - Results
  - SuppressTrace
  - CanUserPerform
---

# Suppress a noisy Spyderbat detection

## Confirm it is benign first

Suppression hides future matching activity. Establish what the trace actually did — run
the triage skill, walk the process tree, and identify the workload — before suppressing
anything. A suppression applied too widely is how a real detection gets silenced.

## Steps

1. **Resolve the org** — `OrgList`, then use its `org_uid`.
2. **Check you may suppress** — `CanUserPerform`
   (`POST /api/v1/rbac/capabilities/`). A Read Only key cannot suppress.
3. **Identify the trace** — via `Search` / `Results` against `model_spydertrace`, or from
   the triage step that surfaced it.
4. **Preview** — `SuppressTrace` (`POST /api/v1/org/{orgUID}/suppress/trace`) with
   `preview` set. Spyderbat's documentation describes `preview=true` rendering the
   generated policy, its warnings and its scope without applying it. **Always preview.**
   This is the only dry-run affordance in the entire Spyderbat API.
5. **Read the scope in the preview.** Confirm the policy is scoped to the narrowest thing
   that is genuinely benign — a namespace or a cluster, not the whole org — before
   applying.
6. **Get human confirmation, then apply** — re-call `SuppressTrace` without preview.

## Reversibility — read before applying

Suppression produces a managed policy object, but the **public contract exposes no
explicit unsuppress operation**, and no reversal window is documented anywhere. Treat an
applied suppression as something a human must go and undo in the console or through
Guardian policy management (`spyctl`), not something you can cleanly revert through this
API. Never apply a suppression on your own initiative.

## Related surfaces

Suppression sits next to Guardian policies and rulesets (`AnalyticsPolicy`,
`AnalyticsRuleset`, 10 operations), which are the documented long-term tuning surface and
are managed through `spyctl` — see <https://docs.spyderbat.com/concepts/suppression>.
