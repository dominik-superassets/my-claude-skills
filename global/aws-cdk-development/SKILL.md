---
name: aws-cdk-development
description: Use when creating CDK stacks, defining constructs, implementing infrastructure as code, or deploying with cdk synth/deploy. Covers CDK app structure, construct patterns, stack composition, and deployment workflows.
metadata:
  author: DDD
  version: 1.0.0
  category: aws
---

# AWS CDK Development

Build AWS infrastructure using the Cloud Development Kit (CDK) with TypeScript or Python. This skill provides patterns, validation workflows, and integration with MCP servers for up-to-date AWS guidance.

## When to Use

- Creating new CDK stacks or constructs
- Refactoring existing CDK infrastructure
- Validating stacks before deployment
- Implementing Lambda functions within CDK
- Checking AWS service availability and features

## Core Principle: Let CDK Generate Names

**CRITICAL**: Do NOT explicitly specify resource names when optional.

CDK-generated names enable:
- **Reusable patterns** - Deploy the same construct multiple times without conflicts
- **Parallel deployments** - Multiple stacks can deploy simultaneously
- **Stack isolation** - Each stack gets uniquely identified resources

For different environments (dev, staging, prod), use separate AWS accounts rather than naming conventions for stronger security boundaries.

## Integrated MCP Servers

### AWS Documentation MCP
Always verify before implementing:
- Service availability in target regions
- Latest features and API specifications
- Service limits and quotas

### AWS CDK MCP
Use for CDK-specific guidance:
- Construct recommendations
- Best practice patterns
- Configuration options

## Validation Strategy

1. **cdk-nag** - Synthesis-time security and best practice checks
2. **cdk synth** - Validate CloudFormation generation
3. **Unit tests** - Test construct behavior with assertions

## References

- [CDK Patterns and Best Practices](references/cdk-patterns.md) - Detailed patterns, anti-patterns, security, cost optimization
- [Validation Script](scripts/validate-stack.sh) - Pre-deployment checks
