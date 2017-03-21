# Overview
The CommandHandler allows for any command to be executed on a server as if it were being called from the command line prompt.
This assumes any required software for the command is installed and execute rights exist for the user making the execute request.

(Example : If the command is "sqlplus.exe", the handler assumes that Oracle has been installed where the command will run and that
whatever credentails the handler is running under has rights to run "sqlplus.exe" on that destination.)

## Plan Details
### Config

The config section of the plan specifies the command to execute and where it should be executed.  It also indicates how long it
should attempt to run the command before giving up and what to do if that time is exceeded.

#### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.CommandLine:CommandHandler
    Config:
      Type: Yaml
      Values:
        RunOn: myserver.mydomain.com
        WorkingDirectory: C:\Temp
        Command: powershell.exe
        TimeoutMills: 10000
        TimeoutAction: Error
        ValidExitCodes:
        - "EqualTo 0 Complete"
````

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|RunOn|String|No|Server where the command should be executed.  (Default = localhost)
|WorkingDirectory|String|No|Directory where the command should be run from.  (Default = C:\Temp)
|Command|String|Yes|The command to execute (ex: powershell.exe)
|TimeoutMills|long|No|Number of milliseconds to wait before timing out. (Default = Never Timeout)
|TimeoutStatus|"None"<br>"New"<br>"Initializing"<br>"Running"<br>"Waiting"<br>"Cancelling"<br>"Complete"<br>"Success"<br>"CompletedWithErrors"<br>"SuccessWithErrors"<br>"Failed"<br>"Cancelled"<br>"Tombstoned"<br>"Any"|No|Status to return when a timeout occurs.  Click [here](#statustype-values) for more details on Synapse StatusType values.  (Default value = "None".  Return status will be evaluated based on Exit Code instead.)
|KillRemoteProcessOnTimeout|boolean|No|Specifies whether the process running on the "RunOn" server should be terminiated when a timeout occurs.  This only applies when RunOn is specified.  Process will always terminate on timeout when RunOn is not specified.  (Default Value = false)
|ValidExitCodes|"EqualTo"<br>"NotEqualTo"<br>"LessThan"<br>"LessThanOrEqualTo"<br>"GreaterThan"<br>"GreaterThanOrEqualTo"<br>"Between"<br>"NotBetween"|No|Specifies what status to report back based on exit code returned from the action exexuted Click [here](#validexitcodes-values-and-syntax) for detailed scription of the syntax. (Default = "EqualTo 0").<br><br>**Syntax : [OPERATOR] VALUE1 [VALUE2] [STATUS]**

### Parameters

The Parameters section specifies any arguments passed to the command, and optionally manipulates the argument string using Regular Expression replacement. 

#### Sample
````yaml
  Parameters:
    Type: Yaml
    Values:
      Arguments: -ExecutionPolicy Bypass -File C:\Temp\test-base64.ps1 -p1 aaa -p2 bbb -p3 ccc
      Expressions:
      - Find: (-p1\s*)aaa(\s*-p2)
        ReplaceWith: ${1}xxx${2}
      - Find: bbb
        ReplaceWith: yyy
      - Find: ccc
        ReplaceWith: Hello World
        Encoding: Base64
````

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|Arguments|String|No|The arguments passed into the command.
|Expressions|List of :<br>- Find:<br>..ReplaceWith:<br>..Encoding:|No|Performs a Regular Expression replacement of the Arguments element matching on element "Find" and replacing with value "ReplaceWith".  Optional "Encoding" of the value can occur.<br><br>Supported Encoding : <br>"Base64"<br><br>Click [here](#regex-arguments) for detailed description of each element.

## Detailed Descriptions
Below are detailed descriptions of the enumerations and synatax used in the config and parameter sections above.

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

### EncodingType Values

Tells the CommandHandler how the values should be encoded when replaced.

|Value|Description
|-----|-----------
|None|No Encoding Used.
|Base64|Encodes value as BASE64 string.

### Regex Arguments
A detailed description of the Regex variable replacement object used in the Parameters section of the CommandHandler.

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|Find|String|Yes|Regex string to match against the Argument string.
|ReplaceWith|String|Yes|String to replace matched values with
|Encoding|[Enum](#encodingtype-values)|No|Specifies how to encode the ReplaceWith string before replacement.  Click [here](#encodingtype-values) for valid values.  (Default = None)

