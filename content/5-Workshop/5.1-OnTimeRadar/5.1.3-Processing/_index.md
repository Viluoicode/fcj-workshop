---
title: "Receiving and Processing Events"
date: 2026-11-23
weight: 3
chapter: false
pre: " <b> 5.1.3. </b> "
---

# Receiving and Processing Events

## One Adapter per Carrier

The three carriers use different payloads. The formats are simulated in this project, not the carriers' official specifications:

| Carrier | Example field | Example value |
|---|---|---|
| GHN | text status | `delivering` |
| GHTK | numeric status | `status_id = 45` |
| J&T | scan type | `scanType = DELIVERY` |

Each adapter is one class that turns its carrier's payload into the same event: carrier, tracking code, common status, time it happened, promised delivery date, and region. Adding a carrier means adding one class; the rest of the pipeline does not change.

The adapters also check their input. A tracking code must match `^[A-Za-z0-9._-]{1,64}$`, because it becomes part of a DynamoDB key, a URL, and an S3 path. A payload with the wrong shape is answered with **400**, so the carrier does not keep retrying it.

## The Eight Statuses

```text
CREATED → PICKED_UP → IN_TRANSIT (repeats between hubs) → OUT_FOR_DELIVERY → DELIVERED
                                                 ↕
                                          FAILED_ATTEMPT → RETURNED
CREATED → CANCELLED
```

## Keeping Each Shipment in Order

The API Lambda puts each event on an **SQS FIFO** queue with `MessageGroupId` set to the shipment, then answers `202` straight away. FIFO delivers the events of one shipment strictly in order, while other shipments are processed in parallel.

The **eventId** is a hash of the event's content. If a carrier sends the same webhook twice, both copies get the same ID. SQS drops a copy that arrives within five minutes, and DynamoDB catches any later one.

## Deciding What an Event Means

The processor compares each event with what it already knows about the shipment:

| Situation | Outcome | Effect |
|---|---|---|
| Same `eventId` seen before | `Duplicate` | nothing is written |
| Happened earlier than the latest applied event | `Stale` | kept in history, status unchanged |
| Shipment already finished (delivered, returned, cancelled) | `Rejected` | kept in history, status unchanged |
| Moves backwards, e.g. in transit after out for delivery | `Rejected` | kept in history, status unchanged |
| Anything else | `Applied` | status updated |

A shipment counts as **late** when the delivery date is after the promised date.

## Writing the Result

The status item and the history item are written to **DynamoDB** in a single transaction, with a version check, so they can never disagree and two writers cannot overwrite each other. The table uses one partition per shipment, so a single query returns the status together with the whole history.

The processor then:

- appends the event to `s3://<data bucket>/events/dt=YYYY-MM-DD/` as JSON lines, for Athena;
- publishes `Deliveries` and `LateDeliveries` metrics to CloudWatch, per carrier.

The SQS trigger uses **partial batch failure**. If one message fails, that message and every later message in the same group go back to the queue, and nothing after the failure is processed out of order. After five failed attempts, a message moves to the dead-letter queue, which triggers an alarm.

## The Replayer

Every 15 minutes EventBridge Scheduler runs the Replayer Lambda. It reads the next 20 Olist orders from S3, turns each order into a series of carrier webhooks, and sends them to the public API, exactly like a real carrier would.

A cursor in DynamoDB remembers where the replay stopped. The cursor moves forward only when no webhook failed for a temporary reason (throttling, 5xx, or network), so the next run sends the same orders again. Copies that were already processed come back as `Duplicate`.

## Try It

```powershell
$api = "<ApiUrl>"
$h   = @{ "X-Webhook-Secret" = "<WebhookSecret>" }
Invoke-RestMethod -Method Post "$api/carriers/ghn/webhook" -Headers $h -ContentType "application/json" -InFile tools/sample-webhook-ghn.json
Invoke-RestMethod -Method Post "$api/carriers/ghn/webhook" -Headers $h -ContentType "application/json" -InFile tools/sample-webhook-ghn-delivered.json
Start-Sleep 5
Invoke-RestMethod "$api/shipments/ghn/GHNTEST001" -Headers $auth | ConvertTo-Json -Depth 5
```

The shipment ends as `Delivered` and `isLate = true`, with two `Applied` events. Sending the first webhook again adds nothing, and a wrong secret returns 401. `$auth` holds a Cognito token (see 5.1.5).

> 📷 *Screenshots to add: the SQS FIFO queue and its DLQ in the console, and one shipment item in DynamoDB.*
