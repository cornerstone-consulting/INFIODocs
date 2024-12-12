# Infio Plugin

## Table of Contents
- [Infio Plugin Overview](#infio-plugin-overview)
  - [Modes of Operation](#modes-of-operation)
    - [Offline Mode](#offline-mode)
    - [Mixed Mode](#mixed-mode)
    - [Profiler Extended Events](#profiler-extended-events)
  - [Features](#-features)
  - [Prerequisites](#-prerequisites)
- [Usage](#usage)
  - [Assessment Mode Selection](#assessment-mode-selection)
    - [Offline Mode](#1-offline-mode)
    - [Mixed Mode](#2-mixed-mode)
  - [Configuration](#configuration)
    - [Example Input](#example-input)
  - [Next Steps](#next-steps)
- [Uploading Files to an S3 Bucket](#uploading-files-to-an-s3-bucket)
  - [Important Notes Regarding File Uploads Based on Assessment Mode](#important-notes-regarding-file-uploads-based-on-assessment-mode)
  - [Method 1: Using AWS Console](#method-1-using-aws-console)
  - [Method 2: Using AWS CLI Copy Command](#method-2-using-aws-cli-copy-command)

### Infio Plugin Overview

The **Infio Plugin** simplifies migration from **SQL Server** to **PostgreSQL** using **Babelfish**. It automates the creation of essential dependency, assessor input files, helping you assess compatibility and identify issues during migration.

### Modes of operation

The INFIO Tool provides two distinct execution modes for assessments. Users can select the mode that aligns with their requirements and data collection preferences for running assessments. Below is a detailed explanation of each mode:

#### Offline mode
- **Automated**: Collects DDL, DMS, and Object Dependency Files.
- **Manual**: Profiler Extended Events (instructions provided after assessment).

#### Mixed mode
- **Automated**: DDL Dependency File.
- **Manual**: Profiler Extended Events, DMS and Database Dependency Files.

#### Profiler extended events
In both modes, **Profiler Extended Events** collection is a manual step. After the assessment, you'll receive instructions on how to capture them.

### 🌟 Features
- **Configuration Workflow**: Simple user input configuration for easy setup.
- **Object Dependency Analysis**: Identifies Object dependencies across SQL Server databases.
- **DMS Compatibility**: Generates DMS Assessor reports compatible with Migration Service.
- **Schema-level Dependency Assessment**: Analyzes DDL objects and their interdependencies.
- **Structured Output**: The outputs are saved as structured **SQL** and **HTML** files at a predefined location specified in the INFIO plugin tool, ensuring they are easy to parse and review.

### ✅ Prerequisites

Ensure the following before using the Infio Plugin:
- **SQL Server Credentials**: Ensure that you have valid access credentials for your SQL Server instance, and verify that your IP address is authorized to connect to the instance.
--
### 🚀 Usage

#### Assessment Mode Selection

When running the **Infio Plugin**, you will be prompted to choose an **Assessment Mode**. There are two options available:

#### 1. **Offline Mode**
- INFIO plugin will collect Data Definition Language (DDL), DMS Assessor and Object Dependency input data.
- Profiler Extended Events must be manually collected from the source server.

#### 2. **Mixed Mode**
- INFIO plugin will collect Data Definition Language (DDL) input data only. INFIO tool will collect DMS Assessor and Object Dependency input data.
- Profiler Extended Events must be manually collected from the source server. 

After selecting the mode, the Infio Plugin will proceed with the automated tasks, and at the end of the process, you'll receive detailed instructions for any manual steps for capturing Profiler Extended Events from SQL server.

### Configuration

After choosing assessment mode, you will be prompted to provide the following details (some of them are mandatory):

- **Application Name**: Enter the application name (should be the same as the one you entered during Infio tool setup).
- **SQL Server Name**: Hostname or IP address of the SQL Server instance. 
- **Username**: Your SQL Server username. 
- **Password**: Your SQL Server password.
- **Databases to Include**: Comma-separated list of databases to analyze, you can press enter to include all databases in the SQL Server.
- **Databases to Exclude**: Comma-separated list of databases to exclude from analysis, you can press enter to keep the default excluded databases only (master, model, msdb, tempdb).

> **Note**: Ensure that the SQL Server username and password provided are for a read-only access user. This ensures that only data retrieval operations are allowed, and no changes can be made to the database.


Example input:

```plaintext
=== Configuration ===
Enter Application Name (should be the same as the one you entered during Infio tool setup): db-migration-test
Enter SQL Server Name: ec2-12-34-567-890.compute-1.amazonaws.com
Enter SQL Server Username: admin
Enter SQL Server Password: 
Enter Databases to Include (comma-separated) Press enter to include all databases in the SQL Server: db1, db2
Enter Databases to Exclude (comma-separated) Press enter to keep the default excluded databases only (master, model, msdb, tempdb):
```

Once the configuration is completed, the plugin will display updates in the terminal as each tool is executed. These updates include the following messages according to you have performed assessment mode:
- "Running Object Dependency Tool..."
- "Running DMS Assessor Tool..."
- "Running DDL Tool..."
- "SQL file saved at: C:\Users\<user_name>\infio-plugin\<application_name>\source\database_dependency\sql_server_database_dependencies.sql"
- "Profiler Extended Events collection (manual process)" will be noted at the end of the tool executions.

#### **Next Steps**:
- After the tools are executed, you will receive detailed instructions for the manual step of collecting **Profiler Extended Events**.

## 🚀 Uploading Files to an S3 Bucket

After following the instructions to capture **Profiler Extended Events**, you will see the next steps for uploading the generated files into your S3 bucket. There are two ways to do this:

### **Important Notes Regarding File Uploads Based on Assessment Mode**

The files you need to upload (DDL, DMS, and Object Dependency) depend on the **assessment mode** you have chosen. Here's the breakdown:

- **Offline Mode**: 
    - You will need to upload **DDL, DMS and Object_dependencies folder's files** to the `<application_name>/source/ddl/`, `<application_name>/source/dms/` and `<application_name>/source/object_dependency` respectively folders of the S3 bucket.

- **Mixed Mode**: 
    - Similarly, in **Offline mode**, you will only need to upload the **DDL files** to the `<application_name>/source/ddl/` folder.

There are two methods to upload those files to S3 bucket. 
### **Method 1: Using AWS Console**

1. **Log in to AWS Console**:
    - Navigate to [AWS Console](https://aws.amazon.com/console/).
    - Enter your credentials and log in.

2. **Locate Your S3 Bucket**:
    - In the AWS services search bar, type 'S3' and select it.
    - In the S3 Dashboard, find your bucket (e.g., `infio-private-bucket`).
    - Click on the bucket name to open it.

3. **Create Folder Structure in the Bucket**:
    - Inside the bucket, click on **Create folder**.
    - Name the folder `<application_name>` (replace with your application name, e.g., `my_app`).
    - Inside the `<application_name>` folder, create the following subfolders:
      - `source/ddl/`
      - `source/dms/sql_server_output_files/`
      - `source/object_dependency/`
      - `source/sql_statements/`

4. **Upload Files from Your Local System**:
    - Navigate to the target folder, e.g., `source/ddl/`.
    - Click **Upload** at the top of the screen.
    - Click **Add files**/Add folder to select the files or folders from your local system’s `ddl`, `dms`, and `object_dependencies` folder.
    - Click **Upload** to start the transfer.
    - Repeat the upload process for the other folders (`source/dms/` and `source/object_dependency/`).

5. **Verify the Uploaded Files**:
    - Navigate to each folder (`ddl`, `dms`, `object_dependency`) in the S3 bucket.
    - Confirm that the files have been uploaded successfully.

> **Note**: Ensure that the files are in the correct folder structure in the S3 bucket.
---

### **Method 2: Using AWS CLI Copy Command**

Alternatively, you can use the AWS CLI to upload the files from your local machine directly to the S3 bucket. Follow these steps:

1. **Install AWS CLI (if not installed)**:
    - If you don't have the AWS CLI installed, follow the instructions in the [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

2. **Configure AWS CLI**:
    - Run the following command to configure your AWS credentials:
      ```bash
      aws configure
      ```
    - Enter your AWS Access Key ID, Secret Access Key, default region name, and default output format when prompted.

3. **Upload DDL Files**:
    - Run the following command to upload the DDL files:
      ```bash
      aws s3 cp /path/to/your/ddl/files s3://your-bucket-name/application_name/source/ddl/ --recursive
      ```
    - Replace `/path/to/your/ddl/files` with the local path where the DDL files are located.
    - Repeat the upload process for the other folders (`source/dms/` and `source/object_dependency/`).

> Replace `your-bucket-name` with your actual S3 bucket name and ensure the paths are correct for your local environment.
---
Both methods will ensure your files are properly uploaded to the appropriate locations within your S3 bucket.
