# Overview
The ActiveDirectory Api (AdApi) provides a simple rest interface to the ActiveDirectory Handler to perform common actions and workflows against an Active Directory instance.  The AdApi execute calls synchronously.

# Url Structure
## Base Url
````
<protocol>://<host>:<port>/ad/<object>/<identity>
````

The general format for the rest URL is above, using HTTP Verbs to indicate what action to perform.  While this format isn't absolute, and will be different for some actions (ex: AddToGroup requires a 2nd identity to identify the group), the general format applies to most rest calls.  The specific URL formats and HTTP Verbs for each action are below :

|Action|Verb|Url Format
|------|----|----------
|Query|GET|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;
|Create|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;
|Modify|PUT|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;
|Delete|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;
|AddToGroup|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;/&lt;groupIdentity&gt;
|RemoveFromGroup|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;&lt;groupIdentity&gt;
|AddAccessRule|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;type&gt;/&lt;rights&gt;
|RemoveAccessRule|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;type&gt;/&lt;rights&gt;
|SetAccessRule|PUT|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;type&gt;/&lt;rights&gt;
|PurgeAccessRules|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;


## Query String

The query string is used to control how the output from the rest call will be formatted and what it will contain.  This maps directly to the [config](./handler/#config) section of the ActiveDirectory plan.  Below is a list of query paramters that can be applied to any request.

|Parameter|Type/Value|Default Value|Description
|---------|----------|-------------|-----------
|querygroupmembership|boolean|false|Returns a list of groups the User or Group is a member of.
|returnobjects|boolean|true|Returns the object along with the status of the action.
|returnobjectproperties|boolean|true|Returns the raw DirectoryEntry properties associated with the object.
|returnaccessrules|boolean|false|Returns all the access rules associated with the object.

<!--
|outputtype|"Json"<br>"Yaml"<br>"Xml"|Json|Specifies the output type of the adapter.
|prettyprint|boolean|false|Returns data with newlines and indentions to make more human readable.
-->

# Result Object / Error Messages

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
            "User": null,
            "Group": null,
            "OrganizationalUnit": null
        }
    ]
}
````

The AdApi returns a list of "Results".  Each result starts with a "Statuses" block, which contains information about the success or failure of the action requested.  Statuses is an array of statuses because a single request could results in multiple action (Create a user and add him to 3 groups).  Inside each individual status is a status code, a message, and the action performed.  The numeric values are mapped below : 

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
|1|Query
|2|Create
|3|Modify
|4|Delete
|5|AddToGroup
|6|RemoveFromGroup


The rest of the request message tell information about the original request (Type and Identity) and then contains a list of each ActiveDirectory object that could be returned.  The "Type" field indicates which type of object is being returned, so all other AD Object lists should be null.   The numberic "Type" field is mapped below :

**Type**

|Value|Description
|-----|-----------
|0|None
|1|User
|2|Group
|3|Computer (Not Yet Implemented)
|4|OrganizationalUnit
|5|GroupPolicy (Not Yet Implemented)


# Api Calls

The sections below will show examples of each Api call available for every object type and action.  Each assumes no query string parameters are passed in (using default config files).  It also assumes an ActiveDirectory of "**sandbox.local**"

## Create (HTTP POST) or Modify (HTTP PUT)

The format for the URL and message body are identical for a Create and a Modify action.  The only difference in the response would be in the "Statuses > Action" field, which would reflect either "Create" or "Modify" depending on which was called.

---
### Create/Modify User

**Request**

````
{{protocol}}://{{host}}:{{port}}/ad/user/cn=mfox,ou=Synapse,dc=sandbox,dc=local

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
  HomeDrive: "F:"
}
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 2
                }
            ],
            "Type": 1,
            "Identity": "cn=mfox,ou=Synapse,dc=sandbox,dc=local",
            "User": {
                "EmailAddress": "mfox@company.com",
                "EmployeeId": "42",
                "GivenName": "Michael",
                "MiddleName": null,
                "Surname": "Fox",
                "VoiceTelephoneNumber": "1-800-555-1212",
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
                    "description": [
                        "American Actor, Back to the Future."
                    ],
                    "telephoneNumber": [
                        "1-800-555-1212"
                    ],
                    "givenName": [
                        "Michael"
                    ],
                    "distinguishedName": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "9/15/2017 7:00:18 PM"
                    ],
                    "whenChanged": [
                        "9/15/2017 7:00:18 PM"
                    ],
                    "displayName": [
                        "Michael J. Fox"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "mfox"
                    ],
                    "objectGUID": [
                        "1722b838-57e1-4058-a394-338882af9e2f"
                    ],
                    "userAccountControl": [
                        "328320"
                    ],
                    "badPwdCount": [
                        "0"
                    ],
                    "codePage": [
                        "0"
                    ],
                    "countryCode": [
                        "0"
                    ],
                    "employeeID": [
                        "42"
                    ],
                    "homeDirectory": [
                        "C:\\Temp"
                    ],
                    "homeDrive": [
                        "F:"
                    ],
                    "badPasswordTime": [],
                    "lastLogoff": [],
                    "lastLogon": [],
                    "scriptPath": [
                        "C:\\Temp\\Scripts"
                    ],
                    "pwdLastSet": [],
                    "primaryGroupID": [
                        "513"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1773"
                    ],
                    "accountExpires": [],
                    "logonCount": [
                        "0"
                    ],
                    "sAMAccountName": [
                        "mfox"
                    ],
                    "sAMAccountType": [
                        "805306368"
                    ],
                    "userPrincipalName": [
                        "mfox@sandbox.local"
                    ],
                    "objectCategory": [
                        "CN=Person,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "1/1/1601 12:00:00 AM"
                    ],
                    "mail": [
                        "mfox@company.com"
                    ]
                },
                "AccountExpirationDate": "2017-10-10T11:35:46-05:00",
                "AccountLockoutTime": null,
                "AllowReversiblePasswordEncryption": true,
                "BadLogonCount": 0,
                "DelegationPermitted": true,
                "Enabled": true,
                "HomeDirectory": "C:\\Temp",
                "HomeDrive": "F:",
                "LastBadPasswordAttempt": null,
                "LastLogon": null,
                "LastPasswordSet": "2017-09-15T14:00:18.1021386-05:00",
                "PasswordNeverExpires": true,
                "PasswordNotRequired": false,
                "PermittedLogonTimes": null,
                "ScriptPath": "C:\\Temp\\Scripts",
                "SmartcardLogonRequired": true,
                "UserCannotChangePassword": false,
                "ContextType": 1,
                "Description": "American Actor, Back to the Future.",
                "DisplayName": "Michael J. Fox",
                "DistinguishedName": "CN=mfox,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "1722b838-57e1-4058-a394-338882af9e2f",
                "Name": "mfox",
                "SamAccountName": "mfox",
                "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1773",
                "StructuralObjectClass": "user",
                "UserPrincipalName": "mfox@sandbox.local",
                "Groups": [
                    {
                        "ContextType": 1,
                        "Description": "All domain users",
                        "DisplayName": null,
                        "DistinguishedName": "CN=Domain Users,CN=Users,DC=sandbox,DC=local",
                        "Guid": "b9c132e5-198a-4b08-b52f-6eef92445374",
                        "Name": "Domain Users",
                        "SamAccountName": "Domain Users",
                        "Sid": "S-1-5-21-4054027134-3251639354-3875066094-513",
                        "StructuralObjectClass": "group",
                        "UserPrincipalName": null,
                        "Groups": []
                    }
                ]
            },
            "Group": null,
            "OrganizationalUnit": null
        }
    ]
}
````

