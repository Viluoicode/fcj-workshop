---
title: "OnTime Radar - Shipment SLA Service"
date: 2026-11-23
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# OnTime Radar - Shipment SLA Service

## Vấn đề

Một shop bán hàng online ở Việt Nam thường giao hàng qua hai đến bốn hãng vận chuyển cùng lúc, ví dụ GHN, GHTK và J&T. Mỗi hãng có bộ mã trạng thái riêng, trang tra cứu riêng và định dạng webhook riêng. Thực tế điều này gây ra bốn vấn đề:

- Nhân viên chăm sóc khách hàng phải mở nhiều trang web chỉ để trả lời một câu: "đơn của tôi đang ở đâu?"
- Shop chỉ biết đơn bị giao trễ khi khách phàn nàn.
- Không ai nói được hãng nào giao đúng hạn ở khu vực nào, nên việc chọn hãng dựa vào cảm tính.
- Webhook có thể đến hai lần hoặc sai thứ tự, khiến trạng thái đơn bị nhảy lùi.

## Tôi đã xây dựng gì

OnTime Radar là một backend serverless kèm dashboard web. Hệ thống nhận sự kiện trạng thái từ mọi hãng và giữ một dòng thời gian đáng tin cậy cho mỗi vận đơn:

1. **Một mô hình trạng thái chung.** Mỗi hãng có một adapter nhỏ để chuyển mã của hãng về tám trạng thái chung.
2. **Đúng thứ tự, không trùng lặp.** Sự kiện của cùng một vận đơn được xử lý lần lượt. Sự kiện lặp lại bị bỏ qua, còn cập nhật đến muộn hoặc vô lý được ghi lại nhưng không làm đổi trạng thái.
3. **Tra cứu ở một chỗ.** Một lần gọi API trả về trạng thái hiện tại và toàn bộ lịch sử của vận đơn.
4. **Cảnh báo sớm.** Khi tỷ lệ đơn giao trễ duy trì ở mức cao, CloudWatch bật alarm và SNS gửi email kèm link tới danh sách đơn cần xử lý.
5. **Báo cáo SLA.** Mọi sự kiện được lưu thêm vào S3, và Athena trả lời các câu hỏi như "tỷ lệ đúng hạn theo hãng và khu vực trong 30 ngày qua".
6. **Dashboard có phân quyền.** Nhân viên đăng nhập bằng Cognito. Tài khoản CSKH được tra cứu đơn, còn tài khoản quản lý xem thêm được báo cáo SLA.

Lưu lượng là dữ liệu thật được phát lại: một Lambda chạy theo lịch phát lại khoảng 98.000 đơn hàng lịch sử của bộ dữ liệu công khai Olist dưới dạng webhook của hãng, mỗi 15 phút một lần. Ngày đặt hàng, ngày giao, ngày hẹn giao và khu vực lấy từ bộ dữ liệu. Tên hãng, mã vận đơn và các bước đang vận chuyển là giả lập.

## Ai sử dụng

| Người dùng | Vai trò đăng nhập | Họ cần gì |
|---|---|---|
| Chăm sóc khách hàng | `cskh` | Trả lời "đơn tôi đâu?" trong vài giây |
| Nhân viên vận hành | `cskh` | Biết ngay đơn nào đang gặp vấn đề |
| Quản lý vận hành | `manager` | So sánh các hãng theo khu vực bằng số liệu thật |
| Hệ thống của hãng vận chuyển | webhook secret | Đẩy cập nhật trạng thái |

## Các dịch vụ AWS sử dụng

API Gateway (HTTP API), AWS Lambda (.NET 10, arm64), Amazon SQS FIFO, Amazon DynamoDB, Amazon S3, AWS Glue Data Catalog, Amazon Athena, Amazon EventBridge Scheduler, Amazon Cognito, Amazon CloudFront, Amazon CloudWatch, Amazon SNS, AWS IAM và AWS SAM / CloudFormation. Việc deploy chạy từ GitHub Actions qua OIDC.

## Trạng thái dự án (23/11/2026)

| Phần | Trạng thái |
|---|---|
| Webhook, xử lý đúng thứ tự, timeline trên DynamoDB, event lake trên S3 | Đã deploy và đang chạy |
| Phát lại dữ liệu Olist mỗi 15 phút | Chạy từ 16/11; đến 23/11 đã gửi 13.580/98.202 đơn |
| Báo cáo SLA bằng Athena và API | Đã deploy |
| Dashboard web trên CloudFront có đăng nhập Cognito | Đã deploy |
| Alarm và email cảnh báo | Đã deploy; đã xác nhận đăng ký email |
| CI/CD qua GitHub Actions + OIDC có bước duyệt | Hoạt động từ 20/11 |
| Kiểm thử chaos trên API đã deploy | Đạt ngày 23/11 |
| Chi phí từ Cost Explorer và Pricing Calculator | **Đang làm** |
| Video demo | **Đang làm** (đã có kịch bản) |

## Phát triển có AI hỗ trợ

Tôi dùng công cụ AI để tìm hiểu, gợi ý code, review code, đề xuất ý tưởng kiểm thử và soạn nháp tài liệu. Tôi tự quyết định yêu cầu, review mọi thay đổi, deploy và kiểm tra hệ thống trên AWS, và xác minh các con số được ghi trong workshop này.

## Các phần

1. [Kiến trúc](5.1.1-architecture/)
2. [Chuẩn bị và triển khai](5.1.2-deploy/)
3. [Nhận và xử lý sự kiện](5.1.3-processing/)
4. [Báo cáo SLA bằng Athena](5.1.4-analytics/)
5. [Dashboard, đăng nhập và bảo mật](5.1.5-security/)
6. [Giám sát và kiểm thử](5.1.6-testing/)
7. [Chi phí và dọn dẹp](5.1.7-cleanup/)
8. [Xử lý sự cố và hướng phát triển](5.1.8-troubleshooting/)
