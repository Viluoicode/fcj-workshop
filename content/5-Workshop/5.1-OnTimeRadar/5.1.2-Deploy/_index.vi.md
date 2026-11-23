---
title: "Chuẩn bị và triển khai"
date: 2026-11-23
weight: 2
chapter: false
pre: " <b> 5.1.2. </b> "
---

# Chuẩn bị và triển khai

## Công cụ

| Công cụ | Cách kiểm tra |
|---|---|
| .NET SDK 10 | `dotnet --list-sdks` |
| AWS CLI v2, đã đăng nhập, region `ap-southeast-1` | `aws sts get-caller-identity` |
| AWS SAM CLI (phiên bản hỗ trợ runtime `dotnet10`) | `sam --version` |
| Amazon.Lambda.Tools | `dotnet tool install -g Amazon.Lambda.Tools` |
| Python 3.10+ (để chuẩn bị dữ liệu Olist) | `python --version` |

Tôi chọn .NET 10 vì runtime `dotnet8` của Lambda đã hết hỗ trợ từ ngày 10/11/2026, còn `dotnet10` là phiên bản hỗ trợ dài hạn hiện tại.

## Bước 1 - Build và test trên máy

```powershell
git clone https://github.com/Viluoicode/ontime-radar.git
cd ontime-radar
dotnet build
dotnet test
```

Test không cần tài khoản AWS và không cần Docker. Nhóm test service dùng các bản giả nhỏ trong bộ nhớ thay cho DynamoDB, SQS, S3 và Athena (xem 5.1.6).

## Bước 2 - Deploy lần đầu từ máy cá nhân

```powershell
Copy-Item samconfig.example.toml samconfig.toml
sam build
sam deploy --guided
```

Bước deploy có hướng dẫn sẽ hỏi các tham số sau:

| Tham số | Giá trị cần nhập |
|---|---|
| `WebhookSecret` | chuỗi ngẫu nhiên ít nhất 16 ký tự; hãng vận chuyển gửi chuỗi này trong header |
| `AlertEmail` | địa chỉ nhận alarm |
| `ReplayState` | tạm để `DISABLED`; bật sau khi đã kiểm thử |
| `OrdersPerRun` | `20` đơn cho mỗi lượt 15 phút |
| `LateRatePercent` | `20`: báo động khi từ 20% đơn giao trở lên bị trễ |
| `MinDeliveries` | `10`: một khung 15 phút chỉ được tính khi có ít nhất 10 đơn đã giao |

`samconfig.toml` chứa webhook secret nên đã được đưa vào `.gitignore`.

Sau khi deploy:

1. Mở email từ AWS và bấm **Confirm subscription**, nếu không alarm sẽ không đến được ai.
2. Ghi lại các output của stack: `ApiUrl`, `DataBucketName`, `TableName`, `UserPoolId`, `WebUrl` và `LoginUrl`.

> 📷 *Ảnh cần bổ sung: stack CloudFormation `shipment-sla` ở trạng thái `CREATE_COMPLETE` và tab Resources.*

## Bước 3 - Tạo tài khoản nhân viên

Hệ thống không cho tự đăng ký. Quản trị viên tạo từng tài khoản, và Cognito gửi mật khẩu tạm qua email:

```powershell
$pool = "<UserPoolId>"
aws cognito-idp admin-create-user --user-pool-id $pool --username staff@example.com `
  --user-attributes Name=email,Value=staff@example.com Name=email_verified,Value=true
aws cognito-idp admin-add-user-to-group --user-pool-id $pool --username staff@example.com --group-name cskh
```

Dùng nhóm `manager` cho tài khoản cần xem thêm báo cáo SLA.

## Bước 4 - Nạp dữ liệu Olist và bắt đầu phát lại

Tải *Brazilian E-Commerce Public Dataset by Olist* trên Kaggle về thư mục `data/olist/`, sau đó:

```powershell
$bucket = "<DataBucketName>"
python tools/prepare_olist.py --input data/olist --output data/prepared
aws s3 sync data/prepared/chunks "s3://$bucket/dataset/chunks/"
sam remote invoke ReplayerFunction --stack-name shipment-sla
```

Khi lượt chạy thử báo `Failed = 0`, đổi `ReplayState` thành `ENABLED` trong `samconfig.toml` và chạy lại `sam build; sam deploy`. Với 20 đơn mỗi 15 phút, mỗi ngày hệ thống phát lại khoảng 1.900 đơn và chạy được khoảng 50 ngày.

## Bước 5 - Đưa website lên

```powershell
.\tools\deploy-web.ps1
```

Script tải thư mục `web/` lên bucket web private và xóa cache của CloudFront. Mở output `WebUrl` rồi đăng nhập.

## Bước 6 - Deploy tự động (GitHub Actions + OIDC)

Bước này chỉ làm một lần:

1. Deploy `infra/github-oidc.yaml` thành một stack riêng tên `github-oidc-deploy`. Stack này tạo OIDC provider của GitHub, một role dùng để deploy và permissions boundary.
2. Trên GitHub, tạo environment tên `production`, bật **Required reviewers** và chỉ cho phép nhánh `main`.
3. Thêm ARN của role deploy làm secret `AWS_DEPLOY_ROLE_ARN` của environment. Đây là tên role, không phải access key.

Từ đó, mỗi lần đẩy thay đổi code lên `main` sẽ chạy: test → chờ duyệt → nhận role qua OIDC → `sam deploy` → tải website → kiểm tra `/health`. Lần deploy tự động hoàn toàn đầu tiên thành công vào ngày 20/11/2026.

> 📷 *Ảnh cần bổ sung: một lượt chạy GitHub Actions đang chờ duyệt và chính lượt đó sau khi thành công.*
