# AWS Infrastructure Troubleshooting

## MCP Server Connection Issues

### Verify Environment

```bash
# Check environment variables
mcp__aws_dev__check_environment_variables()

# Verify AWS credentials
aws sts get-caller-identity --profile synthcoreai-dev

# Test MCP connection
mcp__aws_dev__get_aws_account_info()
```

### Common Fixes

1. **Credentials expired** - Re-authenticate via AWS SSO
2. **Wrong profile** - Verify `.mcp.json` configuration
3. **Region mismatch** - Confirm us-east-1 is set

---

## Resource Creation Failures

| Issue | Cause | Solution |
|-------|-------|----------|
| Access Denied | Insufficient IAM permissions | Add required actions to policy |
| Resource exists | Naming conflict | Check for existing resources, use unique name |
| Security scan fails | Checkov found issues | Review results, fix or approve exceptions |
| Cost approval | Resource >= $10/month | Get user approval before proceeding |

---

## Dev/Prod Drift Detection

### Check Sync Status

```typescript
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

### Find Resources Needing Promotion

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

---

## Cost Monitoring Issues

### Set Up Billing Alarms

```
Recommended thresholds:
- $25/month - Warning
- $50/month - Alert
- $100/month - Critical
```

### Monthly Review Format

```
Monthly AWS Costs:

aws-dev:
  S3 Storage: $0.45
  SES Emails: $0.30
  Lambda: $0.15
  Total: $0.90

aws-prod:
  S3 Storage: $2.50
  SES Emails: $5.00
  RDS Database: $26.78
  Lambda: $1.20
  Total: $35.48

Combined Total: $36.38/month
```

---

## Security Scan Failures

### Checkov Common Issues

| Finding | Fix |
|---------|-----|
| S3 bucket public access | Enable PublicAccessBlockConfiguration |
| Unencrypted storage | Enable encryption at rest |
| Over-permissive IAM | Restrict to specific resources/actions |
| Missing tags | Add required MANAGED_BY tags |

### Approve Security Findings

If issues are acceptable (with user approval):

```typescript
await mcp__aws_dev__approve_security_findings({
  security_scan_token: securityScan.security_scan_token,
  approved_findings: ["CKV_AWS_xxx"],
  reason: "User approved: <justification>"
});
```

---

## IAM Permission Errors

### Diagnose Missing Permissions

1. Check error message for required action
2. Review attached policies
3. Identify gap
4. Add minimum required permission

### Common Missing Permissions

| Error | Add Permission |
|-------|---------------|
| s3:PutObject denied | `s3:PutObject` on bucket ARN |
| ses:SendEmail denied | `ses:SendEmail` on `*` |
| rds:DescribeDBInstances denied | `rds:DescribeDBInstances` |
| lambda:InvokeFunction denied | `lambda:InvokeFunction` on function ARN |

---

## Related Resources

- **MCP Configuration:** `.mcp.json`
- **AWS Setup Scripts:** `scripts/aws-setup/`
- **Security Guidelines:** `CLAUDE.md`
