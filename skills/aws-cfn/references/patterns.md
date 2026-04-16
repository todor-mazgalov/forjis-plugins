<!-- Parent: aws-cfn/SKILL.md -->
# CloudFormation Resource Patterns

Short, security-focused snippets for common AWS resources. Each shows only the critical best-practice properties -- adapt and compose them for your stack.

## Compute

### EC2 with Launch Template
```yaml
LaunchTemplate:
  Type: AWS::EC2::LaunchTemplate
  Properties:
    LaunchTemplateData:
      InstanceType: !Ref InstanceType
      ImageId: !Ref AMIId
      EbsOptimized: true
      MetadataOptions: { HttpTokens: required, HttpEndpoint: enabled }
      BlockDeviceMappings:
        - DeviceName: /dev/xvda
          Ebs: { VolumeType: gp3, Encrypted: true, KmsKeyId: !GetAtt MyKMSKey.Arn }
      SecurityGroupIds: [!Ref AppSecurityGroup]
      IamInstanceProfile: { Arn: !GetAtt InstanceProfile.Arn }
```

### ECS Fargate Service
```yaml
FargateService:
  Type: AWS::ECS::Service
  Properties:
    Cluster: !Ref Cluster
    LaunchType: FARGATE
    DesiredCount: 2
    NetworkConfiguration:
      AwsvpcConfiguration:
        Subnets: [!Ref PrivateSubnet1, !Ref PrivateSubnet2]
        SecurityGroups: [!Ref ServiceSecurityGroup]
    DeploymentConfiguration:
      MinimumHealthyPercent: 100
      MaximumPercent: 200
      DeploymentCircuitBreaker: { Enable: true, Rollback: true }
```

### Lambda Function
```yaml
MyFunction:
  Type: AWS::Lambda::Function
  Properties:
    Runtime: python3.12
    Architectures: [arm64]
    Handler: index.handler
    MemorySize: 256
    Timeout: 30
    ReservedConcurrentExecutions: 100
    Role: !GetAtt FunctionRole.Arn
    TracingConfig: { Mode: Active }
    VpcConfig: # include when accessing private resources
      SecurityGroupIds: [!Ref LambdaSecurityGroup]
      SubnetIds: [!Ref PrivateSubnet1, !Ref PrivateSubnet2]
```

### EKS Cluster
```yaml
EKSCluster:
  Type: AWS::EKS::Cluster
  Properties:
    Version: "1.30"
    RoleArn: !GetAtt ClusterRole.Arn
    ResourcesVpcConfig:
      SubnetIds: [!Ref PrivateSubnet1, !Ref PrivateSubnet2]
      EndpointPublicAccess: false
      EndpointPrivateAccess: true
    EncryptionConfig:
      - Provider: { KeyArn: !GetAtt MyKMSKey.Arn }
        Resources: [secrets]
```

## Database

### RDS Instance
```yaml
Database:
  Type: AWS::RDS::DBInstance
  DeletionPolicy: Snapshot
  UpdateReplacePolicy: Snapshot
  Properties:
    Engine: !Ref DBEngine  # postgres, mysql, oracle-ee, sqlserver-se, etc.
    DBInstanceClass: !Ref DBInstanceClass
    StorageType: gp3
    StorageEncrypted: true
    MultiAZ: true
    DeletionProtection: true
    BackupRetentionPeriod: 30
    CopyTagsToSnapshot: true
    EnablePerformanceInsights: true
```

### Aurora Serverless v2
```yaml
AuroraCluster:
  Type: AWS::RDS::DBCluster
  DeletionPolicy: Snapshot
  UpdateReplacePolicy: Snapshot
  Properties:
    Engine: !Ref AuroraEngine  # aurora-mysql or aurora-postgresql
    ServerlessV2ScalingConfiguration: { MinCapacity: 0.5, MaxCapacity: 16 }
    StorageEncrypted: true
    DeletionProtection: true
    BackupRetentionPeriod: 30
```

### DynamoDB Table
```yaml
MyTable:
  Type: AWS::DynamoDB::Table
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain
  Properties:
    BillingMode: PAY_PER_REQUEST
    SSESpecification: { SSEEnabled: true, SSEType: KMS, KMSMasterKeyId: !GetAtt MyKMSKey.Arn }
    DeletionProtectionEnabled: true
    PointInTimeRecoverySpecification: { PointInTimeRecoveryEnabled: true }
    AttributeDefinitions: [{ AttributeName: pk, AttributeType: S }]
    KeySchema: [{ AttributeName: pk, KeyType: HASH }]
```