---
### Create/Modify Group

**Request**

````
{{protocol}}://{{host}}:{{port}}/ad/group/cn=FamousActors,ou=Synapse,dc=sandbox,dc=local
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

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 2
                }
            ],
            "Type": 2,
            "Identity": "cn=FamousActors,ou=Synapse,dc=sandbox,dc=local",
            "User": null,
            "Group": {
                "GroupScope": 2,
                "IsSecurityGroup": true,
                "Members": null,
                "Properties": {
                    "objectClass": [
                        "top",
                        "group"
                    ],
                    "cn": [
                        "FamousActors"
                    ],
                    "description": [
                        "Famous Actors and Actresses"
                    ],
                    "distinguishedName": [
                        "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "9/15/2017 7:11:21 PM"
                    ],
                    "whenChanged": [
                        "9/15/2017 7:11:21 PM"
                    ],
                    "uSNCreated": [],
                    "info": [
                        "Group of famous actors and actresses."
                    ],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "FamousActors"
                    ],
                    "objectGUID": [
                        "19cdf305-c43b-497a-a932-6091f4a09dbb"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1774"
                    ],
                    "sAMAccountName": [
                        "FamousActors"
                    ],
                    "sAMAccountType": [
                        "268435456"
                    ],
                    "managedBy": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "groupType": [
                        "-2147483640"
                    ],
                    "objectCategory": [
                        "CN=Group,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "1/1/1601 12:00:00 AM"
                    ]
                },
                "ContextType": 1,
                "Description": "Famous Actors and Actresses",
                "DisplayName": null,
                "DistinguishedName": "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "19cdf305-c43b-497a-a932-6091f4a09dbb",
                "Name": "FamousActors",
                "SamAccountName": "FamousActors",
                "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1774",
                "StructuralObjectClass": "group",
                "UserPrincipalName": null,
                "Groups": []
            },
            "OrganizationalUnit": null
        }
    ]
}
````

---
### Create/Modify Organizational Unit

**Request**

````
{{protocol}}://{{host}}:{{port}}/ad/ou/ou=AmericanActors,ou=Synapse,dc=sandbox,dc=local

Body:
{
  Description: "Hello World",
  ManagedBy: "mfox",
  Properties: {
  	"postalCode": [ "90210" ]
  }
}
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 2
                }
            ],
            "Type": 4,
            "Identity": "ou=AmericanActors,ou=Synapse,dc=sandbox,dc=local",
            "User": null,
            "Group": null,
            "OrganizationalUnit": {
                "DistinguishedName": "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "95637ae6-9f24-420f-b573-7c2ab3496419",
                "Name": "OU=AmericanActors",
                "NativeGuid": "e67a6395249f0f42b5737c2ab3496419",
                "Parent": {
                    "Guid": "4322bb5f-01d8-445c-86ea-b021fe21609b",
                    "Name": "OU=Synapse",
                    "NativeGuid": "5fbb2243d8015c4486eab021fe21609b",
                    "Parent": null,
                    "Path": "LDAP://OU=Synapse,DC=sandbox,DC=local",
                    "Properties": {
                        "objectClass": [
                            "top",
                            "organizationalUnit"
                        ],
                        "ou": [
                            "Synapse"
                        ],
                        "description": [
                            "Modified By Guid"
                        ],
                        "distinguishedName": [
                            "OU=Synapse,DC=sandbox,DC=local"
                        ],
                        "instanceType": [
                            "4"
                        ],
                        "whenCreated": [
                            "6/19/2017 9:22:59 PM"
                        ],
                        "whenChanged": [
                            "8/28/2017 7:17:04 PM"
                        ],
                        "uSNCreated": [],
                        "uSNChanged": [],
                        "nTSecurityDescriptor": [],
                        "name": [
                            "Synapse"
                        ],
                        "objectGUID": [
                            "4322bb5f-01d8-445c-86ea-b021fe21609b"
                        ],
                        "objectCategory": [
                            "CN=Organizational-Unit,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                        ],
                        "dSCorePropagationData": [
                            "7/25/2017 3:33:45 PM",
                            "7/12/2017 4:07:35 PM",
                            "7/12/2017 4:05:57 PM",
                            "7/3/2017 3:03:47 PM",
                            "1/1/1601 12:00:00 AM"
                        ]
                    },
                    "SchemaClassName": "organizationalUnit",
                    "SchemaEntry": null,
                    "UsePropertyCache": true,
                    "Username": null
                },
                "Path": "LDAP://OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local",
                "Properties": {
                    "objectClass": [
                        "top",
                        "organizationalUnit"
                    ],
                    "ou": [
                        "AmericanActors"
                    ],
                    "description": [
                        "Hello World"
                    ],
                    "postalCode": [
                        "90210"
                    ],
                    "distinguishedName": [
                        "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "9/15/2017 7:19:05 PM"
                    ],
                    "whenChanged": [
                        "9/15/2017 7:19:05 PM"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "AmericanActors"
                    ],
                    "objectGUID": [
                        "95637ae6-9f24-420f-b573-7c2ab3496419"
                    ],
                    "managedBy": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "objectCategory": [
                        "CN=Organizational-Unit,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "1/1/1601 12:00:00 AM"
                    ]
                },
                "SchemaClassName": "organizationalUnit",
                "SchemaEntry": {
                    "Guid": "228d9a87-c302-11cf-9aa4-00aa004a5691",
                    "Name": "organizationalUnit",
                    "NativeGuid": "{228D9A87-C302-11CF-9AA4-00AA004A5691}",
                    "Parent": null,
                    "Path": "LDAP://schema/organizationalUnit",
                    "Properties": null,
                    "SchemaClassName": "Class",
                    "SchemaEntry": null,
                    "UsePropertyCache": true,
                    "Username": null
                },
                "UsePropertyCache": true,
                "Username": null
            }
        }
    ]
}
````

## Query (HTTP GET) or Delete (HTTP DELETE)

The format for the URL is identical for a Query and a Delete action.  The only differences would be :
* On Delete, no object will be returned (kinda obvious)
* The "Statuses > Action" field would reflect either "Query" or "Delete", depending on which was called.

