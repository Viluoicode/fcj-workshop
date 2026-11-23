---
title: "Chi phí và dọn dẹp"
date: 2026-11-23
weight: 7
chapter: false
pre: " <b> 5.1.7. </b> "
---

# Chi phí và dọn dẹp

## Ước tính chi phí

{{% notice warning %}}
Phần này **đang làm**. Con số bên dưới là ước tính ban đầu. Báo cáo cuối cần chi phí đo được từ Cost Explorer và một bản ước tính AWS Pricing Calculator đã lưu, dựa trên lưu lượng thực tế đó.
{{% /notice %}}

Với replayer chạy mỗi 15 phút (khoảng 1.900 đơn và 9.000 sự kiện mỗi ngày), tôi ước tính **khoảng 3–5 USD mỗi tháng**. Phần lớn nhất là lượt ghi DynamoDB, vì mỗi sự kiện ghi một bản ghi trạng thái và một bản ghi lịch sử trong một transaction.

| Dịch vụ | Yếu tố tạo chi phí | Ghi chú |
|---|---|---|
| DynamoDB (on-demand) | lượt ghi cho mỗi sự kiện, cộng truy vấn từ dashboard | phần lớn nhất |
| Lambda (arm64) | số lần gọi API, processor và replayer | phần lớn nằm trong free tier |
| API Gateway (HTTP API) | request webhook và request từ dashboard | |
| SQS FIFO | message và lượt gửi lại | |
| S3 | event lake, bộ dữ liệu, website | dung lượng nhỏ |
| Athena | số byte đã quét | giới hạn 1 GB mỗi truy vấn, phần lớn truy vấn lấy từ cache |
| CloudFront, Cognito, CloudWatch, SNS | lưu lượng, người dùng, log, alarm, email | sử dụng thấp |

*(Cần bổ sung: số liệu Cost Explorer theo từng dịch vụ trong một tuần đầy đủ, và link tới bản ước tính Pricing Calculator đã lưu.)*

> 📷 *Ảnh cần bổ sung: Cost Explorer lọc theo các dịch vụ của dự án, nhóm theo dịch vụ.*

## Thiết kế giúp giảm chi phí thế nào

- **Không server, không VPC:** chỉ dùng Lambda và dịch vụ được quản lý sẵn, nên không có NAT gateway, vốn đắt hơn cả toàn bộ ứng dụng.
- **Lambda chạy arm64 (Graviton):** rẻ hơn x86 khoảng 20% với cùng thời gian chạy.
- **DynamoDB on-demand:** trả tiền theo request, không tốn gì khi hệ thống không có tải.
- **Giới hạn Athena:** tối đa 1 GB mỗi truy vấn, partition projection để chỉ quét đúng các ngày được hỏi, và cache hai lớp giúp 79% truy vấn báo cáo không phải quét.
- **Giới hạn tốc độ** ở API Gateway, nên một đợt dồn request bị từ chối trước khi chạy Lambda.
- **Giữ log 14 ngày** thay vì giữ vĩnh viễn.

## Dọn dẹp tài nguyên

Chỉ xóa tài nguyên sau khi dự án đã được chấm điểm. Làm theo thứ tự sau:

1. Dừng phát lại: đặt `ReplayState` thành `DISABLED` rồi deploy, hoặc tắt lịch trong EventBridge Scheduler.
2. Lưu lại bằng chứng cần giữ: các file daily-check, ảnh chụp và dữ liệu xuất từ Cost Explorer.
3. Làm rỗng hai bucket. CloudFormation không xóa được bucket còn chứa object.

   ```powershell
   aws s3 rm "s3://<DataBucketName>" --recursive
   aws s3 rm "s3://<web bucket>" --recursive
   ```

4. Xóa stack ứng dụng:

   ```powershell
   sam delete --stack-name shipment-sla
   ```

   Lệnh này xóa API, các Lambda, hàng đợi, bảng, Cognito user pool, CloudFront distribution, alarm, bảng Glue và workgroup Athena.
5. Xóa stack dùng để deploy `github-oidc-deploy` (OIDC provider, role deploy và boundary):

   ```powershell
   aws cloudformation delete-stack --stack-name github-oidc-deploy --region ap-southeast-1
   ```

6. Trên GitHub, xóa secret `AWS_DEPLOY_ROLE_ARN` và environment `production`.
7. Kiểm tra không còn gì sót lại: trên console, xem CloudFormation, S3, CloudFront và các log group của CloudWatch ở `ap-southeast-1`. Sau đó kiểm tra hóa đơn vào ngày hôm sau.