### ElastiCache
```yaml
CacheCluster:
  Type: AWS::ElastiCache::ReplicationGroup
  Properties:
    Engine: redis
    CacheNodeType: !Ref CacheNodeType
    NumCacheClusters: 2
    AutomaticFailoverEnabled: true
    MultiAZEnabled: true
    AtRestEncryptionEnabled: true
    TransitEncryptionEnabled: true
```

## Storage

### S3 Secure Bucket
```yaml
SecureBucket:
  Type: AWS::S3::Bucket
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain
  Properties:
    BucketEncryption:
      ServerSideEncryptionConfiguration:
        - ServerSideEncryptionByDefault: { SSEAlgorithm: "aws:kms", KMSMasterKeyID: !GetAtt MyKMSKey.Arn }
          BucketKeyEnabled: true
    PublicAccessBlockConfiguration: { BlockPublicAcls: true, BlockPublicPolicy: true, IgnorePublicAcls: true, RestrictPublicBuckets: true }
    VersioningConfiguration: { Status: Enabled }
    LoggingConfiguration: { DestinationBucketName: !Ref AccessLogBucket }
```

### EFS
```yaml
FileSystem:
  Type: AWS::EFS::FileSystem
  DeletionPolicy: Retain
  UpdateReplacePolicy: Retain
  Properties:
    Encrypted: true
    KmsKeyId: !GetAtt MyKMSKey.Arn
    ThroughputMode: elastic
    BackupPolicy: { Status: ENABLED }
    LifecyclePolicies: [{ TransitionToIA: AFTER_30_DAYS }]
```

### EBS Volume
```yaml
DataVolume:
  Type: AWS::EC2::Volume
  DeletionPolicy: Snapshot
  Properties:
    VolumeType: gp3
    Size: !Ref VolumeSize
    Encrypted: true
    KmsKeyId: !GetAtt MyKMSKey.Arn
```

## Networking

### VPC with Flow Logs
```yaml
VPC:
  Type: AWS::EC2::VPC
  Properties:
    CidrBlock: !Ref VPCCidr
    EnableDnsSupport: true
    EnableDnsHostnames: true

FlowLog:
  Type: AWS::EC2::FlowLog
  Properties:
    ResourceId: !Ref VPC
    ResourceType: VPC
    TrafficType: ALL
    LogDestinationType: cloud-watch-logs
    LogGroupName: !Ref FlowLogGroup
```

### ALB / NLB
```yaml
ALB:
  Type: AWS::ElasticLoadBalancingV2::LoadBalancer
  Properties:
    Type: application
    Subnets: [!Ref PublicSubnet1, !Ref PublicSubnet2]
    SecurityGroups: [!Ref ALBSecurityGroup]
    LoadBalancerAttributes:
      - { Key: access_logs.s3.enabled, Value: "true" }
      - { Key: access_logs.s3.bucket, Value: !Ref ALBLogBucket }
      - { Key: routing.http.drop_invalid_header_fields.enabled, Value: "true" }

NLB:  # no security groups -- NLBs are transparent to traffic
  Type: AWS::ElasticLoadBalancingV2::LoadBalancer
  Properties:
    Type: network
    Scheme: internal
    Subnets: [!Ref PrivateSubnet1, !Ref PrivateSubnet2]
```

### API Gateway
```yaml
# REST API (v1) -- use for WebSocket or REST with request validation
RestApi:
  Type: AWS::ApiGateway::RestApi
  Properties:
    Name: !Sub "${AWS::StackName}-api"
    EndpointConfiguration: { Types: [REGIONAL] }

# HTTP API (v2) -- simpler, cheaper, supports OIDC/JWT natively
HttpApi:
  Type: AWS::ApiGatewayV2::Api
  Properties:
    Name: !Sub "${AWS::StackName}-http-api"
    ProtocolType: HTTP
```

### CloudFront
```yaml
Distribution:
  Type: AWS::CloudFront::Distribution
  Properties:
    DistributionConfig:
      Enabled: true
      HttpVersion: http2and3
      DefaultCacheBehavior:
        ViewerProtocolPolicy: redirect-to-https
        TargetOriginId: primary
        Compress: true
      ViewerCertificate: { AcmCertificateArn: !Ref Cert, MinimumProtocolVersion: TLSv1.2_2021, SslSupportMethod: sni-only }
      WebACLId: !GetAtt WebACL.Arn
```

## Messaging

