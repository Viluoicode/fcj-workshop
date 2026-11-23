---
title: "Project Proposal"
date: 2026-11-16
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Project Proposal - OnTime Radar (Shipment SLA Service)

## 1. Summary

OnTime Radar is a serverless service on AWS for online shops that ship through several carriers. It receives status webhooks from every carrier, turns them into one trustworthy timeline per shipment, warns staff by email when deliveries start running late, and reports the on-time rate per carrier and region. A web dashboard with login gives customer service and managers what each of them needs.

The project runs on real traffic. A scheduled job replays about 98,000 historical orders from the public Olist e-commerce dataset as carrier webhooks, every 15 minutes, so the system is tested on realistic volume rather than a few hand-made requests.

This is my second internship project. The first one, Job Market AWS Pipeline (a batch data pipeline), is kept in the Workshop as an additional case study.

## 2. Problem

A small or medium online shop in Vietnam often uses two to four carriers at the same time. Every carrier uses its own status codes, its own tracking website, and its own webhook format. The shop pays for this every day:

| Pain | Consequence |
|---|---|
| Staff check several carrier websites to track one order | slow answers to customers |
| A late parcel is noticed only when the customer complains | bad reviews and returns |
| There is no on-time data per carrier or region | carriers are chosen by feel, and there is nothing to negotiate with |
| Webhooks arrive twice or in the wrong order | the order status jumps backwards and staff stop trusting it |

## 3. Objectives and Success Criteria

### Objectives

1. Accept webhooks from three carriers (GHN, GHTK, J&T; simulated formats) and map them onto one set of eight statuses.
2. Process each shipment's events in order, ignore duplicates, and refuse impossible status changes.
3. Give a full shipment timeline in one API call.
4. Detect a rising late-delivery rate and email the operations team.
5. Report the on-time rate by carrier, region, and week.
6. Provide a web dashboard with two roles: customer service and manager.
7. Deploy everything as code, automatically, without long-lived access keys.

### Success criteria

| Criterion | How it is checked |
|---|---|
| No wrong final status when events are duplicated or shuffled | Chaos test against the deployed API: 0 differences from an independent answer key |
| No event lost | Replayer `Failed = 0`; dead-letter queue empty |
| A late-delivery problem reaches a person without anyone watching a screen | Alarm → SNS email, tested with the demo script |
| A customer-service account cannot see management reports | 401/403 permission matrix checked on the live API |
| Reports cost little to run | Athena query cap of 1 GB; cache hit rate measured |
| The system can be rebuilt and removed from code | One SAM template; documented cleanup |

## 4. Proposed Architecture

![OnTime Radar architecture: webhooks through API Gateway, SQS FIFO, Lambda processor, DynamoDB and S3; dashboard on CloudFront with Cognito; Athena reports; CloudWatch and SNS alerts; GitHub Actions deploys through OIDC](../images/5-Workshop/5.1-OnTimeRadar/architecture.png)

The system has three flows:

- **Events in:** carrier webhook → API Gateway → Lambda adapter → SQS FIFO (one group per shipment) → Lambda processor → DynamoDB (status + history in one transaction) and an S3 event lake.
- **Staff dashboard:** browser → CloudFront → private S3 website; sign-in with Cognito; API Gateway checks the token; lookups read DynamoDB, and reports run in Athena over the event lake.
- **Alerts:** processor metrics → CloudWatch alarm on the late rate and on the dead-letter queue → SNS email.

Deployment: GitHub Actions runs the tests, waits for my approval, gets a temporary role through OIDC, and runs `sam deploy`.

## 5. AWS Services and Why

