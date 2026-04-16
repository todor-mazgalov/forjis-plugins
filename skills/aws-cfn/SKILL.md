---
name: aws-cfn
description: Guide for developing AWS CloudFormation templates following best practices for security, reliability, cost optimization, and operational excellence. Use when creating, reviewing, or refactoring CloudFormation/IaC templates for AWS infrastructure.
---

# AWS CloudFormation Development Guide

Develop production-grade AWS CloudFormation templates that are secure, maintainable, and aligned with the AWS Well-Architected Framework.

---

# Process

## Phase 1: Requirements Gathering

Before writing any template, clarify the following with the user:

1. **Target environment**: dev / staging / production
2. **AWS region(s)**: single-region or multi-region
3. **Resources needed**: compute, storage, networking, databases, serverless, etc.
4. **Compliance requirements**: SOC2, HIPAA, PCI-DSS, FedRAMP, or none
5. **Cost constraints**: any budget limits or instance-size preferences
6. **Existing infrastructure**: VPCs, accounts, shared services already in place
7. **Deployment strategy**: CI/CD pipeline, manual, or CloudFormation StackSets

If the user provides a high-level description (e.g., "I need a three-tier web app"), ask targeted follow-up questions before generating templates.

---

## Phase 2: Template Design

### 2.1 Choose Template Format

- **YAML** is the default — it supports comments and is more readable
- Use **JSON** only when the user explicitly requests it or tooling requires it

### 2.2 Structure the Template

Follow this canonical section order:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: >-
  Clear, one-line description of what this stack provisions.

Metadata:
  # Template metadata, interface labels

Parameters:
  # Inputs with AllowedValues, constraints, defaults

Mappings:
  # Region/environment lookups

Conditions:
  # Conditional resource creation

Rules:
  # Cross-parameter validation

Resources:
  # AWS resources (REQUIRED section)

Outputs:
  # Exported values, cross-stack references
```

### 2.3 Modular Stack Design

Split large infrastructures into nested or cross-referenced stacks:

| Stack | Responsibility |
|-------|---------------|
| `network.yaml` | VPC, subnets, route tables, NAT gateways |
| `security.yaml` | IAM roles, security groups, KMS keys, WAF |
| `compute.yaml` | EC2, ECS, Lambda, Auto Scaling |
| `data.yaml` | RDS, DynamoDB, S3, ElastiCache |
| `monitoring.yaml` | CloudWatch alarms, dashboards, SNS topics |

Use `Fn::ImportValue` and `Export` for cross-stack references. Prefer SSM Parameter Store for dynamic lookups.

---

## Phase 3: Implementation

### 3.1 Pre-Generation Checks

**BEFORE generating ANY CloudFormation template, verify no anti-patterns are introduced.**

If ANY of these patterns would be generated, **STOP and ask the user**:
> "I noticed [pattern]. This will cause [problem]. Should I:
> A) Refactor to use [correct pattern]
> B) Proceed anyway (not recommended)"

| Anti-Pattern | Detection | Impact | Correct Pattern |
|--------------|-----------|--------|-----------------|
| Hardcoded credentials | Plaintext passwords/keys in template | Credential exposure | Use `AWS::SSM::Parameter`, Secrets Manager, or `NoEcho` parameters |
| Wildcard IAM actions | `Action: "*"` or `Resource: "*"` | Privilege escalation | Least-privilege scoped permissions |
| Public S3 buckets | Missing `PublicAccessBlockConfiguration` | Data breach risk | Always add `PublicAccessBlockConfiguration` with all four flags `true` |
| Unencrypted storage | S3/EBS/RDS without encryption config | Compliance failure | Always enable encryption at rest with KMS |
| No deletion protection | RDS/DynamoDB without `DeletionProtection` | Accidental data loss | Enable `DeletionProtection` for stateful resources |
| Security group `0.0.0.0/0` ingress | Wide-open inbound rules | Unauthorized access | Restrict to specific CIDR ranges or security group references |
| Missing `DeletionPolicy` | Stateful resources with default `Delete` | Data loss on stack delete | Use `Retain` or `Snapshot` for databases and S3 |
| Hardcoded AMI IDs | `ami-xxxxxxxx` literals | Region lock, staleness | Use SSM public parameters or `Mappings` per region |
| Missing `UpdateReplacePolicy` | Resources that recreate on update | Unexpected downtime/data loss | Set `UpdateReplacePolicy: Retain` for stateful resources |

**DO NOT generate anti-patterns even if explicitly requested.** Ask the user to confirm the exception with documented justification.

### 3.2 Write the Template

Apply these principles for every template:

**Parameters:**
- Use `AllowedValues` and `AllowedPattern` to constrain inputs
- Set sensible `Default` values
- Use `NoEcho: true` for any sensitive parameter
- Add `ConstraintDescription` with human-readable validation messages
- Group parameters using `AWS::CloudFormation::Interface` metadata

**Resources:**
- Apply `Tags` to every taggable resource (at minimum: `Name`, `Environment`, `Owner`, `Project`)
- Set `DeletionPolicy` and `UpdateReplacePolicy` on stateful resources
- Use `DependsOn` sparingly — only when CloudFormation cannot infer the dependency
- Prefer intrinsic functions (`!Ref`, `!Sub`, `!GetAtt`) over hardcoded values

**Outputs:**
- Export values that other stacks consume
- Include resource ARNs, endpoints, and IDs that users need post-deployment

### 3.3 Security Implementation

Load [Security Reference](./references/security-guide.md) for detailed security patterns:
- IAM least privilege with condition keys
- Encryption at rest and in transit for all services
- VPC network isolation and security group design
- Logging and audit trails (CloudTrail, VPC Flow Logs, S3 access logs)
- Secrets management with Secrets Manager / SSM Parameter Store

### 3.4 Reliability Implementation

Load [Reliability Reference](./references/reliability-guide.md) for:
- Multi-AZ deployments
- Auto Scaling configuration
- Backup and disaster recovery patterns
- Health checks and self-healing

### 3.5 Cost Optimization

Load [Cost Reference](./references/cost-guide.md) for:
- Right-sizing instance types
- Spot/Reserved instance strategies
- Resource scheduling for non-production
- S3 lifecycle policies

---

## Phase 4: Validation and Review

### 4.1 Template Validation

After generating a template, always validate:

```bash
# Syntax validation
aws cloudformation validate-template --template-body file://template.yaml

