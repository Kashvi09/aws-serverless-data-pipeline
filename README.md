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

## Known Limitations

## Future Improvements

## Interview Questions & Answers