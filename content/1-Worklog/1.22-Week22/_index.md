---
title: "Week 22 Worklog"
date: 2026-11-29
weight: 22
chapter: false
pre: " <b> 1.22. </b> "
---

### Week 22 Objectives:

* Job Market: finish the move of Gold to Apache Iceberg.
* OnTime Radar: test how ingestion behaves when events arrive out of order or malformed.
* *(to be filled)*

### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - Job Market: fix the Iceberg/Parquet comparison (millisecond timestamps), record day 1 as matching, then switch the readers to Iceberg and remove the Glue crawler <br> - OnTime Radar: return 400 instead of 500 for a wrongly shaped payload; build a chaos scenario generator with independent ground truth; test 100 arrival orders in memory; run the chaos runner against the deployed API (2,375 webhooks, 0 mismatches, DLQ empty) <br> - Update the report Workshop to match the current Job Market architecture | 23/11/2026 | 23/11/2026 | <https://github.com/Viluoicode/Job-Market-Pipeline-cloud> <br> <https://github.com/Viluoicode/ontime-radar> <br> <https://iceberg.apache.org/docs/latest/> |
| Tue | | 24/11/2026 | 24/11/2026 | |
| Wed | | 25/11/2026 | 25/11/2026 | |
| Thu | | 26/11/2026 | 26/11/2026 | |
| Fri | | 27/11/2026 | 27/11/2026 | |

### Week 22 Achievements:

* *(to be filled at the end of the week)*
