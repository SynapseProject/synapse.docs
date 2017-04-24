# Overview
The OracleHandler executes SQL commands against an Oracle database, returning the results and/or paramter values
either in the handler's Exit Data, or saved off to a file.  Results can be returned in one of many formats including Xml, Json, Yaml and CSV.

## Plan Details
### Config

The config section of the plan specifies the information needed to connect to the target database, either through the provided fields, or by using a raw Connection String provided.

#### Samples
````yaml
    Config:
      Type: Yaml
      Values:
        User: scott
        Password: tiger
        DataSource: (DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))(CONNECT_DATA =(SERVER = DEDICATED)(SERVICE_NAME = XE)))
        OutputType: Xml
        OutputFile: C:\Temp\output.xml
        PrettyPrint: true
````
````yaml
    Config:
      Type: Yaml
      Values:
        ConnectionString: user id=scott;password=tiger;data source=(DESCRIPTION = (ADDRESS = (PROTOCOL = TCP)(HOST = localhost)(PORT = 1521))(CONNECT_DATA =(SERVER = DEDICATED)(SERVICE_NAME = XE)));
        OutputType: Yaml
        PrettyPrint: false
````


|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|DataSource|String|No*|Adds "data source=xxxx;" to the connection string.
|User|String|No*|Adds "user id=xxxx;" to the connection string.
|Password|String|No*|Adds "password=xxxx;" to the connection string.
|ConnectionString|String|No*|The raw connection string used to connect to the database.  The handler uses the [.NET Framework Data Provider for Oracle](https://www.connectionstrings.com/oracle/) format to connect.
|OutputType|"None"<br>"Csv"<br>"Xml"<br>"Json"<br>"Yaml"|No|Specifies the format for the results and/or paramters returned from the call.  Default = "Csv"
|OutputFile|String|No|When provided, indicates the results returned from the call should be written to a file instead of being returned in ExitData.  This should be used when the size of the result set is too great to store in memory.
|PrettyPrint|bool|No|Formats Json and Xml output with line breaks and indention.  (Default = false)

\* Note: Either "ConnectionString" or "Datasource, User, Password, etc.." must be provided, but not both.

Connection String information can be found online.  A good resource can be found at [http://connectionstrings.com](http://www.connectionstrings.com).

### Parameters

The Parameters section specifies the sql statement or stored procedure that is to be executed, and any parameters to that command or stored procedure.

#### Samples
````yaml
  Parameters:
    Type: Yaml
    Values:
      Text: SELECT * from PRESIDENTS WHERE AGE > :AGE
      IsQuery: true
      Parameters:
      - Name: AGE
        Value: 70
````

````yaml
  Parameters:
    Type: Yaml
    Values:
      StoredProcedure: GETPRESIDENTS
      Parameters:
      - Name: P_RESULTS
        Direction: Output
        Type: RefCursor
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
|Type|[System.Data.DbType](https://msdn.microsoft.com/en-us/library/system.data.dbtype(v=vs.110).aspx) or <br>[Oracle.ManagedDataAccess.Client.OracleDbType](https://docs.oracle.com/cd/B19306_01/win.102/b14307/OracleDbTypeEnumerationType.htm)|No|Maps to the [System.Data.DbType](https://msdn.microsoft.com/en-us/library/system.data.dbtype(v=vs.110).aspx) or [Oracle.ManagedDataAccess.Client.OracleDbType](https://docs.oracle.com/cd/B19306_01/win.102/b14307/OracleDbTypeEnumerationType.htm) class.  Represents the variable type.
|Direction|"Input"<br>"Output"<br>"InputOutput<br>"ReturnValue"<br><br>|No|Maps to System.Data.ParameterDirection class.  Represents the direction of the variable.
