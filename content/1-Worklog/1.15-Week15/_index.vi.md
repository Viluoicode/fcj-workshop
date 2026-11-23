---
title: "Worklog Tuần 15"
date: 2026-10-11
weight: 15
chapter: false
pre: " <b> 1.15. </b> "
---

### Mục tiêu tuần 15:

* Hệ thống hóa thiết kế Silver gồm identity, lifecycle và Data Quality đã được triển khai trước đó.
* Học thêm về Agentic AI và data product tại sự kiện AWS.

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Rà soát các trường Silver chuẩn, job_id ổn định và thiết kế dedup_hash giữa các nguồn | 05/10/2026 | 05/10/2026 | <https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html> |
| 3 | - Rà soát deduplication trong cùng nguồn, chuẩn hóa timestamp và kiểm tra trường bắt buộc | 06/10/2026 | 06/10/2026 | <https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html> |
| 4 | - Rà soát cơ chế lifecycle reconciliation với first_seen_at, last_seen_at, trạng thái active và status reason | 07/10/2026 | 07/10/2026 | <https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html> |
| 5 | - Tham dự AWS for Software and Technology Companies Day và Track B về Kiro và Amazon Bedrock AgentCore | 08/10/2026 | 08/10/2026 | <https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html> |
| 6 | - Rà soát bảy quy tắc data quality của Silver và quy tắc chỉ publish Silver latest pointer sau khi validation thành công | 09/10/2026 | 09/10/2026 | <https://docs.aws.amazon.com/glue/latest/dg/what-is-glue.html> |

### Kết quả đạt được trong tuần 15:

* Giải thích được thiết kế Silver (identity, lifecycle, Data Quality) như đã triển khai trong repository.
* Liên hệ các bài học về data product sẵn sàng cho AI và governance từ sự kiện AWS với dự án.
