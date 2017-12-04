# Overview
The ActiveDirectory Api (Myriad) provides a simple rest interface to the ActiveDirectory Handler to perform common actions and workflows against an Active Directory instance.  The AdApi execute calls synchronously.

# Url Structure
## Base Url
````
<protocol>://<host>:<port>/myriad/<object>/<identity>
````

The general format for the rest URL is above, using HTTP Verbs to indicate what action to perform.  While this format isn't absolute, and will be different for some actions (ex: AddToGroup requires a 2nd identity to identify the group), the general format applies to most rest calls.  The specific URL formats and HTTP Verbs for each action are below :

|Action|Verb|Url Format
|------|----|----------
|Get|GET|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/&lt;object&gt;/&lt;identity&gt;
|Create|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/&lt;object&gt;/&lt;identity&gt;
|Modify|PUT|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/&lt;object&gt;/&lt;identity&gt;
|Delete|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/&lt;object&gt;/&lt;identity&gt;
|AddToGroup|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/&lt;object&gt;/&lt;identity&gt;/&lt;groupIdentity&gt;
|RemoveFromGroup|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/&lt;object&gt;/&lt;identity&gt;&lt;groupIdentity&gt;
|AddAccessRule|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;type&gt;/&lt;rights&gt;
|RemoveAccessRule|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;type&gt;/&lt;rights&gt;
|SetAccessRule|PUT|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;type&gt;/&lt;rights&gt;
|PurgeAccessRules|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;
|Search|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/search
|[Search (Custom)](#custom-searches)|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/search/&lt;planname&gt;
|AddRole|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/role/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;role&gt;
|RemoveRole|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/myriad/role/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;role&gt;


## Query String

The query string is used to control how the output from the rest call will be formatted and what it will contain.  This maps directly to the [config](./handler/#config) section of the ActiveDirectory plan.  Below is a list of query paramters that can be applied to any request.

|Parameter|Type/Value|Default Value|Description
|---------|----------|-------------|-----------
|returngroupmembership|boolean|false|Returns a list of groups the User or Group is a member of.
|returnobjects|boolean|true|Returns the object along with the status of the action.
|returnobjectproperties|boolean|true|Returns the raw DirectoryEntry properties associated with the object.
|returnaccessrules|boolean|false|Returns all the access rules associated with the object.

<!--
|outputtype|"Json"<br>"Yaml"<br>"Xml"|Json|Specifies the output type of the adapter.
|prettyprint|boolean|false|Returns data with newlines and indentions to make more human readable.
-->

# Result Object / Error Messages

## General Structure

The AdApi returns a list of "Results".  Each result starts with a "Statuses" block, which contains information about the success or failure of the action requested.  

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 4,
                    "Message": "User [HonestPolitician] Not Found.",
                    "Action": 1
                }
            ],
            "Type": 1,
            "Identity": "HonestPolitician",
            "Object": null
        }
    ]
}
````

Statuses is an array of statuses because a single request could results in multiple action (Create a user and add him to 3 groups).  Inside each individual status is a status code, a message, and the action performed.  The numeric values are mapped below : 

**Status**

|Value|Description
|-----|-----------
|0|Unknown
|1|Success
|2|MissingInput
|3|AlreadyExists
|4|DoesNotExist
|5|PasswordPolicyNotMet
|6|InvalidPath
|7|NotSupported
|8|NotAllowed
|9|InvalidAttribute
|10|ConnectionError
|11|InvalidName
|12|InvalidContainer
|13|MultipleMatches


**Action**

|Value|Description
|-----|-----------
|0|None
|1|Get
|2|Create
|4|Modify
|8|Delete
|16|Rename (Not Yet Implemented)
|32|Move (Not Yet Implemented)
|64|AddToGroup
|128|RemoveFromGroup
|256|Search
|512|AddAccessRule
|1024|RemoveAccessRule
|2048|SetAccessRule
|4096|PurgeAccessRules
|8192|AddRole
|16384|RemoveRole

The rest of the request message tell information about the original request (Type and Identity) and then contains an "Object" field contains the output object or null if it wasn't requested.  The "Type" field indicates which type of object is being returned, which indicatest he type of object returned   The numberic "Type" field  and associated "Object Type" that is returned is mapped below :

**Type**

