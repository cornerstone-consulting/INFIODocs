# Infio Documentation

# Table of Contents
- [Deploy the Solution](#deploy-the-solution)
  - [Prerequisites to Deploy the INFIO Tool](#prerequisites-to-deploy-the-infio-tool)
    - [Skillsets](#1-skillsets)
    - [Others](#2-others)
  - [Prerequisites Checklist](#prerequisites-checklist)
    - [Required IAM Permissions](#1-required-iam-permissions)
    - [SQL Server Database Permissions](#2-sql-server-database-permissions)
- [Deployment Steps](#deployment-steps)
  - [How to Deploy EC2 from INFIO AMI](#how-to-deploy-ec2-from-infio-ami)
  - [Log in to a Windows INFIO EC2 Instance in a Private Subnet Using a Bastion Host](#log-in-to-a-windows-infio-ec2-instance-in-a-private-subnet-using-a-bastion-host)
  - [VPC Endpoint Deployment for CloudFormation Service (Optional)](#vpc-endpoint-deployment-for-cloudformation-service-optional)
  - [VPC Endpoint Deployment for S3, Secret manager, and KMS](#vpc-endpoint-deployment-for-s3-secret-manager-and-kms)
  - [Setting up AWS Resources via CloudFormation](#steps-for-setting-up-aws-resources-for-infio-assessment-tool-via-cloudformation)
- [INFIO Assessment Tool Usage Guide](#infio-assessment-tool-usage-guide)
  - [Filling Out the Questionnaire](#filling-out-the-questionnaire)
  - [Navigating to the Configuration Page](#navigating-to-the-configuration-page)
    - [Configuration Page Setup](#configuration-page-setup)
  - [Assessment Mode Selection](#assessment-mode-selection)
    - [Offline Mode](#1-offline-mode)
    - [Mixed Mode](#2-mixed-mode)
  - [Generating the Summary Report](#generating-the-summary-report)
  - [Generating the Consolidated Report for Multiple Applications](#generating-the-consolidated-report-for-multiple-applications)
  - [Exporting Assessment Answers](#exporting-assessment-answers)



### Prerequisites to Deploy the INFIO Tool

#### 1. Skillsets

To deploy the solution successfully, the following skills and knowledge are required:

- Familiarity with AWS.
- Proficiency in AWS CLI and various AWS services (such as S3, EC2, VPC, CloudFormation, KMS, and Secrets Manager).
- Familiarity with database migration processes.
- Basic understanding of PostgreSQL and SQL Server.

> **Important Note**: The INFIO does not require the use of AWS account root privileges for deployment or operation.

#### 2. Others

The INFIO EC2 instance includes all necessary tools to ensure secure and efficient SQL Server to PostgreSQL migration:

- **AWS CLI v2**: Facilitates deploying AWS services using CloudFormation.
- **PostgreSQL**: Preinstalled to store assessment data and configuration details, serving as a local database for quick access to critical migration information.
- **Babelfish Compass**: Preinstalled to identify SQL Server features incompatible with PostgreSQL and provide guidance for adjustments.

---

### Prerequisites Checklist

Before deploying the database migration assessment tool, ensure the following prerequisites are met:

1. Required IAM Permissions
2. SQL Server Database Permissions

### 1. Required IAM Permissions

Below are the required permissions for an IAM user to create AWS resources within an AWS account:

#### EC2 Permissions

- `ec2:CreateVpcEndpoint`: Allows creation of a VPC endpoint.
- `ec2:DescribeVpcEndpoints`: Grants permission to view details of existing VPC endpoints.
- `ec2:DeleteVpcEndpoints`: Permits deletion of VPC endpoints.
- `ec2:ModifyVpcEndpoint`: Allows modification of VPC endpoint attributes.
- `ec2:DescribeRouteTables`: Enables viewing associated route tables.
- `ec2:DescribeVpcs`: Grants permission to view VPC configurations.
- `ec2:DescribeSubnets`: Allows viewing subnet details within a VPC.
- `ec2:DescribeSecurityGroups`: Permits viewing of security groups.
- `ec2:CreateSecurityGroup`: Grants permission to create new security groups.
- `ec2:DeleteSecurityGroup`: Allows deletion of security groups.
- `ec2:RevokeSecurityGroupEgress`: Permits removal of outbound rules.
- `ec2:AuthorizeSecurityGroupIngress`: Allows adding inbound rules.
- `ec2:AuthorizeSecurityGroupEgress`: Grants permission to add outbound rules.

#### CloudFormation Permissions

- `cloudformation:DescribeStacks`: View details of stacks.
- `cloudformation:DescribeStackEvents`: See events related to stack creation, update, or deletion.
- `cloudformation:DescribeStackResources`: Retrieve information about stack resources.
- `cloudformation:GetTemplate`: Retrieve the CloudFormation template.
- `cloudformation:ValidateTemplate`: Validate a CloudFormation template.
- `cloudformation:CreateStack`: Create a new stack.

#### KMS Permissions

- `kms:DescribeKey`: View metadata of a specific KMS key.
- `kms:EnableKeyRotation`: Enable automatic key rotation.
- `kms:TagResource`: Tag a KMS key.
- `kms:Decrypt`: Decrypt data using the KMS key.
- `kms:Encrypt`: Encrypt data with the KMS key.
- `kms:GenerateDataKey`: Generate a data encryption key (DEK).
- `kms:CreateKey`: Create new KMS keys.
- `kms:CreateAlias`: Create an alias for a KMS key.

#### Secrets Manager Permissions

- `secretsmanager:ListSecrets`: List all secrets.
- `secretsmanager:UpdateSecret`: Update secret value or metadata.
- `secretsmanager:TagResource`: Add or modify secret tags.
- `secretsmanager:GetSecretValue`: Retrieve a secret's value.
- `secretsmanager:DeleteSecret`: Delete a specified secret.
- `secretsmanager:CreateSecret`: Create new secrets.
- `secretsmanager:GetRandomPassword`: Generate random passwords.

#### S3 Permissions

- `s3:GetEncryptionConfiguration`: View bucket encryption configuration.
- `s3:PutEncryptionConfiguration`: Modify bucket encryption settings.
- `s3:PutBucketTagging`: Tag S3 buckets with metadata.
- `s3:PutObjectTagging`: Tag individual objects in the bucket.
- `s3:PutBucketPolicy`: Apply or update bucket policies.
- `s3:PutBucketPublicAccessBlock`: Modify public access block configuration.
- `s3:GetBucketLocation`: Retrieve the geographical region of a bucket.
- `s3:ListBucket`: List objects in the bucket.
- `s3:GetObject`: Retrieve objects from the bucket.
- `s3:PutObject`: Upload objects to the bucket.
- `s3:CreateBucket`: Create a new S3 bucket.

#### CloudWatch Logs Permissions

- `logs:CreateLogGroup`: Create a new log group.
- `logs:CreateLogStream`: Create a log stream.
- `logs:PutLogEvents`: Upload log events to a stream.

### 2. SQL Server Database Permissions

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

---

## Deployment Steps

### How to Deploy EC2 from INFIO AMI

1.  Navigate to the [AWS Marketplace](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Faws.amazon.com%2Fmarketplace%3F%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fawsmp-contessa&forceMobileApp=0) to download the INFIO AMI. During this process, you will be prompted to answer several necessary questions, detailed below:
    - **Region**: Select the region that you want to deploy your EC2 instance. The default is `us-east-1`.
    - **Region**: Select the region that you want to deploy your EC2 instance. The default is `us-east-1`.
    - **EC2 Instance Type**: Select the instance type that you want to use for the EC2 instance. The default is `m6a.large`.
    - **Subnet ID**: Provide the ID of a subnet where the EC2 instance will be deployed.
    - **EC2 Encryption Key**: Alias of the KMS Customer managed encryption key you would like to use to encrypt the INFIO EC2 instance, default is AWS managed `aws/ebs` KMS key.
    - **VPCCIDR**: CIDR of the VPC where your INFIO EC2 instance will be deployed.
    - **VPCID**: VPC ID where your INFIO EC2 instance will be deployed.
    - **InboundCIDRIP**: CIDR or a single IP address that will be allowed to RDP inbound to the INFIO EC2 Instance.

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
   - **Bastion Host Security Group**:  
     - Allow inbound SSH (port 22) from your IP – For Linux-based bastion hosts.  
     - Allow inbound RDP (port 3389) from your IP – For Windows-based bastion hosts.  
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
From the AWS Console in the same account where you have deployed the **INFIO EC2 instance**, you can deploy an AWS VPC Endpoint for CloudFormation service. This step is necessary if your **VPC landing zone** does not support EC2 instances hosted in a private subnet to connect to AWS services (e.g., when EC2 instances in private subnets cannot access CloudFormation service). Access to the CloudFormation Service is required to deploy additional AWS resources from the **INFIO EC2 instance**. 

> **Note**: Verify network connectivity or consult your cloud network administrator before deploying this endpoint.

**Deployment Steps:**

1. **Create the Stack in AWS Console**  
   - Open **CloudFormation** in the AWS Console.  
   - Navigate to the **Stacks** section and click on the **Create stack** button.  
   - Choose **With new resources (standard)** from the dropdown.  

2. **Upload the Template**  
   - Under the **Prepare template** option, select **Choose an existing template**.  
   - Click on the **Upload a template file** button.
   - For the upload file, go to the `Desktop` of the Windows EC2 instance (**INFIO EC2 instance**).  
   - Locate the folder containing the **INFIO Assessment Tool**. Inside, you'll find the `aws-infra-setup` folder, which contains the `INFIO-CF-VPCEndpoint.json` CloudFormation template.  
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

### VPC Endpoint Deployment for S3, Secret manager, and KMS

**Purpose:** 
- This deployment is necessary if your **VPC landing zone** does not support **EC2 instances hosted in a private subnet** to connect with AWS services. The provided **CloudFormation template** creates a secure environment for the INFIO assessment tool within an AWS VPC. It establishes **VPC endpoints** for **S3**, **Secrets Manager**, and **KMS** services, enabling secure access from the INFIO EC2 instance.

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
   - Find the template file: `INFIO-S3-KMS-SM-VPCEndpoints.json`.
   - Open the command prompt from `aws-infra-setup` folder and run the below command.

```bash
aws cloudformation create-stack --stack-name INFIOVPCEndpoints \
    --template-body file://C:/Users/Administrator/Desktop/INFIO%20Assessment%20Tool/aws-infra-setup/INFIO-S3-KMS-SM-VPCEndpoints.json \
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

---

### Steps for setting up AWS Resources for Infio Assessment Tool via CloudFormation

This CloudFormation template provisions the necessary AWS resources for the **INFIO Assessment Tool**, including a KMS key, Secrets Manager secret, and S3 bucket. These resources are optimized to ensure secure data storage and management, enabling robust encryption and controlled access with AWS's advanced security features.

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

---

### INFIO Assessment Tool Usage Guide

1. Navigate to the `Desktop:\INFIO Assessment Tool` folder located on the desktop of the INFIO EC2 instance.
2. Inside the **INFIO Assessment Tool** folder, you will find three subfolders:
    - `aws-infra-setup`
    - `infio`
    - `infio-plugin`
3. Open the `infio` folder. Inside this folder, locate and run the INFIO **exe file** to start the application. It will run the INFIO tool in Google Chrome browser.
4. Once the tool is launched, go to the `Assessment` page where INFIO will prompt you to enter the application name and company name. If your company name is not available in the list, you can enter the company name.

**Filling Out the Questionnaire**

1. Complete all required fields in the questionnaire on the assessment page. This typically includes details about your application and its requirements.
2. Ensure that all information is accurate before proceeding to the next step.

 **Navigating to the AWS Secrets Manager Page**

1. Once the questionnaires are completed, navigate to the **AWS Secrets Manager** page where your SQL Server database credentials are stored.  
   - To view or edit these credentials, click on **Edit Secrets**. From there, you can access and modify the stored details.

2. If you need to store additional secrets in the **Secrets Manager**, follow these steps:  
   - Click on the **Create Secret** button.  
   - Ensure the secret name begins with `infio_`.  
   - Add the **username** and **password** to allow INFIO to automatically use these credentials in the configurations.

**Navigating to the Configuration Page**

1. After submitting the questionnaire, proceed to the configuration page setup.

**Configuration Page Setup**

On the configuration page, you need to click on the `add Configurations` button, where you will be prompted to provide information for various components. Follow these steps:

1. Select the application name and provide the details of your SQL Server, including the server name, username, and password.
2. Ensure that the SQL Server username and password provided are for a **read-only access user**. This ensures that only data retrieval operations are allowed, and no modifications can be made to the database.
3. INFIO retrieves the SQL Server credentials (username and password) directly from the **AWS Secrets Manager**, where they have been securely stored. Ensure that the secret name starts with `infio_` as per the setup instructions so select the secret from drop down.  
4. Specify the source and destination directories where your application's DDL, DMS, object dependencies, and profiler extended events are stored, as well as where assessment reports will be generated.
5. Enter the **S3 bucket name** where your application's DDL, DMS, object dependencies, and profiler extended events are stored.
6. Enter the database name(s) you want to include in the assessment. You can leave this blank to include all databases in the SQL Server.
7. Enter the database name(s) you want to exclude. Press enter to retain the default excluded databases (master, model, msdb, tempdb).
8. Specify any additional settings or configurations required for your SQL Server.
9. Enter the SQL Server **port number**.

> **Note**: Make sure SQL Server is accessible from the INFIO EC2 instance.

#### Final Review and Confirmation

1. Review all the information you have entered on the configuration page.
2. Make any necessary adjustments or corrections.
3. Once all details are correct, click the **"Submit"** or **"Finish"** button to complete the setup process.

#### Assessment Mode Selection

1. Now, you will be asked to choose assessment modes.
2. The INFIO Tool provides two distinct execution modes for assessments. You can select the mode that aligns with their requirements and data collection files preferences for running assessments. Below is a detailed explanation of each mode:

#### 1. **Offline Mode**

- **Purpose**: Ideal for scenarios requiring manual collection of DMS, DDL, and object dependencies files using the INFIO plugin, which can then be uploaded to an S3 bucket.
- **Functionality**:
    - The INFIO plugin collects:
        - Data Definition Language (DDL) input files.
        - DMS Assessor input files.
        - Object Dependency input files.
    - **Manual Effort Required**: Profiler Extended Events must be manually collected from the source SQL server.

#### 2. **Mixed Mode**

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
> For more details, refer to the [INFIO Plugin Documentation](https://github.com/AakashLimbad208/infio-docs/blob/main/infio-plugin.md).

**Summary**

- **Offline Mode** relies on the INFIO plugin for broader file collection.
- **Mixed Mode** distributes the workload between the INFIO plugin and the INFIO tool.

#### Generating the Summary Report

1. **Run the Assessment**
   - Ensure the necessary files are in the S3 bucket.
   - Select the application name and click the `Run Assessment` button.
   - After successfully running assessments, you need to understand the reports that INFIO will generate.

2. **Understanding the Generated Reports**
   - The INFIO tool will generate a Database Migration Assessment Report, evaluating the compatibility of the SQL Server database with Babelfish for Aurora PostgreSQL.
   - The report highlights supported and unsupported features, providing recommendations and workarounds for unsupported database objects, transaction commands, and specific data types.
   - Manual intervention may be required for tasks such as adding primary keys to tables and addressing incomplete backup strategies.
   - This report guides users to streamline migration processes and address compatibility challenges effectively.

3. **Generate the Report**
   - To generate the generate the reports, click the `Generate Report` button to create the report for the selected application.

4. **Report Storage Locations**
   - The report will be saved in the specified S3 bucket and you can download the report from below S3 path:
     ```
     S3: <application_name>/destination/reports/<application_name>.html
     ```
   - A local copy of the report will also be saved at INFIO EC2 instance:
     ```
     Local path: C:\Users\Administrator\infio\applications\<application_name>\destination\reports\<application_name>.html
     ```
     > **Note**: In both locations, `<application_name>` represents the name of the application for which the report was generated.

5. **Download and review**
    - You can download the summary report from the S3 bucket or INFIO EC2 instance.
    - Review the compatibility results and recommendations.

#### Generating the Consolidated Report for Multiple Applications

A **Consolidated Report** provides a unified summary of report results across multiple applications within a specific company. This report combines data from individual application assessments, offering a comprehensive overview of database compatibility, migration readiness, and potential issues. It is particularly useful for organizations managing multiple applications and needing a holistic view of their migration landscape.

To generate the Consolidated Report:

1. Navigate to the **Reports Page** in the INFIO tool.  
2. Click on the **`Generate Consolidated Report`** button.  
3. In the prompt, select the **Company Name** for which you want to consolidate the reports.  
4. INFIO will automatically aggregate the data from all applications linked to the selected company and generate the consolidated report.

The generated report can be accessed at the following location:  
```sh
C:\Users\Administrator\infio\consolidated_reports\<Company Name>
```
- This consolidated report enables users to review and analyze the overall migration strategy for all applications within the company efficiently.

#### **Exporting Assessment Answers**

To view and export the assessment answers you provided in the INFIO tool, follow these steps:

1. Navigate to the **Reports Section** in the INFIO tool.  
2. Select the **Application Name** for which you want to export the assessment answers.  
3. Once selected, INFIO will generate a CSV file containing all the answers you filled out on the assessment page.  

The exported CSV file will be saved at the following location:  
```sh
C:\Users\Administrator\infio\applications\<application_name>\destination\assessment\<Application_name>_assessment_report.csv
```
- This file allows you to review or share the assessment data for further analysis or reporting purposes.
- You can also generate the report from the reports page after selecting application name as well.
