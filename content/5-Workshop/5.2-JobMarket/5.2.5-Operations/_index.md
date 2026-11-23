---
title: "Orchestration, Security, and Monitoring"
date: 2026-11-23
weight: 5
chapter: false
pre: " <b> 5.2.5. </b> "
---

# Orchestration, Security, and Monitoring

## The Workflow

EventBridge starts the state machine every day at 18:00 UTC, which is 01:00 the next morning in Vietnam. The Step Functions states, in order:

1. **InitializeRun** fixes one run ID and one UTC date. Every stage receives both.
2. **AcquireLock** writes the lock item to DynamoDB only if no lock exists. If one already exists, the run ends at **PipelineBusy** without touching any data.
3. **Ingest**, **BronzeToSilver**, and **SilverToGold** each start a Glue job and wait for it to finish.
4. **ReadGoldVersions** reads the record the Gold job wrote, and **GoldVersionsValid** checks that it belongs to this run and names all three tables.
5. **BuildCompletion** and **PublishCompletion** write `state/pipeline/latest.json` with the three Iceberg snapshot IDs.
6. **ReleaseLock** deletes the lock, but only if this execution still owns it.

If any step fails, the workflow goes to **ReleaseFailedLock** and ends as failed. The whole run has a 90-minute limit, and each Glue job has its own timeout and allows only one run at a time.

> 📷 *Screenshot to add: a successful execution graph in Step Functions.*

### When a Run Is Killed

A timeout or a manual abort skips ReleaseLock, so the lock would stay and block the next night's run. The lock-janitor Lambda handles this case. It runs when an execution ends as FAILED, TIMED_OUT, or ABORTED, and again every 15 minutes. It deletes the lock only when:

- the execution that owns the lock has ended, and
- none of the three Glue jobs is still running.

The delete is conditional on the owner it just inspected. The janitor never starts or stops anything. Its decision (`released`, `waiting_for_glue`, `owner_running`, or `no_lock`) is written to its log.

## Security

- Every stage has its own IAM role: ingestion, Silver, Gold, Step Functions, EventBridge, the health monitor, the janitor, and the dashboard reader. Each role can touch only the S3 prefixes it needs.
- Runtime roles can read their scripts but cannot change them. Only Step Functions can write the completion marker.
- The dashboard role can read published data, run Athena queries, and write its own query results. It cannot change the lake or start a job.
- All three buckets are private, encrypted, and reject plain HTTP. The lake and script buckets keep versions for 30 days.
- No access key is stored anywhere. AWS services use their roles, and I use short-lived credentials.
- `scripts/check_permissions.py` runs IAM policy simulations against the deployed roles (69 of 69 passed on the last check). Checkov scans the Terraform code on every pull request, and every accepted finding has a written reason next to the resource.

## Monitoring

The health-monitor Lambda runs every 15 minutes and reads only metadata: the completion marker, the Silver pointer, and the manifest. It sends two metrics to CloudWatch:

- `Healthy`: 1 when the run IDs match, ingestion succeeded, and the data is under 26 hours old.
- `SourceSuccessRatio`: the share of boards that answered successfully.

Six alarms send email through SNS:

| Alarm | Fires when |
|---|---|
| `pipeline-failed` | an execution fails |
| `pipeline-timed-out` | an execution hits the 90-minute limit |
| `pipeline-aborted` | an execution is stopped |
| `pipeline-unhealthy` | `Healthy` is below 1, or missing, for two 15-minute periods |
| `source-coverage-low` | fewer than 100% of boards succeed for two periods |
| `lock-janitor-errors` | the janitor Lambda itself throws an error |

A missing `Healthy` metric counts as unhealthy. So if the monitor itself stops running, that is noticed too.

Logs are kept for 30 days for Glue and 14 days for Lambda. They live in CloudWatch Logs, under one log group per job and per function.

> 📷 *Screenshots to add: the CloudWatch alarm list and the SNS alert email.*
