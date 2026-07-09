---
title: "Prerequisites & Setup"
date: 2026-07-09
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

# Prerequisites & Setup

### 1. What you need

* An **AWS account** with administrator access (a personal/sandbox account is fine).
* **Node.js 20+** and **npm**.
* **AWS CLI v2** configured with credentials (`aws configure`).
* **AWS CDK CLI**: `npm install -g aws-cdk`
* **Git**.

{{% notice warning %}}
Set up an **AWS Budget** with an email alert (e.g., $5) before starting. It takes 2 minutes and protects you from surprises.
{{% /notice %}}

### 2. Clone the project

```bash
git clone https://github.com/Thien-132/music-instrument-store.git
cd music-instrument-store
npm install
```

The repository is a monorepo:

```
music-instrument-store/
├── frontend/          # Next.js storefront
├── services/          # Lambda functions (order-api, order-processing, notification, ...)
├── packages/          # Shared code (shared-utils: idempotency helpers)
├── infrastructure/    # CDK app: 4 stacks
└── docs/              # Design documents
```

### 3. Bootstrap CDK

CDK needs a one-time bootstrap per account/region (creates the staging S3 bucket and IAM roles):

```bash
cd infrastructure
npm install
npx cdk bootstrap
```

![cdk bootstrap output](/images/5-Workshop/5.2-bootstrap.png)
<!-- TODO: chèn ảnh chụp kết quả cdk bootstrap -->

### 4. Verify

```bash
npx cdk list
```

You should see the four stacks:

```
MusicStoreSecurityStack-dev
MusicStoreDatabaseStack-dev
MusicStoreAuthStack-dev
MusicStoreBackendStack-dev
```

{{% notice info %}}
The `-dev` suffix comes from the CDK context variable `env` (defaults to `dev`). Deploy a separate environment with `npx cdk deploy --context env=staging ...`.
{{% /notice %}}
