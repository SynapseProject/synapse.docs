# WebApi Authorization

## Overview

The WebApi Authorization section is used to granulate who can access the Controller, Node, and Admin methods.  Configure Authorization by adding one or more `Providers`, using the `AppliesTo` settings to filter what is affected by the specific Provider.

## YAML Layout

```yaml
WebApi:
  Authorization:
    AllowAnonymous: true|false
    Providers:
    - Type: {Provider Library:Name}
      AppliesTo:
        ServerRole: {ServerRole}
        Topics:
        - {Topic Name}
      Config:
        {Provider-specific configuration data}
```

## Fields

|Name|Type|Required|Description
|-|-|-|-
|AllowAnonymous|boolean|No, default is `True`|Set this `False` to specifically disallow anonymous access to authorization-checked actions.
|Providers|Section|No|Create entries in the section to control access to API methods.  If no Providers are configured, the API defaults to "allow access to all methods for everyone."  *Note:* The Controller->Start methods will still honor Plan security, even if "everyone" can execute the Start method itself.
|Type|string|Yes|Syntax Library:Namespace.ClassName (as with Hander Type declaration)
|AppliesTo|Section|No|Used to filter the Providers list at runtime.
|AppliesTo.ServerRole|bitmask enumeration|No|Specifies with of the WebApi functions are governed by this Provider. You may combine any of the values as csv: `ServerRole: Node, Admin, ...`.  The default value is `Universal`.
|AppliesTo.Topics|string list|No|Adds to the filter for specific API methods/method groups.  See below for valid options.

#### ServerRole Enumeration
```java
[Flags]
public enum ServerRole
{
    Controller = 1,
    Node = 2,
    Admin = 4,
    Server = 7,
    Enterprise = 8, //future use
    Universal = 15  //future use
}
```

#### Topics:
|Node Topic|Description
|-|-
|Drainstop|Allows Drainstop execution and viewing job queue depth
|**Admin Topic**|**Description**
|About|Allows execution of /admin/hello/about, which dumps the Synapse.Server.config.yaml and the server file inventory in a json response.
|AutoUpdate|Allows starting the AutoUpdate process.
|AutoUpdateLogs|Allows AutoUpdate log inspection.
|Logs|Allows Synapse.Server log inspection.

## Runtime Filtering of Providers Detail

As stated above, omitting a Providers list translates to "allow access to all methods for everyone."  Further, omitting a Provider to govern any particular function follows in suit, and access is granted by default.  As such, populating the Providers list translates to "this function requires authorization."  At runtime, the Authorization provider list is filtered in one of the two patterns below:

1. ContainsRole( role ) .and. NoTopics()
2. ContainsRole( role ) .and. ContainsTopic( topic )

#### Pattern 1: ContainsRole( role ) .and. NoTopics()

Each of the main WebApi entry points supports "global" Authorization, meaning, you can authorize at the Controller, Node, or Admin level, together, or separate.  In the following example, the first provider will authorize the Controller and Node, and the second will authorize the Admin interface only.

```yaml
WebApi:
  Authorization:
    Providers:
    - Type: {Provider Library:Name}
      AppliesTo:
        ServerRole: Controller, Node
      Config:
        {Provider-specific configuration data}
    - Type: {Provider Library:Name}
      AppliesTo:
        ServerRole: Admin
      Config:
        {Provider-specific configuration data}
```

#### Pattern 2: ContainsRole( role ) .and. ContainsTopic( topic )

For WebApi methods that advertise a Topic, you may further granulate access with the Topics list.  The following example extends the previous example by providing specific access to the Node->Drainstop and Admin->About/AutoUpdate methods.  When a topic list is specified, any Provider entries matching Pattern #1 are ignored in favor of those matching Pattern #2.

```yaml
WebApi:
  Authorization:
    Providers:
    - Type: {Provider Library:Name}
      AppliesTo:
        ServerRole: Controller, Node
      Config:
        {Provider-specific configuration data}
    - Type: {Provider Library:Name}
      AppliesTo:
        ServerRole: Node
        Topics:
        - Drainstop
      Config:
        {Provider-specific configuration data}
    - Type: {Provider Library:Name}
      AppliesTo:
        ServerRole: Admin
      Config:
        {Provider-specific configuration data}
    - Type: {Provider Library:Name}
      AppliesTo:
        ServerRole: Admin
        Topics:
        - About
        - AutoUpdate
      Config:
        {Provider-specific configuration data}
```

**Note:** the runtime engine processes the entire list of matching Providers list, sequentially.  If multiple Providers address the same ServerRole/Topic, the engine will process each Provider and ultimately reports the cumulative authorization outcome.

## Providers

### Synapse.Authorization:WindowsPrincipalProvider

