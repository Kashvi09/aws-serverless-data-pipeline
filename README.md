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
| Lambda | Runs code automatically in response to S3 upload events; processes incoming files |



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

## Setup / Deployment Guide

## Milestone Log

Detailed write-ups (why each decision was made, what was built, lessons learned) live in [`/docs`](./docs):

- [Milestone 1: Account Safety Net](./docs/milestone-1-account-setup.md)
- [Milestone 2: S3 Ingestion Bucket + IAM Role](./docs/milestone-2-s3-iam.md)
- [Milestone 3: Lambda Function + S3 Event Trigger](./docs/milestone-3-lambda-trigger.md)

## Known Limitations

## Future Improvements

## Interview Questions & Answers