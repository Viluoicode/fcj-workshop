---
title: "Worklog Tuần 14"
date: 2026-10-04
weight: 14
chapter: false
pre: " <b> 1.14. </b> "
---

### Mục tiêu tuần 14:

* Bổ sung test tự động và dashboard reader, redeploy và ghi lại bằng chứng kiểm chứng.
* Tăng cường bảo mật: tách role theo từng stage, giới hạn dashboard reader và kiểm tra policy.

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Thêm test cho dashboard, kiểm tra freshness và health monitor; xây dựng dashboard reader (AssumeRole, kiểm tra publication); redeploy và ghi lại bằng chứng kiểm chứng | 28/09/2026 | 28/09/2026 | <https://docs.pytest.org/> |
| 3 | - Rà soát cấu hình bucket private, mã hóa, versioning, HTTPS-only và retention | 29/09/2026 | 29/09/2026 | <https://developer.hashicorp.com/terraform/docs> |
| 4 | - Chạy kiểm tra quyền IAM và security acceptance; lưu bằng chứng có ghi ngày | 30/09/2026 | 30/09/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html> |
| 5 | - Tách role Silver, Gold và crawler; kiểm tra policy bằng IAM Access Analyzer; chụp ảnh dashboard và chạy handoff check | 01/10/2026 | 01/10/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/what-is-access-analyzer.html> |
| 6 | - *(chưa cập nhật)* | 02/10/2026 | 02/10/2026 | |

### Kết quả đạt được trong tuần 14:

* Bổ sung các module test tự động và dashboard reader có giới hạn quyền.
* Triển khai security hardening với role riêng cho từng stage và truy cập bucket chỉ qua HTTPS.
* Lưu bằng chứng có ghi ngày cho validation, permissions và security acceptance.
