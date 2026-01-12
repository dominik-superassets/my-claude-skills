---
name: aws-infrastructure-manager
description: Comprehensive AWS infrastructure management using aws-dev and aws-prod MCP servers. Use when creating AWS resources, managing infrastructure, setting up services, configuring IAM policies, deploying to AWS, syncing dev and prod environments, estimating costs, or managing cloud infrastructure. Ensures proper MCP server usage, dev-to-prod promotion, cost estimation, security scanning, and resource tracking across environments.
---

# AWS Infrastructure Manager

## Overview

Manage all AWS infrastructure for SynthCore using the aws-dev and aws-prod MCP servers defined in `.mcp.json`. This skill ensures proper environment separation, cost control, security compliance, and automated dev-to-prod promotion while giving coding agents full infrastructure management capabilities under guidance.

## When to Use This Skill

Use when:
- Creating any AWS resource (S3, SES, RDS, Lambda, Cognito, etc.)
- Setting up new AWS services
- Creating or modifying IAM policies
- Deploying infrastructure changes
- Syncing dev and prod environments
- Estimating infrastructure costs
- Managing cloud resources
- Configuring AWS services

## Critical Principles

### 1. Environment Separation
- **aws-dev**: Development and testing environment (full access)
- **aws-prod**: Production environment (read-only by default, write with approval)
- **Always test on aws-dev first** before promoting to aws-prod
- **Keep environments in sync** - resources in dev should eventually be promoted to prod

### 2. MCP Server Configuration

From `.mcp.json`:

```json
{
  "aws-dev": {
    "AWS_PROFILE": "synthcoreai-dev",
    "AWS_REGION": "us-east-1",
    "SECURITY_SCANNING": "enabled"
  },
  "aws-prod": {
    "AWS_PROFILE": "synthcoreai-prod",
    "AWS_REGION": "us-east-1",
    "SECURITY_SCANNING": "enabled",
    "readonly": true
  }
}
```

### 3. Cost Approval Threshold
- **Automatic approval**: Resources costing <$10/month
- **Requires approval**: Resources costing ≥$10/month
- **Always estimate costs** before creating resources
- **Show monthly cost projection** to user

### 4. Security & Compliance
- **Security scanning enabled** on both environments
- **Run Checkov** when creating resources
- **Enforce tagging standards** (MANAGED_BY, MCP_SERVER_SOURCE_CODE, etc.)
- **Least-privilege IAM policies** by default

## AWS Infrastructure Workflow

### Step 1: Check Environment Variables

Before any AWS operation, verify MCP server environment is properly configured:

```bash
# ALWAYS run this first
mcp__aws-dev__check_environment_variables()
```

This ensures:
- AWS credentials are configured
- Required environment variables are set
- MCP server can connect to AWS

### Step 2: Get AWS Session Info

Verify which AWS account and region will be used:

```bash
# For development work
mcp__aws-dev__get_aws_session_info(environment_token)

# For production queries (read-only)
mcp__aws-prod__get_aws_session_info(environment_token)
```

**CRITICAL**: Always display this information to the user so they know which AWS account will be affected.

Display format:
```
📍 AWS Context:
   Profile: synthcoreai-dev (or Environment Variables)
   Authentication: AWS SSO Profile
   Account ID: 123456789012
   Region: us-east-1
```

### Step 3: Resource Creation Workflow

Follow this exact sequence for **EVERY** resource creation:

#### 3.1 Start with aws-dev

Always create and test resources on aws-dev first:

