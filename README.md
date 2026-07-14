# aws-serverless-data-pipeline
# Serverless Data Ingestion & Query Pipeline

> Status: 🚧 In Progress

## Overview

## Architecture

## AWS Services Used
| Service | Purpose |
|---|---|
| IAM | Non-root user + group for daily console/CLI access, MFA-protected |
| CloudWatch | Billing alarm — monitors estimated account charges |
| SNS | Delivers billing alarm notification via email |


## Cost Breakdown
| Service | Purpose | Free Tier Eligible? | Possible Cost | Delete After Project? |
|---|---|---|---|---|
| IAM User/Group | Non-root access | Yes — always free | $0 | No — keep permanently |
| CloudWatch Alarm | Billing tripwire | Yes — first 10 alarms free | $0 at this usage | No — keep permanently |
| SNS Topic | Email delivery | Yes — 1,000 emails/month free | $0 at this scale | No — keep permanently |
| S3 Bucket (kashvi-pipeline-raw-data) | Raw data ingestion | Yes (free tier) | Near-zero | No |
| IAM Role (lambda-s3-read-role) | Lambda execution permissions | Always free | $0 | No |

## Setup / Deployment Guide

## Milestone Log

### Milestone 1: Account Safety Net
**Why:** Before creating an AWS resource, I first secured the account itself, since a misconfigured and compromised account poses more risk than any single service. I created a least privelege IAM user (instead of using the root for daily work) and enabled MFA on both accounts. I also set up a CloudWatch billing alarm that triggers an SNS email notification if estimated charges cross $1, so I'd catch any unexpected cost immediately instead of discovering it at the end of the month.

**What was built:**
- IAM user + group (least privilege, MFA-enabled)
- CloudWatch billing alarm
- SNS email notification

**Lessons Learned**
- AWS billing metrics only exist in the us-east-1 region, regardless of which region your actual resources live in.
- Root account should never be used for daily work — enabling MFA on root and then creating a separate least-privilege IAM user is the standard first step on any new AWS account.

### Milestone 2: S3 Ingestion Bucket + IAM Role
**Why:** We need an S3 bucket so that any incoming data in the pipeline has a place to land and be stored. Before writing Lambda code, we also make an IAM role - because when lambda eventually runs, it needs explicit permission to read from this bucket. AWS doesn't grant access by default; every permission must be deliberately defined. Creating this role now, scoped with read-only access, means Lambda will only ever be able to do what we've explicitly allowed — nothing more.

**What was built:**
- S3 bucket: kashvi-pipeline-raw-data, region ap-south-1 (nearest to my physical location, for lower latency). Public access is fully blocked — this bucket only needs to be reachable by internal AWS services, not the public internet.
- IAM role: lambda-s3-read-role, attached to the AmazonS3ReadOnlyAccess managed policy. This gives Lambda read-only permissions (no write/delete), scoped to the actions it can perform but not yet to a specific bucket.

**Lessons Learned**
- True least privilege has two dimensions: which actions are allowed, and which resources they apply to. AmazonS3ReadOnlyAccess only restricts the actions (read-only) — it doesn't restrict which bucket, so this role can technically read from any S3 bucket in the account, not just this one. This will be fixed with a custom scoped policy when we rewrite the project as CloudFormation.
- IAM roles grant permissions but don't trigger anything on their own — that's a separate mechanism (S3 Event Notifications, set up in the next milestone). It's easy to conflate "who's allowed to act" with "what causes the action to happen," but they're distinct pieces of the architecture.

## Known Limitations

## Future Improvements

## Interview Questions & Answers