<!-- Parent: aws-cfn/SKILL.md -->
# CloudFormation Review Checklist

Pre-deployment validation checklist for CloudFormation templates. Score each section and block deployment if total falls below 60/100.

---

## Security (30 points)

| # | Check | Points | Pass? |
|---|-------|--------|-------|
| S1 | No plaintext credentials or secrets in template | 5 | |
| S2 | All IAM policies follow least privilege (no `*` actions/resources) | 5 | |
| S3 | All S3 buckets have `PublicAccessBlockConfiguration` with all four flags `true` | 4 | |
| S4 | All storage resources encrypted at rest (S3, RDS, EBS, DynamoDB, EFS) | 4 | |
| S5 | KMS keys have `EnableKeyRotation: true` | 2 | |
| S6 | No `0.0.0.0/0` ingress on non-80/443 ports | 3 | |
| S7 | Sensitive parameters use `NoEcho: true` | 2 | |
| S8 | S3 bucket policies enforce TLS (`aws:SecureTransport`) | 2 | |
| S9 | Security groups reference other SGs instead of CIDRs where possible | 2 | |
| S10 | Lambda functions use VPC config when accessing private resources | 1 | |

---

## Reliability (20 points)

| # | Check | Points | Pass? |
|---|-------|--------|-------|
| R1 | Production databases deployed Multi-AZ | 4 | |
| R2 | Stateful resources have `DeletionPolicy: Snapshot` or `Retain` | 4 | |
| R3 | Stateful resources have `UpdateReplacePolicy: Snapshot` or `Retain` | 2 | |
| R4 | RDS has `DeletionProtection: true` | 2 | |
| R5 | RDS `BackupRetentionPeriod > 0` | 2 | |
| R6 | Load balancer health checks configured with appropriate thresholds | 2 | |
| R7 | Auto Scaling configured for production compute | 2 | |
| R8 | Resources distributed across at least 2 AZs | 2 | |

---

## Maintainability (20 points)

| # | Check | Points | Pass? |
|---|-------|--------|-------|
| M1 | No hardcoded values that should be parameters (AMI IDs, instance types, CIDRs) | 4 | |
| M2 | Logical IDs are descriptive PascalCase names | 2 | |
| M3 | Template has a clear `Description` | 2 | |
| M4 | Parameters have `AllowedValues`/`AllowedPattern` constraints | 3 | |
| M5 | Parameters have `ConstraintDescription` messages | 1 | |
| M6 | Parameters grouped with `AWS::CloudFormation::Interface` metadata | 2 | |
| M7 | Uses `!Sub` instead of `!Join` for string interpolation | 1 | |
| M8 | Physical resource names use `!Sub` with stack name (not hardcoded) | 2 | |
| M9 | Template under 500 resources (split into stacks if larger) | 2 | |
| M10 | `DependsOn` only used when CloudFormation cannot infer dependency | 1 | |

---

## Cost Optimization (15 points)

| # | Check | Points | Pass? |
|---|-------|--------|-------|
| C1 | Instance types parameterized per environment | 3 | |
| C2 | Non-production environments use smaller instance types | 2 | |
| C3 | S3 buckets have lifecycle rules for log/temp data | 2 | |
| C4 | VPC endpoints used for S3 and DynamoDB (gateway endpoints are free) | 2 | |
| C5 | EBS volumes use gp3 instead of gp2 | 2 | |
| C6 | Graviton/ARM instances considered for compatible workloads | 2 | |
| C7 | Non-production resources have scheduling (scale down off-hours) | 2 | |

---

## Operational Excellence (15 points)

| # | Check | Points | Pass? |
|---|-------|--------|-------|
| O1 | All taggable resources have required tags (Name, Environment, Owner, Project, ManagedBy) | 4 | |
| O2 | CloudWatch alarms defined for critical metrics | 3 | |
| O3 | Logging enabled (VPC Flow Logs, S3 access logs, ALB access logs, CloudTrail) | 3 | |
| O4 | Outputs include key resource identifiers (ARNs, endpoints, IDs) | 2 | |
| O5 | Outputs export values needed by downstream stacks | 2 | |
| O6 | Template validated with `cfn-lint` and `cfn-nag` | 1 | |

---

## Scoring

| Score | Rating | Action |
|-------|--------|--------|
| 90-100 | Excellent | Deploy |
| 80-89 | Good | Deploy with notes |
| 70-79 | Needs Improvement | Fix before deploying |
| 60-69 | Poor | Significant rework needed |
| <60 | **BLOCKED** | Do not deploy |

---

## Validation Commands

Run these before deployment:

```bash
# Syntax check
aws cloudformation validate-template --template-body file://template.yaml

# Lint
cfn-lint template.yaml

# Security scan
cfn_nag_scan --input-path template.yaml

# Policy-as-code validation
cfn-guard validate -d template.yaml -r rules.guard
```
