---
title: "Cost and Cleanup"
date: 2026-11-23
weight: 7
chapter: false
pre: " <b> 5.1.7. </b> "
---

# Cost and Cleanup

## Cost Estimate

{{% notice warning %}}
This section is **in progress**. The figure below is an early estimate. The final report needs the measured cost from Cost Explorer and a saved AWS Pricing Calculator estimate based on that traffic.
{{% /notice %}}

With the replayer running every 15 minutes (about 1,900 orders and 9,000 events a day), I estimate **about USD 3–5 per month**. DynamoDB writes are the largest item, because every event writes a status item and a history item in one transaction.

| Service | What drives the cost | Note |
|---|---|---|
| DynamoDB (on-demand) | writes per event, plus queries from the dashboard | largest item |
| Lambda (arm64) | API, processor, and replayer invocations | mostly within the free tier |
| API Gateway (HTTP API) | webhook and dashboard requests | |
| SQS FIFO | messages and retries | |
| S3 | event lake, dataset, website | small amounts of data |
| Athena | bytes scanned | capped at 1 GB per query, and most queries are served from cache |
| CloudFront, Cognito, CloudWatch, SNS | traffic, users, logs, alarms, emails | low use |

*(To fill in: the Cost Explorer figures per service for a full week, and a link to the saved Pricing Calculator estimate.)*

> 📷 *Screenshot to add: Cost Explorer filtered to this project's services, grouped by service.*

## How the Design Keeps Cost Down

- **No servers and no VPC:** Lambda and managed services only, so there is no NAT gateway, which would cost more than the whole application.
- **arm64 (Graviton) Lambdas:** about 20% cheaper than x86 for the same running time.
- **DynamoDB on-demand:** pay per request, with nothing to pay while the system is idle.
- **Athena limits:** a 1 GB cap per query, partition projection so only the requested days are scanned, and a two-layer cache that served 79% of report queries without a scan.
- **Throttling** at API Gateway, so a flood of requests is rejected before it runs Lambda.
- **14-day log retention** instead of keeping logs forever.

## Cleanup

Delete the resources only after the project has been graded. Follow this order:

1. Stop the replay: set `ReplayState` to `DISABLED` and deploy, or disable the schedule in EventBridge Scheduler.
2. Save any evidence you need: the daily-check files, screenshots, and Cost Explorer exports.
3. Empty the two buckets. CloudFormation cannot delete a bucket that still contains objects.

   ```powershell
   aws s3 rm "s3://<DataBucketName>" --recursive
   aws s3 rm "s3://<web bucket>" --recursive
   ```

4. Delete the application stack:

   ```powershell
   sam delete --stack-name shipment-sla
   ```

   This removes the API, Lambdas, queues, table, Cognito user pool, CloudFront distribution, alarms, Glue table, and Athena workgroup.
5. Delete the deployment stack `github-oidc-deploy` (the OIDC provider, the deploy role, and the boundary):

   ```powershell
   aws cloudformation delete-stack --stack-name github-oidc-deploy --region ap-southeast-1
   ```

6. In GitHub, remove the `AWS_DEPLOY_ROLE_ARN` secret and the `production` environment.
7. Check that nothing is left: in the console, look at CloudFormation, S3, CloudFront, and CloudWatch log groups in `ap-southeast-1`. Then check the bill a day later.