|Value|Description|Object Type Returned
|-----|-----------|--------------------
|0|None|None
|1|User|UserPrincipalObject
|2|Group|GroupPrincipalObject
|3|Computer (Not Yet Implemented)|????
|4|OrganizationalUnit|OrganizationalUnitObject
|5|GroupPolicy (Not Yet Implemented)|????
|6|Search|SearchResultsObject

## Object Field Structure

The object field varies both on the type of object returned, and the [query string](#query-string) flags that indicate what to return.  In fact, returning the object itself is even optional (see [returnobjects](#query-string) query variable).  The sections below illustrate the structure of the return objects.

### Common Sections

These are the parts of the "Object" property that all ActiveDirectory objects can return (if requested). 

#### Properties

**Applies To:** All Objects

In come cases, roperties can contain more than one value.  Thus, the properties are turned as an array of values with the property name being the "key".

````yaml
"Object": {

    <CLIPPED>

    "Properties": {
        "objectClass": [
            "top",
            "person",
            "organizationalPerson",
            "user"
        ],
        "cn": [
            "mfox"
        ],
        "sn": [
            "Fox"
        ],
        "c": [
            "us"
        ],
        "l": [
            "Houston"
        ],
        "st": [
            "Texas"
        ],
        "title": [
            "Mr. McFly"
        ],
        "description": [
            "Time Traveler"
        ],

    <CLIPPED>

    }
}
````

#### Groups (Group Membership)

**Applies To:** : Users and Groups

This section details all group a Principal is a member of.  It has the same return structure as the "GroupPrincipalObject" described below, without the "Groups" and "AccessRules" returned.

````yaml
"Object": {

    <CLIPPED>

    "Groups": [
        {
            "ContextType": "Domain",
            "Description": "All domain users",
            "DisplayName": null,
            "DistinguishedName": "CN=Domain Users,CN=Users,DC=sandbox,DC=local",
            "Guid": "b9c132e5-198a-4b08-b52f-6eef92445374",
            "Name": "Domain Users",
            "SamAccountName": "Domain Users",
            "Sid": "S-1-5-21-4054027134-3251639354-3875066094-513",
            "StructuralObjectClass": "group",
            "UserPrincipalName": null,
            "Groups": null,
            "AccessRules": null
        },
        {
            "ContextType": "Domain",
            "Description": "Dummy CoreServices Group",
            "DisplayName": "CoreServicesDisplayName",
            "DistinguishedName": "CN=CoreServices,OU=Synapse,DC=sandbox,DC=local",
            "Guid": "b7675317-8233-4b41-b324-fb848227eb03",
            "Name": "CoreServices",
            "SamAccountName": "CoreServicesSam",
            "Sid": "S-1-5-21-4054027134-3251639354-3875066094-2120",
            "StructuralObjectClass": "group",
            "UserPrincipalName": null,
            "Groups": null,
            "AccessRules": null
        }
    ],

    <CLIPPED>

    }
}
````

#### AccessRules

**Applies To** : Users, Groups and Organizational Units

The "AccessRules" section reports all Access Rules associated with an object.  These include both ones directly applied to the object, and those applied through inheritance.

````yaml
"Object": {

    <CLIPPED>

    "AccessRules": [
        {
            "ControlType": "Allow",
            "Rights": "GenericRead",
            "IdentityReference": "S-1-5-10",
            "IdentityName": "SELF",
            "InheritanceFlags": "None",
            "IsInherited": "false"
        },
        {
            "ControlType": "Allow",
            "Rights": "ReadControl",
            "IdentityReference": "S-1-5-11",
            "IdentityName": "Authenticated Users",
            "InheritanceFlags": "None",
            "IsInherited": "false"
        },
        {
            "ControlType": "Allow",
            "Rights": "GenericAll",
            "IdentityReference": "S-1-5-18",
            "IdentityName": "SYSTEM",
            "InheritanceFlags": "None",
            "IsInherited": "false"
        },
        {
            "ControlType": "Allow",
            "Rights": "GenericAll",
            "IdentityReference": "S-1-5-32-548",
            "IdentityName": "Account Operators",
            "InheritanceFlags": "None",
            "IsInherited": "false"
        },
        {
            "ControlType": "Allow",
            "Rights": "ReadProperty, WriteProperty",
            "IdentityReference": "S-1-5-10",
            "IdentityName": "SELF",
            "InheritanceFlags": "None",
            "IsInherited": "false"
        },
        {
            "ControlType": "Allow",
            "Rights": "ExtendedRight",
            "IdentityReference": "S-1-5-10",
            "IdentityName": "SELF",
            "InheritanceFlags": "None",
            "IsInherited": "false"
        },
        {
            "ControlType": "Allow",
            "Rights": "ReadProperty, WriteProperty",
            "IdentityReference": "S-1-5-10",
            "IdentityName": "SELF",
            "InheritanceFlags": "None",
            "IsInherited": "false"
        },
        {
            "ControlType": "Allow",
            "Rights": "CreateChild, Self, WriteProperty, ExtendedRight, Delete, GenericRead, WriteDacl, WriteOwner",
            "IdentityReference": "S-1-5-32-544",
            "IdentityName": "Administrators",
            "InheritanceFlags": "ContainerInherit",
            "IsInherited": "true"
        }
    ]

    <CLIPPED>

    }
}
````

