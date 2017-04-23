# Overview
The SqlServerHandler executes SQL commands against a Microsoft SqlServer database, returning the results and/or paramter values
either in the handler's Exit Data, or saved off to a file.  Results can be returned in one of many formats including Xml, Json, Yaml and CSV.

## Plan Details
### Config

The config section of the plan specifies the information needed to connect to the target database, either through the provided fields, or by using a raw Connection String provided.

#### Samples
````yaml
    Config:
      Type: Yaml
      Values:
        DataSource: localhost
        Database: SANDBOX
        IntegratedSecurity: true
        ConnectionTimeout: 30
        OutputType: Yaml
        OutputFile: C:\Temp\output.yaml
````
````yaml
    Config:
      Type: Yaml
      Values:
        ConnectionString: data source=localhost;Integrated Security=SSPI;database=SANDBOX;connection timeout=30;
        OutputType: Xml
````


|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|DataSource|String|No*|Adds "data source=xxxx;" to the connection string.
|User|String|No*|Adds "user id=xxxx;" to the connection string.
|Password|String|No*|Adds "password=xxxx;" to the connection string.
|Database|String|No*|Adds "database=xxxx;" to the connection string.
|IntegratedSecurity|bool|No*|Adds "Integrated Security=SSPI;" to the connection string.
|TrustedConnection|bool|No*|Adds "Trusted_Connection=yes" to the connection string.
|ConnectionTimeout|int|No*|Adds "connection timeout=" to the connection string.
|ConnectionString|String|No*|The raw connection string used to connect to the database.  The handler uses the [.NET Framework Data Provider for SQL Server](https://www.connectionstrings.com/sql-server/) format to connect.
|OutputType|"None"<br>"Csv"<br>"Xml"<br>"Json"<br>"Yaml"|No|Specifies the format for the results and/or paramters returned from the call.  Default = "Csv"
|OutputFile|String|No|When provided, indicates the results returned from the call should be written to a file instead of being returned in ExitData.  This should be used when the size of the result set is too great to store in memory.

\* Note: Either "ConnectionString" or "Datasource, User, Password, etc.." must be provided, but not both.

Connection String information can be found online.  A good resource can be found at [http://connectionstrings.com](http://www.connectionstrings.com).

### Parameters

The Parameters section specifies the sql statement or stored procedure that is to be executed, and any parameters to that command or stored procedure.

#### Samples
````yaml
  Parameters:
    Type: Yaml
    Values:
      Text: SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE' AND TABLE_CATALOG = @DatabaseName
      Parameters:
      - Name: DatabaseName
        Value: SANDBOX
````

````yaml
  Parameters:
    Type: Yaml
    Values:
      StoredProcedure: dbo.MultiParams
      Parameters:
      - Name: Param1
        Value: 10
        Direction: Input
        Type: Int16
      - Name: Param2
        Direction: Output
        Type: Int16
      - Name: Param3
        Direction: Output
        Type: Int16
      - Name: Param4
        Direction: Output
        Type: DateTime
      - Name: Results
        Value: 21
        Direction: ReturnValue
        Type: Int16
````

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|Text|String|No*|The SQL text command to execute.
|StoredProcedure|String|No*|The name of the stored procedure to execute.
|TableDirect|String|No*|The name of a table, or list of tables to return all rows and columns from.   Click [here](https://msdn.microsoft.com/en-us/library/system.data.commandtype(v=vs.110).aspx) for more details.
|IsQuery|bool|No|If set to true, will exeute the method "ExecuteReader" and return one or more result sets.  Used primarily when data is returned.<br><br>If set to false, will execute the method "ExecuteNonQuery" which will return the number of rows affected.  Mostly used for INSERT, UPDATE and DELETE statements.<br><br>(Default = true)
|Parameters|List of ParameterType objects|No|A list of parameters to be substituted into the "Text" field, or passed into the "StoredProcedure" specified.  See table below for paramter fields and descriptions.

\* Note: One (and only one) of "Text", "StoredProcedure" or "TableDirect" must be provided.

##### ParameterType Object
|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|Name|String|No|The name of the variable.
|Value|String|No|The value for the variable.
|Size|int|No|The size of the variable.  Used primarily to allocate enough space for values returned in Output or InputOutput variables.
|Type|[System.Data.DbType](https://msdn.microsoft.com/en-us/library/system.data.dbtype(v=vs.110).aspx)|No|Maps to the [System.Data.DbType](https://msdn.microsoft.com/en-us/library/system.data.dbtype(v=vs.110).aspx) class.  Represents the variable type.
|Direction|"Input"<br>"Output"<br>"InputOutput<br>"ReturnValue"<br><br>|No|Maps to System.Data.ParameterDirection class.  Represents the direction of the variable.
