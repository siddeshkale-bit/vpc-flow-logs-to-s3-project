# Project: Configure VPC Flow Logs and Store Logs in S3 Using IAM Role

## Overview
This project demonstrates how to enable VPC Flow Logs for a VPC and store those logs in an Amazon S3 bucket using a custom IAM role.  
The logs help with network monitoring, security auditing, and traffic analysis.

---

## Architecture Components
- **VPC**: vpc-07fdda9051cff5a82  
- **S3 Bucket**: vpc-flow-logs-bucket-25  
- **IAM Role**: VPCFlowLogsToS3Role  
- **Destination**: Amazon S3  
- **Region**: us-east-1 (N. Virginia)

---

## Steps Performed

### 1. Create EC2 Instance (for generating traffic)
- Launched a t2.micro instance in the `us-east-1` region.
- Used default VPC networking.
- Instance traffic helps generate flow logs.

---

### 2. Create S3 Bucket
- Bucket name: **vpc-flow-logs-bucket-25**
- Region: **us-east-1**
- Block Public Access: Enabled (recommended)
- AWS automatically added a resource policy when Flow Logs were created.

---

### 3. Create IAM Role (VPCFlowLogsToS3Role)
A custom IAM role was created with:
- **Trust policy** allowing VPC Flow Logs service to assume the role.

### Trust Relationship JSON:
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Principal": {
"Service": "vpc-flow-logs.amazonaws.com"
},
"Action": "sts:AssumeRole"
}
]
}

### Permissions Policy (Custom Managed Policy)
{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": "s3:PutObject",
"Resource": "arn:aws:s3:::vpc-flow-logs-bucket-25/890742569078/us-east-1/vpc-07fdda9051cff5a82/*"
}
]
}


Policy name: **VPCFlowLogsS3PutObjectPolicy**

---

### 4. Create VPC Flow Log
Applied to:
- **VPC:** vpc-07fdda9051cff5a82

Settings:
- **Filter:** ALL  
- **Destination:** Amazon S3 bucket  
- **Bucket ARN:** `arn:aws:s3:::vpc-flow-logs-bucket-25`  
- **IAM Role:** VPCFlowLogsToS3Role  
- **Log Format:** Text (Default)

Flow Log status became **Active**.

---

### 5. Verify Logs in S3
Inside the bucket, logs appeared in auto-created folders:

AWSLogs/
└── 890742569078/
└── vpcflowlogs/
└── us-east-1/
└── 2025/12/01/
└── <logfiles>.log.gz


Downloaded and extracted `.log.gz` files to view sample entries.

---

## Sample Log Entry and Explanation

### Raw Log Line:
2 890742569078 eni-063ffc72aec32784c 167.94.146.46 172.31.22.45 22072 49089 6 1 60 1764585588 1764585601 REJECT OK

### Explanation:

| Field | Meaning |
|-------|---------|
| 2 | Log format version |
| 890742569078 | AWS account ID |
| eni-063ffc72aec32784c | Network interface (ENI) that observed the traffic |
| 167.94.146.46 | Source IP (external) |
| 172.31.22.45 | Destination IP inside VPC |
| 22072 | Source port |
| 49089 | Destination port |
| 6 | Protocol (6 = TCP) |
| 1 | Number of packets |
| 60 | Number of bytes |
| 1764585588 | Start time (epoch) |
| 1764585601 | End time (epoch) |
| REJECT | Traffic was denied by SG/NACL |
| OK | Log successfully delivered |

---

## Why "ALL" Traffic Type Was Used?
Selecting **All** captures:
- Accepted traffic  
- Rejected traffic  
- Unexpected or malicious traffic  

This gives complete visibility for security and troubleshooting.

---

## Deliverables Included
- IAM Role trust relationship screenshot  
- IAM policy JSON  
- S3 bucket screenshot showing logs  
- Flow Log configuration screenshot  
- Sample log and explanation  
- README.md (this file)

---

## Conclusion
The project successfully demonstrates:
- Creation of an IAM role for VPC Flow Logs  
- Delivery of VPC logs to an S3 bucket  
- Verification of network flows  
- Understanding of log formats and traffic behavior  

This setup is essential for auditing and analyzing VPC traffic securely and efficiently.
