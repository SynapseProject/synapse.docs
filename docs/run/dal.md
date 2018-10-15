# Controller Data Access Layer

Synapse exposes an interface for developing a Controller Data Access Layer (DAL), whose function is to support enforcing the RBAC, fetching Plans, and recording Plan execution status as relayed from a Synapse Node.  The list below shows the available Controller DALs and their associated Config.  All DALs provide the same capability.

### Installation path

Locate the DAL files in a folder matching the Type, within the Synapse.Controller installation folder.  Example: _Synapse.Controller.Dal.{DAL-type}:{DAL-type}Dal_.

### DAL Configuration

The general layout for DAL configurations is:

```yaml
Controller:
  ...
  Dal:
    Type: Synapse.Controller.Dal.{DAL-type}:{DAL-type}Dal
    Config:
      {proprietary DAL-type config settings}
    LdapRoot:
```

|Setting|Description
|--------|--------
|Type|The library:classname of the DAL to invoke.
|Config|The DAL-specific runtime configuration.
|LdapRoot|LDAP connection string for supporting Security lookups from an LDAP provider, such as Active Directory.

## FileSystemDal

Status: Complete and fully functional.  The FileSystemDal is intended to support "databaseless" installations.

```yaml
Controller:
  ...
  Dal:
    Type: Synapse.Controller.Dal.FileSystem:FileSystemDal
    Config:
      PlanFolderPath: Plans
      HistoryFolderPath: History
      ProcessPlansOnSingleton: false
      ProcessActionsOnSingleton: true
      Security:
        FilePath: Security
        IsRequired: false
        GlobalExternalGroupsCsv: Everyone
    LdapRoot:
```

| Setting | Description
|--------|--------
|PlanFolderPath|The folder where Plan YAML files are stored.  This path is assumed to be relative unless specified as a rooted, absolute path. 
|HistoryFolderPath|The folder where ResultPlan YAML files are stored, post-execution.  This path is assumed to be relative unless specified as a rooted, absolute path. 
|ProcessPlansOnSingleton|Controls the multithreading access to ResultPlan files for complete-Plan updates.
|ProcessActionsOnSingleton|Controls the multithreading access to ResultPlan files for partial-Plan updates.
|Security.FilePath|The folder where RBAC files are stored.  This path is assumed to be relative unless specified as a rooted, absolute path.
|Security.IsRequired|Indicates whether the FileSystemDal allows execution without a Security setup.
|Security.GlobalExternalGroupsCsv|Globally available groups specified outside the RBAC files.


## AwsS3Dal

Status: Complete and fully functional.  The AwsS3Dal is intended to support "databaseless" installations.

```yaml
Controller:
  ...
  Dal:
    Type: Synapse.Controller.Dal.AwsS3:AwsS3Dal
    Config:
      AwsAccessKey: [optional]
      AwsSecretAccessKey: [optional]
      DefaultBucketName: s3://some-bucket-name [optional]
      PlanFolderPath: Plans
      HistoryFolderPath: History
      WriteHistoryAs: Yaml (default) | FormattedJson | CompressedJson
      ProcessPlansOnSingleton: false
      ProcessActionsOnSingleton: true
      Security:
        FilePath: Security
        IsRequired: false
        ValidateSignature: false
        SignaturePublicKeyFile: 
        GlobalExternalGroupsCsv: Everyone
    LdapRoot:
```

| Setting | Description
|--------|--------
|AwsAccessKey, AwsSecretAccessKey|Optional settings to provide authentication for the AWS client.  Supply both settings or neither, where the latter option then relies on native AWS IAM configuration.
|DefaultBucketName|The base S3 Bucket path used in relative-paths for Plans, History, and Security.  If each of those paths are specified as absolute paths, then this setting may be omitted.
|PlanFolderPath|The folder where Plan YAML files are stored.  This path is assumed to be relative unless specified as a rooted, absolute path. 
|HistoryFolderPath|The folder where ResultPlan YAML/JSON files are stored, post-execution.  This path is assumed to be relative unless specified as a rooted, absolute path. 
|WriteHistoryAs|Yaml (default), FormattedJson, or CompressedJson.  Enum to control output serialization format.  Some tools, such as AWS Athena, consume CompressedJson by default.
|ProcessPlansOnSingleton|Controls the multithreading access to ResultPlan files for complete-Plan updates.
|ProcessActionsOnSingleton|Controls the multithreading access to ResultPlan files for partial-Plan updates.
|Security.FilePath|The folder where RBAC files are stored.  This path is assumed to be relative unless specified as a rooted, absolute path.
|Security.IsRequired|Indicates whether the FileSystemDal allows execution without a Security setup.
|Security.GlobalExternalGroupsCsv|Globally available groups specified outside the RBAC files.


## ComponentizedDal

Status: Complete and fully functional.  The ComponentizedDal is intended to support multiple DALs simultaneously, where the Plans, History, and Security may be separated by implementation and are specified by keys referencing an array of local configuration settings.

| Setting | Description
|--------|--------
|ExecuteReaderKey|References the array key for fetching Plans for execution.
|HistoryWriterKey|References the array key for persisting/fetching Plan execution History.
|SecurityProviderKey|References the array key for Security calls.
|DalComponents|An array of Key/Type/Config structures, where each specifies a complete DAL configuration.  The Key setting will be referenced by the ExecuteReaderKey/HistoryWriterKey/SecurityProviderKey settings above.  DalComponents may be used amongst multiple settings, if desired.


```yaml
#Example 1:
Controller:
  ...
  Dal:
    Type: Synapse.Controller.Dal.Componentized:ComponentizedDal
    LdapRoot: 
    Config:
      ExecuteReaderKey: FileSystem
      HistoryWriterKey: AwsS3
      SecurityProviderKey: Xyz
      DalComponents:
      - Key: AwsS3
        Type: Synapse.Controller.Dal.AwsS3:AwsS3Dal
        Config:
          [...]
      - Key: FileSystem
        Type: Synapse.Controller.Dal.FileSystem:FileSystemDal
        Config:
          [...]
      - Key: Xyz
        Type: Synapse.Controller.Dal.Xyz:Xyz3Dal
        Config:
          [...]

#Example 2:
Controller:
  ...
  Dal:
    Type: Synapse.Controller.Dal.Componentized:ComponentizedDal
    LdapRoot: 
    Config:
      ExecuteReaderKey: AwsS3
      HistoryWriterKey: AwsS3
      SecurityProviderKey: Xyz
      DalComponents:
      - Key: AwsS3
        Type: Synapse.Controller.Dal.AwsS3:AwsS3Dal
        Config:
          [...]
      - Key: Xyz
        Type: Synapse.Controller.Dal.Xyz:Xyz3Dal
        Config:
          [...]
```


## MongoDB

Status: Under development.  Supports fetching Plans for execution from disk, records status in MongoDB.  Does not yet enforce RBAC.

```yaml
Controller:
  ...
  Dal:
    Type: Synapse.Controller.Dal.MongoDB:MongoDBDal
    Config:
      {no further settings at this time}
    LdapRoot:
```

## SQL Server

Status: Under initial development.  Intended for use with Synapse.Enterprise.

```yaml
Controller:
  ...
  Dal:
    Type: Synapse.Controller.Dal.SqlServer:SqlServerDal
    Config:
      {no further settings at this time}
    LdapRoot:
```