# CloudLaunch - AWS Infrastructure Project

This repository contains the implementation of CloudLaunch, a lightweight platform showcasing a basic company website with secure storage for internal documents using AWS core services (S3, IAM, and VPC).

---

## Task 1: Static Website Hosting Using S3 + IAM User with Limited Permissions

### S3 Buckets Configuration

I created three S3 buckets with different access levels:

#### 1. cloudlaunch-site-bucket-john
- Configured for static website hosting
- Contains HTML and CSS resources
- Publicly accessible (read-only)
- **Website URL**: [http://cloudlaunch-site-bucket-john.s3-website-us-east-1.amazonaws.com](http://cloudlaunch-site-bucket-john.s3-website-us-east-1.amazonaws.com)

#### 2. loudlaunch-private-bucket-alex2024
- Not publicly accessible
- IAM user has GetObject and PutObject permissions only
- No delete permissions

#### 3. cloudlaunch-visible-only-bucket-alex2024
- Not publicly accessible
- IAM user can only list the bucket but cannot access its contents

### CloudFront Distribution (Bonus)
Not configured (optional requirement)

### IAM User and Policy

Created an IAM user named `cloudlaunch-user` with limited permissions to the S3 buckets. The user can:

- List all three buckets
- GetObject/PutObject on private bucket
- GetObject on site bucket only
- No DeleteObject permissions anywhere
- No access to visible-only bucket contents
- Read-only access to VPC resources

### IAM Policy JSON
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": [
                "arn:aws:s3:::cloudlaunch-site-bucket-john",
                "arn:aws:s3:::loudlaunch-private-bucket-alex2024",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket-alex2024"
            ]
        },
        {
            "Sid": "ReadSiteBucket",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::cloudlaunch-site-bucket-john/*"
        },
        {
            "Sid": "ReadWritePrivateBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::loudlaunch-private-bucket-alex2024/*"
        }
    ]
}
```

**Additional Policy Attached**: `AmazonVPCReadOnlyAccess` (AWS Managed Policy)

---

## Task 2: VPC Design for CloudLaunch Environment

Created a secure, logically separated network environment with the following components:

### VPC Configuration
- **VPC Name**: cloudlaunch-vpc
- **VPC ID**: vpc-07ce9aa167f3e05e8
- **CIDR Block**: 10.0.0.0/16
- **Region**: us-east-1

### Subnet Configuration

#### Public Subnet
- **Name**: cloudlaunch-public-subnet
- **CIDR**: 10.0.1.0/24
- **Availability Zone**: us-east-1a (or your chosen AZ)
- **Purpose**: For future load balancers/public-facing services

#### Application Subnet
- **Name**: cloudlaunch-app-subnet
- **CIDR**: 10.0.2.0/24
- **Availability Zone**: us-east-1a
- **Purpose**: For application servers (private)

#### Database Subnet
- **Name**: cloudlaunch-db-subnet
- **CIDR**: 10.0.3.0/28
- **Availability Zone**: us-east-1a
- **Purpose**: For database services (private)

### Internet Gateway
- **Name**: cloudlaunch-igw
- **Status**: Attached to cloudlaunch-vpc
- **Purpose**: Provides internet access to public subnet

### Route Tables

#### Public Route Table (cloudlaunch-public-rt)
- **Associated with**: cloudlaunch-public-subnet
- **Routes**:
  - 10.0.0.0/16 → Local
  - 0.0.0.0/0 → cloudlaunch-igw (Internet Gateway)

#### Application Route Table (cloudlaunch-app-rt)
- **Associated with**: cloudlaunch-app-subnet
- **Routes**:
  - 10.0.0.0/16 → Local
  - No internet access (fully private)

#### Database Route Table (cloudlaunch-db-rt)
- **Associated with**: cloudlaunch-db-subnet
- **Routes**:
  - 10.0.0.0/16 → Local
  - No internet access (fully private)

### Security Groups

#### cloudlaunch-app-sg
- **Description**: Allow HTTP access within VPC only
- **Inbound Rules**:
  - Type: HTTP (80), Source: 10.0.0.0/16 (VPC CIDR)
- **Outbound Rules**:
  - Default: All traffic

#### cloudlaunch-db-sg
- **Description**: Allow MySQL access from app subnet only
- **Inbound Rules**:
  - Type: MySQL/Aurora (3306), Source: 10.0.2.0/24 (App subnet)
- **Outbound Rules**:
  - Default: All traffic

### IAM VPC Read-Only Permissions

The `cloudlaunch-user` has been granted read-only access to VPC resources through the `AmazonVPCReadOnlyAccess` managed policy, allowing them to:
- View VPCs, subnets, route tables, and security groups
- Cannot modify or delete any VPC resources

---

## Security Best Practices Implemented

✅ **Principle of Least Privilege**: IAM policies grant only necessary permissions

✅ **Network Segmentation**: Separate subnets for different tiers (public, app, database)

✅ **Security Groups**: Restrictive inbound/outbound rules based on requirements

✅ **Private Subnets**: Application and database subnets have no direct internet access

✅ **No Delete Permissions**: IAM user cannot delete any S3 objects

✅ **Password Change Required**: User must change password on first login

---

## Architecture Benefits

- **High Availability**: Resources distributed across availability zones
- **Scalability**: VPC design supports future expansion
- **Security**: Multi-layered security with network and application-level controls
- **Cost-Effective**: All resources within AWS Free Tier limits

---

## Technical Specifications

- **Frontend**: HTML, CSS
- **Hosting**: AWS S3 Static Website Hosting
- **Network**: Custom AWS VPC with public and private subnets
- **Security**: IAM-based access control with custom policies
- **Region**: US East (N. Virginia) - us-east-1
- **Cost**: Free Tier eligible (no EC2, NAT Gateway, or RDS used)

---

## Project Status

✅ Task 1 completed - S3 static website live with IAM access controls

✅ Task 2 completed - VPC network architecture fully configured

✅ All security controls implemented

✅ Within AWS Free Tier limits

---

## Notes

- All resources deployed in **us-east-1** region
- No paid AWS resources used (EC2, NAT Gateway, RDS, etc.)
- VPC architecture designed for future expansion
- **Important**: Resources will be deleted after evaluation to avoid AWS charges

---
