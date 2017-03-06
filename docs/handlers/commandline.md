# Overview
The CommandLine Handler allows Synapse to execute external operating system commands or run supported scripts on servers.

# CommandHandler
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
|TimeoutAction|"Continue"<br>"Error"<br>"KillProcessAndContinue"<br>"KillProcessAndError"|No|What action to take when a timeout occurs.  Click [here](#timeoutaction-values) for detailed description of valid values.
|ValidExitCodes|"EqualTo"<br>"NotEqualTo"<br>"LessThan"<br>"LessThanOrEqualTo"<br>"GreaterThan"<br>"GreaterThanOrEqualTo"<br>"Between"<br>"NotBetween"|No|Specifies what status to report back based on exit code returned from the action exexuted Click [here](#validexitcodes-values-and-syntax) for detailed scription of the syntax. (Default = "EqualTo 0").<br><br>**Syntax : [OPERATOR] VALUE1 [VALUE2] [STATUS]**

### Parameters

The Parameters section specifies any arguments passed to the command, and optionally manipulates the argument string based on defined
[Parser](#argument-parser-values) types.

#### Sample (Parser : None)
````yaml
  Parameters:
    Type: Yaml
    Values:
      Parser: None
      Arguments: -ExecutionPolicy Bypass -File C:\Temp\test-base64.ps1 -p1 aaa -p2 bbb -p3 ccc
````

#### Sample (Parser : Regex)
````yaml
  Parameters:
    Type: Yaml
    Values:
      Parser: Regex
      Arguments: 
        ArgString: -ExecutionPolicy Bypass -File C:\Temp\test-base64.ps1 -p1 aaa -p2 bbb -p3 ccc
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
|Parser|"None"<br>"Base64"|No|Tells handler how the Arguments should be parsed.  Click [here](#argument-parser-values) for detailed description of valid values.  (Default = None)
|Arguments|String<br><br>or<br><br>ArgString<br>..Expressions<br>....Find<br>....ReplaceWith<br>....Encoding|No|The Arguments section varies depending on the Parser specified.  <br><br>For Parser "None", it simply takes it as a String and appends it to the command.  <br><br>For Parser "Regex", it performs a sequential regular expression replacement on the "ArgString", replacing any matches in the "Find" element with the value in the "ReplaceWith" element, optionally encoding based on the value specified in the "Encoding" element.  Click [here](#regex-arguments) for detailed view of the Regex Argument elements.



# ScriptHandler
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
|TimeoutAction|"Continue"<br>"Error"<br>"KillProcessAndContinue"<br>"KillProcessAndError"|No|What action to take when a timeout occurs.  Click [here](#timeoutaction-values) for detailed description of valid values.
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

### Argument Parser Values

Tells the CommandHandler how to process the Arguments section of the Parameters.

|Value|Description
|-----|-----------
|None|Arguments value is taken as specified.
|Regex|Evaluates ArgString using a regular expression specified in the "Find" element.  When a match occurs, it replaces the value(s) with the regex value in the "ReplaceWith" element. 

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
|ArgString|String|Yes|A templated argument string used as the target for the regular expression matching.
|Expressions|List of RegexSubstitutionType Elements (see next row)|An array of elements to match and replace against the ArgString above.
|**RegexSubstitutuionType Element Definition :**|||
|Find|String|Yes|Regex string to match against the ArgString
|ReplaceWith|String|Yes|String to replace matched values with
|Encoding|[Enum](#encodingtype-values)|No|Specifies how to encode the ReplaceWith string before replacement.  Click [here](#encodingtype-values) for valid values.  (Default = None)

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