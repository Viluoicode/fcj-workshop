---
title: "OnTime Radar - Shipment SLA Service"
date: 2026-11-23
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# OnTime Radar - Shipment SLA Service

## The Problem

An online shop in Vietnam usually ships through two to four carriers at once, such as GHN, GHTK, and J&T. Each carrier has its own status codes, its own tracking page, and its own webhook format. In practice this causes four problems:

- Customer service opens several websites to answer one question: "where is my order?"
- The shop learns that a parcel is late only when the customer complains.
- Nobody can say which carrier delivers on time in which region, so carriers are chosen by feel.
- Webhooks arrive twice or out of order, so a parcel's status can jump backwards.

## What I Built

OnTime Radar is a serverless backend and web dashboard. It receives status events from every carrier and keeps one trusted timeline per shipment:

1. **One status model.** A small adapter per carrier maps its codes onto eight shared statuses.
2. **Correct order, no duplicates.** Events for the same shipment are processed one at a time. Repeated events are ignored, and late or impossible updates are recorded without changing the status.
3. **One place to look.** A single API call returns a shipment's current status and its full history.
4. **Early warning.** When the share of late deliveries stays high, CloudWatch raises an alarm and SNS sends an email with a link to the list of problem orders.
5. **SLA reports.** Every event is also stored in S3, and Athena answers questions such as "on-time rate per carrier and region over the last 30 days".
6. **A dashboard with roles.** Staff sign in with Cognito. Customer-service accounts can look up orders, and managers can also see the SLA reports.

The traffic is real data replayed: a scheduled Lambda replays about 98,000 historical orders from the public Olist e-commerce dataset as carrier webhooks, every 15 minutes. Order dates, delivery dates, promised dates, and regions come from the dataset. Carrier names, tracking codes, and in-transit steps are simulated.

## Who Uses It

| User | Login role | What they need |
|---|---|---|
| Customer service | `cskh` | Answer "where is my order?" in seconds |
| Operations staff | `cskh` | Know immediately which orders are in trouble |
| Operations manager | `manager` | Compare carriers by region with real numbers |
| Carrier systems | webhook secret | Push status updates |

## AWS Services Used

API Gateway (HTTP API), AWS Lambda (.NET 10, arm64), Amazon SQS FIFO, Amazon DynamoDB, Amazon S3, AWS Glue Data Catalog, Amazon Athena, Amazon EventBridge Scheduler, Amazon Cognito, Amazon CloudFront, Amazon CloudWatch, Amazon SNS, AWS IAM, and AWS SAM / CloudFormation. Deployments run from GitHub Actions through OIDC.

## Project Status (23 November 2026)

| Part | Status |
|---|---|
| Webhooks, ordered processing, DynamoDB timeline, S3 event lake | Deployed and running |
| Olist replay every 15 minutes | Running since 16 November; 13,580 of 98,202 orders sent by 23 November |
| Athena SLA reports and API | Deployed |
| Web dashboard on CloudFront with Cognito login | Deployed |
| Alarms and email alerts | Deployed; email subscription confirmed |
| CI/CD through GitHub Actions + OIDC with approval | Working since 20 November |
| Chaos test against the deployed API | Passed on 23 November |
| Cost from Cost Explorer and Pricing Calculator | **In progress** |
| Demo video | **In progress** (script written) |

## AI-Assisted Development

I used AI tools for research, code suggestions, code review, test ideas, and drafting documentation. I decided the requirements, reviewed every change, deployed and checked the system on AWS, and verified the numbers reported in this workshop.

## Sections

1. [Architecture](5.1.1-architecture/)
2. [Prerequisites and deployment](5.1.2-deploy/)
3. [Receiving and processing events](5.1.3-processing/)
4. [SLA reports with Athena](5.1.4-analytics/)
5. [Dashboard, login, and security](5.1.5-security/)
6. [Monitoring and testing](5.1.6-testing/)
7. [Cost and cleanup](5.1.7-cleanup/)
8. [Troubleshooting and future work](5.1.8-troubleshooting/)
