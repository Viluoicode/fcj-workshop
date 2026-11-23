---
title: "Worklog Tuần 22"
date: 2026-11-29
weight: 22
chapter: false
pre: " <b> 1.22. </b> "
---

### Mục tiêu tuần 22:

* Job Market: hoàn tất việc chuyển Gold sang Apache Iceberg.
* OnTime Radar: kiểm thử ingestion khi sự kiện đến sai thứ tự hoặc sai định dạng.
* *(chưa cập nhật)*

### Các công việc đã thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Job Market: sửa phép so Iceberg/Parquet (timestamp mức mili giây), ghi nhận ngày 1 khớp hoàn toàn, sau đó chuyển phần đọc sang Iceberg và bỏ Glue crawler <br> - OnTime Radar: trả 400 thay vì 500 khi payload sai cấu trúc; viết bộ sinh kịch bản chaos có đáp án độc lập; kiểm thử 100 thứ tự đến trong bộ nhớ; chạy chaos runner trên API đã deploy (2.375 webhook, 0 sai lệch, DLQ trống) <br> - Cập nhật Workshop trong báo cáo theo kiến trúc hiện tại của Job Market | 23/11/2026 | 23/11/2026 | <https://github.com/Viluoicode/Job-Market-Pipeline-cloud> <br> <https://github.com/Viluoicode/ontime-radar> <br> <https://iceberg.apache.org/docs/latest/> |
| 3 | | 24/11/2026 | 24/11/2026 | |
| 4 | | 25/11/2026 | 25/11/2026 | |
| 5 | | 26/11/2026 | 26/11/2026 | |
| 6 | | 27/11/2026 | 27/11/2026 | |

### Kết quả đạt được trong tuần 22:

* *(cập nhật vào cuối tuần)*
