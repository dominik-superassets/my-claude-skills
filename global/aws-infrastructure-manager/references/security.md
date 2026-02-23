# AWS Security Best Practices

## IAM Policy Naming Convention

**ALWAYS follow this pattern:**
```
SynthCoreAI-{Service}-{Actions}
```

**Examples:**
- `SynthCoreAI-SES-SendOnly` - Send emails via SES
- `SynthCoreAI-S3-ReadWrite` - Full S3 bucket access
- `SynthCoreAI-RDS-ReadOnly` - Database read access
- `SynthCoreAI-Cognito-UserManagement` - Manage Cognito users
- `SynthCoreAI-Lambda-Execute` - Execute Lambda functions

---

## Least-Privilege Principle

**CORRECT - Minimum permissions:**
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

**WRONG - Over-permissive:**
```json
{
  "Effect": "Allow",
  "Action": "ses:*",
  "Resource": "*"
}
```

---

## Policy Extension Workflow

When user requests new functionality:

1. **Analyze current policies** - Check what's attached
2. **Identify gap** - Minimum additional permissions needed
3. **Suggest extension** - Propose new actions
4. **Create policy** - Follow naming convention
5. **Test on aws-dev** - Verify permissions work
6. **Promote to aws-prod** - Apply same policy

**Example:**
```
User: "I need to add file upload to S3"

Analysis:
- Current policy: None for S3
- Required actions: s3:PutObject, s3:GetObject
- Suggested policy: SynthCoreAI-S3-FileUpload

Policy:
{
  "PolicyName": "SynthCoreAI-S3-FileUpload",
  "PolicyDocument": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": ["s3:PutObject", "s3:GetObject"],
        "Resource": "arn:aws:s3:::synthcoreai-*/*"
      }
    ]
  }
}
```

---

## Secrets Management

**CORRECT - Use AWS Secrets Manager:**
```typescript
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

const client = new SecretsManagerClient({ region: 'us-east-1' });
const response = await client.send(
  new GetSecretValueCommand({ SecretId: 'synthcoreai/ses-credentials' })
);
const secrets = JSON.parse(response.SecretString);
```

**WRONG - Hardcoded:**
```typescript
const AWS_ACCESS_KEY_ID = 'AKIA...'; // Never do this!
```

---

## Encryption Requirements

| Service | Encryption |
|---------|-----------|
| S3 | AES-256 or KMS at rest |
| RDS | Enable encryption for databases |
| Secrets Manager | Automatic KMS encryption |
| In Transit | Always HTTPS/TLS |

---

## Access Control

- **Least privilege** for all IAM policies
- **MFA required** for production access
- **Temporary credentials** via STS when possible
- **Regular audit** of IAM policies and users

---

## Required Tags (ALL Resources)

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

---

## Security Scan Workflow

Security scanning is ENABLED on both environments:

```typescript
// Run Checkov on all resource creation
const securityScan = await mcp__aws_dev__run_checkov({
  explained_token: explained.explained_token
});

// If issues found, present options:
if (securityScan.scan_status === 'FAILED') {
  // 1) Fix issues
  // 2) Proceed anyway (with user approval)
  // 3) Cancel
}
```

---

## Quick Reference: Approval Requirements

| Task | MCP Server | Approval | Security Scan |
|------|-----------|----------|---------------|
| Create resource <$10/mo | aws-dev -> aws-prod | No | Yes |
| Create resource >=$10/mo | aws-dev -> aws-prod | Yes | Yes |
| Read prod resources | aws-prod | No | No |
| Update prod resources | aws-prod | Yes | Yes |
| Create IAM policy | aws-dev -> aws-prod | No | Yes |
| Delete resources | aws-dev or aws-prod | Yes | No |
