# MCP Servers Reference

Detailed information about the 8 MCP servers integrated with this skill.

## Cost Management Servers

### 1. AWS Billing and Cost Management MCP Server

**Purpose**: Real-time billing and cost management

**Capabilities**:
- View current AWS spending and trends
- Analyze billing details across services
- Track budget utilization
- Monitor cost allocation tags
- Review consolidated billing for organizations

**Use Cases**:
- "What's my current AWS spend this month?"
- "Which services are costing the most?"
- "How much of my budget have I used?"

### 2. AWS Pricing MCP Server

**Purpose**: Pre-deployment cost estimation and optimization

**Capabilities**:
- Estimate costs before deploying resources
- Compare pricing across regions
- Calculate Total Cost of Ownership (TCO)
- Evaluate different service options for cost efficiency
- Get current pricing information for AWS services

**Use Cases**:
- "Estimate the cost of running a Lambda function with 1M invocations"
- "Compare EC2 pricing between us-east-1 and eu-west-1"
- "What would it cost to run an RDS db.r5.large?"

### 3. AWS Cost Explorer MCP Server

**Purpose**: Detailed cost analysis and reporting

**Capabilities**:
- Analyze historical spending patterns
- Create custom cost reports
- Identify cost anomalies and trends
- Forecast future costs
- Analyze cost by service, region, or tag
- Generate cost optimization recommendations

**Use Cases**:
- "Show me spending trends for the last 3 months"
- "Identify cost anomalies in my account"
- "What are my cost optimization recommendations?"

## Monitoring & Observability Servers

### 4. Amazon CloudWatch MCP Server

**Purpose**: Metrics, alarms, and logs analysis

**Capabilities**:
- Query CloudWatch metrics and logs
- Create and manage CloudWatch alarms
- Analyze application performance metrics
- Troubleshoot operational issues
- Set up custom dashboards
- Monitor resource utilization

**Use Cases**:
- "Check the error rate for my Lambda function"
- "Show me CPU utilization for my EC2 instances"
- "Query logs for errors in the last hour"

### 5. Amazon CloudWatch Application Signals MCP Server

**Purpose**: Application monitoring and performance insights

**Capabilities**:
- Monitor application health and performance
- Analyze service-level objectives (SLOs)
- Track application dependencies
- Identify performance bottlenecks
- Monitor service map and traces

**Use Cases**:
- "Show me the service map for my application"
- "What's the p99 latency for my API?"
- "Identify slow downstream dependencies"

### 6. AWS Managed Prometheus MCP Server

**Purpose**: Prometheus-compatible monitoring

**Capabilities**:
- Query Prometheus metrics
- Monitor containerized applications
- Analyze Kubernetes workload metrics
- Create PromQL queries
- Track custom application metrics

**Use Cases**:
- "Query container CPU usage with PromQL"
- "Show me Kubernetes pod metrics"
- "Monitor my EKS cluster health"

## Audit & Security Servers

### 7. AWS CloudTrail MCP Server

**Purpose**: AWS API activity and audit analysis

**Capabilities**:
- Analyze AWS API calls and user activity
- Track resource changes and modifications
- Investigate security incidents
- Audit compliance requirements
- Identify unusual access patterns
- Review who made what changes when

**Use Cases**:
- "Who deleted this S3 bucket?"
- "Show all IAM changes in the last 24 hours"
- "List failed login attempts"
- "Find all actions by a specific user"

### 8. AWS Well-Architected Security Assessment Tool MCP Server

**Purpose**: Security assessment against Well-Architected Framework

**Capabilities**:
- Assess security posture against AWS best practices
- Identify security gaps and vulnerabilities
- Get security improvement recommendations
- Review security pillar compliance
- Generate security assessment reports

**Assessment Areas**:
- Identity and Access Management (IAM)
- Detective controls and monitoring
- Infrastructure protection
- Data protection and encryption
- Incident response preparedness

**Use Cases**:
- "Assess my account's security posture"
- "What security improvements should I make?"
- "Review my IAM configuration against best practices"

## Server Selection Guide

| Task | Primary Server | Supporting Servers |
|------|----------------|-------------------|
| Pre-deployment cost estimation | Pricing MCP | - |
| Current billing review | Billing MCP | Cost Explorer MCP |
| Historical cost analysis | Cost Explorer MCP | Billing MCP |
| Cost optimization | Cost Explorer MCP | CloudWatch MCP |
| Application monitoring | CloudWatch MCP | Application Signals MCP |
| Container/K8s monitoring | Prometheus MCP | CloudWatch MCP |
| Security audit | CloudTrail MCP | Well-Architected MCP |
| Security assessment | Well-Architected MCP | CloudTrail MCP |
| Troubleshooting | CloudWatch MCP | CloudTrail MCP |