---
### Query/Delete User

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/user/mfox  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/user/CN=mfox,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/user/S-1-5-21-4054027134-3251639354-3875066094-1773  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/user/1722b838-57e1-4058-a394-338882af9e2f  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox  (By SamAccountName)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox@sandbox.local  (By UserPrincipal)
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 1
                }
            ],
            "Type": 1,
            "Identity": "mfox",
            "User": {
                "EmailAddress": "mfox@company.com",
                "EmployeeId": "42",
                "GivenName": "Michael",
                "MiddleName": null,
                "Surname": "Fox",
                "VoiceTelephoneNumber": "1-800-555-1212",
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
                    "description": [
                        "American Actor, Back to the Future."
                    ],
                    "telephoneNumber": [
                        "1-800-555-1212"
                    ],
                    "givenName": [
                        "Michael"
                    ],
                    "distinguishedName": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "9/15/2017 7:00:18 PM"
                    ],
                    "whenChanged": [
                        "9/15/2017 7:00:18 PM"
                    ],
                    "displayName": [
                        "Michael J. Fox"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "mfox"
                    ],
                    "objectGUID": [
                        "1722b838-57e1-4058-a394-338882af9e2f"
                    ],
                    "userAccountControl": [
                        "328320"
                    ],
                    "badPwdCount": [
                        "0"
                    ],
                    "codePage": [
                        "0"
                    ],
                    "countryCode": [
                        "0"
                    ],
                    "employeeID": [
                        "42"
                    ],
                    "homeDirectory": [
                        "C:\\Temp"
                    ],
                    "homeDrive": [
                        "F:"
                    ],
                    "badPasswordTime": [],
                    "lastLogoff": [],
                    "lastLogon": [],
                    "scriptPath": [
                        "C:\\Temp\\Scripts"
                    ],
                    "pwdLastSet": [],
                    "primaryGroupID": [
                        "513"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1773"
                    ],
                    "accountExpires": [],
                    "logonCount": [
                        "0"
                    ],
                    "sAMAccountName": [
                        "mfox"
                    ],
                    "sAMAccountType": [
                        "805306368"
                    ],
                    "managedObjects": [
                        "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local",
                        "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "userPrincipalName": [
                        "mfox@sandbox.local"
                    ],
                    "objectCategory": [
                        "CN=Person,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "1/1/1601 12:00:00 AM"
                    ],
                    "mail": [
                        "mfox@company.com"
                    ]
                },
                "AccountExpirationDate": "2017-10-10T11:35:46-05:00",
                "AccountLockoutTime": null,
                "AllowReversiblePasswordEncryption": true,
                "BadLogonCount": 0,
                "DelegationPermitted": true,
                "Enabled": true,
                "HomeDirectory": "C:\\Temp",
                "HomeDrive": "F:",
                "LastBadPasswordAttempt": null,
                "LastLogon": null,
                "LastPasswordSet": "2017-09-15T14:00:18.1021386-05:00",
                "PasswordNeverExpires": true,
                "PasswordNotRequired": false,
                "PermittedLogonTimes": null,
                "ScriptPath": "C:\\Temp\\Scripts",
                "SmartcardLogonRequired": true,
                "UserCannotChangePassword": false,
                "ContextType": 1,
                "Description": "American Actor, Back to the Future.",
                "DisplayName": "Michael J. Fox",
                "DistinguishedName": "CN=mfox,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "1722b838-57e1-4058-a394-338882af9e2f",
                "Name": "mfox",
                "SamAccountName": "mfox",
                "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1773",
                "StructuralObjectClass": "user",
                "UserPrincipalName": "mfox@sandbox.local",
                "Groups": []
            },
            "Group": null,
            "OrganizationalUnit": null
        }
    ]
}
````

---
### Query/Delete Group

**Request**

````
{{protocol}}://{{host}}:{{port}}/ad/group/FamousActors  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/group/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local  (By DistinguishedName)
{{protocol}}://{{host}}:{{port}}/ad/group/S-1-5-21-4054027134-3251639354-3875066094-1774  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/group/19cdf305-c43b-497a-a932-6091f4a09dbb  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/group/FamousActors  (By SamAccountName)
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 1
                }
            ],
            "Type": 2,
            "Identity": "FamousActors",
            "User": null,
            "Group": {
                "GroupScope": 2,
                "IsSecurityGroup": true,
                "Members": null,
                "Properties": {
                    "objectClass": [
                        "top",
                        "group"
                    ],
                    "cn": [
                        "FamousActors"
                    ],
                    "description": [
                        "Famous Actors and Actresses"
                    ],
                    "distinguishedName": [
                        "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "9/15/2017 7:11:21 PM"
                    ],
                    "whenChanged": [
                        "9/15/2017 7:11:21 PM"
                    ],
                    "uSNCreated": [],
                    "info": [
                        "Group of famous actors and actresses."
                    ],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "FamousActors"
                    ],
                    "objectGUID": [
                        "19cdf305-c43b-497a-a932-6091f4a09dbb"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1774"
                    ],
                    "sAMAccountName": [
                        "FamousActors"
                    ],
                    "sAMAccountType": [
                        "268435456"
                    ],
                    "managedBy": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "groupType": [
                        "-2147483640"
                    ],
                    "objectCategory": [
                        "CN=Group,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "1/1/1601 12:00:00 AM"
                    ]
                },
                "ContextType": 1,
                "Description": "Famous Actors and Actresses",
                "DisplayName": null,
                "DistinguishedName": "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "19cdf305-c43b-497a-a932-6091f4a09dbb",
                "Name": "FamousActors",
                "SamAccountName": "FamousActors",
                "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1774",
                "StructuralObjectClass": "group",
                "UserPrincipalName": null,
                "Groups": []
            },
            "OrganizationalUnit": null
        }
    ]
}
````

---
### Query/Delete Organizational Unit

**Request**

