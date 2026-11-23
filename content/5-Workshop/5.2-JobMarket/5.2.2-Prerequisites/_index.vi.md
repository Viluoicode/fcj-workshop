---
title: "Prerequisite và môi trường"
date: 2026-11-12
weight: 2
chapter: false
pre: " <b> 5.2.2. </b> "
---

# Prerequisite và môi trường

## Công cụ cần thiết

- AWS account có quyền tạo tài nguyên của dự án.
- AWS CLI được cấu hình bằng credential ngắn hạn hoặc credential được cho phép.
- Terraform 1.10 trở lên (cần cho cơ chế khóa state trên S3).
- Python 3.11 cho local test suite.
- Java 17 cho local PySpark tests.
- Git và terminal.
- Browser để sử dụng AWS Console và Streamlit dashboard.

Region triển khai là `ap-southeast-1`. Không commit credential, Terraform state, file `.tfvars` thật, email hoặc source data đã tải.

## Chuẩn bị repository

```powershell
git clone https://github.com/Viluoicode/Job-Market-Pipeline-cloud.git
cd Job-Market-Pipeline-cloud
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
```

## Validation trước khi deploy

```powershell
terraform -chdir=infra init -backend=false
terraform -chdir=infra fmt -check -recursive
terraform -chdir=infra validate
python -m pytest tests -q
```

Local tests mock public API và AWS client. Việc pass local test là cần thiết nhưng không thay thế evidence từ AWS deployment.

## Cấu hình Terraform

Sao chép `infra/terraform.tfvars.example` thành file cấu hình riêng không được commit và kiểm tra:

- Region và project name.
- Bật schedule và cron expression.
- Alert email và monitoring subscription.
- Glue worker type, worker count và timeout.
- Ingestion và monitoring success threshold.
- Freshness và lifecycle threshold.
- Dashboard reader principal ARN.
- Hành vi cleanup bucket.

## Triển khai bằng Terraform

### Bước 1 - Tạo bucket chứa state (làm một lần)

Terraform lưu state trong một bucket S3 do tôi tạo tay, nằm ngoài stack. Nhờ vậy lệnh `terraform destroy` không bao giờ xóa mất chính state mô tả stack đó.

```powershell
$account = aws sts get-caller-identity --query Account --output text
$stateBucket = "jobmarket-aws-tfstate-$account-ap-southeast-1"
aws s3api create-bucket --bucket $stateBucket --region ap-southeast-1 `
  --create-bucket-configuration LocationConstraint=ap-southeast-1
aws s3api put-bucket-versioning --bucket $stateBucket --versioning-configuration Status=Enabled
aws s3api put-public-access-block --bucket $stateBucket --public-access-block-configuration `
  BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
"bucket = `"$stateBucket`"" | Set-Content -Encoding utf8 infra/backend.hcl
```

`backend.hcl` chứa account ID nên không đưa lên Git.

### Bước 2 - Plan và apply

```powershell
terraform -chdir=infra init -backend-config=backend.hcl
terraform -chdir=infra plan -out=workshop.tfplan
terraform -chdir=infra apply workshop.tfplan
```

Đọc kỹ plan trước khi apply: nó liệt kê mọi bucket, Glue job, IAM role và alarm sắp được tạo và bắt đầu tính tiền. Ở lần triển khai đầu, để `enable_schedule = false` để chưa có gì tự chạy cho đến khi bạn kiểm tra thủ công.

> 📷 *Ảnh cần bổ sung: dòng `Apply complete!` kèm số resource.*

### Bước 3 - Chạy pipeline thủ công một lần

```powershell
$pipelineArn = terraform -chdir=infra output -raw state_machine_arn
aws stepfunctions start-execution --state-machine-arn $pipelineArn --region ap-southeast-1
```

Một lượt chạy đầy đủ mất khoảng mười phút.

### Bước 4 - Kiểm tra kết quả

Trên AWS Console (region Singapore):

1. **Step Functions**: execution kết thúc ở trạng thái `SUCCEEDED`, mọi bước đều xanh.
2. **S3 → lake bucket**: có file `state/pipeline/latest.json` ghi đúng run vừa chạy.
3. **S3**: mở `control/ingestion/run_id=<run>/manifest.json` và xem số board thành công, lỗi.
4. **DynamoDB → bảng lock**: bảng trống, nghĩa là lock đã được nhả.
5. **Athena**: truy vấn đếm trên `fact_job_posting` trả về dữ liệu cho snapshot hôm nay.

Có thể chạy cùng phép kiểm tra độ mới từ terminal:

```powershell
$lake = terraform -chdir=infra output -raw lake_bucket
python scripts/check_freshness.py --bucket $lake --region ap-southeast-1
```

> 📷 *Ảnh cần bổ sung: sơ đồ execution trong Step Functions, file manifest trên S3 và kết quả truy vấn Athena.*

### Bước 5 - Bật lịch chạy hằng ngày

Khi lượt chạy thủ công đạt mọi bước kiểm tra, đặt `enable_schedule = true` trong `terraform.tfvars`, chạy lại plan và apply, rồi vào **EventBridge → Rules** xác nhận rule của pipeline ở trạng thái `ENABLED`. Từ đó pipeline tự chạy mỗi ngày lúc 18:00 UTC (01:00 giờ Việt Nam).

## Chuẩn bị bảo mật

Chỉ sử dụng deployment identity cho thay đổi hạ tầng có chủ đích. Dashboard phải assume dedicated restricted reader role; không sử dụng administrator credential làm fallback ngầm định cho dashboard.

