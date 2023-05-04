---
layout: post
title:  "AWS Airflow(MWAA) 설치하기"
author: Kimuksung
categories: [ AWS, Airflow ]
tags: [ AWS, Airflow ]
image: assets/images/mwaa-logo.jpeg
# 댓글 기능
comments: False
---

안녕하세요

오늘은 AWS 인프라를 구축하면서 AWS Airflow인 MWAA를 설치하는 과정에 대해서 설명드리려 합니다. 

간단하게 AWS CLI를 사용하여 구축하였습니다.

#### Install MWAA

##### 1. AWS CLI

---

##### 1-1) AWS CLI 설치 

- Command로 작업하도록 도와주는 역할
- [참조 Document](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/getting-started-install.html#getting-started-install-instructions)

```python
$ curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
$ sudo softwareupdate --install-rosetta
$ sudo installer -pkg AWSCLIV2.pkg -target /

# 제대로 동작하는지 체크
$ which aws
$ aws --version
> aws-cli/2.8.5 Python/3.9.11 Darwin/21.6.0 exe/x86_64 prompt/off
```

##### 1-2) Certification 설정 

- aws 사용자 계정에서 본인의 계정의 Certification -> ( access_key_id , access_key ) 값을 가져와 아래 코드 처럼 입력하여 줍니다.
- [AWS Certification을 모른다면?](https://Kimuksung.github.io/aws-credentials/)
    <a href='https://ifh.cc/v-FVv3Zz' target='_blank'><img src='https://ifh.cc/g/FVv3Zz.jpg' border='0'></a>
    
- [참조 Document](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/cli-configure-files.html)
- CLI 아래와 같이 설정
    ```bash
    $ aws configure
    AWS Access Key ID [None]: id
    AWS Secret Access Key [None]: key
    Default region name [None]: ap-northeast-2
    Default output format [None]: json
    ```
    

##### 2. MWAA

---

##### 2-1) download or save mwaa-environment-public-network.yml - [링크](https://docs.aws.amazon.com/ko_kr/mwaa/latest/userguide/quick-start.html#quick-start-createstack)

- 환경 설정
    - stack-name = cli 넘겨줄 값
    - EnvironmentName = 구성할 이름을 지정한 value
    - 네트워크 세팅
        - VPC => VpcCIDR
        - Default 값은 2개의 Public Subnet과 2개의 Private Subnet으로 구성됩니다.
    - WorkerNode 세팅
        - MaxWorkerNodes
        - Default = 2  
<br>
- Resource
    - VPC, InternetGateway
        - value 값에 맞추어 VPC,InternetGateway 생성합니다.
    - NatGateway 
        - Public Subnet에 추가하여 연결
        - Private Subnet을 외부와 연결해주기 위한 용도
    - RouteTable
        - public 1개
        - private 2개
    - SecurityGroup
        - VPC Security Group 생성
    - EnvironmentBucket
        - S3 버킷 생성  
<br>
- MWAA 설정
    - SecurityGroup
        - Resource VPC 연결
    - IAM Role
    - MwaaExecutionPolicy  
<br>
- code
    
    ```bash
    AWSTemplateFormatVersion: "2010-09-09"
    
    Parameters:
    
      EnvironmentName:
        Description: An environment name that is prefixed to resource names
        Type: String
        Default: MWAAEnvironment
    # Network Setting
      VpcCIDR:
        Description: The IP range (CIDR notation) for this VPC
        Type: String
        Default: 10.192.0.0/16
    
      PublicSubnet1CIDR:
        Description: The IP range (CIDR notation) for the public subnet in the first Availability Zone
        Type: String
        Default: 10.192.10.0/24
    
      PublicSubnet2CIDR:
        Description: The IP range (CIDR notation) for the public subnet in the second Availability Zone
        Type: String
        Default: 10.192.11.0/24
    
      PrivateSubnet1CIDR:
        Description: The IP range (CIDR notation) for the private subnet in the first Availability Zone
        Type: String
        Default: 10.192.20.0/24
      PrivateSubnet2CIDR:
        Description: The IP range (CIDR notation) for the private subnet in the second Availability Zone
        Type: String
        Default: 10.192.21.0/24
      # 최대 worker 수
      MaxWorkerNodes:
        Description: The maximum number of workers that can run in the environment
        Type: Number
        Default: 2
      # Log 구성
      DagProcessingLogs:
        Description: Log level for DagProcessing
        Type: String
        Default: INFO
      SchedulerLogsLevel:
        Description: Log level for SchedulerLogs
        Type: String
        Default: INFO
      TaskLogsLevel:
        Description: Log level for TaskLogs
        Type: String
        Default: INFO
      WorkerLogsLevel:
        Description: Log level for WorkerLogs
        Type: String
        Default: INFO
      WebserverLogsLevel:
        Description: Log level for WebserverLogs
        Type: String
        Default: INFO
    
    Resources:
      #####################################################################################################################
      # CREATE VPC
      #####################################################################################################################
    
      VPC:
        Type: AWS::EC2::VPC
        Properties:
          CidrBlock: !Ref VpcCIDR
          EnableDnsSupport: true
          EnableDnsHostnames: true
          Tags:
            - Key: Name
              # vpc에서 보이는 이름 설정 값
              Value: MWAAEnvironment
    
      InternetGateway:
        Type: AWS::EC2::InternetGateway
        Properties:
          Tags:
            - Key: Name
              # 인터넷 게이트웨이에 보이는 이름 설정 값
              Value: MWAAEnvironment
    
      InternetGatewayAttachment:
        Type: AWS::EC2::VPCGatewayAttachment
        Properties:
          InternetGatewayId: !Ref InternetGateway
          VpcId: !Ref VPC
    
      PublicSubnet1:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          AvailabilityZone: !Select [ 0, !GetAZs '' ]
          CidrBlock: !Ref PublicSubnet1CIDR
          MapPublicIpOnLaunch: true
          Tags:
            - Key: Name
              # Subnet 이름
              Value: !Sub ${EnvironmentName} Public Subnet (AZ1)
    
      PublicSubnet2:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          AvailabilityZone: !Select [ 1, !GetAZs  '' ]
          CidrBlock: !Ref PublicSubnet2CIDR
          MapPublicIpOnLaunch: true
          Tags:
            - Key: Name
              # Subnet 이름
              Value: !Sub ${EnvironmentName} Public Subnet (AZ2)
    
      PrivateSubnet1:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          AvailabilityZone: !Select [ 0, !GetAZs  '' ]
          CidrBlock: !Ref PrivateSubnet1CIDR
          MapPublicIpOnLaunch: false
          Tags:
            - Key: Name
              # Subnet 이름
              Value: !Sub ${EnvironmentName} Private Subnet (AZ1)
    
      PrivateSubnet2:
        Type: AWS::EC2::Subnet
        Properties:
          VpcId: !Ref VPC
          AvailabilityZone: !Select [ 1, !GetAZs  '' ]
          CidrBlock: !Ref PrivateSubnet2CIDR
          MapPublicIpOnLaunch: false
          Tags:
            - Key: Name
              # Subnet 이름
              Value: !Sub ${EnvironmentName} Private Subnet (AZ2)
    
      NatGateway1EIP:
        Type: AWS::EC2::EIP
        DependsOn: InternetGatewayAttachment
        Properties:
          Domain: vpc
    
      NatGateway2EIP:
        Type: AWS::EC2::EIP
        DependsOn: InternetGatewayAttachment
        Properties:
          Domain: vpc
    
      NatGateway1:
        Type: AWS::EC2::NatGateway
        Properties:
          AllocationId: !GetAtt NatGateway1EIP.AllocationId
          SubnetId: !Ref PublicSubnet1
    
      NatGateway2:
        Type: AWS::EC2::NatGateway
        Properties:
          AllocationId: !GetAtt NatGateway2EIP.AllocationId
          SubnetId: !Ref PublicSubnet2
    
      PublicRouteTable:
        Type: AWS::EC2::RouteTable
        Properties:
          VpcId: !Ref VPC
          Tags:
            - Key: Name
              # Route Table Name
              Value: !Sub ${EnvironmentName} Public Routes
    
      DefaultPublicRoute:
        Type: AWS::EC2::Route
        DependsOn: InternetGatewayAttachment
        Properties:
          RouteTableId: !Ref PublicRouteTable
          DestinationCidrBlock: 0.0.0.0/0
          GatewayId: !Ref InternetGateway
    
      PublicSubnet1RouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          RouteTableId: !Ref PublicRouteTable
          SubnetId: !Ref PublicSubnet1
    
      PublicSubnet2RouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          RouteTableId: !Ref PublicRouteTable
          SubnetId: !Ref PublicSubnet2
    
      PrivateRouteTable1:
        Type: AWS::EC2::RouteTable
        Properties:
          VpcId: !Ref VPC
          Tags:
            - Key: Name
              # Route Table Name
              Value: !Sub ${EnvironmentName} Private Routes (AZ1)
    
      DefaultPrivateRoute1:
        Type: AWS::EC2::Route
        Properties:
          RouteTableId: !Ref PrivateRouteTable1
          DestinationCidrBlock: 0.0.0.0/0
          NatGatewayId: !Ref NatGateway1
    
      PrivateSubnet1RouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          RouteTableId: !Ref PrivateRouteTable1
          SubnetId: !Ref PrivateSubnet1
    
      PrivateRouteTable2:
        Type: AWS::EC2::RouteTable
        Properties:
          VpcId: !Ref VPC
          Tags:
            - Key: Name
              # Route Table Name
              Value: !Sub ${EnvironmentName} Private Routes (AZ2)
    
      DefaultPrivateRoute2:
        Type: AWS::EC2::Route
        Properties:
          RouteTableId: !Ref PrivateRouteTable2
          DestinationCidrBlock: 0.0.0.0/0
          NatGatewayId: !Ref NatGateway2
    
      PrivateSubnet2RouteTableAssociation:
        Type: AWS::EC2::SubnetRouteTableAssociation
        Properties:
          RouteTableId: !Ref PrivateRouteTable2
          SubnetId: !Ref PrivateSubnet2
    
      SecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
          GroupName: "mwaa-security-group"
          GroupDescription: "Security group with a self-referencing inbound rule."
          VpcId: !Ref VPC
    
      SecurityGroupIngress:
        Type: AWS::EC2::SecurityGroupIngress
        Properties:
          GroupId: !Ref SecurityGroup
          IpProtocol: "-1"
          SourceSecurityGroupId: !Ref SecurityGroup
    
      EnvironmentBucket:
        Type: AWS::S3::Bucket
        Properties:
          VersioningConfiguration:
            Status: Enabled
          PublicAccessBlockConfiguration: 
            BlockPublicAcls: true
            BlockPublicPolicy: true
            IgnorePublicAcls: true
            RestrictPublicBuckets: true
    
      #####################################################################################################################
      # CREATE MWAA
      #####################################################################################################################
    
      MwaaEnvironment:
        Type: AWS::MWAA::Environment
        DependsOn: MwaaExecutionPolicy
        Properties:
          # ${AWS::Region} = aws/config region
          # ${AWS::AccountId} = ??
          # {AWS::StackName} = command 시 stack-name 값
          # 알파벳으로 시작하여야 한다.
          Name: !Sub "${AWS::StackName}-MwaaEnvironment"
          SourceBucketArn: !GetAtt EnvironmentBucket.Arn
          ExecutionRoleArn: !GetAtt MwaaExecutionRole.Arn
          DagS3Path: dags
          NetworkConfiguration:
            SecurityGroupIds:
              - !GetAtt SecurityGroup.GroupId
            SubnetIds:
              - !Ref PrivateSubnet1
              - !Ref PrivateSubnet2
          WebserverAccessMode: PUBLIC_ONLY
          MaxWorkers: !Ref MaxWorkerNodes
          LoggingConfiguration:
            DagProcessingLogs:
              LogLevel: !Ref DagProcessingLogs
              Enabled: true
            SchedulerLogs:
              LogLevel: !Ref SchedulerLogsLevel
              Enabled: true
            TaskLogs:
              LogLevel: !Ref TaskLogsLevel
              Enabled: true
            WorkerLogs:
              LogLevel: !Ref WorkerLogsLevel
              Enabled: true
            WebserverLogs:
              LogLevel: !Ref WebserverLogsLevel
              Enabled: true
      SecurityGroup:
        Type: AWS::EC2::SecurityGroup
        Properties:
          VpcId: !Ref VPC
          GroupDescription: !Sub "Security Group for Amazon MWAA Environment ${AWS::StackName}-MwaaEnvironment"
          GroupName: !Sub "airflow-security-group-${AWS::StackName}-MwaaEnvironment"
      
      SecurityGroupIngress:
        Type: AWS::EC2::SecurityGroupIngress
        Properties:
          GroupId: !Ref SecurityGroup
          IpProtocol: "-1"
          SourceSecurityGroupId: !Ref SecurityGroup
    
      SecurityGroupEgress:
        Type: AWS::EC2::SecurityGroupEgress
        Properties:
          GroupId: !Ref SecurityGroup
          IpProtocol: "-1"
          CidrIp: "0.0.0.0/0"
    
      MwaaExecutionRole:
        Type: AWS::IAM::Role
        Properties:
          AssumeRolePolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Principal:
                  Service:
                    - airflow-env.amazonaws.com
                    - airflow.amazonaws.com
                Action:
                 - "sts:AssumeRole"
          Path: "/service-role/"
    
      MwaaExecutionPolicy:
        DependsOn: EnvironmentBucket
        Type: AWS::IAM::ManagedPolicy
        Properties:
          Roles:
            - !Ref MwaaExecutionRole
          PolicyDocument:
            Version: 2012-10-17
            Statement:
              - Effect: Allow
                Action: airflow:PublishMetrics
                Resource:
                  - !Sub "arn:aws:airflow:${AWS::Region}:${AWS::AccountId}:environment/${EnvironmentName}"
              - Effect: Deny
                Action: s3:ListAllMyBuckets
                Resource:
                  - !Sub "${EnvironmentBucket.Arn}"
                  - !Sub "${EnvironmentBucket.Arn}/*"
    
              - Effect: Allow
                Action:
                  - "s3:GetObject*"
                  - "s3:GetBucket*"
                  - "s3:List*"
                Resource:
                  - !Sub "${EnvironmentBucket.Arn}"
                  - !Sub "${EnvironmentBucket.Arn}/*"
              - Effect: Allow
                Action:
                  - logs:DescribeLogGroups
                Resource: "*"
    
              - Effect: Allow
                Action:
                  - logs:CreateLogStream
                  - logs:CreateLogGroup
                  - logs:PutLogEvents
                  - logs:GetLogEvents
                  - logs:GetLogRecord
                  - logs:GetLogGroupFields
                  - logs:GetQueryResults
                  - logs:DescribeLogGroups
                Resource:
                  - !Sub "arn:aws:logs:${AWS::Region}:${AWS::AccountId}:log-group:airflow-${AWS::StackName}*"
              - Effect: Allow
                Action: cloudwatch:PutMetricData
                Resource: "*"
              - Effect: Allow
                Action:
                  - sqs:ChangeMessageVisibility
                  - sqs:DeleteMessage
                  - sqs:GetQueueAttributes
                  - sqs:GetQueueUrl
                  - sqs:ReceiveMessage
                  - sqs:SendMessage
                Resource:
                  - !Sub "arn:aws:sqs:${AWS::Region}:*:airflow-celery-*"
              - Effect: Allow
                Action:
                  - kms:Decrypt
                  - kms:DescribeKey
                  - "kms:GenerateDataKey*"
                  - kms:Encrypt
                NotResource: !Sub "arn:aws:kms:*:${AWS::AccountId}:key/*"
                Condition:
                  StringLike:
                    "kms:ViaService":
                      - !Sub "sqs.${AWS::Region}.amazonaws.com"
    Outputs:
      VPC:
        Description: A reference to the created VPC
        Value: !Ref VPC
    
      PublicSubnets:
        Description: A list of the public subnets
        Value: !Join [ ",", [ !Ref PublicSubnet1, !Ref PublicSubnet2 ]]
    
      PrivateSubnets:
        Description: A list of the private subnets
        Value: !Join [ ",", [ !Ref PrivateSubnet1, !Ref PrivateSubnet2 ]]
    
      PublicSubnet1:
        Description: A reference to the public subnet in the 1st Availability Zone
        Value: !Ref PublicSubnet1
    
      PublicSubnet2:
        Description: A reference to the public subnet in the 2nd Availability Zone
        Value: !Ref PublicSubnet2
    
      PrivateSubnet1:
        Description: A reference to the private subnet in the 1st Availability Zone
        Value: !Ref PrivateSubnet1
    
      PrivateSubnet2:
        Description: A reference to the private subnet in the 2nd Availability Zone
        Value: !Ref PrivateSubnet2
    
      SecurityGroupIngress:
        Description: Security group with self-referencing inbound rule
        Value: !Ref SecurityGroupIngress
    
      MwaaApacheAirflowUI:
        Description: MWAA Environment
        Value: !Sub  "https://${MwaaEnvironment.WebserverUrl}"
    ```
    

##### 2-2) Create Stack with AWS CLI

- file:// 은 필수 값
- download file split data = `-` 이지만 코드 명령어는 `_`

```bash
$ cd file # move yml file
$ aws cloudformation create-stack --stack-name mwaa-environment-public-network --template-body file://mwaa_public_network.yml --capabilities CAPABILITY_IAM
```

##### 2-3) 결과

- 아래와 같이 설치된 것을 볼 수 있습니다.

![https://ifh.cc/g/RF070R.png](https://ifh.cc/g/RF070R.png)