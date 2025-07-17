# INFIO Plugin

## Table of Contents
- [INFIO Plugin Overview](#infio-plugin-overview)
- [What INFIO Plugin Collects](#what-infio-plugin-collects)
- [Use Cases for the INFIO Plugin](#use-cases-for-the-infio-plugin)
- [Key Benefits](#key-benefits)
- [Prerequisites](#prerequisites-for-using-the-infio-plugin)
- [Modes of Operation](#modes-of-operation)
- [INFIO Plugin Usage Guide](#infio-plugin-usage-guide)
- [Uploading Files to S3 Bucket or INFIO Directory](#uploading-files-to-s3-bucket-or-infio-directory)

## INFIO Plugin Overview

The INFIO Plugin is a lightweight utility designed to extend INFIO tool's assessment capabilities by enabling seamless data collection directly from your SQL Server environment. It plays a crucial role in situations where direct connection with the SQL Server is limited or where additional insights—like real-time client side SQL query capture—are required.

---

## What INFIO Plugin Collects

The plugin gathers key information to help evaluate the feasibility and efficiency of migrating your SQL Server databases to AWS cloud-native services like Babelfish for Aurora PostgreSQL. Specifically, it collects:

1. **Schema Metadata & DDL Information:** Extracts definitions of database objects such as tables, views, indexes, stored procedures, and triggers to assess schema structure.

2. **Object Dependency Mapping:** Analyzes relationships and dependencies among database objects to understand complexity.

3. **DMS Assessor:** Evaluates the presence of certain data types and table features. It also checks backups, permissions, and replication setup to make sure the database is ready for AWS DMS.

4. **Client-side SQL Query via Extended Events:** Leverages SQL Server Extended Events to capture runtime client-side SQL queries from client applications, providing insights into query patterns and frequency.

---

## Use Cases for the INFIO Plugin

The plugin is particularly useful in the following scenarios:

### 1. No Direct Connectivity of SQL Server to the INFIO Instance
When your SQL Server environment is isolated or in a different network with no route to the INFIO instance, the plugin acts as a bridge for local data collection and runs directly from the SQL Server environment.

### 2. Limited Network Bandwidth
If the available network bandwidth cannot support the transfer of large volumes of data between the SQL Server and the INFIO instance, the plugin helps collect data locally on the SQL Server environment, reducing data transfer load.

### 3. Real-time SQL Query Capture from SQL Server
If you want to monitor live SQL query activity directly from the SQL Server but cannot run the plugin on the INFIO instance itself, this plugin lets you capture and analyze real-time client-side queries right from the SQL Server environment.

### 4. Collecting Real-time Queries from the INFIO Instance
In cases where the INFIO instance has access to SQL Server and centralized monitoring is preferred, the plugin can securely fetch real-time client-side query activity from the INFIO instance, enabling efficient and centralized query monitoring.

> **Note:** For the first three use cases, the INFIO Plugin must be executed from any Windows machine. This requires setting up the necessary environment and dependencies on the SQL Server side to support the plugin. In the fourth use case, the plugin is executed from the INFIO EC2 instance and it already has all the required prerequisites installed to support this functionality.

---

## Key Benefits

- **Flexible Deployment:** Works in various network configurations
- **Real-Time Client-side Queries:** Monitor & capture real-time client-side SQL queries using extended event sessions
- **Network Efficient:** Optimized for different bandwidth and connectivity scenarios

Choose your deployment scenario based on your network setup. The INFIO Plugin adapts to your environment, ensuring you get the insights you need regardless of your technical constraints.

---

## Prerequisites for Using the INFIO Plugin

Before using the INFIO Plugin, ensure the following requirements are met:

### 1. Python and mssql-scripter

This specifically applies to the use cases of:
- [No Direct Connectivity of SQL Server to the INFIO Instance](#1-no-direct-connectivity-of-sql-server-to-the-infio-instance)
- [Limited Network Bandwidth](#2-limited-network-bandwidth)
- [Real-time SQL Query Capture from SQL Server](#3-real-time-sql-query-capture-from-sql-server)

> **Note:** If you are running the plugin from the **INFIO EC2 instance**, all required dependencies are **pre-installed** and you can skip installing **Python and mssql-scripter**.

If you are running the INFIO Plugin directly from a **SQL Server machine or any Windows machine**, please ensure the following dependencies are installed:

- **Python version 3.8+** is required to execute the INFIO Plugin. You can download it from the official Python website: [Download Python](https://www.python.org/downloads/)
- After installing Python, install `mssql-scripter` using pip in command prompt:

  ```bash
  pip install mssql-scripter
  ```

### 2. Download Input Files and Copy to infio-plugin Directory

This applies specifically to use cases of:
- [No Direct Connectivity of SQL Server to the INFIO Instance](#1-no-direct-connectivity-of-sql-server-to-the-infio-instance)
- [Limited Network Bandwidth](#2-limited-network-bandwidth)
- [Real-time SQL Query Capture from SQL Server](#3-real-time-sql-query-capture-from-sql-server)

> **Note:** If you are running the plugin from the **INFIO EC2 instance**, you can skip downloading the input ZIP file and placing it in the infio-plugin directory.

If you are running the INFIO Plugin directly from a **SQL Server machine or any Windows machine**, follow these steps:

1. Download the input zip file from [here](https://github.com/cornerstone-consulting/INFIODocs/blob/main/input.zip)
2. Place it in the machine where you're running the INFIO Plugin (SQL Server machine or any other Windows machine)
3. Extract the contents of the zip file
4. Copy the extracted folder to the following path:
   ```bash
   C:\Users\<user_name>\infio-plugin
   ```
   > **Note:** Create the infio-plugin directory manually if it does not exist. Replace `<user_name>` with your logged-in Windows username (e.g., administrator).

5. After copying, verify that the input folder is located at:
   ```bash
   C:\Users\<user_name>\infio-plugin\input
   ```

### 3. SQL Server Access Configuration

This guide provides an automated SQL script to configure all necessary permissions for the INFIO Plugin on your SQL Server instance. Instead of manually setting up permissions through multiple steps, you can use our single script that handles everything automatically.

#### Prerequisites
Before running the script:

1. **SQL Server Access:** You must have administrative privileges on the SQL Server instance
2. **SQL Server Agent:** Ensure SQL Server Agent is running on your SQL Server instance
3. **Login Creation:** The SQL Server login must already exist (the script does not create logins)

#### Configuration Options

1. **@LoginName (Required)**
   - Replace `'<SQL_SERVER_USERNAME>'` with your actual SQL Server login name
   - Example: 
     ```sql
     DECLARE @LoginName NVARCHAR(100) = 'sql_ro';
     ```

2. **@DatabaseList (Optional)**
   - Option A: Leave empty (`''`) to grant permissions on ALL databases
     ```sql
     DECLARE @DatabaseList NVARCHAR(MAX) = '';  -- All databases
     ```
   - Option B: Specify specific databases as comma-separated list
     ```sql
     DECLARE @DatabaseList NVARCHAR(MAX) = 'MyDB1,MyDB2,MyDB3';
     ```

Copy and run the following script in SQL Server Management Studio (SSMS):

```sql
-- ==========================================
-- CHANGE THESE TWO VALUES ONLY:
-- ==========================================

DECLARE @LoginName NVARCHAR(100) = '<SQL_SERVER_USERNAME>';  -- 🔄 Change to your username
DECLARE @DatabaseList NVARCHAR(MAX) = '';   -- 🔄 Leave empty '' for ALL databases

-- ==========================================
-- DO NOT MODIFY BELOW THIS LINE
-- ==========================================
DECLARE @SQL NVARCHAR(MAX);
DECLARE @DBName NVARCHAR(200);
-- Grant database-level permissions
EXEC('GRANT VIEW DATABASE STATE TO [' + @LoginName + ']');
EXEC('GRANT SELECT ON OBJECT::[sys].[sql_expression_dependencies] TO [' + @LoginName + ']');
-- Grant server-level permissions
EXEC('GRANT CONNECT SQL TO [' + @LoginName + ']');
EXEC('GRANT VIEW ANY DEFINITION TO [' + @LoginName + ']');
EXEC('GRANT VIEW SERVER STATE TO [' + @LoginName + ']');
EXEC('GRANT ALTER ANY EVENT SESSION TO [' + @LoginName + ']');
-- Create temp table for database list
CREATE TABLE #DBList (DatabaseName NVARCHAR(200));
-- Check if database list is empty
IF LTRIM(RTRIM(ISNULL(@DatabaseList, ''))) = ''
BEGIN
    -- No list provided - get ALL databases
    INSERT INTO #DBList (DatabaseName)
    SELECT name FROM sys.databases WHERE state_desc = 'ONLINE';
END
ELSE
BEGIN
    -- Add master and msdb to the list
    SET @DatabaseList = @DatabaseList + ',master,msdb';
    
    -- Parse comma-separated list into temp table
    DECLARE @xml XML = N'<r><![CDATA[' + REPLACE(@DatabaseList, ',', ']]></r><r><![CDATA[') + ']]></r>';
    INSERT INTO #DBList (DatabaseName)
    SELECT LTRIM(RTRIM(r.value('.', 'NVARCHAR(200)')))
    FROM @xml.nodes('//r') AS records(r);
END
-- Cursor to loop through specified databases only
DECLARE db_cursor CURSOR FOR
SELECT DISTINCT dl.DatabaseName 
FROM #DBList dl
INNER JOIN sys.databases sd ON dl.DatabaseName = sd.name
WHERE sd.state_desc = 'ONLINE';
OPEN db_cursor;
FETCH NEXT FROM db_cursor INTO @DBName;
WHILE @@FETCH_STATUS = 0
BEGIN
    -- Build dynamic SQL for each database
    SET @SQL = '
    USE [' + @DBName + '];
    -- ✅ Step 1: Create user if not exists
    IF NOT EXISTS (
        SELECT 1 FROM sys.database_principals 
        WHERE name = ''' + @LoginName + '''
    )
    BEGIN
        CREATE USER [' + @LoginName + '] FOR LOGIN [' + @LoginName + '];
    END;
    -- ✅ Step 2: Add to db_datareader role if not already a member
    IF NOT EXISTS (
        SELECT 1 
        FROM sys.database_role_members drm
        JOIN sys.database_principals r ON drm.role_principal_id = r.principal_id
        JOIN sys.database_principals m ON drm.member_principal_id = m.principal_id
        WHERE r.name = ''db_datareader'' AND m.name = ''' + @LoginName + '''
    )
    BEGIN
        EXEC sp_addrolemember ''db_datareader'', ''' + @LoginName + ''';
    END;
    ';
    
    EXEC sp_executesql @SQL;
    FETCH NEXT FROM db_cursor INTO @DBName;
END;
-- Cleanup
CLOSE db_cursor;
DEALLOCATE db_cursor;
DROP TABLE #DBList;
-- Grant MSDB SQL Agent roles
USE [msdb];
IF EXISTS (SELECT 1 FROM sys.database_principals WHERE name = @LoginName)
BEGIN
    EXEC('ALTER ROLE [SQLAgentUserRole] ADD MEMBER [' + @LoginName + ']');
    EXEC('ALTER ROLE [SQLAgentReaderRole] ADD MEMBER [' + @LoginName + ']');
    EXEC('ALTER ROLE [SQLAgentOperatorRole] ADD MEMBER [' + @LoginName + ']');
END;
USE [master];
PRINT 'Permissions granted successfully for user: ' + @LoginName;
```

#### Server-Level Permissions
The script automatically grants these server-level permissions:

1. **CONNECT SQL** - Allows connection to the SQL Server instance
2. **VIEW ANY DEFINITION** - Enables viewing of object definitions across the server
3. **VIEW SERVER STATE** - Allows viewing server-wide configuration and performance data
4. **ALTER ANY EVENT SESSION** - Permits creation and management of Extended Events sessions
5. **VIEW DATABASE STATE** - Enables viewing database-level configuration and state

#### Database-Level Permissions
For each database (either all or specified), the script:

1. Creates a database user if it doesn't exist
2. Adds the user to `db_datareader` role for read-only access
3. Ensures system databases (master and msdb) are always included

#### SQL Agent Permissions
The script grants these SQL Agent roles on the msdb database:

1. **SQLAgentUserRole** - Basic SQL Agent permissions
2. **SQLAgentReaderRole** - Read SQL Agent job information
3. **SQLAgentOperatorRole** - View and read SQL Agent job properties

These roles are essential for the Extended Events tool to generate .xel files.

#### Security Considerations
- ✅ **Read-Only Access:** The script ensures the user has only read permissions, preventing any data modifications
- ✅ **No Elevation:** The user cannot elevate privileges or modify security settings
- ✅ **Audit Trail:** All permission grants are logged in SQL Server's security logs

---

## Modes of Operation

The **INFIO Plugin** supports different modes to help you collect and analyze data for migration. Choose the mode that matches your needs:

### 1. Rapid Data Collection
- Quickly collects the most important assessment files
- Generates:
  - DDL (schema) files
  - DMS Assessor files
  - Database object dependency files

### 2. Full Data Collection
- Complete data collection with real-time query capture
- Generates:
  - DDL files
  - DMS Assessor files
  - Database object dependency files
  - Creates and schedules an Extended Events session (`.xel` file)

> **Important:** After running this mode, use the [Get XML File of Extended Events](#7-get-xml-file-of-extended-events) mode to download and convert the `.xel` file into `.xml` for analysis.

### 3. DDL Tool Only
- Generates only DDL and schema files for the chosen databases

### 4. DMS Assessor Tool Only
- Creates only DMS Assessor files for the chosen databases

### 5. Database Object Dependency Tool Only
- Generates files showing how your database objects depend on each other
- Helps identify possible migration issues

### 6. Extended Events Extractor Tool Only
- Schedules and starts an Extended Events session to capture `.xel` files

### 7. Get XML File of Extended Events
- Downloads the `.xel` file from SQL Server
- Converts it to `.xml` for review

### 8. File Relocation (Copy) Tool
- Moves generated folders and files (`ddl`, `dms`, `object_dependency`, `sql_statements`) into the correct INFIO directory for assessment

> **Note:** The File Relocation Tool automates the manual process of copying files from the infio-plugin directory to the INFIO tool directory. You can also perform these steps manually as described in the [Uploading Files section](#uploading-files-to-s3-bucket-or-infio-directory).

---

## INFIO Plugin Usage Guide

### 1. INFIO Plugin Execution

1. For use case [Collecting Real-time Queries from the INFIO Instance](#4-collecting-real-time-queries-from-the-infio-instance) or if running from INFIO EC2 instance:
   - Navigate to `Desktop:\INFIO Assessment Tool` folder on the INFIO EC2 instance desktop
   - Inside the **INFIO Assessment Tool** folder, you will find three subfolders:
     - `aws-infra-setup`
     - `infio`
     - `infio-plugin`
   - Open the `infio-plugin` folder and double-click **infio-plugin.exe**

2. If running from SQL Server instance:
   - Locate the `infio-plugin` folder
   - Double-click **infio-plugin.exe** to start the plugin

### 2. Mode Selection

- Upon launching, select one of the [modes of operation](#modes-of-operation) (1-8)
- Enter the number corresponding to your chosen mode

### 3. Configuration

Configuration requirements vary by mode:

#### For All Modes (except modes 7 and 8):

For all modes other than **Mode 7 (Get XML File of Extended Events)** and **Mode 8 (File Relocation Tool)**, the following inputs are required:

- **Application Name**: Must match the name used during INFIO tool setup
- **SQL Server Name**: Hostname or IP address of the SQL Server instance
- **Username**: SQL Server username
- **Password**: SQL Server password
- **Port**: SQL Server port number (default is `1433` if left blank)
- **Databases to Include**: Comma-separated list of databases to analyze, or press enter to include all databases
- **Databases to Exclude**: Comma-separated list of databases to exclude, or press enter to exclude only the default system databases (`master`, `model`, `msdb`, `tempdb`)

> **Note:** Providing explicit database include/exclude lists helps you fine-tune the scope of the assessment according to your migration priorities.

#### For Extended Events Extractor Tool (Mode 6):

The **Extended Events Extractor Tool** (Mode 6) is used to create and schedule an Extended Events session on your SQL Server instance, which will capture real-time client-side SQL query activity during a defined time window. This is valuable when you want to analyze live workload patterns as part of your migration assessment.

Along with the details above, you will also need:

- **Session Name**: Name of the Extended Event session to create
- **Directory Path**: Path on the SQL Server to store the `.xel` file (e.g., `C:\ExtendedEvents`)

**Schedule Configuration:**

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

#### For Get XML File of Extended Events (Mode 7):

The **Get XML File of Extended Events Tool** (Mode 7) is used to download the `.xel` file generated by a previously scheduled Extended Events session on the SQL Server and convert it into an `.xml` format for analysis within the INFIO assessment tool. This step is necessary when you want to review or include real-time SQL query events as part of your migration assessment.


- **Application Name**: Must match the name used during INFIO tool setup
- **SQL Server Name**: Hostname or IP address of the SQL Server instance
- **Username**: SQL Server username
- **Password**: SQL Server password
- **Port**: SQL Server port number (default is `1433` if left blank)
- **Session Name**: Must exactly match the name created in [Extended Events Extractor Tool](#6-extended-events-extractor-tool-only) (case-sensitive)
- **Directory Path**: Path where the `.xel` file is stored (must match the path used during Extended Event creation)

Example input:
```plaintext
=== Configuration ===
Enter Application Name: db-migration-test
Enter SQL Server Name: ec2-12-34-567-890.compute-1.amazonaws.com
Enter SQL Server Username: admin
Enter SQL Server Password: 
Enter SQL Server Port (default 1433): 
Enter the Extended event session name to extract the events: infio-session
Please enter the **exact same directory path** used earlier during Extended Events session creation to store the `.xel` files.(e.g., C:\\ExtendedEvents): C:\ExtendedEvents
```

#### For File Relocation Tool (Mode 8):

The **File Relocation Tool** (Mode 8) allows you to automatically move all generated assessment files — including DDL, DMS, object dependency, and SQL statements — from the INFIO plugin directory to the designated INFIO tool directory. This step is required to prepare your files for running the INFIO assessment.


- **Application Name**: Must match the name used during INFIO tool setup
- **Destination Folder Path**: Full absolute path to the INFIO tools directory (e.g., `C:\Users\administrator\infio\applications`)

Example input:
```plaintext
Enter the Application Name you used while creating the application in the INFIO tool (lowercase, max 16 chars, start with letter, no '--' or trailing '-'): db-migration-test
Enter the destination folder path to copy the files (e.g., C:\Users\<user_name>\infio\applications): C:\Users\administrator\infio\applications
```

> **Note:** The File Relocation Tool automates the manual process of copying files from the infio-plugin directory to the INFIO tool directory. You can also perform these steps manually as described in the [Uploading Files section](#uploading-files-to-s3-bucket-or-infio-directory).

### 4. Generated File Locations

After running tools based on the selected mode, files will be stored at:

- **Database Object Dependency files:**
  ```
  C:\Users\<user_name>\infio-plugin\<application_name>\source\object_dependency\sql_server_database_dependencies.sql
  ```

- **DDL files:**
  ```
  C:\Users\<user_name>\infio-plugin\<application_name>\source\ddl\<database_name+sql_server_name+sql_server_port>.sql
  ```

- **DMS Assessor files:**
  ```
  C:\Users\<user_name>\infio-plugin\<application_name>\source\dms\sql_server_output_files\<database_name+sql_server_name+sql_server_port>.html
  ```

- **Extended Events XML files:**
  ```
  C:\Users\<user_name>\infio-plugin\<application_name>\source\sql_statements\<session_name>.xml
  ```

> **Note:** Replace `<user_name>` with your Windows username, `<application_name>` with your application name, `<database_name>` with the SQL Server database name, and `<session_name>` with the Extended Events session name.

> **Important:** To run the INFIO tool assessment, you must upload or copy the generated files to either an S3 bucket or the INFIO directory on the INFIO EC2 instance. Follow the steps in the [next section](#uploading-files-to-s3-bucket-or-infio-directory).

---

## Uploading Files to S3 Bucket or INFIO Directory

> **Note:** To run an assessment in the INFIO tool, users must upload the required files to either a designated S3 bucket or the INFIO EC2 instance's INFIO directory. Choose only one location for file upload.

### Option 1: Upload Files to an S3 Bucket

1. **Log in to AWS Console:**
   - Open Microsoft Edge browser within INFIO EC2 instance
   - Navigate to [AWS Console](https://aws.amazon.com/console/)
   - Enter your AWS credentials and log in

2. **Locate Your S3 Bucket:**
   - In the AWS services search bar, type 'S3' and click on it
   - In the S3 Dashboard, find your bucket `infio-private-bucket`
   - Click on the bucket name to open it

3. **Create Directories in the S3 Bucket:**
   - Inside the bucket, click **Create folder**
   - Name the folder `<application_name>` (replace with your application name)
   
   The directories needed depend on your assessment mode:
   
   **Offline Mode:**
   - `<application_name>/source/ddl/`
   - `<application_name>/source/dms/sql_server_output_files/`
   - `<application_name>/source/object_dependency_sql_files/`
   - `<application_name>/source/sql_statements/`
   
   **Mixed Mode:**
   - `<application_name>/source/ddl/`
   - `<application_name>/source/sql_statements/`

4. **Upload Files from INFIO EC2 Instance:**

   Files are generated in: `C:\Users\Administrator\infio-plugin\<application-name>\source\`

   > **Note:** To avoid manually creating SQL Server Profiler Events files, use the INFIO plugin to generate Extended Events files. Upload the Extended Events file from `C:\Users\Administrator\infio-plugin\<application-name>\source\sql_statements\` to `s3://infio-private-bucket/<application-name>/source/sql_statements/`.

   | **Step** | **Action** | **Source Directory** | **Target Directory (S3)** |
   |----------|------------|---------------------|---------------------------|
   | 1 | Navigate to target folder in S3 | - | `s3://infio-private-bucket/<application-name>/source/ddl/` |
   | 2 | Click **Upload** | - | - |
   | 3 | Click **Add files** and select DDL files | `C:\Users\Administrator\infio-plugin\<application-name>\source\ddl\` | `s3://infio-private-bucket/<application-name>/source/ddl/` |
   | 4 | Click **Upload** to start transfer | - | - |
   | 5 | Upload DMS files *(Offline Mode only)* | `C:\Users\Administrator\infio-plugin\<application-name>\source\dms\` | `s3://infio-private-bucket/<application-name>/source/dms/sql_server_output_files/` |
   | 6 | Upload Object Dependency files *(Offline Mode only)* | `C:\Users\Administrator\infio-plugin\<application-name>\source\object_dependency\` | `s3://infio-private-bucket/<application-name>/source/object_dependency_sql_files/` |
   | 7 | Upload SQL Server Profiler Events file | Generate using [this guide](https://github.com/cornerstone-consulting/INFIODocs/blob/main/profiler-events-guide.md) | `s3://infio-private-bucket/<application-name>/source/sql_statements/` |

5. **Verify the Uploaded Files:**
   - Navigate to each folder in the S3 bucket
   - Confirm successful upload

### Option 2: Upload Files to INFIO EC2 Instance's INFIO Directory

> **Note:** This process can be automated using the [File Relocation Tool](#8-file-relocation-copy-tool) (Mode 8) in the INFIO Plugin.

1. **Locate the Source Directories:**
   - Run the INFIO plugin first to generate required files
   - Files are stored in: `C:\Users\Administrator\infio-plugin\<application-name>\source\`

2. **Copy Files to Target Directories:**

   | **File Type** | **Source Directory** | **Target Directory** |
   |---------------|---------------------|---------------------|
   | DDL files | `C:\Users\Administrator\infio-plugin\<application-name>\source\ddl\` | `C:\Users\Administrator\infio\applications\<application-name>\source\ddl\` |
   | DMS files | `C:\Users\Administrator\infio-plugin\<application-name>\source\dms\` | `C:\Users\Administrator\infio\applications\<application-name>\source\dms\sql_server_output_files\` |
   | Object Dependency files | `C:\Users\Administrator\infio-plugin\<application-name>\source\object_dependency\` | `C:\Users\Administrator\infio\applications\<application-name>\source\object_dependency_sql_files\` |

3. **Upload SQL Server Profiler Events or Extended Events File:**
   
   **Option A: SQL Server Profiler Events**
   - Generate XML file using [this guide](https://github.com/cornerstone-consulting/INFIODocs/blob/main/profiler-events-guide.md)
   - Transfer to INFIO EC2 instance
   - Copy to: `C:\Users\Administrator\infio\applications\<application-name>\source\sql_statements\`
   
   **Option B: Extended Events (Recommended)**
   - Use INFIO plugin to generate Extended Events file
   - Copy from: `C:\Users\Administrator\infio-plugin\<application-name>\source\sql_statements\`
   - To: `C:\Users\Administrator\infio\applications\<application-name>\source\sql_statements\`

4. **Verify File Uploads:**
   - Navigate to destination directories
   - Confirm all files are present and accessible

---