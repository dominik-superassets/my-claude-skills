# Common AWS Services Patterns

## S3 (Storage)

**Use cases:** File uploads, static assets, backups, data lakes

**Cost:** ~$0.023/GB/month + request costs

```typescript
create_resource({
  resource_type: "AWS::S3::Bucket",
  properties: {
    BucketName: "synthcoreai-{purpose}-{env}",
    PublicAccessBlockConfiguration: {
      BlockPublicAcls: true,
      BlockPublicPolicy: true,
      IgnorePublicAcls: true,
      RestrictPublicBuckets: true
    },
    VersioningConfiguration: {
      Status: "Enabled"
    },
    Tags: [...]
  }
});
```

---

## SES (Email)

**Use cases:** Transactional emails, notifications, newsletters

**Cost:** $0.10 per 1,000 emails

```typescript
// SES setup via scripts/aws-setup/
// Credentials in AWS Secrets Manager
// IAM user: synthcoreai-ses-sender
// Policy: SynthCoreAI-SES-SendOnly
```

---

## Cognito (Authentication)

**Use cases:** User authentication, SSO, user pools

**Cost:** Free up to 50,000 MAU

```typescript
create_resource({
  resource_type: "AWS::Cognito::UserPool",
  properties: {
    UserPoolName: "synthcoreai-users-{env}",
    AutoVerifiedAttributes: ["email"],
    Policies: {
      PasswordPolicy: {
        MinimumLength: 8,
        RequireUppercase: true,
        RequireLowercase: true,
        RequireNumbers: true,
        RequireSymbols: true
      }
    },
    Tags: [...]
  }
});
```

---

## RDS (Database)

**Use cases:** PostgreSQL database, relational data

**Cost:** $15-50/month depending on instance size

```typescript
create_resource({
  resource_type: "AWS::RDS::DBInstance",
  properties: {
    DBInstanceIdentifier: "synthcoreai-db-{env}",
    Engine: "postgres",
    EngineVersion: "15.4",
    DBInstanceClass: "db.t3.micro",
    AllocatedStorage: 20,
    StorageType: "gp3",
    MasterUsername: "synthcoreai_admin",
    BackupRetentionPeriod: 7,
    Tags: [...]
  }
});
```

---

## Lambda (Serverless Functions)

**Use cases:** Background jobs, API processing, scheduled tasks

**Cost:** $0.20 per 1M requests

```typescript
create_resource({
  resource_type: "AWS::Lambda::Function",
  properties: {
    FunctionName: "synthcoreai-{function}-{env}",
    Runtime: "nodejs20.x",
    Handler: "index.handler",
    Role: "arn:aws:iam::123456789012:role/lambda-execution-role",
    Code: {
      ZipFile: "..." // Or S3Bucket + S3Key
    },
    Tags: [...]
  }
});
```

---

## Quick Cost Reference

| Service | Typical Cost |
|---------|-------------|
| S3 (10GB) | ~$0.25/month |
| SES (5K emails) | ~$0.50/month |
| RDS db.t3.micro | ~$15/month |
| RDS db.t3.small | ~$27/month |
| Lambda (1M requests) | ~$0.20/month |
| Cognito (<50K users) | Free |
| CloudWatch Logs | $0.50/GB |

---

## Amplify Deployment Integration

**Development:**
- Branch: `dev`
- Trigger: Automatic on push
- Environment: aws-dev resources
- URL: `https://dev.synthcoreai.com`

**Production:**
- Branch: `main`
- Trigger: Manual deployment
- Environment: aws-prod resources
- URL: `https://synthcoreai.com`
