---
title: "Dashboard, Testing và kết quả"
date: 2026-11-23
weight: 6
chapter: false
pre: " <b> 5.2.6. </b> "
---

# Dashboard, Testing và kết quả

## Dashboard kiểm tra gì trước khi hiển thị

Ứng dụng Streamlit luôn chuyển sang role chỉ đọc của dashboard trước. Sau đó nó:

1. Đảm bảo không có lượt chạy nào đang giữ lock.
2. Đọc completion marker, con trỏ Silver và manifest tương ứng.
3. Kiểm tra cả ba khớp run ID và ngày UTC, ingestion đã thành công và dữ liệu chưa quá 26 giờ.
4. Truy vấn `fact_job_posting` bằng `FOR VERSION AS OF` với snapshot ID ghi trong marker.
5. Lặp lại bước 1–3 sau khi truy vấn. Nếu có gì thay đổi ở giữa, nó bỏ kết quả và hiển thị thông báo thay vì dữ liệu.

Dashboard không bao giờ lấy tạm "ngày mới nhất trong bảng". Dữ liệu cũ hoặc publish dở dang bị từ chối, không được hiển thị.

## Dashboard hiển thị gì

- Bộ lọc theo nền tảng, nhóm vai trò, địa điểm và hình thức làm việc, cùng ô tìm theo chức danh hoặc công ty.
- Số tin, số công ty, số địa điểm và tỷ lệ remote.
- Số tin theo nhóm vai trò và các công ty tuyển nhiều nhất.
- Bảng danh sách tin kèm link đến trang tuyển dụng gốc.
- Thời điểm publish, tuổi dữ liệu, độ phủ board, Athena query ID và số byte đã quét.

> 📷 *Ảnh cần bổ sung: màn hình tổng quan dashboard và bảng danh sách tin (repository đã có `dashboard-overview.png` và `dashboard-postings.png` chụp ngày 21/11).*

## Mở dashboard trên máy

```powershell
$env:AWS_REGION = 'ap-southeast-1'
$env:LAKE_BUCKET = terraform -chdir=infra output -raw lake_bucket
$env:PIPELINE_LOCK_TABLE = terraform -chdir=infra output -raw pipeline_lock_table
$env:ATHENA_DATABASE = terraform -chdir=infra output -raw gold_database
$env:ATHENA_WORKGROUP = terraform -chdir=infra output -raw athena_workgroup
$env:AWS_DASHBOARD_ROLE_ARN = terraform -chdir=infra output -raw dashboard_reader_role_arn
.\.venv\Scripts\python.exe -m streamlit run dashboard/app.py --server.address 127.0.0.1
```

Sau đó mở `http://localhost:8501` và bấm **Refresh data**. Xem dữ liệu không deploy gì và không khởi động lượt chạy nào.

## Kiểm thử tự động

Mỗi pull request chạy `ruff`, 251 test pytest, `terraform validate` và quét Checkov trên GitHub Actions. Các test giả lập job API và AWS, bao gồm:

- ingestion: gọi lại khi lỗi, board lỗi và board rỗng, giới hạn trang, thứ tự khi lấy song song, thứ tự commit manifest;
- Silver: định danh, loại trùng, mọi trường hợp lifecycle ở 5.2.4 và phần chuẩn bị Data Quality;
- Gold: ba bảng mart, nhóm vai trò, ghi đè ngày rỗng và ghi Iceberg;
- dashboard và hai Lambda: lock, publication cũ hoặc bị thay đổi, quyết định của janitor;
- chính repository: các file evidence là JSON hợp lệ và không chứa AWS account ID.

## Kiểm thử kịch bản lỗi trên AWS

| Kịch bản | Cách tạo ra | Kết quả mong đợi |
|---|---|---|
| Lượt chạy thứ hai khi đang có lượt chạy | Khởi động execution thủ công trong lúc lượt chạy đêm đang chạy | Kết thúc ở `PipelineBusy`, dữ liệu không đổi |
| Một board ngừng trả lời | Đã xảy ra thật với Lever `mistral` hồi tháng 11 | `source-coverage-low` chuyển sang ALARM; Silver giữ các tin của board ở trạng thái chưa xác minh |
| Dữ liệu bị cũ | Tạm dừng lịch chạy hơn 26 giờ | `pipeline-unhealthy` báo động, dashboard từ chối hiển thị dữ liệu |
| Lượt chạy bị hủy | Dừng một execution trên console | `pipeline-aborted` báo động; janitor nhả lock khi Glue đã dừng |

*(Cần bổ sung: ngày bạn chạy từng kịch bản, kèm ảnh chụp hoặc dòng log làm bằng chứng.)*

## Kết quả đo được

Đây là các quan sát có ghi ngày, không phải trạng thái trực tiếp:

| Chỉ số | Giá trị |
|---|---|
| Lượt chạy theo lịch thành công, 15/09 – 21/11/2026 | 68/68 |
| Số tin đang mở ngày 21/11/2026 | 8.672 tin từ 467 công ty |
| Số board thu thập được ngày 21/11/2026 | 36/36 |
| Thời gian từ lúc kích hoạt đến khi Gold được publish (trung vị, 14–20/11) | 9 phút |
| Chi phí Glue mỗi lượt (trước credit) | khoảng 0,11 USD |
| Số byte quét của một truy vấn dashboard | khoảng 1 MiB |

Thời gian chạy và chi phí được đo bằng `scripts/run_metrics.py` trước khi thêm lấy dữ liệu song song và Iceberg. Thay đổi đó cũng bỏ crawler, nên chi phí mỗi lượt hiện thấp hơn. Gold có thể có nhiều dòng hơn Bronze của một lượt chạy, vì tin của board bị lỗi vẫn giữ quan sát cuối khi còn trong khoảng thời gian mới.
