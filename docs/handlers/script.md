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
        Args : -ExecutionPolicy Bypass
        ScriptArgs : "-p1 xxx -p2 yyy -p3 zzz"
        ParameterType: Script
        TimeoutMills: 10000
        TimeoutAction: Error
        ValidExitCodes:
        - "EqualTo 0"
````

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|RunOn|String|No|Server where the command should be executed.  (Default = localhost)
|WorkingDirectory|String|No|Directory where the command should be run from.  (Default = C:\Temp)
|Type|"Powershell"|Yes|Tells the handler what type of script is being executed.  Click [here](#scripttype-values) for detailed description of the valid values.
|Args|String|No|Arguments passed into the script engine.
|ScriptArgs|String|No|Arguments passed into the script.
|ParameterType|"Script"<br>"File"|No|Tells the handler whether the Parameter section is the script itself, or the location of the script.  Click [here](#parametertype-values) for detailed description of the valid values (Default = "Script").
|TimeoutMills|long|No|Number of milliseconds to wait before timing out. (Default = Never Timeout)
|TimeoutStatus|"None"<br>"New"<br>"Initializing"<br>"Running"<br>"Waiting"<br>"Cancelling"<br>"Complete"<br>"Success"<br>"CompletedWithErrors"<br>"SuccessWithErrors"<br>"Failed"<br>"Cancelled"<br>"Tombstoned"<br>"Any"|No|Status to return when a timeout occurs.  Click [here](#statustype-values) for more details on Synapse StatusType values.  (Default value = "None".  Return status will be evaluated based on Exit Code instead.)
|KillRemoteProcessOnTimeout|boolean|No|Specifies whether the process running on the "RunOn" server should be terminiated when a timeout occurs.  This only applies when RunOn is specified.  Process will always terminate on timeout when RunOn is not specified.  (Default Value = false)
|ValidExitCodes|"EqualTo"<br>"NotEqualTo"<br>"LessThan"<br>"LessThanOrEqualTo"<br>"GreaterThan"<br>"GreaterThanOrEqualTo"<br>"Between"<br>"NotBetween"|No|Specifies what status to report back based on exit code returned from the action exexuted Click [here](#validexitcodes-values-and-syntax) for detailed scription of the syntax. (Default = "EqualTo 0").<br><br>**Syntax : [OPERATOR] VALUE1 [VALUE2] [STATUS]**

### Parameters

The Parameters section is read in as a string.  It is either the script itself, or the location of the script to execute 
(specified by the [ParameterType](#parametertype-values) element in the Config section.)

#### Sample (ParameterType : Script)
````yaml
  Parameters:
    Type: Unspecified
    Values: |
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
````

#### Sample (ParameterType : File)
````yaml
  Parameters:
    Type: Unspecified
    Values: C:\MyDir\MyScript.ps1
````

## Detailed Descriptions
Below are detailed descriptions of the enumerations and synatax used in the config and parameter sections above.

### ScriptType Values

Tells the ScriptHandler what type of script it is executing.

|Value|Description
|-----|-----------
|Powershell|Indicates the script to run is a Powershell script.

### ParameterType Values

Tells the ScriptHandler how the Parameters section of the plan should be treated.

|Value|Description
|-----|-----------
|Script|Tells the handler that the Paramter value is the script itself.
|File|Tells the handler that the Parameter value is a file that contians the script.

### TimeoutAction Values
Specifies what action to take when a timeout occurs.

|Value|Description
|-----|-----------
|Continue|Ignore the timeout and continue.  Leave remote process running.
|Error|Stop plan execution and return error status.  Leave remote process running.
|KillProcessAndContinue|Ignore the itmeout and continue.  Kill remotely running process.
|KillProcessAndError|Stop plan execution and return error status.  Kill remomtely running process.

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

````
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
````
