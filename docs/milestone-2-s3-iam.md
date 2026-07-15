# Milestone 2: S3 Ingestion Bucket + IAM Role
**Why:** We need an S3 bucket so that any incoming data in the pipeline has a place to land and be stored. Before writing Lambda code, we also make an IAM role - because when lambda eventually runs, it needs explicit permission to read from this bucket. AWS doesn't grant access by default; every permission must be deliberately defined. Creating this role now, scoped with read-only access, means Lambda will only ever be able to do what we've explicitly allowed — nothing more.

**What was built:**
- S3 bucket: kashvi-pipeline-raw-data, region ap-south-1 (nearest to my physical location, for lower latency). Public access is fully blocked — this bucket only needs to be reachable by internal AWS services, not the public internet.
- IAM role: lambda-s3-read-role, attached to the AmazonS3ReadOnlyAccess managed policy. This gives Lambda read-only permissions (no write/delete), scoped to the actions it can perform but not yet to a specific bucket.

**Lessons Learned**
- True least privilege has two dimensions: which actions are allowed, and which resources they apply to. AmazonS3ReadOnlyAccess only restricts the actions (read-only) — it doesn't restrict which bucket, so this role can technically read from any S3 bucket in the account, not just this one. This will be fixed with a custom scoped policy when we rewrite the project as CloudFormation.
- IAM roles grant permissions but don't trigger anything on their own — that's a separate mechanism (S3 Event Notifications, set up in the next milestone). It's easy to conflate "who's allowed to act" with "what causes the action to happen," but they're distinct pieces of the architecture.