# Milestone 1: Account Safety Net
**Why:** Before creating an AWS resource, I first secured the account itself, since a misconfigured and compromised account poses more risk than any single service. I created a least privelege IAM user (instead of using the root for daily work) and enabled MFA on both accounts. I also set up a CloudWatch billing alarm that triggers an SNS email notification if estimated charges cross $1, so I'd catch any unexpected cost immediately instead of discovering it at the end of the month.

**What was built:**
- IAM user + group (least privilege, MFA-enabled)
- CloudWatch billing alarm
- SNS email notification

**Lessons Learned**
- AWS billing metrics only exist in the us-east-1 region, regardless of which region your actual resources live in.
- Root account should never be used for daily work — enabling MFA on root and then creating a separate least-privilege IAM user is the standard first step on any new AWS account.

**Screenshots**
![Cloudwatch billing alarm](../screenshots/milestone-1/Cloudwatch%20billing%20alarm.png)