### SQS Queue with DLQ
```yaml
MyQueue:
  Type: AWS::SQS::Queue
  Properties:
    KmsMasterKeyId: !GetAtt MyKMSKey.Arn
    VisibilityTimeout: 300
    RedrivePolicy: { deadLetterTargetArn: !GetAtt DLQ.Arn, maxReceiveCount: 3 }

DLQ:
  Type: AWS::SQS::Queue
  Properties:
    KmsMasterKeyId: !GetAtt MyKMSKey.Arn
    MessageRetentionPeriod: 1209600
```

### SNS Topic
```yaml
AlertTopic:
  Type: AWS::SNS::Topic
  Properties:
    KmsMasterKeyId: !GetAtt MyKMSKey.Arn

TopicPolicy:  # restrict who can publish
  Type: AWS::SNS::TopicPolicy
  Properties:
    Topics: [!Ref AlertTopic]
    PolicyDocument:
      Statement:
        - Effect: Allow
          Principal: { Service: cloudwatch.amazonaws.com }
          Action: sns:Publish
          Resource: !Ref AlertTopic
```

### EventBridge Rule
```yaml
ScheduledRule:
  Type: AWS::Events::Rule
  Properties:
    ScheduleExpression: "rate(1 hour)"
    State: ENABLED
    Targets:
      - Arn: !GetAtt TargetFunction.Arn
        Id: Target
        RetryPolicy: { MaximumRetryAttempts: 2, MaximumEventAgeInSeconds: 3600 }
        DeadLetterConfig: { Arn: !GetAtt DLQ.Arn }
```

## Security

### IAM Role (least privilege)
```yaml
AppRole:
  Type: AWS::IAM::Role
  Properties:
    AssumeRolePolicyDocument:
      Statement:
        - Effect: Allow
          Principal: { Service: !Ref TrustedService }
          Action: sts:AssumeRole
          Condition: { StringEquals: { "aws:SourceAccount": !Ref "AWS::AccountId" } }
    Policies:
      - PolicyName: AppPolicy
        PolicyDocument:
          Statement:
            - Effect: Allow
              Action: [s3:GetObject, s3:PutObject]
              Resource: !Sub "${MyBucket.Arn}/*"
```

### KMS Key
```yaml
EncryptionKey:
  Type: AWS::KMS::Key
  Properties:
    EnableKeyRotation: true
    KeyPolicy:
      Statement:
        - Sid: Admin
          Effect: Allow
          Principal: { AWS: !Sub "arn:aws:iam::${AWS::AccountId}:root" }
          Action: "kms:*"
          Resource: "*"
        - Sid: Usage
          Effect: Allow
          Principal: { AWS: !GetAtt AppRole.Arn }
          Action: [kms:Encrypt, kms:Decrypt, kms:GenerateDataKey]
          Resource: "*"
```

### Secrets Manager
```yaml
AppSecret:
  Type: AWS::SecretsManager::Secret
  Properties:
    KmsKeyId: !GetAtt EncryptionKey.Arn
    GenerateSecretString: { SecretStringTemplate: '{"username":"admin"}', GenerateStringKey: password, PasswordLength: 32, ExcludeCharacters: '"@/\' }
```

### Security Group
```yaml
AppSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: !Sub "${AWS::StackName} app tier"
    VpcId: !Ref VPC
    SecurityGroupIngress:
      - { IpProtocol: tcp, FromPort: 443, ToPort: 443, SourceSecurityGroupId: !Ref LBSecurityGroup }
    SecurityGroupEgress:
      - { IpProtocol: tcp, FromPort: 443, ToPort: 443, CidrIp: 0.0.0.0/0 }
```

### WAF Web ACL
```yaml
WebACL:
  Type: AWS::WAFv2::WebACL
  Properties:
    Scope: REGIONAL   # use CLOUDFRONT for CloudFront distributions
    DefaultAction: { Allow: {} }
    VisibilityConfig: { SampledRequestsEnabled: true, CloudWatchMetricsEnabled: true, MetricName: waf }
    Rules:
      - Name: CommonRules
        Priority: 1
        OverrideAction: { None: {} }
        Statement: { ManagedRuleGroupStatement: { VendorName: AWS, Name: AWSManagedRulesCommonRuleSet } }
        VisibilityConfig: { SampledRequestsEnabled: true, CloudWatchMetricsEnabled: true, MetricName: common }
      - Name: RateLimit
        Priority: 2
        Action: { Block: {} }
        Statement: { RateBasedStatement: { Limit: 2000, AggregateKeyType: IP } }
        VisibilityConfig: { SampledRequestsEnabled: true, CloudWatchMetricsEnabled: true, MetricName: rate }
```
