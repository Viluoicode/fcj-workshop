---
title: "Dự án 1 - Job Market AWS Pipeline"
date: 2026-11-12
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Job Market AWS Pipeline

## Tổng quan

Job Market AWS Pipeline là workshop data engineering trên AWS, thu thập tin tuyển dụng từ các public API, lưu các observation có thể truy vết trong data lake, chuyển đổi qua ba tầng Bronze, Silver và Gold, sau đó cung cấp dữ liệu đã publish cho Amazon Athena và dashboard Streamlit chạy tại máy local.

Dự án giải quyết một bài toán thực tế: các nền tảng tuyển dụng sử dụng schema khác nhau, source có thể lỗi hoặc trả về dữ liệu không đầy đủ, và một posting có thể thay đổi hoặc biến mất theo thời gian. Một script tải dữ liệu thành công chưa đủ để chứng minh dữ liệu còn mới, đầy đủ, đã deduplicate hoặc an toàn để publish.

## Lịch sử dự án

Bản prototype đầu tiên được bắt đầu từ cuối tháng 6/2026, trước khi thực tập bắt đầu. Trong thời gian thực tập (Tuần 13-24), dự án được xác định lại phạm vi, thiết kế lại, hardening và triển khai trên AWS với Terraform, Data Quality gate, orchestration, monitoring và test.

## Mục tiêu dự án

- Thu thập một mẫu dữ liệu tuyển dụng được cấu hình mỗi ngày mà không phụ thuộc vào máy cá nhân.
- Lưu source observation và committed manifest để truy vết.
- Chuẩn hóa dữ liệu và theo dõi posting lifecycle trong tầng Silver.
- Áp dụng Data Quality trước khi cập nhật published state.
- Tạo Gold data product có thể truy vấn theo job, role, company, location và remote.
- Tự động hóa workflow và ngăn overlapping writer.
- Theo dõi freshness và source coverage độc lập với pipeline chính.
- Cung cấp dashboard có khả năng từ chối publication stale, chưa hoàn tất hoặc đang chạy.

## Phạm vi

Mẫu dữ liệu được cấu hình gồm 36 boards thuộc Greenhouse, Lever, Ashby và Arbeitnow. Hệ thống được xây dựng như một batch pipeline phục vụ học tập và portfolio. Dự án không khẳng định đại diện toàn bộ thị trường việc làm, không đề xuất việc làm cá nhân và không đảm bảo mọi posting vẫn còn mở trên website của nhà tuyển dụng.

## Các dịch vụ AWS chính

- Amazon S3
- AWS Glue Python Shell và AWS Glue Spark
- AWS Step Functions
- Amazon EventBridge
- Amazon DynamoDB
- AWS Glue Data Catalog (bảng Gold dạng Apache Iceberg)
- Amazon Athena
- AWS Lambda
- Amazon CloudWatch
- Amazon SNS
- AWS IAM

## Quá trình phát triển có AI hỗ trợ

AI hỗ trợ nghiên cứu, đề xuất triển khai, review code, xây dựng ý tưởng kiểm thử và tài liệu. Tôi chịu trách nhiệm xác định yêu cầu, kiểm tra quyết định kiến trúc, xác minh code được đề xuất, kiểm tra hành vi trên AWS, xử lý lỗi, kiểm thử pipeline và ghi lại các kết quả đã được kiểm chứng.

## Các phần của Workshop

1. Kiến trúc và quyết định thiết kế
2. Prerequisite và chuẩn bị môi trường
3. Ingestion từ nhiều nguồn và Bronze storage
4. Silver lifecycle, Data Quality và Gold marts
5. Orchestration, security và monitoring
6. Dashboard, testing và kết quả đã kiểm chứng
7. Chi phí, cleanup, giới hạn và reflection
8. Xử lý sự cố và hướng phát triển

## Evidence hiện có

Từ 15/09 đến 21/11/2026, cả 68 lượt chạy đêm theo lịch đều thành công. Snapshot ngày 21/11 có 8.672 tin đang mở từ 467 công ty, thu thập từ 36/36 board. Các con số này mô tả một thời điểm trong quá khứ; mục 5.2.6 hướng dẫn cách kiểm tra trạng thái hiện tại của pipeline.