````
{{protocol}}://{{host}}:{{controllerPort}}/ad/ou/AmericanActors  (By Name)
{{protocol}}://{{host}}:{{controllerPort}}/ad/ou/OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local  (By DistinguishedName)
{{protocol}}://{{host}}:{{controllerPort}}/ad/ou/95637ae6-9f24-420f-b573-7c2ab3496419  (By Guid)
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 1
                }
            ],
            "Type": 4,
            "Identity": "AmericanActors",
            "User": null,
            "Group": null,
            "OrganizationalUnit": {
                "DistinguishedName": "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "95637ae6-9f24-420f-b573-7c2ab3496419",
                "Name": "OU=AmericanActors",
                "NativeGuid": "e67a6395249f0f42b5737c2ab3496419",
                "Parent": {
                    "Guid": "4322bb5f-01d8-445c-86ea-b021fe21609b",
                    "Name": "OU=Synapse",
                    "NativeGuid": "5fbb2243d8015c4486eab021fe21609b",
                    "Parent": null,
                    "Path": "LDAP://OU=Synapse,DC=sandbox,DC=local",
                    "Properties": {
                        "objectClass": [
                            "top",
                            "organizationalUnit"
                        ],
                        "ou": [
                            "Synapse"
                        ],
                        "description": [
                            "Modified By Guid"
                        ],
                        "distinguishedName": [
                            "OU=Synapse,DC=sandbox,DC=local"
                        ],
                        "instanceType": [
                            "4"
                        ],
                        "whenCreated": [
                            "6/19/2017 9:22:59 PM"
                        ],
                        "whenChanged": [
                            "8/28/2017 7:17:04 PM"
                        ],
                        "uSNCreated": [],
                        "uSNChanged": [],
                        "nTSecurityDescriptor": [],
                        "name": [
                            "Synapse"
                        ],
                        "objectGUID": [
                            "4322bb5f-01d8-445c-86ea-b021fe21609b"
                        ],
                        "objectCategory": [
                            "CN=Organizational-Unit,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                        ],
                        "dSCorePropagationData": [
                            "7/25/2017 3:33:45 PM",
                            "7/12/2017 4:07:35 PM",
                            "7/12/2017 4:05:57 PM",
                            "7/3/2017 3:03:47 PM",
                            "1/1/1601 12:00:00 AM"
                        ]
                    },
                    "SchemaClassName": "organizationalUnit",
                    "SchemaEntry": null,
                    "UsePropertyCache": true,
                    "Username": null
                },
                "Path": "LDAP://OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local",
                "Properties": {
                    "objectClass": [
                        "top",
                        "organizationalUnit"
                    ],
                    "ou": [
                        "AmericanActors"
                    ],
                    "description": [
                        "Hello World"
                    ],
                    "postalCode": [
                        "90210"
                    ],
                    "distinguishedName": [
                        "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "9/15/2017 7:19:05 PM"
                    ],
                    "whenChanged": [
                        "9/15/2017 7:19:05 PM"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "AmericanActors"
                    ],
                    "objectGUID": [
                        "95637ae6-9f24-420f-b573-7c2ab3496419"
                    ],
                    "managedBy": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "objectCategory": [
                        "CN=Organizational-Unit,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "1/1/1601 12:00:00 AM"
                    ]
                },
                "SchemaClassName": "organizationalUnit",
                "SchemaEntry": {
                    "Guid": "228d9a87-c302-11cf-9aa4-00aa004a5691",
                    "Name": "organizationalUnit",
                    "NativeGuid": "{228D9A87-C302-11CF-9AA4-00AA004A5691}",
                    "Parent": null,
                    "Path": "LDAP://schema/organizationalUnit",
                    "Properties": null,
                    "SchemaClassName": "Class",
                    "SchemaEntry": null,
                    "UsePropertyCache": true,
                    "Username": null
                },
                "UsePropertyCache": true,
                "Username": null
            }
        }
    ]
}
````

## AddToGroup (HTTP POST) or RemoveFromGroup (HTTP DELETE)

The format for the URL is identical for an AddToGroup and RemoveFromGroup action.  The only real difference would be the status message returned, either "Added To" or "Removed From" depending on which action was called.

### Add/Remove User To Group

**Request**

````
{{protocol}}://{{host}}:{{controllerPort}}/ad/user/mfox/FamousActors  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/user/CN=mfox,OU=Synapse,DC=sandbox,DC=local/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/user/S-1-5-21-4054027134-3251639354-3875066094-1773/S-1-5-21-4054027134-3251639354-3875066094-1774  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/user/1722b838-57e1-4058-a394-338882af9e2f/19cdf305-c43b-497a-a932-6091f4a09dbb  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox/FamousActors  (By SamAccountName)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox@sandbox.local/FamousActors  (By UserPrincipal / Name)

* Note: There is not "UserPrincipal" for a group, so last example is using "Name" for the groupIdentity.
````

**Response**

**Note**: The response below is from the first request above (by Name).  The status message will display whichever "identity" and "groupIdentity" you provide.

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "User [mfox] Added To Group [FamousActors].",
                    "Action": 5
                }
            ],
            "Type": 1,
            "Identity": "mfox",
            "User": {
                "EmailAddress": "mfox@company.com",
                "EmployeeId": "42",
                "GivenName": "Michael",
                "MiddleName": null,
                "Surname": "Fox",
                "VoiceTelephoneNumber": "1-800-555-1212",
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
                    "description": [
                        "American Actor, Back to the Future."
                    ],
                    "telephoneNumber": [
                        "1-800-555-1212"
                    ],
                    "givenName": [
                        "Michael"
                    ],
                    "distinguishedName": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "9/15/2017 7:00:18 PM"
                    ],
                    "whenChanged": [
                        "9/15/2017 7:00:18 PM"
                    ],
                    "displayName": [
                        "Michael J. Fox"
                    ],
                    "uSNCreated": [],
                    "memberOf": [
                        "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "mfox"
                    ],
                    "objectGUID": [
                        "1722b838-57e1-4058-a394-338882af9e2f"
                    ],
                    "userAccountControl": [
                        "328320"
                    ],
                    "badPwdCount": [
                        "0"
                    ],
                    "codePage": [
                        "0"
                    ],
                    "countryCode": [
                        "0"
                    ],
                    "employeeID": [
                        "42"
                    ],
                    "homeDirectory": [
                        "C:\\Temp"
                    ],
                    "homeDrive": [
                        "F:"
                    ],
                    "badPasswordTime": [],
                    "lastLogoff": [],
                    "lastLogon": [],
                    "scriptPath": [
                        "C:\\Temp\\Scripts"
                    ],
                    "pwdLastSet": [],
                    "primaryGroupID": [
                        "513"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1773"
                    ],
                    "accountExpires": [],
                    "logonCount": [
                        "0"
                    ],
                    "sAMAccountName": [
                        "mfox"
                    ],
                    "sAMAccountType": [
                        "805306368"
                    ],
                    "managedObjects": [
                        "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "userPrincipalName": [
                        "mfox@sandbox.local"
                    ],
                    "objectCategory": [
                        "CN=Person,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "1/1/1601 12:00:00 AM"
                    ],
                    "mail": [
                        "mfox@company.com"
                    ]
                },
                "AccountExpirationDate": "2017-10-10T11:35:46-05:00",
                "AccountLockoutTime": null,
                "AllowReversiblePasswordEncryption": true,
                "BadLogonCount": 0,
                "DelegationPermitted": true,
                "Enabled": true,
                "HomeDirectory": "C:\\Temp",
                "HomeDrive": "F:",
                "LastBadPasswordAttempt": null,
                "LastLogon": null,
                "LastPasswordSet": "2017-09-15T14:00:18.1021386-05:00",
                "PasswordNeverExpires": true,
                "PasswordNotRequired": false,
                "PermittedLogonTimes": null,
                "ScriptPath": "C:\\Temp\\Scripts",
                "SmartcardLogonRequired": true,
                "UserCannotChangePassword": false,
                "ContextType": 1,
                "Description": "American Actor, Back to the Future.",
                "DisplayName": "Michael J. Fox",
                "DistinguishedName": "CN=mfox,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "1722b838-57e1-4058-a394-338882af9e2f",
                "Name": "mfox",
                "SamAccountName": "mfox",
                "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1773",
                "StructuralObjectClass": "user",
                "UserPrincipalName": "mfox@sandbox.local",
                "Groups": [
                    {
                        "ContextType": 1,
                        "Description": "All domain users",
                        "DisplayName": null,
                        "DistinguishedName": "CN=Domain Users,CN=Users,DC=sandbox,DC=local",
                        "Guid": "b9c132e5-198a-4b08-b52f-6eef92445374",
                        "Name": "Domain Users",
                        "SamAccountName": "Domain Users",
                        "Sid": "S-1-5-21-4054027134-3251639354-3875066094-513",
                        "StructuralObjectClass": "group",
                        "UserPrincipalName": null,
                        "Groups": []
                    },
                    {
                        "ContextType": 1,
                        "Description": "Famous Actors and Actresses",
                        "DisplayName": null,
                        "DistinguishedName": "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local",
                        "Guid": "19cdf305-c43b-497a-a932-6091f4a09dbb",
                        "Name": "FamousActors",
                        "SamAccountName": "FamousActors",
                        "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1774",
                        "StructuralObjectClass": "group",
                        "UserPrincipalName": null,
                        "Groups": []
                    }
                ]
            },
            "Group": null,
            "OrganizationalUnit": null
        }
    ]
}
````

