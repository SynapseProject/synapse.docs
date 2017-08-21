# Supported Authentication Types and Configurations

## Overview

Synapse supports many mechanisms to identify whether a client is eligible to access a resource.  There are three places authentication is applied in the normal flow of a Synapse call :

* **Client to Controller** : This is typically the identify of the calling user or application, except when "Anonymous" authentication is allowed.
* **Controller to Node** : Under normal setup, this would be the credentials under which the Controller is running (service account, LocalSystem, etc...).  To use the credentails of the calling user or application, enable [impersonation](/run/setup/impersonation) in the [server config file](/run/setup#configuration).
* **Node to Controller** : Under normal setup, this would be the credentials under which the Node is running (service account, LocalSystem, etc...).  To use the credentails of the calling user or application, enable [impersonation](/run/setup/impersonation) in the [server config file](/run/setup#configuration).

*This document assumes that [impersonation](/run/setup/impersonation) is set to "false" in the [server config file](/run/setup#configuration) and that authentication between the Controller and Node uses the credentails under which each service is running.  To see how this changes when using [impersonation](/run/setup/impersonation), please click [here](/run/setup/impersonation).*

## Supported Types

The Synapse Server, Controller and Node support the following authentication types : 

  - Anonymous
  - Basic
  - Ntlm
  - IntegratedWindowsAuthentication
  - Negotiate

## Valid Configurations

Certain combinations of authentication types between the Controller and Node will not allow Synapse to function as expected, mostly because the authentication required by one isn't sufficient to meet the authentication nees of the other.

|Controller \| Node >>><br/>  vvv|Anonymous|Basic|Integrated|Ntlm|Negotiate
|------------------------------|---------|-----|----------|----|---------
|**Anonymous**|Yes|No|Yes|Yes|Yes
|**Basic**|Yes|Yes|No|No|No
|**Integrated**|Yes|No|Yes|Yes|Yes
|**Ntlm**|Yes|No|Yes|Yes|Yes
|**Negotiate**|Yes|No|Yes|Yes|Yes

## Defining Inbound Authentication Types

The authentication type of the Controller or Node is specified in the [server config file](/run/setup#configuration) under the WebApi > Authentication section.  Currently, only Basic authentication requires the "Config" section to be present.

````yaml
WebApi:
  Host: localhost
  Port: 1370
  IsSecure: false
  UseImpersonation: true
  Authentication:
    Scheme: Basic
    Config: 
      LdapRoot: LDAP://dc=sandbox,dc=local
      Domain: SANDBOX
````

## Explicitly Defining Outbound Authentication Type

By default, the authentication type received by the Controller will be used to communicate with the Node (and vice-versa) or will be automatically translated to the required type.  This automatic translation of authentication types does not apply to the "Basic" authentication type however.  If both the Controller and Node are setup for Basic authentication, Synapse Server will simply pass the header received from the client to the Node and then back to the Controller when sending status back.

However, if the Controller receives Basic authentication, but the node is expecting Ntlm authentication, you would need to tell
SynapseServer to not send the Basic header information forward, but use Ntlm instead.  In this case, you would need to explicitly indicate what type of authentication to use when communicating between the Controller and Node by using the following fields in the [server config file](/run/setup#configuration) : 

### Controller To Node Authentication Scheme

Use the "NodeAuthenticationScheme" field under the Controller section.

````yaml
Controller:
  NodeUrl: http://localhost:1375/synapse/node
  NodeAuthenticationScheme: Ntlm
  SignPlan: false
  Assemblies: 
  Dal:
    Type: Synapse.Controller.Dal.FileSystem:FileSystemDal
    LdapRoot: 
    Config:
      PlanFolderPath: Plans
      HistoryFolderPath: History
      ProcessPlansOnSingleton: false
      ProcessActionsOnSingleton: true
      Security:
        FilePath: Security
        IsRequired: false
        ValidateSignature: false
        SignaturePublicKeyFile: 
        GlobalExternalGroupsCsv: Everyone
````

### Node To Controller Authentication Scheme

Use the "ControllerAuthenticationScheme" field under the Node section.

````yaml
Node:
  MaxServerThreads: 0
  AuditLogRootPath: .\Logs
  Log4NetConversionPattern: '%d{ISO8601}|%-5p|(%t)|%m%n'
  SerializeResultPlan: true
  ValidatePlanSignature: false
  ControllerUrl: http://localhost:1370/synapse/execute
  ControllerAuthenticationScheme: Ntlm
````

### Use Case

This is mostly used in the case where the Controller is setup to use either Basic or Ntlm authentication, but the Node is Ntlm only.   When the Controller receives Basic, you need to tell the SynapseServer to use Ntlm instead of passing the Basic header through to the Node.

## Allowing Multiple Authentication Types

Multiple authentication types are supported on the same port for the Controller and Node, allowing the client to supply any valid authentcation type when calling Synapse Server.  Multiple authentication schemes are specified in the [server config file](/run/setup#configuration) by placing a comma-seperated list of supported authentication types in the Scheme field of the WebApi > Authentication element.

````yaml
WebApi:
  Host: localhost
  Port: 1370
  IsSecure: false
  UseImpersonation: true
  Authentication:
    Scheme: Basic, Ntlm
    Config: 
      LdapRoot: LDAP://dc=sandbox,dc=local
      Domain: SANDBOX
````

## Common Configurations

For each scenario below, assume the following setup:

* **User** : The user "SANDBOX\guy" will be calling Synapse Controller to start execution of a plan.
* **Controller** : The Controller service is running under the credentails "SANDBOX\synapse-controller"
* **Node** : The Node service is running under the credentails "SANDBOX\synapse-node"

Also, each scenario below assumes [impersonation](/run/setup/impersonation) is set to false.  To see how each scenario below would differ with user impersonation enabled, click [here](/run/setup/impersonation#common-configurations).

### Simple Windows Authentication

This is where the Controller and Node both accept a single authentcation type of IntegratedWindowsAuthentication, Ntlm or Negotiate.

#### Config Setup

|Synapse Server Config|Value
|---------------------|-----
|WebApi > Authentication > Scheme (Controller)|IntegratedWindowsAuthentication
|WebApi > Authentication > Scheme (Node)|IntegratedWindowsAuthentication
|Controller > NodeAuthenticationScheme|Empty
|Node > ControllerAuthenticationScheme|Empty
|WebApi > [UseImpersonation](/run/setup/impersonation)|false

#### Example Flow

1. Client passes user's (SANDBOX\guy) credentials to Controller.
2. Controller passes plan to Node the credentails under which the Controller service is running (SANDBOX\synapse-controller).
3. Node sends status updates to Controller using the credentails under which the Node service is running (SANDBOX\synapse-node).

#### Diagram

Coming Soon


### No Authentication

This is where the controller and node are open to the world.

#### Config Setup

|Synapse Server Config|Value
|---------------------|-----
|WebApi > Authentication > Scheme (Controller)|Anonymous
|WebApi > Authentication > Scheme (Node)|Anonymous
|Controller > NodeAuthenticationScheme|Empty
|Node > ControllerAuthenticationScheme|Empty
|WebApi > [UseImpersonation](/run/setup/impersonation)|false

#### Example Flow

1. Client passes no credentials to Controller.
2. Controller passes plan to Node the credentails with no credentials.
3. Node sends status updates to Controller with no credentials.

#### Diagram

Coming Soon


### Authentication on Controller Only

This is where the controller has Windows Authentication set (Ntlm, Negotiate, or IntegratedWindowsAuthentication) but the node is running with no authentication.  The key thing to remember here is that if you don't have "SignPlan" set to true in the Controller, and "ValidatePlanSignature" set to true in the node, users could bypass the Controller and execute plans directly if they knew the URL, without any authentication.

#### Config Setup

|Synapse Server Config|Value
|---------------------|-----
|WebApi > Authentication > Scheme (Controller)|Ntlm
|WebApi > Authentication > Scheme (Node)|Anonymous
|Controller > NodeAuthenticationScheme|Empty
|Node > ControllerAuthenticationScheme|Empty
|WebApi > [UseImpersonation](/run/setup/impersonation)|false

#### Example Flow

1. Client passes user's (SANDBOX\guy) credentials to Controller.
2. Controller passes plan to Node the credentails under which the Controller service is running (SANDBOX\synapse-controller).
3. Node sends status updates to Controller using the credentails under which the Node service is running (SANDBOX\synapse-node).

#### Diagram

Coming Soon

### Multi-Authentication on Controller, Windows Authentication on Node

This allows both Basic and Windows Authentication (Ntlm, Negotiate, or IntegratedWindowsAuthentication) to the Controller, but enforces Windows Authentication on the Node.  This setup is used to support applications that don't support sending Windows credentails in their web calls.  **Warning** : Basic authentication is not secure on its own.  The username and password are simply Base64 encoded in the Request header.  This should only be used over a secure connection.

#### Config Setup

|Synapse Server Config|Value
|---------------------|-----
|WebApi > Authentication > Scheme (Controller)|Ntlm, Basic
|WebApi > Authentication > Scheme (Node)|Ntlm
|Controller > NodeAuthenticationScheme|Ntlm
|Node > ControllerAuthenticationScheme|Ntlm
|WebApi > [UseImpersonation](/run/setup/impersonation)|false

#### Example Flow

1. Client passes user's (SANDBOX\guy) credentials to Controller either via Basic or Ntlm authentication.
2. Controller passes plan to Node the credentails under which the Controller service is running (SANDBOX\synapse-controller).
3. Node sends status updates to Controller using the credentails under which the Node service is running (SANDBOX\synapse-node).

#### Diagram

Coming Soon

