---
title: "Clean up"
date: 2026-07-09
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

# Clean up

Destroy everything to stop incurring costs. Because the whole system is defined in CDK, cleanup is almost a single command.

### 1. Destroy the stacks

```bash
cd infrastructure
npx cdk destroy --all
```

Confirm each stack with `y`. Notes:

* The DynamoDB table and S3 bucket are deleted automatically (`RemovalPolicy.DESTROY`, `autoDeleteObjects: true`) — this is why we set those flags for the dev environment.
* CloudWatch **log groups** created by Lambda may remain — delete them under *CloudWatch → Log groups* (filter `/aws/lambda/MusicStore`).

### 2. Delete backup recovery points

AWS Backup recovery points are intentionally **not** deleted with the stack:

1. Open **AWS Backup → Backup vaults → MusicStoreBackupVault**.
2. Select all recovery points → **Delete**.
3. The vault itself is then removed by the stack destroy (or delete it manually).

### 3. Verify nothing is left

```bash
aws dynamodb list-tables
aws sqs list-queues
aws s3 ls
```

Also check **Billing → Cost Explorer** the next day to confirm the daily cost dropped to ~$0.


{{% notice warning %}}
If you deployed the Cognito User Pool or bootstrapped CDK, the `CDKToolkit` stack and its S3 staging bucket remain. Keep them if you plan to deploy again; otherwise delete the CloudFormation stack `CDKToolkit` and empty/delete the `cdk-*-assets` bucket.
{{% /notice %}}

### Congratulations! 🎉

You have built and torn down a production-grade serverless data & messaging layer:

* DynamoDB single-table with PITR + scheduled backups
* SQS pipelines with DLQs and idempotent consumers
* EventBridge domain-event routing
* CloudWatch alarms wired to SNS

The complete source code is available in the project repository.