### Add/Remove Group to Group

The format for the URL is identical for an AddToGroup and RemoveFromGroup action.  The only real difference would be the status message returned, either "Added To" or "Removed From" depending on which action was called.

**Request**

````
{{protocol}}://{{host}}:{{controllerPort}}/ad/group/FamousActors/AllActors  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/group/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local/CN=AllActors,OU=Synapse,DC=sandbox,DC=local (By DistinguishedName)
{{protocol}}://{{host}}:{{port}}/ad/group/S-1-5-21-4054027134-3251639354-3875066094-1774/S-1-5-21-4054027134-3251639354-3875066094-1775  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/group/19cdf305-c43b-497a-a932-6091f4a09dbb/cc8db84b-3aca-4e69-b1d1-6b9a3b30ee73  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/group/FamousActors/AllActors  (By SamAccountName)
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 1
                }
            ],
            "Type": 2,
            "Identity": "FamousActors",
            "User": null,
            "Group": {
                "GroupScope": 2,
                "IsSecurityGroup": true,
                "Members": [
                    {
                        "ContextType": 1,
                        "Description": "American Actor, Back to the Future.",
                        "DisplayName": "Michael J. Fox",
                        "DistinguishedName": "CN=mfox,OU=Synapse,DC=sandbox,DC=local",
                        "Guid": "1722b838-57e1-4058-a394-338882af9e2f",
                        "Name": "mfox",
                        "SamAccountName": "mfox",
                        "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1773",
                        "StructuralObjectClass": "user",
                        "UserPrincipalName": "mfox@sandbox.local",
                        "Groups": []
                    }
                ],
                "Properties": {
                    "objectClass": [
                        "top",
                        "group"
                    ],
                    "cn": [
                        "FamousActors"
                    ],
                    "description": [
                        "Famous Actors and Actresses"
                    ],
                    "member": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "distinguishedName": [
                        "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "9/15/2017 7:11:21 PM"
                    ],
                    "whenChanged": [
                        "9/15/2017 9:13:53 PM"
                    ],
                    "uSNCreated": [],
                    "info": [
                        "Group of famous actors and actresses."
                    ],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "FamousActors"
                    ],
                    "objectGUID": [
                        "19cdf305-c43b-497a-a932-6091f4a09dbb"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1774"
                    ],
                    "sAMAccountName": [
                        "FamousActors"
                    ],
                    "sAMAccountType": [
                        "268435456"
                    ],
                    "managedBy": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "groupType": [
                        "-2147483640"
                    ],
                    "objectCategory": [
                        "CN=Group,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "1/1/1601 12:00:00 AM"
                    ]
                },
                "ContextType": 1,
                "Description": "Famous Actors and Actresses",
                "DisplayName": null,
                "DistinguishedName": "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "19cdf305-c43b-497a-a932-6091f4a09dbb",
                "Name": "FamousActors",
                "SamAccountName": "FamousActors",
                "Sid": "S-1-5-21-4054027134-3251639354-3875066094-1774",
                "StructuralObjectClass": "group",
                "UserPrincipalName": null,
                "Groups": []
            },
            "OrganizationalUnit": null
        }
    ]
}
````

## AddAccessRule (HTTP POST), RemoveAccessRule (HTTP DELETE) or SetAccessRule (HTTP PUT)

The format for the URL is identical for the AddAccessRule, RemoveAccessRule and SetAccessRule actions.  The only differences would be :
* On Delete, no object will be returned (kinda obvious)
* The "Statuses > Action" field would reflect either "AddAccessRule", "RemoveAccessRule or "SetAccessRule", depending on which was called.  Valid values for permitted "rights" can be found [here](handler.md#activedirectoryrights-enumeration).

**NOTE** : For this example, the query flag "returnaccessrules=true" was included to show the access rules being returned.

