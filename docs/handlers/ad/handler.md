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

(1) - Assumes only a single name exists.  Since multiple objects are allowed to have the same name under different organizational units, when multiple objects exist with the same name, an error will occur.

## Actions

Below is a list of supported actions that can be performed on ActiveDirectory objects.  The action will execute against the objects in a certain order, depending on the action.  For example, Organizational Units will be created before users and groups that might be members of that Organizational Unit.

|**Action**|Description|Order of Operations
|----------|-----------|-------------------
|**Query**|Retrieves a single ActiveDirectory object by its [identity](#activedirectory-objects-and-identities).|Users<br>Groups<br>Organizational Units
|**Create**|Creates a single ActiveDirectory object by its distinguished name only. (2)|Organizational Units<br>Groups<br>Users
|**Modify**|Modifies a single ActiveDirectory object by its [identity](#activedirectory-objects-and-identities). (2)|Organizational Units<br>Groups<br>Users
|**Delete**|Deletes a single ActiveDirectory object by its [identity](#activedirectory-objects-and-identities)|Users<br>Groups<br>Organizational Units
|**AddToGroup**|Adds Users and/or Groups to an existing ActiveDirecory group by its [identity](#activedirectory-objects-and-identities).|Users<br>Groups
|**RemoveFromGroup**|Removes Users and/or Groups from an existing ActiveDirectory group by its [identity](#activedirectory-objects-and-identities).|Users<br>Groups

(2) - When the "UseUpsert" config is set, calling "Create" on an existing object can be done using its [identity](#activedirectory-objects-and-identities).  Calling "Modify" on an object that does not exist will only create the object if called using a distinguished name.

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
        QueryGroupMembership: false
        ReturnObjects: true
        SuppressOutput: false
        UseUpsert: true
        OutputType: Yaml
        PrettyPrint: true
````

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Action|"Query"<br>"Create"<br>"Modify"<br>"Delete"<br>"AddToGroup"<br>"RemoveFromGroup"|Yes|Specifies the action that will be executed against the active directory instance.
|RunSequential|boolean|No|Tells the adapter how to process the objects in the plan.  If set to "true", objects will be acted upon in the order they appear in the plan.  (Defualt = "false")
|QueryGroupMembership|boolean|No|Tells the adapter to return a list of groups that the User or Group is a member of.  (Default = "true")
|ReturnObjects|boolean|No|Tells the adapter to return the ActiveDirectory object along with the status of the action.  (Default = "true")
|SuppressOutput|boolean|No|Tells the adapter to not output the ExitData returned from the action into the logs.  (Default = "false")
|UseUpsert|boolean|No|Tells the adapter to perform a "Modify" action when a "Create" action is called and the object exists, and to perform a "Create" action when a "Modify" action is called and the object does not exist.  (Default = "true")
|OutputType|"Json"<br>"Xml"<br>"Yaml"|No|Tells the adapter how to format the returned status from the action.  (Default = "Json")
|PrettyPrint|boolean|No|Tells the adapter whether to indent and add newlines to the returned output.  (Default = "true")


## Parameters

The parameters section of the plan is simply a list of each type of object you wish to perform the action on.  Every object must contain an [identity](#activedirectory-objects-and-identities), but depending on the action, other fields might be required.  Below are examples of each action and object type with the required fields.

---
### Action: Query and Delete, Object Type: All
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

````

For Query and Delete actions, the only field required for every ActiveDirectory object is its identity.  The example above shows each object type with every possible way it can be queried or deleted.

---
### Action: AddToGroup and RemoveFromGroup, Object Type: Users and Groups
````yaml
  Parameters:
    Type: Yaml
    Values:
      Users:
      - Identity: user1                                            # Any Valid User Identity
        Groups:
        - MyGroup001                                               # Any Valid Group Identity
        - cn=MyGroup002,ou=myorgunit,dc=sandbox,dc=local           # Any Valid Group Identity
      Groups:
      - Identity: MyGroup001                                       # Any Valid Group Identity
        Groups:
        - S-1-5-21-4054027134-3251639354-3875066094-1720           # Any Valid Group Identity
````

For Group Membership actions (AddToGroup and RemoveFromGroup), a "Groups" section is added below the Identity of each ActiveDirectory object.  The "Groups" section is a list of valid group [identities](#activedirectory-objects-and-identities) that specify which group the "Add" or "Remove" action it to be performed on.

---
### Action: Create or Modify, Object Type: Users
````yaml
  Parameters:
    Type: Yaml
    Values:
      Users:
      - Identity: cn=mfox,ou=MyOrgUnit,DC=sandbox,DC=local
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

Values under the "Properties" section are dynamic.  Any "attribute" that ActiveDirectory accepts for this type of object can be included here, in a "Key" and "Values" arrangement.  Most settable "attributes" for a User can be found on the microsoft site [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms677980(v=vs.85).aspx).  The "key" should represent the "Ldap-Display-Name" value of each attribute.   Here are the settable properties that apply to Users at the time of this documents writing :

|Property|Description
|--------|-----------
|c|Country/Region (ISO-3166 2-Letter Code)
|co|Country (Human Readable) 
|company|Company 
|countryCode|Country Code (ISO-3166 3-Digit Code)
|department|Department 
|facsimileTelephoneNumber|Fax Number
|homePhone|Home Phone
|info|Notes
|initials|User Initials
|ipPhone|IP Phone
|l|City 
|lockoutTime|Date and Time Account was locked out (Set To "0" For Unlock)
|logonWorkstation|Log On To
|manager|Manager Name (Distinguished Name) 
|mobile|Mobile
|otherFacsimileTelephoneNumber|Other Fax Numbers
|otherHomePhone|Other Home Phones
|otherIpPhone|Other IP Phones
|otherMobile|Other Mobilesa
|otherPager|Other Pagers
|otherTelephone|Other Telephone Numbers
|pager|Pager
|physicalDeliveryOfficeName|Office
|postalCode|Zip/Postal Code 
|postOfficeBox|Post Office Box 
|profilePath|Profile Path
|st|State/Province
|streetAddress|Street 
|title|Job Title 
|url|Other Web Pages
|userWorkstations|Log On To
|wWWHomePage|WWw Home Page

---
### Action: Create or Modify, Object Type: Groups
````yaml
  Parameters:
    Type: Yaml
    Values:
      Groups:
      - Identity: cn=MyNewGroup,ou=Synapse,DC=sandbox,DC=local
        Scope: Universal     # Local, Global or Universal
        IsSecurityGroup: true
        SamAccountName: trumd2
        Description: Some Lame Description
        Properties:
          managedBy:
          - cn=mfox,ou=MyOrgUnit,DC=sandbox,DC=local
          mail:
          - ~null~
          info:
          - Some Random Notes Here
````
**Default Values For Fields**

All the fields above are optional with the exception of "Identity" (which must be a Distinguished Name for "Create" actions).  Below is a list of fields that will be defaulted to values if not explicitly included on an object creation.

* SamAccountName: Defaults to "Name" if not provided on Create.

**Properties**

Values under the "Properties" section are dynamic.  Any "attribute" that ActiveDirectory accepts for this type of object can be included here, in a "Key" and "Values" arrangement.  Most settable "attributes" for a Group can be found on the microsoft site [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms677980(v=vs.85).aspx).  The "key" should represent the "Ldap-Display-Name" value of each attribute.   Here are the settable properties that apply to Groups at the time of this documents writing :

|Property|Description
|--------|-----------
|displayName|Display Name
|info|Comments
|mail|E-mail
|managedBy|Managed By User (Distinguished Name)

---
### Action: Create or Modify, Object Type: Organizational Units
````yaml
  Parameters:
    Type: Yaml
    Values:
      OrganizationalUnits:
      - Identity: ou=MyOrgUnit,dc=sandbox,dc=local
        Description: Some Lame Description
        Properties:
          managedBy:
          - cn=mfox,ou=MyOrgUnit,DC=sandbox,DC=local
          c:
          - US
          l:
          - ~null~
````
**Properties**

Values under the "Properties" section are dynamic.  Any "attribute" that ActiveDirectory accepts for this type of object can be included here, in a "Key" and "Values" arrangement.  Most settable "attributes" for an Organizational Unit can be found on the microsoft site [here](https://msdn.microsoft.com/en-us/library/windows/desktop/ms677623(v=vs.85).aspx).  The "key" should represent the "Ldap-Display-Name" value of each attribute.   Here are the settable properties that apply to Organizational Units at the time of this documents writing :

|Property|Description
|--------|-----------
|countryCode|Country Code (ISO-3166 3-Digit Code)
|co|Country (Human Readable) 
|c|Country/Region (ISO-3166 2-Letter Code)
|l|City 
|managedBy|Managed By User (Distinguished Name)
|postalCode|Zip/Postal Code 
|street|Street 
|st|State/Province

# Important Notes

## Modify : Clear Out A Field (~null~)

The action "Modify" only changes the fields you include in the plan.  You are not required to set every single field you want to keep on a modify.   However, this does beg the question, "How do I 'clear out' a field?".  This is done by placing the value "**~null~**" in the field.  For properties, if any of the possible values is "**~null~**", then it will clear out all values, ignoring any other values specififed in the plan.

