# Milestone 8: CloudWatch Dashboards + SNS Alerting
**Why:**Milestone 1's billing alarm protects against unexpected cost — it fires when estimated charges cross a threshold, regardless of whether the pipeline itself works. This milestone adds a different kind of monitoring: is the pipeline actually functioning correctly during real operation? Rather than manually digging through CloudWatch Logs to check for failures, an alarm on the Errors metric notifies immediately if the ingestion Lambda throws an exception — a runtime health signal, separate from cost.

**What was built:**
- CloudWatch Alarm on process-ingested-data's Errors metric — threshold: Errors >= 1 for 1 datapoint within a 5-minute period, meaning even a single error in any 5-minute window triggers it.
- A new, dedicated SNS topic (pipeline-alerts), kept separate from the billing alerts topic so alert categories are distinguishable at a glance.
- CloudWatch Dashboard (pipeline-health) showing Invocations, Errors, and Duration for both Lambda functions.
- Tested the alarm by intentionally raising an exception as the first line of the ingestion function's code, deploying that broken version, then uploading a test file to S3. The alarm correctly entered ALARM state, confirming the whole chain (error → metric → alarm → SNS → email) works end-to-end. Reverted the code afterward.

**Lessons Learned**
- Lambda automatically emits metrics like Invocations, Errors, and Duration for every function without any code needed — structured numeric metrics AWS tracks natively, distinct from the print() log lines written manually back in Milestone 3.
- A dashboard alone doesn't notify anyone — it's a passive visual summary, only useful if someone actively looks at it. An alarm is the active counterpart, pushing a notification the moment a threshold is crossed.
- Deliberately breaking the function to test the alarm was worth doing — an alarm that's never actually fired is unverified; seeing it correctly transition to ALARM state on a real, intentional failure is the only real proof the whole notification chain works.