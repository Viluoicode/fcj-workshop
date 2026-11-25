---
title: "Dashboard, Testing, and Results"
date: 2026-11-23
weight: 6
chapter: false
pre: " <b> 5.2.6. </b> "
---

# Dashboard, Testing, and Results

## What the Dashboard Checks Before Showing Anything

The Streamlit app always switches to the read-only dashboard role first. Then it:

1. Makes sure no pipeline run currently holds the lock.
2. Reads the completion marker, the Silver pointer, and the matching manifest.
3. Checks that all three agree on the run ID and UTC date, that ingestion succeeded, and that the data is less than 26 hours old.
4. Queries `fact_job_posting` with `FOR VERSION AS OF` the snapshot ID named in the marker.
5. Repeats steps 1–3 after the query. If anything changed in between, it throws the result away and shows a message instead.

It never falls back to "the newest date in the table". Stale or half-published data is refused, not displayed.

## What the Dashboard Shows

- Filters for platform, role family, location, and work mode, plus a search on title or company.
- Number of postings, companies, and locations, and the remote share.
- Postings by role family, and the top employers.
- A table of postings with links to the original job page.
- Publication time, data age, board coverage, the Athena query ID, and the bytes scanned.

![Dashboard overview for the snapshot of 21 November 2026: publication status, coverage note, filters, four KPIs, postings by role family, and top companies](../../../images/5-Workshop/5.2-JobMarket/dashboard-overview.png)

*Overview of the published snapshot of 21 November 2026.*

![Postings table filtered by the search term data engineer, with company, location, source, work mode, last observed time, and a link to each original posting](../../../images/5-Workshop/5.2-JobMarket/dashboard-postings.png)

*Postings table with links to the original job pages.*

## Opening It Locally

```powershell
$env:AWS_REGION = 'ap-southeast-1'
$env:LAKE_BUCKET = terraform -chdir=infra output -raw lake_bucket
$env:PIPELINE_LOCK_TABLE = terraform -chdir=infra output -raw pipeline_lock_table
$env:ATHENA_DATABASE = terraform -chdir=infra output -raw gold_database
$env:ATHENA_WORKGROUP = terraform -chdir=infra output -raw athena_workgroup
$env:AWS_DASHBOARD_ROLE_ARN = terraform -chdir=infra output -raw dashboard_reader_role_arn
.\.venv\Scripts\python.exe -m streamlit run dashboard/app.py --server.address 127.0.0.1
```

Then open `http://localhost:8501` and press **Refresh data**. Viewing data does not deploy anything or start a run.

## Automated Tests

Every pull request runs `ruff`, 251 pytest tests, `terraform validate`, and a Checkov scan in GitHub Actions. The tests mock the job APIs and AWS, and cover:

- ingestion: retries, failed and empty boards, page limits, parallel fetching order, manifest commit order;
- Silver: identity, deduplication, every lifecycle case in 5.2.4, and Data Quality preparation;
- Gold: the three marts, role families, empty-day overwrite, and Iceberg writes;
- the dashboard and both Lambdas: locks, stale or changed publication, and janitor decisions;
- the repository itself: evidence files are valid JSON and contain no AWS account ID.

## Testing Failure Cases on AWS

| Scenario | How to trigger it | Expected result |
|---|---|---|
| A second run while one is active | Start a manual execution during the nightly run | It ends at `PipelineBusy`, and no data changes |
| A board stops answering | Happened for real with Lever `mistral` in November | `source-coverage-low` goes to ALARM; Silver keeps the board's postings as unverified |
| Data becomes stale | Pause the schedule for more than 26 hours | `pipeline-unhealthy` fires, and the dashboard refuses to show data |
| A run is aborted | Stop an execution in the console | `pipeline-aborted` fires; the janitor releases the lock once Glue is idle |

*(Fill in: the date you ran each scenario, plus a screenshot or log line as proof.)*

## Measured Results

These are dated observations, not a live status:

| Measurement | Value |
|---|---|
| Scheduled runs that succeeded, 15 Sep – 21 Nov 2026 | 68 of 68 |
| Open postings on 21 Nov 2026 | 8,672 from 467 employers |
| Boards collected on 21 Nov 2026 | 36 of 36 |
| Time from trigger to published Gold (median, 14–20 Nov) | 9 minutes |
| Glue cost per run (before credits) | about USD 0.11 |
| Bytes scanned by one dashboard query | about 1 MiB |

The run-time and cost figures were measured with `scripts/run_metrics.py` before parallel fetching and Iceberg were added. That change also removed the crawler, so the cost per run is now lower. Gold can hold more rows than one run's Bronze, because postings from a board that failed keep their last observation while they are still fresh.
