---
title: "Xử lý sự cố và hướng phát triển"
date: 2026-11-23
weight: 8
chapter: false
pre: " <b> 5.1.8. </b> "
---

# Xử lý sự cố và hướng phát triển

Đây là các vấn đề đã gặp khi xây dựng và vận hành OnTime Radar. Mỗi mục ghi commit đã sửa lỗi trong repository `ontime-radar`.

## Các vấn đề đã gặp

### 1. API bắt đầu trả 503 khi có lưu lượng thật

- **Hiện tượng:** khi bộ phát lại Olist chạy, API Gateway trả 503 cho một số webhook.
- **Nguyên nhân:** tài khoản AWS mới chỉ chạy được 10 Lambda cùng lúc. Processor FIFO tăng lên mỗi nhóm vận đơn một instance và chiếm hết cả 10, không còn chỗ cho API.
- **Cách sửa:** giới hạn trigger SQS của processor ở 2 instance chạy đồng thời. Bước tiếp theo là xin AWS nâng quota.
- **Commit:** `e835dd5` (16/11).

### 2. Suýt commit một file chứa secret

- **Hiện tượng:** `samconfig.toml`, file chứa webhook secret, vẫn hiện là file chưa được theo dõi dù đã có trong `.gitignore`.
- **Nguyên nhân:** dòng đó có chú thích nằm sau mẫu. Trong `.gitignore`, phần chữ nằm sau mẫu trên cùng một dòng trở thành một phần của mẫu, nên mẫu không bao giờ khớp.
- **Cách sửa:** chuyển chú thích sang một dòng riêng.
- **Commit:** `d77358c` (16/11).

### 3. Alarm giao trễ bật tắt liên tục

- **Hiện tượng:** alarm đầu tiên báo động khi có "từ 5 đơn trễ trở lên mỗi giờ". Với khoảng 80 đơn giao mỗi giờ, nó chuyển sang ALARM ba lần trong một buổi sáng.
- **Nguyên nhân:** ngưỡng là con số cố định nên tăng theo lưu lượng: giờ đông đơn trông giống như có sự cố.
- **Cách sửa:** alarm giờ dùng *tỷ lệ* trễ: từ 20% trở lên trong 2 trên 3 khung, chỉ tính khung có ít nhất 10 đơn đã giao. Alarm cũng gửi email báo "đã hết sự cố".
- **Commit:** `b6c2dfa` (19/11).

### 4. GitHub không nhận được role deploy

- **Hiện tượng:** lần deploy qua OIDC đầu tiên bị từ chối ở bước `AssumeRoleWithWebIdentity`.
- **Nguyên nhân:** token của GitHub giờ ghi ID của chủ sở hữu và repository trong claim `sub` (`repo:owner@<id>/repo@<id>:...`), trong khi trust policy chỉ so theo tên.
- **Cách sửa:** trust policy so theo các ID cố định. Cách này còn an toàn hơn: một repository mới tạo trùng tên cũng không dùng được role.
- **Commit:** `01b4e9c` (20/11).

### 5. Replayer có thể bỏ sót đơn bị lỗi

- **Hiện tượng:** khi vài webhook trong một lượt bị lỗi, con trỏ vẫn tiến lên, nên các đơn đó không bao giờ được gửi lại.
- **Cách sửa:** phản hồi giờ được chia thành Sent, Rejected (dữ liệu sai, gửi lại cũng không giải quyết được) và Failed (bị giới hạn tốc độ, lỗi 5xx, lỗi mạng). Con trỏ chỉ tiến khi Failed bằng 0. Gửi lại vẫn an toàn vì bản trùng bị loại.
- **Commit:** `81576a1` (20/11).

### 6. Payload hỏng trả về 500 thay vì 400

- **Hiện tượng:** bộ sinh chaos phát hiện rằng body không phải object JSON, hoặc số bị gửi dưới dạng chuỗi, làm adapter ném lỗi. API trả 500, và một hãng thật sẽ gửi lại một request lỗi 500 mãi mãi.
- **Cách sửa:** mọi adapter giờ kiểm tra cấu trúc JSON và kiểu giá trị trước, rồi trả 400. Tôi thêm 7 test hồi quy.
- **Commit:** `ac243b0` (23/11).

### 7. Các vấn đề khác (cần bổ sung)

*(Thêm các vấn đề không có trong lịch sử commit, ví dụ lỗi trên console hoặc thông báo `AccessDenied` khi deploy lần đầu, theo cùng bố cục: hiện tượng, nguyên nhân, cách sửa.)*

## Giới hạn đã biết

| Giới hạn | Ảnh hưởng hiện tại | Hướng sửa |
|---|---|---|
| **R1:** hai sự kiện có thời điểm giống hệt nhau; sự kiện đến sau thắng | 0 vận đơn bị ảnh hưởng: dữ liệu phát lại không bao giờ có thời điểm trùng | Phân xử theo thứ tự cố định (hạng trạng thái, rồi mã gốc) |
| **R2:** ngày hẹn giao chỉ nằm trong một sự kiện đến muộn sẽ bị mất, nên vận đơn không được tính vào tỷ lệ đúng hạn | 0 vận đơn bị ảnh hưởng: replayer gửi ngày hẹn trong mọi webhook | Cho sự kiện `Stale` điền các trường còn trống mà không đổi trạng thái |
| DynamoDB và S3 không được ghi trong cùng một transaction | Nếu Lambda dừng giữa hai lần ghi, sự kiện sẽ thiếu trong Athena. Tra cứu và alarm không bị ảnh hưởng | Ghi lake từ DynamoDB Streams |
| Việc tính trễ so ngày theo UTC với ngày hẹn | 280/96.470 đơn (0,29 điểm phần trăm) bị tính trễ dù giao vào buổi tối đúng ngày hẹn theo giờ Brazil | So theo múi giờ của khu vực giao |
| Hãng, mã vận đơn và các bước đang vận chuyển là giả lập | Tỷ lệ giao thất bại không phản ánh hãng thật | Kết nối API thật của hãng |
| Dữ liệu đến từ Brazil | Khu vực là các bang của Brazil, không phải tỉnh của Việt Nam | Dùng dữ liệu đơn hàng Việt Nam khi có |

Mỗi giới hạn R1/R2 có một test tên `Known_gap_...` ghi lại hành vi hiện tại, nên khi sửa quy tắc thì test đó buộc phải đổi theo.

## Hướng phát triển

- Xin AWS nâng quota Lambda chạy đồng thời (trong lần thử dồn tải, 80 request cùng lúc vẫn gây 47 lỗi 503).
- Cấp secret riêng cho từng hãng, hoặc ký webhook bằng HMAC, lưu trong Secrets Manager.
- Thu hẹp quyền S3 của processor và replayer.
- Thêm lifecycle rule cho kết quả Athena, và dừng truy vấn khi bị timeout.
- Ghi event lake từ DynamoDB Streams để đóng kẽ hở nêu ở trên.
- Hoàn thành báo cáo chi phí và quay video demo.
