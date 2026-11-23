---
title: "Nhận và xử lý sự kiện"
date: 2026-11-23
weight: 3
chapter: false
pre: " <b> 5.1.3. </b> "
---

# Nhận và xử lý sự kiện

## Mỗi hãng một adapter

Ba hãng vận chuyển dùng payload khác nhau. Trong dự án này các định dạng là mô phỏng, không phải đặc tả chính thức của các hãng:

| Hãng | Trường ví dụ | Giá trị ví dụ |
|---|---|---|
| GHN | trạng thái dạng chữ | `delivering` |
| GHTK | trạng thái dạng số | `status_id = 45` |
| J&T | loại lần quét | `scanType = DELIVERY` |

Mỗi adapter là một class, biến payload của hãng thành cùng một kiểu sự kiện: hãng, mã vận đơn, trạng thái chung, thời điểm xảy ra, ngày hẹn giao và khu vực. Thêm một hãng mới chỉ cần thêm một class, phần còn lại của pipeline không đổi.

Adapter cũng kiểm tra đầu vào. Mã vận đơn phải khớp `^[A-Za-z0-9._-]{1,64}$`, vì nó trở thành một phần của khóa DynamoDB, của URL và của đường dẫn S3. Payload sai cấu trúc nhận phản hồi **400**, để hãng không gửi lại mãi.

## Tám trạng thái

```text
CREATED → PICKED_UP → IN_TRANSIT (lặp lại khi qua kho) → OUT_FOR_DELIVERY → DELIVERED
                                                 ↕
                                          FAILED_ATTEMPT → RETURNED
CREATED → CANCELLED
```

## Giữ đúng thứ tự cho từng vận đơn

Lambda API đưa mỗi sự kiện vào hàng đợi **SQS FIFO** với `MessageGroupId` là vận đơn, rồi trả `202` ngay. FIFO giao các sự kiện của một vận đơn theo đúng thứ tự, trong khi các vận đơn khác được xử lý song song.

**eventId** là mã băm từ nội dung sự kiện. Nếu hãng gửi một webhook hai lần, hai bản có cùng ID. SQS loại bản đến trong vòng năm phút, và DynamoDB chặn bản đến sau đó.

## Xác định ý nghĩa của một sự kiện

Processor so sánh mỗi sự kiện với những gì đã biết về vận đơn:

| Tình huống | Kết quả | Tác động |
|---|---|---|
| `eventId` đã gặp trước đó | `Duplicate` | không ghi gì |
| Xảy ra trước sự kiện mới nhất đã áp dụng | `Stale` | lưu vào lịch sử, không đổi trạng thái |
| Vận đơn đã kết thúc (đã giao, đã hoàn, đã hủy) | `Rejected` | lưu vào lịch sử, không đổi trạng thái |
| Đi lùi, ví dụ đang vận chuyển sau khi đã đi giao | `Rejected` | lưu vào lịch sử, không đổi trạng thái |
| Các trường hợp còn lại | `Applied` | cập nhật trạng thái |

Một vận đơn bị coi là **giao trễ** khi ngày giao thực tế sau ngày hẹn giao.

## Ghi kết quả

Bản ghi trạng thái và bản ghi lịch sử được ghi vào **DynamoDB** trong một transaction có kiểm tra version, nên hai bản ghi không bao giờ lệch nhau và hai luồng ghi không đè lên nhau. Bảng dùng một partition cho mỗi vận đơn, nên một truy vấn trả về cả trạng thái lẫn toàn bộ lịch sử.

Sau đó processor:

- ghi thêm sự kiện vào `s3://<data bucket>/events/dt=YYYY-MM-DD/` dạng JSON lines để Athena dùng;
- đẩy metric `Deliveries` và `LateDeliveries` lên CloudWatch, tách theo hãng.

Trigger SQS dùng **partial batch failure**. Nếu một message lỗi, message đó và mọi message sau nó trong cùng nhóm quay lại hàng đợi, và không có gì phía sau bị xử lý sai thứ tự. Sau năm lần lỗi, message chuyển sang dead-letter queue và kích hoạt alarm.

## Replayer

Mỗi 15 phút EventBridge Scheduler chạy Lambda Replayer. Nó đọc 20 đơn Olist tiếp theo từ S3, biến mỗi đơn thành một chuỗi webhook của hãng và gửi tới API công khai, giống hệt một hãng thật.

Một con trỏ trong DynamoDB ghi nhớ vị trí phát lại. Con trỏ chỉ tiến lên khi không có webhook nào lỗi vì lý do tạm thời (bị giới hạn tốc độ, lỗi 5xx hoặc lỗi mạng), nên lượt sau sẽ gửi lại đúng các đơn đó. Những bản đã được xử lý sẽ trả về `Duplicate`.

## Chạy thử

```powershell
$api = "<ApiUrl>"
$h   = @{ "X-Webhook-Secret" = "<WebhookSecret>" }
Invoke-RestMethod -Method Post "$api/carriers/ghn/webhook" -Headers $h -ContentType "application/json" -InFile tools/sample-webhook-ghn.json
Invoke-RestMethod -Method Post "$api/carriers/ghn/webhook" -Headers $h -ContentType "application/json" -InFile tools/sample-webhook-ghn-delivered.json
Start-Sleep 5
Invoke-RestMethod "$api/shipments/ghn/GHNTEST001" -Headers $auth | ConvertTo-Json -Depth 5
```

Vận đơn kết thúc ở `Delivered` với `isLate = true` và hai sự kiện `Applied`. Gửi lại webhook đầu tiên không thêm gì, còn secret sai trả về 401. `$auth` chứa token Cognito (xem 5.1.5).

> 📷 *Ảnh cần bổ sung: hàng đợi SQS FIFO và DLQ trên console, và một bản ghi vận đơn trong DynamoDB.*
