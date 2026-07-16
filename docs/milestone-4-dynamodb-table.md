# Milestone 3: DynamoDB Table for Processed Data
**Why:** Until now, Lambda only logged that a file had been uploaded — that information disappeared into CloudWatch logs, not queryable or usable by anything else. This milestone gives the pipeline a persistent, queryable place to store processed results, which Milestone 5's API will read from. We chose DynamoDB over RDS because our data has no relational structure requiring joins across tables — it's a simple "one record per file" lookup, which fits DynamoDB's key-based access model naturally, without RDS's schema rigidity or its continuous hourly billing. 

**What was built:**
- DynamoDB table: ProcessedFilesMetadata, partition key file_id (String), capacity mode: on-demand.
- Updated Lambda function (process-ingested-data): now writes an item to the table on every S3 trigger, storing file_id (reusing the S3 object key, since it's guaranteed unique within the bucket), bucket, and status.
- IAM update: attached AmazonDynamoDBFullAccess to lambda-s3-read-role, alongside the earlier AmazonS3ReadOnlyAccess.

**Lessons Learned**
- On-demand capacity charges only for actual reads/writes performed, with no pre-provisioned throughput to pay for — and like Lambda, DynamoDB's free tier is part of AWS's permanent always-free tier, not just the first 12 months. This makes it a strong fit for a low-traffic personal project.
- DynamoDB's lack of a fixed schema means different items in the same table could technically have different attributes — this flexibility is a tradeoff, not a limitation: useful when relationships between records don't matter, but the wrong choice if the project needed relational queries (joins, multi-table transactions).
- Reusing the S3 object key as the DynamoDB partition key worked because S3 already guarantees uniqueness within a bucket — avoided generating a separate ID for something that already had one.