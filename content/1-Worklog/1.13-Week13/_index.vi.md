---
title: "Worklog Tuần 13"
date: 2026-09-27
weight: 13
chapter: false
pre: " <b> 1.13. </b> "
---

### Mục tiêu tuần 13:

* Chính thức hóa Project 1 (prototype bắt đầu từ cuối tháng 6/2026, trước thực tập) bằng việc xác định lại bài toán dữ liệu tuyển dụng và phạm vi dự án.
* Ôn lại Infrastructure as Code và lựa chọn phương pháp triển khai có thể tái sử dụng.
* Thiết kế kiến trúc AWS data pipeline ban đầu.

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Ôn lại Infrastructure as Code và so sánh template có thể tái sử dụng với việc tạo tài nguyên thủ công trên AWS Console | 21/09/2026 | 21/09/2026 | <https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html> |
| 3 | - Lên văn phòng, kết nối với các bạn thực tập sinh, trao đổi định hướng dự án và đọc tài liệu về kiến trúc data pipeline trên AWS | 22/09/2026 | 22/09/2026 | <https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html> |
| 4 | - Phân tích bốn nền tảng API công khai gồm Greenhouse, Lever, Ashby và Arbeitnow; xác định sự khác nhau giữa các schema | 23/09/2026 | 23/09/2026 | <https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html> |
| 5 | - Xác định yêu cầu dự án: ingestion hằng ngày, traceability, freshness, lifecycle, data quality, analytics và monitoring | 24/09/2026 | 24/09/2026 | <https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html> |
| 6 | - Phác thảo kiến trúc Bronze-Silver-Gold và ánh xạ từng dịch vụ AWS với trách nhiệm tương ứng | 25/09/2026 | 25/09/2026 | <https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html> |

### Kết quả đạt được trong tuần 13:

* Xác định bài toán, phạm vi, đối tượng sử dụng và tiêu chí hoàn thành cho Project 1.
* Lựa chọn kiến trúc batch serverless sử dụng S3, Glue, Step Functions, Athena và các dịch vụ hỗ trợ.
* Hoàn thành kiến trúc ban đầu và các giai đoạn triển khai dự án.
