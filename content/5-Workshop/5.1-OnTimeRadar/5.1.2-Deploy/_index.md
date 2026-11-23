---
title: "Prerequisites and Deployment"
date: 2026-11-23
weight: 2
chapter: false
pre: " <b> 5.1.2. </b> "
---

# Prerequisites and Deployment

## Tools

| Tool | Check |
|---|---|
| .NET SDK 10 | `dotnet --list-sdks` |
| AWS CLI v2, signed in, region `ap-southeast-1` | `aws sts get-caller-identity` |
| AWS SAM CLI (a version that supports the `dotnet10` runtime) | `sam --version` |
| Amazon.Lambda.Tools | `dotnet tool install -g Amazon.Lambda.Tools` |
| Python 3.10+ (prepares the Olist data) | `python --version` |

I chose .NET 10 because Lambda's `dotnet8` runtime reached end of support on 10 November 2026, and `dotnet10` is the current long-term-support version.

## Step 1 - Build and Test Locally

```powershell
git clone https://github.com/Viluoicode/ontime-radar.git
cd ontime-radar
dotnet build
dotnet test
```

The tests need no AWS account and no Docker. The service tests use small in-memory fakes of DynamoDB, SQS, S3, and Athena (see 5.1.6).

## Step 2 - First Deployment From Your Machine

```powershell
Copy-Item samconfig.example.toml samconfig.toml
sam build
sam deploy --guided
```

The guided deploy asks for these parameters:

| Parameter | What to enter |
|---|---|
| `WebhookSecret` | a random string of at least 16 characters; carriers send it in a header |
| `AlertEmail` | the address that receives alarms |
| `ReplayState` | `DISABLED` for now; turn it on after testing |
| `OrdersPerRun` | `20` orders per 15-minute run |
| `LateRatePercent` | `20`: alarm when 20% or more of deliveries are late |
| `MinDeliveries` | `10`: a 15-minute window counts only with at least 10 deliveries |

`samconfig.toml` holds the webhook secret, so it is listed in `.gitignore`.

After the deploy:

1. Open the email from AWS and click **Confirm subscription**, otherwise alarms reach nobody.
2. Note the stack outputs: `ApiUrl`, `DataBucketName`, `TableName`, `UserPoolId`, `WebUrl`, and `LoginUrl`.

> 📷 *Screenshot to add: the CloudFormation stack `shipment-sla` with status `CREATE_COMPLETE` and its Resources tab.*

## Step 3 - Create Staff Accounts

There is no self sign-up. An administrator creates each account, and Cognito emails a temporary password:

```powershell
$pool = "<UserPoolId>"
aws cognito-idp admin-create-user --user-pool-id $pool --username staff@example.com `
  --user-attributes Name=email,Value=staff@example.com Name=email_verified,Value=true
aws cognito-idp admin-add-user-to-group --user-pool-id $pool --username staff@example.com --group-name cskh
```

Use the group `manager` for an account that should also see the SLA reports.

## Step 4 - Load the Olist Data and Start the Replay

Download the *Brazilian E-Commerce Public Dataset by Olist* from Kaggle into `data/olist/`, then:

```powershell
$bucket = "<DataBucketName>"
python tools/prepare_olist.py --input data/olist --output data/prepared
aws s3 sync data/prepared/chunks "s3://$bucket/dataset/chunks/"
sam remote invoke ReplayerFunction --stack-name shipment-sla
```

When the test run reports `Failed = 0`, set `ReplayState` to `ENABLED` in `samconfig.toml` and run `sam build; sam deploy` again. At 20 orders every 15 minutes, the replay sends about 1,900 orders a day and lasts about 50 days.

## Step 5 - Publish the Website

```powershell
.\tools\deploy-web.ps1
```

The script uploads `web/` to the private web bucket and clears the CloudFront cache. Open the `WebUrl` output and sign in.

## Step 6 - Automatic Deployment (GitHub Actions + OIDC)

This step is done once:

1. Deploy `infra/github-oidc.yaml` as a separate stack named `github-oidc-deploy`. It creates the GitHub OIDC provider, a deploy role, and the permissions boundary.
2. In GitHub, create an environment named `production` with **Required reviewers** turned on, limited to the `main` branch.
3. Add the deploy role ARN as the environment secret `AWS_DEPLOY_ROLE_ARN`. This is a role name, not an access key.

From then on, each push to `main` that changes code runs: tests → wait for approval → assume the role through OIDC → `sam deploy` → upload the website → check `/health`. The first fully automatic deployment succeeded on 20 November 2026.

> 📷 *Screenshot to add: a GitHub Actions run waiting for approval, and the same run after it succeeded.*
