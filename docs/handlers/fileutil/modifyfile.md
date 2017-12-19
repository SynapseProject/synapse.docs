# Overview
The ModifyFileHandler allows you to modify the contents of a file.  The table below shows the destinations currently supported by the handler along with the expected format of files / directories in the plan.

|Destination|File Format|Directory Format
|-----------|-----------|----------------
|Windows Server|C:\Temp\MyFile.txt|C:\Temp\MyDirectory\
|Network Attached Storage (NAS)|\\\server\share$\MyDir\MyFile.txt|\\\server\share$\MyDir\
|Amazon S3 Buckets|s3://bucket/mydir/myfile001.txt|s3://bucket/mydir/

**Note**: Directories must ALWAYS end in a slash.  This is mainly because in Windows, there's no way to tell, from the naming convention, if an object is a directory or a file with no file extension.


## Plan Details
### Config

The config section of the plan specifies what technique should be used to modify the file contents, as well as directives on how the file(s) should be treated while being transformed.  It also contains configuration sections to connect to network/cloud based endpoints that require authentication and additional configuration.

#### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.FileUtil:ModifyFileHandler
    Config:
      Type: Yaml
      Values:
        Type : INI
        BackupSource: false
        CreateSettingIfNotFound: true
        RunSequential: true
        Aws:
          AccessKey: xxxxxxxx
          SecretKey: xxxxxxxx
          Region: eu-west-1
