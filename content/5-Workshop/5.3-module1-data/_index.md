---
title: "Module 1: Data Layer"
date: 2026-07-09
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

# Module 1: Data Layer — DynamoDB, S3 & AWS Backup

In this module we define the **DatabaseStack** (`infrastructure/lib/database-stack.ts`): a DynamoDB single-table, an S3 bucket for product images, and automated backups.

### 1. DynamoDB single-table design

Instead of one table per entity, **all entities live in one table** distinguished by key prefixes:

| Entity | PK | SK |
| --- | --- | --- |
| Product | `PRODUCT#<id>` | `METADATA` |
| Order | `ORDER#<id>` | `METADATA` |
| User's orders | `USER#<id>` | `ORDER#<orderId>` (via GSI1) |
| Rating | `PRODUCT#<id>` | `RATING#<userId>` |
| Coupon | `COUPON#<code>` | `METADATA` |
| Idempotency claim | `EVENT#<eventId>` | `METADATA` |

The table is created with on-demand billing and **Point-in-Time Recovery**:

```typescript
this.mainTable = new dynamodb.Table(this, 'MusicStoreMainTable', {
  partitionKey: { name: 'PK', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'SK', type: dynamodb.AttributeType.STRING },
  billingMode: dynamodb.BillingMode.PAY_PER_REQUEST, // serverless cost model
  removalPolicy: cdk.RemovalPolicy.DESTROY,          // dev only!
  pointInTimeRecovery: true,                          // PITR: restore to any second in 35 days
});
```

**GSIs** support additional access patterns without Scans:

```typescript
this.mainTable.addGlobalSecondaryIndex({
  indexName: 'GSI1',
  partitionKey: { name: 'GSI1PK', type: dynamodb.AttributeType.STRING },
  sortKey: { name: 'GSI1SK', type: dynamodb.AttributeType.STRING },
  projectionType: dynamodb.ProjectionType.ALL,
});
// GSI2: query products by category (GSI2PK = TYPE#<type>) instead of scanning
```

### 2. S3 bucket for product images

```typescript
this.productsBucket = new s3.Bucket(this, 'MusicStoreProductsBucket', {
  publicReadAccess: true,          // product images are public content
  removalPolicy: cdk.RemovalPolicy.DESTROY,
  autoDeleteObjects: true,         // empty the bucket on destroy
  cors: [{
    allowedMethods: [s3.HttpMethods.GET, s3.HttpMethods.POST, s3.HttpMethods.PUT],
    allowedOrigins: ['*'],
    allowedHeaders: ['*'],
  }],
});
```

CORS enables the admin panel to upload images directly from the browser with **presigned URLs**.

### 3. AWS Backup — scheduled backups

PITR protects against accidental writes; **AWS Backup** adds long-lived recovery points:

```typescript
const backupPlan = new backup.BackupPlan(this, 'MusicStoreBackupPlan', {
  backupPlanName: 'MusicStoreBackupPlan',
});

// Daily at 00:00 UTC, kept 30 days
backupPlan.addRule(new backup.BackupPlanRule({
  ruleName: 'DailyBackup',
  backupVault,
  scheduleExpression: events.Schedule.expression('cron(0 0 * * ? *)'),
  deleteAfter: cdk.Duration.days(30),
}));

// Weekly on Sunday, kept 90 days
backupPlan.addRule(new backup.BackupPlanRule({
  ruleName: 'WeeklyBackup',
  backupVault,
  scheduleExpression: events.Schedule.expression('cron(0 0 ? * SUN *)'),
  deleteAfter: cdk.Duration.days(90),
}));

backupPlan.addSelection('DynamoDbBackupSelection', {
  resources: [backup.BackupResource.fromDynamoDbTable(this.mainTable)],
});
```

### 4. Deploy and verify

```bash
npx cdk deploy MusicStoreDatabaseStack-dev
```

Then in the AWS console verify:

1. **DynamoDB → Tables** — the table exists with GSI1/GSI2 and PITR **Enabled**.
2. **S3** — the products bucket exists with CORS configured.
3. **AWS Backup → Backup plans** — `MusicStoreBackupPlan` with 2 rules.

### 5. Seed the catalog

```bash
cd ../scripts
node seed-products.mjs   # loads 15 sample instruments into the table
```

