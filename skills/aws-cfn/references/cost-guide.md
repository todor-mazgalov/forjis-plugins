<!-- Parent: aws-cfn/SKILL.md -->
# CloudFormation Cost Optimization Guide

Patterns for cost-efficient AWS infrastructure provisioned through CloudFormation.

---

## Table of Contents

1. [Right-Sizing](#right-sizing)
2. [Compute Optimization](#compute-optimization)
3. [Storage Lifecycle](#storage-lifecycle)
4. [Database Cost Optimization](#database-cost-optimization)
5. [Environment Scheduling](#environment-scheduling)
6. [Network Cost Reduction](#network-cost-reduction)
7. [Cost Tags](#cost-tags)

---

## Right-Sizing

### Parameterize Instance Types

Always make instance types configurable per environment:

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]

  EC2InstanceType:
    Type: String
    Default: t3.medium
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
      - t3.large
      - m6i.large
      - m6i.xlarge

Conditions:
  IsProd: !Equals [!Ref Environment, prod]

Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !If [IsProd, m6i.large, t3.small]
```

### Use Mappings for Environment-Specific Sizing

```yaml
Mappings:
  EnvironmentConfig:
    dev:
      InstanceType: t3.small
      DBInstanceClass: db.t3.small
      MinCapacity: 1
      MaxCapacity: 2
    staging:
      InstanceType: t3.medium
      DBInstanceClass: db.t3.medium
      MinCapacity: 1
      MaxCapacity: 4
    prod:
      InstanceType: m6i.large
      DBInstanceClass: db.r6g.large
      MinCapacity: 2
      MaxCapacity: 20
```

---

## Compute Optimization

### Graviton (ARM) Instances

Graviton processors offer up to 40% better price-performance. Use where compatible:

```yaml
Parameters:
  UseGraviton:
    Type: String
    Default: "true"
    AllowedValues: ["true", "false"]

Conditions:
  UseGravitonInstances: !Equals [!Ref UseGraviton, "true"]

Resources:
  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateData:
        InstanceType: !If [UseGravitonInstances, m7g.large, m6i.large]
```

### Spot Instances for Non-Critical Workloads

```yaml
SpotFleet:
  Type: AWS::EC2::SpotFleet
  Properties:
    SpotFleetRequestConfigData:
      IamFleetRole: !GetAtt SpotFleetRole.Arn
      TargetCapacity: 4
      AllocationStrategy: capacityOptimized
      LaunchTemplateConfigs:
        - LaunchTemplateSpecification:
            LaunchTemplateId: !Ref LaunchTemplate
            Version: !GetAtt LaunchTemplate.LatestVersionNumber
          Overrides:
            - InstanceType: m6i.large
            - InstanceType: m5.large
            - InstanceType: m6a.large
```

### Fargate Spot for ECS

```yaml
FargateCapacityProvider:
  Type: AWS::ECS::CapacityProvider
  Properties:
    AutoScalingGroupProvider:
      ManagedScaling:
        Status: ENABLED
    Name: FargateSpot

ECSService:
  Type: AWS::ECS::Service
  Properties:
    CapacityProviderStrategy:
      - CapacityProvider: FARGATE
        Base: 1
        Weight: 1
      - CapacityProvider: FARGATE_SPOT
        Weight: 3
```

### Lambda Right-Sizing

```yaml
MyFunction:
  Type: AWS::Lambda::Function
  Properties:
    MemorySize: 256
    Timeout: 30
    Architectures:
      - arm64
```

---

## Storage Lifecycle

### S3 Intelligent-Tiering

```yaml
DataBucket:
  Type: AWS::S3::Bucket
  Properties:
    IntelligentTieringConfigurations:
      - Id: EntireBucket
        Status: Enabled
        Tierings:
          - AccessTier: ARCHIVE_ACCESS
            Days: 90
          - AccessTier: DEEP_ARCHIVE_ACCESS
            Days: 180
```

### S3 Lifecycle Rules

```yaml
LogBucket:
  Type: AWS::S3::Bucket
  Properties:
    LifecycleConfiguration:
      Rules:
        - Id: TransitionAndExpire
          Status: Enabled
          Transitions:
            - StorageClass: STANDARD_IA
              TransitionInDays: 30
            - StorageClass: GLACIER
              TransitionInDays: 90
          ExpirationInDays: 365
        - Id: CleanupIncomplete
          Status: Enabled
          AbortIncompleteMultipartUpload:
            DaysAfterInitiation: 7
```

### EBS Volume Types

```yaml
# Use gp3 instead of gp2 — same performance, 20% cheaper
Volume:
  Type: AWS::EC2::Volume
  Properties:
    VolumeType: gp3
    Size: 100
    Iops: 3000
    Throughput: 125
    Encrypted: true
```

---

## Database Cost Optimization

### RDS — Use Aurora Serverless v2 for Variable Workloads

```yaml
AuroraCluster:
  Type: AWS::RDS::DBCluster
  Properties:
    Engine: aurora-postgresql
    EngineVersion: "16.4"
    ServerlessV2ScalingConfiguration:
      MinCapacity: 0.5
      MaxCapacity: 16
    StorageEncrypted: true

AuroraInstance:
  Type: AWS::RDS::DBInstance
  Properties:
    DBClusterIdentifier: !Ref AuroraCluster
    DBInstanceClass: db.serverless
    Engine: aurora-postgresql
```

### DynamoDB — On-Demand for Unpredictable Workloads

```yaml
MyTable:
  Type: AWS::DynamoDB::Table
  Properties:
    BillingMode: PAY_PER_REQUEST
```

### DynamoDB — Provisioned with Auto Scaling for Predictable Workloads

Use provisioned capacity with auto scaling for steady-state workloads (cheaper than on-demand at consistent utilization).

### ElastiCache — Use Serverless for Variable Traffic

```yaml
CacheServerless:
  Type: AWS::ElastiCache::ServerlessCache
  Properties:
    Engine: redis
    ServerlessCacheName: !Sub "${AWS::StackName}-cache"
    CacheUsageLimits:
      DataStorage:
        Maximum: 10
        Unit: GB
```

---

## Environment Scheduling

### Auto Scaling Scheduled Actions

Shut down non-production environments outside business hours:

```yaml
ScaleDown:
  Type: AWS::AutoScaling::ScheduledAction
  Properties:
    AutoScalingGroupName: !Ref AutoScalingGroup
    DesiredCapacity: 0
    MinSize: 0
    Recurrence: "0 20 * * MON-FRI"

ScaleUp:
  Type: AWS::AutoScaling::ScheduledAction
  Properties:
    AutoScalingGroupName: !Ref AutoScalingGroup
    DesiredCapacity: 2
    MinSize: 1
    Recurrence: "0 8 * * MON-FRI"
```

### Conditional Resource Creation

```yaml
Conditions:
  IsProd: !Equals [!Ref Environment, prod]
  IsNotProd: !Not [!Equals [!Ref Environment, prod]]

Resources:
  NATGateway:
    Type: AWS::EC2::NatGateway
    Condition: IsProd
    Properties:
      SubnetId: !Ref PublicSubnet1
      AllocationId: !GetAtt EIP.AllocationId

  # Dev environments can use a NAT instance instead (cheaper)
```

---

## Network Cost Reduction

### VPC Endpoints for AWS Services

Avoid NAT Gateway data processing charges for AWS service traffic:

```yaml
S3Endpoint:
  Type: AWS::EC2::VPCEndpoint
  Properties:
    VpcId: !Ref VPC
    ServiceName: !Sub "com.amazonaws.${AWS::Region}.s3"
    VpcEndpointType: Gateway
    RouteTableIds:
      - !Ref PrivateRouteTable

DynamoDBEndpoint:
  Type: AWS::EC2::VPCEndpoint
  Properties:
    VpcId: !Ref VPC
    ServiceName: !Sub "com.amazonaws.${AWS::Region}.dynamodb"
    VpcEndpointType: Gateway
    RouteTableIds:
      - !Ref PrivateRouteTable

ECREndpoint:
  Type: AWS::EC2::VPCEndpoint
  Properties:
    VpcId: !Ref VPC
    ServiceName: !Sub "com.amazonaws.${AWS::Region}.ecr.dkr"
    VpcEndpointType: Interface
    PrivateDnsEnabled: true
    SubnetIds:
      - !Ref PrivateSubnet1
      - !Ref PrivateSubnet2
    SecurityGroupIds:
      - !Ref EndpointSecurityGroup
```

### Single NAT Gateway for Non-Production

```yaml
# Production: one NAT per AZ for high availability
# Non-production: single NAT to reduce cost
NATGateway:
  Type: AWS::EC2::NatGateway
  Properties:
    SubnetId: !Ref PublicSubnet1
    AllocationId: !GetAtt EIP.AllocationId
```

---

## Cost Tags

### Mandatory Tags for Cost Allocation

```yaml
Resources:
  MyResource:
    Type: AWS::SomeService::Resource
    Properties:
      Tags:
        - Key: Environment
          Value: !Ref Environment
        - Key: Project
          Value: !Ref ProjectName
        - Key: Owner
          Value: !Ref OwnerEmail
        - Key: CostCenter
          Value: !Ref CostCenter
        - Key: ManagedBy
          Value: CloudFormation
        - Key: aws:cloudformation:stack-name
          Value: !Ref AWS::StackName
```

### Tag Enforcement

Use AWS Config rules or Service Control Policies (SCPs) to enforce tagging. Document required tags in the template description:

```yaml
Description: >-
  Required tags: Environment, Project, Owner, CostCenter.
  Deploy with --tags to set stack-level tags.
```