### Users (UserPrincipalObject)

Below is an example of a UserPrincipalObject being returned in the Object field.  In this example, the Properties, Groups and AccessRules were not requested, and are thus "null".

````yaml
"Object": {
    "EmailAddress": "mfox@company.com",
    "EmployeeId": "42",
    "GivenName": "Michael",
    "MiddleName": null,
    "Surname": "Fox",
    "VoiceTelephoneNumber": "1-800-555-1212",
    "Properties": null,
    "AccountExpirationDate": "2017-10-10T16:35:46.0000000Z",
    "AccountLockoutTime": null,
    "AllowReversiblePasswordEncryption": "true",
    "BadLogonCount": "0",
    "DelegationPermitted": "true",
    "Enabled": "true",
    "HomeDirectory": "C:\\Temp",
    "HomeDrive": "F:",
    "LastBadPasswordAttempt": null,
    "LastLogon": null,
    "LastPasswordSet": "2017-10-03T18:11:40.9300291Z",
    "PasswordNeverExpires": "true",
    "PasswordNotRequired": "false",
    "PermittedLogonTimes": null,
    "ScriptPath": "C:\\Temp\\Scripts",
    "SmartcardLogonRequired": "true",
    "UserCannotChangePassword": "false",
    "ContextType": "Domain",
    "Description": "American Actor, Back to the Future.",
    "DisplayName": "Michael J. Fox",
    "DistinguishedName": "CN=mfox,OU=Synapse,DC=sandbox,DC=local",
    "Guid": "49517710-7432-4450-bd6a-cfd4d6b7e0f5",
    "Name": "mfox",
    "SamAccountName": "mfox",
    "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1834",
    "StructuralObjectClass": "user",
    "UserPrincipalName": "mfox@sandbox.local",
    "Groups": null,
    "AccessRules": null
}
````

### Groups (GroupPrincipalObject)

Below is an example of a GroupPrincipalObject being returned in the Object field.  The "Members" field is a list of current members (PrincipalObject class) that are in the group.  In this example, the Properties, Groups and AccessRules were not requested, and are thus "null".

````yaml
"Object": {
    "GroupScope": "Universal",
    "IsSecurityGroup": "true",
    "Members": [
        {
            "ContextType": "Domain",
            "Description": "The Greatest",
            "DisplayName": "Test User",
            "DistinguishedName": "CN=TestUser001,OU=Synapse,DC=sandbox,DC=local",
            "Guid": "521192c2-0659-4b47-8ca1-08063f5a1ddc",
            "Name": "TestUser001",
            "SamAccountName": "TestUser001",
            "Sid": "S-1-5-21-4054027134-3251639354-3875066094-2101",
            "StructuralObjectClass": "user",
            "UserPrincipalName": "TestUser001@sandbox.local",
            "Groups": null,
            "AccessRules": null
        }
    ],
    "Properties": null,
    "ContextType": "Domain",
    "Description": null,
    "DisplayName": null,
    "DistinguishedName": "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local",
    "Guid": "72e11fd9-10f4-4c27-acb4-08dd30c78b8f",
    "Name": "FamousActors",
    "SamAccountName": "FamousActors",
    "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1835",
    "StructuralObjectClass": "group",
    "UserPrincipalName": null,
    "Groups": null,
    "AccessRules": null
}
````

### Organizational Units (OrganizationalUnitObject)

