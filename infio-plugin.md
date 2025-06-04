# Infio Plugin

## Table of Contents
- [Infio Plugin Overview](#infio-plugin-overview)
  - [Features](#features)
  - [Prerequisites](#prerequisites-for-using-the-infio-plugin)
  - [Modes of Operation](#modes-of-operation)
    - [Offline Mode](#1️⃣-offline-mode)
    - [Mixed Mode](#2️⃣-mixed-mode)
    - [Online Mode](#3️⃣-online-mode)
    - [Collect XML File from Extended Events Session](#4️⃣-collect-xml-file-from-extended-events-session)
  - [Configuration](#configuration)
    - [Extended Events Scheduler (Online Mode Only)](#extended-events-scheduler-online-mode-only)
    - [Extended Events XML File Retrieval Guide](#extended-events-xml-file-retrieval-guide)
  - [Uploading Files to an S3 Bucket or INFIO EC2 instance's INFIO directory](#uploading-files-to-an-s3-bucket-or-infio-ec2-instances-infio-directory)

### Infio Plugin Overview

The **Infio Plugin** simplifies migration from **SQL Server** to **Babelfish for Aurora PostgreSQL**. It automates the creation of essential dependency, assessor input files, helping you assess compatibility and identify issues during migration. 

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
#### 1. **Python and mssql-scripter**

If you are running the INFIO Plugin directly from a **SQL Server machine**, please make sure the dependencies below are installed.  
If you are running the plugin from the **INFIO EC2 instance**, all required dependencies are **pre-installed** and you can skip.

- **Python 3.8+** is required to execute the INFIO Plugin. You can download it from the official Python website: [Download Python](https://www.python.org/downloads/)
- After installing Python, install `mssql-scripter` using pip in command prompt of SQL Server machine:

  ```bash
  pip install mssql-scripter
  ```
  
#### 2. **SQL Server Credentials**  
To connect to your SQL Server instance, ensure the following:  

- **Access Credentials**:  
  - Have valid access credentials (username and password) for your SQL Server instance.  
  - Ensure your IP address is authorized to connect to the SQL Server instance.  

- **Read-Only Access**:  
  - The SQL Server username must have read-only access with the `db_datareader` role on all databases, including system databases to run Infio-plugin. Additionally, the msdb database, which is a system database, must have the `SQLAgentUserRole`, `SQLAgentReaderRole` and `SQLAgentOperatorRole` as these SQL agent roles are needed to run extended event tool and generate extended events xml file. 
  - This ensures only data retrieval operations are allowed, preventing changes to the database. 

#### 3. **Server Role Permissions**  
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

#### 4. **Granting Server-Level Permission: `ALTER ANY EVENT SESSION`**

To enable the user to create, modify, and delete **Extended Events** sessions to get extended events, grant the `ALTER ANY EVENT SESSION` permission.

**Steps to Grant the Permission:**

1. **Open SQL Server Management Studio (SSMS)** and connect to your SQL Server instance.
2. In **Object Explorer**, navigate to: **"Security" → "Logins"**
3. Find your user (e.g., `sql_user`). If the user does not exist, create a new login.
4. Right-click on the user and select **"Properties"**.
5. In the **Login Properties** window, click on **"Securables"** in the left pane.
6. Click **"Search"** → Select **"Specific objects"** → Click **"OK"**.
7. In the new window, select **"Server"** and click **"OK"**.
8. In the **Permissions for infiro** list, find **"ALTER ANY EVENT SESSION"**.
9. Check the **"Grant"** box next to **"ALTER ANY EVENT SESSION"**.
10. Click **"OK"** to apply the changes.

These permissions allow the INFIO Plugin to retrieve necessary information from the SQL Server without compromising security.

---

### Modes of operation

The **INFIO Plugin** offers different modes to accommodate various data collection and assessment requirements. Choose the mode that best fits your migration needs. 


#### **1️⃣ Offline Mode**  
🔹 **Purpose:** Best for scenarios where extended events collection is handled manually.  
🔹 **Process:**  
- **Automatically executed**:  
  - **Data Definition Language (DDL) Tool to collect DDL files**  
  - **Database Migration Service (DMS) Assessor Tool**  
  - **Object Dependency analysis files**  
- **Manually executed**:  
  - **SQL Server Profiler Events File Generation**  


#### **2️⃣ Mixed Mode**  
🔹 **Purpose:** A combination of automated and manual processes.  
🔹 **Process:**  
- **Automatically executed**:  
  - **Data Definition Language (DDL) Tool to collect DDL files**  
- **Manually executed**:  
  - **SQL Server Profiler Events File Generation**  


#### **3️⃣ Online Mode**  
🔹 **Purpose:** Fully automated assessment, ideal for real-time data collection and migration readiness analysis.  
🔹 **Process:**  
- **Automatically executed**:  
  - **Data Definition Language (DDL) Tool to collect DDL files**  
  - **Database Migration Service (DMS) Assessor Tool**  
  - **Object Dependency analysis files**  
- **INFIO Plugin Action:**  
  - Creates a **SQL Server Agent jobs** to manage **Extended Events sessions** dynamically.  


#### **4️⃣ Collect XML File from Extended Events Session**  
🔹 **Purpose:** Retrieve and analyze Extended Events session data manually.  
🔹 **Process:**  
- **INFIO Plugin retrieves** the generated **XML file** from an **Extended Events session**.  
- Users must **upload the XML file** to a designated location for further processing for database migration.  

Once an assessment mode is selected, the **INFIO Plugin** will:  
- Execute **automated data collection tasks** based on the selected mode.  
- For collecting SQL Server Profiler Events into XML file, refer to the [SQL Server Profiler Events Guide](https://github.com/cornerstone-consulting/INFIODocs/blob/main/profiler-events-guide.md).


After generating all the required files based on the selected assessment mode, users must upload them to either a designated S3 bucket or the INFIO EC2 instance's INFIO directory. Follow the [file upload process here](#uploading-files-to-an-s3-bucket-or-infio-ec2-instances-infio-directory) for detailed instructions.

---

### Configuration

If you choose any of the options (1, 2, or 3), you will be prompted to provide the following details, some of which are mandatory.

- **Application Name**: Enter the application name (should be the same as the one you entered during Infio tool setup).
- **SQL Server Name**: Hostname or IP address of the SQL Server instance. 
- **Username**: Your SQL Server username. 
- **Password**: Your SQL Server password.
- **Port**: Your SQL Server port number, you can press enter to keep default one which is 1433.
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
Enter SQL Server Port Number. Press enter to keep the default Port number (1433): 1234
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

## Uploading Files to an S3 Bucket or INFIO EC2 instance's INFIO directory

> Note: To run an assessment in the INFIO tool, users must upload the required files to either a designated S3 bucket or the INFIO EC2 instance's INFIO directory. Only one of these locations should be opted for file upload.

- **Option 1: Upload file to an S3 bucket**:

  1. **Log in to AWS Console**:
    - Open microsoft edge browser within INFIO EC2 instance.
    - Navigate to [AWS Console](https://aws.amazon.com/console/) in the browser tab.
    - Enter your aws credentials and log in.

  2. **Locate Your S3 Bucket**:
      - In the AWS services search bar, type 'S3' and click on it.
      - In the S3 Dashboard, find your bucket `infio-private-bucket`.
      - Click on the bucket name to open it.

  3. **Create directories in the S3 bucket**:
      - Inside the bucket, click on **Create folder**.
      - Name the folder `<application_name>` (replace with your application name that you entered while creating application in the INFIO tool, e.g., `my_app`).
      - The files you need to upload (DDL, DMS, Object Dependency, SQL Server Profiler Events XML file) depend on the assessment mode(Offline mode, Mixed mode) you have chosen in the INFIO tool.
        - **Offline Mode**: 
          - You will need to create directories for DDL, DMS, Object Dependency, and Profiler Events files to upload in the S3 bucket that you are currently in:
            - `<application_name>/source/ddl/`
            - `<application_name>/source/dms/sql_server_output_files/`
            - `<application_name>/source/object_dependency_sql_files/`
            - `<application_name>/source/sql_statements/`

        - **Mixed Mode**: 
            - Similarly, in **Mixed mode**, you will only need to create directories for DDL and SQL Server Profiler events files to upload in the S3 bucket that you are currently in:
              - `<application_name>/source/ddl/`
              - `<application_name>/source/sql_statements/`

  4. **Upload files from Your INFIO EC2 instace's infio-plugin directory**:
      - Before uploading files to the S3 bucket, users must run the INFIO plugin to generate the required files based on the selected assessment mode. The files generated by the INFIO plugin will be stored in the following directory on the INFIO EC2 instance:: `C:\Users\Administrator\infio-plugin\<application-name>\source\`.
      Here application name is application name that you entered while creating application in the INFIO tool.


      > Note: If users prefer not to manually create the SQL Server Profiler Events file, they can use the INFIO plugin to generate an Extended Events file. To upload the generated Extended Events file to an S3 bucket, follow the steps up to step 6. In step 7, instead of the Profiler Events file, upload the Extended Events file from: `C:\Users\Administrator\infio-plugin\<application-name>\source\extended_events\`
      to the target S3 directory: `s3://infio-private-bucket/<application-name>/source/sql_statements/`.
      This eliminates the need to follow step 7 as mentioned in the table below.
      

      | **Step** | **Action** | **Source Directory (INFIO EC2 Instance's INFIO Plugin directory)** | **Target Directory (S3 Bucket)** |
      |----------|-----------|--------------------------------------------|----------------------------------|
      | **1** | Navigate to the **target folder** in S3 | - | `s3://infio-private-bucket/<application-name>/source/ddl/` |
      | **2** | Click **Upload** at the top of the S3 screen | - | - |
      | **3** | Click **Add files** | Upload and select all DDL files _(For Both Modes)_ from: <br> `C:\Users\Administrator\infio-plugin\<application-name>\source\ddl\` | `s3://infio-private-bucket/<application-name>/source/ddl/` |
      | **4** | Click **Upload** to start the transfer | - | - |
      | **5** | Upload **DMS files** _(Only for Offline Mode)_ | Select all files from: `C:\Users\Administrator\infio-plugin\<application-name>\source\dms\` | `s3://infio-private-bucket/<application-name>/source/dms/sql_server_output_files/` |
      | **6** | Upload **Object Dependency files** _(Only for Offline Mode)_ | Select all files from: `C:\Users\Administrator\infio-plugin\<application-name>\source\database_dependency\` | `s3://infio-private-bucket/<application-name>/source/object_dependency_sql_files/` |
      | **7** | Upload **SQL Server Profiler Events file** _(For Both Modes)_ | - Generate SQL Server Profiler Events XML file by following [these steps](https://github.com/cornerstone-consulting/INFIODocs/blob/main/profiler-events-guide.md). <br> - Transfer the generated XML file to any location of INFIO EC2 instance and from where it can be uploaded as needed.| `s3://infio-private-bucket/<application-name>/source/sql_statements/` |


  5. **Verify the Uploaded Files**:
      - Navigate to each folder (`ddl`, `dms`, `object_dependency_sql_files` and `sql_statements`) in the S3 bucket.
      - Confirm that the files have been uploaded successfully.

- **Option 2: Upload file to INFIO EC2 instance's INFIO directory**:

  1. **Locate the Source Directories**
  - Before copying files, users must **run the INFIO plugin** to generate the required files based on the selected assessment mode.  
  - The INFIO plugin stores generated files in the following location:
    - `C:\Users\Administrator\infio-plugin\<application-name>\source\ddl\`
    - `C:\Users\Administrator\infio-plugin\<application-name>\source\dms\`
    - `C:\Users\Administrator\infio-plugin\<application-name>\source\database_dependency\`
    - (Replace `<application-name>` with the name entered while creating the application in the INFIO tool.) 

  2. **Copy Files to the Target Directories**
  - Open **File Explorer** and navigate to `C:\Users\Administrator\infio-plugin\<application-name>\source\`.
  - Depending on the **assessment mode** (`Offline Mode` or `Mixed Mode`) in the INFIO tool, copy the required files from their respective directories: 

    | **File Type** | **Source Directory (INFIO EC2 instance's INFIO Plugin directory)** | **Target Directory (INFIO EC2 instance's INFIO Tool directory)** |
    |--------------|------------------------|----------------------------|
    | DDL files | `C:\Users\Administrator\infio-plugin\<application-name>\source\ddl\` | `C:\Users\Administrator\infio\applications\<application-name>\source\ddl\` |
    | DMS files | `C:\Users\Administrator\infio-plugin\<application-name>\source\dms\` | `C:\Users\Administrator\infio\applications\<application-name>\source\dms\sql_server_output_files\` |
    | Object Dependency files | `C:\Users\Administrator\infio-plugin\<application-name>\source\database_dependency\` | `C:\Users\Administrator\infio\applications\<application-name>\source\object_dependency_sql_files\` |
  - Paste the copied files from source directories into their respective target directories.

  3. **Upload SQL Server Profiler Events File or Extended events file**
    - Collect SQL Server profiler events from SQL Server and generate an XML file by following [these steps](https://github.com/cornerstone-consulting/INFIODocs/blob/main/profiler-events-guide.md).  
    - Transfer the generated XML file to the INFIO EC2 instance.  
    - Copy the **Profiler Events XML file** and paste into to following directory:  

      ```
        C:\Users\Administrator\infio\applications\<application-name>\source\sql_statements\
      ```

    > Note: If users prefer not to manually create the SQL Server Profiler Events file, they can use the INFIO plugin to generate an Extended Events file. To upload the generated Extended Events file to an INFIO EC2 instance's INFIO tool directory, instead of the Profiler Events file, upload the Extended Events file from: `C:\Users\Administrator\infio-plugin\<application-name>\source\extended_events\`
    to the target INFIO EC2 instance's INFIO tool directory: `C:\Users\Administrator\infio\applications\<application-name>\source\sql_statements\`.
    This eliminates the need to generate and upload process of SQL Server Profiler Events file.

  4. **Verify the File Uploads**:
  - Navigate to the destination directories (`ddl`, `dms`, `object_dependency_sql_files`, and `sql_statements`) and confirm that the files have been uploaded successfully.

---
