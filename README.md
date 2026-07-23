# aws-serverless-data-pipeline
# Serverless Data Ingestion & Query Pipeline

> Status: 🚧 In Progress

## Overview
This project is a serverless data ingestion and query pipeline built on AWS, deployed entirely through Infrastructure as Code with an automated CI/CD pipeline.

Files uploaded to S3 automatically trigger a Lambda function that processes and stores structured metadata in DynamoDB. A second Lambda function, exposed through an API Gateway HTTP endpoint, allows that data to be queried on demand. The entire stack — S3, Lambda, DynamoDB, API Gateway, and all IAM roles — is defined as a single CloudFormation template and deployed automatically via GitHub Actions on every push to `main`. CloudWatch alarms monitor the pipeline for runtime errors, separate from a billing alarm that protects against unexpected cost.

Beyond the working pipeline, this project doubles as a hands-on exploration of AWS least-privilege IAM practices, cost-conscious serverless architecture, Infrastructure as Code, and CI/CD — including real permission failures encountered and resolved along the way (documented milestone by milestone in [`/docs`](./docs)).

## Architecture
![Pipeline architecture](./diagrams/architecture-diagram.svg)

**Ingestion path:** A file uploaded to S3 triggers the ingestion Lambda, which writes a processed record to DynamoDB.
**Query path:** A user's request to the API Gateway endpoint invokes the query Lambda, which reads from DynamoDB and returns JSON.

The entire stack is defined as Infrastructure as Code ('infrastructure/template.yaml') and deployed automatically via a GitHub Actions CI/CD pipeline on every push to `main`.

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
| CloudFormation | Defines all project resources as a single Infrastructure-as-Code template; enables one-click deploy and teardown |
| GitHub Actions | Automates CloudFormation deployment on push to main (CI/CD) |\
| CloudWatch (Alarms + Dashboard) | Monitors pipeline health — alerts on Lambda errors, visualizes invocation/error/duration metrics |


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
| CloudFormation | Orchestrates resource creation/deletion | Yes — always free (you pay only for the resources it creates, not the service itself) | $0 | No |
| GitHub Actions | Runs the deploy workflow | Yes — unlimited free for public repos, 2,000 min/month free for private | $0 at this usage | No — keep permanently |
| IAM User (github-actions-deployer) | Programmatic-only user for CI/CD deploys | Always free | $0 | No — keep, only while project is active |
| CloudWatch Alarm (ingestion errors) | Detects Lambda failures | Yes — within free 10 alarms/month | $0 | No — keep permanently |
| CloudWatch Dashboard (pipeline-health) | Visual pipeline metrics | Yes — first 3 dashboards free | $0 | No — keep permanently |
| SNS (pipeline-alerts topic) | Delivers error alarm email | Yes — within free tier | $0 | No | 

**Note:** API Gateway is the only resource in this project not part of AWS's *permanent* always-free tier — flagged here as the one line item to revisit if this project is kept running past its first year.

## Setup / Deployment Guide

## Milestone Log

Detailed write-ups (why each decision was made, what was built, lessons learned) live in [`/docs`](./docs):

- [Milestone 1: Account Safety Net](./docs/milestone-1-account-setup.md)
- [Milestone 2: S3 Ingestion Bucket + IAM Role](./docs/milestone-2-s3-iam.md)
- [Milestone 3: Lambda Function + S3 Event Trigger](./docs/milestone-3-lambda-trigger.md)
- [Milestone 4: DynamoDB Table for Processed Data](./docs/milestone-4-dynamodb-table.md)
- [Milestone 5: API Gateway Query Endpoint](./docs/milestone-5-httpapi-setup.md)
- [Milestone 6: Infrastructure As code](./docs/milestone-6-infrasturcture-as-code.md)
- [Milestone 7: CI/CD Pipeline (Github Actions)](./docs/milestone-7-github-actions.md)
- [Milestone 8: CloudWatch Dashboards + SNS Alerting](./docs/milestone-8-sns-alerting.md)

## Known Limitations
- The `/files` API Gateway endpoint is currently open — no authentication or API key required. Anyone with the URL can query it.
- The GitHub Actions deploy user (`github-actions-deployer`) currently has broad managed policies (Full Access across S3, DynamoDB, Lambda, API Gateway, IAM) rather than resource-scoped custom policies — a deliberate simplification, same pattern as earlier milestones.


## Future Improvements
- Add an API Gateway authorizer (API Key or Lambda authorizer) to restrict access to the query endpoint.
- Scope `github-actions-deployer`'s permissions down to exact resource ARNs, matching the least-privilege approach already applied to the Lambda execution roles in Milestone 6.
- Add a CloudWatch alarm on the query Lambda's Errors metric, and a separate alarm on API Gateway's 5xx metric — monitoring at both the Lambda layer and the API layer catches different failure classes (e.g., integration timeouts that wouldn't necessarily show up as a Lambda-level error).

## Interview Questions & Answers
A compiled set of technical Q&A covering IAM, DynamoDB, API Gateway, IaC, and CI/CD decisions made throughout this project → [`docs/interview-questions.md`](./docs/interview-questions.md)