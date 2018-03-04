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


## MongoDB

Status: Under developemnt.  Supports fetching Plans for execution from disk, records status in MongoDB.  Does not yet enforce RBAC.

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

Status: Under initial developemnt.  Intended for use with Synapse.Enterprise.

```yaml
Controller:
  ...
  Dal:
    Type: Synapse.Controller.Dal.SqlServer:SqlServerDal
    Config:
      {no further settings at this time}
    LdapRoot:
```