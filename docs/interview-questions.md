## Interview Questions & Answers

**Q1: Why should you never use the AWS root account for daily work?**
Root has unlimited access to every service and billing, with no way to restrict it. If compromised, the damage is unbounded. A least-privilege IAM user limits the blast radius of any single mistake or compromise.

**Q2: What's the difference between an IAM role and an IAM user?**
A user is for a human logging in with credentials. A role has no permanent credentials — it's temporarily assumed by an AWS service (like Lambda) to perform an action, then the permission ends. This is why services use roles, not users.

**Q3: What's the difference between the Lambda execution role and a resource-based policy (e.g. Lambda's permission for S3 to invoke it)?**
The execution role controls what the Lambda function itself is allowed to do (read S3, write DynamoDB, write logs). A resource-based policy controls who is allowed to invoke the Lambda function in the first place. Two separate trust directions.

**Q4: Why is DynamoDB a better fit than RDS for this pipeline?**
The data has no relational structure requiring joins — it's a simple key-based lookup (one record per file). DynamoDB fits that natively with on-demand pricing and no fixed schema; RDS would add relational overhead and continuous hourly billing this project doesn't need.

**Q5: What's the difference between `scan()` and `query()` in DynamoDB?**
`scan()` reads every item in the table regardless of what's needed — fine at small scale, but doesn't scale or cost-scale well. `query()` retrieves only items matching a specific key, and is the production-appropriate choice as a table grows.

**Q6: Why did you choose HTTP API over REST API in API Gateway?**
Both handle standard HTTP requests. REST API includes extra features (API keys, usage plans, request validation) this project doesn't need. HTTP API is simpler to configure and cheaper per request — the right-sized tool for a simple query endpoint.

**Q7: Why does Infrastructure as Code matter over manually clicking through the console?**
Manual setup is error-prone and hard to reproduce or tear down completely — it's easy to misconfigure something or accidentally leave a resource running. CloudFormation defines the entire environment as code: repeatable, reviewable, and a single "Delete Stack" removes everything with no orphaned resources.

**Q8: Why can't you hardcode AWS credentials in a GitHub Actions workflow file?**
The repo is public — any hardcoded key would be exposed to anyone. Instead, credentials are stored as GitHub's encrypted repository secrets, referenced by name in the workflow and only decrypted at runtime inside the job.

**Q9: What is `iam:PassRole`, and why is it security-sensitive?**
It's the permission required to let a user hand off an IAM role to a service (e.g., letting CloudFormation assign an execution role to a new Lambda function). It's sensitive because, if scoped too broadly, it's a known vector for privilege escalation — a user could pass a far more powerful role than their own permissions would otherwise allow.

**Q10: Walk through what happens end-to-end when the ingestion Lambda throws an error.**
Lambda automatically increments its `Errors` metric in CloudWatch. The CloudWatch Alarm (watching `Errors >= 1` over 5 minutes) evaluates this and transitions from `OK` to `ALARM`. That state change triggers the alarm's SNS action, publishing a message to the `pipeline-alerts` topic. SNS delivers it to the subscribed email. The whole chain is push-based — no manual log-checking required.