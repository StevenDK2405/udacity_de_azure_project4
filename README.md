# Data Integration Pipelines For NYC Payroll Data Analytics

## Context of the project

The City of New York would like to develop a Data Analytics platform on Azure Synapse Analytics to accomplish two primary objectives:

- Analyze how the City's financial resources are allocated and how much of the City's budget is being devoted to overtime.

- Make the data available to the interested public to show how the City’s budget is being spent on salary and overtime pay for all municipal employees.

The project needs Data Engineering skills to create high-quality data pipelines that are dynamic, can be automated, and monitored for efficient operation. The project team also includes the city’s quality assurance experts who will test the pipelines to find any errors and improve overall data quality.

The source data resides in **Azure Data Lake** and needs to be processed in a NYC data warehouse in **Azure Synapse Analytics**. The source datasets consist of CSV files with Employee master data and monthly payroll data entered by various City agencies.

<img src="/img/1.Create Resources/Design.png" title="db schema"  width="800">


## Project resources

For this project, we'll work in the Azure Portal, using several Azure resources including:

- **Azure Data Lake Gen2**
- **Azure SQL DB**
- **Azure Data Factory**
- **Azure Synapse Analytics**

We will connect our **Azure pipelines** to this very Github repo and submit the URL or contents of the repository.


# Steps to reproduce the project



## Step 1 : Prepare the Data Infrastructure


1. Create the data lake and upload data.

- Create an **Azure Data Lake Storage Gen2** (storage account) and associated storage container resource 
<img src="/img/1.Create Resources/Create_StorageAccount.png" title="nycpayrollcontainer container"  width="700">

- Create the three following directories in this storage container :

    - *dirpayrollfiles*
    - *dirhistoryfiles*
    - *dirstaging*
    <img src="/img/1.Create Resources/Create_Folders.png" title="nycpayrollcontainer container"  width="700">

- Upload these files from the */data* folder to the *dirpayrollfiles* folder :

    - *EmpMaster.csv*
    - *AgencyMaster.csv*
    - *TitleMaster.csv*
    - *nycpayroll_2021.csv*
<img src="/img/1.Create Resources/Upload_dirpayrollfiles.png" title="files uploaded to dirpayrollfiles"  width="700">

- Upload the file *nycpayroll_2020.csv* from the project data to the *dirhistoryfiles* folder.
<img src="/img/1.Create Resources/Upload_dirhistoryfiles.png" title="files uploaded to dirhistoryfiles"  width="700">


2. Create an **Azure Data Factory** Resource.
<img src="/img/1.Create Resources/Create_ADF_1.png" title="factory"  width="700">
<img src="/img/1.Create Resources/Create_ADF_2.png" title="factory"  width="700">


3. Create a **SQL Database** to store the current year of the payroll data.

- Add client IP address to the SQL DB firewall
<img src="/img/1.Create Resources/Create_SQLDB_1.png" title="factory"  width="700">
<img src="/img/1.Create Resources/Create_SQLDB_2.png" title="factory"  width="700">

- Create table:
<img src="/img/1.Create Resources/Create_Table.png" title="factory"  width="700">


4. Create A **Synapse Analytics workspace**.
<img src="/img/1.Create Resources/Create_Synapse_WS.png" title="factory"  width="700">


## Step 2: Create Linked Services
- Because of the Udacity account provided to me does not permit the creation of a dedicated SQL pool, so I am utilizing Synapse's SQL serverless with an external table. Within SQL serverless, data is stored in Data Lake Gen 2 storage as files (CSV files in this project). Consequently, when creating a linked service to Synapse, it is not directly established with Synapse or the SQL pool, but rather with Synapse's Data Lake Gen 2. The pipeline flow will proceed as follows:
    - Nycpayrolls-ADLS DataLake -> SQL server -> Synapse DataLake
    - Synapse DataLake -> Aggregation Steps  -> Synapse DataLake

<img src="/img/2.Linked Services/LinkedServices.png" title="factory"  width="700">




## Step 3: Create Datasets in Azure Data Factory
<img src="/img/3.Datasets/Datasets.png" title="factory"  width="700">


## Step 4: Data Load
<img src="/img/4.Data Load/DataLoad1.png" title="factory"  width="700">
<img src="/img/4.Data Load/DataLoad2.png" title="factory"  width="700">
<img src="/img/4.Data Load/DataLoad3.png" title="factory"  width="700">


## Step 5: Aggregation Data Flow

In this step, we'll extract the 2021 year data and historical data, merge, aggregate and store it in **Synapse Analytics**. The aggregation will be on Agency Name, Fiscal Year and TotalPaid.

1. Create a Summary table in Synapse with the SQL script from *summary_table.sql* and create a dataset named *table_synapse_nycpayroll_summary*

2. Create a new dataset for the **Azure Data Lake Gen2** folder that contains the historical files.


3. Create new data flow and name it Dataflow Aggregate Data

4. Create a new Union activity in the data flow and Union with history files

5. Add a Filter activity after Union

6. Derive a new *TotalPaid* column

- In Expression Builder, enter 

7. Add an Aggregate activity to the data flow next to the *TotalPaid* activity

- Under Group By, Select AgencyName and Fiscal Year

8. Add a Sink activity to the Data Flow
- Select the dataset to target (sink) the data into the Synapse Analytics Payroll Summary table.
In Settings, select Truncate Table

9. Create a new Pipeline and add the Aggregate data flow
- Create a new Global Parameter (This will be the Parameter at the global pipeline level that will be passed on to the data flow)
- In Parameters, select **Pipeline Expression**
- Choose the parameter created at the Pipeline level
<img src="/img/5.Aggregation Dataflow/DataFlow.png" title="factory"  width="700">
<img src="/img/5.Aggregation Dataflow/RunSuccessed.png" title="factory"  width="700">

## Step 6: Connect the Project to Github

In this step, we'll connect Azure Data Factory to Github

- Login to your Github account and create a new Repo in Github
- Connect Azure Data Factory to Github
- Select your Github repository in Azure Data Factory
- Publish all objects to the repository in Azure Data Factory