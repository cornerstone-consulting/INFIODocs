# Infio Plugin

## Table of Contents
- [Infio Plugin Overview](#infio-plugin-overview)
  - [Features](#features)
  - [Prerequisites](#-prerequisites)
  - [Modes of Operation](#modes-of-operation)
    - [Offline Mode](#1️⃣-offline-mode)
    - [Mixed Mode](#2️⃣-mixed-mode)
    - [Online Mode](#3️⃣-online-mode)
    - [Collect XML File from Extended Events Session](#4️⃣-collect-xml-file-from-extended-events-session)
  - [Configuration](#configuration)
    - [Extended Events Scheduler (Online Mode Only)](#extended-events-scheduler-online-mode-only)
    - [Extended Events XML File Retrieval Guide](#extended-events-xml-file-retrieval-guide)
- [Uploading Files to an S3 Bucket](#uploading-files-to-an-s3-bucket)
  - [Important Notes Regarding File Uploads Based on Assessment Mode](#important-notes-regarding-file-uploads-based-on-assessment-mode)
  - [Method 1: Using AWS Console](#method-1-using-aws-console)
  - [Method 2: Using AWS CLI Copy Command](#method-2-using-aws-cli-copy-command)

### Infio Plugin Overview

The **Infio Plugin** simplifies migration from **SQL Server** to **PostgreSQL** using **Babelfish**. It automates the creation of essential dependency, assessor input files, helping you assess compatibility and identify issues during migration. 

---

### Features
- **Configuration Workflow**: Simple user input configuration for easy setup.
- **Object Dependency Analysis**: Identifies Object dependencies across SQL Server databases.
- **DMS Compatibility**: Generates DMS Assessor reports compatible with Migration Service.
- **Schema-level Dependency Assessment**: Analyzes DDL objects and their interdependencies.
- **Structured Output**: The outputs are saved as structured **SQL**, **HTML** and **XML** files at a predefined location specified in the INFIO plugin tool, ensuring they are easy to parse and review.

---

### Prerequisites for Using the INFIO Plugin  

Before using the INFIO Plugin, ensure the following requirements are met:  

#### 1. **Python Installation**  
To use the INFIO Plugin, Python must be installed on your machine:  
- **Version Required**: Python 3.12.7  
- **Download Python**:  
  If Python is not installed, download and install it from the official Python website:  
  [Download Python 3.12.7](https://www.python.org/downloads/release/python-3127/)  

- **Add Python to PATH**:  
  During the installation, make sure to check the option to add Python to your system's environment PATH. This ensures seamless usage from the terminal or command prompt.

#### 2. **Install Required Python Package**  
After installing Python, you need to install the **mssql-scripter** package:  
- Open a terminal or command prompt.  
- Run the following command:  
  ```bash
  pip install mssql-scripter
  ```
  
#### 3. **SQL Server Credentials**  
To connect to your SQL Server instance, ensure the following:  

- **Access Credentials**:  
  - Have valid access credentials (username and password) for your SQL Server instance.  
  - Ensure your IP address is authorized to connect to the SQL Server instance.  

- **Read-Only Access**:  
  - The SQL Server username must have **read-only access** with the `db_datareader` role.  
  - This ensures only data retrieval operations are allowed, preventing changes to the database.  

#### 4. **Server Role Permissions**  
The following server roles are required to run the INFIO Plugin:  

```plaintext
##MS_DatabaseConnector##
##MS_DefinitionReader##
##MS_PerformanceDefinitionReader##
##MS_SecurityDefinitionReader##
##MS_ServerPerformanceStateReader##
##MS_ServerSecurityStateReader##
##MS_ServerStateReader##
public
```
These permissions allow the INFIO Plugin to retrieve necessary information from the SQL Server without compromising security.

---

### Modes of operation

The **INFIO Plugin** offers different modes to accommodate various data collection and assessment requirements. Choose the mode that best fits your migration needs. 


#### **1️⃣ Offline Mode**  
🔹 **Purpose:** Best for scenarios where extended events collection is handled manually.  
🔹 **Process:**  
- **Automatically executed**:  
  - **Data Definition Language (DDL) Scripts**  
  - **Database Migration Service (DMS) tasks**  
  - **Object Dependency analysis**  
- **Manually executed**:  
  - **Profiler Extended Events Collection**  


#### **2️⃣ Mixed Mode**  
🔹 **Purpose:** A combination of automated and manual processes.  
🔹 **Process:**  
- **Automatically executed**:  
  - **Data Definition Language (DDL) Scripts**  
- **Manually executed**:  
  - **Profiler Extended Events Collection**  


#### **3️⃣ Online Mode**  
🔹 **Purpose:** Fully automated assessment, ideal for real-time data collection and migration readiness analysis.  
🔹 **Process:**  
- **Automatically executed**:  
  - **Data Definition Language (DDL) Scripts**  
  - **Database Migration Service (DMS) tasks**  
  - **Object Dependency analysis**  
- **INFIO Plugin Action:**  
  - Creates a **SQL Server Agent jobs** to manage **Extended Events sessions** dynamically.  


#### **4️⃣ Collect XML File from Extended Events Session**  
🔹 **Purpose:** Retrieve and analyze Extended Events session data manually.  
🔹 **Process:**  
- **INFIO Plugin retrieves** the generated **XML file** from an **Extended Events session**.  
- Users must **upload the XML file** to a designated location for further processing for database migration.  

Once an assessment mode is selected, the **INFIO Plugin** will:  
- Execute **automated data collection tasks** based on the selected mode.  
- Provide **detailed instructions** for any **manual steps** required to capture **Profiler Extended Events** from SQL Server.  

---

### Configuration

If you choose any of the options (1, 2, or 3), you will be prompted to provide the following details, some of which are mandatory.

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

#### Extended Events Scheduler (Online Mode Only)  

If you select **Online Mode (Option 3)**, the INFIO Plugin will prompt you to schedule the start and stop times for the **Extended Events session** just after running DDL, DML and object dependency tools. This allows for automated collection of performance data during the specified interval.  

#### **Steps:**  

##### **📌 Schedule Start Time:**  
- The plugin will display the **current SQL Server time**.  
- You will be prompted to enter the desired **start time** for the **Extended Events session**.  
- **Input Format:** 24-hour format (`HH:MM`).  
- **Validation:** The start time must be **later than the current SQL Server time**.  

##### **📌 Schedule Stop Time:**  
- After setting the **start time**, you will be prompted to enter the **stop time**.  
- **Input Format:** 24-hour format (`HH:MM`).  
- **Validation:** The stop time must be **later than the start time**.  

##### **📌 Example Input:**  
```plaintext
=== Extended Events Scheduler ===

Please schedule the time for Extended Events to run today.  
Please enter a value greater than the current SQL Server time, 14:30:00.  
Enter the time in the format of hours, minutes, and seconds.  
- Hours should be between 0 and 23 (24-hour format).  
- Minutes and seconds should be between 0 and 59.  

Enter hour (0-23): 15  
Enter minute (0-59): 0  

At what time do you want the Extended Events session to stop today?  
Enter the stop time in hours and minutes (24-hour format).  

Enter stop hour (0-23): 16  
Enter stop minute (0-59): 0  
```

Once the configuration is completed, the plugin will display updates in the terminal as each tool is executed. These updates include the following messages according to you have performed assessment mode:
- "Running Object Dependency Tool..."
- "Running DMS Assessor Tool..."
- "Running DDL Tool..."
- "Running Extended events Tool..."
- "SQL file saved at: C:\Users\<user_name>\infio-plugin\<application_name>\source\database_dependency\sql_server_database_dependencies.sql"
- After the tools are executed, you will receive detailed instructions for the manual step of collecting **Profiler Extended Events** and [uploading these files to an S3 bucket](#uploading-files-to-an-s3-bucket).

---

#### **Extended Events XML File Retrieval Guide**  

The **INFIO Plugin** provides an option to **retrieve and save Extended Events session data** as an XML file for further analysis. If you choose **Option 4: Collect XML File from Extended Events Session**, the plugin will guide you through extracting and saving the event data in XML file.  

##### **📌 Steps to Retrieve Extended Events XML File**  

##### **Step 1: Enter Extended Events Session Name**  
- The INFIO Plugin will prompt you to enter the **exact name** of the **Extended Events session** you previously created in option 3 for online mode.  
- The session name **must match exactly** as it exists in SQL Server (case-sensitive).  
- If an invalid session name is entered, an error will be displayed.  

```plaintext
Please enter the exact name of the Extended Events session that you have created (case-sensitive, e.g., performance_session_2024): 
```

If the session does not exist, the plugin will display:

```
Error: The session '<session_name>' does not exist in SQL Server. Please enter the correct session name.
```

##### **Step 2: Saving XML file**
The retrieved event data is converted into an XML file and saved locally.

The plugin will display the following message:

```
Please wait for a while; this process might take some time to complete.
✅ The Extended Events XML file has been successfully saved at the location: <source_directory>\extended_events\<session_name>.xml
```

---

## Uploading Files to an S3 Bucket

### **Important Notes Regarding File Uploads Based on Assessment Mode**

The files you need to upload (DDL, DMS, Object Dependency, extended events) depend on the **assessment mode** you have chosen. Here's the breakdown:

- **Offline Mode**: 
    - You will need to upload **DDL, DMS and Object_dependencies folder's files** to the `<application_name>/source/ddl/`, `<application_name>/source/dms/sql_server_output_files` and `<application_name>/source/object_dependency_sql_files` respectively folders of the S3 bucket.

- **Mixed Mode**: 
    - Similarly, in **Offline mode**, you will only need to upload the **DDL files** to the `<application_name>/source/ddl/` folder.

- **Online Mode & Collect XML file mode**: 
    - You will need to upload **DDL, DMS and Object_dependencies folder's files** to the `<application_name>/source/ddl/`, `<application_name>/source/dms/sql_server_output_files` `<application_name>/source/object_dependency_sql_files` and `<application_name>/source/sql_statements` respectively folders of the S3 bucket.

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
      - `source/object_dependency_sql_files/`
      - `source/sql_statements/`

4. **Upload Files from Your Local System**:
    - Navigate to the target folder, e.g., `source/ddl/`.
    - Click **Upload** at the top of the screen.
    - Click **Add files**/Add folder to select the files or folders from your local system’s `ddl`, `dms`, and `object_dependencies` folder.
    - Click **Upload** to start the transfer.
    - Repeat the upload process for the other folders (`source/dms/sql_server_output_files`, `source/object_dependency_sql_files/` and `source/sql_statements/`).

5. **Verify the Uploaded Files**:
    - Navigate to each folder (`ddl`, `dms`, `object_dependency` and `sql_statements`) in the S3 bucket.
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
    - Repeat the upload process for the other folders (`source/dms/sql_server_output_files`, `source/object_dependency_sql_files/` and `source/sql_statements/`).

> Replace `your-bucket-name` with your actual S3 bucket name and ensure the paths are correct for your local environment.

Both methods will ensure your files are properly uploaded to the appropriate locations within your S3 bucket.

---
