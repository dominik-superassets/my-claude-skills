---
name: aws-serverless-eda
description: Use when building AWS serverless applications, Lambda functions, API Gateway APIs, or event-driven architectures. Covers Lambda (TypeScript/Python), API Gateway, DynamoDB, Step Functions, EventBridge, SQS, SNS patterns.
metadata:
  author: DDD
  version: 1.0.0
  category: aws
---

# AWS Serverless & Event-Driven Architecture

Comprehensive guidance for building serverless applications and event-driven architectures on AWS based on Well-Architected Framework principles. This skill integrates MCP servers for SAM CLI, Lambda execution, Step Functions, and SNS/SQS messaging.

## When to Use This Skill

- Building serverless applications with Lambda
- Designing event-driven architectures with EventBridge, SQS, SNS
- Implementing microservices or async workflows
- Orchestrating multi-service transactions with Step Functions
- Creating real-time data processing pipelines
- Implementing saga patterns for distributed transactions

## Core Principles (AWS Well-Architected)

**Speedy, Simple, Singular**: Functions should be concise, single-purpose, and environmentally efficient. Minimize cold starts, optimize memory, leverage connection reuse.

**Think Concurrent, Not Total**: Design for concurrent execution limits, downstream throttling, and shared resource contention rather than total request volume.

**Share Nothing**: Runtime environments are short-lived. Use DynamoDB for state, Step Functions for workflow state, S3 for files. Never rely on local filesystem persistence.

**Orchestrate with State Machines**: Use Step Functions instead of Lambda chaining for visual workflows, built-in error handling, and service integrations.

**Events Trigger Transactions**: Prefer event-driven over synchronous request/response for loose coupling, async processing, and independent scaling.

**Design for Failures**: All operations must be idempotent. Implement retry logic with exponential backoff and dead letter queues.

## Reference Documentation

| Reference | Content |
|-----------|---------|
| [Serverless Patterns](references/serverless-patterns.md) | API patterns, data processing, Step Functions orchestration |
| [EDA Patterns](references/eda-patterns.md) | Event routing, sourcing, saga patterns, idempotency |
| [Security](references/security-best-practices.md) | IAM least privilege, data protection, VPC security |
| [Observability](references/observability-best-practices.md) | Metrics, logs, traces, CloudWatch alarms |
| [Performance](references/performance-optimization.md) | Cold starts, memory optimization, provisioned concurrency |
| [Deployment](references/deployment-best-practices.md) | CI/CD, testing strategies, canary/blue-green deployments |

## External Resources

- [AWS Well-Architected Serverless Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/)
- [ServerlessLand.com](https://serverlessland.com) - Pre-built patterns
- [AWS Serverless Workshops](https://serverlessland.com/learn?type=Workshops)
