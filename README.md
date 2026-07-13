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

## Lessons Learned
- AWS billing metrics only exist in the us-east-1 region, regardless of which region your actual resources live in.
- Root account should never be used for daily work — enabling MFA on root and then creating a separate least-privilege IAM user is the standard first step on any new AWS account.

## Known Limitations

## Future Improvements

## Interview Questions & Answers