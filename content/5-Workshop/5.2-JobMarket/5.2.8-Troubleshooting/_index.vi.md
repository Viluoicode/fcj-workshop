---
title: "Xử lý sự cố và hướng phát triển"
date: 2026-11-23
weight: 8
chapter: false
pre: " <b> 5.2.8. </b> "
---

# Xử lý sự cố và hướng phát triển

Trang này ghi lại các vấn đề tôi thực sự gặp khi xây dựng và vận hành pipeline: nguyên nhân là gì và tôi sửa thế nào. Mỗi mục đều ghi commit chứa bản sửa, để có thể đối chiếu với repository.

## Các vấn đề đã gặp

### 1. Một board tuyển dụng ngừng trả dữ liệu

- **Hiện tượng:** từ ngày 6/11, alarm `source-coverage-low` ở trạng thái ALARM liên tục và tin của Mistral AI không còn được cập nhật.
- **Nguyên nhân:** công ty chuyển trang tuyển dụng từ Lever sang Ashby. Board Lever cũ trả về HTTP 404, nên job ingestion ngày nào cũng tính đây là board lỗi.
- **Cách sửa:** thay mục Lever bằng board Ashby `mistral.ai` trong `ingestion/sources.json`, danh mục vẫn giữ 36 board. Tôi thêm một test chặn việc một công ty xuất hiện ở hai nền tảng, vì như vậy tin của họ sẽ bị đếm hai lần.
- **Bài học:** alarm chỉ có ích khi có người đọc. Alarm này báo suốt hai tuần thì tôi mới xử lý.
- **Commit:** `be0e63f` (21/11/2026).

> 📷 *Bổ sung: lịch sử alarm CloudWatch, chuyển từ ALARM sang OK sau khi sửa.*

### 2. Lượt chạy hôm sau không khởi động được (`PipelineBusy`)

- **Hiện tượng:** lượt chạy 01:00 dừng ngay với lỗi `PipelineBusy`.
- **Nguyên nhân:** một execution trước đó bị timeout. Khi timeout, bước `ReleaseLock` không được chạy, lock trong DynamoDB nằm lại và mọi lượt sau đều thấy lock đó rồi dừng.
- **Cách sửa:** thêm một Lambda nhỏ ("lock janitor"). Nó chạy khi execution kết thúc ở trạng thái FAILED, TIMED_OUT hoặc ABORTED, và chạy lại mỗi 15 phút. Nó chỉ xóa lock khi execution sở hữu lock đã kết thúc và không còn Glue job hay crawler nào đang chạy, và lệnh xóa có điều kiện theo đúng owner đã kiểm tra.
- **Kiểm chứng:** `terraform apply` thêm 11 resource, gọi Lambda thật trả về `no_lock`.
- **Commit:** `9dacb34` (21/11/2026).

### 3. Chạy lại một ngày có thể để sót dòng cũ trong Gold

- **Hiện tượng:** một test cho thấy khi chạy lại một ngày mà mart ra rỗng, các dòng cũ vẫn còn trong bảng.
- **Nguyên nhân:** chế độ dynamic partition overwrite của Spark chỉ thay các partition có dòng mới. Kết quả rỗng không chạm partition nào, nên không có gì bị thay.
- **Cách sửa:** job Gold ghi đè rõ ràng đúng `snapshot_date` được yêu cầu, trong một commit Iceberg duy nhất.
- **Commit:** `d733030` (22/11/2026).

### 4. Bảng Iceberg và Parquet "không khớp"

- **Hiện tượng:** sau lần ghi shadow Iceberg đầu tiên, hai bản có cùng số dòng (8.682 / 15 / 15) nhưng phép so `EXCEPT` báo toàn bộ dòng đều khác.
- **Nguyên nhân:** Athena đọc timestamp của Parquet ở mức mili giây, còn Iceberg giữ micro giây của Spark. Hai giá trị là cùng một thời điểm, chỉ khác độ chính xác.
- **Cách sửa:** script so sánh làm tròn timestamp về mili giây trước khi so. Ngày đầu tiên cho kết quả 0 dòng lệch theo cả hai chiều.
- **Commit:** `7ab87d9` (23/11/2026).

