---
name: spyderbat-forward-events-to-siem
description: >-
  Wire Spyderbat detections into a SIEM, log platform or webhook endpoint — enable SIEM
  forwarding on a saved query and consume the forwarded-events iterator, or configure
  notification targets for email, Slack, PagerDuty and generic webhooks.
api: Spyderbat API
generated: '2026-08-29'
method: generated
source: >-
  openapi/spyderbat-openapi.json (all operationIds verified present),
  https://docs.spyderbat.com/concepts/integrations/siem-forwarding.md,
  https://docs.spyderbat.com/concepts/notifications/notification-targets.md
operations:
  - OrgList
  - SavedQueryList
  - SavedQueryCreate
  - SavedQueryShowAdditionalSettings
  - SavedQueryUpdate
  - ForwardEvents
  - NotificationTargetWebhookCreate
  - NotificationTargetSlackCreate
  - NotificationTargetPagerDutyCreate
  - NotificationTargetEmailCreate
  - NotificationSettingsInitialize
  - NotificationSettingsSet
  - NotificationSettingsTest
  - NotificationSettingsEnable
---

# Get Spyderbat events out to a SIEM or a webhook

Spyderbat has two distinct outbound paths, and they are configured differently.

## Path A — SIEM forwarding (pull)

Forwarding is attached to a **saved query**, not configured globally. Nothing forwards
until a saved query has the SIEM Forwarding toggle enabled.

1. **Resolve the org** — `OrgList`.
2. **Find or create the saved query** — `SavedQueryList`, or `SavedQueryCreate` with the
   schema and filter you want. Spyderbat's documented examples:
   - all Spydertraces — `model_spydertrace`, filter `*`
   - high-scoring only — `model_spydertrace`, filter `score > 50`
   - all security flags — `event_redflag`, filter `*`
   - high-severity only — `event_redflag`, filter `severity = "high"`
   - all connections — `model_connection`, filter `*`
3. **Enable forwarding** — the toggle lives in the query's Additional Settings;
   `SavedQueryShowAdditionalSettings` reads that surface and `SavedQueryUpdate` writes it.
   Requires the `org:ManageSiemForwarding` permission.
4. **Consume** — `ForwardEvents`
   (`GET /api/v1/org/{orgUID}/events/*iterator`), or deploy the provider's Event Forwarder
   (<https://github.com/spyderbat/event-forwarder>), which polls, enriches each record with
   a `runtime_details` host-metadata object, and writes to file, webhook, stdout or syslog.

**Two things that surprise people:** forwarding is **not retroactive** — enabling it
forwards only records created afterwards, so a gap before enablement is permanent. And
polling the iterator yourself means **you own the cursor**; the Event Forwarder exists
precisely to manage it.

## Path B — Notifications (push)

1. **Create a target** for the destination type: `NotificationTargetWebhookCreate` (a
   single generic HTTPS URL), `NotificationTargetSlackCreate` (a Slack hook URL),
   `NotificationTargetPagerDutyCreate` (a routing key), or
   `NotificationTargetEmailCreate` (a list of addresses).
2. **Attach notifications to a notifiable object** — usually a saved query or a custom
   flag — with `NotificationSettingsInitialize` then `NotificationSettingsSet`.
3. **Test before you rely on it** — `NotificationSettingsTest`
   (`POST /api/v1/org/{orgUID}/test_notification`) or `OrgTestNotificationTarget`. This is
   the only way to see a real delivery before an incident does.
4. **Enable** — `NotificationSettingsEnable`.

## What is NOT published

Be honest about these when wiring a receiver:

- **No webhook payload schema and no example payload.** Bodies are shaped by
  customer-authored notification templates, so the receiver contract is whatever your
  template emits.
- **No signature, shared secret or verification handshake.** A receiver cannot verify that
  a delivery came from Spyderbat. Put the endpoint behind your own auth.
- **No documented retry or delivery guarantee.**
- **No AsyncAPI document.** There is no machine-readable event contract to generate a
  consumer from.
