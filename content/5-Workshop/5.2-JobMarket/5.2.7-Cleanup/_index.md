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

### Measured Cost

`scripts/run_metrics.py` measured the 7 scheduled runs of 14–20 November 2026 and compared them with Cost Explorer. These are list prices before credits:

| Item | Per run (median) |
|---|---:|
| Glue jobs (3 jobs, 517 DPU-seconds) | USD 0.064 (Cost Explorer: USD 0.0635) |
| Glue crawler | USD 0.049 |
| **Glue total** | **about USD 0.11** |
| Lambda, Step Functions, EventBridge, 6 alarms | within the free tier |
| S3, DynamoDB | a few cents per day |

With one run per day, that is about **USD 3.4 per month** for Glue. These runs were measured before parallel fetching and Iceberg were added. That change removed the crawler, which was about 44% of the Glue cost per run, so the current cost should be closer to USD 2 per month. This lower figure is an estimate until a full week is measured again.

*(To fill in: a link to the saved AWS Pricing Calculator estimate based on these numbers.)*

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

- Screenshots for the main deployment and validation steps (the architecture diagram and dashboard screenshots are done).
- Saved AWS Pricing Calculator estimate.
- Confirmed personal troubleshooting examples and contribution details.
- Final review of image captions and bilingual terminology.

