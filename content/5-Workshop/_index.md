---
title: "Workshop"
date: 2026-11-23
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop: OnTime Radar

This workshop shows how I built **OnTime Radar**, a serverless AWS service that tracks shipments across several carriers and warns staff when deliveries start running late. It has been running on my AWS account since 16 November 2026, fed every 15 minutes with real historical orders from the public Olist dataset.

#### Why this project

A shop that ships through several carriers has the same few problems every day. Each carrier reports status in its own format. Updates arrive twice or out of order. Customer service has to check several websites to answer one question. And late deliveries are discovered only when a customer complains. OnTime Radar gathers every carrier's events into one reliable timeline per shipment, measures on-time performance by carrier and region, and sends an email alert when the late rate climbs.

#### What you will build

- A webhook API that accepts three carrier formats and keeps each shipment's events in order.
- A DynamoDB timeline that ignores duplicates and refuses impossible status changes.
- An S3 event lake with SLA reports in Athena.
- A web dashboard on CloudFront with Cognito login and two staff roles.
- CloudWatch alarms with email alerts, a daily health check, and a chaos test.
- Automatic deployment from GitHub Actions through OIDC, with no stored access keys.

#### Who this is for

Readers who know the basics of Lambda, IAM, and S3 and want to see an event-driven system that handles messy real-world input. You need an AWS account, the AWS SAM CLI, and .NET 10. Section 5.1.2 lists the exact tools.

#### Content

1. [OnTime Radar - Shipment SLA Service](5.1-ontimeradar/)
   1. [Architecture](5.1-ontimeradar/5.1.1-architecture/)
   2. [Prerequisites and deployment](5.1-ontimeradar/5.1.2-deploy/)
   3. [Receiving and processing events](5.1-ontimeradar/5.1.3-processing/)
   4. [SLA reports with Athena](5.1-ontimeradar/5.1.4-analytics/)
   5. [Dashboard, login, and security](5.1-ontimeradar/5.1.5-security/)
   6. [Monitoring and testing](5.1-ontimeradar/5.1.6-testing/)
   7. [Cost and cleanup](5.1-ontimeradar/5.1.7-cleanup/)
   8. [Troubleshooting and future work](5.1-ontimeradar/5.1.8-troubleshooting/)

#### Additional project

2. [Job Market AWS Pipeline](5.2-jobmarket/): the data pipeline I built earlier in the internship (Glue, Step Functions, Iceberg, Athena), kept here as a second, shorter case study.
