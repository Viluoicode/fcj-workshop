---
title: "Prerequisites and Environment"
date: 2026-11-12
weight: 2
chapter: false
pre: " <b> 5.2.2. </b> "
---

# Prerequisites and Environment

## Required Tools

- An AWS account with permission to create the project resources.
- AWS CLI configured with short-lived or otherwise approved credentials.
- Terraform 1.10 or newer (the S3 state lock needs it).
- Python 3.11 for the local test suite.
- Java 17 for local PySpark tests.
- Git and a terminal.
- A browser for the AWS Console and Streamlit dashboard.

The deployment region is `ap-southeast-1`. Credentials, Terraform state, real `.tfvars`, email addresses, and downloaded source data must not be committed.

## Repository Preparation

```powershell
git clone https://github.com/Viluoicode/Job-Market-Pipeline-cloud.git
cd Job-Market-Pipeline-cloud
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
```

## Validate Before Deployment

```powershell
terraform -chdir=infra init -backend=false
terraform -chdir=infra fmt -check -recursive
terraform -chdir=infra validate
python -m pytest tests -q
```

Local tests mock public APIs and AWS clients. Passing locally is necessary but does not replace evidence from an AWS deployment.

## Terraform Configuration

Copy `infra/terraform.tfvars.example` to a private, uncommitted configuration file and review:

- Region and project name.
- Schedule enablement and cron expression.
- Alert email and monitoring subscription.
- Glue worker type, worker count, and timeout.
- Ingestion and monitoring success thresholds.
- Freshness and lifecycle thresholds.
- Dashboard reader principal ARNs.
- Bucket cleanup behavior.

## Deploy with Terraform

### Step 1 - Create the state bucket (once)

Terraform keeps its state in an S3 bucket that I create by hand, outside the stack. Keeping it outside means `terraform destroy` can never delete the state that describes the stack.

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

`backend.hcl` contains the account ID, so it stays out of Git.

### Step 2 - Plan and apply

```powershell
terraform -chdir=infra init -backend-config=backend.hcl
terraform -chdir=infra plan -out=workshop.tfplan
terraform -chdir=infra apply workshop.tfplan
```

Read the plan before you apply it: it lists every bucket, Glue job, IAM role, and alarm that will start costing money. Keep `enable_schedule = false` for the first deployment so nothing runs until you have checked it by hand.

> 📷 *Screenshot to add: the `Apply complete!` summary with the resource count.*

### Step 3 - Run the pipeline once by hand

```powershell
$pipelineArn = terraform -chdir=infra output -raw state_machine_arn
aws stepfunctions start-execution --state-machine-arn $pipelineArn --region ap-southeast-1
```

A full run takes roughly ten minutes.

### Step 4 - Check the result

In the AWS Console (region Singapore):

1. **Step Functions**: the execution ends in `SUCCEEDED` and every stage is green.
2. **S3 → lake bucket**: `state/pipeline/latest.json` exists and names the run you just started.
3. **S3**: open `control/ingestion/run_id=<run>/manifest.json` and look at the board counts.
4. **DynamoDB → lock table**: the table is empty, so the lock was released.
5. **Athena**: a count query on `fact_job_posting` returns rows for today's snapshot.

You can also run the same freshness check from the terminal:

```powershell
$lake = terraform -chdir=infra output -raw lake_bucket
python scripts/check_freshness.py --bucket $lake --region ap-southeast-1
```

> 📷 *Screenshots to add: the Step Functions graph, the manifest in S3, and the Athena query result.*

### Step 5 - Turn on the daily schedule

When the manual run passes every check, set `enable_schedule = true` in `terraform.tfvars`, run plan and apply again, and confirm in **EventBridge → Rules** that the pipeline rule shows `ENABLED`. From then on the pipeline runs every day at 18:00 UTC (01:00 in Vietnam).

## Security Preparation

Use an authenticated deployment identity only for intentional infrastructure changes. The dashboard must assume its dedicated restricted reader role; an administrator credential must not be used as an implicit dashboard fallback.

