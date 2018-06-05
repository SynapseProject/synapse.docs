# Overview
A "RoleManager" is responsible for the administration and enforcement of "roles" assigned to a given principal (user or group) on an object or set of objects in the Active Directory.  The details of how this is accomplished is left to the specified implementation, but at a high level, a "role" should contain one or more allowed "actions" that be assigned to a "principal" (user or group) on one or more "objects" in the Active Directory.

## Terms
- **Role** : A logical grouping of allowed "actions".
- **Action** : A task that can be executed against the Active Directory.  Click [here](handler.md#actions) for a detailed listing.
- **Principal** : A User or Group to which a role can be assigned, either directly or through inheritance.  An optional domain can be prepended for principals in other domains.  Click [here](handler.md#activedirectory-objects-and-identities) for more information about domains.
- **Object** : An ActiveDirectory object to which the role applies (either directly or through inheritance).  An optional domain can be prepended for objects in other domains.  Click [here](handler.md#activedirectory-objects-and-identities) for more information about domains.

## Example

The role "Administrators" can perform all actions against the ActiveDirectory.  This role is assigned to principal (group) "MyOuAdmins", and is applied to the object "MyOu" and all its children (through inheritance).   

Thus, if user "scott" is in the group "MyOuAdmins" and wants to delete a user "tiger" (cn=tiger,ou=MyOu,ou=Dept001,dc=sandbox,dc=local) which is under the "MyOu" OrganizationalUnit, the RoleManager would evaluate scott's roles and determine if he was allowed to perform that action.

# Interface Methods

## Configuration

### Initialize

This method receives the [config section](#handler-configuration) from the Handler config file and passes it into the RoleManager implementations

Inputs:

- **Config** : A generic "object" that contains configuration information specific to the implementation of the RoleManager itself.  

Returns: **Nothing**

## Role Execution

### CanPerformAction

Determines if a "Principal" (user or group) can perform an action on an ActiveDirectory object.

Inputs :

- **Principal** : The user or group trying to perform the action
- **Action** : The action being performed (Click [here](handler.md#actions) for detailed list of actions.)
- **Object** : The ActiveDirectory object on which the action is to be performed.

Returns :

- **Boolean** : A "true" or "false" value indicating whether the principal is allowed to perform the action.

### CanPerformActionOrException

Determines if a "Principal" (user or group) can perform an action on an ActiveDirectory object.  Same as the [CanPerformAction](#canperformaction) method above, but will throw an Exception if the principal is not allowed to perform the action.

## Role Administration

### GetRoles

Gets a list of all roles that can be applied.

Inputs : **None**

Returns : A list of strings that represent the role names that can be applied to principals and objects.

### HasRole

Determines if a "Principal "user or group" as a specific role on an ActiveDirectory object.

Inputs :

- **Principal** : The user or group being asked about.
- **Role** : The role being asked about.
- **Object** : The ActiveDirectory object on which the role should be checked.

Returns :

- **Boolean** : A "true" or "false" value indicating whether the principal has the specified role on the object.

### AddRole

Adds a role for a given principal onto an ActiveDirectory object.

Inputs :

- **Principal** : The user or group being granted a role.
- **Role** : The role being added.
- **Object** : The ActiveDirectory object on which the role applies.

Returns : **Nothing**

### RemoveRole

Removes a role for a given principal from an ActiveDirectory object.

Inputs :

- **Principal** : The user or group for which the role is being removed.
- **Role** : The role being removed.
- **Object** : The ActiveDirectory object on which the role applies.

Returns : **Nothing**

# Handler Configuration

The role manager to use, and any other necessary configuration specific to the RoleManager implementation is found in the Handler Configuration file.   The file "Synapse.Handlers.ActiveDirecory.config.yaml" should be located in the same location as the Handler DLL.   The "RoleManager" section contains the "Name" of the RoleManager to use, and the "Config" associated with it.

If the file cannot be found, or the RoleManager section is not present, the default RoleManager ([DefaultRoleManager](#defaultrolemanager)) will be used.

````yaml
RoleManager:
  Name: Synapse.ActiveDirectory.MyRoleManagerDLL:MyRoleManager
  Config: 
    - More
    - Config
    - Here
````

- **Name** : Contains a colon seperated string that indicates the DLL and the class (with namespace if any) to use for RoleManagement.  In the example above, the class "MyRoleManager" has no namespace.
- **Config** : This Yaml will be passed directly into the "Initialize" method of the RoleManager implementation and has no pre-defined format.  The shape of the data here is however the RoleManager implementation expects it to be.

# Implementations

Below are the known implementations of RoleManager that are available for use in the ActiveDirectory Handler and Api.

## DefaultRoleManager

This very simply returns "true" for all "Has" and "Can" methods, and throws a "NotImplementedException" for the "AddRole" and "RemoveRole" methods.

### Configuration

````yaml
RoleManager:
  Name: Synapse.ActiveDirectory.Core:DefaultRoleManager
  Config: 
````

There is no configuration expected for this implementation.


## DaclRoleManager

This implements the RoleManager by applying DACL's directly onto the ActiveDirectory objects.  These are done by placing AccessRules directly onto the objects for the given principals, then evaulated by looking at the permissions the principal (and all groups that principal belongs to) on that object to determine if the principal can perform actions on the object in question.

### Configuration

````yaml
RoleManager:
  Name: Synapse.ActiveDirectory.DaclRoleManager:DaclRoleManager
  Config: 
    Roles:
    - Name: AdReadOnly
      AllowedActions: Get, Search
      AdRights: GenericRead
    - Name: AdReadWrite
      AllowedActions: Create, Modify, Delete, Rename, Move
      AdRights: GenericWrite
      ExtendsRoles:
      - AdReadOnly
    - Name: AdGroupManagement
      AllowedActions: AddToGroup, RemoveFromGroup
      AdRights: GenericExecute
      ExtendsRoles:
      - AdReadOnly
    - Name: AdAccessRights
      AllowedActions: AddAccessRule, RemoveAccessRule, SetAccessRule, PurgeAccessRules
      AdRights: WriteDacl
      ExtendsRoles:
      - AdReadOnly
    - Name: AdRoleDelegate
      AllowedActions: AddRole, RemoveRole
      AdRights: WriteDacl, WriteOwner
      ExtendsRoles:
      - AdReadOnly
    - Name: AdOwner
      AllowedActions: All
      AdRights: GenericAll
````

The config section contains a list of roles that can be granted to a principal for a given object.  Each entry in that list contains : 

- **Name** : The name of the role.
- **AllowedActions** : A comma-seperated list of [Actions](handler.md#actions) that can be performed by this role
- **AdRights** : A comma-seperated list of [ActiveDirectoryRights](handler.md#activedirectoryrights-enumeration) that when applied to an object in an AccessRule, define a role.
- **ExtendsRoles** : This is a list of other roles that also apply to this role.  All allowed actions will apply to this role, and all AdRights will be applied to objects as well.  This is a way allow for Role Inheritance, prevent duplication, and to simplify the way the config section is defined.  The same outcome could be achieved by just including the AllowedActions and AdRights directly.

### Implementation Summary

|Method|Description
|------|-----------
|Initialize|Builds an in-memory representation of the roles and actions defined in the config file.  If the config section changes, they won't be reflected until the Node / Handler is restarted.
|CanPerformAction|Retrives all the rights for the principal or any group the principal belongs to and examines the cumulative rights to see if the principal belongs to a role that allows the given action.
|GetRoles|Returns a list of role names that can be applied.
|HasRole|Determines if a principal, or a group the principal belongs to, has the necessary Active Directory rights (DACL's) on an ActiveDirectory object to be in the given "role".
|AddRole|Assigns the defined rights (DACL's) for a principal on the object.
|RemoveRole|Removes the defiend rights (DACL's) for a principal on the object.