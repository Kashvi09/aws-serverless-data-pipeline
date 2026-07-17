# aws-serverless-data-pipeline
# Serverless Data Ingestion & Query Pipeline

> Status: 🚧 In Progress

## Overview
This project is a serverless data ingestion and query pipeline built on AWS, designed to accept raw data uploads, process them automatically, and make the processed results queryable through an API — without any servers running continuously. 

The pipeline follows an event-driven architecture: files uploaded to S3 trigger a Lambda function, which will eventually process and store structured data in DynamoDB, queryable via API Gateway. The project also serves as a hands-on exploration of AWS least-privilege IAM practices, cost-conscious architecture decisions, and Infrastructure as Code (introduced in later milestones).

Currently implemented: account security setup, S3 ingestion bucket, and an S3-triggered Lambda function that confirms the event chain end-to-end. Upcoming: DynamoDB storage, API Gateway query endpoint, and a full CloudFormation rewrite.

## Architecture

## AWS Services Used
| Service | Purpose |
|---|---|
| IAM | Non-root user + group for daily console/CLI access, MFA-protected |
| CloudWatch | Billing alarm — monitors estimated account charges |
| SNS | Delivers billing alarm notification via email |
| S3 | Stores raw data files uploaded to the pipeline; triggers Lambda on new uploads |
| IAM Role (lambda-s3-read-role) | Grants Lambda least-privilege permissions to read from S3 and write logs to CloudWatch |
| Lambda | Runs code automatically in response to events (S3 uploads, API requests); two functions: one ingests, one queries |
| DynamoDB | Stores processed file metadata (file_id, bucket, status) for later querying |
| IAM Role (lambda-dynamodb-read-role) | Grants query Lambda least-privilege permissions to read from DynamoDB and write logs to CloudWatch |
| API Gateway (HTTP API) | Exposes a public GET endpoint that queries processed file data |


## Cost Breakdown
| Service | Purpose | Free Tier Eligible? | Possible Cost | Delete After Project? |
|---|---|---|---|---|
| IAM User/Group | Non-root access | Yes — always free | $0 | No — keep permanently |
| CloudWatch Alarm | Billing tripwire | Yes — first 10 alarms free | $0 at this usage | No — keep permanently |
| SNS Topic | Email delivery | Yes — 1,000 emails/month free | $0 at this scale | No — keep permanently |
| S3 Bucket (kashvi-pipeline-raw-data) | Raw data ingestion | Yes (free tier) | Near-zero | No |
| IAM Role (lambda-s3-read-role) | Lambda execution permissions | Always free | $0 | No |
| Lambda (process-incoming-data) | Processes file son S3 upload | Yes - 1M requests + 400,000 GB-seconds/month, permanently | Effectively $0 at test scale | No - keep permanently |
| S3 Event Notification | Triggers lambda on upload | Free(bucket config, not a billed resource) | $0 | No |
| DynammoDB Table (ProcessedFilesMetadata) | Stores processed file metadata | Yes - 25GB storage + 25 write/read capacity units always free(this is part of AWS's always-free tier, not just first 12 months); on-demand mode also has its own always-free monthly request allowance | Effectively $0 at test scale | A handful of test writes/reads | No - keep permanently |
| Lambda (query-processed-files) | Reads DynamoDB, returns JSON via API | Yes — always-free tier | Effectively $0 at test scale | No — keep permanently |
| IAM Role (lambda-dynamodb-read-role) | Query Lambda execution permissions | Always free | $0 | No |
| API Gateway (HTTP API) | Public query endpoint | Yes — **first 12 months only** | $0 now; ~$1/million requests after 12 months | No — but revisit cost assumption after year 1 |

**Note:** API Gateway is the only resource in this project not part of AWS's *permanent* always-free tier — flagged here as the one line item to revisit if this project is kept running past its first year.

## Setup / Deployment Guide

## Milestone Log

Detailed write-ups (why each decision was made, what was built, lessons learned) live in [`/docs`](./docs):

- [Milestone 1: Account Safety Net](./docs/milestone-1-account-setup.md)
- [Milestone 2: S3 Ingestion Bucket + IAM Role](./docs/milestone-2-s3-iam.md)
- [Milestone 3: Lambda Function + S3 Event Trigger](./docs/milestone-3-lambda-trigger.md)
- [Milestone 4: DynamoDB Table for Processed Data](./docs/milestone-4-dynamodb-table.md)
- [Milestone 5: API Gateway Query Endpoint](./docs/milestone-5-httpapi-setup.md)

## Known Limitations
- The `/files` API Gateway endpoint is currently open — no authentication or API key required. Anyone with the URL can query it.

## Future Improvements
- Add an API Gateway authorizer (API Key or Lambda authorizer) to restrict access to the query endpoint.

## Interview Questions & Answers