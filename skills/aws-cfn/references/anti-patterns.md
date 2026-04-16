<!-- Parent: aws-cfn/SKILL.md -->
# CloudFormation Anti-Patterns

Common mistakes in CloudFormation templates and their correct alternatives.

---

## Table of Contents

1. [Security Anti-Patterns](#security-anti-patterns)
2. [Reliability Anti-Patterns](#reliability-anti-patterns)
3. [Maintainability Anti-Patterns](#maintainability-anti-patterns)
4. [Cost Anti-Patterns](#cost-anti-patterns)
5. [Operational Anti-Patterns](#operational-anti-patterns)

---

## Security Anti-Patterns

### 1. Hardcoded Credentials

**BAD:**
```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    MasterUsername: admin
    MasterUserPassword: SuperSecret123!
```

**GOOD:**
```yaml
DBSecret:
  Type: AWS::SecretsManager::Secret
  Properties:
    GenerateSecretString:
      SecretStringTemplate: '{"username": "admin"}'
      GenerateStringKey: password
      PasswordLength: 32

MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    MasterUsername: !Sub "{{resolve:secretsmanager:${DBSecret}:SecretString:username}}"
    MasterUserPassword: !Sub "{{resolve:secretsmanager:${DBSecret}:SecretString:password}}"
```

### 2. Wildcard IAM Permissions

**BAD:**
```yaml
AppPolicy:
  Type: AWS::IAM::Policy
  Properties:
    PolicyDocument:
      Statement:
        - Effect: Allow
          Action: "*"
          Resource: "*"
```

**GOOD:**
```yaml
AppPolicy:
  Type: AWS::IAM::Policy
  Properties:
    PolicyDocument:
      Statement:
        - Effect: Allow
          Action:
            - dynamodb:GetItem
            - dynamodb:PutItem
            - dynamodb:Query
          Resource: !GetAtt MyTable.Arn
```

### 3. Public S3 Buckets

**BAD:**
```yaml
MyBucket:
  Type: AWS::S3::Bucket
  # Missing PublicAccessBlockConfiguration
```

**GOOD:**
```yaml
MyBucket:
  Type: AWS::S3::Bucket
  Properties:
    PublicAccessBlockConfiguration:
      BlockPublicAcls: true
      BlockPublicPolicy: true
      IgnorePublicAcls: true
      RestrictPublicBuckets: true
```

### 4. Wide-Open Security Groups

**BAD:**
```yaml
SecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: 0.0.0.0/0
```

**GOOD:**
```yaml
SecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        SourceSecurityGroupId: !Ref BastionSecurityGroup
```

### 5. Unencrypted Storage

**BAD:**
```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    Engine: postgres
    # Missing StorageEncrypted
```

**GOOD:**
```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    Engine: postgres
    StorageEncrypted: true
    KmsKeyId: !GetAtt MyKMSKey.Arn
```

---

## Reliability Anti-Patterns

### 6. Missing Deletion Policy

**BAD:**
```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  # Default DeletionPolicy is Delete — data lost on stack delete
```

**GOOD:**
```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  DeletionPolicy: Snapshot
  UpdateReplacePolicy: Snapshot
  Properties:
    DeletionProtection: true
```

### 7. Single-AZ Production Deployments

**BAD:**
```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    MultiAZ: false  # Single point of failure
```

**GOOD:**
```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    MultiAZ: true
```

### 8. No Health Checks

**BAD:**
```yaml
TargetGroup:
  Type: AWS::ElasticLoadBalancingV2::TargetGroup
  Properties:
    Port: 8080
    Protocol: HTTP
    # No health check — unhealthy targets receive traffic
```

**GOOD:**
```yaml
TargetGroup:
  Type: AWS::ElasticLoadBalancingV2::TargetGroup
  Properties:
    Port: 8080
    Protocol: HTTP
    HealthCheckPath: /health
    HealthCheckIntervalSeconds: 30
    HealthyThresholdCount: 2
    UnhealthyThresholdCount: 3
```

### 9. No Backup Configuration

**BAD:**
```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    BackupRetentionPeriod: 0  # Backups disabled
```

**GOOD:**
```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    BackupRetentionPeriod: 30
    PreferredBackupWindow: "03:00-04:00"
    CopyTagsToSnapshot: true
```

---

## Maintainability Anti-Patterns

### 10. Hardcoded Values

**BAD:**
```yaml
MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    InstanceType: m5.large
    ImageId: ami-0abcdef1234567890
    SubnetId: subnet-0123456789abcdef0
```

**GOOD:**
```yaml
Parameters:
  InstanceType:
    Type: String
    Default: m5.large
    AllowedValues: [t3.medium, m5.large, m5.xlarge]
  AMIId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64

MyInstance:
  Type: AWS::EC2::Instance
  Properties:
    InstanceType: !Ref InstanceType
    ImageId: !Ref AMIId
    SubnetId: !ImportValue NetworkStack-PrivateSubnet1Id
```

### 11. Cryptic Logical IDs

**BAD:**
```yaml
Resources:
  R1:
    Type: AWS::EC2::SecurityGroup
  SN1:
    Type: AWS::EC2::Subnet
  DB:
    Type: AWS::RDS::DBInstance
```

**GOOD:**
```yaml
Resources:
  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
  PrivateSubnet1:
    Type: AWS::EC2::Subnet
  PrimaryDatabase:
    Type: AWS::RDS::DBInstance
```

### 12. Monolithic Templates

**BAD:** A single 3000-line template with VPC, RDS, ECS, Lambda, CloudFront, and monitoring all in one file.

**GOOD:** Split into focused stacks:
- `network.yaml` — VPC, subnets, routes
- `security.yaml` — IAM, security groups, KMS
- `data.yaml` — RDS, DynamoDB, S3
- `compute.yaml` — ECS, Lambda
- `cdn.yaml` — CloudFront, WAF
- `monitoring.yaml` — CloudWatch, SNS

### 13. Using `!Join` Instead of `!Sub`

**BAD:**
```yaml
Value: !Join
  - ""
  - - "arn:aws:s3:::"
    - !Ref MyBucket
    - "/*"
```

**GOOD:**
```yaml
Value: !Sub "arn:aws:s3:::${MyBucket}/*"
```

---

## Cost Anti-Patterns

### 14. Over-Provisioned Non-Production

**BAD:** Using production-sized instances for dev/staging.

**GOOD:** Use conditions or mappings to size by environment:
```yaml
Mappings:
  EnvConfig:
    dev:
      InstanceType: t3.small
    prod:
      InstanceType: m6i.large
```

### 15. Missing Lifecycle Policies

**BAD:**
```yaml
LogBucket:
  Type: AWS::S3::Bucket
  # Logs accumulate forever
```

**GOOD:**
```yaml
LogBucket:
  Type: AWS::S3::Bucket
  Properties:
    LifecycleConfiguration:
      Rules:
        - Id: ExpireLogs
          Status: Enabled
          ExpirationInDays: 90
          Transitions:
            - StorageClass: GLACIER
              TransitionInDays: 30
```

### 16. No VPC Endpoints

**BAD:** All AWS service traffic routing through NAT Gateway (data processing charges).

**GOOD:** Add gateway endpoints for S3 and DynamoDB, interface endpoints for frequently used services.

---

## Operational Anti-Patterns

### 17. Missing Tags

**BAD:**
```yaml
MyBucket:
  Type: AWS::S3::Bucket
  # No tags — impossible to track ownership or cost
```

**GOOD:**
```yaml
MyBucket:
  Type: AWS::S3::Bucket
  Properties:
    Tags:
      - Key: Name
        Value: !Sub "${AWS::StackName}-data"
      - Key: Environment
        Value: !Ref Environment
      - Key: Owner
        Value: !Ref OwnerEmail
      - Key: Project
        Value: !Ref ProjectName
```

### 18. No Monitoring or Alarms

**BAD:** Resources deployed without any CloudWatch alarms.

**GOOD:**
```yaml
HighCPUAlarm:
  Type: AWS::CloudWatch::Alarm
  Properties:
    AlarmDescription: CPU utilization exceeds 80%
    MetricName: CPUUtilization
    Namespace: AWS/EC2
    Statistic: Average
    Period: 300
    EvaluationPeriods: 3
    Threshold: 80
    ComparisonOperator: GreaterThanThreshold
    AlarmActions:
      - !Ref AlertTopic
    Dimensions:
      - Name: AutoScalingGroupName
        Value: !Ref AutoScalingGroup
```

### 19. No Outputs

**BAD:** Template produces no outputs — users must dig through the Console to find resource identifiers.

**GOOD:** Output every value users or downstream stacks need: ARNs, endpoints, DNS names, security group IDs.

### 20. Unnecessary `DependsOn`

**BAD:**
```yaml
Instance:
  Type: AWS::EC2::Instance
  DependsOn:
    - SecurityGroup  # CloudFormation already infers this from !Ref
  Properties:
    SecurityGroupIds:
      - !Ref SecurityGroup
```

**GOOD:** Only use `DependsOn` when CloudFormation cannot infer the dependency (e.g., an IAM policy that must exist before a resource that uses the role, but doesn't reference the policy directly).