Below is an example of a OrganizationalUnitObject being returned in the Object field.  In this example, the Properties, Groups and AccessRules were not requested, and are thus "null".

````yaml
"Object": {
    "DistinguishedName": "OU=DeleteMe,OU=Synapse,DC=sandbox,DC=local",
    "Guid": "fb93e890-6889-41b7-8bb2-54a90ed27ba2",
    "Name": "OU=DeleteMe",
    "NativeGuid": "90e893fb8968b7418bb254a90ed27ba2",
    "Parent": {
        "Guid": "4322bb5f-01d8-445c-86ea-b021fe21609b",
        "Name": "OU=Synapse",
        "NativeGuid": "5fbb2243d8015c4486eab021fe21609b",
        "Parent": null,
        "Path": "LDAP://OU=Synapse,DC=sandbox,DC=local",
        "Properties": null,
        "SchemaClassName": "organizationalUnit",
        "SchemaEntry": null,
        "UsePropertyCache": "true",
        "Username": null,
        "AccessRules": null
    },
    "Path": "LDAP://OU=DeleteMe,OU=Synapse,DC=sandbox,DC=local",
    "Properties": null,
    "SchemaClassName": "organizationalUnit",
    "SchemaEntry": {
        "Guid": "228d9a87-c302-11cf-9aa4-00aa004a5691",
        "Name": "organizationalUnit",
        "NativeGuid": "{228D9A87-C302-11CF-9AA4-00AA004A5691}",
        "Parent": {
            "Guid": "228d9a86-c302-11cf-9aa4-00aa004a5691",
            "Name": "schema",
            "NativeGuid": "{228D9A86-C302-11CF-9AA4-00AA004A5691}",
            "Parent": null,
            "Path": "LDAP://schema",
            "Properties": null,
            "SchemaClassName": "Schema",
            "SchemaEntry": null,
            "UsePropertyCache": "true",
            "Username": null,
            "AccessRules": null
        },
        "Path": "LDAP://schema/organizationalUnit",
        "Properties": null,
        "SchemaClassName": "Class",
        "SchemaEntry": null,
        "UsePropertyCache": "true",
        "Username": null,
        "AccessRules": null
    },
    "UsePropertyCache": "true",
    "Username": null,
    "AccessRules": null
}
````

### SearchResults (SearchResultsObject)

Below is an example of a SearchResultsObject being returned in the Object field.  The common objects "AccessRules" and "Groups" do not apply to this object type.   The example below returns all "Users" from the Base Search "OU=Synapse,DC=sandbox,DC=local".

Request Body: 

````yaml
{
  "Filter": "(objectClass=User)",
  "SearchBase": "ou=Synapse,dc=sandbox,dc=local",
  "ReturnAttributes": [
    "name",
    "objectGUID",
    "objectSid",
    "distinguishedName",
    "dSCorePropagationData"
  ]
}
````

Response:

