# AWS Resource Creation Workflow

## Step 1: Verify Environment

Before any AWS operation, verify the MCP server environment:

```bash
# ALWAYS run first
mcp__aws-dev__check_environment_variables()
mcp__aws-dev__get_aws_session_info(environment_token)
```

Display to user:
```
AWS Context:
   Profile: synthcoreai-dev
   Account ID: 123456789012
   Region: us-east-1
```

## Step 2: Create on aws-dev

Follow this exact sequence for EVERY resource:

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
      { Key: "Project", Value: "SynthCore" },
      { Key: "PromotionStatus", Value: "dev-only" },
      { Key: "CreatedDate", Value: "2025-11-02" },
      { Key: "CostEstimate", Value: "$0.25/month" }
    ]
  }
});

// 2. Explain to user (MANDATORY)
const explained = await mcp__aws_dev__explain({
  generated_code_token: generated.properties_token,
  content: generated.properties_for_explanation,
  operation: "create",
  context: "S3 bucket creation for document storage"
});

// 3. Show CloudFormation template AND explanation
console.log(explained.cloudformation_template);
console.log(explained.explanation);

// 4. Run security scan
const securityScan = await mcp__aws_dev__run_checkov({
  explained_token: explained.explained_token
});

// 5. If security issues found, ask user how to proceed
if (securityScan.scan_status === 'FAILED') {
  // Ask: "Security issues found. Options: 1) Fix issues 2) Proceed anyway 3) Cancel"
  // WAIT for user response
}

// 6. Create resource
const created = await mcp__aws_dev__create_resource({
  resource_type: "AWS::S3::Bucket",
  credentials_token,
  explained_token: explained.explained_token,
  security_scan_token: securityScan.security_scan_token
});
```

## Step 3: Estimate Costs

Before creating, estimate monthly costs:

| Service | Cost |
|---------|------|
| S3 Standard | $0.023/GB/month |
| S3 Requests | $0.0004/1000 PUT, $0.00004/1000 GET |
| SES | $0.10/1000 emails |
| RDS db.t3.micro | ~$15/month |
| Lambda | $0.20/1M requests |
| Cognito | Free <50K MAU, then $0.0055/MAU |

**Cost Thresholds:**
- < $10/month: Auto-approved
- >= $10/month: REQUIRES user approval

Display format:
```
Cost Estimation:
   Monthly Cost: $X.XX
   Breakdown:
   - Component A: $X.XX
   - Component B: $X.XX

   This exceeds $10/month threshold. Proceed? (y/n)
```

## Step 4: Promote to Production

**Promotion Checklist:**
- [ ] Resource tested on aws-dev
- [ ] Security scan passed
- [ ] Cost estimation approved
- [ ] No active issues
- [ ] User confirms promotion

**Promotion Process:**

```typescript
// 1. Generate for prod (change -dev to -prod)
const prodGenerated = await mcp__aws_prod__generate_infrastructure_code({
  resource_type: "AWS::S3::Bucket",
  credentials_token,
  properties: {
    ...devProperties,
    BucketName: "synthcoreai-documents-prod",
    Tags: [
      ...devProperties.Tags.filter(t => t.Key !== "Environment"),
      { Key: "Environment", Value: "production" }
    ]
  }
});

// 2. Explain and security scan
const prodExplained = await mcp__aws_prod__explain({...});
const prodSecurityScan = await mcp__aws_prod__run_checkov({...});

// 3. Create on aws-prod
const prodCreated = await mcp__aws_prod__create_resource({...});

// 4. Update dev resource tag to "synced"
await mcp__aws_dev__update_resource({
  patch_document: [
    { op: "replace", path: "/Tags/PromotionStatus", value: "synced" },
    { op: "add", path: "/Tags/ProdCounterpart", value: "synthcoreai-documents-prod" }
  ]
});
```

## Promotion Status Values

| Status | Meaning |
|--------|---------|
| `dev-only` | Exists only in dev, needs promotion |
| `pending-prod` | Approved, waiting deployment |
| `synced` | Exists in both with matching config |

## Query Resources by Status

```typescript
// List dev resources marked for promotion
const devResources = await mcp__aws_dev__list_resources({
  resource_type: "AWS::S3::Bucket"
});

// Get details and check PromotionStatus tag
const needsPromotion = devResources.filter(r =>
  r.Tags?.find(t => t.Key === "PromotionStatus" && t.Value === "dev-only")
);
```
