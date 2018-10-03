# Overview
The ScriptHandler allows a script of the [types](#scripttype-values) supported to be executed either locally, or on a remotely specified server.

## Plan Details
### Config

The config section of the plan specifies the script to execute, any arguments needed for the script engine or script itself,
and where it should be executed.  It also indicates how long it should attempt to run the command before giving up and
what to do if that time is exceeded.

#### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.CommandLine:ScriptHandler
    Config:
      Type: Yaml
      Values:
        RunOn: someserver.somedomain.com
        WorkingDirectory: C:\Temp
        Type: Powershell
        Arguments : -ExecutionPolicy Bypass
        ParameterType: Script
        TimeoutMills: 10000
        TimeoutStatus: Failed
        KillRemoteProcessOnTimeout: true
        ValidExitCodes:
        - "EqualTo 0"
        ReturnStdout: true
````

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|RunOn|String|No|Server where the command should be executed.  (Default = localhost)
|WorkingDirectory|String|No|Directory where the command should be run from.  (Default = C:\Temp)
|Type|"Powershell"<br>"Batch"|Yes|Tells the handler what type of script is being executed.  Click [here](#scripttype-values) for detailed description of the valid values.
|Arguments|String|No|Arguments passed into the script engine.
|TimeoutMills|long|No|Number of milliseconds to wait before timing out. (Default = Never Timeout)
|TimeoutStatus|"None"<br>"New"<br>"Initializing"<br>"Running"<br>"Waiting"<br>"Cancelling"<br>"Complete"<br>"Success"<br>"CompletedWithErrors"<br>"SuccessWithErrors"<br>"Failed"<br>"Cancelled"<br>"Tombstoned"<br>"Any"|No|Status to return when a timeout occurs.  Click [here](#statustype-values) for more details on Synapse StatusType values.  (Default value = "None".  Return status will be evaluated based on Exit Code instead.)
|KillRemoteProcessOnTimeout|boolean|No|Specifies whether the process running on the "RunOn" server should be terminiated when a timeout occurs.  This only applies when RunOn is specified.  Process will always terminate on timeout when RunOn is not specified.  (Default Value = false)
|ValidExitCodes|"EqualTo"<br>"NotEqualTo"<br>"LessThan"<br>"LessThanOrEqualTo"<br>"GreaterThan"<br>"GreaterThanOrEqualTo"<br>"Between"<br>"NotBetween"|No|Specifies what status to report back based on exit code returned from the action exexuted Click [here](#validexitcodes-values-and-syntax) for detailed scription of the syntax. (Default = "EqualTo 0").<br><br>**Syntax : [OPERATOR] VALUE1 [VALUE2] [STATUS]**
|ReturnStdout|boolean|No|Specifies whether or not StdOut / Stderr should be returned in the ExitData field and made available in the ParentExitData element for future actions. (Default Value = true)<br><br>Used mostly when excessive script output being stored in memory becomes a memory usage issue for the handler.
|SupportsDryRun|boolean|No|Indicates that the script or command specified has implemented the DryRun functionality, and should be called when the DryRun flag is specified.  (Default Value = false)<br><br>The plan is responsible for passing the ~~IsDryRun~~ flag into the script or command where expected.

### Parameters

The Parameters section specifies any arguments passed to the script, and optionally manipulates the argument string using Regular Expression replacement. 

#### Sample (Script is embedded directly in the plan)
````yaml
  Parameters:
    Type: Yaml
    Values:
      ScriptBlock: |
        param(
          [string]$p1 = 'aaa',
          [string]$p2 = 'bbb',
          [string]$p3 = 'ccc'
        )
        write-host "P1 = [$p1]"
        write-host "P2 = [$p2]"
        write-host "P3 = [$p3]"
        hostname
        whoami
        write-host "Hello World"
        exit -1
      Arguments: "-p1 xxx -p2 yyy -p3 zzz
      Expressions:
      - Find: xxx
        ReplaceWith: ggg
      - Find: yyy
        ReplaceWith: hhh
      - Find: zzz
        ReplaceWith iii
        Encoding: Base64
````

#### Sample (Script is in a seperate file)
````yaml
  Parameters:
    Type: Yaml
    Values:
      Script: C:\MyDir\MyScript.ps1
      Arguments: "-p1 xxx -p2 yyy -p3 zzz"
````

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|Script|String|Yes*|Specifies a file where the script is saved.  Handler will run script from that location.
|ScriptBlock|String|Yes*|Specifies a block of code to run as a script.  String is saved into a temp file and executed.
|Arguments|String|No|The arguments passed into the script.
|Expressions|List of :<br>- Find:<br>..ReplaceWith:<br>..Encoding:|No|Performs a Regular Expression replacement of the Arguments element matching on element "Find" and replacing with value "ReplaceWith".  Optional "Encoding" of the value can occur.<br><br>Supported Encoding : <br>"Base64"<br><br>Click [here](#regex-arguments) for detailed description of each element.

\* **Either Script or ScriptBlock must exist, but not both.**

## Parameter Substitution
The script handler allows for certain Synapse data to be passed into the script arguments via parameter substitution.  These "variables" are specified by enclosing the parameter name between double-tilde delimeters like "~~ PARAMETER ~~".  These parameter names are case insenstive and any white space between the tildes is ignored.

````yaml
Parameters:
    Type: Yaml
    Values:
      Script: C:\Temp\SomeScript.ps1
      Arguments : -p1 "~~IsDryRun~~" -p2 "bbb" -p3 "ccc"
      Expressions:
      - Find: bbb
        ReplaceWith: Action Name = ~~actionName~~
      - Find: ccc
        ReplaceWith: ~~ rUnTiMeTiMeTyPe ~~
````

### Available Handler Variables
|Variable|Native Type|Passed As|Description
|--------|-----------|---------|----------
|InstanceId|Long|String|Unique Identifier for this run of Synapse.
|PlanInstanceId|Long|String|Unique Identifier for this run of Synapse. (Same as InstanceId)
|ActionInstanceId|Long|String|Unique Identifier for this action in this run of Synapse.
|IsDryRun|Boolean|"True" or "False"|Flag indicating whether this is a true "run" of the hander, or a test.
|ParentExitData|Object|Base64 Encoded Result of "ToString()"|Standard Output and Standard Error logs from the script.
|RequestNumber|String|String|Id tying the run back to a change request.
|RequestUser|String|String|User making the request.
|ActionName|String|String|Action name from the plan document.
|RuntimeType|String|String|Description of which handler dll was being used by Synapse.

## Detailed Descriptions
Below are detailed descriptions of the enumerations and synatax used in the config and parameter sections above.

### ScriptType Values

Tells the ScriptHandler what type of script it is executing.

|Value|Description
|-----|-----------
|Powershell|Indicates the script to run is a Powershell script.  Runs using powershell.exe
|Batch|Indicates the script to run is a Windows Batch script.  Runs using cmd.exe

### ValidExitCodes Values and Syntax
A space-delimited string that specifies exit codes and what status they represent.

````
Syntax : "[OPERATOR] VALUE1 [VALUE2] [STATUS]"
````
|Value|Required|Description
|-----|--------|-----------
|OPERATOR|No|Indicates how the exit code is to be evaluated.  See table below for details. (Default = "EqualTo")
|VALUE1|Yes|The value to compare to the exit code.
|VALUE2|Yes*|The 2nd value to compare the exit code to.  Only used with Between and NotBetween Operators.
|STATUS|No|The [StatusType](#staustype-values) to return when the expression is true.  (Default = "Complete")

Possible Operators :

|Operator|Example|Description
|--------|-------|-----------
EQ, EQU, or EqualTo|EQ 0 xxxxx|Returns STATUS when exit code is equal to VALUE1
NE, NEQ, or NotEqualTo|NEQ 0 xxxxx|Return STATUS when exit code is not equal to VALUE1
LT, LSS or LessThan|LessThan 0 xxxxx|Returns STATUS when exit code is less than VALUE1
LE, LEQ, or LessThanOrEqualTo|LE -1 xxxxx|Returns STATUS when exit code is less than or equal to VALUE1
GT, GTR or GreaterThan|GTR 0 xxxxx|Returns STATUS when exit code is greater than VALUE1
GE, GEQ, or GreaterThanOrEqualTo|GreaterThanOrEqualTo 1 xxxxx|Returns STATUS when exit code is greater than or equal to VALUE1
BT, BTW, or Between|BTW 10 20 xxxxx|Returns STATUS when exit code is between VALUE1 and VALUE2 (inclusive)
NB, NBT, or NotBetween|NB -20 -10 xxxxx|Returns STATUS when exit codes is not between VALUE1 and VALUE2 (exclusive)

#### StatusType Values
StatusType values are part of the Synapse Core project.   The values below are current as of the last update to this documentation,
but for the most recent values, please check the [SynapseProject GitHub page](https://github.com/SynapseProject/synapse.core.net).
(synapse.net > Synapse.Core > Classes > Enums > StatusType.cs)

```java
namespace Synapse.Core
{
    public enum StatusType
    {
        None = 0,
        New = 1,
        Initializing = 2,
        Running = 4,
        Waiting = 8,
        Cancelling = 16,
        Zombie = 32,
        Complete = 128,
        Success = 128,
        CompletedWithErrors = 256,
        SuccessWithErrors = 256,
        Failed = 512,
        Cancelled = 1024,
        Tombstoned = 2048,
        Any = 16383
    }
}
```

### Regex Arguments
A detailed description of the Regex variable replacement object used in the Parameters section of the CommandHandler.

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|Find|String|Yes|Regex string to match against the Argument string.
|ReplaceWith|String|Yes|String to replace matched values with
|Encoding|[Enum](#encodingtype-values)|No|Specifies how to encode the ReplaceWith string before replacement.  Click [here](#encodingtype-values) for valid values.  (Default = None)
