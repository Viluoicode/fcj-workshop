---
title: "Ingestion và tầng Bronze"
date: 2026-11-23
weight: 3
chapter: false
pre: " <b> 5.2.3. </b> "
---

# Ingestion và tầng Bronze

## Các board nguồn

`ingestion/sources.json` liệt kê 36 board trên bốn nền tảng:

| Nền tảng | Số board | Ví dụ |
|---|---:|---|
| Greenhouse | 20 | Stripe, Airbnb, Databricks, GitLab |
| Ashby | 11 | OpenAI, Notion, Ramp, Mistral AI |
| Lever | 4 | Spotify, Palantir |
| Arbeitnow | 1 | một feed tổng hợp tin của nhiều công ty |

Không board nào cần API key. Có một test kiểm tra danh sách hợp lệ: nền tảng phải được hỗ trợ, không board nào bị lặp, và không công ty nào xuất hiện ở hai nền tảng. Quy tắc cuối có vì Mistral AI đã chuyển từ Lever sang Ashby hồi tháng 11 (xem 5.2.8).

## Bước 1 - Lấy dữ liệu các board song song

Job gọi năm API cùng lúc. Mỗi lần gọi có timeout 20 giây và thử tối đa ba lần. Nó gọi lại khi gặp lỗi mạng, HTTP 429 hoặc lỗi 5xx, và chờ lâu hơn sau mỗi lần lỗi.

Kết quả được xử lý theo đúng thứ tự trong danh sách, bất kể lời gọi nào xong trước. Job chỉ lấy trước tối đa mười board so với phần đã ghi. Giới hạn này quan trọng vì một lượt chạy tạo khoảng 150 MB JSON, trong khi Glue Python Shell chỉ có khoảng 1 GB bộ nhớ.

## Bước 2 - Đưa mọi phản hồi về cùng một dạng

Mỗi nền tảng đặt tên trường khác nhau. Mỗi nền tảng có một adapter nhỏ để chuyển về một bản ghi chung: nguồn, board, mã tin của nền tảng, công ty, chức danh, địa điểm, hình thức làm việc, link ứng tuyển, ngày đăng và JSON gốc.

Hình thức làm việc có ba giá trị: remote, onsite hoặc hybrid, và chưa rõ. Mỗi nền tảng cho một gợi ý khác nhau (Lever và Ashby có workplace type, Arbeitnow có cờ remote, Greenhouse chỉ có chuỗi địa điểm). Khi không có gợi ý, giá trị để là chưa rõ thay vì đoán. Ngày 20/11 có 47,5% tin ở trạng thái chưa rõ.

Nếu phản hồi không phải là danh sách tin tuyển dụng, board bị đánh dấu lỗi. Nó không bao giờ bị coi là board chỉ đơn giản không có tin.

## Bước 3 - Ghi Bronze, rồi mới ghi manifest

Bản ghi của mỗi board được ghi vào một file riêng, trong thư mục của lượt chạy:

```text
bronze/run_id=<run>/source=<platform>/board=<token>/jobs.json
```

Chỉ khi đã xử lý xong mọi board, job mới ghi manifest:

```text
control/ingestion/run_id=<run>/manifest.json
```

Manifest xếp mỗi board vào một trong bốn nhóm: thành công có tin, thành công nhưng rỗng, lỗi, hoặc chưa đầy đủ (Arbeitnow chỉ đọc hai trang nên luôn chưa đầy đủ). Lượt chạy chỉ thành công khi ít nhất 80% board thành công và thu được ít nhất một tin.

Manifest đóng vai trò điểm commit. Nếu job dừng giữa chừng, các file đã ghi sẽ không bao giờ được đọc vì không manifest nào liệt kê chúng. Nếu chính S3 từ chối ghi, cả lượt chạy dừng lại: lỗi lưu trữ không được phép bị ghi nhận thành board tuyển dụng bị hỏng.

## Bước 4 - Kiểm tra

Sau một lượt chạy, mở manifest trên S3 và đối chiếu số board với alarm `source-coverage-low` trên CloudWatch. 36/36 board thành công nghĩa là độ phủ đầy đủ. Arbeitnow vẫn được ghi là chưa đầy đủ vì giới hạn số trang, điều này là bình thường.

> 📷 *Ảnh cần bổ sung: một file manifest mở trên S3 console, thấy rõ số board.*