The WindowsPrincipalProvider authorizes against a simple list of `domain\username` (users) or Active Directory groups in a standard Allowed/Denied pattern, where Denied principals always supersede Allowed principals.  As mentioned above, the runtime engine processes the Providers list sequentially and the first Provider to answer with Allow/Deny will terminate processing.


```yaml
WebApi:
  Authorization:
    Providers:
    - Type: Synapse.Authorization:WindowsPrincipalProvider
      Config:
        LdapRoot: {LDAP://...}
        ListSourcePath: {file path}
        Users:
          Allowed:
          - {User Principal}
          Denied:
          - {User Principal}
        Groups:
          Allowed:
          - {Group Principal}
          Denied:
          - {Group Principal}
```

#### WindowsPrincipalProvider Config Fields

|Name|Type|Required|Description
|-|-|-|-
LdapRoot|string|Yes*|Required only if using Groups.
|ListSourcePath|string|No|Path to a serialized Users/Groups structure.  Changes to this file are detected at runtime, such that Synapse.Server does not require a restart to honor updates.
|Users|string lists|No|Two lists, `Allowed` and `Denied` of user principals, in the form of `domain\username`
|Groups|string lists|No|Two lists, `Allowed` and `Denied` of group principals, in the form of `groupname`

**Note**: Both Users and Groups are marked as not required.  Omitting `Allowed` translates to "everyone is allowed," and omitting `Denied` translates to "no one is denied."  Providing an `Allowed` list translates to "only these principals are allowed, others are implicitly denied."  Providing a `Denied` list *explicitly* denies named principals and overrides any explicit allows.

||Provided|Omitted
|-|-|-
|**Allowed**|Explicitly granted access. Others implicitly denied.|"Everyone" implicitly allowed.
|**Denied**|Explicitly denied access. Others implicitly allowed.|"Everyone" implicitly allowed.

In the two examples below, with an explicit Allow, the "Synapse Admins" group is permitted access to the Admin methods.  The user "steve" (presumably a member of "Synapse Admins") is explicitly denied access, and any other principals are implicitly denied.  For user "steve," both Provider setups produce the same result since all matching Providers are processed.  However, for group "Synapse Admins," the second Provider setup also grants access to the Controller and Node methods.

```yaml
WebApi:
  Authorization:
    Providers:
    - Type: Synapse.Authorization:WindowsPrincipalProvider
      AppliesTo:
        ServerRole: Admin
      Config:
        LdapRoot: LDAP://myDomain.domain.com/
        Users:
          Denied:
          - myDomain\steve
        Groups:
          Allowed:
          - Synapse Admins
```

```yaml
WebApi:
  Authorization:
    Providers:
    - Type: Synapse.Authorization:WindowsPrincipalProvider
      AppliesTo:
        ServerRole: Admin
      Config:
        Users:
          Denied:
          - myDomain\steve
    - Type: Synapse.Authorization:WindowsPrincipalProvider
      AppliesTo:
        ServerRole: Admin, Controller, Node
      Config:
        LdapRoot: LDAP://myDomain.domain.com/
        Groups:
          Allowed:
          - Synapse Admins
```

### Synapse.Authorization.Suplex:SuplexProvider

The SuplexProvider is nearly identical to the WindowsPrincipalProvider, except instead of sourcing Active Directory for Group information, it sources a Suplex store.  Like the WindowsPrincipalProvider, the SuplexProvider authorizes against a simple list of `username` \ `domain\username` (users) or Suplex groups in a standard Allowed/Denied pattern, where Denied principals always supersede Allowed principals.  As mentioned above, the runtime engine processes the Providers list sequentially and the first Provider to answer with Allow/Deny will terminate processing.


```yaml
WebApi:
  Authorization:
    Providers:
    - Type: Synapse.Authorization.Suplex:SuplexProvider
      Config:
        Connection:
          Type: File #only File is supported at this time, is default value
          Path: {Path to Suplex file-based store}
        ListSourcePath: {file path}
        Users:
          Allowed:
          - {User Principal}
          Denied:
          - {User Principal}
        Groups:
          Allowed:
          - {Group Principal}
          Denied:
          - {Group Principal}
```

#### SuplexProvider Config Fields

Most of the fields in the SuplexProvider mirror those of the WindowsPrincipalProvider.  Only the _Connection_ fields differ, and _WindowsPrincipalProvider.LDAP_ is removed.

|Name|Type|Required|Description
|-|-|-|-
Connection.Type|enum|Yes|Options are **File**, SqlServer, RestApi.  At present, only _File_ is supported, and it's the default value.
Connection.Path|string|Yes|Path to the Suple file-based store.
|ListSourcePath|string|No|_See WindowsPrincipalProvider._
|Users|string lists|No|_See WindowsPrincipalProvider._
|Groups|string lists|No|_See WindowsPrincipalProvider._