| Service | Role in the system | Why this one |
|---|---|---|
| Amazon API Gateway (HTTP API) | entry point for webhooks and the dashboard | JWT check, throttling, and CORS without a server |
| AWS Lambda (.NET 10, arm64) | adapters, processor, replayer | pay per use; arm64 is about 20% cheaper |
| Amazon SQS FIFO + DLQ | keep each shipment's events in order | ordering per message group, with parallel groups |
| Amazon DynamoDB (on-demand) | current status and history | transactions and conditional writes for idempotency |
| Amazon S3 | event lake, dataset, website files | cheap and durable |
| AWS Glue Data Catalog + Amazon Athena | SLA reports with SQL | no database server; partition projection, so no crawler |
| Amazon EventBridge Scheduler | runs the replayer every 15 minutes | simple managed schedule |
| Amazon Cognito | staff login and roles | managed sign-in, groups, and optional MFA |
| Amazon CloudFront | serves the dashboard over HTTPS | keeps the bucket private through OAC, adds security headers |
| Amazon CloudWatch + Amazon SNS | metrics, logs, alarms, email | built-in and cheap at this scale |
| AWS IAM + AWS SAM / CloudFormation | permissions and infrastructure as code | least privilege, a permissions boundary, and repeatable deploys |

## 6. Security and Operations

- Staff log in through Cognito (authorization code + PKCE). The gateway rejects bad tokens (401), and the Lambda rejects the wrong role (403).
- Carriers authenticate with a webhook secret. Input is validated, and bad input gets 400.
- The website bucket is private, CloudFront adds HSTS, CSP, and other security headers, and only GET requests from the dashboard's own domain are allowed by CORS.
- Every role carries a permissions boundary. Deployments use OIDC with an approval step, and no access key is stored anywhere.
- API throttling of 10 requests per second per route protects the account's Lambda concurrency.
- A daily health-check script and two alarms cover the replayer, queues, Lambdas, the event lake, the API, and the website.

## 7. Timeline

The project started on 16 November 2026 (Week 21). Workshop submission is due by 16 December.

| Week | Dates | Work | Status |
|---|---|---|---|
| 21 | 16/11 – 22/11 | Webhooks, FIFO processing, DynamoDB, S3 lake, replayer, Athena reports, Cognito, dashboard on CloudFront, alarms, CI/CD through OIDC, security review fixes | Done |
| 22 | 23/11 – 27/11 | Chaos testing against the deployed API, known limitations, evidence collection | In progress |
| 23 | 30/11 – 04/12 | Cost report (Cost Explorer + Pricing Calculator), demo video | Planned |
| 24 | 07/12 – 11/12 | Final workshop review, report, and submission | Planned |

## 8. Budget

With the replayer sending about 1,900 orders (about 9,000 events) a day, the early estimate is **USD 3–5 per month**, mostly DynamoDB writes. The design keeps cost low with serverless services on arm64, on-demand DynamoDB, no VPC or NAT gateway, a 1 GB cap per Athena query, a two-layer report cache, and 14-day log retention.

This is an estimate, not a bill. The final report will include measured Cost Explorer figures and a saved AWS Pricing Calculator estimate based on them.

## 9. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Duplicate or out-of-order webhooks | FIFO per shipment, deterministic event IDs, version check, and status rules |
| A message that keeps failing | Partial batch failure; after 5 attempts it goes to the DLQ, which raises an alarm |
| A traffic burst uses all Lambda capacity (10 in a new account) | API throttling and a processor concurrency limit; a quota increase is planned |
| Alert fatigue | Alarm on a sustained late *rate*, not a raw count |
| A leaked deploy credential | No stored keys; OIDC bound to repository IDs and the approved environment; permissions boundary |
| An expensive runaway query | Athena workgroup cap of 1 GB per query |
| Misleading numbers | Demo and test shipments are excluded; limitations of the simulated fields and the Brazilian data are documented |
| AI-generated code that is wrong | Code review, more than 200 automated tests including mutation checks, and a chaos test on the live system |

## 10. Deliverables

- Source code, the SAM template, and the CI/CD workflows in the `ontime-radar` repository.
- Architecture diagram (draw.io, AWS icons).
- A running system with the web dashboard, alarms, and daily replay.
- A bilingual Workshop with deployment, testing, cost, and cleanup steps.
- Evidence: daily-check outputs, a chaos test report, and screenshots.
- A demo video *(in progress)*.
- A cost report *(in progress)*.
