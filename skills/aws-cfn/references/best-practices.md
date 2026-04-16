<!-- Parent: aws-cfn/SKILL.md -->
# CloudFormation Best Practices

Core principles for writing maintainable, production-grade CloudFormation templates.

---

## Table of Contents

1. [Template Structure](#template-structure)
2. [Parameters](#parameters)
3. [Naming Conventions](#naming-conventions)
4. [Intrinsic Functions](#intrinsic-functions)
5. [Cross-Stack References](#cross-stack-references)
6. [Tagging Strategy](#tagging-strategy)
7. [Template Modularity](#template-modularity)
8. [Outputs](#outputs)
9. [Metadata and Interface](#metadata-and-interface)

---

## Template Structure

### Section Order

Always follow this order for consistency:

1. `AWSTemplateFormatVersion`
2. `Description`
3. `Metadata`
4. `Parameters`
5. `Mappings`
6. `Conditions`
7. `Rules`
8. `Resources`
9. `Outputs`

### Template Description

Every template must have a `Description` that explains what it provisions and any prerequisites:

```yaml
Description: >-
  Provisions a production-ready VPC with public and private subnets
  across two AZs, NAT gateways, and VPC Flow Logs.
  Prerequisite: None. This is a foundational stack.
```

### YAML Over JSON

- Use YAML for all templates — it supports comments and is more readable
- Use short-form intrinsic functions (`!Ref`, `!Sub`, `!GetAtt`, `!If`)
- Use `>-` for multi-line strings (folds into single line, strips trailing newline)
- Use `|` for literal blocks (preserves newlines)

---

## Parameters

### Constraint Everything

```yaml
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]
    Default: dev
    Description: Deployment environment

  VPCCidr:
    Type: String
    Default: "10.0.0.0/16"
    AllowedPattern: '(\d{1,3}\.){3}\d{1,3}/\d{1,2}'
    ConstraintDescription: Must be a valid CIDR block (e.g., 10.0.0.0/16)

  InstanceType:
    Type: String
    Default: t3.medium
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
      - t3.large
      - m6i.large
    Description: EC2 instance type for the application tier

  NotificationEmail:
    Type: String
    AllowedPattern: '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}'
    ConstraintDescription: Must be a valid email address
```

### Use AWS-Specific Parameter Types

```yaml
Parameters:
  VpcId:
    Type: AWS::EC2::VPC::Id
    Description: Target VPC

  SubnetIds:
    Type: List<AWS::EC2::Subnet::Id>
    Description: Subnets for the application tier

  KeyPairName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: SSH key pair

  AMIId:
    Type: AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>
    Default: /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64
```

### Group Parameters with Interface Metadata

```yaml
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: Network Configuration
        Parameters:
          - VpcId
          - SubnetIds
          - VPCCidr
      - Label:
          default: Application Configuration
        Parameters:
          - Environment
          - InstanceType
      - Label:
          default: Notifications
        Parameters:
          - NotificationEmail
    ParameterLabels:
      VpcId:
        default: "Which VPC should this deploy to?"
      Environment:
        default: "Deployment environment"
```

---

## Naming Conventions

### Logical IDs

Use PascalCase for logical resource IDs. Be descriptive but concise:

| Good | Bad |
|------|-----|
| `AppSecurityGroup` | `SG1` |
| `PrivateSubnet1` | `Subnet` |
| `DatabaseSecret` | `Secret123` |
| `ALBAccessLogBucket` | `Bucket` |

### Physical Names

Avoid hardcoding physical names — let CloudFormation auto-generate them. When names are required, use `!Sub` with the stack name:

```yaml
# GOOD: Dynamic naming
MyBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: !Sub "${AWS::StackName}-data-${AWS::AccountId}"

# BAD: Hardcoded name (will fail on second deploy)
MyBucket:
  Type: AWS::S3::Bucket
  Properties:
    BucketName: my-application-bucket
```

### Tag Names

Use consistent tag key casing — `PascalCase` or `kebab-case`, not mixed:

```yaml
Tags:
  - Key: Environment
    Value: !Ref Environment
  - Key: Project
    Value: !Ref ProjectName
  - Key: Owner
    Value: !Ref OwnerEmail
  - Key: ManagedBy
    Value: CloudFormation
```

---

## Intrinsic Functions

### Prefer `!Sub` Over `!Join`

```yaml
# GOOD: Readable
Value: !Sub "arn:aws:s3:::${MyBucket}/*"

# BAD: Hard to read
Value: !Join
  - ""
  - - "arn:aws:s3:::"
    - !Ref MyBucket
    - "/*"
```

### Use `!GetAtt` for Resource Attributes

```yaml
# Get specific attributes instead of constructing ARNs manually
RoleArn: !GetAtt MyRole.Arn
BucketDomainName: !GetAtt MyBucket.DomainName
DBEndpoint: !GetAtt MyDatabase.Endpoint.Address
```

### Conditions

```yaml
Conditions:
  IsProd: !Equals [!Ref Environment, prod]
  IsNotProd: !Not [Condition: IsProd]
  CreateReadReplica: !And
    - Condition: IsProd
    - !Equals [!Ref EnableReadReplica, "true"]

Resources:
  ReadReplica:
    Type: AWS::RDS::DBInstance
    Condition: CreateReadReplica
    Properties:
      SourceDBInstanceIdentifier: !Ref PrimaryDB
```

---

## Cross-Stack References

### Export Values

```yaml
# network-stack.yaml
Outputs:
  VPCId:
    Value: !Ref VPC
    Export:
      Name: !Sub "${AWS::StackName}-VPCId"
  PrivateSubnet1Id:
    Value: !Ref PrivateSubnet1
    Export:
      Name: !Sub "${AWS::StackName}-PrivateSubnet1Id"
```

### Import Values

```yaml
# app-stack.yaml
Parameters:
  NetworkStackName:
    Type: String
    Description: Name of the network stack to import from

Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      SubnetId: !ImportValue
        Fn::Sub: "${NetworkStackName}-PrivateSubnet1Id"
```

### Prefer SSM Parameters for Loose Coupling

Exports create hard dependencies between stacks. For looser coupling, write values to SSM Parameter Store:

```yaml
# Writer stack
VPCIdParam:
  Type: AWS::SSM::Parameter
  Properties:
    Name: !Sub "/${Environment}/network/vpc-id"
    Type: String
    Value: !Ref VPC

# Reader stack
Parameters:
  VPCId:
    Type: AWS::SSM::Parameter::Value<String>
    Default: !Sub "/${Environment}/network/vpc-id"
```

---

## Tagging Strategy

### Minimum Required Tags

Every taggable resource must have:

| Tag | Purpose |
|-----|---------|
| `Name` | Human-readable resource name |
| `Environment` | dev / staging / prod |
| `Project` | Project or application name |
| `Owner` | Team or individual responsible |
| `ManagedBy` | `CloudFormation` (distinguishes from manual resources) |

### Optional Tags

| Tag | Purpose |
|-----|---------|
| `CostCenter` | Billing allocation |
| `Compliance` | Regulatory scope (HIPAA, PCI, etc.) |
| `DataClassification` | public / internal / confidential / restricted |
| `Backup` | Whether AWS Backup should include this resource |

---

## Template Modularity

### When to Split Stacks

Split when:
- Templates exceed 500 resources
- Components have different lifecycle (network changes rarely, app changes often)
- Different teams own different layers
- You need to reuse infrastructure patterns across projects

### Nested Stacks vs Cross-Stack References

| Approach | Use When |
|----------|----------|
| Nested stacks | Tightly coupled components deployed together |
| Cross-stack exports | Shared infrastructure consumed by multiple stacks |
| SSM parameters | Loose coupling, values change independently |

---

## Outputs

### Always Output Key Resource Identifiers

```yaml
Outputs:
  VPCId:
    Description: VPC identifier
    Value: !Ref VPC
    Export:
      Name: !Sub "${AWS::StackName}-VPCId"

  ALBDNSName:
    Description: Application load balancer DNS name
    Value: !GetAtt ALB.DNSName

  DatabaseEndpoint:
    Description: RDS instance endpoint address
    Value: !GetAtt Database.Endpoint.Address

  DatabasePort:
    Description: RDS instance port
    Value: !GetAtt Database.Endpoint.Port
```

---

## Metadata and Interface

### CloudFormation Interface

Improve the Console experience with parameter grouping and labels:

```yaml
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Network"
        Parameters:
          - VPCCidr
          - PublicSubnet1Cidr
          - PublicSubnet2Cidr
          - PrivateSubnet1Cidr
          - PrivateSubnet2Cidr
      - Label:
          default: "Compute"
        Parameters:
          - InstanceType
          - KeyPairName
      - Label:
          default: "Database"
        Parameters:
          - DBInstanceClass
          - DBName
```