```typescript
// 1. Generate infrastructure code
const generated = await mcp__aws_dev__generate_infrastructure_code({
  resource_type: "AWS::S3::Bucket",
  credentials_token,
  properties: {
    BucketName: "synthcoreai-documents-dev",
    Tags: [
      { Key: "MANAGED_BY", Value: "CCAPI-MCP-SERVER" },
      { Key: "MCP_SERVER_SOURCE_CODE", Value: "https://github.com/awslabs/mcp/tree/main/src/ccapi-mcp-server" },
      { Key: "MCP_SERVER_VERSION", Value: "1.0.0" },
      { Key: "Environment", Value: "development" },
      { Key: "Project", Value: "SynthCore" }
    ]
  }
});

// 2. Explain to user (MANDATORY - show CloudFormation + explanation)
const explained = await mcp__aws_dev__explain({
  generated_code_token: generated.properties_token,
  content: generated.properties_for_explanation,
  operation: "create",
  context: "S3 bucket creation for document storage"
});

// 3. IMMEDIATELY show user the CloudFormation template AND explanation
console.log(explained.cloudformation_template);
console.log(explained.explanation);

// 4. Run security scan (SECURITY_SCANNING=enabled)
const securityScan = await mcp__aws_dev__run_checkov({
  explained_token: explained.explained_token
});

// 5. Show security results to user
console.log(securityScan.summary);

// 6. If security issues found, ask user how to proceed
if (securityScan.scan_status === 'FAILED') {
  // Ask: "Security issues found. Options: 1) Fix issues 2) Proceed anyway 3) Cancel"
  // WAIT for user response
  // Call approve_security_findings() with user's decision
}

// 7. Create resource on aws-dev
const created = await mcp__aws_dev__create_resource({
  resource_type: "AWS::S3::Bucket",
  credentials_token,
  explained_token: explained.explained_token,
  security_scan_token: securityScan.security_scan_token
});
```

#### 3.2 Estimate Costs

Before creating resources, estimate monthly costs:

**Common AWS Service Costs** (as of 2025):
- S3 Standard Storage: $0.023/GB/month
- S3 Requests: $0.0004 per 1,000 PUT, $0.00004 per 1,000 GET
- SES Sending: $0.10 per 1,000 emails
- RDS (db.t3.micro): ~$15/month
- Lambda: $0.20 per 1M requests + $0.00001667/GB-second
- Cognito: Free up to 50,000 MAU, then $0.0055/MAU
- CloudWatch Logs: $0.50/GB ingested

**Cost Estimation Examples**:

```typescript
// S3 Bucket for documents
const s3Cost = {
  storage: "10GB average * $0.023 = $0.23/month",
  requests: "10,000 uploads/month * $0.0004/1000 = $0.004/month",
  total: "$0.25/month",
  approval: "✅ Auto-approved (<$10/month)"
};

// SES Email Service
const sesCost = {
  emails: "5,000 emails/month * $0.10/1000 = $0.50/month",
  total: "$0.50/month",
  approval: "✅ Auto-approved (<$10/month)"
};

// RDS Database (db.t3.small)
const rdsCost = {
  instance: "720 hours * $0.034 = $24.48/month",
  storage: "20GB * $0.115 = $2.30/month",
  backups: "Included in instance cost",
  total: "$26.78/month",
  approval: "⚠️ REQUIRES APPROVAL (≥$10/month)"
};
```

**Display format**:
```
💰 Cost Estimation:
   Monthly Cost: $X.XX
   Breakdown:
   - Component A: $X.XX
   - Component B: $X.XX

   ⚠️ This exceeds $10/month threshold. Proceed? (y/n)
```

If cost ≥ $10/month, **WAIT for user approval** before creating.

#### 3.3 Track for Prod Promotion

After successfully creating a resource on aws-dev, **use AWS tags to track promotion status**:

**Required Tags for Tracking**:
```typescript
{
  Tags: [
    // Standard tags
    { Key: "MANAGED_BY", Value: "CCAPI-MCP-SERVER" },
    { Key: "Environment", Value: "development" | "production" },
    { Key: "Project", Value: "SynthCore" },

    // Tracking tags
    { Key: "PromotionStatus", Value: "dev-only" | "synced" | "pending-prod" },
    { Key: "CreatedDate", Value: "2025-11-02" },
    { Key: "ProdCounterpart", Value: "resource-name-in-prod" },
    { Key: "CostEstimate", Value: "$0.25/month" },
    { Key: "Purpose", Value: "Deal Factory document storage" }
  ]
}
```

**To check what needs promotion**, query resources by tags:

```typescript
// List all dev resources marked for promotion
const devResources = await mcp__aws_dev__list_resources({
  resource_type: "AWS::S3::Bucket" // or any resource type
});

// Filter by PromotionStatus tag
const needsPromotion = devResources.filter(r =>
  r.Tags?.find(t => t.Key === "PromotionStatus" && t.Value === "dev-only")
);
```

