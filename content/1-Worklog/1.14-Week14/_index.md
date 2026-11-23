---
title: "Week 14 Worklog"
date: 2026-10-04
weight: 14
chapter: false
pre: " <b> 1.14. </b> "
---

### Week 14 Objectives:

* Add automated tests and the dashboard reader, then redeploy and record validation evidence.
* Harden security: separate stage roles, restrict the dashboard reader, and validate policies.

### Tasks completed this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| Mon | - Add tests for the dashboard, freshness check, and health monitor; build the dashboard reader (AssumeRole, publication checks); redeploy and record validation evidence | 28/09/2026 | 28/09/2026 | <https://docs.pytest.org/> |
| Tue | - Review private bucket settings, encryption, versioning, HTTPS-only access, and retention | 29/09/2026 | 29/09/2026 | <https://developer.hashicorp.com/terraform/docs> |
| Wed | - Run IAM permission checks and security acceptance; record dated evidence | 30/09/2026 | 30/09/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html> |
| Thu | - Separate Silver, Gold, and crawler roles; validate policies with IAM Access Analyzer; capture dashboard screenshots and run the handoff check | 01/10/2026 | 01/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html> |
| Fri | - *(to be filled)* | 02/10/2026 | 02/10/2026 | |

### Week 14 Achievements:

* Added automated test modules and a restricted dashboard reader.
* Deployed security hardening with separate stage roles and HTTPS-only bucket access.
* Saved dated evidence for validation, permissions, and security acceptance.
