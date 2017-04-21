# Overview
The OdbcHandler executes SQL commands against an ODBC connected database, returning the results and/or paramter values
either in the handler's Exit Data, or saved off to a file.  Results can be returned in one of many formats including Xml, Json, Yaml and CSV.

## Plan Details
### Config

The config section of the plan specifies the information needed to connect to the target database.

#### Sample
````yaml
      Type: Yaml
      Values:
        ConnectionString: DSN=SQL_SANDBOX
        OutputType: Yaml
        OutputFile: C:\Temp\output.yaml
````


|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|ConnectionString|String|Yes|The raw connection string used to connect to the database.  The format of the string is purely dependant on the ODBC driver used.  Check your driver documentation or [http://connectionstrings.com](http://www.connectionstrings.com) for the exact format.
|OutputType|"None"<br>"Csv"<br>"Xml"<br>"Json"<br>"Yaml"|No|Specifies the format for the results and/or paramters returned from the call.  Default = "Csv"
|OutputFile|String|No|When provided, indicates the results returned from the call should be written to a file instead of being returned in ExitData.  This should be used when the size of the result set is too great to store in memory.

### Parameters

The Parameters section specifies any arguments passed to the command, and optionally manipulates the argument string using Regular Expression replacement. 

#### Samples
````yaml
  Parameters:
    Type: Yaml
    Values:
      Text: SELECT TABLE_SCHEMA, TABLE_NAME FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_TYPE = 'BASE TABLE' AND TABLE_CATALOG = ?
      Parameters:
      - Name: DatabaseName
        Value: SANDBOX
````

````yaml
  Parameters:
    Type: Yaml
    Values:
      StoredProcedure: "{ ? = call dbo.MultiParams(?,?,?,?) }"
      IsQuery: true
      Parameters:
      - Name: Results
        Direction: Output
        Type: Int16
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
````

Note : When CommandType is set to "Text", the .NET Framework Data Provider for ODBC does not support passing named parameters to an SQL statement or to a stored procedure called by an OdbcCommand. In either of these cases, use the question mark (?) placeholder.

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
