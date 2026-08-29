---
name: spyderbat-api-key-and-org-setup
description: >-
  Get a working Spyderbat API call — create an API key, find the org UID, understand the
  RBAC role the key inherits, and make the first authenticated request. Start here before
  any other Spyderbat skill.
api: Spyderbat API
generated: '2026-08-29'
method: generated
source: >-
  https://docs.spyderbat.com/tutorials/integrations/how-to-set-up-your-spyderbat-api-key-and-use-the-spyderbat-api.md
  and openapi/spyderbat-openapi.json (all operationIds verified present)
operations:
  - OrgList
  - OrgLoad
  - ListRoles
  - OrgListRole
  - CanUserPerform
  - SrcList
  - OrgTypeLimitActiveSources
---

# Spyderbat: key, org, first call

## 1. Create the key

In the Spyderbat console, open the account menu (top right) → **API Keys** → create a
named key. The key is masked in the list as `****xxxx`; retrieve the full value from the
**Example Usage** button (the middle of the three per-row icons), which shows it as the
Bearer token in a curl example. It starts with `eyJ` — it is a JWT.

Keys can carry an expiration date. Rotate before it lapses; expiry is a silent 403 later.

## 2. Understand what the key can do

Spyderbat uses RBAC. **A key is bound to one user account and inherits that user's role in
each organization it belongs to.** Two roles are documented — **Admin** and **Read
Only** — plus named capabilities such as `org:ManageSiemForwarding` for specific features.

Treat the key like a password: it carries the full authority of the person who made it.

## 3. Find the org UID

Every meaningful call needs it — 192 of 197 operations take an `orgUID` path parameter.

- **From the API:** `OrgList` (`GET /api/v1/org/`) returns every org the key can see, with
  names and UIDs. This is the reliable way.
- **From the console URL:** the segment between `org/` and the next `/`, e.g.
  `https://app.spyderbat.com/app/org/P6V31v0uIG5dtqXTHLsd/dashboard` → `P6V31v0uIG5dtqXTHLsd`.
  The API Keys page (`/app/user/apikey`) does **not** contain the org UID — navigate to
  the Dashboard instead.
- **From the console:** the Example Usage dialog has an org dropdown and a Copy org ID
  button.

> **The failure mode to guard against:** using the wrong org UID produces **zero results
> with no error message**. If a query comes back empty, re-confirm the org UID with
> `OrgList` before concluding the environment is quiet.

## 4. First calls

```
curl https://api.prod.spyderbat.com/api/v1/org/ \
  -H "Authorization: Bearer API_key"

curl https://api.prod.spyderbat.com/api/v1/org/Org_id/source/ \
  -H "Authorization: Bearer API_key"
```

The second is `SrcList` — the monitored sources (machines, VMs, cluster nodes) in the org.

## 5. Test permissions before acting

`CanUserPerform` (`POST /api/v1/rbac/capabilities/`) checks whether the caller may perform
an action. Use it before any write rather than discovering a 403 mid-flow — 403 is
declared on 188 of the 197 operations and is by far the most common failure of this API.
`ListRoles` and `OrgListRole` show what roles exist and who holds them.

## 6. Know your quotas

`OrgTypeLimitActiveSources` and `OrgTypeLimitOrgRoles` return the plan-derived ceilings for
the org. Spyderbat publishes no public pricing page, so these runtime endpoints are the
only readable statement of what the account is entitled to.

## Reference

- Interactive reference (ReDoc): <https://api.spyderbat.com/openapi>
- Raw spec: <https://api.spyderbat.com/openapi.json>
- Docs index for agents: <https://docs.spyderbat.com/llms.txt> — and every docs page is
  available as Markdown by appending `.md` to its URL.
- Note: the docs link to `https://api.spyderbat.com/openapi.pdf` as "the API
  documentation". Despite the extension it serves HTML that mounts the ReDoc viewer, not
  a PDF.
