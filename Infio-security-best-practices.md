# Infio security best practices

## Table of Contents
- [Overview](#overview)
- [EC2 Image Security](#ec2-image-security)
  - [AMI Hardening](#ami-hardening)
  - [Security Updates](#security-updates)
  - [Custom Scripts Security](#custom-scripts-security)
- [Network Security](#network-security)
  - [VPC and Subnet Isolation](#vpc-and-subnet-isolation)
  - [Security Groups](#security-groups)
  - [Private Key Encryption](#private-key-encryption)
- [Data Security](#data-security)
  - [No Persistent Storage](#no-persistent-storage)
  - [Encryption in Transit](#encryption-in-transit)
  - [Encryption at Rest](#encryption-at-rest)
- [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
  - [Principle of Least Privilege](#principle-of-least-privilege)
  - [Customer IAM Roles](#customer-iam-roles)
- [Logging and Monitoring](#logging-and-monitoring)
  - [Activity Logging](#activity-logging)
- [Vulnerability Management](#vulnerability-management)
  - [Regular Scans](#regular-scans)
  - [Patch Management](#patch-management)
- [Customer Responsibilities](#customer-responsibilities)
  - [Securing VPC](#securing-vpc)
  - [Applying IAM Best Practices](#applying-iam-best-practices)
  - [Data Backup](#data-backup)
- [Secure Storage and Key Management](#secure-storage-and-key-management)
  - [AWS Key Management Service (KMS)](#aws-key-management-service-kms)
  - [Secrets Management](#secrets-management)
- [Frequently Asked Questions (FAQ)](#frequently-asked-questions-faq)
  - [How does INFIO ensure secure communication with my SQL Server?](#how-does-infio-ensure-secure-communication-with-my-sql-server)
  - [How is the EC2 instance secured from external access?](#how-is-the-ec2-instance-secured-from-external-access)
  - [Can I access the EC2 instance via RDP?](#can-i-access-the-ec2-instance-via-rdp)
  - [What kind of data is stored on the EC2 instance?](#what-kind-of-data-is-stored-on-the-ec2-instance)
  - [Are logs from the EC2 instance available to me?](#are-logs-from-the-ec2-instance-available-to-me)
  - [How is access to the tool controlled?](#how-is-access-to-the-tool-controlled)
  - [What happens if vulnerabilities are discovered in the tool?](#what-happens-if-vulnerabilities-are-discovered-in-the-tool)
  - [How can I ensure that my VPC and EC2 instance are secure?](#how-can-i-ensure-that-my-vpc-and-ec2-instance-are-secure)
  - [How do you ensure that the AMI is secure before deployment?](#how-do-you-ensure-that-the-ami-is-secure-before-deployment)
  - [Can I update the software on the EC2 instance?](#can-i-update-the-software-on-the-ec2-instance)
  - [What if I need additional security measures for my use case?](#what-if-i-need-additional-security-measures-for-my-use-case)
  - [Can I use my own encryption keys for data at rest?](#can-i-use-my-own-encryption-keys-for-data-at-rest)
  - [How are my credentials stored and secured?](#how-are-my-credentials-stored-and-secured)
  - [How do I update to the latest version of the tool?](#how-do-i-update-to-the-latest-version-of-the-tool)
  - [Can the tool be accessed by third parties?](#can-the-tool-be-accessed-by-third-parties)
  - [What's the required runtime for INFIO in the VPC to finalize the assessment?](#whats-the-required-runtime-for-infio-in-the-vpc-to-finalize-the-assessment)
  - [In case INFIO needs to run for an extended period, will there be any antivirus software on the AMI?](#in-case-infio-needs-to-run-for-an-extended-period-will-there-be-any-antivirus-software-on-the-ami)
  - [Is there a file that is generated as part of the online INFIO assessment process? What information is captured as part of the output file?](#is-there-a-file-that-is-generated-as-part-of-the-online-infio-assessment-process-what-information-is-captured-as-part-of-the-output-file)



---

## Overview

The INFIO Tool is a Windows-based AMI designed to streamline the process of assessing SQL Server databases for migration. Deployed within the customer's AWS Virtual Private Cloud (VPC), the tool runs on an EC2 instance configured with Python, Babelfish COMPASS, PostgreSQL, and custom scripts.  

It securely connects to the customer’s SQL Server hosted on the same private subnet, performing comprehensive compatibility checks and analysis. The tool ensures robust security with encryption for data in transit and at rest, strict IAM role policies, and regular patching.  

Customers can launch the AMI in their own VPC, ensuring full control and data privacy throughout the assessment process.  

The security of INFIO is of the highest priority. Below are the key security best practices applied during the development, deployment, and use of the tool.

---

## EC2 Image Security

### AMI Hardening

- The AMI (Amazon Machine Image) provided with INFIO is built following AWS and industry best practices for hardening:
  - All unnecessary services and ports have been disabled to reduce the attack surface.
  - Only essential software components (Python, Compass, PostgreSQL, and custom scripts) are installed.
  - Admin privileges should be restricted and granted only to authorized users and following the principle of least privilege (PoLP) access.
  - Firewall rules (Security Groups) are configured to block all incoming traffic except those explicitly allowed.

### Security Updates

- The AMI includes the latest security patches for all pre-installed software. Regular updates are encouraged to ensure your EC2 instance remains secure.
- Customers are notified when new security updates are available.
- To update the instance, use AWS Systems Manager for automated patch management or manually apply patches.

### Custom Scripts Security

- The custom scripts embedded in the AMI are written following secure coding guidelines to protect against common vulnerabilities, such as SQL injection, code injection, and cross-site scripting (XSS).
- Input sanitization, output encoding, and proper exception handling are implemented across all scripts.

---

## Network Security

### VPC and Subnet Isolation

- The EC2 instance is deployed within the customer’s VPC, ensuring that the instance operates within a highly secure, isolated network environment.
- The EC2 instance is provisioned in a private subnet, meaning no direct internet access is allowed. Access to and from the instance can only occur through internal AWS network connections or VPN gateways, depending on the VPC configuration.
- Direct access to your Source SQL Server is facilitated via the private subnet, preventing any public exposure of the database.
- If applicable, VPC peering ensures secure data transfer between the EC2 instance and the customer’s SQL Server database on the same subnet or across peered VPCs.

### Security Groups

- Security Groups control both inbound and outbound traffic to the EC2 instance. These groups follow a whitelist model—only allowing traffic from trusted IP ranges or specific services (e.g., your SQL Server).
- For inbound traffic, access is restricted to necessary ports, such as:
  - Port 3389 (RDP) is restricted to specific internal IPs for secure remote management.
  - Port 443/80 (HTTP/S) are restricted to private IP ranges within the same subnet.
- Outbound traffic is limited to essential services, and any unwanted outgoing communication is blocked.

### Private Key Encryption

- Access to the EC2 instance is restricted to those with a key pair. The private key is securely stored by the customer, and access logs are reviewed periodically to detect any unauthorized access attempts.

---

## Data Security

### No Persistent Storage

- INFIO does not permanently store customer data on the EC2 instance. Data required during the assessment (such as metadata and query results) is held temporarily in memory or in local database storage for the duration of the session.
- Once the assessment process is complete, all temporary files and logs can be deleted by the user or automated cleanup can be configured.

### Encryption in Transit

- Data exchanged between INFIO and the SQL Server is encrypted using SSL/TLS protocols, preventing interception and man-in-the-middle (MITM) attacks.
- All communication between the EC2 instance and AWS services (such as S3, Secrets Manager, and KMS) uses AWS secure private connection.

### Encryption at Rest

- Any data written to disk during the assessment (such as temporary logs or configuration files) is encrypted using AWS Key Management Service (KMS).
- PostgreSQL data storage on the instance, if applicable, is encrypted using Transparent Data Encryption (TDE) for protection against unauthorized disk access.

---

## Role-Based Access Control (RBAC)

### Principle of Least Privilege

- The EC2 instance is provisioned with the minimum set of AWS Identity and Access Management (IAM) roles and policies required to perform database assessments.
- The instance may be given limited access to specific AWS services (e.g., S3 for temporary file storage, KMS for encryption, Secrets Manager for retrieving sensitive credentials).
- Customers are encouraged to review and customize the IAM policies assigned to the instance to further restrict or expand permissions based on their needs.

#### IAM Policy Attached to EC2 Role

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "cloudformation:DescribeStacks",
        "cloudformation:DescribeStackEvents",
        "cloudformation:DescribeStackResources",
        "cloudformation:GetTemplate",
        "cloudformation:ValidateTemplate"
      ],
      "Resource": "arn:aws:cloudformation:us-east-1:020877189064:stack/*",
      "Effect": "Allow",
      "Sid": "CloudFormationPermissionsForInfioEC2Instance"
    },
    {
      "Action": [
        "cloudformation:CreateStack"
      ],
      "Resource": "*",
      "Effect": "Allow",
      "Sid": "CloudFormationCreatePermissionsForInfioEC2Instance"
    },
    {
      "Action": [
        "kms:DescribeKey",
        "kms:EnableKeyRotation",
        "kms:TagResource",
        "kms:Decrypt",
        "kms:Encrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "arn:aws:kms:us-east-1:020877189064:key/*",
      "Effect": "Allow",
      "Sid": "KMSPermissionsForInfioEC2Instance"
    },
    {
      "Action": [
        "kms:CreateKey",
        "kms:CreateAlias"
      ],
      "Resource": "*",
      "Effect": "Allow",
      "Sid": "KMSCreatePermissionsForInfioEC2Instance"
    },
    {
      "Condition": {
        "StringEquals": {
          "aws:ResourceTag/Purpose": "infio"
        }
      },
      "Action": [
        "secretsmanager:UpdateSecret",
        "secretsmanager:TagResource",
        "secretsmanager:GetSecretValue",
        "secretsmanager:DeleteSecret"
      ],
      "Resource": [
        "arn:aws:secretsmanager:us-east-1:020877189064:secret:infio-db-credentials-*",
        "arn:aws:secretsmanager:us-east-1:020877189064:secret:infio*"
      ],
      "Effect": "Allow",
      "Sid": "SecretsManagerPermissionsForInfioEC2Instance"
    },
    {
      "Action": [
        "secretsmanager:ListSecrets",
        "secretsmanager:CreateSecret",
        "secretsmanager:GetRandomPassword"
      ],
      "Resource": "*",
      "Effect": "Allow",
      "Sid": "SecretsManagerCreatePermissionsForInfioEC2Instance"
    },
    {
      "Action": [
        "s3:GetEncryptionConfiguration",
        "s3:PutEncryptionConfiguration",
        "s3:PutBucketTagging",
        "s3:PutObjectTagging",
        "s3:PutBucketPolicy",
        "s3:PutBucketPublicAccessBlock",
        "s3:GetBucketLocation",
        "s3:ListBucket",
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::infio-private-bucket",
        "arn:aws:s3:::infio-private-bucket/*",
        "arn:aws:s3:::infio-cf-templates",
        "arn:aws:s3:::infio-cf-templates/*"
      ],
      "Effect": "Allow",
      "Sid": "S3BucketPermissionsForInfioEC2Instance"
    },
    {
      "Action": [
        "s3:CreateBucket"
      ],
      "Resource": "arn:aws:s3:::infio-private-bucket",
      "Effect": "Allow",
      "Sid": "S3BucketCreatePermissionsForInfioEC2Instance"
    },
    {
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:us-east-1:020877189064:log-group:/*",
      "Effect": "Allow",
      "Sid": "CloudWatchLogsPermissionsForInfioEC2Instance"
    }
  ]
}
```

## Customer IAM Roles

### Dedicated IAM Role
Customers should use a **dedicated IAM role** for deploying the INFIO infrastructure. This ensures:
- Limited access to sensitive resources.
- Avoiding over-privileged accounts.

### Multi-Factor Authentication (MFA)
Enforce **MFA** for:
- Accessing the AWS Management Console.
- Performing administrative actions on the EC2 instance.

---

## Logging and Monitoring

### Activity Logging
- **Detailed Logs**: Activities such as script executions, database connections, and file transfers are stored in INFIO log files.
- **CloudWatch Notifications**: 
  - Custom metrics and alerts detect suspicious activities (e.g., unauthorized access, abnormal traffic spikes).
- **CloudWatch Logs**: Additional logging for EC2 instance actions, such as:
  - Instance start/stop.
  - IAM role changes.
  - Network configurations.

### Vulnerability Management
- **Regular Scans**: Continuous vulnerability scanning ensures no known vulnerabilities in the AMI system or software components.
- **Patch Management**: 
  - The AMI is updated regularly with the latest security patches.
  - Customers are encouraged to automate patch management using AWS Systems Manager.

---

## Customer Responsibilities

### Securing VPC
- Correctly configure VPC components such as:
  - Network ACLs.
  - Route tables to prevent unauthorized traffic.
- **Best Practices**:
  - Limit internet gateway access.
  - Configure VPN or Direct Connect.
  - Use AWS Network Firewall for advanced security.

### IAM Best Practices
- Enforce strong password policies.
- Enable MFA.
- Apply least privilege principles.

### Data Backup
- Customers are responsible for backing up temporary files or results produced by the assessment tool.
- Use scheduled backups to Amazon S3 or third-party solutions.

---

## Secure Storage and Key Management

### AWS Key Management Service (KMS)
- **Encryption**: KMS is used for encrypting data at rest and sensitive information.
- **Custom Keys**: Customers can specify the ARN for their KMS Customer Managed Key (CMK).

### Secrets Management
- **Sensitive Credentials**:
  - Stored in AWS Secrets Manager with encryption enabled.
  - Ensures no plain text credentials are exposed.

---

## Frequently Asked Questions (FAQ)

### How does INFIO ensure secure communication with my SQL Server?
INFIO uses the **AWS network** to securely connect and communicate with the source SQL Server. By deploying the INFIO instance within a **private subnet**, direct access to the SQL Server is isolated from public networks.

### How is the EC2 instance secured from external access?
The EC2 instance is deployed in a **private subnet** without direct access to the public internet. Only authorized internal resources, such as your database server within the VPC, can communicate with the instance through configured security groups.

### Can I access the EC2 instance via RDP?
Yes, RDP access is allowed but limited to **specific IP addresses**, and only **key-based authentication** is supported. You must use a securely stored private key to access the instance. We recommend regularly rotating keys and disabling RDP access when not needed.

### What kind of data is stored on the EC2 instance?
No customer data is stored persistently. Data processed during the assessment, such as:
- SQL DDL statements.
- SQL dynamic queries.
- SQL Server configuration details.
- Query results or logs.

This data can be temporarily written to disk but is **deleted after the assessment completes**. You may configure custom cleanup options based on your requirements.

### Are logs from the EC2 instance available to me?
Yes, all activity and access logs can be streamed to your **AWS CloudWatch** account, where you can monitor them in real-time. These logs provide insight into tool performance, user activity, and any security-related events.

### How is access to the tool controlled?
Access is controlled via **AWS IAM roles** and security groups. The principle of least privilege is applied, ensuring that the instance has minimal permissions. You can modify the IAM role assigned to the instance to restrict or expand its access to AWS services.

### What happens if vulnerabilities are discovered in the tool?
We continuously monitor the AMI for new vulnerabilities and apply patches when necessary. If vulnerabilities are discovered, a new version of the AMI is released, and customers are notified to update their instances.

### How can I ensure that my VPC and EC2 instance are secure?
We recommend following AWS VPC security best practices, including:
- Use of **Network ACLs**, **Security Groups**, and **AWS Network Firewall**.
- Monitoring traffic with **VPC Flow Logs**.
- Using **AWS Shield** for advanced DDoS protection.

### How do you ensure that the AMI is secure before deployment?
The AMI undergoes extensive testing, including:
- Vulnerability scanning.
- Penetration testing.
- Security audits.

We follow strict guidelines for patch management and secure software development practices.

### Can I update the software on the EC2 instance?
Yes, you can update the software components (such as PostgreSQL or Python) installed on the EC2 instance. We recommend backing up your instance before making any updates. You can also automate software patching via **AWS Systems Manager**.

### What if I need additional security measures for my use case?
We understand that security requirements vary by organization. You can configure additional security controls such as:
- Network access rules.
- Custom encryption keys.
- Monitoring alerts based on your specific needs.

### Can I use my own encryption keys for data at rest?
Yes, the tool supports **customer-managed encryption keys (CMKs)** through **AWS Key Management Service (KMS)** for encrypting EBS volumes or any sensitive data handled by the tool.

### How are my credentials stored and secured?
The tool uses **AWS Secrets Manager** to store credentials securely. All sensitive information is encrypted, and only the tool’s processes have access to these credentials.

### How do I update to the latest version of the tool?
Updated AMI images will be made available with security patches and new features. You can launch a new EC2 instance using the latest AMI and migrate any configurations from your previous instance.

### Can the tool be accessed by third parties?
No, the tool is designed to operate securely within the customer’s VPC and is only accessible by authorized systems in the customer’s network. No external access is allowed.

### What's the required runtime for INFIO in the VPC to finalize the assessment?
INFIO depends on the input files to start and assess the source environment. If all source files are available, INFIO might need a couple of hours (depending on the file size) to complete the assessment.

### In case INFIO needs to run for an extended period, will there be any antivirus software on the AMI?
Customers are advised to follow their organization’s best security practices to implement the necessary antivirus measures on the EC2 instance deployed using INFIO AMI. Please note that the AMI itself does not come with a pre-installed antivirus.

### Is there a file that is generated as part of the online INFIO assessment process? What information is captured as part of the output file?
The data collected during the assessment includes:
- SQL Server schema definition language.
- T-SQL code embedded within the application.
- SQL Server metadata.
- Configuration details.

This information is stored locally on the EC2 instance and is utilized solely for reporting purposes.
