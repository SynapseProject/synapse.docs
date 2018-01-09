# Overview
The CopyFileHandler allows files and directories to be moved and/or copied from one destination to another.  The table below shows the destinations currently supported by the handler along with the expected format of files / directories in the plan.

|Destination|File Format|Directory Format
|-----------|-----------|----------------
|Windows Server|C:\Temp\MyFile.txt|C:\Temp\MyDirectory\
|Network Attached Storage (NAS)|\\\server\share$\MyDir\MyFile.txt|\\\server\share$\MyDir\
|Amazon S3 Buckets|s3://bucket/mydir/myfile001.txt|s3://bucket/mydir/

**Note**: Directories must ALWAYS end in a slash.  This is mainly because in Windows, there's no way to tell, from the naming convention, if an object is a directory or a file with no file extension.


## Plan Details
### Config

The config section of the plan specifies the action to perform, as well as flags indicating how the action should be performed.  It also contains configuration sections to connect to network/cloud based endpoints that require authentication and additional configuration.

#### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.FileUtil:CopyFileHandler
    Config:
      Type: Yaml
      Values:
        Action: Copy
        OverwriteExisting: true
        IncludeSubdirectories : true
        PurgeDestination: false
        Verbose: true
        Aws:
          AccessKey: xxxxxxxx
          SecretKey: xxxxxxxx
          Region: eu-west-1
````

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Action|"Copy"<br>"Move"|Yes|Indicates the action to take on the files / directories specified.
|OverwriteExisting|Boolean|No|Whether or not to overwrite existing files / directories (Default = true)
|Recurse|Boolean|No|Should sub-directories be included in the action.  Only applies to "Copy". (Default = true)
|PurgeDestination|Boolean|No|Should the destination file/directory be purged before beginning the action (Default = false)
|Verbose|Boolean|No|Log details of each file / directory acted upon. (Default = true)
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
      FileSets:
      - Sources: 
        - C:\Temp\FileUtil\Source1\
        - \\localhost\c$\Temp\FileUtil\Source2\
        - s3://mybucket/Source/
        Destinations: 
        - C:\Temp\FileUtil\Destination1\
        - \\localhost\c#\Temp\FileUtil\Destination2\
        - s3://mybucket/Destination/
````

|Element|Type/Value|Required|Description
|-------|----|--------|-----------
|FileSets|List of "Sources" and "Destinations"|Yes|A grouping of files and directories that are copied or moved to the destination(s).
|Sources|List of String|Yes|One or more "sources" (files or directories) that should be copied or moved to the destination(s).
|Destinations|List of String|Yes|One or more "destinations" (files or directories) that indicates where the sources are to be copied or moved.

