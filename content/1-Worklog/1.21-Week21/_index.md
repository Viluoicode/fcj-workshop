---
title: "Week 21 Worklog"
date: 2026-11-22
weight: 21
chapter: false
pre: " <b> 1.21. </b> "
---

### Week 21 Objectives:

* Start Project 2, OnTime Radar (Shipment SLA Service): a serverless backend that tracks shipments from several carriers and flags late deliveries.
* Harden Project 1 (Job Market AWS Pipeline): remote Terraform state, automatic lock recovery, a security scan in CI, and a fix for a broken source.

### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - OnTime Radar: build the first pipeline: carrier webhooks through API Gateway, an SQS FIFO queue so each shipment is processed in order, DynamoDB for state and history, an S3 event lake, and alarms for late deliveries and the DLQ <br> - Add SLA reporting with Athena over the event lake, a CI workflow, and Cognito login with two roles (customer service and manager) <br> - Restructure the internship report worklog into a theory phase and a project phase | 16/11/2026 | 16/11/2026 | <https://github.com/Viluoicode/ontime-radar> <br> <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html> |
| Tue | - OnTime Radar: check the CloudFormation stack in the console, grant and adjust IAM permissions for the deploy user, and review Billing and Cost Management <br> - Research a UI design approach for the dashboard <br> - Register for AWS Student Community Day Vietnam 2026 (21/11) | 17/11/2026 | 17/11/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html> <br> <https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-what-is.html> |
| Wed | - Team project `kfc-force-agent`: open issue #12 and a pull request that adds an AI agent understanding Vietnamese, with tool use, Amazon Bedrock AgentCore, and DynamoDB | 18/11/2026 | 18/11/2026 | <https://github.com/HuynhHao080/kfc-force-agent> <br> <https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html> |
| Thu | - OnTime Radar: build the dashboard website and host it on S3 + CloudFront <br> - Add a daily health check, and change the late-delivery alarm to use a rate sustained over 2 of 3 periods <br> - Redraw the architecture diagram to match what is deployed | 19/11/2026 | 19/11/2026 | <https://github.com/Viluoicode/ontime-radar> <br> <https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html> |
| Fri | - OnTime Radar: deploy from GitHub Actions through OIDC with a manual approval step; add API throttling, a permissions boundary on every app role, tracking-code validation, an Athena result cache, and 66 service tests <br> - Job Market: switch to 15 single-label role families, a three-state work mode, a deliberate rule for choosing the deduplication representative, and restore UTF-8 Vietnamese docs | 20/11/2026 | 20/11/2026 | <https://github.com/Viluoicode/ontime-radar> <br> <https://github.com/Viluoicode/Job-Market-Pipeline-cloud> <br> <https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services> |
| Sat | - Job Market: move Terraform state to S3, rewrite runtime IAM in HCL, add the lock-janitor Lambda, add ruff and Checkov to CI (41 findings handled), move Mistral AI from Lever to Ashby, and add a run-metrics script | 21/11/2026 | 21/11/2026 | <https://github.com/Viluoicode/Job-Market-Pipeline-cloud> <br> <https://developer.hashicorp.com/terraform/language/backend/s3> |
| Sun | - Job Market: fetch board APIs in parallel (5 workers), shadow-write Gold to Apache Iceberg, redact the account ID from evidence files, rewrite the README with the AWS architecture diagram, and set 30-day retention for Glue logs and old S3 versions | 22/11/2026 | 22/11/2026 | <https://github.com/Viluoicode/Job-Market-Pipeline-cloud> <br> <https://docs.aws.amazon.com/athena/latest/ug/querying-iceberg.html> |

### Week 21 Achievements:

* OnTime Radar is deployed: webhooks, ordered processing, SLA reporting, a hosted dashboard with login, alarms, and OIDC deployment with approval.
* Job Market now recovers an orphaned lock without manual work, keeps its Terraform state in S3, and passes a Checkov scan in CI.
* Found and fixed a source that had returned HTTP 404 since 6 November.
* Contributed an AgentCore + DynamoDB agent feature to a team project.
