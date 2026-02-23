---
name: aws-cost-operations
description: Use when analyzing AWS costs, reviewing billing, optimizing spending, or estimating deployment costs. Also for CloudWatch monitoring, CloudTrail auditing, and security assessments.
metadata:
  author: DDD
  version: 1.0.0
  category: aws
---

# AWS Cost & Operations

Manage AWS costs, monitor infrastructure, and maintain security posture through integrated MCP servers. This skill provides real-time access to billing data, cost estimation, observability metrics, and security assessments.

## Integrated MCP Servers

### Cost Management
- **AWS Billing and Cost Management** - Current spending, budgets, billing details
- **AWS Pricing** - Pre-deployment cost estimation, regional pricing comparison
- **AWS Cost Explorer** - Historical analysis, cost anomalies, optimization recommendations

### Monitoring & Observability
- **Amazon CloudWatch** - Metrics, alarms, logs, dashboards
- **CloudWatch Application Signals** - APM, SLOs, distributed tracing
- **AWS Managed Prometheus** - Container/Kubernetes metrics, PromQL queries

### Audit & Security
- **AWS CloudTrail** - API activity audit, change tracking, incident investigation
- **Well-Architected Security Assessment** - Security posture, compliance, recommendations

## When to Use

**Cost Analysis**
- Estimate costs before deploying new infrastructure
- Investigate unexpected spending increases
- Review monthly costs and optimize resources
- Compare pricing across regions

**Monitoring**
- Set up or query CloudWatch alarms and metrics
- Troubleshoot application performance issues
- Analyze logs for errors or patterns
- Monitor container workloads

**Security & Audit**
- Track who made changes to AWS resources
- Investigate security incidents
- Assess security posture against best practices
- Audit compliance requirements

## References

- `references/mcp-servers.md` - Detailed MCP server capabilities and use cases
- `references/operations-patterns.md` - Cost, monitoring, and security workflows
- `references/cloudwatch-alarms.md` - Common alarm configurations (CDK examples)