**Promotion Status Values**:
- `dev-only` - Resource exists only in dev, needs prod promotion
- `pending-prod` - Approved for promotion, waiting for deployment
- `synced` - Resource exists in both dev and prod with matching configuration

#### 3.4 Promote to Production

When ready to promote to aws-prod:

**Promotion Checklist**:
- [ ] Resource tested and verified working on aws-dev
- [ ] Security scan passed
- [ ] Cost estimation completed and approved
- [ ] No active issues or errors
- [ ] Dependencies identified and documented
- [ ] User confirms promotion to prod

**Promotion Process**:

```typescript
// 1. Switch to aws-prod MCP server
// 2. Generate infrastructure code (use SAME properties as dev, change environment tags)
const prodGenerated = await mcp__aws_prod__generate_infrastructure_code({
  resource_type: "AWS::S3::Bucket",
  credentials_token,
  properties: {
    ...devProperties,
    BucketName: "synthcoreai-documents-prod", // Change -dev to -prod
    Tags: [
      ...devProperties.Tags.filter(t => t.Key !== "Environment"),
      { Key: "Environment", Value: "production" }
    ]
  }
});

// 3. Explain and show to user
const prodExplained = await mcp__aws_prod__explain({
  generated_code_token: prodGenerated.properties_token,
  content: prodGenerated.properties_for_explanation,
  operation: "create",
  context: "Promoting S3 bucket from dev to prod"
});

// 4. MANDATORY security scan on prod as well
const prodSecurityScan = await mcp__aws_prod__run_checkov({
  explained_token: prodExplained.explained_token
});

// 5. Create on aws-prod
const prodCreated = await mcp__aws_prod__create_resource({
  resource_type: "AWS::S3::Bucket",
  credentials_token,
  explained_token: prodExplained.explained_token,
  security_scan_token: prodSecurityScan.security_scan_token
});

// 6. Update promotion status tag on dev resource
await mcp__aws_dev__update_resource({
  resource_type: "AWS::S3::Bucket",
  identifier: "synthcoreai-documents-dev",
  credentials_token,
  explained_token,
  patch_document: [
    {
      op: "replace",
      path: "/Tags/PromotionStatus",
      value: "synced"
    },
    {
      op: "add",
      path: "/Tags/ProdCounterpart",
      value: "synthcoreai-documents-prod"
    }
  ]
});
```

### Step 4: IAM Policy Management

#### Naming Convention

**ALWAYS follow this naming pattern**:
```
SynthCoreAI-{Service}-{Actions}
```

**Examples**:
- `SynthCoreAI-SES-SendOnly` - Send emails via SES
- `SynthCoreAI-S3-ReadWrite` - Full S3 bucket access
- `SynthCoreAI-RDS-ReadOnly` - Database read access
- `SynthCoreAI-Cognito-UserManagement` - Manage Cognito users
- `SynthCoreAI-Lambda-Execute` - Execute Lambda functions

#### Least-Privilege Principle

When creating IAM policies, **ALWAYS start with minimum required permissions**:

**Example - SES Send-Only Policy**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ses:SendEmail",
        "ses:SendRawEmail"
      ],
      "Resource": "*"
    }
  ]
}
```

**NOT this** (over-permissive):
```json
{
  "Effect": "Allow",
  "Action": "ses:*",  // ❌ Too broad!
  "Resource": "*"
}
```

#### Policy Extension Workflow

When user requests new functionality requiring additional AWS permissions:

1. **Analyze current policies** - Check what policies are attached to IAM user/role
2. **Identify gap** - Determine minimum additional permissions needed
3. **Suggest extension** - Propose new actions to add to policy
4. **Create new policy if needed** - Follow naming convention
5. **Test on aws-dev first** - Verify permissions work
6. **Promote to aws-prod** - Apply same policy to prod

**Example**:

```
User: "I need to add file upload to S3"

Analysis:
- Current policy: None for S3
- Required actions: s3:PutObject, s3:GetObject
- Suggested policy: SynthCoreAI-S3-FileUpload

Policy Creation:
{
  "PolicyName": "SynthCoreAI-S3-FileUpload",
  "PolicyDocument": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "s3:PutObject",
          "s3:GetObject"
        ],
        "Resource": "arn:aws:s3:::synthcoreai-*/*"
      }
    ]
  }
}