````

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Type|"[Regex](#regex)"<br>"[XmlTransform](#xmltransform)"<br>"[XPath](#xpath)"<br>"[KeyValue](#keyvalue)"<br>"[INI](#ini)"|Yes|Indicates how the file will be transformed. Details of each type of transformation can be found [here](#transformation-types)
|BackupSource|Boolean|No|Creates a backup of the original file.  The backup file will have the same name as the original file with a timestamp appened. (Default = false)
|CreateSettingIfNotFound|Boolean|No|Creates a new setting in the destination file if the source file does not contain the setting already. (Default = false)
|RunSequential|Boolean|No|Runs the files in the order they appear in the plan.  This is useful for chained dependencies between file modifications. (Default = false)
|Aws|[AwsConfig](#awsconfig)|No*|Details on how to connect to Amazon Web Services to access S3 Buckets.<br><br>* = Required if any endpoint is an S3 bucket.

#### AwsConfig 

This configuration section contains information on how to connect to Amazon Web Services to access the S3 endpoints.  This is a required section if any endpoint is an S3 bucket.  For optional elements, the values will be pulled from the environment variables of the server and/or the AWS credentails file on the server.

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|AccessKey|String|No|The Access Id Key used to make programatic requests to AWS.
|SecretKey|String|No|The Secret Access Key used to make programatic requests to AWS.
|Region|[String](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region)|Yes|A string value representing a valid Amazon RegionEndpoint.  <br><br>While strictly speaking, AWS S3 doesn't require a Region Endpoint, the client being used to access S3 needs to connect somewhere, so the endpoint is required.


### Parameters

The Parameters section is a list of files to be transformed.  If no destination is provided, the transformation will happen "in place" (overwrite the original source file).  The "SettingsFile" and "Settings" section explain how the file is to be transformed, and is specific to each type of transformation.  More details on what values are appropriate there can be found [here](#transformation-types). 

#### Sample
````yaml
  Parameters:
    Type: Yaml
    Values:
      Files:
      - Source: C:\Temp\FileMunge\input.ini
        Destination: s3://mybucket/Temp/FileMunge/output/output.ini
        SettingsFile: 
          Name: \\localhost\c$\Temp\FileMunge\initransform.csv
          HasEncryptedValues: true
        CreateSettingsIfNotFound: false
        Settings:
        - Key: TESTKEY
          Value: Guy Waguespack Was Here
        - Key: "[ODBC 32 bit Data Sources]:TESTKEY"
          Value: Guy Waguespack Was Here Too
````

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|Source|String|Yes|The source file to be transformed.
|Destination|String|No|The destination for the transformed file.  If no value provided, the transformation will overwrite the source file (in-place transformation).
|**SettingsFile**
|-Name|String|No|A file containing transformation information, either in CSV or XML format.  See individual [transformations](#transformation-types) for more details.
|-HasEncryptedValues|Boolean|No|Tells the handler that the file contains one or more encrypted values, so attempt to decrypt all values using the Crypto information provided.  (Default = false)
|-Crypto|CryptoProvider|No|Contains the information needed to decrypt any values in the file.  These values will override any Crypto details provided at the plan level and are only necessary here if the details used to encrypt the values are different from the plan level cryptography.  A more detailed description on plan cryptography can be found [here](/run/crypto/).
|CreateSettingIfNotFound|Boolean|No|Same as the config-element of the same name, but applies only to this particular file transformation.  This is used to "override" the plan-level setting.
|Settings|List of Key-Value Pairs|No|This is a list of settings that are to be applied to the transformation process.  These values are applied AFTER the SettingsFile has been applied, and are executed in the order they appear in the list.


## Transformation Types

### Regex

Performs a line-by-line Regular expression search, replacing any "Key" found with the "Value" specified.  Regex grouping can be used to save sections from the original match string and apply them into the replacement value.  Below are examples of how group matching works.

#### Simple Regular-Expression 

Performs normal Regex replacement since no match groups were specified.

````
Key:    ABCDEFGHI
Value:  xxx

Original: ABCDEFGHI
Result: xxx

````

#### Save All Groups As Is

Auto-saves all "groups" since no individual group match variables were specified in the value.

````
Key:    (ABC)DEF(GHI)
Value:  xxx

Original: ABCDEFGHI
Result: ABCxxxGHI
````

#### Save Select Groups

Saves only first group (ABC) and discards (GHI) since group match varables are used in the value.

````
Key:    (ABC)DEF(GHI)
Value:  ${1}xxx

Original: ABCDEFGHI
Result: ABCxxx
````

#### SettingsFile Format

Comma-seperated values (CSV) with the Regex match string as the first value and replacement value as the 2nd.

````text
(OLFCMD_URL\s*=\s*).*?$ , http://mynewurl.com
(ENDUR_JRE\s*=\s*).*?$ , ${1}jdk1.9.0.69
````

#### Settings

The "Key" values is the Regex match string, while the "Value" value is the value to replace it with.

````yaml
Settings:
- Key: (SET\s*ENDUR_VER\s*=\s*).*?$
  Value: ${1}NEW_VERSION_NUMBER_HERE
````

### XmlTransform

Uses an XmlTransformableDocument (Microsoft.Web.XmlTransform) to apply changes to an Xml Document.  Details on this process can be found at [CodePlex](https://xdt.codeplex.com).

#### SettingsFile Format

An Xml document representing the values to be transformed.  The format for this document can be found at [CodePlex](https://xdt.codeplex.com).

#### Settings

Individual settings do not apply to XmlTransformation.

### XPath

Uses X-path based parsing of an XML document to replace values, with the "Key" being an X-Path formatted string, and the "Value" being the value to replace.

#### SettingsFile Format

Comma-seperated values (CSV) where the first value is the X-Path location for the value to be replaces (Key) and the second value is the value to replace (Value).

````text
"//configuration/applicationSettings/MyService.Properties.Settings/setting[@name="initVector"]/value", "MyNewValue"
"//configuration/applicationSettings/MyService.Properties.Settings/setting[@name="KeyContainerName"]/value", "MyOtherNewValue"
````

#### Settings

The "Key" contains the X-Path location for the value to be replaces, the "Value" contains the value to replace it with.

````yaml
  Settings:
  - Key: '//configuration/applicationSettings/MyService.Properties.Settings/setting[@name="passPhrase"]/value'
    Value: SomePassPhrase
  - Key: '//configuration/applicationSettings/MyService.Properties.Settings/setting[@name="saltValue"]/value'
    Value: SomeSaltyValue
````

### KeyValue

Transforms a simple Key/Value based file where key and value are seperated by a colon or equal sign on the same line.

#### SettingsFile Format

Comma-seperated values (CSV) where the first value is the "Key" to find, and the second value is the "Value" to replace the existing value with.

````text
"baseGroup.ABCDEFG-svrMessageLogger.component", "New Value From File"
"MyKey", "MyNewValue"
````

#### Settings

The "Key" value is the key (value before the first ":" or "=" on the line) to find, while the "Value" is the value to replace the existing value with.

````yaml
Settings:
- Key: baseGroup.ABCDEFG-svrMessageLogger.isLogging
  Value: false
````

### INI

An INI file is treated as a Key/Value property file with the addition of optional "Sections".  A key "MyKey" in "Section1" is a different property than the key "MyKey" in "Section2", or "MyKey" in no section at all.  The replacement of values is the same as the KeyValue transformation, only the method of finding the Key/Value pair to replace can be slight different if the value is under a "section".

**Note**: If the Key contains an "=" or ":" in it, the values must be escaped with a slash.  Also, shame on you for designing such a bad property file.   :)

#### SettingsFile Format

Comma-seperated values (CSV) where

- If the line contains 2 values, the first is the "Key" and the second is the "Value".
- If the line contains 3 values, the first is the "Section", the second is the "Key" and the third is the "Value".

````text
"SECTION2", "NewKeyFromFile", "New Value From File"
"NewKeyFromFile", "New Value From File"
"SECTION2", "This is a key, not a section"
"key\=key", "guy\=guy"
````

#### Settings

Same as with Key/Value transformation, but if you need to specify a "Section", it should be pre-pended to the "Key" and seperated with a colon.  (See example below)

````yAML
Settings:
- Key: TESTKEY
  Value: Guy Waguespack Was Here
- Key: "[ODBC 32 bit Data Sources]:TESTKEY"
  Value: Guy Waguespack Was Here Too
````