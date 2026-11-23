---
title: "Workshop"
date: 2026-11-23
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Workshop: OnTime Radar

Workshop này trình bày cách tôi xây dựng **OnTime Radar**, một dịch vụ serverless trên AWS theo dõi vận đơn của nhiều hãng vận chuyển và cảnh báo nhân viên khi đơn bắt đầu bị giao trễ. Hệ thống chạy trên tài khoản AWS của tôi từ ngày 16/11/2026, cứ 15 phút lại được nạp các đơn hàng lịch sử thật từ bộ dữ liệu công khai Olist.

#### Vì sao chọn đề tài này

Một shop giao hàng qua nhiều hãng gặp cùng vài vấn đề mỗi ngày. Mỗi hãng báo trạng thái theo định dạng riêng. Cập nhật đến hai lần hoặc sai thứ tự. Nhân viên CSKH phải xem nhiều trang web để trả lời một câu hỏi. Và đơn giao trễ chỉ được phát hiện khi khách phàn nàn. OnTime Radar gom sự kiện của mọi hãng thành một dòng thời gian đáng tin cậy cho mỗi vận đơn, đo tỷ lệ giao đúng hạn theo hãng và khu vực, và gửi email cảnh báo khi tỷ lệ trễ tăng cao.

#### Bạn sẽ xây dựng được gì

- Một API nhận webhook theo ba định dạng của các hãng và giữ đúng thứ tự sự kiện của từng vận đơn.
- Một dòng thời gian trên DynamoDB bỏ qua bản trùng và từ chối các thay đổi trạng thái vô lý.
- Event lake trên S3 và báo cáo SLA bằng Athena.
- Dashboard web trên CloudFront, đăng nhập bằng Cognito với hai vai trò nhân viên.
- CloudWatch alarm kèm email cảnh báo, kiểm tra sức khỏe hằng ngày và kiểm thử chaos.
- Deploy tự động từ GitHub Actions qua OIDC, không lưu access key.

#### Dành cho ai

Người đã nắm cơ bản về Lambda, IAM và S3, muốn xem một hệ thống hướng sự kiện xử lý dữ liệu đầu vào lộn xộn của thực tế. Bạn cần tài khoản AWS, AWS SAM CLI và .NET 10. Mục 5.1.2 liệt kê đầy đủ công cụ.

#### Nội dung

1. [OnTime Radar - Shipment SLA Service](5.1-ontimeradar/)
   1. [Kiến trúc](5.1-ontimeradar/5.1.1-architecture/)
   2. [Chuẩn bị và triển khai](5.1-ontimeradar/5.1.2-deploy/)
   3. [Nhận và xử lý sự kiện](5.1-ontimeradar/5.1.3-processing/)
   4. [Báo cáo SLA bằng Athena](5.1-ontimeradar/5.1.4-analytics/)
   5. [Dashboard, đăng nhập và bảo mật](5.1-ontimeradar/5.1.5-security/)
   6. [Giám sát và kiểm thử](5.1-ontimeradar/5.1.6-testing/)
   7. [Chi phí và dọn dẹp](5.1-ontimeradar/5.1.7-cleanup/)
   8. [Xử lý sự cố và hướng phát triển](5.1-ontimeradar/5.1.8-troubleshooting/)

#### Dự án bổ sung

2. [Job Market AWS Pipeline](5.2-jobmarket/): pipeline dữ liệu tôi xây dựng trước đó trong kỳ thực tập (Glue, Step Functions, Iceberg, Athena), giữ lại ở đây như một case study thứ hai, ngắn hơn.
