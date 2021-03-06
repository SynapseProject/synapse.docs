# Overview
The ActiveDirectory Handler allows a simple, programatic way to interact with an ActiveDirectory instance.

# Supported Objects and Actions
## ActiveDirectory Objects and Identities

Below is a table of supported active diretory object types, and the supported way to "identify" an object for all actions except for Create.  Create (or Modify using upsert) MUST provide the full distinguished name.

|**Object Type**|Distinghished Name|Name|UserPrincipal|SamAccountName|Sid|Guid
|---------|-----------------|----|-------------|--------------|---|----
|**User**|Yes|Yes|Yes|Yes|Yes|Yes
|**Group**|Yes|Yes|No|Yes|Yes|Yes
|**Organizational Unit**|Yes|Yes(1)|No|No|No|Yes
|**Computer**|Yes|Yes|Yes|Yes|Yes|Yes

(1) - Assumes only a single name exists.  Since multiple objects are allowed to have the same name under different organizational units, when multiple objects exist with the same name, an error will occur.

### Domains

By default, the handler assumes the identity is in the same domian as the server where the handler is running.  When specifying an identity in a different domain, a domain qualifier must be pre-pended to the identity, unless already included in the identity itself (Distinguished Name and UserPrincipalName).  Below shows examples of every identity type supported in both the "default" domain (SANDBOX) and a different domain (SB2 for this example).

|**Identity Type**|Examples
|-----------------|--------
|**DistinguishedName**|cn=user002,ou=myorgunit,**dc=sandbox,dc=local**<br />cn=user002,ou=myorgunit,**dc=sb2,dc=local**
|**Name**|Joe User<br/>**SB2\\**Joe User
|**UserPrincipal**|User003**@sandbox.local**<br/>User003**@sb2.local**
|**SamAccountName**|User004Sam<br/>**SB2\\**User004Sam
|**SecurityIdentifier (Sid)**|S-1-5-21-4054027134-3251639354-3875066094-1704<br/>**SB2\\**S-1-5-21-4054027134-3251639354-3875066094-1720
|**Guid**|2c534dc4-06ba-4dc0-b86c-0eabdeb158f5<br/>**SB2\\**0fe13b30-7500-424e-a7c9-d0d7cc9ecec6

**Note :** If no domain is explicitly provided, the server domain (where the hander is running) is assumed.

## Actions

Below is a list of supported actions that can be performed on ActiveDirectory objects.  The action will execute against the objects in a certain order, depending on the action.  For example, Organizational Units will be created before users and groups that might be members of that Organizational Unit.