Next Steps:
1. Create policy on aws-dev
2. Attach to IAM user: synthcoreai-dev-user
3. Test file upload functionality
4. If successful, create same policy on aws-prod
5. Track in resource-tracking.json
```

## Common AWS Services

### S3 (Storage)

**Use cases**: File uploads, static assets, backups, data lakes

**Cost**: ~$0.023/GB/month + request costs

**Example**:
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

### SES (Email)

**Use cases**: Transactional emails, notifications, newsletters

**Cost**: $0.10 per 1,000 emails

**Example**:
```typescript
// SES is already set up via scripts/aws-setup/
// Credentials stored in AWS Secrets Manager
// IAM user: synthcoreai-ses-sender
// Policy: SynthCoreAI-SES-SendOnly
```

### Cognito (Authentication)

**Use cases**: User authentication, SSO, user pools

**Cost**: Free up to 50,000 MAU

**Example**:
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

### RDS (Database)

**Use cases**: PostgreSQL database, relational data

**Cost**: $15-50/month depending on instance size

**Example**:
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

### Lambda (Serverless Functions)

**Use cases**: Background jobs, API processing, scheduled tasks

**Cost**: $0.20 per 1M requests

**Example**:
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

## Security Best Practices

### 1. Secrets Management

**NEVER hardcode credentials** in code or environment variables:

✅ **CORRECT** - Use AWS Secrets Manager:
```typescript
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: 'us-east-1' });
const response = await client.send(
  new GetSecretValueCommand({ SecretId: 'synthcoreai/ses-credentials' })
);
const secrets = JSON.parse(response.SecretString);
```

❌ **WRONG** - Hardcoded:
```typescript
const AWS_ACCESS_KEY_ID = 'AKIA...'; // Never do this!
```

### 2. Encryption

- **S3**: Enable encryption at rest (AES-256 or KMS)
- **RDS**: Enable encryption for databases
- **Secrets Manager**: Encrypts secrets automatically with KMS
- **In Transit**: Always use HTTPS/TLS

### 3. Access Control

- **Principle of least privilege** for all IAM policies
- **MFA required** for production access
- **Temporary credentials** via STS when possible
- **Regular audit** of IAM policies and users

### 4. Tagging Standards

**ALWAYS include these tags** on ALL resources:

```typescript
{
  Tags: [
    { Key: "MANAGED_BY", Value: "CCAPI-MCP-SERVER" },
    { Key: "MCP_SERVER_SOURCE_CODE", Value: "https://github.com/awslabs/mcp/tree/main/src/ccapi-mcp-server" },
    { Key: "MCP_SERVER_VERSION", Value: "1.0.0" },
    { Key: "Environment", Value: "development" | "production" },
    { Key: "Project", Value: "SynthCore" },
    { Key: "CostCenter", Value: "Engineering" },
    { Key: "CreatedBy", Value: "Claude Code" },
    { Key: "CreatedDate", Value: "2025-11-02" }
  ]
}
```

## Deployment Integration

### Amplify Deployment Flow

SynthCore uses AWS Amplify for automatic deployments:

**Development**:
- Branch: `dev`
- Trigger: Automatic on push to dev branch
- Environment: aws-dev resources
- URL: `https://dev.synthcoreai.com`

**Production**:
- Branch: `main`
- Trigger: Manual deployment
- Environment: aws-prod resources
- URL: `https://synthcoreai.com`

**Deployment Checklist**:
- [ ] All aws-dev resources tested and working
- [ ] Resources promoted to aws-prod
- [ ] Environment variables configured in Amplify
- [ ] Build completes successfully
- [ ] No linting errors (pnpm lint)
- [ ] No TypeScript errors (pnpm type-check)
- [ ] Deployment verified on production URL

## Resource Tracking via AWS Tags

Instead of maintaining a separate tracking file, **use AWS resource tags as the source of truth**:

### Query Dev Resources Needing Promotion

```typescript
// Get all S3 buckets in dev
const devBuckets = await mcp__aws_dev__list_resources({
  resource_type: "AWS::S3::Bucket"
});

// Filter for resources needing promotion
const needsPromotion = devBuckets.resources
  .map(async id => await mcp__aws_dev__get_resource({
    resource_type: "AWS::S3::Bucket",
    identifier: id
  }))
  .filter(bucket =>
    bucket.Tags?.find(t =>
      t.Key === "PromotionStatus" &&
      (t.Value === "dev-only" || t.Value === "pending-prod")
    )
  );

console.log("Resources needing promotion to prod:", needsPromotion);
```