---
### AddAccessRule/RemoveAccessRule/SetAccessRule to User

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/accessrule/user/mfox/user001/Allow/GenericAll  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/user/CN=mfox,OU=Synapse,DC=sandbox,DC=local/CN=user001,OU=Synapse,DC=sandbox,DC=local/Allow/GenericAll  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/user/S-1-5-21-4054027134-3251639354-3875066094-1773/S-1-5-21-4054027134-3251639354-3875066094-1206/Allow/GenericAll  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/user/1722b838-57e1-4058-a394-338882af9e2f/4db94271-1fde-402a-a8c5-1564dfd8d62b/Allow/GenericAll  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox/user001/Allow/GenericAll  (By SamAccountName)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox@sandbox.local/user001@sandbox.local/Allow/GenericAll  (By UserPrincipal)
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 9
                }
            ],
            "Type": 1,
            "Identity": "mfox",
            "User": {
                "EmailAddress": "mfox@company.com",
                "EmployeeId": "42",
                "GivenName": "Michael",
                "MiddleName": null,
                "Surname": "Fox",
                "VoiceTelephoneNumber": "1-800-555-1212",
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
                    "description": [
                        "American Actor, Back to the Future."
                    ],
                    "telephoneNumber": [
                        "1-800-555-1212"
                    ],
                    "givenName": [
                        "Michael"
                    ],
                    "distinguishedName": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "10/3/2017 6:11:40 PM"
                    ],
                    "whenChanged": [
                        "10/3/2017 6:12:08 PM"
                    ],
                    "displayName": [
                        "Michael J. Fox"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "mfox"
                    ],
                    "objectGUID": [
                        "49517710-7432-4450-bd6a-cfd4d6b7e0f5"
                    ],
                    "userAccountControl": [
                        "328320"
                    ],
                    "badPwdCount": [
                        "0"
                    ],
                    "codePage": [
                        "0"
                    ],
                    "countryCode": [
                        "0"
                    ],
                    "employeeID": [
                        "42"
                    ],
                    "homeDirectory": [
                        "C:\\Temp"
                    ],
                    "homeDrive": [
                        "F:"
                    ],
                    "badPasswordTime": [],
                    "lastLogoff": [],
                    "lastLogon": [],
                    "scriptPath": [
                        "C:\\Temp\\Scripts"
                    ],
                    "pwdLastSet": [],
                    "primaryGroupID": [
                        "513"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1834"
                    ],
                    "accountExpires": [],
                    "logonCount": [
                        "0"
                    ],
                    "sAMAccountName": [
                        "mfox"
                    ],
                    "sAMAccountType": [
                        "805306368"
                    ],
                    "userPrincipalName": [
                        "mfox@sandbox.local"
                    ],
                    "objectCategory": [
                        "CN=Person,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "10/3/2017 6:12:08 PM",
                        "1/1/1601 12:00:00 AM"
                    ],
                    "mail": [
                        "mfox@company.com"
                    ]
                },
                "AccountExpirationDate": "2017-10-10T11:35:46-05:00",
                "AccountLockoutTime": null,
                "AllowReversiblePasswordEncryption": true,
                "BadLogonCount": 0,
                "DelegationPermitted": true,
                "Enabled": true,
                "HomeDirectory": "C:\\Temp",
                "HomeDrive": "F:",
                "LastBadPasswordAttempt": null,
                "LastLogon": null,
                "LastPasswordSet": "2017-10-03T13:11:40.9300291-05:00",
                "PasswordNeverExpires": true,
                "PasswordNotRequired": false,
                "PermittedLogonTimes": null,
                "ScriptPath": "C:\\Temp\\Scripts",
                "SmartcardLogonRequired": true,
                "UserCannotChangePassword": false,
                "ContextType": 1,
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
                "AccessRules": [
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131072,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-18",
                        "IdentityName": "SYSTEM",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-512",
                        "IdentityName": "Domain Admins",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-1206",
                        "IdentityName": "User001",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-1-0",
                        "IdentityName": "Everyone",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-560",
                        "IdentityName": "Windows Authorization Access Group",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-32-561",
                        "IdentityName": "Terminal Server License Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-32-561",
                        "IdentityName": "Terminal Server License Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-517",
                        "IdentityName": "Cert Publishers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-553",
                        "IdentityName": "RAS and IAS Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-553",
                        "IdentityName": "RAS and IAS Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-553",
                        "IdentityName": "RAS and IAS Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-553",
                        "IdentityName": "RAS and IAS Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-526",
                        "IdentityName": "Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-527",
                        "IdentityName": "Enterprise Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-3-0",
                        "IdentityName": "CREATOR OWNER",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 32,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 3,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 304,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-519",
                        "IdentityName": "Enterprise Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 4,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983485,
                        "IdentityReference": "S-1-5-32-544",
                        "IdentityName": "Administrators",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    }
                ]
            },
            "Group": null,
            "OrganizationalUnit": null
        }
    ]
}
````

---
### AddAccessRule/RemoveAccessRule/SetAccessRule to Group

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/accessrule/group/FamousActors/mfox/Allow/GenericAll  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/group/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local/CN=mfox,OU=Synapse,DC=sandbox,DC=local/Allow/GenericAll  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/group/S-1-5-21-4054027134-3251639354-3875066094-1835/S-1-5-21-4054027134-3251639354-3875066094-1773/Allow/GenericAll  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/group/72e11fd9-10f4-4c27-acb4-08dd30c78b8f/1722b838-57e1-4058-a394-338882af9e2f/Allow/GenericAll  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/group/FamousActors/mfox/Allow/GenericAll  (By SamAccountName)
````

**Response**
````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 9
                }
            ],
            "Type": 2,
            "Identity": "FamousActors",
            "User": null,
            "Group": {
                "GroupScope": 2,
                "IsSecurityGroup": true,
                "Members": null,
                "Properties": {
                    "objectClass": [
                        "top",
                        "group"
                    ],
                    "cn": [
                        "FamousActors"
                    ],
                    "distinguishedName": [
                        "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "10/3/2017 6:19:31 PM"
                    ],
                    "whenChanged": [
                        "10/3/2017 6:22:01 PM"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "FamousActors"
                    ],
                    "objectGUID": [
                        "72e11fd9-10f4-4c27-acb4-08dd30c78b8f"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1835"
                    ],
                    "sAMAccountName": [
                        "FamousActors"
                    ],
                    "sAMAccountType": [
                        "268435456"
                    ],
                    "groupType": [
                        "-2147483640"
                    ],
                    "objectCategory": [
                        "CN=Group,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "10/3/2017 6:22:01 PM",
                        "1/1/1601 12:00:00 AM"
                    ]
                },
                "ContextType": 1,
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
                "AccessRules": [
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-18",
                        "IdentityName": "SYSTEM",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-512",
                        "IdentityName": "Domain Admins",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-1834",
                        "IdentityName": "mfox",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-560",
                        "IdentityName": "Windows Authorization Access Group",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-526",
                        "IdentityName": "Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-527",
                        "IdentityName": "Enterprise Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-3-0",
                        "IdentityName": "CREATOR OWNER",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 32,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 3,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 304,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-519",
                        "IdentityName": "Enterprise Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 4,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983485,
                        "IdentityReference": "S-1-5-32-544",
                        "IdentityName": "Administrators",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    }
                ]
            },
            "OrganizationalUnit": null
        }
    ]
}
````

---
### AddAccessRule/RemoveAccessRule/SetAccessRule to OrganizationalUnit

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/accessrule/ou/AmericanActors/FamousActors/Allow/GenericAll  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/ou/CN=AmericanActors,OU=Synapse,DC=sandbox,DC=local/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local/Allow/GenericAll  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/ou/768d8062-c1d2-4d67-9ad6-57c73cc3982b/72e11fd9-10f4-4c27-acb4-08dd30c78b8f/Allow/GenericAll  (By Guid)
````

