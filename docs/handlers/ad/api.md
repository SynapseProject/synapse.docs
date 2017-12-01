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
|Get|GET|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;
|Create|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;
|Modify|PUT|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;
|Delete|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;
|AddToGroup|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;/&lt;groupIdentity&gt;
|RemoveFromGroup|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/&lt;object&gt;/&lt;identity&gt;&lt;groupIdentity&gt;
|AddAccessRule|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;type&gt;/&lt;rights&gt;
|RemoveAccessRule|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;type&gt;/&lt;rights&gt;
|SetAccessRule|PUT|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;type&gt;/&lt;rights&gt;
|PurgeAccessRules|DELETE|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/accessrule/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;
|Search|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/search
|[Search (Custom)](#custom-searches)|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/search/&lt;planname&gt;
|AddRole|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/role/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;role&gt;
|RemoveRole|POST|&lt;protocol&gt;://&lt;host&gt;:&lt;port&gt;/ad/role/&lt;object&gt;/&lt;identity&gt;/&lt;principal&gt;/&lt;role&gt;


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

---
## Get (HTTP GET) or Delete (HTTP DELETE)

The format for the URL is identical for a Get and a Delete action.  The only differences would be :
* On Delete, no object will be returned (kinda obvious)
* The "Statuses > Action" field would reflect either "Get" or "Delete", depending on which was called.

---
### Get/Delete User

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/user/mfox  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/user/CN=mfox,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/user/S-1-5-21-4054027134-3251639354-3875066094-1773  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/user/1722b838-57e1-4058-a394-338882af9e2f  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox  (By SamAccountName)
{{protocol}}://{{host}}:{{port}}/ad/user/mfox@sandbox.local  (By UserPrincipal)
````

---
### Get/Delete Group

**Request**

````
{{protocol}}://{{host}}:{{port}}/ad/group/FamousActors  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/group/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local  (By DistinguishedName)
{{protocol}}://{{host}}:{{port}}/ad/group/S-1-5-21-4054027134-3251639354-3875066094-1774  (By Sid)
{{protocol}}://{{host}}:{{port}}/ad/group/19cdf305-c43b-497a-a932-6091f4a09dbb  (By Guid)
{{protocol}}://{{host}}:{{port}}/ad/group/FamousActors  (By SamAccountName)
````

---
### Get/Delete Organizational Unit

**Request**

````
{{protocol}}://{{host}}:{{controllerPort}}/ad/ou/AmericanActors  (By Name)
{{protocol}}://{{host}}:{{controllerPort}}/ad/ou/OU=AmericanActors,OU=Synapse,DC=sandbox,DC=local  (By DistinguishedName)
{{protocol}}://{{host}}:{{controllerPort}}/ad/ou/95637ae6-9f24-420f-b573-7c2ab3496419  (By Guid)
````

---
## AddToGroup (HTTP POST) or RemoveFromGroup (HTTP DELETE)

The format for the URL is identical for an AddToGroup and RemoveFromGroup action.  The only real difference would be the status message returned, either "Added To" or "Removed From" depending on which action was called.

---
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

---
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

---
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


---
### AddAccessRule/RemoveAccessRule/SetAccessRule to OrganizationalUnit

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/accessrule/ou/AmericanActors/FamousActors/Allow/GenericAll  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/ou/CN=AmericanActors,OU=Synapse,DC=sandbox,DC=local/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local/Allow/GenericAll  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/ou/768d8062-c1d2-4d67-9ad6-57c73cc3982b/72e11fd9-10f4-4c27-acb4-08dd30c78b8f/Allow/GenericAll  (By Guid)
````

---
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

---
### PurgeAccessRules on Organizational Unit

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/accessrule/group/AmericanActors/FamousActors/  (By Name)
{{protocol}}://{{host}}:{{port}}/ad/group/CN=AmericanActors,OU=Synapse,DC=sandbox,DC=local/CN=FamousActors,OU=Synapse,DC=sandbox,DC=local  (By Distinguished Name)
{{protocol}}://{{host}}:{{port}}/ad/group/768d8062-c1d2-4d67-9ad6-57c73cc3982b/72e11fd9-10f4-4c27-acb4-08dd30c78b8f  (By Guid)
````

---
## Search

The "Search" action takes in a filter string, search base and a list of attributes to return, and returns the attributes for all DirectoryEntry objects that matches the filter string, assuming the requesting user has the rights to see that object in the first place.

**Requests**

````
{{protocol}}://{{host}}:{{port}}/ad/search

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
{{protocol}}://{{host}}:{{port}}/ad/search/GetAllGroups

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
## AddRole (HTTP POST) or RemoveRole (HTTP DELETE) To User

### Requests

````
{{protocol}}://{{host}}:{{controllerPort}}/ad/role/user/<identity>/<principal>/<role>
````

---
## AddRole (HTTP POST) or RemoveRole (HTTP DELETE) To Group

### Requests

````
{{protocol}}://{{host}}:{{controllerPort}}/ad/role/group/<identity>/<principal>/<role>
````

---
## AddRole (HTTP POST) or RemoveRole (HTTP DELETE) To OrganizationalUnit

### Requests

````
{{protocol}}://{{host}}:{{controllerPort}}/ad/role/ou/<identity>/<principal>/<role>
````

