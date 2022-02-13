---
layout: post
title: "AWS CloudFormation을 이용하여 템플릿 구축하기"
date: "2022-02-13 13:03:44 +0900"
tags: [aws]
---

![Untitled](https://i.imgur.com/Bv0VkSb.png)

## About

AWS CloudFormation을 AWS 인프라를 json 혹은 yaml 형식의 템플릿으로 저장하여 쉽게 리소스를 생성 및 관리할 수 있는 서비스입니다. 매번 새로운 프로젝트를 시작할 때 마다 거의 유사한 구조의 인프라를 구축하게 되는데 이럴 경우 AWS CloudFormation을 이용하면 훨씬 쉽게 구축할 수 있습니다.

### Infrastructure

![Infra](https://i.imgur.com/Pt3Da7G.png)

이 글에서는 위 사진과 같은 인프라를 yaml 형식으로 템플릿을 구축 할 것이고 이를 CloudFormation을 이용하여 구축할 것입니다.

yaml 파일은 다음과 같습니다.

[sweatpotato13/aws_vpc_with_private_and_public_subnets-2az.yaml](https://gist.githubusercontent.com/sweatpotato13/54d918f344606732e815aeb01de41c3e/raw/d07abe5cd380a6cd3b2aa335f6fe4b5ade781d2e/aws_vpc_with_private_and_public_subnets-2az.yaml)

```yaml
Parameters:
  AZ1:
    Description: AvailabilityZone for first zone
    Type: 'AWS::EC2::AvailabilityZone::Name'
  AZ2:
    Description: AvailabilityZone for second zone
    Type: 'AWS::EC2::AvailabilityZone::Name'
  VPCCidr:
    Description: Cidr Block for VPC
    Type: String
    Default: 10.0.0.0/16
  PublicSubnet1Cidr:
    Description: Cidr Block for Public Subnet
    Type: String
    Default: 10.0.0.0/24
  PublicSubnet2Cidr:
    Description: Cidr Block for Public Subnet
    Type: String
    Default: 10.0.1.0/24
  PrivateSubnet1Cidr:
    Description: Cidr Block for Private Subnet
    Type: String
    Default: 10.0.2.0/24
  PrivateSubnet2Cidr:
    Description: Cidr Block for Private Subnet
    Type: String
    Default: 10.0.3.0/24
    
Resources:
  PubPrivateVPC:
    Type: 'AWS::EC2::VPC'
    Properties:
      CidrBlock: !Ref VPCCidr
    Metadata:
      'AWS::CloudFormation::Designer':
        id: 97b1431f-6d39-4bd1-a3b5-311010ec22bf
  PublicSubnet1:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref PubPrivateVPC
      CidrBlock: !Ref PublicSubnet1Cidr
      AvailabilityZone: !Ref AZ1
      MapPublicIpOnLaunch: true
    Metadata:
      'AWS::CloudFormation::Designer':
        id: d0a2117d-e90d-40ed-8c2e-6a6a6182ba05
  PublicSubnet2:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref PubPrivateVPC
      CidrBlock: !Ref PublicSubnet2Cidr
      AvailabilityZone: !Ref AZ2
      MapPublicIpOnLaunch: true
    Metadata:
      'AWS::CloudFormation::Designer':
        id: de6cc68c-416f-47e4-af01-03c277c93d9a
  PrivateSubnet1:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref PubPrivateVPC
      CidrBlock: !Ref PrivateSubnet1Cidr
      AvailabilityZone: !Ref AZ1
      MapPublicIpOnLaunch: false
    Metadata:
      'AWS::CloudFormation::Designer':
        id: e0ad3ab0-b054-42b4-b7b3-5d49d84e6bb7
  PrivateSubnet2:
    Type: 'AWS::EC2::Subnet'
    Properties:
      VpcId: !Ref PubPrivateVPC
      CidrBlock: !Ref PrivateSubnet2Cidr
      AvailabilityZone: !Ref AZ2
      MapPublicIpOnLaunch: false
    Metadata:
      'AWS::CloudFormation::Designer':
        id: 1ef4bbe2-35ff-4f29-9c03-d9c9ddc4cf6c
  InternetGateway:
    Type: 'AWS::EC2::InternetGateway'
    Metadata:
      'AWS::CloudFormation::Designer':
        id: 31711ec4-075c-492e-b9ba-0f2e113a5577
  GatewayToInternet:
    Type: 'AWS::EC2::VPCGatewayAttachment'
    Properties:
      VpcId: !Ref PubPrivateVPC
      InternetGatewayId: !Ref InternetGateway
    Metadata:
      'AWS::CloudFormation::Designer':
        id: 07928be6-1dd2-4eb7-aabf-02645754ede1
  PublicRouteTable:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref PubPrivateVPC
    Metadata:
      'AWS::CloudFormation::Designer':
        id: 1d527e87-e0dd-4359-8f3c-ae43c0ffcc34
  PublicRoute:
    Type: 'AWS::EC2::Route'
    DependsOn: GatewayToInternet
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway
    Metadata:
      'AWS::CloudFormation::Designer':
        id: 3272e7d7-11ed-4ef5-9809-b90f1e29cd5f
  PublicSubnet1RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PublicSubnet1
      RouteTableId: !Ref PublicRouteTable
  PublicSubnet2RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable
  NatGateway:
    Type: 'AWS::EC2::NatGateway'
    DependsOn: NatPublicIP
    Properties:
      SubnetId: !Ref PublicSubnet1
      AllocationId: !GetAtt NatPublicIP.AllocationId
    Metadata:
      'AWS::CloudFormation::Designer':
        id: 7b09d95e-fab4-458c-9c53-cca2b36d0bd7
  NatPublicIP:
    Type: 'AWS::EC2::EIP'
    DependsOn: PubPrivateVPC
    Properties:
      Domain: vpc
    Metadata:
      'AWS::CloudFormation::Designer':
        id: ff7bb56d-c4d6-42dd-bf88-7f6ac5634f0a
  PrivateRouteTable:
    Type: 'AWS::EC2::RouteTable'
    Properties:
      VpcId: !Ref PubPrivateVPC
    Metadata:
      'AWS::CloudFormation::Designer':
        id: ff36d740-d6e1-4260-a2e2-ca47f92ab6e0
  PrivateRoute:
    Type: 'AWS::EC2::Route'
    Properties:
      NatGatewayId: !Ref NatGateway
      RouteTableId: !Ref PrivateRouteTable
      DestinationCidrBlock: 0.0.0.0/0
    Metadata:
      'AWS::CloudFormation::Designer':
        id: ffb3b98f-02f3-426a-a94e-644bf476355d
  PrivateSubnet1RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PrivateSubnet1
      RouteTableId: !Ref PrivateRouteTable
  PrivateSubnet2RouteTableAssociation:
    Type: 'AWS::EC2::SubnetRouteTableAssociation'
    Properties:
      SubnetId: !Ref PrivateSubnet2
      RouteTableId: !Ref PrivateRouteTable
Metadata:
  'AWS::CloudFormation::Designer':
    31711ec4-075c-492e-b9ba-0f2e113a5577:
      size:
        width: 60
        height: 60
      position:
        x: 720
        'y': 690
      z: 1
      embeds: []
    97b1431f-6d39-4bd1-a3b5-311010ec22bf:
      size:
        width: 600
        height: 600
      position:
        x: 60
        'y': 90
      z: 1
      embeds:
        - 1ef4bbe2-35ff-4f29-9c03-d9c9ddc4cf6c
        - e0ad3ab0-b054-42b4-b7b3-5d49d84e6bb7
        - de6cc68c-416f-47e4-af01-03c277c93d9a
        - d0a2117d-e90d-40ed-8c2e-6a6a6182ba05
    ff36d740-d6e1-4260-a2e2-ca47f92ab6e0:
      size:
        width: 240
        height: 240
      position:
        x: 720
        'y': 390
      z: 1
      embeds:
        - ffb3b98f-02f3-426a-a94e-644bf476355d
    ff7bb56d-c4d6-42dd-bf88-7f6ac5634f0a:
      size:
        width: 60
        height: 60
      position:
        x: 840
        'y': 690
      z: 1
      embeds: []
      dependson:
        - 97b1431f-6d39-4bd1-a3b5-311010ec22bf
    1d527e87-e0dd-4359-8f3c-ae43c0ffcc34:
      size:
        width: 240
        height: 240
      position:
        x: 720
        'y': 90
      z: 1
      embeds:
        - 3272e7d7-11ed-4ef5-9809-b90f1e29cd5f
    07928be6-1dd2-4eb7-aabf-02645754ede1:
      source:
        id: 97b1431f-6d39-4bd1-a3b5-311010ec22bf
      target:
        id: 31711ec4-075c-492e-b9ba-0f2e113a5577
      z: 1
    3272e7d7-11ed-4ef5-9809-b90f1e29cd5f:
      size:
        width: 60
        height: 60
      position:
        x: 750
        'y': 150
      z: 2
      parent: 1d527e87-e0dd-4359-8f3c-ae43c0ffcc34
      embeds: []
      isassociatedwith:
        - 31711ec4-075c-492e-b9ba-0f2e113a5577
      iscontainedinside:
        - 1d527e87-e0dd-4359-8f3c-ae43c0ffcc34
        - 1d527e87-e0dd-4359-8f3c-ae43c0ffcc34
      dependson:
        - 07928be6-1dd2-4eb7-aabf-02645754ede1
    1ef4bbe2-35ff-4f29-9c03-d9c9ddc4cf6c:
      size:
        width: 150
        height: 150
      position:
        x: 90
        'y': 450
      z: 2
      parent: 97b1431f-6d39-4bd1-a3b5-311010ec22bf
      embeds: []
      iscontainedinside:
        - 97b1431f-6d39-4bd1-a3b5-311010ec22bf
        - 97b1431f-6d39-4bd1-a3b5-311010ec22bf
    e0ad3ab0-b054-42b4-b7b3-5d49d84e6bb7:
      size:
        width: 150
        height: 150
      position:
        x: 390
        'y': 360
      z: 2
      parent: 97b1431f-6d39-4bd1-a3b5-311010ec22bf
      embeds: []
      iscontainedinside:
        - 97b1431f-6d39-4bd1-a3b5-311010ec22bf
        - 97b1431f-6d39-4bd1-a3b5-311010ec22bf
    de6cc68c-416f-47e4-af01-03c277c93d9a:
      size:
        width: 150
        height: 150
      position:
        x: 390
        'y': 150
      z: 2
      parent: 97b1431f-6d39-4bd1-a3b5-311010ec22bf
      embeds: []
      iscontainedinside:
        - 97b1431f-6d39-4bd1-a3b5-311010ec22bf
        - 97b1431f-6d39-4bd1-a3b5-311010ec22bf
    d0a2117d-e90d-40ed-8c2e-6a6a6182ba05:
      size:
        width: 240
        height: 240
      position:
        x: 90
        'y': 150
      z: 2
      parent: 97b1431f-6d39-4bd1-a3b5-311010ec22bf
      embeds:
        - 7b09d95e-fab4-458c-9c53-cca2b36d0bd7
      iscontainedinside:
        - 97b1431f-6d39-4bd1-a3b5-311010ec22bf
        - 97b1431f-6d39-4bd1-a3b5-311010ec22bf
    7b09d95e-fab4-458c-9c53-cca2b36d0bd7:
      size:
        width: 60
        height: 60
      position:
        x: 120
        'y': 210
      z: 3
      parent: d0a2117d-e90d-40ed-8c2e-6a6a6182ba05
      embeds: []
      iscontainedinside:
        - d0a2117d-e90d-40ed-8c2e-6a6a6182ba05
        - d0a2117d-e90d-40ed-8c2e-6a6a6182ba05
    ffb3b98f-02f3-426a-a94e-644bf476355d:
      size:
        width: 60
        height: 60
      position:
        x: 750
        'y': 450
      z: 2
      parent: ff36d740-d6e1-4260-a2e2-ca47f92ab6e0
      embeds: []
      isassociatedwith:
        - 7b09d95e-fab4-458c-9c53-cca2b36d0bd7
      iscontainedinside:
        - ff36d740-d6e1-4260-a2e2-ca47f92ab6e0
        - ff36d740-d6e1-4260-a2e2-ca47f92ab6e0
```

### CloudFormation

* AWS CloudFormation의 스택 탭으로 가서 새 리소스를 생성합니다.

![1](https://i.imgur.com/lYVr9b3.png)

* 템플릿을 불러옵니다.

S3에 저장되어 있다면 S3를 통하여 불러올 수 있고, 혹은 파일을 직접 업로드 할 수도 있습니다. 저는 직접 파일을 업로드 하였습니다.

![2](https://i.imgur.com/7Jg5ZdB.png)

* 스택 이름과 파라미터를 지정합니다.

제가 위에 올린 템플릿을 이용하면 아래 사진과 같은 파라미터를 볼 수 있습니다.

az1과 az2는 2개의 zone에 대한 이름을 지정합니다.

Subnet CIDR 및 VPC CIDR은 각각의 zone에 대한 CIDR을 지정합니다. CIDR값은 Default로 작성되어 있는 값을 그대로 사용하여도 좋습니다.

![3](https://i.imgur.com/WptGSlK.png)

* 스택 옵션을 구성합니다.

CloudFormation을 이용하여 리소스를 생성할 수 있도록 권한을 주어야 합니다. 기본적으로 사진과 같은 권한이 생성되어 있을 것 입니다.

또한 스택을 생성하다가 실패했을 시 어떤 행동을 취할건지 정할 수 있는데, 저는 해당 템플릿을 이용하여 생성한 모든 리소스를 롤백 하는 것으로 설정하였습니다.

![4](https://i.imgur.com/H3EMpys.png)


여기까지 진행하셨다면 템플릿에 맞는 리소스가 생성 되었습니다.

## 마무리

이 글에서 CloudFormation을 사용하여 AWS 리소스를 생성하는 방법을 소개하였습니다. 이 글에서는 네트워크 인프라만 생성하고 EC2 및 EKS 등의 서비스는 생성하지 않았습니다. 본인의 필요에 따라서 템플릿을 수정하여 사용하시면 됩니다. yaml 작성 형식은 아래 링크를 참조하세요

[AWS CloudFormation 템플릿 형식](https://docs.aws.amazon.com/ko_kr/AWSCloudFormation/latest/UserGuide/template-formats.html)