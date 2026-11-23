---
title: "Architecture"
date: 2026-11-23
weight: 1
chapter: false
pre: " <b> 5.1.1. </b> "
---

# Architecture

![OnTime Radar architecture on AWS: carriers and the replayer send webhooks through API Gateway into SQS FIFO; the processor writes DynamoDB and S3; staff use a CloudFront website with Cognito login; Athena reports; CloudWatch alarms notify through SNS](../../../images/5-Workshop/5.1-OnTimeRadar/architecture.png)

The diagram is drawn in draw.io with the official AWS icons, and the editable source is `docs/diagrams/architecture.drawio` in the repository. The numbers on the arrows match the steps below. Everything runs in one region, `ap-southeast-1` (Singapore).

## Flow 1 - Receiving Carrier Events (steps 1-6)

1. A carrier (or the replayer) sends a webhook to **API Gateway**. Webhooks authenticate with a shared secret header, not with Cognito.
2. The **API Lambda** picks the adapter for that carrier, maps the payload to one of eight common statuses, and computes a deterministic `eventId` from the content.
3. The event is put on an **SQS FIFO** queue with the shipment as the message group, and the carrier gets `202 Accepted` right away.
4. The **Processor Lambda** takes events one shipment at a time. A small state machine decides whether each one is `Applied`, `Stale`, `Rejected`, or `Duplicate`, and **DynamoDB** saves the new status and the history item in one transaction.
5. The processor also appends the event to the **S3** data bucket under `events/dt=YYYY-MM-DD/` as JSON lines.
6. It publishes delivered and late-delivery counts to **CloudWatch**, per carrier.

A message that fails five times moves to a **dead-letter queue**, which has its own alarm. Every 15 minutes, **EventBridge Scheduler** starts the **Replayer Lambda**, which reads the next Olist orders from S3 and sends them through exactly the same path as a real carrier (S1-S2).

## Flow 2 - Staff Using the Dashboard (steps 7-11)

7. The browser loads the website from **CloudFront**. The web bucket is private, and only CloudFront can read it, through Origin Access Control.
8. The user signs in with **Cognito** (authorization code + PKCE) and receives a token that says which group they belong to.
9. The browser calls API Gateway with that token. The gateway rejects a missing or expired token (401), and the Lambda rejects the wrong role (403).
10. To look up a shipment, the Lambda runs one DynamoDB query that returns the status and the whole history.
11. For SLA reports, the Lambda runs SQL in **Athena** over the event lake. The table schema lives in the **Glue Data Catalog**.

## Flow 3 - Alerts (steps 12-13)

12. A CloudWatch alarm fires when the late-delivery rate is 20% or more in 2 of 3 consecutive 15-minute windows, or when the dead-letter queue is not empty.
13. **SNS** emails the on-call person, once when the problem starts and once when it is over.

## Deployment

Pushing to `main` runs GitHub Actions. Tests run first, then the workflow waits for my approval, then it gets a temporary IAM role through **OIDC** and deploys the stack with **AWS SAM / CloudFormation**. No AWS access key is stored on GitHub.

## Why These Services

| Need | Choice | Reason |
|---|---|---|
| Keep events for one shipment in order, while different shipments still run in parallel | SQS FIFO, message group = shipment | FIFO guarantees order inside a group, and groups are processed in parallel |
| Handle the same webhook twice safely | Deterministic `eventId` + DynamoDB condition | The second copy becomes `Duplicate` and writes nothing |
| Status and history must never disagree | DynamoDB transaction with a version number | Both items are written together, and two writers cannot overwrite each other |
| Pay only for what is used | Lambda (arm64), DynamoDB on-demand, no VPC | No servers, no NAT gateway, and arm64 costs about 20% less than x86 |
| SLA analysis without a database server | S3 + Athena + partition projection | Queries scan only the days they need, with no crawler |
| Login with two roles | Cognito + the API Gateway JWT authorizer | Bad tokens are stopped at the gateway before Lambda runs |
| Host the website | S3 + CloudFront with OAC | HTTPS at almost no cost, and the bucket stays private |
| Deploy without long-lived keys | GitHub OIDC + approval | A leaked repository secret cannot be used to deploy |

## Well-Architected View

- **Operational excellence:** the whole stack is one SAM template. A one-command daily check reports 10 health items. Deployment is automated and needs approval.
- **Security:** Cognito login with roles, webhook secret, private buckets, CloudFront security headers, a permissions boundary on every role, API throttling, input validation, and no stored keys.
- **Reliability:** FIFO ordering, idempotent writes, transactions, partial batch failure with a DLQ, and a replayer that never skips a failed order.
- **Performance efficiency:** the median time from webhook to stored status was 17 ms (p95 40 ms) in the chaos test. A two-layer cache means 79% of report queries do not scan S3.
- **Cost optimization:** serverless on arm64, on-demand DynamoDB, a 1 GB cap per Athena query, partition projection, and 14-day log retention.
