---
title: "Troubleshooting and Future Work"
date: 2026-11-23
weight: 8
chapter: false
pre: " <b> 5.2.8. </b> "
---

# Troubleshooting and Future Work

This page records problems I actually ran into while building and running the pipeline, what caused them, and how I fixed them. Each entry points to the commit where the fix landed, so the story can be checked against the repository.

## Problems I Hit

### 1. One job board stopped returning data

- **Symptom:** from 6 November the `source-coverage-low` alarm stayed in ALARM, and Mistral AI postings stopped updating.
- **Cause:** the company moved its careers page from Lever to Ashby. The old Lever board answered with HTTP 404, so the ingestion job counted it as a failed board every day.
- **Fix:** I replaced the Lever entry with the Ashby board `mistral.ai` in `ingestion/sources.json`, which kept the catalog at 36 boards. I also added a test that rejects a company listed under two platforms, because that would count its postings twice.
- **Lesson:** a coverage alarm is only useful if someone reads it. This one fired for two weeks before I acted on it.
- **Commit:** `be0e63f` (21/11/2026).

> 📷 *Add: the CloudWatch alarm history showing ALARM, then OK after the fix.*

### 2. The next day's run refused to start (`PipelineBusy`)

- **Symptom:** the 01:00 run stopped immediately with `PipelineBusy`.
- **Cause:** a previous execution had timed out. A timeout skips the `ReleaseLock` step, so the DynamoDB lock stayed in the table, and every later run saw it and backed off.
- **Fix:** I added a small Lambda ("lock janitor"). It runs when an execution ends as FAILED, TIMED_OUT, or ABORTED, and again every 15 minutes. It deletes the lock only when the owning execution has finished and no Glue job or crawler is still running, and the delete is conditional on the owner it inspected.
- **Verification:** `terraform apply` added 11 resources, and a real invoke returned `no_lock`.
- **Commit:** `9dacb34` (21/11/2026).

### 3. Re-running a day could leave old rows in Gold

- **Symptom:** a test showed that re-running a day whose mart came out empty still left the previous rows in the table.
- **Cause:** Spark's dynamic partition overwrite only replaces partitions that receive new rows. An empty result touches no partition, so nothing is replaced.
- **Fix:** the Gold job overwrites the requested `snapshot_date` explicitly, in a single Iceberg commit.
- **Commit:** `d733030` (22/11/2026).

### 4. Iceberg and Parquet tables "did not match"

- **Symptom:** after the first Iceberg shadow write, both copies had the same row counts (8,682 / 15 / 15), but an `EXCEPT` comparison reported every row as different.
- **Cause:** Athena reads Parquet timestamps at millisecond precision, while Iceberg keeps Spark's microseconds. The values were the same moment written with different precision.
- **Fix:** the comparison script rounds timestamps to milliseconds before comparing. Day 1 then showed 0 differing rows in both directions.
- **Commit:** `7ab87d9` (23/11/2026).

### 5. Parallel fetching could run out of memory

- **Symptom (found during design, before it failed in production):** a single run produces about 150 MB of Bronze JSON, while Glue Python Shell has about 1 GB of RAM.
- **Cause:** fetching all 36 boards in parallel and only writing at the end would hold every response in memory at once.
- **Fix:** 5 worker threads fetch the boards. Results are written to S3 one at a time, in board order, and at most 10 boards are held in memory. A failed S3 write still aborts the whole run.
- **Commit:** `066c331` (22/11/2026).

### 6. A lint upgrade would have broken the Glue job

- **Symptom:** after adding `ruff`, its upgrade rules wanted to rewrite code to use `datetime.UTC`.
- **Cause:** `datetime.UTC` only exists from Python 3.11, but Glue Python Shell runs Python 3.9.
- **Fix:** I set `target-version = "py39"` in `pyproject.toml` so the linter only suggests syntax the runtime supports.
- **Commit:** `37d537f` (21/11/2026).

### 7. Smaller fixes

| Problem | Fix | Commit |
|---|---|---|
| A unit test failed because Spark guessed the Bronze schema differently from production | Declared the schema explicitly in the test fixture | `d1136a4` (28/07) |
| The dashboard broke after a Streamlit upgrade removed `use_container_width` | Switched to `width="stretch"` | `a9dd23e` (29/07) |
| Vietnamese text in the docs turned into broken characters | Forced UTF-8 and added a test that checks file encoding | `e829916` (20/11) |
| CI failed because `pandas` was missing | Added it to `requirements-dev.txt` | `44cf03d` (20/11) |

### 8. Other problems (to be filled in)

*(Add any problem that never made it into a commit, for example console errors, IAM `AccessDenied` messages, or a failed `terraform apply`, using the same layout: symptom, cause, fix, and how you checked.)*

## Future Work

*(Draft list. Keep the items you actually plan, and remove the rest.)*

- **Switch readers to Iceberg.** Athena and the dashboard still read the Parquet copy. Once the shadow comparison stays clean for several days, they can read Iceberg, and the three marts can then be published together.
- **Lift the Arbeitnow page limit.** The feed is capped at two pages, so some postings are skipped.
- **A shared dashboard.** The Streamlit app runs only on my machine. A hosted, read-only version would let other people use it without AWS credentials.
- **Deploy from CI.** CI already runs tests, `terraform validate`, and Checkov. The next step is a reviewed `plan` on each pull request, using short-lived OIDC credentials.
- **Better role classification.** Titles are mapped to roles with rules. A labelled sample would show how often those rules are wrong.
