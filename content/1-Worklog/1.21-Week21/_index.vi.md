---
title: "Worklog Tuần 21"
date: 2026-11-22
weight: 21
chapter: false
pre: " <b> 1.21. </b> "
---

### Mục tiêu tuần 21:

* Bắt đầu Project 2, OnTime Radar (Shipment SLA Service): backend serverless theo dõi vận đơn của nhiều hãng vận chuyển và phát hiện đơn giao trễ.
* Củng cố Project 1 (Job Market AWS Pipeline): đưa Terraform state lên S3, tự động gỡ lock, quét bảo mật trong CI và sửa một nguồn dữ liệu bị lỗi.

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - OnTime Radar: dựng pipeline đầu tiên: webhook của các hãng qua API Gateway, hàng đợi SQS FIFO để mỗi vận đơn được xử lý đúng thứ tự, DynamoDB lưu trạng thái và lịch sử, event lake trên S3, alarm cho đơn trễ và DLQ <br> - Thêm báo cáo SLA bằng Athena trên event lake, workflow CI và đăng nhập Cognito với hai vai trò (CSKH và quản lý) <br> - Sắp xếp lại worklog báo cáo thực tập thành giai đoạn lý thuyết và giai đoạn dự án | 16/11/2026 | 16/11/2026 | <https://github.com/Viluoicode/ontime-radar> <br> <https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/FIFO-queues.html> |
| 3 | - OnTime Radar: kiểm tra CloudFormation stack trên console, cấp và chỉnh quyền IAM cho user dùng để deploy, xem Billing and Cost Management <br> - Tìm hiểu hướng thiết kế giao diện cho dashboard <br> - Đăng ký AWS Student Community Day Vietnam 2026 (21/11) | 17/11/2026 | 17/11/2026 | <https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html> <br> <https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/billing-what-is.html> |
| 4 | - Dự án nhóm `kfc-force-agent`: tạo issue #12 và pull request thêm AI agent hiểu tiếng Việt, có tool use, dùng Amazon Bedrock AgentCore và DynamoDB | 18/11/2026 | 18/11/2026 | <https://github.com/HuynhHao080/kfc-force-agent> <br> <https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html> |
| 5 | - OnTime Radar: làm website dashboard và host trên S3 + CloudFront <br> - Thêm health check hằng ngày, đổi alarm đơn trễ sang dùng tỷ lệ kéo dài 2 trong 3 kỳ <br> - Vẽ lại sơ đồ kiến trúc cho khớp với hệ thống đã deploy | 19/11/2026 | 19/11/2026 | <https://github.com/Viluoicode/ontime-radar> <br> <https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html> |
| 6 | - OnTime Radar: deploy từ GitHub Actions qua OIDC có bước duyệt thủ công; thêm throttling cho API, permissions boundary cho mọi role ứng dụng, kiểm tra mã vận đơn, cache kết quả Athena và 66 test cho service <br> - Job Market: chuyển sang 15 nhóm vai trò (mỗi tin một nhóm), hình thức làm việc ba trạng thái, quy tắc chọn bản đại diện khi loại trùng, và khôi phục tài liệu tiếng Việt UTF-8 | 20/11/2026 | 20/11/2026 | <https://github.com/Viluoicode/ontime-radar> <br> <https://github.com/Viluoicode/Job-Market-Pipeline-cloud> <br> <https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services> |
| 7 | - Job Market: chuyển Terraform state lên S3, viết lại IAM runtime bằng HCL, thêm Lambda lock janitor, thêm ruff và Checkov vào CI (xử lý 41 finding), chuyển Mistral AI từ Lever sang Ashby, thêm script đo thời gian và chi phí mỗi lượt chạy | 21/11/2026 | 21/11/2026 | <https://github.com/Viluoicode/Job-Market-Pipeline-cloud> <br> <https://developer.hashicorp.com/terraform/language/backend/s3> |
| CN | - Job Market: gọi API các board song song (5 worker), ghi shadow Gold sang Apache Iceberg, che account ID trong các file evidence, viết lại README kèm sơ đồ kiến trúc AWS, đặt thời hạn lưu 30 ngày cho log Glue và phiên bản S3 cũ | 22/11/2026 | 22/11/2026 | <https://github.com/Viluoicode/Job-Market-Pipeline-cloud> <br> <https://docs.aws.amazon.com/athena/latest/ug/querying-iceberg.html> |

### Kết quả đạt được trong tuần 21:

* OnTime Radar đã được deploy: nhận webhook, xử lý đúng thứ tự, báo cáo SLA, dashboard có đăng nhập được host sẵn, alarm và deploy qua OIDC có bước duyệt.
* Job Market giờ tự gỡ lock mồ côi mà không cần thao tác tay, lưu Terraform state trên S3 và đạt bước quét Checkov trong CI.
* Phát hiện và sửa một nguồn đã trả HTTP 404 từ ngày 6/11.
* Đóng góp tính năng agent dùng AgentCore + DynamoDB cho một dự án nhóm.
