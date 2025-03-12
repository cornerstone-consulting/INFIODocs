# Infio Documentation

# Table of Contents

- [Solution Overview](#solution-overview)
- [Purpose of This Document](#purpose-of-this-document)
- [Target Audience](#target-audience)
- [Features and Benefits](#features-and-benefits)
  - [INFIO Assessment Covers](#infio-assessment-covers)
  - [Key Assessment Outcomes](#key-assessment-outcomes)
- [Use Cases](#use-cases)
- [Concepts and Definitions](#concepts-and-definitions)
- [Architecture Overview](#architecture-overview)
  - [Components](#components)
- [AWS Well-Architected Design Considerations](#aws-well-architected-design-considerations)
- [AWS Services in this Solution](#aws-services-in-this-solution)
- [Rotating Programmatic Credentials and Cryptographic Keys](#rotating-programmatic-credentials-and-cryptographic-keys)
- [Plan Your Deployment](#plan-your-deployment)
  - [Cost](#cost)
- [Security](#security)
  - [IAM Roles](#iam-roles)
  - [Amazon S3 Security](#amazon-s3-security)
  - [Instance Metadata Service (IMDS)](#instance-metadata-service-imds)
- [Supported AWS Regions](#supported-aws-regions)
- [AWS CloudFormation Quotas](#aws-cloudformation-quotas)
- [Prerequisites to Deploy the INFIO Tool](#prerequisites-to-deploy-the-infio-tool)  
  - [1. Skillsets](#1-skillsets)  
  - [2. Others](#2-others)  
  - [3. License Manager](#3-license-manager) 
- [Prerequisites Checklist](#prerequisites-checklist)
  - [1. Required IAM Permissions](#1-required-iam-permissions)
  - [2. SQL Server Database Permissions](#2-sql-server-database-permissions)
  - [3. Configuring Inbound Rules for SQL Server and Target RDS for AWS Schema conversion](#3-configuring-inbound-rules-for-sql-server-and-target-rds-for-aws-schema-conversion)
- [Deployment Steps](#deployment-steps)
  - [How to Deploy EC2 from INFIO AMI](#how-to-deploy-ec2-from-infio-ami)
  - [Log in to a Windows INFIO EC2 Instance in a Private Subnet Using a Bastion Host](#log-in-to-a-windows-infio-ec2-instance-in-a-private-subnet-using-a-bastion-host)
  - [VPC Endpoint Deployment for CloudFormation Service (Optional)](#vpc-endpoint-deployment-for-cloudformation-service-optional)
  - [VPC Endpoint Deployment for S3, Secret manager, KMS, EC2, DMS, RDS, and IAM and security group for all VPC endpoints](#vpc-endpoint-deployment-for-s3-secret-manager-kms-ec2-dms-rds-and-iam-and-security-group-for-all-vpc-endpoints)
  - [Steps for setting up AWS Resources for Infio Assessment Tool via CloudFormation](#Steps-for-setting-up-AWS-resources-for-infio-assessment-tool-via-cloudFormation)
- [INFIO Assessment Tool Usage Guide](#infio-assessment-tool-usage-guide)
  - [Infio Dashboard Overview](#infio-dashboard-overview)
  - [Configuration Page Setup](#configuration-page-setup)
  - [Application Discovery](#application-discovery)
  - [Steps to Perform Assessment in INFIO](#steps-to-perform-assessment-in-infio)
  - [Generating the Summary Report](#generating-the-summary-report)
    - [Migration Assessment Report Covers](#migration-assessment-report-covers)
    - [Report Sections Overview](#report-sections-overview)
    - [Key Insights from TCO Analysis](#key-insights-from-tco-analysis)
  - [SQL server to Aurora postgresql](#sql-server-to-aurora-postgresql)
    - [Configure your data providers for DMS Schema conversion](#configure-your-data-providers-for-dms-schema-conversion)
    - [Steps to create a migration project](#steps-to-create-a-migration-project)
    - [Steps for Schema Conversion](#steps-for-schema-conversion)
    - [Export Schema Conversion Report](#export-schema-conversion-report)
- [Monitoring](#monitoring)
- [Recovery and Backup](#recovery-and-backup)
- [Managing Licenses](#managing-licenses)
- [Technical Support & Emergency Maintenance](#technical-support--emergency-maintenance)
- [FAQs](#faqs)


---
### Solution Overview
**INFIO** is designed to streamline and automate the database migration assessment process, ensuring a smooth and secure transition for complex database systems. This solution is tailored to assess and gather all critical information necessary to build a comprehensive migration plan from **MS SQL** to **Babelfish for Aurora PostgreSQL**. This assessment includes the INFIO Database Migration Assessment Report, which provides a comprehensive evaluation of database objects, dependencies, and potential compatibility issues. The report helps users understand migration feasibility, estimate effort, and identify required modifications to ensure a successful transition.

An important part of INFIO is **Sql Server** to **Aurora Postgres** assessment. This assessment contains DMS Schema conversion which is the report that it generates to help you convert your schema. This database migration assessment report summarizes all of the schema conversion tasks. It also details the action items for schema that can't be converted to the DB engine of your target DB instance. You can view the report in the S3 bucket or in INFIO ec2 instance as a PDF or comma-separated value (CSV) file.

The schema conversion report includes the following:
- An executive summary
- Recommendations, including conversion of server objects, backup suggestions, and linked server changes

With schema conversion report, you get a summary of your schema conversion tasks and the details for items that DMS Schema Conversion can't automatically convert to your target database. Database migration assessment reports help evaluate how much of your migration project DMS Schema Conversion can automate. These reports also help to estimate the amount of manual effort that is required to complete the conversion

The solution provides deep insights into the existing database environment while adhering to **AWS best practices** for cloud migration. It emphasizes **security**, **scalability**, **reliability**, and **cost optimization** throughout the process.

---

### Purpose of This Document
This user guide provides comprehensive instructions for deploying and using the **INFIO Tool**. It covers:

- Necessary prerequisites
- Step-by-step deployment procedures
- Usage guidelines
- Troubleshooting tips
- Frequently asked questions

---

### Target Audience
This guide is intended for:

- **IT professionals**
- **Database administrators**
- **Cloud engineers**

who are responsible for assessing database compatibility for migration purposes.

---

### Features and Benefits
#### INFIO Assessment Covers:
- **Application Deep-Dive**: A thorough analysis of the application architecture and its interactions with the database.
- **Compass for Database Structure**: An in-depth review of the current database structure to ensure an accurate transition to Babelfish for Aurora PostgreSQL.
- **Compass for Application Code**: A detailed assessment of application code for compatibility with Babelfish for Aurora PostgreSQL.
- **Database Migration Service (DMS) Assessment**: Evaluation of AWS DMS to support an efficient migration with minimal downtime.
- **Cross-Database Dependency Identification**: Detection and resolution of dependencies across multiple databases.
- **Schema conversion** - Transforms the existing SQL Server schema into a Aurora PostgreSQL schema, ensuring a smooth migration.

### Key Assessment Outcomes:
- **Target State Architecture**: A secure, optimized architecture for the target PostgreSQL environment.
- **Total Cost of Ownership (TCO) Analysis**: Cost analysis to understand the cost implications of the migration.
- **Application and Database Code Compatibility Report**: A detailed report highlighting unsupported T-SQL features and multiple viable solutions to ensure code is Babelfish compliant.
- **Data Migration Feasibility**: Assessment of the data migration process, ensuring minimal disruptions and successful transfer.
- **Dependency Relationships**: A clear understanding of relationships across all database objects to ensure a smooth migration process.
- **Schema conversion assessment report**: A database migration assessment report summarizes all of the schema conversion tasks which includes an executive summary, recommendations, including conversion of server objects, backup suggestions, and linked server changes.

---

### Use Cases
INFIO can be used to:

- **Assess large-scale heterogeneous database migrations to AWS**: Enable a single-pane-of-glass view for large-scale database workload migration assessments to AWS.
- **Provide prebuilt automation and reporting**: Utilize role-based access via a single web interface designed specifically for database migration assessments.
- **Generates assessment report for schema converion**: Provides detailed reports on schema compatibility and necessary modifications for a seamless migration.

---

### Concepts and Definitions
This section describes key concepts and defines terminology specific to this solution:

- **INFIO AMI**: Prebuilt Amazon Machine Image hosting INFIO tool and services.
- **Stack**: CloudFormation stack used for AWS resource deployment.
- **Application**: A group of resources that make up a single business service or application.
  - **Server**: Source database server to be assessed.
  - **Database**: Source database to be assessed.

---

### Architecture Overview
![Screenshot_12-12-2024_131219_](https://github.com/user-attachments/assets/bc6ea1ba-24f6-4749-a692-3c6db88e469b)

#### Components:
- **AMI**: The INFIO AMI, a pre-configured, Windows-based Amazon Machine Image (AMI) available on AWS Marketplace, can be used to rapidly deploy EC2 instances equipped with the necessary tools and scripts for implementing the INFIO assessment solution.
- **Amazon S3**: Used for storing input files prior to assessments and output files following the assessment process.
- **Secret Manager**: Used as a repository for storing database credentials required to connect to databases.
- **Key Management Services**: Serves as a secure storage location for encryption keys used to protect resources during INFIO deployments.
- **CloudFormation Stack**: Used for automated deployments of AWS resources.
- **VPC Endpoints (Optional)**: Provide secure, private connectivity between your VPC and AWS services without exposing your traffic to the public internet.
- **DMS**: DMS Schema Conversion automatically converts your source SQL Server database schemas and most of the database code objects to a format compatible with the target Aurora postgres database. This conversion includes tables, views, stored procedures, functions, data types, synonyms, and so on. DMS Schema Conversion to assess the complexity of your migration for your source data provider, and to convert database schemas and code objects.

---

### AWS Well-Architected Design Considerations:

| Pillar                     | Design Consideration                                                                                                  |
|----------------------------|----------------------------------------------------------------------------------------------------------------------|
| **Operational Excellence** | Resources defined as Infrastructure as Code using CloudFormation.                                                   |
| **Security**               | IAM used for authentication and authorization, with narrowly scoped role permissions. VPC endpoints are used for private communication. |
| **Reliability**            | This is an AMI-based solution, and a new preconfigured EC2 instance can be provisioned quickly in case of any failure. |
| **Performance Efficiency** | Right-sizing EC2 instance types and sizes to align with the assessment workload.                                     |
| **Cost Optimization**      | EC2 lifecycle management and right-sizing help reduce costs.                                                         |
| **Sustainability**         | The INFIO Plugin streamlines input file gathering, reducing EC2 runtime. Effective EC2 instance lifecycle management further minimizes resource consumption. |

---

#### AWS Services in this Solution
- **Amazon VPC (Core)**: Provides the underlying network infrastructure to securely connect EC2 instances and other AWS services, including VPC endpoints for private access to resources like S3, KMS, and Secrets Manager.
- **Amazon EC2 (Core)**: Provides scalable compute resources for running INFIO tool or services within the VPC. Instances in private subnets securely access AWS services via VPC endpoints.
- **Amazon S3 (Core)**: Acts as a storage solution for database assessment-related artifacts, such as DDL and SQL statement files, accessed via VPC endpoints.
- **AWS Secrets Manager (Supporting)**: Securely manages and retrieves sensitive information, such as database credentials.
- **AWS Key Management Service (Supporting)**: Manages encryption keys used to protect data and resources.
- **AWS CloudFormation (Supporting)**: Accelerates AWS resource provisioning with infrastructure as code.
- **VPC Endpoints (Optional & Supporting)**: Provide secure, private connectivity between your VPC and AWS services.
- **AWS DMS(Supporting)**: Provides assessment report for schema conversion. 

---

### Rotating Programmatic Credentials and Cryptographic Keys
- Secret Manager keys and Customer Manager keys are typically rotated on a periodic basis to enhance security.
- The **INFIO Tool** is designed for short-term use, typically lasting a few hours to a few days. The decision to rotate keys rests solely with the customer and is not enforced by the tool, as frequent rotation is uncommon for the tool’s intended use case.

---

### Plan Your Deployment  

#### Cost  
You are responsible for the cost of the AWS services used while running this solution. As of this revision, the estimated cost for running this solution with default settings in the **US East (N. Virginia)** region is detailed below.  

To get the most recent and accurate cost estimate for your AWS architecture, you can use the [AWS Pricing Calculator](https://calculator.aws/).  

| AWS Service           | Factors                                  | Cost/Month [USD]       |
|------------------------|------------------------------------------|------------------------|
| **Amazon S3**         | Storage (10GB) & 500 get and put requests/month | $0.23                 |
| **AWS Secrets Manager** | 5 secrets x 30 days duration            | $0.23                 |
| **AWS EC2**           | m6a.large, 10GB EBS                     | $2.00                 |
| **AWS Key Management Service (KMS)** | 2 CMK x Number of symmetric requests (1000) | $130.80 |
| **Security (IAM Roles)** | IAM roles for EC2 access to AWS services | $2.00                 |

---

### Security  

#### IAM Roles  
AWS Identity and Access Management (IAM) roles allow you to assign granular access policies and permissions to services and users in the AWS Cloud. This solution creates IAM roles that grant the EC2 instances access to the other AWS services used in this solution.  

#### Amazon S3 Security  
To enhance security while accessing private objects stored in Amazon S3, we are utilizing **VPC S3 endpoints**. This approach allows our application to securely communicate with S3 without requiring public access to the buckets. By avoiding the creation of public buckets, we significantly reduce the potential attack surface and mitigate the risk of unauthorized access to sensitive data.  

#### Instance Metadata Service (IMDS)  
The EC2 instances created in this solution support the ability to disable **Instance Metadata Service Version 1 (IMDSv1)** if required.  

- **IMDSv2** is available and recommended for use, as indicated by AWS best practices.  
- This solution uses the **AWS CLI** and **Boto3** for API calls, ensuring compatibility with IMDSv2.  
- Using the latest versions of AWS SDKs, we align with AWS security standards, allowing customers to disable IMDSv1 as an option.  

---

#### Supported AWS Regions  
INFIO on AWS is available in the following AWS Regions:  

| Region Name          |
|-----------------------|
| **US East (N. Virginia)** |

---

### AWS CloudFormation Quotas  
Your AWS account has CloudFormation quotas that you should be aware of when launching the stack for this solution. By understanding these quotas, you can avoid limitation errors that would prevent you from deploying this solution successfully.  

---
### Prerequisites to Deploy the INFIO Tool

#### 1. Skillsets

To deploy the solution successfully, the following skills and knowledge are required:

- Familiarity with AWS.
- Proficiency in AWS CLI and various AWS services (such as S3, EC2, VPC, CloudFormation, KMS, DMS and Secrets Manager).
- Familiarity with database migration processes and schema conversions.
- Basic understanding of PostgreSQL and SQL Server.

> **Important Note**: The INFIO does not require the use of AWS account root privileges for deployment or operation.

#### 2. Others

The INFIO EC2 instance includes all necessary tools to ensure secure and efficient SQL Server to PostgreSQL migration:

- **AWS CLI v2**: Facilitates deploying AWS services using CloudFormation.
- **PostgreSQL**: Preinstalled to store assessment data and configuration details, serving as a local database for quick access to critical migration information.
- **Babelfish Compass**: Preinstalled to identify SQL Server features incompatible with PostgreSQL and provide guidance for adjustments.

#### 3. License manager

AWS License Manager must be configured to manage and track licenses for EC2 instances. Ensure proper license configurations are set up and associated with instances.

---

### Prerequisites Checklist

Before deploying the database migration assessment tool, ensure the following prerequisites are met:

1. Required IAM Permissions
2. SQL Server Database Permissions
3. Configuring Inbound Rules for SQL Server and Target RDS for AWS Schema conversion  

#### 1. Required IAM Permissions  

Below are the required permissions for an IAM user to create AWS resources within an AWS account:  

#### EC2 Permissions  

| Permission                         | Description                                           |
|------------------------------------|-------------------------------------------------------|
| `ec2:CreateVpcEndpoint`            | Allows creation of a VPC endpoint.                   |
| `ec2:DescribeVpcEndpoints`         | Grants permission to view details of existing VPC endpoints. |
| `ec2:DeleteVpcEndpoints`           | Permits deletion of VPC endpoints.                   |
| `ec2:ModifyVpcEndpoint`            | Allows modification of VPC endpoint attributes.      |
| `ec2:DescribeRouteTables`          | Enables viewing associated route tables.             |
| `ec2:DescribeVpcs`                 | Grants permission to view VPC configurations.        |
| `ec2:DescribeSubnets`              | Allows viewing subnet details within a VPC.          |
| `ec2:DescribeSecurityGroups`       | Permits viewing of security groups.                  |
| `ec2:CreateSecurityGroup`          | Grants permission to create new security groups.     |
| `ec2:DeleteSecurityGroup`          | Allows deletion of security groups.                  |
| `ec2:RevokeSecurityGroupEgress`    | Permits removal of outbound rules.                   |
| `ec2:AuthorizeSecurityGroupIngress`| Allows adding inbound rules.                         |
| `ec2:AuthorizeSecurityGroupEgress` | Grants permission to add outbound rules.             |

#### CloudFormation Permissions  

| Permission                         | Description                                           |
|------------------------------------|-------------------------------------------------------|
| `cloudformation:DescribeStacks`    | View details of stacks.                              |
| `cloudformation:DescribeStackEvents` | See events related to stack creation, update, or deletion. |
| `cloudformation:DescribeStackResources` | Retrieve information about stack resources.        |
| `cloudformation:GetTemplate`       | Retrieve the CloudFormation template.                |
| `cloudformation:ValidateTemplate`  | Validate a CloudFormation template.                  |
| `cloudformation:CreateStack`       | Create a new stack.                                  |

#### KMS Permissions  

| Permission                         | Description                                           |
|------------------------------------|-------------------------------------------------------|
| `kms:DescribeKey`                  | View metadata of a specific KMS key.                 |
| `kms:EnableKeyRotation`            | Enable automatic key rotation.                       |
| `kms:TagResource`                  | Tag a KMS key.                                       |
| `kms:Decrypt`                      | Decrypt data using the KMS key.                      |
| `kms:Encrypt`                      | Encrypt data with the KMS key.                       |
| `kms:GenerateDataKey`              | Generate a data encryption key (DEK).                |
| `kms:CreateKey`                    | Create new KMS keys.                                 |
| `kms:CreateAlias`                  | Create an alias for a KMS key.                       |

#### Secrets Manager Permissions  

| Permission                         | Description                                           |
|------------------------------------|-------------------------------------------------------|
| `secretsmanager:ListSecrets`       | List all secrets.                                    |
| `secretsmanager:UpdateSecret`      | Update secret value or metadata.                     |
| `secretsmanager:TagResource`       | Add or modify secret tags.                           |
| `secretsmanager:GetSecretValue`    | Retrieve a secret's value.                           |
| `secretsmanager:DeleteSecret`      | Delete a specified secret.                           |
| `secretsmanager:CreateSecret`      | Create new secrets.                                  |
| `secretsmanager:GetRandomPassword` | Generate random passwords.                           |

#### S3 Permissions  

| Permission                         | Description                                           |
|------------------------------------|-------------------------------------------------------|
| `s3:GetEncryptionConfiguration`    | View bucket encryption configuration.                |
| `s3:PutEncryptionConfiguration`    | Modify bucket encryption settings.                   |
| `s3:PutBucketTagging`              | Add or update tags on an S3 bucket.                  |
| `s3:PutObjectTagging`              | Add or update tags on individual objects.            |
| `s3:PutBucketPolicy`               | Apply or update bucket policies to control access.   |
| `s3:PutBucketPublicAccessBlock`    | Configure public access block settings for a bucket. |
| `s3:GetBucketLocation`             | Retrieve the geographical region of a bucket.        |
| `s3:ListBucket`                    | List objects and folders within a bucket.            |
| `s3:GetObject`                     | Retrieve an object from a bucket.                    |
| `s3:PutObject`                     | Upload or overwrite an object in a bucket.           |
| `s3:GetBucketVersioning`           | Check if versioning is enabled for a bucket.        |
| `s3:GetObjectVersion`              | Retrieve a specific version of an object.            |
| `s3:PutBucketVersioning`           | Enable or modify versioning settings for a bucket.   |
| `s3:CreateBucket`                  | Create a new S3 bucket.                              |

#### CloudWatch Logs Permissions 

| Permission                 | Description                                              |
|----------------------------|----------------------------------------------------------|
| `logs:CreateLogGroup`      | Create a new log group in CloudWatch Logs.               |
| `logs:CreateLogStream`     | Create a log stream within a log group.                  |
| `logs:PutLogEvents`        | Upload log events to a specific log stream.              |
| `logs:DescribeLogStreams`  | Retrieve details about log streams within a log group.   |
| `logs:DescribeLogGroups`   | Retrieve details about existing log groups.              |

#### AWS License Manager Persmissions

| Permission                                      | Description                                              |
|------------------------------------------------|----------------------------------------------------------|
| `license-manager:CheckoutLicense`              | Check out a license from AWS License Manager.           |
| `license-manager:ExtendLicenseConsumption`     | Extend the consumption period of a checked-out license. |
| `license-manager:ListReceivedLicenses`         | List licenses received from a license issuer.           |
| `license-manager:GetLicense`                   | Retrieve details about a specific license.              |


#### AWS DMS Permissions 

| Permission                                  | Description                                                       |
|--------------------------------------------|-------------------------------------------------------------------|
| `dms:CreateDataProvider`                   | Create a data provider for database migration.                   |
| `dms:CreateMigrationProject`               | Create a migration project in AWS DMS.                           |
| `dms:CreateDataMigration`                   | Initiate a new data migration process.                           |
| `dms:CreateReplicationInstance`            | Create a replication instance for data migration.                |
| `dms:ListDataProviders`                     | List all configured data providers.                              |
| `dms:ListMigrationProjects`                 | List existing migration projects in AWS DMS.                     |
| `dms:AddTagsToResource`                     | Add tags to a DMS resource for better management.                |
| `dms:StartDataMigration`                    | Start a data migration process.                                  |
| `dms:DescribeInstanceProfiles`              | Get details of DMS instance profiles.                           |
| `dms:ListInstanceProfiles`                   | List all available instance profiles in AWS DMS.                 |
| `dms:StartMetadataModelAssessment`          | Start an assessment of a metadata model.                        |
| `dms:ListMetadataModelAssessments`          | List previous metadata model assessments.                        |
| `dms:ExportMetadataModelAssessment`         | Export metadata model assessment reports.                        |
| `dms:UpdateDataProvider`            | Update an existing data provider for database migration. |
| `dms:DeleteDataProvider`            | Delete a configured data provider.                   |
| `dms:UpdateMigrationProject`        | Update an existing migration project in AWS DMS.     |
| `dms:DeleteMigrationProject`        | Remove a migration project from AWS DMS.             |


#### AWS RDS Permissions 

| Permission                         | Description                                              |
|------------------------------------|----------------------------------------------------------|
| `rds:DescribeDBClusters`          | Retrieve details about Amazon RDS database clusters.    |
| `rds:DescribeDBInstances`         | Get information about RDS database instances.           |
| `rds:DescribeDBClusterSnapshots`  | View details of DB cluster snapshots.                   |
| `rds:ListTagsForResource`         | List tags assigned to an RDS resource.                  |

#### AWS IAM Permissions  

| Permission        | Description                                      |
|------------------|--------------------------------------------------|
| `iam:PassRole`   | Allow passing an IAM role to AWS services.       |
| `iam:ListRoles`  | List all IAM roles in the AWS account.           |




#### 2. SQL Server Database Permissions

To set up the minimum permissions for an on-premises SQL Server database:

1. Create a new SQL Server account with password authentication using SQL Server Management Studio (SSMS).
2. In the **User Mappings** section, select the `MSDB` and `MASTER` databases and assign the `DB_OWNER` role to the target database.
3. Explicitly grant the **Connect SQL** privilege.
4. Run the following SQL commands:

    ```sql
    GRANT VIEW SERVER STATE TO user;
    USE MSDB;
    GRANT SELECT ON MSDB.DBO.BACKUPSET TO user;
    GRANT SELECT ON MSDB.DBO.BACKUPMEDIAFAMILY TO user;
    GRANT SELECT ON MSDB.DBO.BACKUPFILE TO user;
    ```

#### 3. Configuring Inbound Rules for SQL Server and Target RDS for AWS Schema conversion

1. On the SQL Server, allow inbound traffic on the SQL port (1433 or as configured) from the security group attached to the infio-assessment-instance (infio-ec2-dms-sg).
2. On the target RDS, allow inbound traffic on port 5432 from the security group attached to the infio-assessment-instance (infio-ec2-dms-sg).

---

## Deployment Steps

### How to Deploy EC2 from INFIO AMI

1.  Navigate to the [AWS Marketplace](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Faws.amazon.com%2Fmarketplace%3F%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fawsmp-contessa&forceMobileApp=0) to download the INFIO AMI. During this process, you will be prompted to answer several necessary questions, detailed below:
    - **EC2InstanceType**: Specifies the EC2 instance type. Default is `m6a.large`.  
    - **SubnetID**: The ID of the subnet where the Infio EC2 instance will be deployed.  
    - **DMSSubnetGroupSubnetIDs**: A comma-separated list of subnet IDs for the source SQL Server and target PostgreSQL.  
    - **SourceSQLServerPort**: The port number used by the source SQL Server. Default is `1433`.  
    - **SourceSQLServerIP**: The IP address of the source SQL Server in `x.x.x.x/32` format.  
    - **TargetPGServerSG**: The Security Group ID attached to the target PostgreSQL database.  
    - **InfioEC2EncryptionKeyAlias**: The KMS encryption key alias used to encrypt the Infio EC2 instance. Default is `aws/ebs`.  
    - **VPCCIDR**: The CIDR of the VPC where the Infio EC2 instance will be deployed.  
    - **VPCID**: The VPC ID where the Infio EC2 instance will be deployed.  
    - **InboundCIDRIP**: The CIDR or IP address allowed for RDP access to the Infio EC2 instance. Must be in `xx.xx.xx.xx/xx` format.  

2. After completing the questionnaire, AWS Marketplace will deploy an EC2 instance in the private subnet.

> **Note on Availability of INFIO** : INFIO does not necessitate high availability at this stage. The nature of our workload and the expected patterns indicate that a single AZ deployment will adequately support our operational needs without compromising performance.

> **Note on Deployment Time**: The deployment time for INFIO tool will be around 15 minutes at most, enabling us to move forward with our tool timelines efficiently.

> **Important Note**: When the INFIO EC2 instance is created, a PEM key will be automatically generated. Download this key to retrieve the Administrator password for the EC2 instance. You will need the PEM key to decrypt the password in order to RDP or connect to INFIO EC2 instance.

---

### Log in to a Windows INFIO EC2 Instance in a Private Subnet Using a Bastion Host

1. **Ensure the Bastion Host Setup**  
   - Place the bastion host in a **public subnet** with a public IP address.  

2. **Configure Security Groups**:
   - **Note**: The bastion host can run on any operating system (Linux or Windows). Ensure the security group rules are configured according to the OS being used for the bastion host, and follow the next steps for that.   
   - **Bastion Host Security Group**:  
     - Allow inbound **SSH (port 22)** from your IP for Linux-based bastion hosts.  
     - Allow inbound **RDP (port 3389)** from your IP for Windows-based bastion hosts.  
   - **INFIO EC2 Instance Security Group**:  
     - Allow inbound RDP (port 3389) only from the bastion host's private IP address.  

4. **Connect to the Bastion Host**  
   - **For Linux-based bastion hosts**:  
     - Use SSH tunneling with PuTTY to access the INFIO EC2 instance. Comprehensive documentation on this configuration is available.  
   - **For Windows-based bastion hosts**:  
     - Use **Remote Desktop Connection (RDP)** from your local machine to connect to the bastion host using its public IP address.  
     - Enter your bastion host Windows credentials.

5. **Connect to the Private EC2 Instance**  
   - From the bastion host, open another Remote Desktop Connection.  
   - Use the **private IP address** of your Windows EC2 instance in the private subnet.  
   - This establishes a connection path: `Local Machine → Bastion Host Instance → Private INFIO Windows EC2 Instance`.  

---
### VPC Endpoint Deployment for CloudFormation Service (Optional):

**Purpose**:
From the AWS Console in the same account where you have deployed the **INFIO EC2 instance**, you can deploy an AWS VPC Endpoint for CloudFormation service. This step is necessary if your **VPC landing zone** does not support EC2 instances hosted in a private subnet to connect to AWS services (e.g., when EC2 instance in private subnets cannot access CloudFormation service). Access to the CloudFormation Service is required to deploy additional AWS resources from the **INFIO EC2 instance**. 

> **Note**: Verify network connectivity or consult your cloud network administrator before deploying this endpoint.

This cloudformation template will deploy cloudformation endpoint for INFIO EC2 instance and security group for cloudformation endpoint.

**Deployment Steps:**

1. **Create the Stack in AWS Console**  
   - Open **CloudFormation** in the AWS Console.  
   - Navigate to the **Stacks** section and click on the **Create stack** button.  
   - Choose **With new resources (standard)** from the dropdown.  

2. **Upload the Template**  
   - Under the **Prepare template** option, select **Choose an existing template**.  
   - Click on the **Upload a template file** button.
   - For the upload file, go to the `Desktop` of the Windows EC2 instance (**INFIO EC2 instance**).  
   - Locate the folder containing the **INFIO Assessment Tool**. Inside, you'll find the `aws-infra-setup` folder, which contains the `INFIO-CF-VPCEndpoint_EndpointSG-CF.json` CloudFormation template.  
   - Click **Next** to proceed.

4. **Configure the Stack**  
   - Enter `INFIOCFVPCEndpoints` as the stack name in the **Stack name** field.  
   - Provide the parameter values as described below and click the **Next** button:  
     - **SecurityGroupID**: Specify the ID of the security group attached to your INFIO EC2 instance.  
     - **SubnetID**: Specify the subnet where the INFIO EC2 instance is deployed.  
     - **VPCCIDR**: Provide the CIDR block of the VPC where the INFIO EC2 instance is deployed.  
     - **VPCID**: Provide the ID of the VPC where the INFIO EC2 instance is deployed.  

5. **Complete Deployment**  
   - In the **Configure stack options** section, click **Next**, then click the **Submit** button to deploy the stack.

---

### VPC Endpoint Deployment for S3, Secret manager, KMS, EC2, DMS, RDS, and IAM and security group for all VPC endpoints

**Purpose:** 
- This deployment is necessary if your **VPC landing zone** does not support **EC2 instances hosted in a private subnet** to connect with AWS services. The provided **CloudFormation template** creates a secure environment for the INFIO assessment tool within an AWS VPC. It establishes **VPC endpoints** for **S3**, **Secrets Manager**, **KMS**, **EC2**, **DMS**, **RDS**, and **IAM** services, enabling secure access from the INFIO EC2 instance, in which EC2, DMS, RDS and IAM service endpoints are being used for AWS schema conversion.
- This CloudFormation template creates a Security Group (SG) named **infio-vpc-endpoint-sg** for VPC endpoints (S3, Secrets Manager, KMS, EC2, DMS, RDS, and IAM). This security group controls who can access the VPC endpoint and where it can send data.  

> Note: Configuring VPC Endpoints for these services is optional. If your INFIO EC2 instance already has direct access to these services through existing network configurations, you may skip this step.

**Prerequisites:**
- Verify network connectivity or consult your **cloud network administrator** before proceeding.
- Confirm access to the CloudFormation template stored on the INFIO EC2 instance.

Follow the steps for deployment.

**Deploy the Stack Using AWS CLI:**
- Use the following command to deploy the stack from the **INFIO EC2 instance command prompt**:  
*(Alternatively, this can be done via the AWS Console.)*

1. Access the **Desktop** drive of the **INFIO EC2 instance**.
2. Navigate to the folder containing the **INFIO Assessment Tool**.  
   - Inside, locate the `aws-infra-setup` folder.
   - Find the template file: `INFIO-S3-KMS-SM-DMS-EC2-RDS-IAM-VPCEndpoints.json`.
   - Open the command prompt from `aws-infra-setup` folder and run the below command.

```bash
aws cloudformation create-stack --stack-name INFIOVPCEndpoints \
    --template-body file://C:/Users/Administrator/Desktop/INFIO%20Assessment%20Tool/aws-infra-setup/INFIO-S3-KMS-SM-DMS-EC2-RDS-IAM-VPCEndpoints.json \
    --parameters ParameterKey=VPCID,ParameterValue=<ParameterValue1> \
                ParameterKey=VPCCIDR,ParameterValue=<ParameterValue2> \
                ParameterKey=SecurityGroupID,ParameterValue=<ParameterValue3> \
                ParameterKey=SubnetID,ParameterValue=<ParameterValue4> \
    --endpoint-url <cloudformation_endpoint_url>
```
> Note: Follow the next steps for cloudformation_endpoint_url and parametervalues.

3. Change the following parameter values according to your specific details:
      - **ParameterValue1**: Your **VPC ID**.
      - **ParameterValue2**: Your **VPC CIDR block**.
      - **ParameterValue3**: Your **Security Group ID**.
      - **ParameterValue4**: Your **Subnet ID**.

4. Ensure the following parameters are available for the deployment:
    - **VPCCIDR**:  
      The CIDR block of the VPC where the INFIO EC2 instance is deployed.  
      - This restricts outbound traffic from the security group, ensuring only traffic within the specified VPC can access the endpoints.
    - **VPCID**:  
      The ID of the VPC where the INFIO EC2 instance is deployed.  
      - This ensures the VPC endpoint is created within the same VPC for network accessibility.
    - **SecurityGroupID**:  
      The ID of the security group attached to your INFIO EC2 instance.  
      - This defines inbound rules for secure communication with the VPC endpoint.
    - **SubnetID**:  
      The subnet where the INFIO EC2 instance is deployed.
      - Use private subnet id where INFIO Ec2 instance will be deployed. 


5. Find the CloudFormation Endpoint URL:
    - Open the **AWS Console**.
    - Navigate to the **VPC Endpoint** section.
    - Locate the **AWS CloudFormation VPC endpoint** created earlier.
    - Copy the **CloudFormation endpoint URL** and use it in your command, formatted like this:

       Example:  
       `https://vpce-0d160416ea24180580oelz5ve.cloudformation.us-east-1.vpce.amazonaws.com`.
      
6. Final Verification
    - Ensure that this step is completed before proceeding to deploy dependent services.
    - Verify the status of the stack in the **AWS Console** under **CloudFormation services** to confirm that the stack is fully deployed and all resources have been created.

#### VPC Endpoint Security Group inbound and outbound rules(infio-vpc-endpoint-sg)

This **Security Group** allows secure **HTTPS (443) communication** between the **VPC endpoint**, the **INFIO EC2 instance**, and other AWS services inside the **VPC**.

#### 🔹 Inbound Rules (Incoming Traffic)
- **Allows HTTPS traffic from the INFIO EC2 instance** (SecurityGroupID) → Ensures only this EC2 instance can access the VPC endpoint securely.  
- **Allows HTTPS traffic from within the VPC (VPCCIDR)** → Enables access for other AWS services like EC2, Lambda, etc.

#### 🔹 Outbound Rules (Outgoing Traffic)
- **Allows HTTPS responses to all resources inside the VPC (VPCCIDR)** → Ensures AWS services in the VPC can receive responses.  
- **Allows HTTPS responses to the INFIO EC2 instance** (SecurityGroupID) → Ensures the EC2 instance can get responses from the VPC endpoint.

This setup ensures **secure communication** between the **VPC endpoint, INFIO EC2 instance, and other AWS resources inside the VPC**.

---

### Steps for setting up AWS Resources for Infio Assessment Tool via CloudFormation

- This CloudFormation template provisions the necessary AWS resources for the **INFIO Assessment Tool**, including a KMS key, secrets manager secret, and S3 bucket. These resources are optimized to ensure secure data storage and management, enabling robust encryption and controlled access with AWS's advanced security features. The Secrets Manager secrets are specifically intended to securely store credentials and connection details for the data providers used by AWS Database Migration Service (DMS) and Infio tool, ensuring secure and seamless data migration operations.
- When an **INFIO EC2 instance** is deployed from the **AWS Marketplace**, the following additional resources are automatically created:

**1️⃣ EC2 IAM Role**  
- An **IAM role** is attached to the instance to grant **secure access** to AWS services.  

**2️⃣ Security Group (SG) "infio-ec2-dms-sg" for the INFIO EC2 Instance**  
- A **Security Group** is created to **control inbound and outbound traffic**.  
- It ensures **secure communication** between the EC2 instance and other services inside the **VPC**.
- This security group is attached with INFIO EC2 instance and DMS instance profile.

**3️⃣ DMS Instance Profile (Required by Schema Conversion)**  
- A **DMS (Database Migration Service) instance profile** is created.  
- This is required by **Schema conversion (SC)** for **securely managing migration-related tasks**.  

#### Deployment Steps

1. Go to the `Desktop` of the Windows EC2 instance (**INFIO EC2 instance**). Locate the folder containing the `INFIO Assessment Tool`. Inside, you'll find the `aws-infra-setup` folder, and within it, the `INFIO-Setup-CF.json` CloudFormation template.

2. Use the below AWS CLI command to deploy the CloudFormation stack from the INFIO EC2 CMD terminal. *(Alternatively, you can also create the stack from the AWS Console.)*
> **Note**: AWS CLI is preinstalled in the INFIO EC2 instance.

  **Command**:

   ```bash
   aws cloudformation create-stack --stack-name INFIOResources \
       --template-body file://C:/Users/Administrator/Desktop/INFIO%20Assessment%20Tool/aws-infra-setup/INFIO-Setup-CF.json \
       --endpoint-url <cloudformation_endpoint_url>
   ```
The `--endpoint-url` parameter is **optional**. Utilize it only if you created a **CloudFormation endpoint** in the previous step. If not, please omit this parameter from the command line.

#### To Locate the CloudFormation Endpoint URL

1. Open the **AWS Console**.
2. Navigate to the **VPC Endpoint** section where the **AWS CloudFormation VPC endpoint** was created.
3. Copy the **URL** and paste it into your command.

    **Example Format**:

    ```plaintext
    https://vpce-0d160416ea2418058-0oelz5ve.cloudformation.us-east-1.vpce.amazonaws.com
    ```

#### Security Group inbound and outbound rules for INFIO EC2 instace(infio-ec2-dms-sg)  

This **Security Group** manages access for **database and network communication** between **EC2, SQL Server, PostgreSQL, and VPC endpoints**.

> Note: SQL server port (can be 1433 or anything that your input) is taken as input from the user. 

🔹 **Inbound Rules (Incoming Traffic)**  
- **`SQL server Port (1443 or custom)` from `self`** → Allows internal communication within the same security group.  
- **`Port 5432` from `self`** → Enables PostgreSQL instances in the same security group to communicate.  
- **`Port 443` from `VPC CIDR`** → Allows secure HTTPS traffic from resources inside the VPC.  
- **`Port 80` from `VPC CIDR`** → Permits HTTP connections from resources inside the VPC.  
- **`Port 3389` from `Inbound CIDR / IP`** → Permits RDP connections from user provided CIDR or IP address.  
- **`SQL server Port (1443 or custom)` from `VPC CIDR`** → Permits database connections from resources inside the VPC.  
- **`SQL server Port (1443 or custom)` from `SQL Server IP`** → Allows incoming connections from a specific SQL Server instance.  
 
🔹 **Outbound Rules (Outgoing Traffic)**  
- **`SQL server Port (1443 or custom)` to `SQL Server IP`** → Allows outgoing database connections to SQL Server.  
- **`Port 443` to `VPC CIDR`** → Allows HTTPS traffic to resources inside the VPC.  
- **`Port 80` from `VPC CIDR`** → Permits HTTP traffic to resources inside the VPC.  
- **`Port 5432` to `PostgreSQL SG`** → Permits outgoing connections to PostgreSQL instances.  
- **`SQL server Port (1443 or custom)` to `self`** → Allows internal SQL Server communication within the same security group.  
- **`Port 5432` to `self`** → Enables internal PostgreSQL communication within the same security group.
 
User needs to manually add: 
🔹 **Outbound Rules (Outgoing Traffic)**   
- **`Port 443` to `VPC Endpoint SG`** → Enables secure communication with VPC endpoints.  

---

### INFIO Assessment Tool Usage Guide

1. Before running the INFIO Assessment Tool, you must complete the CloudFormation template steps outlined above to create the necessary INFIO resources and ensure the tool runs successfully.
2. Navigate to the `Desktop:\INFIO Assessment Tool` folder located on the desktop of the INFIO EC2 instance.
3. Inside the **INFIO Assessment Tool** folder, you will find three subfolders:
    - `aws-infra-setup`
    - `infio`
    - `infio-plugin`
4. Open the `infio` folder. Inside this folder, locate and run the **start_infio batch file** to start the application. It will run the INFIO tool in Google Chrome browser.
5. If you encounter the error "License validation failed. The app cannot be started," please contact Cornerstone Support for assistance with license validation at support@cornerstone-consulting.io.
6. Once the tool is launched, go to the `Assessment` page where INFIO will prompt you to enter the application name and company name. If your company name is not available in the list, you can enter the company name.

#### Infio Dashboard Overview

- When a user opens Infio, Infio directed to the **Dashboard** as the homepage.  

**Dashboard Sections**
The dashboard provides an overview of key metrics related to application assessments:  
- **Submitted Applications**: Number of applications submitted.  
- **Assessment Discovered**: Number of assessments identified.  
- **Assessments Run**: Number of assessments executed.  
- **Reports Generated**: Number of reports created based on assessments.  

**Application Status Table**
Displays the current status of applications across different stages:  
| Applications | Assessment Discovered | Assessments Run | Reports Generated |
|-------------|----------------------|----------------|------------------|
| 0 | 0 | 0 | 0 |

**Left Navigation Panel**
The sidebar contains the following options:  
- **🏠 Home** - Redirects to the dashboard.  
- **⚙️ Configure** - Allows users to set up configurations.  
- **🔍 Discover** - Helps in identifying available applications.  
- **📊 Assess** - Facilitates running assessments on discovered applications.  
- **📄 Reports** - Contains generated reports and their details.  
  - Expandable submenu for detailed report views.  
- **🚀 Deploy** - Deployment-related functionality.  
- **🔄 Migrate** - Migration-related functionality.

After dashboard page, you need to go to the configure page setup.

---

### Configuration Page Setup

On the configuration page, you need to click on the `create new config` button, where you will be prompted to provide information for various components. Follow these steps:

**Configure Application**
1. **Add New Company** or **Choose an Existing Company**.
2. Enter **Company Name**.
3. Enter **Application Name**.

**Configure Database**
1. Click **Add Server** to configure a new database.
2. Fill in the following details:
   - **Source DB Server Name**: Enter the database host name.
   - **Port**: Default is `1433` or you can enter your custom port number.
   - **Source Secret**: Choose from available secrets or you can add, edit, and delete existing secrets from drop down.
   - **Workload Type**: Select **PRODUCTION** or **NON-PRODUCTION**.
   - **Database to Include**: After entering the Source DB Server Name, click the Refresh button located just below the Secret Configurations section. This will load all the databases available on your source database server. Alternatively, you can leave this field empty to include all databases.
   - **Database to Exclude**: Select the databases you want to exclude. If left empty, the default system databases (master, model, msdb, tempdb) will be excluded automatically.
3. Click on **Secret Configurations** to:
   - **Add (+)** a new secret.
   - **Edit (✏️)** an existing secret.
   - **Delete (🗑️)** a secret.

**Actions**
- **Cancel**: Discard changes.
- **Submit**: Save the configuration.

> Note: You can add up to a total of five server databases within a single application.

Once the configuration is saved, users will see the following information at the top of the page:

- Application Name: Displays the name of the configured application.
- Company Name: Shows the name of the organization associated with the application.
- Bucket Name: Indicates the S3 bucket used for storing related data.
- Number of Servers: Specifies how many database servers are included in this configuration.

You can make necessary adjustments using the edit or delete options, ensuring that database configurations align with your requirements.

After that on the left sidebar, click on **Discover** to access the **Application Discovery** page.

---

### Application Discovery

**1. Application Name (Dropdown)**
- Click the refresh button and after that click on the dropdown.
- Select the name of the application you want to configure.
- If the application name is not available, ensure it has been added in the configuration settings.

**2. Select Servers (Dropdown)**
- Click the refresh button and after that and click on the dropdown to view available servers.
- Choose the servers associated with your application.
- Ensure that the required servers have been added in the **Configure** section before proceeding.

**3. Is your application a third-party or Commercial Off-the-Shelf (COTS) Solution? (Dropdown)**
- Select **Yes** if your application is a third-party or off-the-shelf solution.
- Select **No** if the application is custom-built or proprietary.

**4. Is there any additional information you would like to provide about your application? (Text Box)**
- Enter any relevant details about the application that might be useful for discovery.
- This could include the purpose of the application, specific configurations, or dependencies.

After filling in all the fields, click on **Discover Server Info** at the bottom of the page to proceed.
This will trigger the system to fetch **Server & Environment Metrics** related to the selection.

**Fill in the Server & Environment Metrics**

- Click on **Discover Server Info** to fetch server and environment details automatically.
- If any data is incorrect, update it manually and click **Update Server Info**.
- Ensure that the required configurations are set before proceeding.

**SQL Server Instance Count**
- Enter the number of **SQL Server Instances** in the respective fields for:
  - **Production**
  - **Non-Production**

**2. Total Number of CPU Cores**
- Enter the total **CPU Core count** for:
  - **Production**
  - **Non-Production**

**3. SQL Server Node Distribution**
- In the **Primary Node** field, enter the count of SQL Server primary nodes.
- In the **Secondary Node** dropdown, select the number of secondary nodes.

**4. Total Storage Distribution**
- Enter the total **EBS/RDS storage** (in GB).
- Enter the total **FSx-SAZ, FSx-MAZ, or Other storage** (in GB).

**5. Save the Changes**
- After entering all the required details, click on the **Update Server Info** button to save the changes.

After that on the left sidebar, click on **Assess** under the "Discover" section to access the **Assessment** page.

---

### Steps to Perform Assessment in INFIO

**1. Select the Target State**
- Ensure that the checkbox **"SQL Server to Babelfish"** is selected.

**2. Choose the Application**
- In the **"Select Application"** dropdown, choose the relevant application (e.g., `northwind`).

**3. Select the Server**
- Click on the **"Select Servers"** dropdown.
- Choose the appropriate server from the list (e.g., `ec2-1-23-456-789.compute-1.amazonaws.com`).

**4. Choose the Assessment Mode**

**1. Offline Mode**

- **Purpose**: Ideal for scenarios requiring manual collection of DMS, DDL, and object dependencies files using the INFIO plugin, which can then be uploaded to an S3 bucket.
- **Functionality**:
    - The INFIO plugin collects:
        - Data Definition Language (DDL) input files.
        - DMS Assessor input files.
        - Object Dependency input files.
    - **Manual Effort Required**: Profiler Extended Events must be manually collected from the source SQL server.

**2. Mixed Mode**

- **Purpose**: Combines automated and manual file collection, providing flexibility in how files are gathered.
- **Functionality**:
  - **Option 1: Split Collection of files between INFIO Plugin and INFIO Tool**:
    - The INFIO plugin collects:
      - Data Definition Language (DDL) input files.
    - The INFIO tool collects:
      - DMS Assessor input files.
      - Object Dependency input files.

  - **Option 2: All Files Collected by INFIO Plugin**:
    - The INFIO plugin collects:
      - Data Definition Language (DDL) input files.
      - DMS Assessor input files.
      - Object Dependency input files.

- **Manual Effort Required**:
  - In both approaches, Profiler Extended Events must be manually collected from the source SQL Server.

> **Note**: Before running an assessment and generating reports, you need to run the INFIO plugin and upload the files on S3 bucket based on the selected assessment mode.  
> For more details, refer to the [INFIO Plugin Documentation](https://github.com/cornerstone-consulting/INFIODocs/blob/main/infio-plugin.md).

**Summary**

- **Offline Mode** relies on the INFIO plugin for broader file collection.
- **Mixed Mode** distributes the workload between the INFIO plugin and the INFIO tool.

> **Important Note**: Make sure SQL Server is accessible from the INFIO EC2 instance security group.

**5. Review Database Configurations**
- Verify the displayed details:

**6. Run the Assessment**
- Click the **"Run Assessment"** button to start the assessment process.
 
 After successfully running assessment process on the left sidebar, click **Reports** and then select **Reports Overview**.
 
---

#### Generating the Summary Report

**Step 1: Select Report Generation Mode**
  - `Individual`: An individual report provides a comprehensive analysis of the feasibility and compatibility of migrating an existing **SQL Server database** to **Babelfish for Aurora PostgreSQL** or other AWS services for single application that you have selected. This report helps users understand potential migration challenges, highlights areas that require manual intervention, and offers insights to ensure a smooth transition.  
  - `Consolidated`: A **Consolidated Report** provides a unified summary of report results across multiple applications within a specific company. This report combines data from individual application assessments, offering a comprehensive overview of database compatibility, migration readiness, and potential issues. It is particularly useful for organizations managing multiple applications and needing a holistic view of their migration landscape.

**Step 2: Select the Application and/or company**
  - Select an application if you selected individual report generation mode or select company if you selected consolidated report generation for multiple applications within a specific company.


**Step 3: Generate the Report**
   - To generate the generate the reports, click the `Generate Report` button to create the report for the selected application.

**Step 4: Report Storage Locations**
  - The individual mode report will be saved in the specified S3 bucket in html and pdf format and you can download the report from below S3 path:
  ```
  S3: <application_name>/destination/reports/<application_name>.html/pdf
  ```
  - A local copy of the report will also be saved at INFIO EC2 instance:
    ```
    Local path: C:\Users\Administrator\infio\applications\<application_name>\destination\reports\<application_name>.html/pdf
    ```
  > **Note**: In both locations, `<application_name>` represents the name of the application for which the report was generated.

  The consolidated mode report can be accessed at the following location:  
  ```sh
  C:\Users\Administrator\infio\consolidated_reports\<Company Name>
  ```
  - This consolidated report enables users to review and analyze the overall migration strategy for all applications within the company efficiently.

**Step 5: Download and review**
    - You can download the summary report from the S3 bucket or INFIO EC2 instance.
    - Review the compatibility results and recommendations.

---

##### Migration Assessment Report Covers
The report evaluates the migration process by:  
- **Assessing Compatibility**: Identifies which database components are supported, unsupported, or require modifications.  
- **Estimating Migration Effort**: Highlights the level of effort needed for conversion, including schema changes and manual adjustments.  
- **Identifying Potential Issues**: Lists unsupported features and provides possible workarounds.  
- **Providing Schema and Code Analysis**: Examines the database structure, stored procedures, and application logic for compatibility gaps.  
- **Analyzing Cost (TCO)**: Compares cost estimates for different deployment models, helping businesses optimize infrastructure and licensing expenses.  

##### Report Sections Overview  

**1. Executive Summary**
- Summarizes the overall **migration feasibility** based on compatibility analysis.  
- Provides a **migration effort estimate** (low, medium, or high).  
- Highlights key database areas that need **manual review or modifications**.  

**2. Compatibility Analysis**  
- Presents **graphical insights** (charts, percentages) on the **schema compatibility**.  
- Breaks down database components into **fully compatible, partially compatible, or unsupported** categories.  

**3. Conversion Effort Matrix**
- Categorizes database objects based on **migration readiness**.  
- Identifies **stored procedures, triggers, constraints, and data types** that may require modifications.  
- Helps users estimate the level of effort required for successful migration.  

**4. Unsupported Features & Workarounds**  
- Lists **SQL Server features** that are not directly supported by Babelfish.  
- Suggests **possible solutions or changes** to adapt the database structure.  
- Highlights whether the required effort is **low, medium, or high**.  

**5. Schema and Object-Level Assessment**  
- Analyzes the **database structure**, including **tables, indexes, constraints, and keys**.  
- Identifies **potential performance issues** or missing optimizations.  
- Reports on any database objects that require changes before migration.  

**6. Total Cost of Ownership (TCO) Analysis**
The **TCO section** in the INFIO report provides an estimate of **migration costs** based on different deployment models. It helps organizations evaluate the **financial impact** of moving from **SQL Server to Babelfish for Aurora PostgreSQL** or other AWS-managed services.  

##### Key Insights from TCO Analysis  
- **Cost Comparison**: Evaluates multiple deployment options, comparing the estimated **monthly and yearly costs** across configurations.  
- **SQL Server Compute Resource Overview**: Analyzes the existing **infrastructure usage**, including instance count and compute resource allocation.  
- **TCO Bar Chart – Cost Breakdown**: Provides **graphical insights** into cost distribution among different AWS services.  
- **Key Assumptions**: Details **instance types, storage configurations, licensing models**, and estimated I/O usage, ensuring businesses make **informed financial decisions**.  

This section allows decision-makers to compare costs and choose the **most cost-effective migration strategy** based on licensing, infrastructure, and long-term operational expenses.  

---


#### SQL Server to Aurora PostgreSQL

If you choose **SQL Server to Aurora PostgreSQL**, follow the steps below:

The **SQL Server to Aurora PostgreSQL** assessment provides insights to help you convert your schema for conversion. This database conversion assessment generate report and it includes:

- **Executive Summary**: A high-level overview of the assessment results.
- **Recommendations**:
  - Conversion of server objects.
  - Backup suggestions.
  - Changes required for linked servers.

You can view this report in the **AWS DMS Console** or **S3** export it as a **PDF** or **CSV** file for further review.

#### Configure your data providers for DMS Schema conversion

Before configuring data providers, you must select the **Application Name** that you want to proceed with.
Once you have selected the application and then click on Data providers tab, and follow these steps to configure the **Source** and **Target Data Providers**.

##### **Source Data Provider**
1. Specify the **Source Server Port** (default: `1433`).
2. Enter the **Source Default Database Name** (e.g., `master`).
3. Provide the **Source Server Name** (e.g., `ec2-xx-xx-xx-xx.compute-1.amazonaws.com`).

##### **Target Data Provider**
1. Specify the **Target Server Port** (default: `5432`).
2. Enter the **Target Default Database Name** (e.g., `postgres`).
3. Select the **Target Server Name** from the dropdown (e.g., `info-dms-cs.cluster-xxxxx.us-east-1.rds.amazonaws.com`).

#### **Step 3: Save Configuration**
- Click **Create Data Providers** to save the configuration.

Once the data providers are configured, you can proceed with further schema conversion steps.

> **Important Note**: Make sure SQL Server and RDS instance are accessible from the INFIO EC2 instance security group.

#### **Steps to create a migration project**

After configuring the **Source** and **Target Data Providers**, follow these steps to create a **Database Migration Project**.

- Go to migrations tab, and click on create migration project button, and model will be open to create migration project. 
- The **DMS Project Name** will be auto-filled (e.g., `info-dms-commissions`).
- Choose the **Instance Profile** from the dropdown.
- Select **`infio-ec2-dms-role`** as the instance profile.
- ⚠ **Note:** The DMS instance profile is **private by default**. If necessary for debugging/troubleshooting, you can make it **public** through the AWS **IAM Console**.

##### **Configure Source Data Provider**
1. Verify the **Source Data Provider** (e.g., `info-sct-commissions-source`).
2. Select **Source Secret Name** from the dropdown.
   - Choose **`infio-source-db-credentials`** (pre-created during EC2 deployment from AMI).
3. Select **IAM Role for Source Secret**.
   - Choose **`infio-ec2-dms-role`**.

##### **Configure Target Data Provider**
1. Verify the **Target Data Provider** (e.g., `info-sct-commissions-target`).
2. Select **Target Secret Name** from the dropdown.
   - Choose **`infio-target-db-credentials`** (pre-created during EC2 deployment from AMI).
3. Select **IAM Role for Target Secret**.
   - Choose **`infio-ec2-dms-role`**.

##### **Configure S3 bucket setting**
1. Select **S3 bucket** from the dropdown.
   - Choose **`infio-private-bucket`** (pre-created during EC2 deployment from AMI).
3. Select **IAM Role for S3 bucket**.
   - Choose **`infio-ec2-dms-role`**.

##### **Save Configuration**
- Click **Create migration project** to proceed with the complete migration setup.


#### **Steps for Schema Conversion**

Once the **Migration Project** and **Data Providers** are created, follow these steps to perform **Schema Conversion**.

##### **Navigate to Schema Conversion Tab**
- Go to the **Schema Conversion** tab in the migration tool.

##### **Launch Schema Conversion**
- Click on **"Launch Schema Conversion"**.
- A status message will appear:  
  - **"Launching Schema Conversion, it may take a few minutes to complete."**
  - **"Schema Conversion is still in progress. Please wait..."**
- Wait for the process to complete.

##### **Export Schema Conversion Report**
- Once the schema conversion process is successful, the **"Export Schema Conversion Report"** button will become available and Infio prompted message of "Successfully launched schmea conversion...".
- Click **"Export Schema Conversion Report"**.
- The exported report will be stored in **S3 bucket**:  
  - **`infio-private-bucket`** (also visible in the status message).

#### **Verify Exported Report**
- Go to the **AWS S3 Console**.
- Navigate to **`infio-private-bucket`**.
- Locate the schema conversion report.

Once the report is available, you can see report location in status message, which is in **infio-private-bucket** bucket. 

---

### Monitoring

For monitoring purposes, the logs contain entries for errors, application startup events, and other activity logs. These logs are organized by month and can be found in the following directory:

```bash
Users/user_name/infio/logs/
```
![image](https://github.com/user-attachments/assets/5aa8fe3a-0fb4-40ed-9b76-4d051e6ac55e)

This structure allows you to easily access and review logs on a monthly basis.

---

### Recovery and Backup

INFIO stores only configuration data, such as secret keys, source-destination folder paths, S3 bucket names, and assessment-related questionnaires needed to run assessments and generate reports. Since this data is solely used for configuration and report generation, it does not require backup.

---
### Managing Licenses

INFIO deployment does not require specific license management. It uses open-source tools and technologies such as Babelfish Compass, Python, PostgreSQL, and Java. The focus remains on utilizing these tools effectively for the migration process, without any licensing constraints.

---

### Technical Support & Emergency Maintenance

For technical assistance and emergency maintenance, please contact Cornerstone Consulting Group at the following email:  
[support@cornerstone-consulting.io](mailto:support@cornerstone-consulting.io)  

Our support team will respond within 24 hours of receiving your email. They will assist with running assessments, generating reports, and guiding you on necessary execution and deployment steps. To ensure effective assistance, please provide as much detailed information as possible when reaching out. We are committed to delivering prompt and effective support to ensure a seamless deployment experience for our clients.

Please note, as outlined in the deployment guide, we offer a unified service level model rather than multiple service level agreements (SLAs). This approach simplifies support and ensures consistent service across our client base.

---

### FAQs  

**Q1: Why should I set the EC2 instance keyboard layout to English?**  
If the EC2 instance keyboard layout is set to a non-English language, users may experience:  
- **Login failures** due to incorrect password input (special characters may be mapped differently).  
- **Command-line errors** when executing scripts or commands in the terminal.  
- **Issues with remote access tools (RDP, SSH, or console login)** that expect an English keyboard layout.  

> **Solution:**  
Before using the EC2 instance, ensure the keyboard layout is set to **English (US)**:
- **Windows EC2**: Go to **Settings → Time & Language → Language & Region**, then set the keyboard to **English (United States)**.  

---

**Q2: What happens if the EC2 instance is stopped during the assessment?**  
A: The assessment process will be halted. You can restart the instance and rerun the service to continue from where it left off.  

**Q3: Can I use a different database type other than SQL Server?**  
A: Currently, the tool supports only SQL Server. Support for other database types may be added in future releases.  

**Q4: How can I deploy the INFIO tool in my AWS account?**  
A: There are two deployment options:  
- Cloud deployment through AWS Marketplace using INFIO AMI  
- On-premises deployment using the INFIO plug-in  

**Q5: How often should I update the tool and its dependencies?**  
A: It is recommended to check for updates quarterly to ensure compatibility with the latest AWS services and security patches.  

**Q6: How are database credentials managed securely?**  
A: Database credentials are stored and managed through AWS Secrets Manager, ensuring secure access to the SQL Server.  

**Q7: Is the data transfer between components secure?**  
A: Yes, all data transfers occur within a private subnet in the VPC, and connections to S3, Secrets Manager, and KMS key are made through secure channels using VPC endpoints.  

**Q8: What security measures are in place for data transfer?**  
A: Security measures include:  
- TLS encryption for all data in transit  
- VPC private subnet isolation  
- IAM role-based access control  
- Encryption at rest in S3  
- Secure credential management  

**Q9: Where are the assessment reports stored?**  
A: Assessment reports are stored in two locations:  
- Amazon S3 bucket for long-term storage  
- Locally on the EC2 instance for immediate access  

**Q10: What happens if the connection to the source SQL Server is lost during assessment?**  
A: The system will:  
1. Log the connection failure  
2. Resume from the last successful checkpoint if possible  

**Q11: What are the minimum system requirements for running INFIO?**  
A:  
- **For AWS deployment:**  
  - EC2 instance (recommended t3.medium or higher)  
  - VPC with private subnet  
  - Internet access for AWS services  

- **For On-premises:**  
  - Windows server meeting INFIO plug-in specifications  
  - Network connectivity to SQL Server  
  - S3 bucket, Secrets Manager, and KMS key access  

**Q12: What permissions are required for the IAM role?**  
A: The IAM role needs permissions for EC2, Secrets Manager, S3, and KMS.  

**Q13: How do I update the configuration file?**  
A: Edit the configuration file with the new database details and save it.  

**Q14: Can I run the assessment tool on multiple databases?**  
A: Yes, you can configure and run the tool on multiple databases by updating the configuration file.  

**Q15: When should I use VPC Endpoints in our environment?**  
A: If your EC2 instance is located in a private subnet without internet access, implementing a VPC endpoint is essential to access necessary AWS services. Confirm with your cloud network team whether your EC2 instance already has the required secure connectivity to CloudFormation, S3, Secrets Manager, and KMS services. If secure connectivity is in place, deploying additional VPC endpoints may not be necessary. Otherwise, you will need to deploy VPC endpoints to ensure the INFIO tool can securely communicate with these services.  

**Q16: Why is AWS DMS Schema Conversion (SC) failing?**  
If AWS DMS SC is failing, it could be due to networking issues. One possible reason is that the instance profile is private by default. Try setting the instance profile to public temporarily for debugging and troubleshooting from the AWS IAM console. Once the networking issue is resolved, revert it back to private for security best practices. Other potential issues can be **security groups, RDS or EC2 IP, database name, or incorrect credentials**.  

**Q17. How can I check logs for DMS Schema Conversion failures?**  
AWS DMS Migration Projects are directly linked to CloudWatch Logs, where detailed logs of failures are recorded. Check CloudWatch logs to identify the specific reason for failure, such as authentication issues, connectivity problems, or missing configurations.  

**Q18. What should I verify if authentication to the database is failing?**  
If authentication against the database is failing, ensure that:  
- The username and password in AWS Secrets Manager are correct.  
- The correct database name is being used (e.g., `master`, `postgres`, etc.).  
- The credentials match those required by the **source and target databases**.  

**Q19: How can subnet configuration impact AWS DMS connectivity?**  
If the DMS instance is unable to connect to the source or target database, check the subnets provided for creating the subnet group. The list of subnets provisioned during deployment should match the subnets that can communicate with the target RDS and source EC2 instance.  

**Q20: How does the DMS instance profile affect connectivity?**  
When an INFIO Assessment EC2 instance is launched, an instance profile is automatically created. Ensure that:  
- The instance profile has the necessary IAM permissions to access AWS resources.  
- The subnets where the DMS instance is deployed allow communication with the target RDS and source EC2 instance.  
- The security group rules allow inbound and outbound traffic as required.  

**Q21: Can I rerun a failed migration project, or do I need to delete and recreate it?**  
If a migration project fails, you need to delete the existing migration project from INFIO and then recreate it before running the migration again. This ensures a clean setup and avoids potential conflicts from previous failed attempts.  

**Q22: If the Schema Conversion (SC) runs multiple times, what happens to the files in S3?**  
If the SC process runs multiple times, the files in S3 get overwritten with the latest output. The existing files are replaced with the newly generated schema conversion results. To retain previous versions, consider enabling versioning on the S3 bucket or manually saving copies before rerunning the SC process.  

---