### Check Dev/Prod Sync Status

```typescript
// Compare dev and prod resources
async function checkSyncStatus(resourceType: string) {
  const devResources = await mcp__aws_dev__list_resources({ resource_type: resourceType });
  const prodResources = await mcp__aws_prod__list_resources({ resource_type: resourceType });

  const report = {
    devOnly: [],
    prodOnly: [],
    synced: [],
    needsReview: []
  };

  for (const devId of devResources.resources) {
    const devResource = await mcp__aws_dev__get_resource({
      resource_type: resourceType,
      identifier: devId
    });

    const prodCounterpart = devResource.Tags?.find(t => t.Key === "ProdCounterpart")?.Value;

    if (!prodCounterpart) {
      report.devOnly.push(devId);
    } else if (prodResources.resources.includes(prodCounterpart)) {
      report.synced.push({ dev: devId, prod: prodCounterpart });
    } else {
      report.needsReview.push({ dev: devId, expectedProd: prodCounterpart });
    }
  }

  return report;
}
```

### Best Practices

- **Tag immediately** after creating any resource
- **Query tags dynamically** instead of maintaining static files
- **Use consistent tag keys** across all resources
- **Review weekly** to catch resources stuck in dev-only state
- **Automate promotion** for low-cost resources (<$10/month)

## Cost Monitoring

### Monthly Cost Review

Track AWS costs across both environments:

```
📊 Monthly AWS Costs (November 2025)

aws-dev:
├─ S3 Storage: $0.45
├─ SES Emails: $0.30
├─ Lambda Execution: $0.15
└─ Total: $0.90

aws-prod:
├─ S3 Storage: $2.50
├─ SES Emails: $5.00
├─ RDS Database: $26.78
├─ Lambda Execution: $1.20
└─ Total: $35.48

Combined Total: $36.38/month
```

**Cost Alerts**:
- Set up CloudWatch billing alarms
- Alert at $25, $50, $100 thresholds
- Review monthly with user
- Optimize expensive services

## Troubleshooting

### MCP Server Connection Issues

```bash
# Verify environment variables
mcp__aws_dev__check_environment_variables()

# Check AWS credentials
aws sts get-caller-identity --profile synthcoreai-dev

# Test MCP connection
mcp__aws_dev__get_aws_account_info()
```

### Resource Creation Fails

Common issues:
1. **Insufficient permissions** - Add required actions to IAM policy
2. **Resource already exists** - Check for naming conflicts
3. **Security scan fails** - Review Checkov results, fix issues
4. **Cost approval needed** - Estimate and get user approval

### Dev/Prod Drift Detection

Run drift check to find unsynchronized resources:

```typescript
const driftReport = resources
  .filter(r => r.status === 'drift' || r.status === 'dev-only')
  .map(r => ({
    name: r.name,
    issue: r.status,
    action: r.status === 'drift'
      ? 'Update prod to match dev'
      : 'Promote to prod'
  }));
```

## Resources

This skill integrates with:
- **MCP Configuration**: `.mcp.json` (aws-dev and aws-prod servers configuration)
- **AWS Setup Scripts**: `scripts/aws-setup/` (automated credential and IAM setup)
- **CLAUDE.md**: AWS access patterns, security guidelines, and autonomous decision-making authority
- **AWS Resource Tags**: Query dynamically via MCP list_resources() and get_resource() calls

## Quick Reference

| Task | MCP Server | Approval Needed | Security Scan |
|------|-----------|----------------|---------------|
| Create resource <$10/month | aws-dev → aws-prod | No | Yes |
| Create resource ≥$10/month | aws-dev → aws-prod | Yes | Yes |
| Read prod resources | aws-prod | No | No |
| Update prod resources | aws-prod | Yes | Yes |
| Create IAM policy | aws-dev → aws-prod | No | Yes |
| Delete resources | aws-dev or aws-prod | Yes | No |

**Remember**: Always test on aws-dev first, estimate costs, run security scans, and promote to aws-prod after verification!
