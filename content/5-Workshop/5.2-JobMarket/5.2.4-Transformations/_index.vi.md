---
title: "Silver, Data Quality và Gold"
date: 2026-11-23
weight: 4
chapter: false
pre: " <b> 5.2.4. </b> "
---

# Silver, Data Quality và Gold

## Từ Bronze sang Silver

Job Silver chỉ đọc các file có tên trong manifest của lượt chạy này. Trước khi bắt đầu, nó kiểm tra manifest thuộc đúng lượt chạy và đúng ngày UTC, lượt chạy đã thành công, và manifest còn mới.

### Gán định danh cho mỗi tin

- `job_id` là mã băm SHA-256 của nền tảng, board và mã tin của nền tảng. Board nằm trong khóa vì hai công ty trên cùng nền tảng có thể dùng trùng mã nội bộ.
- `dedup_hash` được tạo từ tên công ty, chức danh và địa điểm đã chuẩn hóa. Gold dùng nó để nhận ra cùng một công việc được đăng trên hai nền tảng.

### Xác định tin còn mở hay đã đóng

Job so sánh dữ liệu hôm nay với lượt Silver trước đó:

| Điều xảy ra hôm nay | Silver ghi nhận |
|---|---|
| Tin được nhìn thấy | Giữ thời điểm thấy lần đầu, cập nhật thời điểm thấy gần nhất, đánh dấu active |
| Board trả về đầy đủ nhưng không có tin này | Đã đóng: vắng mặt trong board đầy đủ |
| Board lỗi hoặc bị cắt ngắn | Giữ quan sát cuối: board chưa được xác minh |
| Bảy ngày không thấy | Đã đóng: quan sát quá cũ |
| Tin đã đóng xuất hiện lại | Mở lại và giữ nguyên thời điểm thấy lần đầu |

Quy tắc quan trọng nhất là board lỗi không bao giờ làm đóng tin nào. Nếu Lever sập một ngày, các tin của nó giữ quan sát cuối, thay vì trông như hàng loạt tin vừa đóng.

### Cổng kiểm tra chất lượng dữ liệu

Trước khi publish Silver, Glue Data Quality kiểm tra bảy quy tắc:

1. Có ít nhất một dòng.
2. Mọi dòng đều có `job_id`.
3. Không `job_id` nào bị lặp.
4. Mọi dòng đều có `dedup_hash`.
5. Ít nhất 30% dòng biết được hình thức làm việc. Ngưỡng thấp vì nhiều board không ghi thông tin này.
6. Nền tảng thuộc một trong bốn nền tảng được hỗ trợ.
7. Ít nhất 90% dòng có tên công ty.

Nếu có quy tắc không đạt, lượt chạy dừng và Silver cũ vẫn là bản đang được publish. Chỉ khi các quy tắc đạt, job mới ghi lượt chạy mới và chuyển `state/silver/latest.json` trỏ tới nó.

## Từ Silver sang Gold

Gold giữ các tin đang active và được nhìn thấy trong khoảng thời gian còn mới. Có ba bảng:

| Bảng | Mỗi dòng là | Dùng để |
|---|---|---|
| `fact_job_posting` | một tin đang mở, không trùng | xem tin, công ty, địa điểm, hình thức làm việc và link ứng tuyển |
| `demand_by_role` | một nhóm vai trò trong một ngày | so sánh nhu cầu giữa các vai trò |
| `role_opportunity` | một nhóm vai trò trong một ngày | xếp hạng nhu cầu, tỷ lệ remote và công ty tuyển nhiều nhất cho mỗi vai trò |

Khi cùng một công việc xuất hiện trên hai nền tảng, Gold giữ bản được thấy gần nhất, sau đó đến bản có ngày đăng, rồi đến bản có `job_id` nhỏ nhất, để lựa chọn luôn giống nhau.

### Nhóm vai trò

Mỗi chức danh được so với 15 nhóm vai trò theo thứ tự ưu tiên cố định, ví dụ Forward Deployed / Solutions, ML / AI Engineer, Data Engineer, DevOps / SRE, Backend Engineer, và cuối cùng là Software Engineer (General). Mỗi tin thuộc tối đa một nhóm nên các con số cộng lại khớp nhau. Một số chức danh như recruiter hay account executive được loại ra có chủ đích. Ngày 20/11 có khoảng 60% tin không thuộc nhóm nào, chủ yếu là sales, pháp lý và phần cứng. Các tin này vẫn được tính trong tổng nhưng không có trong biểu đồ theo vai trò.

### Ghi Gold dưới dạng Iceberg

Mỗi bảng là một bảng Apache Iceberg nằm trong `gold_iceberg/` của lake bucket. Mỗi lượt chạy thay đúng một `snapshot_date` của mỗi bảng trong một lần commit, kể cả khi kết quả rỗng. Vì vậy người đọc chỉ thấy phiên bản hôm qua hoặc hôm nay, không bao giờ thấy một ngày ghi dở.

Sau đó job ghi một bản ghi nhỏ chứa snapshot ID mới của từng bảng. Step Functions kiểm tra bản ghi này thuộc đúng lượt chạy hiện tại rồi chép ba ID vào completion marker. Phía đọc truy vấn bằng `FOR VERSION AS OF <snapshot id>`, nên ba bảng luôn được đọc tại cùng một thời điểm.

Trước khi chuyển hẳn, tôi chạy Iceberg ở chế độ "shadow" song song với các bảng Parquet cũ trong vài ngày và so sánh từng dòng (xem 5.2.8, vấn đề 4). Các bảng Parquet cũ được giữ lại ở dạng chỉ đọc để làm lịch sử.

> 📷 *Ảnh cần bổ sung: kết quả Glue Data Quality của một lượt chạy, và một truy vấn Athena dùng `FOR VERSION AS OF`.*
