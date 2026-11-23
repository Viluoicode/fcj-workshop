---
title: "Worklog Tuần 16"
date: 2026-10-18
weight: 16
chapter: false
pre: " <b> 1.16. </b> "
---

### Mục tiêu tuần 16:

* Rà soát các Gold mart, Glue Catalog và tầng Athena đã xây dựng trước đó.
* Bắt đầu vẽ sơ đồ kiến trúc dùng icon AWS cho dự án.

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Rà soát fact_job_posting và quy tắc deduplicate giữa các nguồn | 12/10/2026 | 12/10/2026 | <https://docs.aws.amazon.com/athena/latest/ug/what-is.html> |
| 3 | - Rà soát demand_by_role, role_opportunity và các nhóm role theo title | 13/10/2026 | 13/10/2026 | <https://docs.aws.amazon.com/athena/latest/ug/what-is.html> |
| 4 | - Rà soát Glue Crawler, Data Catalog, Athena workgroup và giới hạn scan; chạy các truy vấn phân tích trong sql/athena_analysis.sql | 14/10/2026 | 14/10/2026 | <https://docs.aws.amazon.com/athena/latest/ug/what-is.html> |
| 5 | - Liệt kê các dịch vụ, luồng dữ liệu và luồng điều khiển cần vẽ; bắt đầu sơ đồ draw.io | 15/10/2026 | 15/10/2026 | |
| 6 | - *(chưa cập nhật)* | 16/10/2026 | 16/10/2026 | |

### Kết quả đạt được trong tuần 16:

* Giải thích được ba Gold mart và cách Athena đọc partition đã publish.
* Bắt đầu sơ đồ kiến trúc (docs/diagrams/AWS architecture.drawio).
