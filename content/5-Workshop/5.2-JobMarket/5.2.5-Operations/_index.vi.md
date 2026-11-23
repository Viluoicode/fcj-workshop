---
title: "Orchestration, Security và Monitoring"
date: 2026-11-23
weight: 5
chapter: false
pre: " <b> 5.2.5. </b> "
---

# Orchestration, Security và Monitoring

## Workflow

EventBridge khởi động state machine mỗi ngày lúc 18:00 UTC, tức 01:00 sáng hôm sau theo giờ Việt Nam. Các state của Step Functions theo thứ tự:

1. **InitializeRun** cố định một run ID và một ngày UTC. Mọi bước sau đều nhận cả hai.
2. **AcquireLock** ghi bản ghi lock vào DynamoDB chỉ khi chưa có lock. Nếu lock đã có, lượt chạy kết thúc ở **PipelineBusy** mà không chạm vào dữ liệu.
3. **Ingest**, **BronzeToSilver** và **SilverToGold**: mỗi state khởi động một Glue job và chờ job chạy xong.
4. **ReadGoldVersions** đọc bản ghi mà job Gold để lại, và **GoldVersionsValid** kiểm tra bản ghi thuộc đúng lượt chạy và có đủ ba bảng.
5. **BuildCompletion** và **PublishCompletion** ghi `state/pipeline/latest.json` kèm ba snapshot ID của Iceberg.
6. **ReleaseLock** xóa lock, nhưng chỉ khi execution này vẫn là chủ của lock.

Nếu có bước nào lỗi, workflow chuyển sang **ReleaseFailedLock** và kết thúc ở trạng thái lỗi. Cả lượt chạy có giới hạn 90 phút, mỗi Glue job có timeout riêng và chỉ cho phép một lượt chạy cùng lúc.

> 📷 *Ảnh cần bổ sung: sơ đồ một execution thành công trong Step Functions.*

### Khi lượt chạy bị dừng đột ngột

Khi bị timeout hoặc bị hủy tay, bước ReleaseLock bị bỏ qua, nên lock nằm lại và chặn lượt chạy đêm hôm sau. Lambda lock janitor xử lý trường hợp này. Nó chạy khi execution kết thúc ở trạng thái FAILED, TIMED_OUT hoặc ABORTED, và chạy lại mỗi 15 phút. Nó chỉ xóa lock khi:

- execution đang giữ lock đã kết thúc, và
- không còn Glue job nào trong ba job đang chạy.

Lệnh xóa có điều kiện theo đúng owner vừa kiểm tra. Janitor không bao giờ khởi động hay dừng bất cứ thứ gì. Quyết định của nó (`released`, `waiting_for_glue`, `owner_running` hoặc `no_lock`) được ghi vào log.

## Bảo mật

- Mỗi bước có IAM role riêng: ingestion, Silver, Gold, Step Functions, EventBridge, health monitor, janitor và dashboard reader. Mỗi role chỉ chạm được vào các prefix S3 mà nó cần.
- Các role khi chạy được đọc script nhưng không sửa được script. Chỉ Step Functions được ghi completion marker.
- Role của dashboard được đọc dữ liệu đã publish, chạy truy vấn Athena và ghi kết quả truy vấn của chính nó. Nó không sửa được lake và không khởi động được job.
- Cả ba bucket đều private, được mã hóa và từ chối HTTP thường. Lake bucket và script bucket giữ phiên bản cũ trong 30 ngày.
- Không lưu access key ở đâu cả. Dịch vụ AWS dùng role của mình, còn tôi dùng credential ngắn hạn.
- `scripts/check_permissions.py` chạy mô phỏng IAM policy trên các role đã deploy (69/69 đạt ở lần kiểm tra gần nhất). Checkov quét code Terraform ở mỗi pull request, và mọi finding được chấp nhận đều có lý do ghi ngay cạnh resource.

## Giám sát

Lambda health monitor chạy mỗi 15 phút và chỉ đọc metadata: completion marker, con trỏ Silver và manifest. Nó gửi hai metric lên CloudWatch:

- `Healthy`: bằng 1 khi run ID khớp nhau, ingestion thành công và dữ liệu chưa quá 26 giờ.
- `SourceSuccessRatio`: tỷ lệ board trả lời thành công.

Sáu alarm gửi email qua SNS:

| Alarm | Cảnh báo khi |
|---|---|
| `pipeline-failed` | một execution bị lỗi |
| `pipeline-timed-out` | một execution chạm giới hạn 90 phút |
| `pipeline-aborted` | một execution bị dừng |
| `pipeline-unhealthy` | `Healthy` nhỏ hơn 1 hoặc bị thiếu trong hai kỳ 15 phút |
| `source-coverage-low` | dưới 100% board thành công trong hai kỳ |
| `lock-janitor-errors` | chính Lambda janitor bị lỗi |

Khi thiếu metric `Healthy`, hệ thống coi là không ổn. Vì vậy nếu bản thân bộ giám sát ngừng chạy thì cũng bị phát hiện.

Log được giữ 30 ngày với Glue và 14 ngày với Lambda. Log nằm trong CloudWatch Logs, mỗi job và mỗi function có log group riêng.

> 📷 *Ảnh cần bổ sung: danh sách alarm trong CloudWatch và email cảnh báo từ SNS.*
