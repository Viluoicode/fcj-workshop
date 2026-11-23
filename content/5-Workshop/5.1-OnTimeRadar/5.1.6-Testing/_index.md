---
title: "Monitoring and Testing"
date: 2026-11-23
weight: 6
chapter: false
pre: " <b> 5.1.6. </b> "
---

# Monitoring and Testing

## Alarms

| Alarm | Fires when | Why it is built this way |
|---|---|---|
| `LateDeliveryAlarm` | late deliveries are 20% or more of deliveries in 2 of 3 consecutive 15-minute windows, counting only windows with at least 10 deliveries | The first version fired on "5 late per hour" and flipped three times in one morning as traffic grew (see 5.1.8). A rate that must last is quieter and still catches real problems |
| `DlqAlarm` | the dead-letter queue holds at least one message | A message that failed five times needs a person to look at it |

Both alarms email through SNS, once when they go to ALARM and once when they return to OK. The late-delivery email links to the "needs attention" page. The trade-off is that a real problem is reported after about 30–45 minutes instead of at once.

All Lambda logs are kept in CloudWatch Logs for 14 days.

> 📷 *Screenshots to add: the two alarms in CloudWatch, and an alarm email (hide the address).*

## Daily Check

One read-only command checks the whole system in about 40 seconds:

```powershell
.\tools\daily-check.ps1 -SaveTo docs\evidence
```

Result on 23 November 2026 at 18:16:

| # | Check | Result |
|---|---|---|
| 1 | CloudFormation stack | `UPDATE_COMPLETE` |
| 2 | Replayer schedule | `ENABLED` |
| 3 | Replayer in the last hour | 4 runs, 404 events sent, 0 failed; 13,580 of 98,202 orders replayed |
| 4 | Lambda in the last 24 hours | 0 errors, 0 throttles |
| 5 | SQS and DLQ | 0 waiting, DLQ 0 |
| 6 | Event lake today | 688 files in `events/dt=2026-11-23/` |
| 7 | CloudWatch alarms | both OK |
| 8 | API | `/health` 200, report without token 401 |
| 9 | Website | HTTP 200, 4 of 4 security headers |
| 10 | Alert email | 1 confirmed subscription |

All 11 lines passed.

## Automated Tests

`dotnet test` runs more than 200 tests (212 were recorded on 23 November) without AWS or Docker:

- **Core:** the status rules, the three adapters, and the simulator.
- **Service:** the real API, processor, and store code, running against small in-memory fakes of DynamoDB, SQS, S3, and Athena. A fake throws an error for any call it does not understand, so a code change cannot silently bypass it.

To prove the tests catch real mistakes, I broke the code on purpose in three risky places and checked that the tests failed (mutation testing):

| Area | Deliberate bug | Result |
|---|---|---|
| Store | treat any failed condition as a duplicate | test fails |
| Processor | ignore the FIFO rule after a failure in a batch | 2 tests fail |
| Authorization | remove the role check from `/sla/trend` | 2 tests fail |

GitHub Actions runs the build, the tests, and `sam validate --lint` on every push.

## Chaos Test Against the Deployed API

On 23 November I sent deliberately messy traffic to the real API and compared the final state with an answer key that was generated separately:

| Item | Value |
|---|---|
| Shipments / distinct events | 87 / 480 |
| Webhooks sent | 2,375: each event about 5 times, shuffled within each shipment, plus 24 broken payloads |
| Accepted (202) / rejected (400) | 2,351 / 24, exactly as expected |
| History items in DynamoDB | 480, one per event |
| Differences from the answer key | **0** |
| DLQ before → after | 0 → 0 |
| Time from receive to processed, p50 / p95 / max | 17 / 40 / 192 ms |

The test data used `DEMO` tracking codes, so it does not affect the SLA reports.

## Demo Script

`tools/demo.ps1` walks through five scenes, each showing one customer problem and how the system answers it:

| Scene | Problem | What the system shows |
|---|---|---|
| 1 | every carrier has its own format | 3 different webhooks become the same status |
| 2 | duplicates and wrong order | 8 webhooks still end at `Delivered`; duplicates dropped, old ones `Stale`, backwards ones `Rejected` |
| 3 | customer service opens several sites | one API call returns a whole real timeline |
| 4 | late parcels found only after complaints | late orders → alarm → email within minutes |
| 5 | carriers chosen by feel | on-time rate per carrier and region from Athena |

*(Demo video: in progress. The script is ready; the recording is not done yet.)*
