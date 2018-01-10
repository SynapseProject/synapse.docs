# Overview
The DeleteFileHandler deletes files and directories from a destination.  The table below shows the destinations currently supported by the handler along with the expected format of files / directories in the plan.

|Destination|File Format|Directory Format
|-----------|-----------|----------------
|Windows Server|C:\Temp\MyFile.txt|C:\Temp\MyDirectory\
|Network Attached Storage (NAS)|\\\server\share$\MyDir\MyFile.txt|\\\server\share$\MyDir\
|Amazon S3 Buckets|s3://bucket/mydir/myfile001.txt|s3://bucket/mydir/

**Note**: Directories must ALWAYS end in a slash.  This is mainly because in Windows, there's no way to tell, from the naming convention, if an object is a directory or a file with no file extension.


## Plan Details
### Config

The config section of the plan specifies how the delete action should be performed.  It also contains configuration sections to connect to network/cloud based endpoints that require authentication and additional configuration.

#### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.FileUtil:DeleteFileHandler
    Config:
      Type: Yaml
      Values:
        Recursive : true
        StopOnError: true
        Verbose: true
        Aws:
          AccessKey: xxxxxxxx
          SecretKey: xxxxxxxx
          Region: eu-west-1
````

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Recurse|Boolean|No|Should sub-directories be included in the action.  If value is "false", directory MUST be empty, or an error will occur. (Default = true)
|StopOnError|Boolean|No|Should Action Continue When An Error Is Encountered. (Default = true)
|Verbose|Boolean|No|Log details as each file / directory is deleted. It does NOT provide a list of directory contents on deletion. (Default = true)
|Aws|[AwsConfig](#awsconfig)|No*|Details on how to connect to Amazon Web Services to access S3 Buckets.<br><br>* = Required if any endpoint is an S3 bucket.

#### AwsConfig 

This configuration section contains information on how to connect to Amazon Web Services to access the S3 endpoints.  This is a required section if any endpoint is an S3 bucket.  For optional elements, the values will be pulled from the environment variables of the server and/or the AWS credentails file on the server.

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|AccessKey|String|No|The Access Id Key used to make programatic requests to AWS.
|SecretKey|String|No|The Secret Access Key used to make programatic requests to AWS.
|Region|[String](http://docs.aws.amazon.com/general/latest/gr/rande.html#s3_region)|Yes|A string value representing a valid Amazon RegionEndpoint.  <br><br>While strictly speaking, AWS S3 doesn't require a Region Endpoint, the client being used to access S3 needs to connect somewhere, so the endpoint is required.


### Parameters

The Parameters section is a list of "FileSets", that contain one or more "Sources" that are to be copied or moved to one or more "Destinations".  (You can only "Move" sources to a single destination.) 

#### Sample
````yaml
  Parameters:
    Type: Yaml
    Values:
      Targets:
      - C:\Temp\FileUtil\Source1\web.config
      - C:\Temp\FileUtil\Source2\
      - \\server\c$\Temp\FileUtil\Source3\
      - \\server\c$FileUtil\Source4\myfile.txt
      - s3://mybucket/Destination/EmptyFolder/
      - s3://mybucket/Destination/Full Folder/
      - s3://mybucket/Destination/SomeFile.yaml
````

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|Targets|List of String|Yes|A list of one or more files and/or directories to be deleted.

