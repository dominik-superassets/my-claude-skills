---
name: aws-infrastructure-manager
description: Use when creating AWS resources, managing dev/prod infrastructure, deploying to AWS, or configuring IAM policies. Handles MCP server selection (aws-dev/aws-prod), security scanning, cost estimation, and environment promotion.
metadata:
  author: DDD
  version: 1.0.0
  category: aws
---

# AWS Infrastructure Manager

Manage AWS infrastructure using the aws-dev and aws-prod MCP servers. This skill ensures proper environment separation, cost control, security compliance, and automated dev-to-prod promotion.

## When to Use

- Creating any AWS resource (S3, SES, RDS, Lambda, Cognito)
- Setting up or configuring AWS services
- Creating or modifying IAM policies
- Deploying infrastructure changes
- Syncing dev and prod environments
- Estimating infrastructure costs

## Critical Principles

**Environment Separation:**
- `aws-dev` - Development (full access, test here first)
- `aws-prod` - Production (read-only by default, write requires approval)
- Always create on aws-dev first, then promote to aws-prod

**Cost Thresholds:**
- < $10/month: Auto-approved
- >= $10/month: Requires user approval
- Always show cost estimation before creating

**Security Requirements:**
- Run Checkov security scan on all resource creation
- Use least-privilege IAM policies
- Enforce tagging standards on all resources

## MCP Server Selection Rule

**ALWAYS check environment first:**
```bash
mcp__aws-dev__check_environment_variables()
mcp__aws-dev__get_aws_session_info(environment_token)
```

**Display to user before any operation:**
```
AWS Context:
   Profile: synthcoreai-dev
   Account ID: 123456789012
   Region: us-east-1
```

## Workflow Summary

1. **Verify environment** - Check MCP connection and display AWS context
2. **Create on aws-dev** - Generate code, explain, security scan, create
3. **Estimate costs** - Show breakdown, get approval if >= $10/month
4. **Track promotion** - Tag resources with PromotionStatus
5. **Promote to aws-prod** - After testing, replicate to production
6. **Update status** - Mark dev resource as "synced"

## References

Detailed guidance available in `references/`:
- `workflow.md` - Complete resource creation workflow with code examples
- `services.md` - S3, SES, RDS, Lambda, Cognito patterns and costs
- `security.md` - IAM policies, secrets management, tagging standards
- `troubleshooting.md` - Connection issues, drift detection, common errors

## Related Resources

- **MCP Config:** `.mcp.json` (aws-dev and aws-prod servers)
- **AWS Setup:** `scripts/aws-setup/`
- **AWS Tags:** Query via `list_resources()` and `get_resource()`
