# Steps to Capture and Save SQL Server Profiler Trace events in XML Format  
 
## Step 1: Launch SQL Server Profiler  
- Open **SQL Server Management Studio (SSMS)**.  
- Go to **Tools** > **SQL Server Profiler**.  
- Click **File** > **New Trace**.  
- Connect to the SQL Server instance when prompted.  
 
## Step 2: Configure Trace Properties  
- In the **Trace Properties** window, enter a descriptive name in the **Trace name** field.  
- Check **Save to file** and select a location to store the trace data.  
- Set the **maximum file size** (default: 5MB).  
- Click **Run** to start capturing events.  
 
## Step 3: Select Events and Columns  
- Navigate to the **Events Selection** tab.  
- Select specific events to capture by checking the checkboxes.  
- *(Optional)* Click **Show all events** to capture all database activity.  
- Choose the required data columns for detailed analysis.  
- Click **Run** to begin tracing.  
 
## Step 4: Stop the Trace  
- Monitor the live trace data as SQL Server events are recorded.  
- Once you have collected enough data, click the **Stop Selected Trace** button.  
 
## Step 5: Save the Trace in XML Format  
- Click **File** > **Save As** > **Trace XML File**.  
- Choose a file name and save it with the **.xml** extension.  
- The trace data is now stored in an XML file for future analysis.  