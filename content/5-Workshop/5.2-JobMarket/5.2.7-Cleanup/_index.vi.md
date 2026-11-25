---
title: "Chi phí, Cleanup và Reflection"
date: 2026-11-12
weight: 7
chapter: false
pre: " <b> 5.2.7. </b> "
---

# Chi phí, Cleanup, giới hạn và Reflection

## Kiểm soát chi phí

Dự án sử dụng batch processing hằng ngày, serverless query, Glue worker nhỏ, stage timeout, retention rule, Athena scan limit và monthly budget alert.

### Chi phí đo được

`scripts/run_metrics.py` đã đo 7 lượt chạy theo lịch trong ngày 14–20/11/2026 và đối chiếu với Cost Explorer. Đây là giá niêm yết trước khi trừ credit:

| Hạng mục | Mỗi lượt (trung vị) |
|---|---:|
| Glue job (3 job, 517 DPU-giây) | 0,064 USD (Cost Explorer: 0,0635 USD) |
| Glue crawler | 0,049 USD |
| **Tổng Glue** | **khoảng 0,11 USD** |
| Lambda, Step Functions, EventBridge, 6 alarm | nằm trong free tier |
| S3, DynamoDB | vài cent mỗi ngày |

Với một lượt mỗi ngày, chi phí Glue khoảng **3,4 USD mỗi tháng**. Các lượt này được đo trước khi thêm lấy dữ liệu song song và Iceberg. Thay đổi đó đã bỏ crawler, vốn chiếm khoảng 44% chi phí Glue mỗi lượt, nên chi phí hiện tại khoảng 2 USD mỗi tháng. Con số thấp hơn này là ước tính cho tới khi đo lại một tuần đầy đủ.

*(Cần bổ sung: link tới bản ước tính AWS Pricing Calculator đã lưu, dựa trên các số liệu này.)*

Budget alert chỉ thông báo và không tự dừng chi phí.

## Storage retention

| Dữ liệu | Retention mặc định |
|---|---:|
| Bronze và ingestion manifest | 30 ngày |
| Quality result | 90 ngày |
| Athena query result | 7 ngày |
| Glue temporary data | 7 ngày |
| Silver, Gold và state | Không tự động hết hạn |

Noncurrent object đã versioning vẫn có thể phát sinh chi phí sau khi current object hết hạn.

## Cleanup an toàn

1. Disable EventBridge pipeline schedule.
2. Xác nhận không có Step Functions execution, Glue job hoặc crawler đang chạy.
3. Kiểm tra DynamoDB lock và làm theo recovery runbook nếu run bị gián đoạn.
4. Export evidence cần giữ lại.
5. Kiểm tra `terraform plan -destroy`.
6. Chỉ bật bucket cleanup option sau khi xác nhận dữ liệu được giữ lại có thể bị xóa.
7. Chạy `terraform destroy`.
8. Xác minh subscription, alarm, log group, bucket và scheduled rule đã được xóa đúng dự kiến.

Destroy stack có thể xóa dữ liệu đang giữ. Không thực hiện cleanup khi chưa kiểm tra target chính xác và evidence cần lưu.

## Giới hạn đã biết

- Arbeitnow pagination bị giới hạn hai trang.
- Gold publication không atomic trên cả ba marts.
- Role classification dùng title rule và có thể chồng lặp.
- Remote metadata bị thiếu không chứng minh posting là onsite.
- Configured source không đại diện toàn bộ thị trường lao động.
- Local dashboard không phải public multi-user application.
- Kết quả lịch sử không chứng minh pipeline hiện tại healthy.

## Reflection cá nhân

Dự án giúp tôi kết nối các dịch vụ AWS riêng lẻ thành một hệ thống dữ liệu có thể vận hành. Bài học quan trọng nhất là xử lý dữ liệu thành công vẫn chưa đủ: hệ thống phải giữ observation identity, phân biệt source failure với thay đổi nghiệp vụ, áp dụng quality trước khi publish, giới hạn permission, tạo health signal và cho consumer cách từ chối kết quả stale.

AI hỗ trợ tăng tốc nghiên cứu, triển khai, review và tài liệu, nhưng đồng thời làm validation quan trọng hơn. Tôi cần kiểm tra code được đề xuất, so sánh với hành vi dịch vụ AWS, chạy test, review permission và xác minh kết quả thay vì chấp nhận generated output khi chưa có evidence.

## Các phần còn lại trước khi nộp

- Screenshot cho các bước deployment và validation chính (sơ đồ kiến trúc và ảnh dashboard đã xong).
- AWS Pricing Calculator estimate đã lưu.
- Ví dụ troubleshooting cá nhân và chi tiết đóng góp đã xác nhận.
- Kiểm tra cuối caption ảnh và thuật ngữ song ngữ.

