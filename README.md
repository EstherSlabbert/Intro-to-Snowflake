# Snowflake

[Snowflake](https://www.snowflake.com/en/uk/) is a cloud-based data warehousing platform that enables organizations to efficiently store, manage, and analyze large volumes of data. [Learning](https://learn.snowflake.com/en/).

It is comprised of 3 main components:

- **Database storage**: This layer handles the storage of structured and semi-structured data, allowing for efficient management and retrieval.
- **Query processing**: This component is responsible for executing SQL queries and providing results, optimized for performance and scalability.
- **Cloud services**: This layer offers various services, such as user management, securoty, and metadata management, facilitating seamless operation and integration within the cloud environment.

**How is data organized?**

Data in Snowflake is organized by databases and schemas. A database is a logical grouping of schemas. A schema is a logical grouping of tables, views, functions, and procedures, etc. A table is a physical object which is logically organized in a row-and-column format. A view is a virtual table that is based on a query that runs from data on one or more tables to retrieve specific data - it does not store this data in another place, but references the tables it queries from.

**Where does the query processing happen?**

Query processing happens in a Virtual Warehouse. A warehouse is a cluster of compute resources in Snowflake; it provides computing power (i.e. CPU, memory and temporary storage (used to perform operations)). Operations such as retrieving rows from tables and views using SQL, creating tables and views, updating rows and columns, loading data etc. use *credits*.

Queries available:

- [SQL data types documentation](https://docs.snowflake.com/en/sql-reference-data-types)
- [SQL command reference documentation](https://docs.snowflake.com/en/sql-reference-commands)
- [Function and stored procedure documentation](https://docs.snowflake.com/en/sql-reference-functions)

- Data Retrieval Querying:
  - SELECT: Retrieve data from one or more tables or views. (Use with ORDER BY, WHERE, [NOT] LIKE, LIMIT etc. keywords and/or CURRENT_DATABASE(), CURRENT_WAREHOUSE() etc. functions) e.g. `SELECT email FROM table_name WHERE email LIKE '%.uk';` would select emails ending in `.uk`.
  - LIST: Returns a list of file that have been staged.
- Data Definition Language (DDL) Queries:
  - CREATE: Create databases, schemas, tables, views, roles, stages, and more.
  - ALTER: Modify existing databases, schemas, tables, views, roles, stages, and more.
  - DROP: Remove databases, schemas, tables, views, roles, etc.
  - TRUNCATE: Remove all records from a table.
- Data Manipulation Language (DML) Queries:
  - INSERT: Add new records/rows to a table.
  - UPDATE: Modify existing records/rows in a table.
  - DELETE: Remove records/rows from a table.
  - MERGE: Perform INSERT, UPDATE, or DELETE based on a matched condition.
- Transaction control: Manages transactions
  - BEGIN TRANSACTION
  - COMMIT
  - ROLLBACK
- Data Control Language (DCL) Queries:
  - GRANT: Grants privileges to users/roles
  - REVOKE: Remove privileges from users/roles
- Account and Session Management:
  - USE: Set the active database, schema, warehouse, or role.
  - SHOW: Display information about Snowflake objects like databases, tables, schemas, etc.
  - DESC[RIBE]: Provide metadata information about database object
  - SET: Assign a value to a variable
- Stored Procedures and User-defined functions: Creation and execution of procedural code with SQL e.g.function `CREATE_DATABASE('TEST');` could output COMPANY_DOMAIN_TEST_DB database with preset schemas.
- Analytical Queries:
  - Windows functions
  - Aggreggations
  - Group Sets

**What cloud services are included?**

The cloud services layer is a collection of services that coordinate activities across Snowflake. It ties the components of Snowflake together.
Cloud services include authentication and access control, infrastructure and metadata management, query parsing and optimisation.

## Getting started

### Roles

Snowflake follows a mix of DAC (Discretionary Access Control) and RBAC (Role-Based Access Control), details of which can be found [here](https://docs.snowflake.com/en/user-guide/security-access-control-overview). Access control privileges determine who can access and perform operations on specific objects in Snowflake.

Roles (default):

- **GLOBALORGADMIN**: Role that performs organization-level tasks such as managing the lifecycle of accounts and viewing organization-level usage information. Only exists in organization account.
- **ORGADMIN**: Role that uses a regular account to manage operations at the organization level. (Note: to be phased out.)
- **ACCOUNTADMIN**: Responsible for managing the Snowflake account, including user management, security, resources, and billing. (Highest level of access - encapsulates the SYSADMIN and SECURITYADMIN system-defined roles.)
- **SECURITYADMIN**: Responsible for creating, monitoring, and managing users and roles. i.e. granting and revoking access of users or privileges of roles.
- **USERADMIN**: Designed for user management, this role can create, modify, and delete users and roles but does not have access to database objects.
- **SYSADMIN**: Responsible for database management, including creating and modifying databases, schemas, warehouses. i.e. administrative tasks concerning database objects.
- **PUBLIC**: Default role allowing access to public database objects. Every user within an account is impilcitly granted this role.

### Loading data into Snowflake

There are many ways to load data into Snowflake, but I will cover the ones that I have used myself.

**Option 1: Using Stages**:

A Snowflake stage is a location in cloud storage that you use to load and unload data from a table. Stages are temporary storage locations for data files (known as staging areas) used during the loading and unloading processes.

- Internal: Used to store data files internally within Snowflake. Each user and table in Snowflake gets an internal stage by default for staging data files. (`PUT` command uploads data file into a stage, this compresses files by default using `gzip`. `COPY INTO` command loads data into a table from a data file in a stage)
  - User stage: accessed using `@~`. Only available for a specific user. This stage is convenient if your files will only be accessed  by a single user, but needs to be copied into multiple tables, however, it does not support setting file format options.
  - Table stage: accessed using `@%Table_Name`. Only available for a specific table. You might use a table stage if you only need to copy files into a single table, but want to make the files accessible to multiple users. OWNERSHIP privilege on the table is required to use this stage.
  - Named stages: accessed using `@Stage_Name`. Named stages are database objects that provide the greatest degree of flexibility for data loading. Users with the appropriate privileges on the stage can load data into any table.
- External: Used to store data files externally in Amazon S3, Google Cloud Storage, or Microsoft Azure.

**Option 2: Using Programming languages**:

Snowflake provides drivers for different programming languages (e.g. Python, C#, etc.) that can be used to load data programmatically, provided you use the appropriate driver to connect and execute commands.

Using C# (simple & untidy code):

```csharp
using Snowflake.Data.Client;

// replace with actual details here OR set up an Options file and appsettings.json + environment variables to connect
string user = "NAME.SURNAME@COMPANY.COM";
string account = "company-uksouth";
string role = "GBL ROL DPRO DOMAIN DB";
string warehouse = "COMPANY_DOMAIN_INTEGRATION_WH";
string database = "COMPANY_DOMAIN_NAME_DB";
string schema = "CURATED";
string table = "TABLE_NAME";
string filePath = "C:/Users/username/path/data.csv";

string connection = $"account={account};warehouse={warehouse};database={database};schema={schema};role={role};user={user};authenticator=externalbrowser"; // add password={password};authenticator=snowflake if you don't want to use authenticator=externalbrowser

using (SnowflakeDbConnection conn = new SnowflakeDbConnection())
{
    conn.ConnectionString = connectionString;
    conn.Open();

    // UPLOAD file into stage
    using (var command = conn.CreateCommand())
    {
        command.CommandText = $"PUT file:///{filePath} @~";
        command.ExecuteNonQuery();
    }

    // LOAD data into table from staged file
    using (var command = conn.CreateCommand())
    {
        command.CommandText = $"COPY INTO {table} FROM @~ FILE_FORMAT=(TYPE=CSV FIELD_DELIMITER=',' SKIP_HEADER=1);";
        command.ExecuteNonQuery();
    }

    // DIRECT INSERT
    using (var command = conn.CreateCommand())
    {
        command.CommandText = $"INSERT INTO {table} (column1, column2) VALUES ('value1', 'value2');";
        command.ExecuteNonQuery();
    }
    conn.Close();
}
```

### Querying data from Snowflake

Again, there are many ways to query data from Snowflake.

**Option 1: Snowflake UI**:

- Log in to Snowflake via your organization's login.
- Navigate to the 'Projects' section in the panel on the LHS of your screen. (The Dashboard section here can also be used to create Snowsight Dashboards using data in Snowflake and scripts)
- Select the Worksheet subsection under Projects.
- Create a new Worksheet here. Ensure you select the appropriate role, database, schema and warehouse. (You can rename the worksheet by double-clicking on the dated sheet name)
- Write your SQL query in the worksheet and then run it clicking the Play button in the top RHS of your screen.

**Option 2: SnowSQL**:

- Download and install SnowSQL from the Snowflake website. (see SnowSQL section below)
- Connect using `snowsql -a %SNOWSQL_ACCOUNT% -u %SNOWSQL_USER% --authenticator externalbrowser` or other connection.
- Set the context by using `USE` commands to set the role, warehouse, database, schema if they are unset.
- Write your SQL query and run it using `;` at the end of the command and pressing Enter.

**Option 3: Programming languages**:

Using C#:

```csharp
using Snowflake.Data.Client;

// replace with actual details here OR set up an Options file and appsettings.json + environment variables to connect
string user = "NAME.SURNAME@COMPANY.COM";
string account = "company-uksouth";
string role = "GBL ROL DPRO DOMAIN DB";
string warehouse = "COMPANY_DOMAIN_INTEGRATION_WH";
string database = "COMPANY_DOMAIN_NAME_DB";
string schema = "CURATED";
string table = "TABLE_NAME";

string connection = $"account={account};warehouse={warehouse};database={database};schema={schema};role={role};user={user};authenticator=externalbrowser"; // add password={password};authenticator=snowflake if you don't want to use authenticator=externalbrowser

using (SnowflakeDbConnection conn = new SnowflakeDbConnection())
{
    conn.ConnectionString = connectionString;
    conn.Open();
    // READ
    using (var command = conn.CreateCommand())
    {
        command.CommandText = $"SELECT * FROM {database}.{schema}.{table};";
        using (var reader = command.ExecuteReader())
        {
            while (reader.Read())
            {
                object[] values = new object[reader.FieldCount];
                reader.GetValues(values);

                Console.WriteLine("Response: " + string.Join(", ", values));
            }
        }
    }
    conn.Close();
}
```

## SnowSQL (and stages)

[SnowSQL](https://docs.snowflake.com/user-guide/snowsql) is the command line client for connecting to Snowflake to execute SQL queries and perform DDL (Data Definition Language - define data structures i.e. create, alter, drop, truncate) and DML (Data Manipulation Language - insert, update, delete) operations.

Windows Installation:

- [SnowSQL installer](https://www.snowflake.com/en/developers/downloads/snowsql/) select SnowSQL for Windows. This should download the msi file.
- Run the downloaded installer and follow the prompts to install SnowSQL. (Default location for binaries `%USERPROFILE%\.snowsql`, consequently the config file will be `%USERPROFILE%\.snowsql\config`.)
- Check installation by running `snowsql -v` in the terminal (Powershell will suffice) to check the version installed. It should return the version if the installation was successful.

Windows Configuration:

After running the snowsql command a [configuration file](https://docs.snowflake.com/en/user-guide/snowsql-config) will be generated. Modifying this is optional, but makes it simpler to connect.

- Open config file (located `%USERPROFILE%\.snowsql\config`)
- You must change the security of the file if you are storing a plain text password in this file. (possible option: `icacls "%USERPROFILE%\.snowsql\config" /grant "%USERNAME%":F /inheritance:r` this removes inheritance, ensuring only the current user has permissions.) Otherwise, use environment variables (set them in Control Panel) and reference them in the config file as follows:

    ```config
    [connections]
    account = %SNOWSQL_ACCOUNT%
    username = %SNOWSQL_USER%
    password = %SNOWSQL_PASSWORD%
    ```

- Set other desired options and save config file.

Windows connection:

- Connection via Powershell: run `snowsql -a %SNOWSQL_ACCOUNT% -u %SNOWSQL_USER% --authenticator externalbrowser` no need for password, or just `snowsql` if you've specified a password and configuration.
- Set up context and run commands:
  - Run the following commands (switch NAMEs out with actual names and role name with actual role name):
  
  ```snowsql
  -- Set Context
  USE WARHOUSE NAME_WH;
  USE DATABASE NAME_DB;
  USE SCHEMA NAME_SCHEMA;
  USE ROLE 'GBL ROL DPRO DEV DOMAIN SYSADMIN';
  ```

  - You can now run SQL queries in the Powershell terminal. e.g.

    ```snowsql
    -- A stage refers to a location of data files within cloud storage (encrypts and compresses data by default), which is used to load data into tables
    -- List all stages associated with current schema
    SHOW STAGES; -- if 0 Rows: no stages are available/have been created
    list @~; -- shows the user stage, which is an implicitly created stage created for each user (not accessible to other users)

    -- Create Database
    CREATE DATABASE NEW_DB;

    -- Create Schema
    USE DATABASE NEW_DB;
    CREATE SCHEMA NEW_SCHEMA;

    -- Create Warehouse (if you have the permissions)
    CREATE OR REPLACE WAREHOUSE NEW_WAREHOUSE WITH 
        WAREHOUSE_SIZE='X-SMALL'
        AUTO_SUSPEND = 180
        AUTO_RESUME = TRUE
        INITIALLY_SUSPENDED=TRUE;


    -- Checks current session's database, schema and warehouse
    SELECT CURRENT_DATABASE(), CURRENT_SCHEMA(), CURRENT_WAREHOUSE();

    -- Create Table
    USE SCHEMA NEW_SCHEMA;
    CREATE OR REPLACE TABLE NEW_TABLE (ID INT, NAME VARCHAR(255));

    -- Lists available tables
    SHOW TABLES;

    -- Stage file greater than 50mb to user stage
    PUT file:///Users/username/Documents/data.csv @~/NEW_TABLE/staged; -- PUT and GET can only be run from SnowSQL
    -- NOTE: There is a direct correlation between the fields in the files and the columns in the table. Records will not be loaded if the numbers, positions, and data types don't align with the data.

    list @~;
    list @~/NEW_TABLE;
    SELECT * FROM NEW_TABLE; -- should still be empty
    !system.clear; -- cleans terminal

    -- Load data into table from user stage
    COPY INTO NEW_TABLE FROM @~/NEW_TABLE/staged/data.csv FILE_FORMAT=(TYPE= 'CSV' RECORD_DELIMITER = '\n' FIELD_DELIMITER = ',' SKIP_HEADER = 1); -- recommended to use a file format object you have created e.g. FORMAT_NAME = 'CSV_FORMAT_CLEAN_DATA' -- csv default delimiters: records = newline character, fields = commas

    SELECT * FROM NEW_TABLE LIMIT 10; -- retrieves 10 records from table

    -- Remove staged file
    REMOVE @~/staged/data.csv; -- Note: Once removed cannot be recovered

    -- Clear out the data in a table
    TRUNCATE TABLE NEW_TABLE;

    -- Create File Format
    CREATE OR REPLACE FILE FORMAT CSV_FORMAT_CLEAN_DATA TYPE = 'CSV' RECORD_DELIMITER = '\n' FIELD_DELIMITER = ',' FIELD_OPTIONALLY_ENCLOSED_BY= '"' SKIP_HEADER = 1 COMPRESSION = 'AUTO';

    -- Uploads csv file to table stage
    list @%NEW_TABLE; -- lists contents of table stage (which can be used by any user with access to the table)
    PUT file:///Users/username/Documents/data.csv @%NEW_TABLE/staged; --stages a file to a table stage (gzips files on upload)
    -- Note: Table stage only allows the data staged there to be loaded into one table
    
    -- Load data into table from table stage
    COPY INTO NEW_TABLE FROM @%NEW_TABLE/staged/data.csv FILE_FORMAT=(FORMAT_NAME = 'CSV_FORMAT_CLEAN_DATA') VALIDATION_MODE='RETURN_ALL_ERRORS'; -- returns information on all errors upontrying to load data into a table, even those for the partially loaded files (can also use 'RETURN_ERRORS')

    -- Check data loaded into table
    SELECT * FROM NEW_TABLE LIMIT 10;
    -- Clear out the data in a table
    TRUNCATE TABLE NEW_TABLE;
    -- Remove staged file
    REMOVE @%NEW_TABLE/staged/data.csv;

    -- Create Named stage (used by anyone regardless of user or table)
    CREATE OR REPLACE STAGE NEW_NAMED_STAGE FILE_FORMAT=(FORMAT_NAME = 'CSV_FORMAT_CLEAN_DATA');

    -- Shows all available stages in schema
    SELECT * FROM information_schema.stages;

    -- Uploads csv file to named stage
    PUT file:///Users/username/Documents/data.csv @NEW_NAMED_STAGE;

    LISt @NEW_NAMED_STAGE;

    -- Loads data into table from named stage
    COPY INTO NEW_TABLE FROM @NEW_NAMED_STAGE/data.csv FILE_FORMAT=(FORMAT_NAME = 'CSV_FORMAT_CLEAN_DATA') ON_ERROR='CONTINUE' PURGE = 'TRUE'; -- purge removes data from the stage once it has been successfully loaded into the table

    -- Check data loaded into table
    SELECT * FROM NEW_TABLE LIMIT 10;

    -- Delete stage
    DROP STAGE IF EXISTS NEW_NAMED_STAGE;

    -- Delete Database
    DROP DATABASE IF EXISTS NEW_DB;

    -- quits current session
    !q -- !exit; or !disconnect also works
    ```

## Alternatives

Selecting a solution should be based on the specific business needs, existing infrastructure, and long-term data strategy goals.

- Amazon Redshift: A fully managed data warehouse service on AWS, designed for large-scale data processing and analytics.
- Google BigQuery: A serverless, highly scalable, and cost-effective multi-cloud data warehouse offered by GCP.
- Microsoft Azure Synapse Analytics: A cloud-based analytics service that integrates big data and data warehousing.
- Cloudera Data Platform (CDP): A hybrid data platform designed for enterprise data management, which combines Hadoop's distributed processing paradigm with modern cloud-native architectures.
- Databricks: Unified analytics platform known for its collaborative environment based on Apache Spark.
- Teradata Vantage: An analytics platform that provides enterprise-scale analytics capabilities and hybrid cloud, multi-cloud, and on-premises deployment options.

Snowflakes Distinctions:

In essence, while the core capabilities related to data warehousing and analytics may seem similar, the operational efficiencies, ease of integration, and specific functionalities offered by each platform can differ significantly, making some platforms more suitable for certain needs over others.

- Architecture: Snowflake's architecture features a unique separation of storage and compute resources, allowing for independent scaling. Other platforms may have different scaling models, such as node-based or serverless.
- Ease of Use and Management: Snowflake is often praised for its low maintenance needs and ease of use, requiring minimal manual intervention. This contrasts with some platforms that may require more administrative effort.
- Data Sharing: Snowflake excels in secure data sharing without moving or copying data, a feature that is more advanced compared to many alternatives.
- Integration Ecosystem: While all platforms integrate with third-party tools, some like Synapse are tightly integrated into their cloud ecosystem, whereas Snowflake provides broader cross-cloud compatability.
- Real-time and ML Capabilities: Platforms like Databricks offer specialised real-time processing and ML capabilities via Apache Spark, which may suit organisations with significant data science requirements differently than Snowflake's offerings.
- Pricing Models: Pricing models can vary significantly, from Snowflake's compute plus storage separation to usage-based pricing in platforms like Google BigQuery, affecting how organisations budget and manage costs.

Why use Snowflake?

- Independent scalability: Snowflake's architecture allows for seamless scaling of compute and storage resources independently. This makes it easier to handle varying workloads and large volumes of data efficiently. Snowflake's unique architecture enables the independent scaling of storage and compute resources, providing flexibility and cost-effectiveness. (Performance i.e. unique architecture enhances performance and allows concurrent processing of multiple workloads without performance degradation)
- Ease of use: Snowflake is designed to be user-friendly (simple and accessible), requiring minimal management and no infrastructure maintenance, which is ideal for organisations without extensive IT resources.
- Cross-Cloud Flexibility: Snowflake is available on major cloud platforms like AWS, Azure, GCP. This gives users flexibility to operate Snowflake on their preferred cloud provider or across multiple clouds (i.e. choice and flexibility).
- Seamless Data Sharing: Snowflake enables secure data sharing within and outside an organisation without the need to move or copy data, fostering collaboration. (live data sharing) (robust security features include end-to-end encryption, comliance with data regulations like GDPR and HIPAA + features like role-based access control)
- Broad integration: Snowflake integrates well with various data integration, BI, and analytics tools, making it easier to build a comprehensive data analytics stack.
- Low administrative overhead: With automatic tuning and resource allocation, Snowflake reduces the need for manual interventions and continuous optimisations.
- Cost efficiency: With pay-as-you-go pricing, organisations can manage their expenses effectively based on their actual usage, avoiding unnecessary costs associated with provisioning and maintaining infrastructure.
