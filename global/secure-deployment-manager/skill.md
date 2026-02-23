---
name: secure-deployment-manager
description: Use when deploying to AWS Amplify, debugging 401/403 auth errors, troubleshooting deployment failures, or investigating JWT/cookie issues in Next.js 15 + Cognito environments.
metadata:
  author: DDD
  version: 1.1.0
  category: deployment
---

# Secure Deployment Manager

Deploy code changes to AWS Amplify systematically using evidence-based debugging and incremental deployment. This skill prevents hours of trial-and-error by providing structured root cause analysis for Next.js 15 + AWS Cognito + Amplify environments.

## When to Use

- Deploying new features to AWS Amplify
- Fixing production 401/403 authentication errors
- Troubleshooting "works locally, fails on Amplify" issues
- Investigating deployment failures or build errors
- Debugging JWT/cookie authentication flows
- Analyzing Next.js middleware vs API route behavior
- Fixing database lookup mismatches (cognitoId vs id)

## Core Philosophy

1. **Diagnose Before Fix** - Never jump to solutions without understanding root cause. Use logs, diagnostics, and debug endpoints to gather evidence.

2. **Incremental Deployment** - One logical change per deployment. Monitor each to completion and verify functionality before proceeding.

3. **Security-First** - Authentication issues are architecture problems. Understand token flow completely. Never bypass security for convenience.

## Key Commands

```bash
# List recent deployments
aws amplify list-jobs \
  --app-id <APP_ID> \
  --branch-name <BRANCH> \
  --max-items 5 \
  --region us-east-1 \
  --profile <PROFILE>

# Get job details
aws amplify get-job \
  --app-id <APP_ID> \
  --branch-name <BRANCH> \
  --job-id <ID> \
  --query 'job.{summary:summary,steps:steps[*].{stepName:stepName,status:status}}'

# Monitor job completion (background)
sleep 480 && aws amplify get-job \
  --app-id <APP_ID> \
  --branch-name <BRANCH> \
  --job-id <ID> \
  --query 'job.summary.{JobId:jobId,Status:status,EndTime:endTime}'
```

## References

- `references/deployment-workflow.md` - Complete 6-phase deployment process
- `references/auth-troubleshooting.md` - Authentication playbook for Next.js + Cognito
- `references/debugging-checklist.md` - Systematic debugging checklist
- `references/cloudwatch-alarms.md` - CloudWatch alarm configurations
- `references/operations-patterns.md` - AWS cost and operations patterns
