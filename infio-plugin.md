# Infio Plugin

## Table of Contents
- [Infio Plugin Overview](#infio-plugin-overview)
- [Infio Plugin Collects](#infio-plugin-collects)
- [Use cases for the Infio Plugin](#use-cases-for-the-infio-plugin)
- [Key Benefits](#key-benefits)
- [Prerequisites](#prerequisites-for-using-the-infio-plugin)
- [Modes of Operation](#modes-of-operation)
- [Infio plugin Usage Guide](#infio-plugin-usage-guide)
- [Uploading Files to an S3 Bucket or INFIO EC2 instance's INFIO directory](#uploading-files-to-an-s3-bucket-or-infio-ec2-instances-infio-directory)

### Infio Plugin Overview

The Infio Plugin is a lightweight utility designed to extend Infio tool’s assessment capabilities by enabling seamless data collection directly from your SQL Server environment. It plays a crucial role in situations where direct integration with the SQL Server is limited or where additional insights—like real-time client side SQL query capture—are required.

---

### Infio Plugin Collects

The plugin gathers key information to help evaluate the feasibility and efficiency of migrating your SQL Server databases to AWS cloud-native services like Babelfish for Aurora PostgreSQL. Specifically, it collects:

1. **Schema Metadata & DDL Information:** Extracts definitions of database objects such as tables, views, indexes, stored procedures, and triggers to assess schema structure.

2. **Object Dependency Mapping:** Analyzes relationships and dependencies among database objects to understand complexity.

3. **DMS Assessor:** Evaluates the presence of certain data types and table features. It also checks backups, permissions, and replication setup to make sure the database is ready for AWS DMS.

4. **Client side SQL query via Extended Events:** Leverages SQL Server Extended Events to capture runtime client side SQL queries from client applications, providing insights into query patterns, and frequency.

---

### Use Cases for the Infio Plugin

The plugin is particularly useful in the following scenarios:

#### 1. No Direct Connectivity to the Infio Instance
- When your SQL Server environment is isolated or in a different network with no route to the Infio instance, the plugin acts as a bridge for local data collection and run plugin from SQL Server environment.

#### 2. Limited Network Bandwidth
- If the available network bandwidth cannot support the transfer of large volumes of data between the SQL Server and the Infio instance, the plugin helps perform the assessment locally on the SQL Server environment, reducing data transfer load.

#### 3. Real-time SQL Query Capture from SQL Server
- If you want to monitor live SQL query activity directly from the SQL Server but cannot run the plugin on the Infio instance itself, this plugin lets you capture and analyze real-time client-side queries right from the SQL Server environment.

#### 4. Collecting Real-time Queries from the Infio Instance
- In cases where the Infio instance has access to SQL Server and centralized monitoring is preferred, the plugin can securely fetch real-time client-side query activity from the Infio instance, enabling efficient and centralized query monitoring.


> Note: For the first three use cases, the INFIO Plugin must be executed from the source SQL Server environment. This requires setting up the necessary environment and dependencies on the SQL Server side to support the plugin. In the fourth use case, the plugin executed from Infio EC2 instance and it already has all the required prerequisites installed to support this functionality.

---

### Key Benefits

- Flexible Deployment: Works in various network configurations.
- Real-Time client side queries: Monitor & capture real time client side Sql queries using extended event session.
- Network Efficient: Optimized for different bandwidth and connectivity scenarios.

Choose your deployment scenario based on your network setup. The INFIO Plugin adapts to your environment, ensuring you get the insights you need regardless of your technical constraints.

---

### Prerequisites for Using the INFIO Plugin  

Before using the INFIO Plugin, ensure the following requirements are met:

#### 1. Python and mssql-scripter

- This applies specifically to use cases [1](#1-no-direct-connectivity-to-the-infio-instance), [2](#2-limited-network-bandwidth) and [3](#3-real-time-sql-query-capture-from-sql-server) only.
- If you are running the INFIO Plugin directly from a **SQL Server machine**, please make sure the dependencies below are installed.  
- If you are running the plugin from the **INFIO EC2 instance**, all required dependencies are **pre-installed** and you can skip.

- **Python 3.8+** is required to execute the INFIO Plugin. You can download it from the official Python website: [Download Python](https://www.python.org/downloads/)
- After installing Python, install `mssql-scripter` using pip in command prompt of SQL Server machine:

  ```bash
  pip install mssql-scripter
  ```

#### 2. Download input zip file and copy into infio-plugin directory

- Again this applies specifically to use cases [1](#1-no-direct-connectivity-to-the-infio-instance), [2](#2-limited-network-bandwidth) and [3](#3-real-time-sql-query-capture-from-sql-server) only.
- Download the input zip file from [here](https://github.com/cornerstone-consulting/INFIODocs/blob/main/input.zip).
- Extract the contents of the zip file and copy the extracted folder into the following path:
```bash
C:\Users\administrator\infio-plugin
```
- After copying the input folder, verify that it is located at the following path:
```bash
C:\Users\administrator\infio-plugin\input
```


#### 3. SQL Server Access: 
To connect to your SQL Server instance, ensure the following:  

- **Access Credentials**:  
  - Have valid access credentials (username, password and port) for your SQL Server.  
  - For use case [4](#4-collecting-real-time-queries-from-the-infio-instance) only, make sure your Infio EC2 instance's IP address and port are authorized to connect to the SQL Server instance.  

- **Read-Only Access**:  
  - The SQL Server username must have read-only access with the `db_datareader` role on all databases, including system databases to run Infio-plugin. 
  
- **SQL Agent Role**:
  - In addition to `db_datareader`, you must have the `SQLAgentUserRole`, `SQLAgentReaderRole` and `SQLAgentOperatorRole` roles assigned on the `msdb` system database. As these SQL agent roles are needed to run extended event tool and generate extended events `.xel` file. 
  - This ensures only data retrieval operations are allowed, preventing changes to the database. 

#### 4. Server Role Permissions
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

#### 5. Granting Server-Level Permission: `ALTER ANY EVENT SESSION`

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

The **INFIO Plugin** offers different modes to accommodate various data collection. Choose the mode that best fits your needs. 

#### 1. **Rapid Data Collection - Offline mode**

- **Purpose:** Ideal for scenarios where essential assessment related files need to be collected.

Files generated by the plugin:
- Data Definition Language (DDL) files.
- Database Migration Service (DMS) assessor files.
- Database Object Dependency files.

#### 2. **DDL-Only Collection - Mixed mode**
-  **Purpose:** It will generate only DDL files.  

Files generated by the plugin:
  - Data Definition Language (DDL) files.  


#### 3. **Full Data Collection - Online mode**   
- **Purpose:** Fully data collection mode, ideal for real-time clinet side SQL qurey collection files and migration readiness analysis.

Files generated by the plugin:
  - Data Definition Language (DDL) files.
  - Database Migration Service (DMS) assessor files.
  - Database Object Dependency files.
  - Create and schedule an Extended Events session on SQL Server using SQL Server Agent to run at the specified time and generate `.xel` file into SQL Server.


  > Note: Running the [Extended Events Retrieval Mode](#extended-events-retrieval-mode) is mandatory after completing the Full Data Collection – Online Mode to retrieve `.xel` file and process the Extended Events data to convert into `.xml` file. 


#### 4. **Extended events Retrieval mode**  
- **Purpose:** Retrieve and analyze the Extended Events .xel file from SQL Server, convert it to an .xml format, and store it locally for further processing.

Files generated by the plugin:
- Get Extended events `.xel` file from SQL Server and convert it into `.xml` format file.

---

### Infio plugin Usage Guide


#### 1. Infio Plugin opening guide

1. This applies specifically to use case [4](#4-collecting-real-time-queries-from-the-infio-instance) only or if you are running plugin from Infio EC2 instance. Navigate to the `Desktop:\INFIO Assessment Tool` folder located on the desktop of the INFIO EC2 instance.
2. Inside the **INFIO Assessment Tool** folder, you will find three subfolders:
    - `aws-infra-setup`
    - `infio`
    - `infio-plugin`
3. Open the `infio-plugin` folder. Inside this folder, locate and double click on the **infio-plugin.exe** to start the plugin application.
4. If you are running plugin from SQL Server instance, you just need to locate `infio-plugin` folder and double click on the **infio-plugin.exe** to start the plugin.


#### 2. Mode selection guide

- Upon launching the INFIO Plugin, you will be prompted to select one of the previously described modes of operation [here](#modes-of-operation).
- After that, enter the number corresponding to your chosen mode. It could be 1/2/3/4.


#### 3. Configuration

- If you choose one mode of operation 1, 2 or 3, you will be prompted to provide the following details, some of which are mandatory.

  - **Application Name**: Enter the application name (should be the same as the one you entered during Infio tool setup).
  - **SQL Server Name**: Enter the Hostname or IP address of the SQL Server instance. 
  - **Username**: Enter your SQL Server username. 
  - **Password**: Enter your SQL Server password.
  - **Port**: Enter your SQL Server port number, you can press enter to keep default one which is `1433`.
  - **Databases to Include**: Comma-separated list of databases to analyze, you can press enter to include all databases in the SQL Server.
  - **Databases to Exclude**: Comma-separated list of databases to exclude from analysis, you can press enter to keep the default excluded databases only (master, model, msdb, tempdb).


- If you choose mode 4, you will be prompted to provide only the following details.
  - **Application Name**: Enter the application name (should be the same as the one you entered during Infio tool setup).
  - **SQL Server Name**: Enter the Hostname or IP address of the SQL Server instance. 
  - **Username**: Enter your SQL Server username. 
  - **Password**: Enter your SQL Server password.
  - **Port**: Enter your SQL Server port number, you can press enter to keep default one which is `1433`.

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

#### 4. Executing tools

- After providing all the required SQL Server details, the plugin will execute the tool(s) to generate the corresponding file(s) and  display their output locations.

#### 5. Extended Events Scheduler (Online Mode Only)  

If you select [**Online Mode**](#3-full-data-collection---online-mode), the INFIO Plugin will prompt you to schedule the start and stop times for the **Extended Events session** just after running DDL, DML and database object dependency tools. This allows for automated collection of real time extended events data during the specified interval, and generate the `.xel` file into SQL Server.    

**Schedule Start Time:**  
- The plugin will display the **current SQL Server time**.  
- You will be prompted to enter the desired **start time** for the **Extended Events session**.  
- **Input Format:** 24-hour format (`HH:MM`).  
- **Validation:** The start time must be **later than the current SQL Server time**.  

**Schedule Stop Time:**  
- After setting the **start time**, you will be prompted to enter the **stop time**.  
- **Input Format:** 24-hour format (`HH:MM`).  
- **Validation:** The stop time must be **later than the start time**.  

  **Example Input:**  
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

- Once your session has run as scheduled, you can download the .xel Extended Events session file and convert it into an .xml file using [mode 4](#4-extended-events-retrieval-mode).

#### 6. **Extended Events XML File Retrieval Guide**  

The **INFIO Plugin** provides an mode to **retrieve and save Extended Events session data** as an XML file for further analysis. If you choose **Mode 4: Collect XML File from Extended Events Session**, the plugin will guide you through extracting and saving the extended event data in XML file. 

> Note: Before running the Extended Events Retrieval Mode, ensure that the scheduled job on the SQL Server (configured in [mode 3](#3-full-data-collection---online-mode)) has been executed.

- First it will prompt you to enter application name, SQL server name, SQL server username, SQL server password, and SQL server port number.
- After that Plugin will prompt you to enter the **exact name** of the **Extended Events session** you previously created in [mode 3 for online mode](#3-full-data-collection---online-mode).  
- The session name **must match exactly** as it exists in SQL Server (case-sensitive).  
- If an invalid session name is entered, an error will be displayed.  

  ```plaintext
  Please enter the exact name of the Extended Events session that you have created (case-sensitive, e.g., performance_session_2024): 
  ```

- If the session does not exist, the plugin will display:

  ```
  Error: The session '<session_name>' does not exist in SQL Server. Please enter the correct session name.
  ```

- This process may take some time to complete, depending on the file size of `.xel` and duration of the Extended Events session run for.
- The retrieved event data is converted into an XML file and saved locally.

---

#### 7. Generated File Locations

After running tools based on the selected mode, the plugin will store the generated files in their respective locations and display a prompt with the file paths.

- Database Object dependency files will be stored at:
`C:\Users\<user_name>\infio-plugin\<application_name>\source\database_dependency\sql_server_database_dependencies.sql`

- DDL files will be stored at:
`C:\Users\<user_name>\infio-plugin\<application_name>\source\ddl\<database_name+sql_server_name+sql_server_port>.sql`

- DMS Assessor file will be stored at:
`C:\Users\<user_name>\infio-plugin\<application_name>\source\dms\sql_server_output_files\<database_name>.html`

- Extended events xml file will be stored at:
`C:\Users\<user_name>\infio-plugin\<application_name>\source\extended_events\<session_name>.xml`

> Note: Here, `user_name` refers to your Windows username, `application_name` is the name you provided during setup, `database_name` is the SQL Server database name, and `session_name` is the name of the Extended Events session.

> Note: If you want to run the INFIO tool, you must upload or copy the generated files to either an S3 bucket or the infio directory on the INFIO EC2 instance. You can follow the steps outlined [here](#uploading-files-to-an-s3-bucket-or-infio-ec2-instances-infio-directory).

---

#### Uploading Files to an S3 Bucket or INFIO EC2 instance's INFIO directory

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
