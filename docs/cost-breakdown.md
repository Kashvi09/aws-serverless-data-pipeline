## Cost Breakdown
| Service | Purpose | Free Tier Eligible? | Possible Cost | Delete After Project? |
|---|---|---|---|---|
| IAM User/Group | Non-root access | Yes — always free | $0 | No — keep permanently |
| CloudWatch Alarm | Billing tripwire | Yes — first 10 alarms free | $0 at this usage | No — keep permanently |
| SNS Topic | Email delivery | Yes — 1,000 emails/month free | $0 at this scale | No — keep permanently |
| S3 Bucket (kashvi-pipeline-raw-data) | Raw data ingestion | Yes (free tier) | Near-zero | No |
| IAM Role (lambda-s3-read-role) | Lambda execution permissions | Always free | $0 | No |
| Lambda (process-incoming-data) | Processes files on S3 upload | Yes - 1M requests + 400,000 GB-seconds/month, permanently | Effectively $0 at test scale | No - keep permanently |
| S3 Event Notification | Triggers lambda on upload | Free(bucket config, not a billed resource) | $0 | No |
| DynamoDB Table (ProcessedFilesMetadata) | Stores processed file metadata | Yes - 25GB storage + 25 write/read capacity units always free(this is part of AWS's always-free tier, not just first 12 months); on-demand mode also has its own always-free monthly request allowance | Effectively $0 at test scale | No - keep permanently |
| Lambda (query-processed-files) | Reads DynamoDB, returns JSON via API | Yes — always-free tier | Effectively $0 at test scale | No — keep permanently |
| IAM Role (lambda-dynamodb-read-role) | Query Lambda execution permissions | Always free | $0 | No |
| API Gateway (HTTP API) | Public query endpoint | Yes — **first 12 months only** | $0 now; ~$1/million requests after 12 months | No — but revisit cost assumption after year 1 |
| CloudFormation | Orchestrates resource creation/deletion | Yes — always free (you pay only for the resources it creates, not the service itself) | $0 | No |
| GitHub Actions | Runs the deploy workflow | Yes — unlimited free for public repos, 2,000 min/month free for private | $0 at this usage | No — keep permanently |
| IAM User (github-actions-deployer) | Programmatic-only user for CI/CD deploys | Always free | $0 | No — keep, only while project is active |
| CloudWatch Alarm (ingestion errors) | Detects Lambda failures | Yes — within free 10 alarms/month | $0 | No — keep permanently |
| CloudWatch Dashboard (pipeline-health) | Visual pipeline metrics | Yes — first 3 dashboards free | $0 | No — keep permanently |
| SNS (pipeline-alerts topic) | Delivers error alarm email | Yes — within free tier | $0 | No | 

**Note:** API Gateway is the only resource in this project not part of AWS's *permanent* always-free tier — flagged here as the one line item to revisit if this project is kept running past its first year.