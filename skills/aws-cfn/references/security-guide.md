<!-- Parent: aws-cfn/SKILL.md -->
# CloudFormation Security Guide

Comprehensive security patterns for AWS CloudFormation templates covering IAM, encryption, network isolation, secrets management, and compliance.

---

## Table of Contents

1. [Generation Guardrails](#generation-guardrails)
2. [IAM Least Privilege](#iam-least-privilege)
3. [Encryption at Rest and in Transit](#encryption-at-rest-and-in-transit)
4. [Network Security](#network-security)
5. [Secrets Management](#secrets-management)
6. [Logging and Audit](#logging-and-audit)
7. [S3 Security](#s3-security)
8. [Lambda Security](#lambda-security)
9. [Security Checklist](#security-checklist)

---

## Generation Guardrails

### MANDATORY PRE-GENERATION CHECKS

**BEFORE generating ANY CloudFormation resource, verify these security requirements.**

| Resource Type | Required Security Config | If Missing |
|---------------|------------------------|------------|
| S3 Bucket | `PublicAccessBlockConfiguration`, `BucketEncryption`, `VersioningConfiguration` | STOP — ask user |
| RDS Instance | `StorageEncrypted`, `DeletionProtection`, `BackupRetentionPeriod > 0` | STOP — ask user |
| DynamoDB Table | `SSESpecification`, `DeletionProtectionEnabled`, `PointInTimeRecoverySpecification` | STOP — ask user |
| EC2 Instance | No `0.0.0.0/0` ingress, `EbsOptimized`, encrypted volumes | STOP — ask user |
| Lambda Function | VPC placement (if accessing private resources), reserved concurrency | Warn user |
| IAM Role | No `*` actions or resources, condition keys where applicable | STOP — ask user |
| Security Group | No `0.0.0.0/0` ingress on non-80/443 ports | STOP — ask user |
| ELB/ALB | HTTPS listeners, security policy, access logs | STOP — ask user |
| API Gateway | Authorization (Cognito/IAM/Lambda), throttling, WAF | Warn user |
| SNS/SQS | KMS encryption, access policy restrictions | Warn user |

---

## IAM Least Privilege

### Principle

Every IAM policy must grant the minimum permissions required. Never use wildcard actions or resources in production.

### BAD — Wildcard Permissions

```yaml
# BLOCKED: Never generate this
PolicyDocument:
  Statement:
    - Effect: Allow
      Action: "*"
      Resource: "*"
```

### GOOD — Scoped Permissions

```yaml
PolicyDocument:
  Statement:
    - Effect: Allow
      Action:
        - s3:GetObject
        - s3:PutObject
      Resource: !Sub "arn:aws:s3:::${MyBucket}/*"
    - Effect: Allow
      Action:
        - s3:ListBucket
      Resource: !Sub "arn:aws:s3:::${MyBucket}"
```

### IAM Condition Keys

Use condition keys to further restrict access:

```yaml
PolicyDocument:
  Statement:
    - Effect: Allow
      Action:
        - s3:PutObject
      Resource: !Sub "arn:aws:s3:::${MyBucket}/*"
      Condition:
        StringEquals:
          s3:x-amz-server-side-encryption: "aws:kms"
        ArnLike:
          aws:SourceArn: !Sub "arn:aws:service:::${AWS::AccountId}"
```

### Service-Linked Roles

Prefer AWS-managed service-linked roles over custom roles when the service supports them. They are automatically scoped and maintained by AWS.

### Permission Boundaries

For environments with multiple teams, use permission boundaries:

```yaml
DeveloperRole:
  Type: AWS::IAM::Role
  Properties:
    PermissionsBoundary: !Sub "arn:aws:iam::${AWS::AccountId}:policy/DeveloperBoundary"
    AssumeRolePolicyDocument:
      Statement:
        - Effect: Allow
          Principal:
            Service: lambda.amazonaws.com
          Action: sts:AssumeRole
```

---

## Encryption at Rest and in Transit

### S3 Encryption

```yaml
MyBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: aws:kms
            KMSMasterKeyID: !GetAtt MyKMSKey.Arn
          BucketKeyEnabled: true
```

### RDS Encryption

```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    StorageEncrypted: true
    KmsKeyId: !GetAtt MyKMSKey.Arn
```

### EBS Encryption

```yaml
MyVolume:
  Type: AWS::EC2::Volume
  Properties:
    Encrypted: true
    KmsKeyId: !GetAtt MyKMSKey.Arn
```

### DynamoDB Encryption

```yaml
MyTable:
  Type: AWS::DynamoDB::Table
  Properties:
    SSESpecification:
      SSEEnabled: true
      SSEType: KMS
      KMSMasterKeyId: !GetAtt MyKMSKey.Arn
```

### KMS Key Pattern

```yaml
MyKMSKey:
  Type: AWS::KMS::Key
  Properties:
    Description: Encryption key for application data
    EnableKeyRotation: true
    KeyPolicy:
      Statement:
        - Sid: AllowKeyAdministration
          Effect: Allow
          Principal:
            AWS: !Sub "arn:aws:iam::${AWS::AccountId}:root"
          Action: "kms:*"
          Resource: "*"
        - Sid: AllowKeyUsage
          Effect: Allow
          Principal:
            AWS: !GetAtt AppRole.Arn
          Action:
            - kms:Encrypt
            - kms:Decrypt
            - kms:GenerateDataKey
          Resource: "*"

MyKMSKeyAlias:
  Type: AWS::KMS::Alias
  Properties:
    AliasName: !Sub "alias/${AWS::StackName}-key"
    TargetKeyId: !Ref MyKMSKey
```

### Enforce TLS in Transit

```yaml
BucketPolicy:
  Type: AWS::S3::BucketPolicy
  Properties:
    Bucket: !Ref MyBucket
    PolicyDocument:
      Statement:
        - Sid: DenyNonSSL
          Effect: Deny
          Principal: "*"
          Action: s3:*
          Resource:
            - !Sub "arn:aws:s3:::${MyBucket}"
            - !Sub "arn:aws:s3:::${MyBucket}/*"
          Condition:
            Bool:
              aws:SecureTransport: "false"
```

---

## Network Security

### VPC Design

- Place workloads in **private subnets** by default
- Use **public subnets** only for load balancers and NAT gateways
- Deploy across **at least 2 Availability Zones**

### Security Group Rules

```yaml
# GOOD: Restricted ingress
AppSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Application tier security group
    VpcId: !Ref VPC
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 443
        ToPort: 443
        SourceSecurityGroupId: !Ref ALBSecurityGroup
    SecurityGroupEgress:
      - IpProtocol: tcp
        FromPort: 443
        ToPort: 443
        CidrIp: 0.0.0.0/0
```

### BAD — Wide-open Security Group

```yaml
# BLOCKED: Never generate 0.0.0.0/0 on non-standard ports
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 22
    ToPort: 22
    CidrIp: 0.0.0.0/0
```

### Network ACLs

Use NACLs as an additional defense layer for sensitive subnets:

```yaml
PrivateSubnetNACL:
  Type: AWS::EC2::NetworkAcl
  Properties:
    VpcId: !Ref VPC

PrivateInboundRule:
  Type: AWS::EC2::NetworkAclEntry
  Properties:
    NetworkAclId: !Ref PrivateSubnetNACL
    RuleNumber: 100
    Protocol: 6
    RuleAction: allow
    CidrBlock: !GetAtt VPC.CidrBlock
    PortRange:
      From: 443
      To: 443
```

### VPC Flow Logs

Always enable VPC Flow Logs for auditing:

```yaml
VPCFlowLog:
  Type: AWS::EC2::FlowLog
  Properties:
    ResourceId: !Ref VPC
    ResourceType: VPC
    TrafficType: ALL
    LogDestinationType: cloud-watch-logs
    LogGroupName: !Ref FlowLogGroup
    DeliverLogsPermissionArn: !GetAtt FlowLogRole.Arn
```

---

## Secrets Management

### Never Hardcode Secrets

```yaml
# BLOCKED: Never put secrets in templates
Resources:
  MyDB:
    Type: AWS::RDS::DBInstance
    Properties:
      MasterUserPassword: "MyP@ssw0rd123"  # NEVER DO THIS
```

### Use Secrets Manager

```yaml
DBSecret:
  Type: AWS::SecretsManager::Secret
  Properties:
    GenerateSecretString:
      SecretStringTemplate: '{"username": "admin"}'
      GenerateStringKey: "password"
      PasswordLength: 32
      ExcludeCharacters: '"@/\'

MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    MasterUsername: !Sub "{{resolve:secretsmanager:${DBSecret}:SecretString:username}}"
    MasterUserPassword: !Sub "{{resolve:secretsmanager:${DBSecret}:SecretString:password}}"
```

### Use SSM Parameter Store for Config

```yaml
Parameters:
  DatabaseHost:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /myapp/prod/db-host
```

### NoEcho for Sensitive Parameters

```yaml
Parameters:
  APIKey:
    Type: String
    NoEcho: true
    Description: Third-party API key
    MinLength: 20
```

---

## Logging and Audit

### CloudTrail

```yaml
Trail:
  Type: AWS::CloudTrail::Trail
  Properties:
    IsLogging: true
    S3BucketName: !Ref AuditBucket
    EnableLogFileValidation: true
    IsMultiRegionTrail: true
    IncludeGlobalServiceEvents: true
    CloudWatchLogsLogGroupArn: !GetAtt TrailLogGroup.Arn
    CloudWatchLogsRoleArn: !GetAtt TrailRole.Arn
    KMSKeyId: !GetAtt AuditKMSKey.Arn
```

### S3 Access Logs

```yaml
MyBucket:
  Type: AWS::S3::Bucket
  Properties:
    LoggingConfiguration:
      DestinationBucketName: !Ref AccessLogBucket
      LogFilePrefix: !Sub "${AWS::StackName}/"
```

### ALB Access Logs

```yaml
MyALB:
  Type: AWS::ElasticLoadBalancingV2::LoadBalancer
  Properties:
    LoadBalancerAttributes:
      - Key: access_logs.s3.enabled
        Value: "true"
      - Key: access_logs.s3.bucket
        Value: !Ref ALBLogBucket
```

---

## S3 Security

### Complete Secure Bucket

```yaml
SecureBucket:
  Type: AWS::S3::Bucket
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain
  Properties:
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault:
            SSEAlgorithm: aws:kms
            KMSMasterKeyID: !GetAtt MyKMSKey.Arn
          BucketKeyEnabled: true
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
    VersioningConfiguration:
      Status: Enabled
    LoggingConfiguration:
      DestinationBucketName: !Ref AccessLogBucket
    LifecycleConfiguration:
      Rules:
        - Id: TransitionToIA
          Status: Enabled
          Transitions:
            - StorageClass: STANDARD_IA
              TransitionInDays: 90
    ObjectLockEnabled: true
```

---

## Lambda Security

### Secure Lambda Pattern

```yaml
MyFunction:
  Type: AWS::Lambda::Function
  Properties:
    Runtime: python3.12
    Handler: index.handler
    Code:
      S3Bucket: !Ref DeploymentBucket
      S3Key: lambda/function.zip
    Role: !GetAtt FunctionRole.Arn
    VpcConfig:
      SecurityGroupIds:
        - !Ref LambdaSecurityGroup
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
    ReservedConcurrentExecutions: 100
    Environment:
      Variables:
        SECRET_ARN: !Ref MySecret
    TracingConfig:
      Mode: Active
```

### Lambda Resource Policy

```yaml
LambdaPermission:
  Type: AWS::Lambda::Permission
  Properties:
    FunctionName: !Ref MyFunction
    Action: lambda:InvokeFunction
    Principal: apigateway.amazonaws.com
    SourceArn: !Sub "arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${MyApi}/*"
```

---

## Security Checklist

| # | Check | Category |
|---|-------|----------|
| 1 | No plaintext secrets in template | Secrets |
| 2 | All IAM policies use least privilege | IAM |
| 3 | No wildcard `*` in IAM actions or resources | IAM |
| 4 | Permission boundaries applied where applicable | IAM |
| 5 | All S3 buckets block public access | S3 |
| 6 | All S3 buckets have encryption enabled | Encryption |
| 7 | All S3 buckets enforce TLS via bucket policy | Encryption |
| 8 | All RDS instances have `StorageEncrypted: true` | Encryption |
| 9 | All EBS volumes are encrypted | Encryption |
| 10 | KMS keys have rotation enabled | Encryption |
| 11 | No `0.0.0.0/0` ingress on non-80/443 ports | Network |
| 12 | Workloads in private subnets | Network |
| 13 | VPC Flow Logs enabled | Logging |
| 14 | CloudTrail enabled with log validation | Logging |
| 15 | ALB/S3 access logs enabled | Logging |
| 16 | Security groups use SG references, not CIDRs, where possible | Network |
| 17 | Sensitive parameters use `NoEcho: true` | Secrets |
| 18 | Secrets Manager used for database credentials | Secrets |
| 19 | Lambda functions use VPC when accessing private resources | Network |
| 20 | API Gateway has authorization configured | API |
