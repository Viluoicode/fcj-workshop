---
title: "Worklog Tuần 17"
date: 2026-10-25
weight: 17
chapter: false
pre: " <b> 1.17. </b> "
---

### Mục tiêu tuần 17:

* Vẽ phần orchestration, writer lock và ranh giới IAM trong sơ đồ kiến trúc.
* Đối chiếu sơ đồ với code Terraform.

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Vẽ workflow Step Functions (ingestion, Silver, Gold, xác minh crawler, completion marker) | 19/10/2026 | 19/10/2026 | <https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html> |
| 3 | - Vẽ DynamoDB writer lock và lịch chạy hằng ngày của EventBridge | 20/10/2026 | 20/10/2026 | <https://docs.aws.amazon.com/step-functions/latest/dg/welcome.html> |
| 4 | - Vẽ ranh giới các IAM role (ingestion, Silver, Gold, crawler, orchestration, monitoring, dashboard reader) | 21/10/2026 | 21/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html> |
| 5 | - Đối chiếu sơ đồ với các file Terraform trong infra/ và sửa chỗ chưa khớp | 22/10/2026 | 22/10/2026 | <https://developer.hashicorp.com/terraform/docs> |
| 6 | - *(chưa cập nhật)* | 23/10/2026 | 23/10/2026 | |

### Kết quả đạt được trong tuần 17:

* Hoàn thành bản nháp sơ đồ kiến trúc dùng icon AWS gồm luồng dữ liệu, luồng điều khiển và ranh giới IAM.
* Bản xuất PNG vẫn chưa có (dự kiến Tuần 21).