### 5. Lấy dữ liệu song song có thể làm hết bộ nhớ

- **Hiện tượng (phát hiện khi thiết kế, trước khi lỗi xảy ra thật):** một lượt chạy tạo khoảng 150 MB JSON Bronze, trong khi Glue Python Shell chỉ có khoảng 1 GB RAM.
- **Nguyên nhân:** nếu lấy cả 36 board song song rồi mới ghi một lần ở cuối, toàn bộ phản hồi sẽ nằm trong bộ nhớ cùng lúc.
- **Cách sửa:** 5 thread lấy dữ liệu các board. Kết quả được ghi lên S3 lần lượt theo đúng thứ tự board, và bộ nhớ giữ tối đa 10 board. Lỗi ghi S3 vẫn dừng cả lượt chạy.
- **Commit:** `066c331` (22/11/2026).

### 6. Nâng cấp lint suýt làm hỏng Glue job

- **Hiện tượng:** sau khi thêm `ruff`, các quy tắc nâng cấp cú pháp muốn sửa code sang dùng `datetime.UTC`.
- **Nguyên nhân:** `datetime.UTC` chỉ có từ Python 3.11, còn Glue Python Shell chạy Python 3.9.
- **Cách sửa:** đặt `target-version = "py39"` trong `pyproject.toml` để linter chỉ gợi ý cú pháp mà runtime hỗ trợ.
- **Commit:** `37d537f` (21/11/2026).

### 7. Các lỗi nhỏ khác

| Vấn đề | Cách sửa | Commit |
|---|---|---|
| Unit test lỗi vì Spark tự đoán schema Bronze khác với môi trường thật | Khai báo schema rõ ràng trong test fixture | `d1136a4` (28/07) |
| Dashboard lỗi sau khi Streamlit bỏ tham số `use_container_width` | Đổi sang `width="stretch"` | `a9dd23e` (29/07) |
| Tiếng Việt trong tài liệu bị lỗi ký tự | Ép UTF-8 và thêm test kiểm tra encoding | `e829916` (20/11) |
| CI lỗi vì thiếu `pandas` | Thêm vào `requirements-dev.txt` | `44cf03d` (20/11) |

### 8. Các vấn đề khác (cần bổ sung)

*(Thêm các vấn đề không có trong commit, ví dụ lỗi trên Console, lỗi IAM `AccessDenied` hoặc `terraform apply` thất bại, theo cùng bố cục: hiện tượng, nguyên nhân, cách sửa và cách kiểm chứng.)*

## Hướng phát triển

*(Danh sách nháp. Giữ lại những mục bạn thực sự dự định làm, bỏ các mục còn lại.)*

- **Chuyển phần đọc sang Iceberg.** Athena và dashboard vẫn đang đọc bản Parquet. Khi phép so shadow ổn định trong vài ngày, có thể chuyển sang đọc Iceberg, và khi đó có thể publish ba mart cùng một lúc.
- **Bỏ giới hạn trang của Arbeitnow.** Feed này đang bị giới hạn hai trang nên một số tin bị bỏ sót.
- **Dashboard dùng chung.** Ứng dụng Streamlit hiện chỉ chạy trên máy tôi. Một bản host sẵn, chỉ đọc, sẽ cho người khác dùng mà không cần credential AWS.
- **Triển khai từ CI.** CI đã chạy test, `terraform validate` và Checkov. Bước tiếp theo là chạy `plan` trên mỗi pull request để review, dùng credential ngắn hạn qua OIDC.
- **Phân loại vai trò tốt hơn.** Title đang được gán vào vai trò bằng quy tắc. Một mẫu dữ liệu có gán nhãn sẽ cho biết các quy tắc đó sai bao nhiêu.
