---
title: "Week 18 Worklog"
date: 2026-11-01
weight: 18
chapter: false
pre: " <b> 1.18. </b> "
---

### Week 18 Objectives:

* Review monitoring, the dashboard reader, and security boundaries of the deployed project.

### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - Review the Lambda health monitor and the identity comparison across pipeline completion, Silver state, and ingestion manifest | 26/10/2026 | 26/10/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html> |
| Tue | - Review the CloudWatch metrics (Healthy, SourceSuccessRatio), alarms, and SNS notification | 27/10/2026 | 27/10/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html> |
| Wed | - Review the dashboard reader role and its limits (published Gold data, metadata, Athena, Catalog, results, pipeline lock) | 28/10/2026 | 28/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html> |
| Thu | - Review the dashboard checks before and after Athena reads (stale, in-progress, or changed publication) | 29/10/2026 | 29/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html> |
| Fri | - *(to be filled)* | 30/10/2026 | 30/10/2026 | |

### Week 18 Achievements:

* Verified how monitoring and the dashboard reader detect stale or inconsistent publication.
