CloudLaunch - AWS Cloud Infrastructure Assessment
Project Overview
CloudLaunch is a lightweight cloud platform that demonstrates the implementation of core AWS services including S3, IAM, and VPC. This project showcases secure static website hosting, private document storage with controlled access, and a well-architected network infrastructure.
Architecture Overview
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   CloudFront    │────│   S3 Website     │────│  Private S3     │
│  (HTTPS/Global) │    │     Bucket       │    │    Buckets      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │
                                │
        ┌───────────────────────┼───────────────────────┐
        │                   VPC │ (10.0.0.0/16)        │
        │  ┌─────────────────────┼─────────────────────┐ │
        │  │ Public Subnet       │                     │ │
        │  │ (10.0.1.0/24)      │                     │ │
        │  └─────────────────────┼─────────────────────┘ │
        │  ┌─────────────────────┼─────────────────────┐ │
        │  │ App Subnet          │                     │ │
        │  │ (10.0.2.0/24)      │                     │ │
        │  └─────────────────────┼─────────────────────┘ │
        │  ┌─────────────────────┼─────────────────────┐ │
        │  │ DB Subnet           │                     │ │
        │  │ (10.0.3.0/28)      │                     │ │
        │  └─────────────────────┼─────────────────────┘ │
        └───────────────────────┼───────────────────────┘
                                │
                    ┌───────────┼───────────┐
                    │  Internet │ Gateway   │
                    │   (IGW)   │           │
                    └───────────┼───────────┘
                                │
                            Internet

Task 1: Static Website Hosting with S3 and IAM
Implementation Summary
Successfully implemented a three-tier S3 bucket architecture with granular IAM permissions for secure document management and static website hosting.
S3 Buckets Created
1. cloudlaunch-site-bucket2 (Public Website)

Purpose: Hosts the static website
Access: Public read-only access
Features:

Static website hosting enabled
Custom error pages configured
CloudFront distribution for global CDN



2. cloudlaunch-private-bucket2 (Private Documents)

Purpose: Secure document storage
Access: Restricted to cloudlaunch-user (GetObject, PutObject only)
Security: No public access, no delete permissions

3. cloudlaunch-visible-only-bucket0 (List-Only Access)

Purpose: Demonstrates granular permission controls
Access: cloudlaunch-user can list bucket but cannot access contents

IAM User Configuration
Username: cloudlaunch-user
Permissions Summary:

✅ ListBucket on all three buckets
✅ GetObject on site bucket (read website files)
✅ GetObject + PutObject on private bucket (manage documents)
✅ No DeleteObject permissions anywhere (security control)
✅ No content access to visible-only bucket
✅ VPC read-only access for monitoring

Access Links
Static Website: http://cloudlaunch-site-bucket2.s3-website.eu-north-1.amazonaws.com
CloudFront URL: https://d14wpaxhovu1be.cloudfront.net
Custom IAM Policy
json{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListAllBuckets",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::cloudlaunch-site-bucket2",
                "arn:aws:s3:::cloudlaunch-private-bucket2",
                "arn:aws:s3:::cloudlaunch-visible-only-bucket0"
            ]
        },
        {
            "Sid": "GetObjectSiteBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-site-bucket2/*"
        },
        {
            "Sid": "GetPutObjectPrivateBucket",
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject"
            ],
            "Resource": "arn:aws:s3:::cloudlaunch-private-bucket2/*"
        },
        {
            "Sid": "VPCReadOnlyAccess",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeVpcs",
                "ec2:DescribeSubnets",
                "ec2:DescribeRouteTables",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeInternetGateways"
            ],
            "Resource": "*"
        }
    ]
}

Task 2: VPC Network Architecture
Implementation Summary
Designed and implemented a secure, scalable VPC architecture following AWS best practices for network segmentation and security.
VPC Configuration
VPC Name: cloudlaunch-vpc
CIDR Block: 10.0.0.0/16
Region: eu-north-1
Subnet Architecture
Subnet NameCIDR BlockTypePurposecloudlaunch-public-subnet10.0.1.0/24PublicLoad balancers, public servicescloudlaunch-app-subnet10.0.2.0/24PrivateApplication serverscloudlaunch-db-subnet10.0.3.0/28PrivateDatabase services
Network Components
Internet Gateway

Name: cloudlaunch-igw
Purpose: Enables internet access for public subnet

Route Tables

cloudlaunch-public-rt

Routes: 0.0.0.0/0 → Internet Gateway
Associated: Public subnet
Purpose: Internet access for public resources


cloudlaunch-app-rt

Routes: Local VPC traffic only
Associated: App subnet
Purpose: Private communication within VPC


cloudlaunch-db-rt

Routes: Local VPC traffic only
Associated: Database subnet
Purpose: Maximum security isolation



Security Groups
1. cloudlaunch-app-sg
Inbound Rules:
- HTTP (80) from VPC CIDR (10.0.0.0/16)
- Purpose: Allow internal web traffic
2. cloudlaunch-db-sg
Inbound Rules:
- MySQL (3306) from App Subnet (10.0.2.0/24)
- Purpose: Database access from app tier only

AWS Account Access
Console Access

Console URL: https://124355637901.signin.aws.amazon.com/console
Account ID: 124355637901
Username: cloudlaunch-user
Password: CloudLaunch2024! (User must change on first login)

Programmatic Access

Access Key ID: Available upon request (not stored in repository for security)
Secret Access Key: Available upon request (not stored in repository for security)


Security Best Practices Implemented
Access Control

✅ Principle of least privilege for IAM user
✅ No unnecessary delete permissions
✅ Bucket-specific access controls
✅ Network-level security with security groups

Network Security

✅ Private subnets with no internet access
✅ Layered security approach (public → app → database)
✅ Restricted port access based on service requirements

Data Protection

✅ Private buckets with encryption in transit
✅ Public bucket limited to website files only
✅ CloudFront for HTTPS encryption and DDoS protection


Testing and Verification
S3 Bucket Access Testing

Public Website: Accessible via browser without authentication
Private Bucket: Accessible only with cloudlaunch-user credentials
Visible-Only Bucket: Listed in S3 console but contents not accessible

IAM Permission Testing

Login as cloudlaunch-user
Verify limited S3 access permissions
Confirm VPC read-only access
Test that delete operations are denied

Network Connectivity

Public Subnet: Has route to internet via IGW
Private Subnets: No internet access, local VPC communication only
Security Groups: Correct port restrictions in place


Cost Optimization
All resources implemented are within AWS Free Tier limits:

S3: Standard storage, minimal data transfer
CloudFront: 50GB data transfer and 2M requests/month free
VPC: No charges for basic VPC components
IAM: No charges for users and policies

Estimated Monthly Cost: $0.00 (within free tier limits)
