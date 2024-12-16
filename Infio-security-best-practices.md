# Table of Contents

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
  - [Is there a file generated as part of the online INFIO assessment process? What information is captured as part of the output file?](#is-there-a-file-generated-as-part-of-the-online-infio-assessment-process-what-information-is-captured-as-part-of-the-output-file)

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