````yaml
"Object": {
    "Results": [
        {
            "Path": "LDAP://CN=mfox,OU=Synapse,DC=sandbox,DC=local",
            "Properties": {
                "name": [
                    "mfox"
                ],
                "objectGUID": [
                    "49517710-7432-4450-bd6a-cfd4d6b7e0f5"
                ],
                "objectSid": [
                    "S-1-5-21-4054027134-3251639354-3875066094-1834"
                ],
                "distinguishedName": [
                    "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                ],
                "dSCorePropagationData": [
                    "11/13/2017 8:20:12 PM",
                    "11/13/2017 8:19:00 PM",
                    "11/13/2017 7:39:32 PM",
                    "11/13/2017 4:36:09 PM",
                    "7/14/1601 10:36:49 PM"
                ]
            }
        },
        {
            "Path": "LDAP://CN=TestUser001,OU=Synapse,DC=sandbox,DC=local",
            "Properties": {
                "name": [
                    "TestUser001"
                ],
                "objectGUID": [
                    "521192c2-0659-4b47-8ca1-08063f5a1ddc"
                ],
                "objectSid": [
                    "S-1-5-21-4054027134-3251639354-3875066094-2101"
                ],
                "distinguishedName": [
                    "CN=TestUser001,OU=Synapse,DC=sandbox,DC=local"
                ],
                "dSCorePropagationData": [
                    "11/14/2017 9:40:42 PM",
                    "11/13/2017 8:20:12 PM",
                    "11/13/2017 8:19:00 PM",
                    "11/13/2017 7:39:32 PM",
                    "7/14/1601 10:36:48 PM"
                ]
            }
        },
        {
            "Path": "LDAP://CN=SomeUser001,OU=Synapse,DC=sandbox,DC=local",
            "Properties": {
                "name": [
                    "SomeUser001"
                ],
                "objectGUID": [
                    "5bf5fb03-062a-4e23-93e9-9ae09d11870d"
                ],
                "objectSid": [
                    "S-1-5-21-4054027134-3251639354-3875066094-2130"
                ],
                "distinguishedName": [
                    "CN=SomeUser001,OU=Synapse,DC=sandbox,DC=local"
                ],
                "dSCorePropagationData": [
                    "11/14/2017 10:41:30 PM",
                    "11/14/2017 10:38:02 PM",
                    "11/13/2017 8:20:12 PM",
                    "11/13/2017 8:19:00 PM",
                    "7/14/1601 10:32:32 PM"
                ]
            }
        },
        {
            "Path": "LDAP://CN=CrappyUser001,OU=Synapse,DC=sandbox,DC=local",
            "Properties": {
                "name": [
                    "CrappyUser001"
                ],
                "objectGUID": [
                    "2ae96754-6856-4beb-a292-9ccb3315fcdd"
                ],
                "objectSid": [
                    "S-1-5-21-4054027134-3251639354-3875066094-2294"
                ],
                "distinguishedName": [
                    "CN=CrappyUser001,OU=Synapse,DC=sandbox,DC=local"
                ],
                "dSCorePropagationData": [
                    "1/1/1601 12:00:00 AM"
                ]
            }
        }
    ]
}
````


# Api Calls

The sections below show examples of each Api call available for every object type and action.  It also shows a sample body on requests that require one (POST or PUT).


## Create (HTTP POST) or Modify (HTTP PUT)

The format for the URL and message body are identical for a Create and a Modify action.  The only difference in the response would be in the "Statuses > Action" field, which would reflect either "Create" or "Modify" depending on which was called.

---
### Create/Modify User

**Request**

````
{{protocol}}://{{host}}:{{port}}/myriad/user/cn=mfox,ou=Synapse,dc=sandbox,dc=local

Body : 
{
  Password: "bi@02LL49_VWQ{b",
  GivenName: "Michael",
  Surname: "Fox",
  Description: "American Actor, Back to the Future.",
  UserPrincipalName: "mfox@sandbox.local",
  SamAccountName: "mfox",
  DisplayName: "Michael J. Fox",
  EmailAddress: "mfox@company.com",
  VoiceTelephoneNumber: "1-800-555-1212",
  EmployeeId: "42",
  Enabled: true,
  AccountExpirationDate: "2017-10-10T16:35:46.0417954Z",
  SmartcardLogonRequired: true,
  DelegationPermitted: true,
  HomeDirectory: "C:\\Temp",
  ScriptPath: "C:\\Temp\\Scripts",
  PasswordNotRequired: false,
  PasswordNeverExpires: true,
  UserCannotChangePassword: false,
  AllowReversiblePasswordEncryption: true,
  HomeDrive: "F",
  Properties: {
    "mobile": [ "713-867-5309" ],
    "otherMobile": [ "281-867-5309", "832-867-5309" ],
    "pager": [ "~null~" ]
}
````

---
### Create/Modify Group

**Request**

````
{{protocol}}://{{host}}:{{port}}/myriad/group/cn=FamousActors,ou=Synapse,dc=sandbox,dc=local

Body:
{
  Description: "Famous Actors and Actresses",
  Scope: "Universal",
  IsSecurityGroup: "true",
  ManagedBy: mfox,
  Properties: {
	"info": [ "Group of famous actors and actresses." ]
  }
}
````

---
### Create/Modify Organizational Unit

**Request**

````
{{protocol}}://{{host}}:{{port}}/myriad/ou/ou=AmericanActors,ou=Synapse,dc=sandbox,dc=local

Body:
{
  Description: "Hello World",
  ManagedBy: "mfox",
  Properties: {
  	"postalCode": [ "90210" ]
  }
}
````

---
## Get (HTTP GET) or Delete (HTTP DELETE)

The format for the URL is identical for a Get and a Delete action.  The only differences would be :

* On Delete, no object will be returned (kinda obvious).
* The "Statuses > Action" field would reflect either "Get" or "Delete", depending on which was called.

---
### Get/Delete User

**Requests**

````
{{protocol}}://{{host}}:{{port}}/myriad/user/mfox  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/user/CN=mfox,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/myriad/user/S-1-5-21-4054027134-3251639354-3875066094-1773  (By Sid)
{{protocol}}://{{host}}:{{port}}/myriad/user/1722b838-57e1-4058-a394-338882af9e2f  (By Guid)
{{protocol}}://{{host}}:{{port}}/myriad/user/mfox  (By SamAccountName)
{{protocol}}://{{host}}:{{port}}/myriad/user/mfox@sandbox.local  (By UserPrincipal)
````

---
### Get/Delete Group

**Request**

````
{{protocol}}://{{host}}:{{port}}/myriad/group/FamousActors  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/group/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local  (By DistinguishedName)
{{protocol}}://{{host}}:{{port}}/myriad/group/S-1-5-21-4054027134-3251639354-3875066094-1774  (By Sid)
{{protocol}}://{{host}}:{{port}}/myriad/group/19cdf305-c43b-497a-a932-6091f4a09dbb  (By Guid)
{{protocol}}://{{host}}:{{port}}/myriad/group/FamousActors  (By SamAccountName)
````

---
### Get/Delete Organizational Unit

**Request**

````
{{protocol}}://{{host}}:{{controllerPort}}/myriad/ou/AmericanActors  (By Name)
{{protocol}}://{{host}}:{{controllerPort}}/myriad/ou/OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local  (By DistinguishedName)
{{protocol}}://{{host}}:{{controllerPort}}/myriad/ou/95637ae6-9f24-420f-b573-7c2ab3496419  (By Guid)
````

---
## AddToGroup (HTTP POST) or RemoveFromGroup (HTTP DELETE)

The format for the URL is identical for an AddToGroup and RemoveFromGroup action.  The only real difference would be the status message returned, either "Added To" or "Removed From" depending on which action was called.

---
### Add/Remove User To Group

**Request**

````
{{protocol}}://{{host}}:{{controllerPort}}/myriad/user/mfox/FamousActors  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/user/CN=mfox,OU=Synapse,DC=sandbox,DC=local/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/myriad/user/S-1-5-21-4054027134-3251639354-3875066094-1773/S-1-5-21-4054027134-3251639354-3875066094-1774  (By Sid)
{{protocol}}://{{host}}:{{port}}/myriad/user/1722b838-57e1-4058-a394-338882af9e2f/19cdf305-c43b-497a-a932-6091f4a09dbb  (By Guid)
{{protocol}}://{{host}}:{{port}}/myriad/user/mfox/FamousActors  (By SamAccountName)
{{protocol}}://{{host}}:{{port}}/myriad/user/mfox@sandbox.local/FamousActors  (By UserPrincipal / Name)

* Note: There is not "UserPrincipal" for a group, so last example is using "Name" for the groupIdentity.
````

---
### Add/Remove Group to Group

The format for the URL is identical for an AddToGroup and RemoveFromGroup action.  The only real difference would be the status message returned, either "Added To" or "Removed From" depending on which action was called.

**Request**

````
{{protocol}}://{{host}}:{{controllerPort}}/myriad/group/FamousActors/AllActors  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/group/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local/CN=AllActors,OU=Synapse,DC=sandbox,DC=local (By DistinguishedName)
{{protocol}}://{{host}}:{{port}}/myriad/group/S-1-5-21-4054027134-3251639354-3875066094-1774/S-1-5-21-4054027134-3251639354-3875066094-1775  (By Sid)
{{protocol}}://{{host}}:{{port}}/myriad/group/19cdf305-c43b-497a-a932-6091f4a09dbb/cc8db84b-3aca-4e69-b1d1-6b9a3b30ee73  (By Guid)
{{protocol}}://{{host}}:{{port}}/myriad/group/FamousActors/AllActors  (By SamAccountName)
````

---
## AddAccessRule (HTTP POST), RemoveAccessRule (HTTP DELETE) or SetAccessRule (HTTP PUT)

The format for the URL is identical for the AddAccessRule, RemoveAccessRule and SetAccessRule actions.  The only differences would be :
* On Delete, no object will be returned (kinda obvious)
* The "Statuses > Action" field would reflect either "AddAccessRule", "RemoveAccessRule or "SetAccessRule", depending on which was called.  Valid values for permitted "rights" can be found [here](handler.md#activedirectoryrights-enumeration).

---
### AddAccessRule/RemoveAccessRule/SetAccessRule to User

**Requests**

````
{{protocol}}://{{host}}:{{port}}/myriad/accessrule/user/mfox/user001/Allow/GenericAll  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/user/CN=mfox,OU=Synapse,DC=sandbox,DC=local/CN=user001,OU=Synapse,DC=sandbox,DC=local/Allow/GenericAll  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/myriad/user/S-1-5-21-4054027134-3251639354-3875066094-1773/S-1-5-21-4054027134-3251639354-3875066094-1206/Allow/GenericAll  (By Sid)
{{protocol}}://{{host}}:{{port}}/myriad/user/1722b838-57e1-4058-a394-338882af9e2f/4db94271-1fde-402a-a8c5-1564dfd8d62b/Allow/GenericAll  (By Guid)
{{protocol}}://{{host}}:{{port}}/myriad/user/mfox/user001/Allow/GenericAll  (By SamAccountName)
{{protocol}}://{{host}}:{{port}}/myriad/user/mfox@sandbox.local/user001@sandbox.local/Allow/GenericAll  (By UserPrincipal)
````

---
### AddAccessRule/RemoveAccessRule/SetAccessRule to Group

**Requests**

````
{{protocol}}://{{host}}:{{port}}/myriad/accessrule/group/FamousActors/mfox/Allow/GenericAll  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/group/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local/CN=mfox,OU=Synapse,DC=sandbox,DC=local/Allow/GenericAll  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/myriad/group/S-1-5-21-4054027134-3251639354-3875066094-1835/S-1-5-21-4054027134-3251639354-3875066094-1773/Allow/GenericAll  (By Sid)
{{protocol}}://{{host}}:{{port}}/myriad/group/72e11fd9-10f4-4c27-acb4-08dd30c78b8f/1722b838-57e1-4058-a394-338882af9e2f/Allow/GenericAll  (By Guid)
{{protocol}}://{{host}}:{{port}}/myriad/group/FamousActors/mfox/Allow/GenericAll  (By SamAccountName)
````


---
### AddAccessRule/RemoveAccessRule/SetAccessRule to OrganizationalUnit

**Requests**

````
{{protocol}}://{{host}}:{{port}}/myriad/accessrule/ou/AmericanActors/FamousActors/Allow/GenericAll  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/ou/CN=AmericanActors,OU=Synapse,DC=sandbox,DC=local/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local/Allow/GenericAll  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/myriad/ou/768d8062-c1d2-4d67-9ad6-57c73cc3982b/72e11fd9-10f4-4c27-acb4-08dd30c78b8f/Allow/GenericAll  (By Guid)
````

---
## PurgeAccessRules (HTTP DELETE)

The format for the URL is identical for the AddAccessRule, RemoveAccessRule and SetAccessRule actions, except there is no need for a type or rights, since purge removes all rules (Allow and Deny) for the given principal (user or group).

---
### PurgeAccessRules on User

**Requests**

````
{{protocol}}://{{host}}:{{port}}/myriad/accessrule/user/mfox/user001  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/user/CN=mfox,OU=Synapse,DC=sandbox,DC=local/CN=user001,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/myriad/user/S-1-5-21-4054027134-3251639354-3875066094-1773/S-1-5-21-4054027134-3251639354-3875066094-1206  (By Sid)
{{protocol}}://{{host}}:{{port}}/myriad/user/1722b838-57e1-4058-a394-338882af9e2f/4db94271-1fde-402a-a8c5-1564dfd8d62b  (By Guid)
{{protocol}}://{{host}}:{{port}}/myriad/user/mfox/user001/Allow/GenericAll  (By SamAccountName)
{{protocol}}://{{host}}:{{port}}/myriad/user/mfox@sandbox.local/user001@sandbox.local  (By UserPrincipal)
````

---
### PurgeAccessRules on Group

**Requests**

````
{{protocol}}://{{host}}:{{port}}/myriad/accessrule/group/FamousActors/mfox/  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/group/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local/CN=mfox,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/myriad/group/S-1-5-21-4054027134-3251639354-3875066094-1835/S-1-5-21-4054027134-3251639354-3875066094-1773  (By Sid)
{{protocol}}://{{host}}:{{port}}/myriad/group/72e11fd9-10f4-4c27-acb4-08dd30c78b8f/1722b838-57e1-4058-a394-338882af9e2f  (By Guid)
{{protocol}}://{{host}}:{{port}}/myriad/group/FamousActors/mfox  (By SamAccountName)
````

---
### PurgeAccessRules on Organizational Unit

**Requests**

````
{{protocol}}://{{host}}:{{port}}/myriad/accessrule/group/AmericanActors/FamousActors/  (By Name)
{{protocol}}://{{host}}:{{port}}/myriad/group/CN=AmericanActors,OU=Synapse,DC=sandbox,DC=local/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/myriad/group/768d8062-c1d2-4d67-9ad6-57c73cc3982b/72e11fd9-10f4-4c27-acb4-08dd30c78b8f  (By Guid)
````

---
## Search

The "Search" action takes in a filter string, search base and a list of attributes to return, and returns the attributes for all DirectoryEntry objects that matches the filter string, assuming the requesting user has the rights to see that object in the first place.

**Requests**

````
{{protocol}}://{{host}}:{{port}}/myriad/search

Body:
{
  "Filter": "(objectClass=User)",
  "SearchBase": "ou=Synapse,dc=sandbox,dc=local",
  "ReturnAttributes": [
    "name",
    "objectGUID",
    "objectSid",
    "distinguishedName",
    "dSCorePropagationData"
  ]
}
````

---
### Custom Searches

The ActiveDirectory Api allows for you to write a "custom search" as a plan and call it from the Api.  This can be used for common AD queries or reporting where the requestor might not necessarily want or need to know the raw Search Filter String.   The example below "GetAllGroups" returns all groups a principal (user or group) is a member of, either directly or through inheritance.  This plan is included with the set of standard plans as an example.

Note: For custom searches, the EXACT name of the dynamic parameter ("distinguishedname" in this example) must be passed in on the body of the message.

#### Plan

````yaml
Name: SearchGetAllGroups
Description: Get All Groups For A Security Principal
IsActive: true
Actions:
- Name: GetAllGroups
  Handler:
    Type: Synapse.Handlers.ActiveDirectory:ActiveDirectoryHandler
    Config:
      Type: Yaml
      Values:
        Action: Search
        RunSequential: false
        ReturnObjects: true
        OutputType: Yaml
        PrettyPrint: true
        SuppressOutput: false
  Parameters:
    Type: Yaml
    Values:
      SearchRequests:
      - Filter: "(|(member:1.2.840.113556.1.4.1941:=~~distinguishedname~~)(distinguishedName=~~distinguishedname~~))"
        Parameters:
        - Find: ~~distinguishedname~~
          ReplaceWith: xxxxxxxx
        ReturnAttributes: 
        - name
        - displayName
        - userPrincipalName
        - sAMAccountName
        - objectGUID
        - objectSid
    Dynamic:
    - Name: distinguishedname
      Path: SearchRequests[0]:Parameters[0]:ReplaceWith
````

#### Request

````
{{protocol}}://{{host}}:{{port}}/myriad/search/GetAllGroups

Body:
{
  "distinguishedname": "cn=TestUser001,ou=Synapse,dc=sandbox,dc=local"
}
````

---
## Roles

Roles are a way to assign a group of permissions to a User / Group that enforce what actions the user is allowed to call using the API.  This is accomplished by using a "RoleManager" class that implements the IRoleManager interface.  

The details of how this is accomplished is detailed in the RoleManager implementation itself, but at a very high level, a "role" should represent a set of "actions" that can be assigned to a "principal" (user or group) on a given object or set of objects.

---
### AddRole (HTTP POST) or RemoveRole (HTTP DELETE) To User

#### Requests

````
{{protocol}}://{{host}}:{{controllerPort}}/myriad/role/user/<identity>/<principal>/<role>
````

---
### AddRole (HTTP POST) or RemoveRole (HTTP DELETE) To Group

#### Requests

````
{{protocol}}://{{host}}:{{controllerPort}}/myriad/role/group/<identity>/<principal>/<role>
````

---
### AddRole (HTTP POST) or RemoveRole (HTTP DELETE) To OrganizationalUnit

#### Requests

````
{{protocol}}://{{host}}:{{controllerPort}}/myriad/role/ou/<identity>/<principal>/<role>
````

