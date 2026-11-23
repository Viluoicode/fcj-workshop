---
title: "Week 17 Worklog"
date: 2026-10-25
weight: 17
chapter: false
pre: " <b> 1.17. </b> "
---

### Week 17 Objectives:

* Draw the orchestration, writer lock, and IAM boundaries in the architecture diagram.
* Check the diagram against the Terraform code.

### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - Draw the Step Functions workflow (ingestion, Silver, Gold, crawler verification, completion marker) | 19/10/2026 | 19/10/2026 | <https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html> |
| Tue | - Draw the DynamoDB writer lock and the EventBridge daily schedule | 20/10/2026 | 20/10/2026 | <https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html> |
| Wed | - Draw the IAM role boundaries (ingestion, Silver, Gold, crawler, orchestration, monitoring, dashboard reader) | 21/10/2026 | 21/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html> |
| Thu | - Compare the diagram with the Terraform files in infra/ and correct differences | 22/10/2026 | 22/10/2026 | <https://developer.hashicorp.com/terraform/docs> |
| Fri | - *(to be filled)* | 23/10/2026 | 23/10/2026 | |

### Week 17 Achievements:

* Drafted the architecture diagram with AWS icons covering data flow, control flow, and IAM boundaries.
* The PNG export is still pending (planned for Week 21).
