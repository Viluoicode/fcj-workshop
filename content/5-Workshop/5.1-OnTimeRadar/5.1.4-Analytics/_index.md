---
title: "SLA Reports With Athena"
date: 2026-11-23
weight: 4
chapter: false
pre: " <b> 5.1.4. </b> "
---

# SLA Reports With Athena

## The Event Lake

Every processed event is stored in the data bucket as one JSON line, in a folder per processing day:

```text
s3://<data bucket>/events/dt=2026-11-23/....jsonl
```

The SAM template creates a Glue database `shipment_sla_lake` and a table `events` over this folder. The table uses **partition projection** on `dt`: Athena works out the folder for each day from the query itself. New data can be queried as soon as it lands, with no crawler and no `MSCK REPAIR`.

A dedicated Athena workgroup, `shipment-sla-wg`, stops any single query after it has scanned 1 GB, so a mistake cannot become an expensive bill.

## Querying by Hand

In the Athena console, choose the workgroup `shipment-sla-wg` and run, for example:

```sql
SELECT carrier, count(*) AS delivered, count_if(islate) AS late
FROM shipment_sla_lake.events
WHERE dt >= '2026-11-01' AND status = 'Delivered' AND outcome = 'Applied'
GROUP BY carrier;
```

Filtering on `dt` keeps the scan to the days you ask for.

> 📷 *Screenshot to add: this query and its result in the Athena console, with the scanned size visible.*

## Report API

The dashboard does not run SQL itself. It calls these endpoints:

| Endpoint | Who can call it | Returns |
|---|---|---|
| `GET /sla/summary?days=&groupBy=carrier\|region` | manager | delivered, late, and late % per carrier or region |
| `GET /sla/trend?days=&carrier=` | manager | per delivery day: orders, late %, average delivery time |
| `GET /shipments?days=&status=&carrier=&region=&late=&overdue=&limit=` | customer service + manager | a filtered list of shipments |
| `GET /shipments/{carrier}/{tracking}` | customer service + manager | one shipment with its full timeline |

User input never goes into the SQL text. `groupBy` must be one of two fixed words, and `days` must be a whole number, which rules out SQL injection. An invalid parameter returns **400** with the reason. Demo shipments (codes starting with `DEMO`) are left out of the SLA numbers.

## Caching

The data changes only every 15 minutes, when the replayer runs. Re-running the same query on every page load would waste time and money, so there are two cache layers:

1. The Lambda keeps each result in memory for 5 minutes.
2. Athena's *query result reuse* returns a previous result for up to 15 minutes without scanning S3.

I measured 38 queries on 20 November 2026:

| Path | Time |
|---|---|
| Full scan of S3 | about 2.4 s |
| Result reused by Athena | about 0.55 s |
| Lambda memory | about 0 ms |

79% of the queries did not have to scan S3.

## Measured Numbers

On 19 November 2026, with about 5,700 of the 98,202 Olist orders replayed, the dashboard showed 5,425 delivered orders, a 96.4% on-time rate, and an average delivery time of 13.3 days.

These numbers reflect how the replay works: orders are sent by order date, so the early orders from 2016–2017 dominate. They are not the late rate of the Olist dataset as a whole.