**Response**
````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 9
                }
            ],
            "Type": 4,
            "Identity": "AmericanActors",
            "User": null,
            "Group": null,
            "OrganizationalUnit": {
                "DistinguishedName": "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "768d8062-c1d2-4d67-9ad6-57c73cc3982b",
                "Name": "OU=AmericanActors",
                "NativeGuid": "62808d76d2c1674d9ad657c73cc3982b",
                "Parent": {
                    "Guid": "4322bb5f-01d8-445c-86ea-b021fe21609b",
                    "Name": "OU=Synapse",
                    "NativeGuid": "5fbb2243d8015c4486eab021fe21609b",
                    "Parent": null,
                    "Path": "LDAP://OU=Synapse,DC=sandbox,DC=local",
                    "Properties": null,
                    "SchemaClassName": "organizationalUnit",
                    "SchemaEntry": null,
                    "UsePropertyCache": true,
                    "Username": null,
                    "AccessRules": null
                },
                "Path": "LDAP://OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local",
                "Properties": {
                    "objectClass": [
                        "top",
                        "organizationalUnit"
                    ],
                    "ou": [
                        "AmericanActors"
                    ],
                    "description": [
                        "Hello World"
                    ],
                    "postalCode": [
                        "90210"
                    ],
                    "distinguishedName": [
                        "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "10/3/2017 6:27:26 PM"
                    ],
                    "whenChanged": [
                        "10/3/2017 6:27:58 PM"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "AmericanActors"
                    ],
                    "objectGUID": [
                        "768d8062-c1d2-4d67-9ad6-57c73cc3982b"
                    ],
                    "managedBy": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "objectCategory": [
                        "CN=Organizational-Unit,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "10/3/2017 6:27:58 PM",
                        "1/1/1601 12:00:00 AM"
                    ]
                },
                "SchemaClassName": "organizationalUnit",
                "SchemaEntry": {
                    "Guid": "228d9a87-c302-11cf-9aa4-00aa004a5691",
                    "Name": "organizationalUnit",
                    "NativeGuid": "{228D9A87-C302-11CF-9AA4-00AA004A5691}",
                    "Parent": null,
                    "Path": "LDAP://schema/organizationalUnit",
                    "Properties": null,
                    "SchemaClassName": "Class",
                    "SchemaEntry": null,
                    "UsePropertyCache": true,
                    "Username": null,
                    "AccessRules": null
                },
                "UsePropertyCache": true,
                "Username": null,
                "AccessRules": [
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-18",
                        "IdentityName": "SYSTEM",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-512",
                        "IdentityName": "Domain Admins",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-1835",
                        "IdentityName": "FamousActors",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-550",
                        "IdentityName": "Print Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-526",
                        "IdentityName": "Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-527",
                        "IdentityName": "Enterprise Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-3-0",
                        "IdentityName": "CREATOR OWNER",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 32,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 3,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 304,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-519",
                        "IdentityName": "Enterprise Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 4,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983485,
                        "IdentityReference": "S-1-5-32-544",
                        "IdentityName": "Administrators",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    }
                ]
            }
        }
    ]
}
````


## PurgeAccessRules (HTTP DELETE)

The format for the URL is identical for the AddAccessRule, RemoveAccessRule and SetAccessRule actions, except there is no need for a type or rights, since purge removes all rules (Allow and Deny) for the given principal (user or group).

**NOTE** : For this example, the query flag "returnaccessrules=true" was included to show the access rules being returned.

---
### PurgeAccessRules on User

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/accessrule/user/mfox/user001  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/user/CN=mfox,OU=Synapse,DC=sandbox,DC=local/CN=user001,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/user/S-1-5-21-4054027134-3251639354-3875066094-1773/S-1-5-21-4054027134-3251639354-3875066094-1206  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/user/1722b838-57e1-4058-a394-338882af9e2f/4db94271-1fde-402a-a8c5-1564dfd8d62b  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox/user001/Allow/GenericAll  (By SamAccountName)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox@sandbox.local/user001@sandbox.local  (By UserPrincipal)
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 12
                }
            ],
            "Type": 1,
            "Identity": "mfox",
            "User": {
                "EmailAddress": "mfox@company.com",
                "EmployeeId": "42",
                "GivenName": "Michael",
                "MiddleName": null,
                "Surname": "Fox",
                "VoiceTelephoneNumber": "1-800-555-1212",
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
                    "description": [
                        "American Actor, Back to the Future."
                    ],
                    "telephoneNumber": [
                        "1-800-555-1212"
                    ],
                    "givenName": [
                        "Michael"
                    ],
                    "distinguishedName": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "10/3/2017 6:11:40 PM"
                    ],
                    "whenChanged": [
                        "10/3/2017 6:33:32 PM"
                    ],
                    "displayName": [
                        "Michael J. Fox"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "mfox"
                    ],
                    "objectGUID": [
                        "49517710-7432-4450-bd6a-cfd4d6b7e0f5"
                    ],
                    "userAccountControl": [
                        "328320"
                    ],
                    "badPwdCount": [
                        "0"
                    ],
                    "codePage": [
                        "0"
                    ],
                    "countryCode": [
                        "0"
                    ],
                    "employeeID": [
                        "42"
                    ],
                    "homeDirectory": [
                        "C:\\Temp"
                    ],
                    "homeDrive": [
                        "F:"
                    ],
                    "badPasswordTime": [],
                    "lastLogoff": [],
                    "lastLogon": [],
                    "scriptPath": [
                        "C:\\Temp\\Scripts"
                    ],
                    "pwdLastSet": [],
                    "primaryGroupID": [
                        "513"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1834"
                    ],
                    "accountExpires": [],
                    "logonCount": [
                        "0"
                    ],
                    "sAMAccountName": [
                        "mfox"
                    ],
                    "sAMAccountType": [
                        "805306368"
                    ],
                    "managedObjects": [
                        "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "userPrincipalName": [
                        "mfox@sandbox.local"
                    ],
                    "objectCategory": [
                        "CN=Person,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "10/3/2017 6:33:32 PM",
                        "10/3/2017 6:12:08 PM",
                        "1/1/1601 12:00:00 AM"
                    ],
                    "mail": [
                        "mfox@company.com"
                    ]
                },
                "AccountExpirationDate": "2017-10-10T11:35:46-05:00",
                "AccountLockoutTime": null,
                "AllowReversiblePasswordEncryption": true,
                "BadLogonCount": 0,
                "DelegationPermitted": true,
                "Enabled": true,
                "HomeDirectory": "C:\\Temp",
                "HomeDrive": "F:",
                "LastBadPasswordAttempt": null,
                "LastLogon": null,
                "LastPasswordSet": "2017-10-03T13:11:40.9300291-05:00",
                "PasswordNeverExpires": true,
                "PasswordNotRequired": false,
                "PermittedLogonTimes": null,
                "ScriptPath": "C:\\Temp\\Scripts",
                "SmartcardLogonRequired": true,
                "UserCannotChangePassword": false,
                "ContextType": 1,
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
                "AccessRules": [
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131072,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-18",
                        "IdentityName": "SYSTEM",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-512",
                        "IdentityName": "Domain Admins",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-1-0",
                        "IdentityName": "Everyone",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-560",
                        "IdentityName": "Windows Authorization Access Group",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-32-561",
                        "IdentityName": "Terminal Server License Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-32-561",
                        "IdentityName": "Terminal Server License Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-517",
                        "IdentityName": "Cert Publishers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-553",
                        "IdentityName": "RAS and IAS Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-553",
                        "IdentityName": "RAS and IAS Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-553",
                        "IdentityName": "RAS and IAS Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-553",
                        "IdentityName": "RAS and IAS Servers",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-526",
                        "IdentityName": "Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-527",
                        "IdentityName": "Enterprise Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-3-0",
                        "IdentityName": "CREATOR OWNER",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 32,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 3,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 304,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-519",
                        "IdentityName": "Enterprise Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 4,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983485,
                        "IdentityReference": "S-1-5-32-544",
                        "IdentityName": "Administrators",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    }
                ]
            },
            "Group": null,
            "OrganizationalUnit": null
        }
    ]
}
````