# Linting with cfn-lint
cfn-lint template.yaml

# Security scanning with cfn-nag
cfn_nag_scan --input-path template.yaml

# Policy validation with cfn-guard
cfn-guard validate -d template.yaml -r rules.guard
```

### 4.2 Review Checklist

Load [Review Checklist](./references/review-checklist.md) and verify:
- All security checks pass
- Parameters are properly constrained
- Tags are applied to all taggable resources
- Stateful resources have deletion protection
- No hardcoded values that should be parameters
- Cross-stack references use exports properly
- Template description is clear and accurate

### 4.3 Scoring System (100 Points)

| Category | Points | Focus |
|----------|--------|-------|
| Security | 30 | IAM least privilege, encryption, network isolation, secrets management |
| Reliability | 20 | Multi-AZ, backups, deletion protection, health checks |
| Maintainability | 20 | Parameterization, modularity, naming, descriptions |
| Cost Optimization | 15 | Right-sizing, lifecycle policies, scheduling |
| Operational Excellence | 15 | Tagging, monitoring, logging, outputs |

**Thresholds**: 90+ Excellent | 80-89 Good | 70-79 Needs Improvement | Block: <60

---

## Phase 5: Deployment Guidance

After template review, provide:

1. **Pre-deployment checklist**: IAM permissions needed, prerequisite stacks
2. **Deploy command**:
   ```bash
   aws cloudformation deploy \
     --template-file template.yaml \
     --stack-name my-stack \
     --parameter-overrides Env=prod \
     --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
     --tags Key=Project,Value=MyApp
   ```
3. **Post-deployment validation**: outputs to check, smoke tests to run
4. **Rollback plan**: what to do if deployment fails

---

# Reference Files

Load these as needed during development:

### Core References
- [Security Guide](./references/security-guide.md) — IAM, encryption, network security, secrets management
- [Reliability Guide](./references/reliability-guide.md) — Multi-AZ, Auto Scaling, backups, disaster recovery
- [Cost Guide](./references/cost-guide.md) — Right-sizing, scheduling, lifecycle policies
- [Best Practices](./references/best-practices.md) — Template structure, naming, parameterization
- [Anti-Patterns](./references/anti-patterns.md) — Common mistakes and how to avoid them
- [Review Checklist](./references/review-checklist.md) — Pre-deployment validation checklist

### Patterns
- [Resource Patterns](./references/patterns.md) — Secure snippets for compute, database, storage, networking, messaging, and security resources
