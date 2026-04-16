<!-- Parent: aws-cfn/SKILL.md -->
# CloudFormation Reliability Guide

Patterns for building resilient, fault-tolerant AWS infrastructure with CloudFormation.

---

## Table of Contents

1. [Multi-AZ Deployments](#multi-az-deployments)
2. [Auto Scaling](#auto-scaling)
3. [Health Checks](#health-checks)
4. [Backup and Recovery](#backup-and-recovery)
5. [Deletion Protection](#deletion-protection)
6. [Update Policies](#update-policies)
7. [Disaster Recovery](#disaster-recovery)

---

## Multi-AZ Deployments

### Principle

All production workloads must be deployed across at least two Availability Zones. Single-AZ deployments are acceptable only for development environments.

### Quick Reference: Multi-AZ Properties by Service

| Service | Resource Type | Multi-AZ Property | Notes |
|---------|-------------|-------------------|-------|
| RDS | `AWS::RDS::DBInstance` | `MultiAZ: true` | Synchronous standby replica |
| Aurora | `AWS::RDS::DBCluster` | Deploy instances in 2+ AZs | Cluster storage spans 3 AZs automatically |
| ElastiCache | `AWS::ElastiCache::ReplicationGroup` | `MultiAZEnabled: true` + `AutomaticFailoverEnabled: true` | Requires `NumCacheClusters >= 2` |
| ECS | `AWS::ECS::Service` | List 2+ subnets in `NetworkConfiguration` | Set `DesiredCount >= 2` |
| EC2 ASG | `AWS::AutoScaling::AutoScalingGroup` | List 2+ subnets in `VPCZoneIdentifier` | Set `MinSize >= 2` for HA |
| OpenSearch | `AWS::OpenSearchService::Domain` | `ZoneAwarenessEnabled: true` in `ClusterConfig` | Set `AvailabilityZoneCount: 2` or `3` |
| MSK | `AWS::MSK::Cluster` | List subnets across 2-3 AZs in `BrokerNodeGroupInfo` | Minimum 2 AZs required |
| EFS | `AWS::EFS::FileSystem` | Create mount targets in 2+ AZs | Storage replicates automatically |

### Example: RDS Multi-AZ

```yaml
Database:
  Type: AWS::RDS::DBInstance
  DeletionPolicy: Snapshot
  UpdateReplacePolicy: Snapshot
  Properties:
    MultiAZ: true
    Engine: !Ref DBEngine
    DBInstanceClass: !Ref DBInstanceClass
    StorageEncrypted: true
    DeletionProtection: true
    BackupRetentionPeriod: 30
    CopyTagsToSnapshot: true
    EnablePerformanceInsights: true
```

### Example: ECS Multi-AZ with Circuit Breaker

```yaml
Service:
  Type: AWS::ECS::Service
  Properties:
    Cluster: !Ref Cluster
    DesiredCount: 2
    NetworkConfiguration:
      AwsvpcConfiguration:
        Subnets:
          - !Ref PrivateSubnet1
          - !Ref PrivateSubnet2
        SecurityGroups:
          - !Ref AppSecurityGroup
    DeploymentConfiguration:
      MinimumHealthyPercent: 100
      MaximumPercent: 200
      DeploymentCircuitBreaker:
        Enable: true
        Rollback: true
```

---

## Auto Scaling

### Quick Reference: Auto Scaling by Service

| Service | Scaling Resource | ScalableDimension | Key Metric |
|---------|-----------------|-------------------|------------|
| EC2 | `AWS::AutoScaling::AutoScalingGroup` | N/A (native) | `ASGAverageCPUUtilization` |
| ECS | `AWS::ApplicationAutoScaling::ScalableTarget` | `ecs:service:DesiredCount` | `ECSServiceAverageCPUUtilization` |
| DynamoDB | `AWS::ApplicationAutoScaling::ScalableTarget` | `dynamodb:table:WriteCapacityUnits` | `DynamoDBWriteCapacityUtilization` |
| Aurora | `AWS::ApplicationAutoScaling::ScalableTarget` | `rds:cluster:ReadReplicaCount` | `RDSReaderAverageCPUUtilization` |
| Lambda | Built-in (provisioned concurrency) | `lambda:function:ProvisionedConcurrency` | `ProvisionedConcurrencyUtilization` |
| Neptune | `AWS::ApplicationAutoScaling::ScalableTarget` | `neptune:cluster:ReadReplicaCount` | Custom CloudWatch metric |

### EC2 Auto Scaling Group

```yaml
AutoScalingGroup:
  Type: AWS::AutoScaling::AutoScalingGroup
  UpdatePolicy:
    AutoScalingRollingUpdate:
      MinInstancesInService: 1
      MaxBatchSize: 1
      PauseTime: PT10M
      WaitOnResourceSignals: true
  Properties:
    MinSize: !Ref MinInstances
    MaxSize: !Ref MaxInstances
    VPCZoneIdentifier:
      - !Ref PrivateSubnet1
      - !Ref PrivateSubnet2
    HealthCheckType: ELB
    HealthCheckGracePeriod: 300
    LaunchTemplate:
      LaunchTemplateId: !Ref LaunchTemplate
      Version: !GetAtt LaunchTemplate.LatestVersionNumber
```

### Application Auto Scaling (ECS example)

This pattern applies to any service that uses `AWS::ApplicationAutoScaling`. Change the `ResourceId`, `ScalableDimension`, `ServiceNamespace`, and metric to match your service (see table above).

```yaml
ScalableTarget:
  Type: AWS::ApplicationAutoScaling::ScalableTarget
  Properties:
    MaxCapacity: 10
    MinCapacity: 2
    ResourceId: !Sub "service/${Cluster}/${Service.Name}"
    ScalableDimension: ecs:service:DesiredCount
    ServiceNamespace: ecs

ScalingPolicy:
  Type: AWS::ApplicationAutoScaling::ScalingPolicy
  Properties:
    PolicyName: CPUScaling
    PolicyType: TargetTrackingScaling
    ScalingTargetId: !Ref ScalableTarget
    TargetTrackingScalingPolicyConfiguration:
      PredefinedMetricSpecification:
        PredefinedMetricType: ECSServiceAverageCPUUtilization
      TargetValue: 60
      ScaleInCooldown: 300
      ScaleOutCooldown: 60
```

---

## Health Checks

### ALB Health Check

```yaml
TargetGroup:
  Type: AWS::ElasticLoadBalancingV2::TargetGroup
  Properties:
    Port: 8080
    Protocol: HTTP
    VpcId: !Ref VPC
    TargetType: ip
    HealthCheckPath: /health
    HealthCheckProtocol: HTTP
    HealthCheckIntervalSeconds: 30
    HealthyThresholdCount: 2
    UnhealthyThresholdCount: 3
    HealthCheckTimeoutSeconds: 10
    Matcher:
      HttpCode: "200"
```

### Route 53 Health Check

```yaml
HealthCheck:
  Type: AWS::Route53::HealthCheck
  Properties:
    HealthCheckConfig:
      Type: HTTPS
      FullyQualifiedDomainName: !Ref DomainName
      Port: 443
      ResourcePath: /health
      RequestInterval: 30
      FailureThreshold: 3
      EnableSNI: true
```

---

## Backup and Recovery

### AWS Backup Plan

```yaml
BackupPlan:
  Type: AWS::Backup::BackupPlan
  Properties:
    BackupPlan:
      BackupPlanName: !Sub "${AWS::StackName}-backup"
      BackupPlanRule:
        - RuleName: DailyBackup
          TargetBackupVault: !Ref BackupVault
          ScheduleExpression: "cron(0 3 * * ? *)"
          StartWindowMinutes: 60
          CompletionWindowMinutes: 180
          Lifecycle:
            DeleteAfterDays: 90
            MoveToColdStorageAfterDays: 30
        - RuleName: MonthlyBackup
          TargetBackupVault: !Ref BackupVault
          ScheduleExpression: "cron(0 3 1 * ? *)"
          Lifecycle:
            DeleteAfterDays: 365

BackupSelection:
  Type: AWS::Backup::BackupSelection
  Properties:
    BackupPlanId: !Ref BackupPlan
    BackupSelection:
      SelectionName: TaggedResources
      IamRoleArn: !GetAtt BackupRole.Arn
      ListOfTags:
        - ConditionType: STRINGEQUALS
          ConditionKey: backup
          ConditionValue: "true"
```

### DynamoDB Point-in-Time Recovery

```yaml
MyTable:
  Type: AWS::DynamoDB::Table
  Properties:
    PointInTimeRecoverySpecification:
      PointInTimeRecoveryEnabled: true
```

### RDS Automated Backups

```yaml
MyDatabase:
  Type: AWS::RDS::DBInstance
  Properties:
    BackupRetentionPeriod: 30
    PreferredBackupWindow: "03:00-04:00"
    CopyTagsToSnapshot: true
```

---

## Deletion Protection

### Stateful Resources

Always set `DeletionPolicy` and `UpdateReplacePolicy` on stateful resources:

```yaml
# Databases — snapshot before delete
MyDatabase:
  Type: AWS::RDS::DBInstance
  DeletionPolicy: Snapshot
  UpdateReplacePolicy: Snapshot
  Properties:
    DeletionProtection: true

# S3 — retain on delete
DataBucket:
  Type: AWS::S3::Bucket
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain

# DynamoDB — retain on delete
DataTable:
  Type: AWS::DynamoDB::Table
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain
  Properties:
    DeletionProtectionEnabled: true

# EFS — retain on delete
FileSystem:
  Type: AWS::EFS::FileSystem
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain
```

### Stack Termination Protection

Always recommend enabling termination protection for production stacks:

```bash
aws cloudformation update-termination-protection \
  --enable-termination-protection \
  --stack-name my-production-stack
```

---

## Update Policies

### Rolling Updates

```yaml
AutoScalingGroup:
  Type: AWS::AutoScaling::AutoScalingGroup
  UpdatePolicy:
    AutoScalingRollingUpdate:
      MinInstancesInService: 1
      MaxBatchSize: 1
      PauseTime: PT10M
      WaitOnResourceSignals: true
      SuspendProcesses:
        - HealthCheck
        - ReplaceUnhealthy
        - AZRebalance
        - AlarmNotification
        - ScheduledActions
```

### ECS Rolling Deployment

```yaml
ECSService:
  Type: AWS::ECS::Service
  Properties:
    DeploymentConfiguration:
      MinimumHealthyPercent: 100
      MaximumPercent: 200
      DeploymentCircuitBreaker:
        Enable: true
        Rollback: true
```

---

## Disaster Recovery

### Cross-Region Replication (S3)

```yaml
PrimaryBucket:
  Type: AWS::S3::Bucket
  Properties:
    VersioningConfiguration:
      Status: Enabled
    ReplicationConfiguration:
      Role: !GetAtt ReplicationRole.Arn
      Rules:
        - Id: ReplicateAll
          Status: Enabled
          Destination:
            Bucket: !Sub "arn:aws:s3:::${ReplicaBucketName}"
            StorageClass: STANDARD_IA
```

### RDS Read Replica (Cross-Region)

```yaml
ReadReplica:
  Type: AWS::RDS::DBInstance
  Properties:
    SourceDBInstanceIdentifier: !Sub "arn:aws:rds:${PrimaryRegion}:${AWS::AccountId}:db:${PrimaryDBIdentifier}"
    DBInstanceClass: !Ref DBInstanceClass
    StorageEncrypted: true
    KmsKeyId: !Ref DRKMSKey
```

### DynamoDB Global Table

```yaml
GlobalTable:
  Type: AWS::DynamoDB::GlobalTable
  Properties:
    TableName: MyGlobalTable
    AttributeDefinitions:
      - AttributeName: pk
        AttributeType: S
    KeySchema:
      - AttributeName: pk
        KeyType: HASH
    BillingMode: PAY_PER_REQUEST
    StreamSpecification:
      StreamViewType: NEW_AND_OLD_IMAGES
    SSESpecification:
      SSEEnabled: true
    Replicas:
      - Region: us-east-1
        PointInTimeRecoverySpecification:
          PointInTimeRecoveryEnabled: true
      - Region: eu-west-1
        PointInTimeRecoverySpecification:
          PointInTimeRecoveryEnabled: true
```
