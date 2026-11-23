---
title: "Báo cáo SLA bằng Athena"
date: 2026-11-23
weight: 4
chapter: false
pre: " <b> 5.1.4. </b> "
---

# Báo cáo SLA bằng Athena

## Event lake

Mỗi sự kiện đã xử lý được lưu vào bucket dữ liệu dưới dạng một dòng JSON, trong thư mục theo ngày xử lý:

```text
s3://<data bucket>/events/dt=2026-11-23/....jsonl
```

SAM template tạo một Glue database `shipment_sla_lake` và một bảng `events` trên thư mục này. Bảng dùng **partition projection** trên `dt`: Athena tự suy ra thư mục của từng ngày từ chính câu truy vấn. Dữ liệu mới truy vấn được ngay khi vừa ghi, không cần crawler và không cần `MSCK REPAIR`.

Một workgroup Athena riêng là `shipment-sla-wg` dừng mọi truy vấn khi đã quét quá 1 GB, nên một câu truy vấn viết nhầm không thể thành hóa đơn lớn.

## Truy vấn thủ công

Trên Athena console, chọn workgroup `shipment-sla-wg` và chạy thử, ví dụ:

```sql
SELECT carrier, count(*) AS delivered, count_if(islate) AS late
FROM shipment_sla_lake.events
WHERE dt >= '2026-11-01' AND status = 'Delivered' AND outcome = 'Applied'
GROUP BY carrier;
```

Lọc theo `dt` giúp chỉ quét đúng những ngày cần.

> 📷 *Ảnh cần bổ sung: câu truy vấn này và kết quả trên Athena console, thấy rõ dung lượng đã quét.*

## API báo cáo

Dashboard không tự chạy SQL mà gọi các endpoint sau:

| Endpoint | Ai được gọi | Trả về |
|---|---|---|
| `GET /sla/summary?days=&groupBy=carrier\|region` | quản lý | số đơn đã giao, số đơn trễ và % trễ theo hãng hoặc khu vực |
| `GET /sla/trend?days=&carrier=` | quản lý | theo từng ngày giao: số đơn, % trễ, thời gian giao trung bình |
| `GET /shipments?days=&status=&carrier=&region=&late=&overdue=&limit=` | CSKH + quản lý | danh sách vận đơn đã lọc |
| `GET /shipments/{carrier}/{tracking}` | CSKH + quản lý | một vận đơn kèm toàn bộ dòng thời gian |

Dữ liệu người dùng nhập không bao giờ được đưa vào câu SQL. `groupBy` chỉ nhận một trong hai từ cố định và `days` phải là số nguyên, nhờ đó loại bỏ SQL injection. Tham số sai trả về **400** kèm lý do. Các vận đơn demo (mã bắt đầu bằng `DEMO`) không được tính vào số liệu SLA.

## Cache

Dữ liệu chỉ thay đổi mỗi 15 phút, khi replayer chạy. Chạy lại cùng một truy vấn mỗi lần tải trang sẽ tốn thời gian và tiền, nên có hai lớp cache:

1. Lambda giữ mỗi kết quả trong bộ nhớ 5 phút.
2. Tính năng *query result reuse* của Athena trả lại kết quả cũ trong tối đa 15 phút mà không quét S3.

Tôi đo 38 truy vấn vào ngày 20/11/2026:

| Đường đi | Thời gian |
|---|---|
| Quét toàn bộ S3 | khoảng 2,4 giây |
| Athena dùng lại kết quả | khoảng 0,55 giây |
| Bộ nhớ Lambda | khoảng 0 ms |

79% truy vấn không phải quét S3.

## Số liệu đo được

Ngày 19/11/2026, khi đã phát lại khoảng 5.700 trên 98.202 đơn Olist, dashboard hiển thị 5.425 đơn đã giao, tỷ lệ đúng hạn 96,4% và thời gian giao trung bình 13,3 ngày.

Các con số này phản ánh cách phát lại: đơn được gửi theo ngày đặt hàng, nên các đơn đầu tiên của năm 2016–2017 chiếm phần lớn. Đây không phải tỷ lệ giao trễ của toàn bộ bộ dữ liệu Olist.
