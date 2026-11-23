---
title: "Giám sát và kiểm thử"
date: 2026-11-23
weight: 6
chapter: false
pre: " <b> 5.1.6. </b> "
---

# Giám sát và kiểm thử

## Alarm

| Alarm | Báo động khi | Vì sao thiết kế như vậy |
|---|---|---|
| `LateDeliveryAlarm` | số đơn trễ chiếm từ 20% số đơn đã giao trở lên trong 2 trên 3 khung 15 phút liên tiếp, chỉ tính khung có ít nhất 10 đơn đã giao | Phiên bản đầu báo theo "5 đơn trễ mỗi giờ" và bật tắt ba lần trong một buổi sáng khi lưu lượng tăng (xem 5.1.8). Tỷ lệ phải kéo dài thì ít báo nhầm hơn nhưng vẫn bắt được sự cố thật |
| `DlqAlarm` | dead-letter queue có ít nhất một message | Message đã lỗi năm lần cần người xem xét |

Cả hai alarm gửi email qua SNS: một lần khi chuyển sang ALARM và một lần khi trở về OK. Email về giao trễ có link tới trang "Cần xử lý". Đánh đổi là sự cố thật được báo sau khoảng 30–45 phút thay vì ngay lập tức.

Mọi log của Lambda được giữ trong CloudWatch Logs 14 ngày.

> 📷 *Ảnh cần bổ sung: hai alarm trên CloudWatch và một email cảnh báo (che địa chỉ email).*

## Kiểm tra hằng ngày

Một lệnh chỉ đọc kiểm tra toàn bộ hệ thống trong khoảng 40 giây:

```powershell
.\tools\daily-check.ps1 -SaveTo docs\evidence
```

Kết quả lúc 18:16 ngày 23/11/2026:

| # | Hạng mục | Kết quả |
|---|---|---|
| 1 | Stack CloudFormation | `UPDATE_COMPLETE` |
| 2 | Lịch của replayer | `ENABLED` |
| 3 | Replayer trong một giờ qua | 4 lượt, gửi 404 sự kiện, 0 lỗi; đã phát lại 13.580/98.202 đơn |
| 4 | Lambda trong 24 giờ qua | 0 lỗi, 0 lần bị giới hạn |
| 5 | SQS và DLQ | 0 message chờ, DLQ 0 |
| 6 | Event lake hôm nay | 688 file trong `events/dt=2026-11-23/` |
| 7 | CloudWatch alarm | cả hai OK |
| 8 | API | `/health` trả 200, báo cáo không có token trả 401 |
| 9 | Website | HTTP 200, đủ 4/4 header bảo mật |
| 10 | Email cảnh báo | 1 đăng ký đã xác nhận |

Cả 11 dòng đều đạt.

## Kiểm thử tự động

`dotnet test` chạy hơn 200 test (ngày 23/11 ghi nhận 212 test) mà không cần AWS hay Docker:

- **Core:** quy tắc trạng thái, ba adapter và bộ giả lập.
- **Service:** code thật của API, processor và tầng lưu trữ, chạy trên các bản giả nhỏ trong bộ nhớ của DynamoDB, SQS, S3 và Athena. Bản giả báo lỗi với bất kỳ lệnh nào nó không hiểu, nên thay đổi code không thể âm thầm bỏ qua nó.

Để chứng minh test bắt được lỗi thật, tôi cố ý làm hỏng code ở ba chỗ dễ sai và kiểm tra test có báo lỗi không (mutation testing):

| Vùng | Lỗi cố ý | Kết quả |
|---|---|---|
| Tầng lưu trữ | coi mọi điều kiện ghi thất bại là bản trùng | test báo lỗi |
| Processor | bỏ quy tắc FIFO sau khi một message trong batch bị lỗi | 2 test báo lỗi |
| Phân quyền | bỏ kiểm tra vai trò khỏi `/sla/trend` | 2 test báo lỗi |

GitHub Actions chạy build, test và `sam validate --lint` ở mỗi lần push.

## Kiểm thử chaos trên API đã deploy

Ngày 23/11, tôi gửi lưu lượng cố ý lộn xộn tới API thật rồi so trạng thái cuối cùng với một bộ đáp án được tạo riêng:

| Mục | Giá trị |
|---|---|
| Vận đơn / sự kiện khác nhau | 87 / 480 |
| Webhook đã gửi | 2.375: mỗi sự kiện khoảng 5 lần, xáo trộn thứ tự trong từng vận đơn, cộng thêm 24 payload hỏng |
| Nhận (202) / từ chối (400) | 2.351 / 24, đúng như kỳ vọng |
| Bản ghi lịch sử trong DynamoDB | 480, mỗi sự kiện một bản |
| Sai lệch so với đáp án | **0** |
| DLQ trước → sau | 0 → 0 |
| Thời gian từ lúc nhận đến lúc xử lý xong, p50 / p95 / max | 17 / 40 / 192 ms |

Dữ liệu kiểm thử dùng mã vận đơn bắt đầu bằng `DEMO`, nên không ảnh hưởng tới báo cáo SLA.

## Kịch bản demo

`tools/demo.ps1` đi qua năm cảnh, mỗi cảnh là một vấn đề của khách hàng và cách hệ thống giải quyết:

| Cảnh | Vấn đề | Hệ thống chứng minh |
|---|---|---|
| 1 | mỗi hãng một định dạng | 3 webhook khác nhau cho ra cùng một trạng thái |
| 2 | trùng lặp và sai thứ tự | 8 webhook vẫn kết thúc ở `Delivered`; bản trùng bị loại, bản cũ là `Stale`, bản đi lùi là `Rejected` |
| 3 | CSKH phải mở nhiều trang | một lần gọi API trả về toàn bộ dòng thời gian thật |
| 4 | chỉ biết giao trễ khi khách phàn nàn | đơn trễ → alarm → email trong vài phút |
| 5 | chọn hãng theo cảm tính | tỷ lệ đúng hạn theo hãng và khu vực từ Athena |

*(Video demo: đang làm. Kịch bản đã sẵn sàng, chưa quay.)*
