# Steps of necessary installation on ec2 instance for AMI creation

## Install AWS CLI on the Windows EC2 Instance

1. Open the Command Prompt (CMD) on your Windows EC2 instance.
2. Run the following command to download and install the AWS CLI:
    ```bash
    msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
    ```
    This will download the MSI installer and begin the installation process.
3. Once the installation is complete, check the AWS CLI version to ensure it is installed correctly by running:
    ```bash
    aws --version
    ```

## Copy CloudFormation Templates from S3 Bucket to EC2 Infio Assessment Tool

1. Use the following AWS CLI command to copy all CloudFormation templates from your S3 bucket to the Infio Assessment Tool folder of the EC2 instance located in the C drive:
    ```bash
    aws s3 cp s3://infio-cf-templates/ "C:/INFIO Assessment Tool/awsinfrasetup/" --recursive
    ```

## Install 64-bit JRE for Windows

1. To install other dependencies, first download the JRE by clicking the following URL: [Java 8 JRE Downloads](https://www.oracle.com/in/java/technologies/javase/javase8-archive-downloads.html).
    - You will be prompted to log in.
    - After logging in, click on the installer file `jre-8u202-windows-x64.exe` to download the file and begin the JRE installation process.
2. Make sure to include Java JRE in the environmental path:
    ```
    C:\Program Files\Java\jre1.8.0_202\bin
    ```

## Install Babelfish Compass for Windows

1. Download and install Babelfish Compass from this URL: [Babelfish Compass Releases](https://github.com/babelfish-for-postgresql/babelfish_compass/releases/tag/v2024-10b).
2. Unzip the `BabelfishCompass.zip` file.
3. Rename the existing version folder if necessary. By default, Compass reports are located at:
    ```
    C:\Users\Administrator\Downloads\BabelfishCompass
    ```
4. Verify Babelfish Compass installation on Windows by running the following command:
    ```bash
    C:\BabelfishCompass\BabelfishCompass.bat -help
    ```
5. After completing this, copy the 'BabelfishCompass' folder into the 'Users' folder in the C drive. Within the 'Users' folder, create a new folder named 'infio' (if it doesn’t already exist) and place the 'BabelfishCompass' folder inside it.

## Install PostgreSQL for Windows

1. Download and install PostgreSQL version 17 or 16.4 from this URL: [PostgreSQL Downloads](https://www.enterprisedb.com/downloads/postgres-postgresql-downloads) for Windows x86-64.
2. After downloading the installer file, install PostgreSQL.
3. Make sure to include the following paths in the environmental path:
    ```
    C:\Program Files\PostgreSQL\16\bin
    C:\Program Files\PostgreSQL\16\lib
    ```

## Install Python Package for Windows

1. Download and install Python 3.12.7 version for Windows x86-64 from this URL: [Python Downloads](https://www.python.org/downloads/windows/).
2. Make sure to check the box **Add python.exe to PATH** during installation.
3. After the installation, check if Python is installed correctly using the following CMD command:
    ```bash
    python -V
    ```

## Download and Set Up the Infio Code from GitHub

1. Extract the downloaded Infio code.
2. Go to the directory where the extracted Infio code is located:
    ```
    C:\Users\Administrator\Downloads\Infio Assessment Tool\
    ```
3. Copy the Infio Assessment Tool folder into the 'Users' folder in the C drive. If the 'Users' folder doesn't exist, create it, then place the 'Infio Assessment Tool' folder inside it.

## Set Up Virtual Environment and Install Dependencies

1. Open the Command Prompt (CMD) from within the Infio Assessment Tool directory.
2. Run the following command to create a virtual environment and install basic dependencies to run the application:
    ```bash
    python -m venv venv
    venv\Scripts\activate
    pip install -r requirements.txt
    ```
3. To run the application, execute the following command, which will give you a local URL:
    ```bash
    streamlit run .\scripts\app.py
    ```

## Set Up PostgreSQL Database

1. Launch the pgAdmin tool and log in using the password set during installation.
2. Once logged in, navigate to the query tool in pgAdmin and run the following query to create a new database:
    ```sql
    CREATE DATABASE infio;
    ```
3. Open the query editor from the `infio` database and input the following SQL statements to create a new database and tables:
    - Open the following file:
      ```
      C:\Infio Assessment Tool\modules\input\postgres\dependency_scripts.sql
      ```
    - Copy all SQL statements from the file and paste them into the pgAdmin query editor.
    - Click Execute to run the SQL commands.

4. Next, go to the following directory:
    ```
    C:\Infio Assessment Tool\modules\input\workaround
    ```
    - Open the files `keyword.sql` and `workaround.sql`.
    - Copy all SQL statements from these files and paste them into the pgAdmin query editor.
    - Click Execute to run the SQL commands.

## Set Environment Variables

1. Launch the CMD terminal and navigate to the directory where the Infio application is located.
2. Run the following commands one by one to set up the necessary environment variables:
    ```bash
    setx postgres_host localhost
    setx postgres_password <postgres_password>
    setx postgres_username postgres
    setx postgres_database infio
    setx postgres_port 5432
    ```
    **Note:** Replace `<postgres_password>` with the password you set during the PostgreSQL setup.

3. After setting the environment variables, close the terminal.
