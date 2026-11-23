---
title: "Cost, Cleanup, and Reflection"
date: 2026-11-12
weight: 7
chapter: false
pre: " <b> 5.2.7. </b> "
---

# Cost, Cleanup, Limitations, and Reflection

## Cost Controls

The project uses daily batch processing, serverless query, small Glue worker settings, stage timeouts, retention rules, Athena scan limits, and a monthly budget alert.

An illustrative estimate is approximately USD 13.88 per month under documented assumptions. This is not a measured bill or an official Singapore-region quotation. The final report still requires a saved AWS Pricing Calculator estimate based on observed job durations.

Budget alerts notify; they do not stop spending.

## Storage Retention

| Data | Default retention |
|---|---:|
| Bronze and ingestion manifests | 30 days |
| Quality results | 90 days |
| Athena query results | 7 days |
| Glue temporary data | 7 days |
| Silver, Gold, and state | No automatic expiration |

Versioned noncurrent objects can remain billable after current-object expiration.

## Safe Cleanup

1. Disable the EventBridge pipeline schedule.
2. Confirm that no Step Functions execution, Glue job, or crawler is running.
3. Confirm the DynamoDB lock state and follow the recovery runbook if a run was interrupted.
4. Export any evidence that must be retained.
5. Review `terraform plan -destroy`.
6. Use the explicit bucket cleanup option only after confirming that retained project data may be deleted.
7. Run `terraform destroy`.
8. Verify that subscriptions, alarms, log groups, buckets, and scheduled rules are removed as intended.

Destroying the stack can remove retained data. Cleanup must never be performed without reviewing exact targets and required evidence.

## Known Limitations

- Arbeitnow pagination is capped at two pages.
- Gold publication is not atomic across all three marts.
- Role classification uses title rules and can overlap.
- Missing remote metadata does not prove that a posting is onsite.
- The configured sources do not represent the complete labor market.
- The local dashboard is not a public multi-user application.
- Historical results are not proof of current pipeline health.

## Personal Reflection

The project helped me connect individual AWS services into an operational data system. The most important learning was that successful data processing is not enough: the system must preserve observation identity, distinguish source failure from business change, enforce quality before publication, restrict permissions, expose health signals, and give consumers a way to reject stale results.

AI support accelerated research, implementation, review, and documentation, but it also made validation more important. I needed to inspect proposed code, compare it with AWS service behavior, run tests, review permissions, and verify results instead of accepting generated output without evidence.

## Items Remaining for Final Submission

- AWS-icon architecture diagram and editable source.
- Screenshots for the main deployment and validation steps.
- Saved AWS Pricing Calculator estimate.
- Confirmed personal troubleshooting examples and contribution details.
- Final review of image captions and bilingual terminology.