|**Action**|Description|Order of Operations
|----------|-----------|-------------------
|**Get**|Retrieves a single ActiveDirectory object by its [identity](#activedirectory-objects-and-identities).|Users<br>Groups<br>Computers<br>Organizational Units
|**Create**|Creates a single ActiveDirectory object by its distinguished name only. (2)|Organizational Units<br>Computers<br>Groups<br>Users
|**Modify**|Modifies a single ActiveDirectory object by its [identity](#activedirectory-objects-and-identities). (2)|Organizational Units<br>Computers<br>Groups<br>Users
|**Delete**|Deletes a single ActiveDirectory object by its [identity](#activedirectory-objects-and-identities)|Users<br>Groups<br>Computers<br>Organizational Units
|**Move**|Moves a single ActiveDirectory object by its [identity](#activedirectory-objects-and-identities) to another Organizational Unit.|Users<br>Groups<br>Computers<br>Organizational Units
|**AddToGroup**|Adds Users and/or Groups to an existing ActiveDirecory group by its [identity](#activedirectory-objects-and-identities).|Users<br>Groups<br>Computers
|**RemoveFromGroup**|Removes Users and/or Groups from an existing ActiveDirectory group by its [identity](#activedirectory-objects-and-identities).|Users<br>Groups<br>Computers
|**AddAccessRule**|Adds Access Rights to an ActiveDirectory object for a given Principal (User or Group)|Users<br>Groups<br>Computers<br>Organizational Units
|**RemoveAccessRule**|Removes Access Rights from an ActiveDirectory object for a given Principal (User or Group)|Users<br>Groups<br>Computers<br>Organizational Units
|**SetAccessRule**|Sets Access Rights on an ActiveDirectory object for a given Principal (User or Group), clearing out any existing rights that might exist.|Users<br>Groups<br>Computers<br>Organizational Units
|**PurgeAccessRules**|Removes All Access Rights to an ActiveDirectory object for a given Principal (User or Group)|Users<br>Groups<br>Computers<br>Organizational Units
|**AddRole**|Adds a role to an ActiveDirectory object for a given Principal (User or Group)|Users<br>Groups<br>Computers<br>Organizational Units
|**RemoveRole**|Removes a role from an ActiveDirectory object for a given Principal (User or Group)|Users<br>Groups<br>Computers<br>Organizational Units
|**Search**|Searches for ActiveDirectory objects based on standard LDAP Filter String syntax|Not Applicable
|**None**|Used only for assignment of actions to roles in the [RoleManager](#role-manager).  This can not be used in plan execution.|N/A
|**All**|Used only for assignment of actions to roles in the [RoleManager](#role-manager).  This can not be used in plan execution.|N/A



(2) - When the "UseUpsert" config is set, calling "Create" on an existing object can be done using its [identity](#activedirectory-objects-and-identities).  Calling "Modify" on an object that does not exist will only create the object if called using a distinguished name.

# Role Manager

The role manager implements an RBAC (Role-Based Access Control) to control access to ActiveDirectory objects.  The interface provides Role Administration functionality (Adding and Removing roles) as well as answers a very simple question... Does User or Group X have permission to perform action Y on ActiveDirectory object Z?

## Handler Config File

The ActiveDirectory Handler takes a configuration file that specifies which RoleManager (if any) should be used as well as any relevant configuration needed by that RoleManager to execute its job.   The RoleManager is loaded by the handler at the time of execution.  The ActiveDirectory handler loads the config information from the file "Synapse.Handlers.ActiveDirectory.config.yaml" which should be located in the same location as the RoleManager DLL.  If no config file is present, or no RoleManager is specified, the "[DefaultRoleManager](rolemanager.md#defaultrolemanager)" will be used.  Below is the format of the config file:

````yaml
RoleManager:
  Name: Synapse.ActiveDirectory.MyRoleManager:MyCustomRoleManager
  Config: 
    Custom: Config Structure
    For: The RoleManager Implementation
````

For more details about which RoleManagers are available and how they work, see the [Role Manager](rolemanager.md) page.

# Plan Details
## Config

The config section of the plan specifies the action to perform against that AD instance, how the action should be run, and what information to return from the call.

### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.ActiveDirectory:ActiveDirectoryHandler
    Config:
      Type: Yaml
      Values:
        Action: Create
        RunSequential: true
        ReturnGroupMembership: false
        ReturnObjects: true
        ReturnObjectProperties: true
        ReturnAccessRules: false
        SuppressOutput: false
        UseUpsert: true
        OutputType: Yaml
        PrettyPrint: true
````

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Action|"Get"<br>"Create"<br>"Modify"<br>"Delete"<br>"AddToGroup"<br>"RemoveFromGroup"|Yes|Specifies the action that will be executed against the active directory instance.
|RunSequential|boolean|No|Tells the adapter how to process the objects in the plan.  If set to "true", objects will be acted upon in the order they appear in the plan.  (Defualt = "false")
|ReturnGroupMembership|boolean|No|Tells the adapter to return a list of groups that the User or Group is a member of.  (Default = "true")
|ReturnObjects|boolean|No|Tells the adapter to return the ActiveDirectory object along with the status of the action.  (Default = "true")
|ReturnObjectProperties|boolean|No|Tells the adapter to return the raw, DirectoryEntry properties associated with an ActiveDirectory object (Default = "true").
|ReturnAccessRules|boolean|No|Tells the adatper to return all the Access Rules associated with the object along with the status of the action (Default = "false")
|SuppressOutput|boolean|No|Tells the adapter to not output the ExitData returned from the action into the logs.  (Default = "false")
|UseUpsert|boolean|No|Tells the adapter to perform a "Modify" action when a "Create" action is called and the object exists, and to perform a "Create" action when a "Modify" action is called and the object does not exist.  (Default = "true")
|OutputType|"Json"<br>"Xml"<br>"Yaml"|No|Tells the adapter how to format the returned status from the action.  (Default = "Json")
|PrettyPrint|boolean|No|Tells the adapter whether to indent and add newlines to the returned output.  (Default = "true")

## Parameters

The parameters section of the plan is simply a list of each type of object you wish to perform the action on.  Every object must contain an [identity](#activedirectory-objects-and-identities), but depending on the action, other fields might be required.  Below are examples of each action and object type with the required fields.

---
### Action: Get and Delete, Object Type: All
````yaml
  Parameters:
    Type: Yaml
    Values:
      Users:
      - Identity: user1                                            # By Name
      - Identity: cn=user2,ou=myorgunit,dc=sandbox,dc=local        # By Disginguished Name
      - Identity: 2c534dc4-06ba-4dc0-b86c-0eabdeb158f5             # By Guid
      - Identity: user3@sandbox.local                              # By User Principal
      - Identity: user4                                            # By SamAccountName
      - Identity: S-1-5-21-4054027134-3251639354-3875066094-1704   # By Sid
      Groups:
      - Identity: MyGroup001                                       # By Name
      - Identity: cn=MyGroup002,ou=myorgunit,dc=sandbox,dc=local   # By Disginguished Name
      - Identity: 0fe13b30-7500-424e-a7c9-d0d7cc9ecec6             # By Guid
      - Identity: MyGroup003                                       # By SamAccountName
      - Identity: S-1-5-21-4054027134-3251639354-3875066094-1720   # By Sid
      OrganizationalUnits:
      - Identity: MyOrgUnit                                        # By Name
      - Identity: ou=myorgunit,dc=sandbox,dc=local                 # By Distinguished Name
      - Identity: fe00548a-1b8b-490f-8601-7646c290fc8a             # By Guid
      Computers:
      - Identity: computer1                                        # By Name
      - Identity: cn=computer1,ou=myorgunit,dc=sandbox,dc=local    # By Disginguished Name
      - Identity: 2c534dc4-06ba-4dc0-b86c-0eabdeb158f8             # By Guid
      - Identity: computer1@sandbox.local                          # By User Principal
      - Identity: computer1Sam                                     # By SamAccountName
      - Identity: S-1-5-21-4054027134-3251639354-3875066094-1705   # By Sid

````

For Get and Delete actions, the only field required for every ActiveDirectory object is its identity.  The example above shows each object type with every possible way it can be queried or deleted.

---
### Action: AddToGroup and RemoveFromGroup, Object Type: Users, Groups and Computers
````yaml
  Parameters:
    Type: Yaml
    Values:
      Users:
      - Identity: user1                                            # Any Valid User Identity
        MemberOf:
        - MyGroup001                                               # Any Valid Group Identity
        - cn=MyGroup002,ou=myorgunit,dc=sandbox,dc=local           # Any Valid Group Identity
      Groups:
      - Identity: MyGroup001                                       # Any Valid Group Identity
        MemberOf:
        - S-1-5-21-4054027134-3251639354-3875066094-1720           # Any Valid Group Identity
      Computers:
      - Identity: MyComputer001                                    # Any Valid Computer Identity
        MemberOf:
        - S-1-5-21-4054027134-3251639354-3875066094-1720           # Any Valid Group Identity
````

For Group Membership actions (AddToGroup and RemoveFromGroup), a "MemberOf" section is added below the Identity of each ActiveDirectory object.  The "MemberOf" section is a list of valid group [identities](#activedirectory-objects-and-identities) that specify which group the "Add" or "Remove" action it to be performed on.

---
### Action: Create or Modify, Object Type: Users
````yaml
  Parameters:
    Type: Yaml
    Values:
      Users:
      - Identity: cn=mfox,ou=MyOrgUnit,DC=sandbox,DC=local
        Name: Michael Fox            # This will change the "Common Name" of the object.  Use To "Rename" a User.
        Password: bi@02LL49_VWQ{b
        GivenName: Michael
        MiddleName: J
        Surname: Fox
        EmailAddress: mfox@company.com
        VoiceTelephoneNumber: 1-800-555-1212
        EmployeeId: 42
        Enabled: true
        SmartcardLogonRequired: true
        DelegationPermitted: false
        HomeDirectory: C:\Users\user1
        ScriptPath: C:\Users\user1\Scripts
        PasswordNotRequired: true
        PasswordNeverExpires: true
        UserCannotChangePassword: true
        AllowReversiblePasswordEncryption: true
        HomeDrive: F
        AccountExpirationDate: 2017-08-10T16:35:46.0417954Z
        PermittedLogonTimes:
        - 255
        - 0
        - 255
        - 0
        - 255
        - 0
        - 255
        - 0
        - 255
        - 0
        - 255
        - 0
        - 255
        - 0
        - 255
        - 0
        - 255
        - 0
        - 255
        - 0
        - 255
        DisplayName : Fox, Michael J
        Description: American Actor, Back to the Future
        UserPrincipalName: mfox@sandbox.local
        SamAccountName: mfox
        Groups:
        - Group001
        - Group002
        - Group003
        Properties:               # See Description Below For Details
          initials:
          - MJF
          otherTelephone:
          - 281-555-1212
          - 832-555-1212
          wWWHomePage:
          - http://www.google.com
          url:
          - http://www.bing.com
          - http://www.yahoo.com
          - http://www.github.com
````

The named fields are directly mapped to values in the [System.DirectoryServices.AccountManagement.UserPrincipal](https://msdn.microsoft.com/en-us/library/system.directoryservices.accountmanagement.userprincipal(v=vs.110).aspx) class in .NET.  Descriptions can be found in the Microsoft documentation [here](https://msdn.microsoft.com/en-us/library/system.directoryservices.accountmanagement.userprincipal(v=vs.110).aspx).

**Default Values For Fields**

All the fields above are optional with the exception of "Identity" (which must be a Distinguished Name for "Create" actions).  Below is a list of fields that will be defaulted to values if not explicitly included on an object creation.

* UserPrincipalName: Defaults to "Name@Domain" if not provided on Create.
* SamAccountName: Defaults to "Name" if not provided on Create.

**Properties**

Values under the "Properties" section are dynamic.  Any "attribute" that ActiveDirectory accepts for this type of object can be included here, in a "Key" and "Values" arrangement.  Most settable "attributes" for a User can be found on the microsoft site [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms677980(v=vs.85).aspx).  The "key" should represent the "Ldap-Display-Name" value of each attribute.

---
### Action: Create or Modify, Object Type: Groups
````yaml
  Parameters:
    Type: Yaml
    Values:
      Groups:
      - Identity: cn=MyNewGroup,ou=Synapse,DC=sandbox,DC=local
        Name: MyNewerGroup            # This will change the "Common Name" of the object.  Use To "Rename" a Group.
        Scope: Universal     # Local, Global or Universal
        IsSecurityGroup: true
        SamAccountName: MyNewGroupSam
        Description: Some Lame Description
        ManagedBy: mfox      # Can Be Identity of any User or Group
        Properties:
          managedBy:
          - cn=mfox,ou=MyOrgUnit,DC=sandbox,DC=local   # Property Takes Precedence Over Field
          mail:
          - ~null~
          info:
          - Some Random Notes Here
````
**Default Values For Fields**

All the fields above are optional with the exception of "Identity" (which must be a Distinguished Name for "Create" actions).  Below is a list of fields that will be defaulted to values if not explicitly included on an object creation.

* SamAccountName: Defaults to "Name" if not provided on Create.

**Properties**

Values under the "Properties" section are dynamic.  Any "attribute" that ActiveDirectory accepts for this type of object can be included here, in a "Key" and "Values" arrangement.  Most settable "attributes" for a Group can be found on the microsoft site [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms677980(v=vs.85).aspx).  The "key" should represent the "Ldap-Display-Name" value of each attribute.

---
### Action: Create or Modify, Object Type: Organizational Units
````yaml
  Parameters:
    Type: Yaml
    Values:
      OrganizationalUnits:
      - Identity: ou=MyOrgUnit,dc=sandbox,dc=local
        Name: MyNewerOrgUnit            # This will change the "Name" of the object.  Use To "Rename" an Organizational Unit.
        Description: Some Lame Description
        ManagedBy: mfox    # Can Be Identity of any User or Group
        Properties:
          managedBy:
          - cn=mfox,ou=MyOrgUnit,DC=sandbox,DC=local    # Property Takes Precedence Over Field
          c:
          - US
          l:
          - ~null~
````
**Properties**

Values under the "Properties" section are dynamic.  Any "attribute" that ActiveDirectory accepts for this type of object can be included here, in a "Key" and "Values" arrangement.  Most settable "attributes" for an Organizational Unit can be found on the microsoft site [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms677623(v=vs.85).aspx).  The "key" should represent the "Ldap-Display-Name" value of each attribute.

---
### Action: Create or Modify, Object Type: Computers
````yaml
  Parameters:
    Type: Yaml
    Values:
      Computers:
      - Identity: cn=MyComputer,dc=sandbox,dc=local
        Name: MyRenamedComputer              # This will change the "Name" of the object.  Use To "Rename" an Organizational Unit.
        Description: Some Lame Description
        ManagedBy: mfox    # Can Be Identity of any User or Group
        Properties:
          managedBy:
          - cn=mfox,ou=MyOrgUnit,DC=sandbox,DC=local    # Property Takes Precedence Over Field
          userAccountControl:
          - 544
          operatingSystem:
          - ~null~
````
**Properties**

Values under the "Properties" section are dynamic.  Any "attribute" that ActiveDirectory accepts for this type of object can be included here, in a "Key" and "Values" arrangement.  Most settable "attributes" for a Computer can be found on the microsoft site [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms675736(v=vs.85).aspx).  The "key" should represent the "Ldap-Display-Name" value of each attribute.

---
### Action: AddAccessRule, RemoveAccessRule or SetAccessRule, Object Type: All

````yaml
  Parameters:
    Type: Yaml
    Values:
      Users:
      - Identity: SomeUser001
        AccessRules:
        - Identity: TestUser001
          Type: Allow
          Rights: Self,GenericRead
          Inheritance: None
        - Identity: TestUser001
          Type: Deny
          Rights: ListChildren
          Inheritance: All
      Groups:
      - Identity: SomeGroup001
        AccessRules:
        - Identity: TestUser001
          Type: Allow
          Rights: Self,GenericRead
          Inheritance: Descendents
        - Identity: TestUser001
          Type: Deny
          Rights: ListChildren
          Inheritance: SelfAndChildren
      OrganizationalUnits:
      - Identity: SomeOrgUnit001
        AccessRules:
        - Identity: MyComputer
          Type: Allow
          Rights: Self,GenericRead
          Inheritance: Children
        - Identity: TestUser001
          Type: Deny
          Rights: ListChildren
      Computers:
      - Identity: SomeComputer001
        AccessRules:
        - Identity: TestUser001
          Type: Allow
          Rights: Self,GenericRead
          Inheritance: Children
        - Identity: TestUser001
          Type: Deny
          Rights: ListChildren
````

The "AccessRules" section creates AccessRules to allow or deny rights on the object to a given Principal (User, Group or Computer).  The "Type" field can either be the value "Allow" or "Deny".  The Rights field maps to the ActiveDirectoryRights Enumeration in C#.  To assign multiple rights in a single rule, comma-separate the values in the field (ex: "Self, GenericRead").   Below is the list of rights supported : 

#### ActiveDirectoryRights Enumeration

The valid values for the list of rights assignable in an access rule are based on the C# enumeration "ActiveDirectoryRights" in the System.DirectoryServices namespace.  Below are the valid values at the time this document was created.  For a more detailed view, see the [Microsoft documentation site](https://msdn.microsoft.com/en-us/library/system.directoryservices.activedirectoryrights(v=vs.110).aspx). 

|Right|Description
|-----|-----------
|AccessSystemSecurity|The right to get or set the SACL in the object security descriptor.
|CreateChild|The right to create children of the object.
|Delete|The right to delete the object.
|DeleteChild|The right to delete children of the object.
|DeleteTree|The right to delete all children of this object, regardless of the permissions of the children.
|ExtendedRight|A customized control access right. For a list of possible extended rights, see the topic "Extended Rights" in the MSDN Library at http://msdn.microsoft.com. For more information about extended rights, see the topic "Control Access Rights" in the MSDN Library at http://msdn.microsoft.com.
|GenericAll|The right to create or delete children, delete a subtree, read and write properties, examine children and the object itself, add and remove the object from the directory, and read or write with an extended right.
|GenericExecute|The right to read permissions on, and list the contents of, a container object.
|GenericRead|The right to read permissions on this object, read all the properties on this object, list this object name when the parent container is listed, and list the contents of this object if it is a container.
|GenericWrite|The right to read permissions on this object, write all the properties on this object, and perform all validated writes to this object.
|ListChildren|The right to list children of this object. For more information about this right, see the topic "Controlling Object Visibility" in the MSDN Library http://msdn.microsoft.com/library.
|ListObject|The right to list a particular object. For more information about this right, see the topic "Controlling Object Visibility" in the MSDN Library at http://msdn.microsoft.com/library.
|ReadControl|The right to read data from the security descriptor of the object, not including the data in the SACL.
|ReadProperty|The right to read properties of the object.
|Self|The right to perform an operation that is controlled by a validated write access right.
|Synchronize|The right to use the object for synchronization. This right enables a thread to wait until that object is in the signaled state.
|WriteDacl|The right to modify the DACL in the object security descriptor.
|WriteOwner|The right to assume ownership of the object. The user must be an object trustee. The user cannot transfer the ownership to other users.
|WriteProperty|The right to write properties of the object.

#### Inheritance Enumeration

The valid values for the list of inheritance assignable to an access rule are based on the C# enumeration "ActiveDirectorySecurityInheritance" in the System.DirectoryServices namespace.  Below are the valid values at the time this document was created.  For a more detailed view, see the [Microsoft documentation site](https://msdn.microsoft.com/en-us/library/system.directoryservices.activedirectorysecurityinheritance(v=vs.110).aspx). 

In the table below, the column "Applies To" assumes an OrgUnit structure like :

**A --> B --> C**

- C is a child of B
- B is a child of A


|Inheritance|Description|Applies To
|-----------|-----------|----------
|None|No Inheritance.|A
|All|Inclues the object, all children, and all their descendants.|A,B,C
|Descendents|Inclues all children and their descendants, but not the object itself.|B,C
|SelfAndChildren|Inclues the object and all children, but not any descendants of the children.|A,B
|Children|Includes all children, but not the object or any of the children's descendants.|B

### Action: AddRole or Remove Role, Object Type: All

````yaml
  Parameters:
    Type: Yaml
    Values:
      Users:
      - Identity: SomeUser001
        Roles:
        - Name: AdOwner
          Principal: TestUser001
      Groups:
      - Identity: SomeGroup001
        Roles:
        - Name: AdReadWrite
          Principal: TestUser001
      OrganizationalUnits:
      - Identity: ou=Synapse,dc=sandbox,dc=local
        Roles:
        - Name: AdSomeOtherRole
          Principal: TestUser001
      Computers:
      - Identity: cn=MyComputer,ou=Computers,dc=sandbox,dc=local
        Roles:
        - Name: AdSomeOtherRole
          Principal: TestUser001
````

The "Roles" element defines the role "name" and to which principal (user or group) the role should be applied or removed from.  See the [Role Manager](#rolemanager) section above for details on how to define roles.

### Action: Move, Object Type: All

````yaml
  Parameters:
    Type: Yaml
    Values:
      Users:
      - Identity: cn=MoveMeUser,ou=Source,ou=MoveMe,ou=Synapse,dc=sandbox,dc=local
        MoveTo: ou=Destination,ou=MoveMe,ou=Synapse,dc=sandbox,dc=local
      Groups:
      - Identity: MoveMeGroup
        MoveTo: Destination
      OrganizationalUnits:
      - Identity: MoveMeOrgUnit
        MoveTo: Destination
      Computers:
      - Identity: MoveMeComputer
        MoveTo: Destination
````

The "MoveTo" element must be the identity of an Organizational Unit.

### Action: Search, Object Type : Not Applicable

````yaml
  Parameters:
    Type: Yaml
    Values:
      SearchRequests:
      - Filter: (objectClass=User)
        SearchBase: ou=Synapse,dc=sandbox,dc=local
        ResultsFile: C:\\Temp\\MyResults.json
        ReturnAttributes:
        - Name
        - objectGUID
````

The "Search" action doesn't apply to any single object, rather it takes a [Search Filter](https://msdn.microsoft.com/en-us/library/aa746475(v=vs.85).aspx) and a SearchBase and returns the defined "ReturnAttributes" for each DirectoryEntry that matches the filter.

The "ResultsFile" parameter saves the search results to an external file.  The location specified here must be accessible by the account under which the Synapse node is running.  This feature was provided to return large search results which exceed the allowed max length of a string.  If the size of your results throw an "OutOfMemory" error, use this field and set the "ReturnObjects" flag in Config to "false".

# Important Notes

## Modify : Clear Out A Field (~null~)

The action "Modify" only changes the fields you include in the plan.  You are not required to set every single field you want to keep on a modify.   However, this does beg the question, "How do I 'clear out' a field?".  This is done by placing the value "**~null~**" in the field.  For properties, if any of the possible values is "**~null~**", then it will clear out all values, ignoring any other values specififed in the plan.

## Properties

Things to know about using properites : 

* Propeprties directly interact with the underlying DirectoryEntry object.  You must provide the values in the format ActiveDirectory expects.  (Example: The property "managedby" expects a DisginguishedName).
* Any values that appear in a plan both as a class value and a property (ManagedBy, DisplayName, etc...) will take the value from the property.   Class variables are processed first, followed by the properties.

## ManagedBy Field vs Property

The "ManagedBy" field of Groups and Organizational Units can be any valid identity of a User or Group.  The handler looks up the User or Group and provides the DistinguishedName to the "managedby" property.

The "managedBy" property must be the DistinguishedName of a User or Group.  Properties directly manipulate the underlying DirectoryEntry and must match expected formats.

## Appendix A: Object Properties

The object "Properties" section refers to the ActiveDirectory "Attributes" on each object.   Since there is not a finite set of attributes that are allowed and custom attributes can be added at any time to any object type, this is represented in the handler as a list of key value pairs.

The "key" represents the ldapDisplayName of an attribute, and the "value" is an array of strings, representing the one or more values the attribute can have.

A list of default available properties for each object type can be found at the links below : 

Object Type|Reference URLs
-----------|--------------
User|[https://social.technet.microsoft.com/wiki/contents/articles/12037.active-directory-get-aduser-default-and-extended-properties.aspx](https://social.technet.microsoft.com/wiki/contents/articles/12037.active-directory-get-aduser-default-and-extended-properties.aspx)
Group|[https://social.technet.microsoft.com/wiki/contents/articles/12079.active-directory-get-adgroup-default-and-extended-properties.aspx](https://social.technet.microsoft.com/wiki/contents/articles/12079.active-directory-get-adgroup-default-and-extended-properties.aspx)
Org Unit|[https://social.technet.microsoft.com/wiki/contents/articles/12089.active-directory-get-adorganizationalunit-default-and-extended-properties.aspx](https://social.technet.microsoft.com/wiki/contents/articles/12089.active-directory-get-adorganizationalunit-default-and-extended-properties.aspx)
Computer|[https://social.technet.microsoft.com/wiki/contents/articles/12056.active-directory-get-adcomputer-default-and-extended-properties.aspx](https://social.technet.microsoft.com/wiki/contents/articles/12056.active-directory-get-adcomputer-default-and-extended-properties.aspx)
