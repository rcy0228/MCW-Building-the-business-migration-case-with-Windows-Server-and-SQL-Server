## Exercise 1: SQL database migration

Duration: 90 minutes

Tailspin Toys needs to migrate their on-premises SQL Server database to Azure SQL Managed Instance. This is part of the migration strategy defined to migrate Tailspin Toys workloads to Azure.

In this exercise, you will go through the steps necessary to migrate Tailspin Toys' on-premises SQL Server database to Azure SQL Managed Instance.

   [SQL database migration](#exercise-1-sql-database-migration)
   
   - [Task 1: Review Creation of Azure SQL MI **(Read-Only)** ](#task-1-review-creation-of-azure-sql-mi-read-only)
   
   - [Task 2: Install Data Migration Assistant](#task-2-install-data-migration-assistant)
   
   - [Task 3: Assess on-premises database compatibility](#task-3-assess-on-premises-database-compatibility)
   
   - [Task 4: Backup on-premises SQL database](#task-4-backup-on-premises-sql-database)
   
   - [Task 5: Migrate database to Azure SQL MI](#task-5-migrate-database-to-azure-sql-mi)

### Task 1: Review Creation of Azure SQL MI **(Read-Only)**
 
  > **Note**: This is a **Read-Only** task. As Deploying the new instance of Azure SQL Managed Instance may take about 6 hour to complete. We have already created a *Managed Instance* for you. You can review this task to understand how the Azure Managed Instance is created and continue from the next task.

1. On the **Home** page within the Azure Portal, towards the top, select **Create a resource**.

    ![The Home page of the Azure Portal is shown with the 'Create a resource' link highlighted.](images/BM-Ex1-T1-S1.png "Create a resource on Azure Portal Home page")

2. Within the **Search services and marketplace** field, type **Azure SQL Managed Instance**, press Enter, and select it in the search results.

    ![The Azure Marketplace search results for Azure SQL Managed Instance are shown with Azure SQL Managed Instance highlighted.](images/BM-Ex1-T1-S2.png "Azure SQL MI in Azure Marketplace")

3. Select **Create**.

     ![Create Managed Instance.](images/BM-Ex1-T1-S3.png "Azure SQL MI in Azure Marketplace")

4. On the **Create Azure SQL Managed Instance** pane, set the following values:

    - **Subscription**: Your Azure subscription for this lab from the drop-down.
    -  **Resource group**: Select the resource group `SQLMI-Shared-RG`.
    - **Managed Instance name**: Enter a unique name, such as `sqlmi--cus`.
    - **Region**: Select Region same as that of the resource group.

     ![The Create Azure SQL Managed Instance pane is shown with fields highlighted and all values entered.](images/BM-Ex1-T1-S4.png "Create Azure SQL Managed Instance pane")

5. For **Compute + storage**, select **Configure Managed Instance**.

     ![The Compute + storage section of the Create Azure SQL Managed Instance pane is shown with the Configure Managed Instance link highlighted.](images/BM-Ex1-T1-S5.png "Compute + storage section with Configure Managed Instance link highlighted")

6. For the **Compute + storage** configured select the following values:

    - **Service tier**: General Purpose
    - **Hardware generation**: Standard-series
    - **vCores**: 8 vCores
    - **Storage in GB**: 64 GB

     ![The Compute + storage pane is shown with necessary values selected and highlighted.](images/BM-Ex1-T1-S6.png "Compute + storage pane with values entered")

7. Select **Apply**.

8. Under **Authentication**, set the **Authentication Method** value to **Use both SQL and Azure AD authentication**.

9. Under **Azure AD admin**, select **Set admin** and choose an Azure AD user for the Azure AD admin. You should choose your own User account.

    > **Note**: To choose the Azure AD admin, an organization account must be selected. A personal Microsoft Account cannot be used for this.

10. Enter a username to use for the **Managed Instance admin login** and a **Password** for this new Administrator user that will be created on the database server. Select **Next: Networking**.

    > **Note**: You can provide the following credentials
        
         Username: demouser
         Password: demo!pass1234567

     ![The Authentication section is shown with the Managed Instance admin login and Password fields entered and highlighted.](images/BM-Ex1-T1-S7.1.png "Authentication values are entered")

12. On the **Networking** pane, enter the following value and then select **Review + create**.

    - **Virtual network / subnet**: `vnet-sqlmi--cus/Management`. 

     ![The Networking tab of the Create Azure SQL Managed Instance pane is shown with the tailspin-spoke-vnet/AzureSQLMI subnet selected with the field highlighted.](images/BM-Ex1-T1-S8.png "Networking values entered")

13. Select **Create**.

     ![create Managed Instance.](images/BM-Ex1-T1-S9.png "Create Managed Insatnce")

   
### Task 2: Install Data Migration Assistant

1. In the Azure Portal, navigate to the Resource Group for the lab i.e, `tailspin-rg` , then navigate to the `tailspin-onprem-sql-vm` virtual machine. This is the simulated on-premises SQL Server VM that contains the database to migrate to Azure SQL MI.

    ![The Virtual machine pane for the Simulated on-premises SQL Server VM is shown in the Azure Portal.](images/BM-Ex1-T2-S1.png "Simulated on-premises SQL Server VM")

2. On the left, select **Bastion** under **Operations**.

    ![The Bastion link under Operations is shown.](images/BM-Ex1-T2-S2.png "Bastion link is highlighted")

3. Enter the **Username** and **Password**, then select **Connect**.
    
    > - **Username**: `demouser`
    > - **Password**: `demo!pass123`

    ![The Bastion pane of the tailspin-onprem-sql-vm Virtual machine is shown with the Username and Password fields entered and highlighted.](images/BM-Ex1-T2-S3.png "Bastion credentials shown entered")

4. In the **tailspin-onprem-sql-vm** virtual machine, go to **Server Manager**, and select **Local Server**.

    ![The Server Manager window is shown with Local Server selected.](images//BM-Ex1-T2-S4.png "Server Manager with Local Server highlighted")

5. Within **Local Server**, select the `On` text link for the **IE Enhanced Security Configuration** property.

    ![The Local Server pane is shown within Server Manager and the value of On is highlighted for IE Enhanced Security Configuration.](images/BM-Ex1-T2-S5.png "Server Manager with IE Enhanced Security Configuration highlighted")

6. On the **Internet Explorer Enhanced Security Configuration** dialog, select **Off** for **Administrators**, then select **OK**.

    ![The Internet Explorer Enhanced Security Configuration window is shown with the Administrators section having the Off value selected and highlighted.](images//BM-Ex1-T2-S6.png "IE Enhanced Security Configuration dialog with Administrators Off property highlighted")

7. In the **tailspin-onprem-sql-vm** virtual machine, open **Internet Explorer** then go to the following link and download the **.NET Framework 4.8 Runtime** installer. This will be needed to install the Microsoft Data Migration Assistant. Select **Run** to run the **.NET Framework 4.8 Runtime** installer.

    <https://dotnet.microsoft.com/en-us/download/dotnet-framework/thank-you/net48-web-installer>
    
    ![Open Internet Explorer.](images/BM-Ex1-T2-S7.1.png "Internet explorer")
    ![Enter the download link.](images/BM-Ex1-T2-S7.2.png "Run the .NET Framework")
    

8. Once it's finished downloading, and follow the prompts to install the .NET Framework. Accept the license terms and click install.

    ![The .NET Framework 4.8 Setup wizard is shown.](images/BM-Ex1-T2-S8.png ".NET Framework 4.8 Setup")
    
9. After the installation is completed on the **Installation is complete** dialogue box click on **Finish**.

      ![The .NET Framework 4.8 Setup wizard shows installation is completed.](images/BM-Ex1-T2-S9.png ".NET Framework 4.8 Setup")
   
    - You will recieve a pop-up to restart your VM click on **Restart Now**.
     
     ![The .NET Framework 4.8 Setup wizard asks for restarting the VM.](images/BM-Ex1-T2-S9.1.png ".NET Framework 4.8 Setup")

10. Using **Internet Explorer**, go to the following link and download the **Microsoft Data Migration Assistant**.
   
    <https://www.microsoft.com/en-us/download/details.aspx?id=53595>  
   
    - Click on download for **Data Migration Assistant**.
    
     ![Enter the DMA download link.](images/BM-Ex1-T2-S10.png "DMA")
   
     - Select **Run** to run the **Microsoft Data Migration Assistant** installer.
    
     ![ DMA Installer.](images/BM-Ex1-T2-S10.1.png "DMA Run")

11. Once it's finished downloading and follow the prompts to install the assistant.

    - On **Welcome to the Microsoft Data Migration Assistant Setup Wizard** click **Next**.
    
     ![The Microsoft Data Migration Assistant Setup wizard is shown.](images/BM-Ex1-T2-S11.1.png "Microsoft Data Migration Assistant Setup wizard")
    
    - Next on the **End-User License Agreement** accept the terms and click on **Next**.
    
     ![The Microsoft Data Migration Assistant Setup wizard is shown.](images/BM-Ex1-T2-S11.2.png "Microsoft Data Migration Assistant Setup wizard")
    
     - On the **Privacy Statemen**t click on **Install**.  

      ![The Microsoft Data Migration Assistant Setup wizard is shown.](images/BM-Ex1-T2-S11.3.png "Microsoft Data Migration Assistant Setup wizard")
      
      - Finally on the **Completed the Microsoft Data Migration Assistant Setup Wizard** click on **Finish**.
      
      ![The Microsoft Data Migration Assistant Setup wizard is shown.](images/BM-Ex1-T2-S11.4.png "Microsoft Data Migration Assistant Setup wizard")


### Task 3: Assess on-premises database compatibility

1. In the **tailspin-onprem-sql-vm** virtual machine, open the **Start menu**, then type **Microsoft Data Migration Assistant** to search the application, then select it to run.

   ![The Data Migration Assistant application is shown.](images/BM-Ex1-T3-S1.png "Data Migration Assistant")

2. On the left, select the Plus sign (`+`) button to create a new project, and enter the following values, then select **Create**.

    - **Project type**: Assessment
    - **Project name**: Tailspin
    - **Assessment type**: Database Engine
    - **Source server type**: SQL Server
    - **Target server type**: Azure SQL Database Managed Instance

    ![The New project dialog is shown with the required values entered in the fields.](images/BM-Ex1-T3-S2.png "Data Migration Assistant New project dialog with values entered")

3. On the **Options** tab, ensure the **Check database compatibility** and **Check feature parity** report types are selected, then select **Next**.

    ![The Options step is shown with Check database compatibility and Check feature parity options selected.](images/BM-Ex1-T3-S3.png "Data Migration Assistant Options pane")

4. On the **Connect to a server** prompt, enter `localhost` for the     **Server name**, and check the **Trust server certificate** option, then select **Connect**.

    ![The Connect to a server dialog box is shown with the values entered to connect to localhost and to trust the server certificate.](images/BM-Ex1-T3-S4.png "Connect to a server configured for localhost")

5. On the **Add sources** prompt, select the **WideWorldImporters** database, then select **Add**.

    ![The Add sources dialog box is shown with the WideWorldImporters database selected.](images/BM-Ex1-T3-S5.png "Add sources with WideWorldImporters database selected")

6. Select **Start Assessment** in the lower right.

    ![The Select sources step is shown with the WideWorldImporters database shown as previously selected and the Start Assessment button is highlighted.](images/BM-Ex1-T3-S6.png "Data Migration Assistant with the Start Assessment button highlighted")

7. On the **Review results** pane, you should see a message that states "**There are no feature parity issues with your server instance.**"

    ![The Review results step is shown with the 'There are no feature parity issues with your server instance' message shown.](images/BM-Ex1-T3-S7.png "Data Migration Assistant showing there are no feature parity issues")

8. On the top left of the **Review results** pane, select **Compatibility issues**. On the **Review results** pane, you should see a message that "**There are no compatibility issues with your database**".


    ![The Compatibility issues option is selected on the Review results pane.](images/BM-Ex1-T3-S8.png "Compatibility issues selected")

9. The Data Migration Assessment is complete. If there were feature parity or compatibility issues found, then you would need to address those before migrating the SQL Server database to Azure SQL MI.

### Task 4: Backup on-premises SQL database

1. In the **tailspin-onprem-sql-vm** virtual machine, open the **Start menu**, then type **Azure Data Studio** to search the application, then select it to run **Azure Data Studio**.

    ![The Search results in the Start menu show a search for Azure Data Studio.](images/BM-Ex1-T4-S1.png "Azure Data Studio in Start menu search")

2. On the left, select the **Extensions** tab, then select the **Azure SQL Migration** extension and install it.

    ![Azure Data Studio is shown displaying the Extensions pane with the Azure SQL Migration extension selected and the Install button is highlighted.](images/BM-Ex1-T4-S2.png "Azure SQL Migration extension highlighted")
 
   > **Note**: If you recive an error Unable to install the extention. Update your Azure Data Studio and reperform Step 2. 
     
    ![Error installing the SQL Migration extention.](images/BM-Ex1-T4-S2-Note1.png "Azure SQL Migration extension highlighted")
    ![Error installing the SQL Migration extention.](images/BM-Ex1-T4-S2-Note2.png "Azure SQL Migration extension highlighted")
 
  
3. Next, you need to enable Preview Features within Azure Data Studio. Select the **Manage** icon (shown as the Gear in the lower left corner of Azure Data Studio) and select **Settings**.

     ![The Manage menu is shown with the Settings option highlighted.](images/BM-Ex1-T4-S3.png "The manage menu open with Settings highlighted")

4. On the **Settings** pane, type **Enable Preview Features** in the search box at the top, then check the **Enable unreleased preview features** box for the **Workbench: Enable Preview Features** option that shows in the search results. This will autosave.

     ![The Settings pane is shown with search results for Enable Preview Features showing the Enable unreleased preview features option selected.](images/BM-Ex1-T4-S4.png "Azure Data Studio settings pane with Preview Features enabled")

5. Next, let's connect to the on-premises SQL Server. Select the **Connections** tab on the left side of Azure Data Studio, then select **New Connection**.

    ![The Connections pane is shown with the New Connection button highlighted.](images/BM-Ex1-T4-S5.png "Azure Data Studio connections tab with New Connection button shown")

6. On the **Connection** pane, enter the following values to connect to the on-premises SQL database, then select **Connect**:

    - **Connection type**: Microsoft SQL Server
    - **Server**: `localhost`
    - **Authentication type**: Windows Authentication
    - **Database**: `WideWorldImporters`

    ![The Connection Details pane is shown with values entered and fields highlighted.](images/BM-Ex1-T4-S6.png "Azure Data Studio with Connection pane shown having all values entered")

7. In the list of servers, right-click the **localhost, WideWorldImporters** server, then select **Manage**.

    ![The right-click menu for the 'localhost, WideWorldImporters' server is shown with the Manage option highlighted.](images/BM-Ex1-T4-S7.png "WideWorldImporters server with right-click menu shown and Manage option highlighted")

8. Select **Backup**.

    ![The Manage page for the database is shown with the Backup button highlighted.](images/BM-Ex1-T4-S8.png "Manage database with Backup button highlighted")

9. On the **Backup database** pane, make sure the **Backup type** is set to **Full**, select the **Reliability** option to **Perform checksum before writing to media**, then make a note of the location of the **Backup files**, and select **Backup**.

    ![The Backup database dialog box is shown with the Backup type set to Full and the Reliability set as desired.](images/BM-Ex1-T4-S9.png "Backup database pane")

10. Open **Internet Explorer**, navigate to the following URL.

    <https://azure.microsoft.com/en-us/products/storage/storage-explorer/#overview>
    
    - Scroll to the bottom of the page and click download **Microsoft Azure Storage Explorer**.
    
    ![Download Azure Storage Explorer.](images/BM-Ex1-T4-S10.1.png "Microsoft Azure Storage Explorer Setup")
    
    - Click **Run** on the installer pop-up.
    
    ![Download Azure Storage Explorer.](images/BM-Ex1-T4-S10.2.png "Microsoft Azure Storage Explorer Setup")
     
     - On the **Setup-Microsoft Azure Storage Explorer** wizard accept the License agreement and leave all settings default and install.  

    ![The Microsoft Azure Storage Explorer Setup wizard is shown.](images/BM-Ex1-T4-S10.3.png "Microsoft Azure Storage Explorer Setup")

11. Launch **Microsoft Azure Storage Explorer**.

12. Select **Sign in with Azure**.

    ![The Azure Storage Explorer window is shown with the Sign in with Azure button highlighted.](images/BM-Ex1-T4-S12.png "Azure Storage Explorer Sign in with Azure")

13. In the **Select Azure Environment** select **Azure**  for **Which Azure environment will you use to sign in?** and click **Next**.
     
     ![The Azure Storage Explorer window is shown with the Sign in with Azure button highlighted.](images/BM-Ex1-T4-S13.png "Azure Storage Explorer Sign in with Azure")

    - Sign in to **Microsoft Account** using your ODL-user.

    ![The Azure Storage Explorer window is shown with the Sign in with Azure button highlighted.](images/BM-Ex1-T4-S13.1.png "Azure Storage Explorer Sign in with Azure")
   
    - You will recive the following message after successfully logging in.
   
     ![The Azure Storage Explorer window is shown with the Sign in with Azure button highlighted.](images/BM-Ex1-T4-S13.2.png "Azure Storage Explorer Sign in with Azure")

14. In the **Explorer** pane, expand the Azure Subscription, locate the Storage Account that was previously created (named similar to `tailspinsqlmistore`), then expand **Blob Container** and select the **sql-backup** container.

    ![Storage Explorer is displaying the Storage Accounts list with the previously created storage account and the nested Blob Containers tree expanded with the sql-backup container selected.](images/BM-Ex1-T4-S14.png "Storage Explorer showing the SQL MI backup storage account expanded")

15. In the **sql-backup** container pane, select **Upload**, then select **Upload Files...**.

    ![The Upload button menu is shown with the Upload files option highlighted.](images/BM-Ex1-T4-S15.png "Storage Explorer with Upload button highlighted and menu for Upload files showing")

16. In the **Upload Files** dialog, in the **Selected files** field, select the **Database Backup File** (`.bak`) for the **WideWorldImporters** database that was previously created (C:\Backup), then select **Upload**.

    ![The Upload Files dialog box is shown with the sql database backup file selected within the Selected filed field.](images/BM-Ex1-T4-S16.png "Storage Explorer Upload File dialog with database backup file selected")

### Task 5: Migrate database to Azure SQL MI

1. Within **Azure Data Studio**, under the list of servers, right-click the **localhost, WideWorldImporters** server, then select **Manage**.

    ![localhost server is highlighted with right-click menu shown with the Manage option highlighted.](images/BM-Ex1-T4-S7.png "WideWorldImporters server with right-click menu shown and Manage option is highlighted")

2. Select the **Azure SQL Migration** option. Select the **Migrate to Azure SQL** button.

    ![The Manage server pane is shown with the Azure SQL Migration option highlighted.](images/BM-Ex1-T5-S1.png "Manage server pane with Azure SQL Migration option highlighted")

3. In **Step 1: Database for assessment**, select the **WideWorldImporters** database, then select **Next**.

    ![Step 1 Database for assessment step is shown with the WideWorldImporters database selected for assessment.](images/BM-Ex1-T5-S4.png "Step 1: Databases for assessment")

4. In **Step 2: Assessment results and recommendations**, select the **Azure SQL Managed Instance** option.

    ![Step 2 Assessment results and recommendations are shown with the Azure SQL target option of Azure SQL Managed Instance selected.](images/BM-Ex1-T5-S5.png "Step 2: Assessment results and recommendations")

5. Scroll down and select the **View/Select** button to select a database.

    ![The View/Select button to choose the Azure SQL Managed Instance to migrate to is highlighted.](images/BM-Ex1-T5-S6.png "View/Select Azure SQL Managed Instance button")

6. Select the **WideWorldImporters** database, and you should see a message stating `No issues for migrating to Azure SQL Managed Instance`., then select the **Select** button.

    ![WideWorldImporters database selected and 'no issues' message shown.](images/BM-Ex1-T5-S7.png "No issues found message is shown")

7. Verify the **Azure SQL Managed Instance** target is selected, notice that it now shows **1 of 1 databases selected**, then select **Next**.

    ![Step 2 Assessment results and recommendations are shown with Azure SQL Managed Instance option selected.](images/BM-Ex1-T5-S8.png "Step 2: Assessment results and recommendations")

8. In **Step 3: Azure SQL target**, enter connection information to your Azure Subscription by first clicking on **link account**.

    ![link azure account.](images/BM-Ex1-T5-Step8.1.png "linking azure account")

9. On the **linked account** pane select **Add an account**.
    
    ![add azure account.](images/BM-Ex1-T5-Step8.2.png "linking azure account")
    
10. Sign into your Azure account using the credentials provided in the **Environment details** page.

      ![add azure account.](images/BM-Ex1-T5-Step8.3.png "linking azure account")
      
     - Once logged in successfully you will revice **Your account was added successfully!** message on the browser. 
       
     ![add azure account.](images/BM-Ex1-T5-Step8.4.png "linking azure account")

11. Back on the Data studio,  **Linked accounts** select the Azure account that was added and **Close**.
 
       ![add azure account.](images/BM-Ex1-T5-Step8.5.png "linking azure account") 

12.  Now for **Step 3: Azure SQL target** enter the following connection information, then select **Next**.
     
       - **Subcription**: `Azure Labs H-02`
       - **Location**: `Central US`
       - **Resource group**: `SQLMI-Shared-RG`
       - **Azure SQL Managed Insatnce**: `sqlmi--cus`

       ![Step 3 Azure SQL target is shown with the Azure account entered and the Location, Resource group, and Azure SQL Managed Instance resource selected as the target for the migration.](images/BM-Ex1-T5-Step8.6.1.png "Step 3: Azure SQL target")

13. On **Step 4: Migration mode**, keep **Online migration** selected, then select **Next**.

    ![Step 4 Migration mode is shown with the Online migration option selected.](images/BM-Ex1-T5-Step9.png "Step 4: Migration mode")

14. In **Step 5: Database backup**, select **My database backups are in an Azure Storage Blob Container**, provide the following details and then select **Next**.
 
    - **Target database name**: `WideWorldImporters`
    - **Resource group**: `tailspin-rg`
    - **Storage Account**: `tailspinsqlmistore`
    - **Blob container**: `sql-backup`

    ![Step 5 Database backup is shown with the 'My database backups are in an Azure Storage Blob Container' option selected, with the Storage account and sql-backup container selected for the location of the source database.](images/BM-Ex1-T5-Step10.png "Step 5: Database backup")

15. In **Step 6: Azure Database Migration Service**, select **Create new** under **Azure Database Migration Service**.

    ![Step 6 Azure Database Migration Service is shown with the Create new link under Azure Database Migration Service highlighted.](images/BM-Ex1-T5-Step11.png "Step 6 Azure Database Migration Service")

16. In the **Create Azure Database Migration Service** pane, enter the following values, then select **Create**.

    - **Resource group**: `tailspin-rg`.
    - **Name**: `tailspin-sql-migration`
    
    ![The Create Database Migration Service dialog is shown with Resource Group and Name for the Azure Database Migration Service to create entered.](images/BM-Ex1-T5-Step12.png "Create Database Migration Service dialog")

17. Once the Database Migration Service has been created, select **Done**.

    ![Step 6 Azure Database Migration Service is created.](images/BM-Ex1-T5-Step13.png "Step 6: Azure Database Migration Service")

18. In **Step 6: Azure Database Migration Service**, select the **Azure Database Migration Service** that was created, then select **Next**.

    ![Step 6 Azure Database Migration Service is shown with the Azure Database Migration Service field now entered.](images/BM-Ex1-T5-Step14.png "Step 6: Azure Database Migration Service")

19. In **Step 7: Summary**, review all the configurations chosen, then select **Start migration**.

    ![Step 7 Summary is shown with all the selected values displayed for review.](images/BM-Ex1-T5-Step15.png "Step 7: Summary")

20. Azure Data Studio will now show **Database migrations in progress - 1**.

    ![The Azure SQL Migration pane in Azure Data Studio shows there is 1 data migration in progress.](images/BM-Ex1-T5-Step16.png "Azure Data Studio showing there is 1 data migration in progress")

21. In the Azure Portal, navigate to the **Azure Database Migration Service** that was created `tailspin-sql-migration` under `tailspin-rg`, then select **Migrations** and the **WideWorldImporters** migration.

    ![The Azure Database Migration Service is shown within the Azure Portal displaying the new database migration in the list of Migrations.](images/BM-Ex1-T5-Step17.png "Azure Database Migration Service list of migrations.")

22. The **WideWorldImporters** migration shows the current status of the migration as `InProgress`. Notice the **Currently restoring files** should say **All backups restored** once the database backup has been restored. Then select **Complete cutover** at the top.

    ![The WideWorldImporters migration is shown within the Azure Portal having a migration status of InProgress](images/BM-Ex1-T5-Step18.png "WideWorldImporters migration showing status as InProgress")

23. In the **Complete cutover** prompt, select the box for **I confirm there are no additional log backups to provide and want to complete cutover.**, then select **Complete cutover**.

    ![The Complete cutover prompt is shown with the confirmation box checked and the Complete cutover button highlighted.](images/BM-Ex1-T5-Step19.png "Complete cutover")

24. The **WideWorldImporters** Migration will now show the status of **Completing**. This will take a few minutes to complete.

    ![The WideWorldImporters migration is shown in the Azure Portal having a migration status of Completing.](images/BM-Ex1-T5-Step20.png "WideWorldImporters migration showing status of Completing")

25. Once the cutover has been completed, the **WideWorldImporters** migration will show a status of **Succeeded**.

    ![The WideWorldImporters migration is shown in the Azure Portal with a migration status of succeeded.](images/BM-Ex1-T5-Step21.png "WideWorldImporters migration showing status of succeeded")

26. Within the Azure Portal, type **Azure SQL Managed Instance** on the **Search resources, services and docs** and select **SQL managed instances**.

      ![Search for SQL MI.](images/BM-Ex1-T5-Step22.png "Managed Instance")
 
27. Under **SQL managed instances**, select **sqlmi--cus** Managed Instance.

      ![Select sqlmi--cus SQL MI.](images/BM-Ex1-T5-Step22.1.png "Managed Instance")

28. When the SQL Server database migration to Azure SQL MI has completed, you will see the **WideWorldImporters** database shown with an **Online** status.

    ![The Azure SQL Managed Instance resource is shown in the Azure Portal with the WideWorldImporters migration showing a Status of Online.](images/BM-Ex1-T5-Step23.png "Azure SQL MI in Azure Portal showing the WideWorldImporters database in Online status")