---
### PurgeAccessRules on Group

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/accessrule/group/FamousActors/mfox/  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/group/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local/CN=mfox,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/group/S-1-5-21-4054027134-3251639354-3875066094-1835/S-1-5-21-4054027134-3251639354-3875066094-1773  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/group/72e11fd9-10f4-4c27-acb4-08dd30c78b8f/1722b838-57e1-4058-a394-338882af9e2f  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/group/FamousActors/mfox  (By SamAccountName)
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 12
                }
            ],
            "Type": 2,
            "Identity": "FamousActors",
            "User": null,
            "Group": {
                "GroupScope": 2,
                "IsSecurityGroup": true,
                "Members": null,
                "Properties": {
                    "objectClass": [
                        "top",
                        "group"
                    ],
                    "cn": [
                        "FamousActors"
                    ],
                    "distinguishedName": [
                        "CN=FamousActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "10/3/2017 6:19:31 PM"
                    ],
                    "whenChanged": [
                        "10/3/2017 6:37:02 PM"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "FamousActors"
                    ],
                    "objectGUID": [
                        "72e11fd9-10f4-4c27-acb4-08dd30c78b8f"
                    ],
                    "objectSid": [
                        "S-1-5-21-4054027134-3251639354-3875066094-1835"
                    ],
                    "sAMAccountName": [
                        "FamousActors"
                    ],
                    "sAMAccountType": [
                        "268435456"
                    ],
                    "groupType": [
                        "-2147483640"
                    ],
                    "objectCategory": [
                        "CN=Group,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "10/3/2017 6:37:02 PM",
                        "10/3/2017 6:22:01 PM",
                        "1/1/1601 12:00:00 AM"
                    ]
                },
                "ContextType": 1,
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
                "AccessRules": [
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-18",
                        "IdentityName": "SYSTEM",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-512",
                        "IdentityName": "Domain Admins",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 256,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-560",
                        "IdentityName": "Windows Authorization Access Group",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-526",
                        "IdentityName": "Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-527",
                        "IdentityName": "Enterprise Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-3-0",
                        "IdentityName": "CREATOR OWNER",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 32,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 3,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 304,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-519",
                        "IdentityName": "Enterprise Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 4,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983485,
                        "IdentityReference": "S-1-5-32-544",
                        "IdentityName": "Administrators",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    }
                ]
            },
            "OrganizationalUnit": null
        }
    ]
}
````

---
### PurgeAccessRules on Organizational Unit

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/accessrule/group/AmericanActors/FamousActors/  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/group/CN=AmericanActors,OU=Synapse,DC=sandbox,DC=local/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/group/768d8062-c1d2-4d67-9ad6-57c73cc3982b/72e11fd9-10f4-4c27-acb4-08dd30c78b8f  (By Guid)
````

**Response**

````
{
    "Results": [
        {
            "Statuses": [
                {
                    "Status": 1,
                    "Message": "Success",
                    "Action": 12
                }
            ],
            "Type": 4,
            "Identity": "AmericanActors",
            "User": null,
            "Group": null,
            "OrganizationalUnit": {
                "DistinguishedName": "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local",
                "Guid": "768d8062-c1d2-4d67-9ad6-57c73cc3982b",
                "Name": "OU=AmericanActors",
                "NativeGuid": "62808d76d2c1674d9ad657c73cc3982b",
                "Parent": {
                    "Guid": "4322bb5f-01d8-445c-86ea-b021fe21609b",
                    "Name": "OU=Synapse",
                    "NativeGuid": "5fbb2243d8015c4486eab021fe21609b",
                    "Parent": null,
                    "Path": "LDAP://OU=Synapse,DC=sandbox,DC=local",
                    "Properties": null,
                    "SchemaClassName": "organizationalUnit",
                    "SchemaEntry": null,
                    "UsePropertyCache": true,
                    "Username": null,
                    "AccessRules": null
                },
                "Path": "LDAP://OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local",
                "Properties": {
                    "objectClass": [
                        "top",
                        "organizationalUnit"
                    ],
                    "ou": [
                        "AmericanActors"
                    ],
                    "description": [
                        "Hello World"
                    ],
                    "postalCode": [
                        "90210"
                    ],
                    "distinguishedName": [
                        "OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "instanceType": [
                        "4"
                    ],
                    "whenCreated": [
                        "10/3/2017 6:27:26 PM"
                    ],
                    "whenChanged": [
                        "10/3/2017 6:41:31 PM"
                    ],
                    "uSNCreated": [],
                    "uSNChanged": [],
                    "nTSecurityDescriptor": [],
                    "name": [
                        "AmericanActors"
                    ],
                    "objectGUID": [
                        "768d8062-c1d2-4d67-9ad6-57c73cc3982b"
                    ],
                    "managedBy": [
                        "CN=mfox,OU=Synapse,DC=sandbox,DC=local"
                    ],
                    "objectCategory": [
                        "CN=Organizational-Unit,CN=Schema,CN=Configuration,DC=sandbox,DC=local"
                    ],
                    "dSCorePropagationData": [
                        "10/3/2017 6:41:31 PM",
                        "10/3/2017 6:27:58 PM",
                        "1/1/1601 12:00:00 AM"
                    ]
                },
                "SchemaClassName": "organizationalUnit",
                "SchemaEntry": {
                    "Guid": "228d9a87-c302-11cf-9aa4-00aa004a5691",
                    "Name": "organizationalUnit",
                    "NativeGuid": "{228D9A87-C302-11CF-9AA4-00AA004A5691}",
                    "Parent": null,
                    "Path": "LDAP://schema/organizationalUnit",
                    "Properties": null,
                    "SchemaClassName": "Class",
                    "SchemaEntry": null,
                    "UsePropertyCache": true,
                    "Username": null,
                    "AccessRules": null
                },
                "UsePropertyCache": true,
                "Username": null,
                "AccessRules": [
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-11",
                        "IdentityName": "Authenticated Users",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-18",
                        "IdentityName": "SYSTEM",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-512",
                        "IdentityName": "Domain Admins",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-548",
                        "IdentityName": "Account Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 3,
                        "IdentityReference": "S-1-5-32-550",
                        "IdentityName": "Print Operators",
                        "InheritanceFlags": 0,
                        "IsInherited": false
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-526",
                        "IdentityName": "Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-527",
                        "IdentityName": "Enterprise Key Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-3-0",
                        "IdentityName": "CREATOR OWNER",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 8,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 16,
                        "IdentityReference": "S-1-5-9",
                        "IdentityName": "ENTERPRISE DOMAIN CONTROLLERS",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 32,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 131220,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 48,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 3,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 304,
                        "IdentityReference": "S-1-5-10",
                        "IdentityName": "SELF",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983551,
                        "IdentityReference": "S-1-5-21-4054027134-3251639354-3875066094-519",
                        "IdentityName": "Enterprise Admins",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 4,
                        "IdentityReference": "S-1-5-32-554",
                        "IdentityName": "Pre-Windows 2000 Compatible Access",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    },
                    {
                        "ControlType": 0,
                        "Rights": 983485,
                        "IdentityReference": "S-1-5-32-544",
                        "IdentityName": "Administrators",
                        "InheritanceFlags": 1,
                        "IsInherited": true
                    }
                ]
            }
        }
    ]
}
````