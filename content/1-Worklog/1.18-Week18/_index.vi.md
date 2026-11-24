---
title: "Worklog Tuần 18"
date: 2026-11-01
weight: 18
chapter: false
pre: " <b> 1.18. </b> "
---

### Mục tiêu tuần 18:

* Rà soát monitoring, dashboard reader và ranh giới bảo mật của dự án đã triển khai.

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Rà soát Lambda health monitor và việc so sánh identity giữa pipeline completion, trạng thái Silver và ingestion manifest | 26/10/2026 | 26/10/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html> |
| 3 | - Rà soát metric CloudWatch (Healthy, SourceSuccessRatio), alarm và thông báo SNS | 27/10/2026 | 27/10/2026 | <https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html> |
| 4 | - Rà soát dashboard reader role và các giới hạn (Gold đã publish, metadata, Athena, Catalog, kết quả, pipeline lock) | 28/10/2026 | 28/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html> |
| 5 | - Rà soát các kiểm tra của dashboard trước và sau khi đọc Athena (publication cũ, đang chạy hoặc đã thay đổi) | 29/10/2026 | 29/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html> |
| 6 | - *(chưa cập nhật)* | 30/10/2026 | 30/10/2026 | |
| 7 | - Tham dự First Cloud AI Journey: Community Day tại Bitexco Financial Tower: sáu phiên về lộ trình học AWS, quy trình phát triển phần mềm có AI, game serverless xây dựng trong kỳ thực tập, Lambda Web Adapter, xử lý bất đồng bộ với SQS và đóng góp mã nguồn mở cho AWS CDK | 31/10/2026 | 31/10/2026 | <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html> |

### Kết quả đạt được trong tuần 18:

* Tham dự Community Day và ghi lại các thực hành với SQS (visibility timeout, DLQ, consumer idempotent) để áp dụng vào dự án của mình.
* Xác minh cách monitoring và dashboard reader phát hiện publication cũ hoặc không nhất quán.
