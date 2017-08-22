# Caller Impersonation

## Overview

Synapse allows the user's credentails to be used for authentication between the Controller and Node, as well as allowing the user's credentials to execute the handler(s) as well.

## Assumptions

The following is a list of assumptions that are made when using impersonation.  If any of these assumptions aren't true, execution errors will occur.

- The credentails provided by the calling client are valid in the specified domain.
- The provided credentails are able to execute commands on the servers where both the Controller and Node reside.
    - Logon to server rights required
    - User must have rights to perform actions executed by the handlers.

## Enabling User Impersonation

User impersonation is enabled for the Controller and/or Node in the [server config file](/run/setup#configuration) under the WebApi section.  The UseImpersonation field is set to either "true" or "false".

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

## Valid Configurations

User impersonation only happens when both the Controller and Node are set to "UseImpersonation", and the authorization schemes allow for the passing of credentails through to the handler.  The table below tells whether a handler will execute under the caller's credentails assuming both the Controller and Node allow impersonation.  Cells marked with "N/A" are not valid configurations for Synapse authentication between the Controller and Node, regardless of impersonation settings.

|Controller \| Node >>><br/>  vvv|Anonymous|Basic|Integrated|Ntlm|Negotiate
|------------------------------|---------|-----|----------|----|---------
|**Anonymous**|No|N/A|No|No|No
|**Basic**|No|Yes|N/A|N/A|N/A
|**Integrated**|No|N/A|Yes|Yes|Yes
|**Ntlm**|No|N/A|Yes|Yes|Yes
|**Negotiate**|No|N/A|Yes|Yes|Yes



## Common Configurations

For each scenario below, assume the following setup:

* **User** : The user "SANDBOX\guy" will be calling Synapse Controller to start execution of a plan.
* **Controller** : The Controller service is running under the credentails "SANDBOX\synapse-controller"
* **Node** : The Node service is running under the credentails "SANDBOX\synapse-node"

Also, each scenario below assumes impersonation is set to true in both the Controller and Node config files.  To see how each scenario below would differ with user impersonation disabled, click [here](/run/setup/authentication#common-configurations).

### Simple Windows Authentication

![Windows Authentication, Impersonation](/img/syn_auth_windows_imp.png "Simple Windows Authentication With Impersonation")

The Controller and Node both accept a single authentcation type of IntegratedWindowsAuthentication, Ntlm or Negotiate and are both set to use client impersonation.

1. Client passes user's (SANDBOX\guy) credentials to Controller.
2. Controller passes plan to Node using the client's credentails (SANDBOX\guy).
3. Node executes handlers using the client's credentials. (SANDBOX\guy).
4. Node sends status updates to Controller using the client's credentials (SANDBOX\guy).

#### Server Config File Setup Details

|Synapse Server Config|Value
|---------------------|-----
|WebApi > Authentication > Scheme (Controller)|IntegratedWindowsAuthentication
|WebApi > Authentication > Scheme (Node)|IntegratedWindowsAuthentication
|Controller > NodeAuthenticationScheme|Empty
|Node > ControllerAuthenticationScheme|Empty
|WebApi > [UseImpersonation](/run/setup/impersonation) (Controller)|true
|WebApi > [UseImpersonation](/run/setup/impersonation) (Node)|true

### No Authentication

![No Authentication, Impersonation](/img/syn_auth_anonymous_imp.png "No Authentication, Impersonation")

This is where the controller and node are open to the world.  Since no credentails are passed, this performs the same as running with impersonation set to false.

1. Client passes no credentials to Controller.
2. Controller passes plan to Node with no credentials.
3. Node executes handlers using the Node service RunAs credentials. (SANDBOX\synapse-node).
4. Node sends status updates to Controller with no credentials.

#### Server Config File Setup Details

|Synapse Server Config|Value
|---------------------|-----
|WebApi > Authentication > Scheme (Controller)|Anonymous
|WebApi > Authentication > Scheme (Node)|Anonymous
|Controller > NodeAuthenticationScheme|Empty
|Node > ControllerAuthenticationScheme|Empty
|WebApi > [UseImpersonation](/run/setup/impersonation) (Controller)|true
|WebApi > [UseImpersonation](/run/setup/impersonation) (Node)|true



### Authentication on Controller Only

![Authentication on Controller Only, Impersonation](/img/syn_auth_ntlmctrlonly_imp.png "Ntlm on Controller Only with Impersonation")

This is where the controller has Windows Authentication set (Ntlm, Negotiate, or IntegratedWindowsAuthentication) but the node is running with no authentication.  The key thing to remember here is that if you don't have "SignPlan" set to true in the Controller, and "ValidatePlanSignature" set to true in the node, users could bypass the Controller and execute plans directly if they knew the URL, without any authentication.  Since user credentials never make it past the Controller, the effect of this pattern is the same whether or not impersonation is enabled.

1. Client passes user's (SANDBOX\guy) credentials to Controller.
2. Controller passes plan to Node with no credentials.
3. Node executes handlers using the Node service RunAs credentials. (SANDBOX\synapse-node).
4. Node sends status updates to Controller using the Node service RunAs credentials (SANDBOX\synapse-node).

#### Server Config File Setup Details

|Synapse Server Config|Value
|---------------------|-----
|WebApi > Authentication > Scheme (Controller)|Ntlm
|WebApi > Authentication > Scheme (Node)|Anonymous
|Controller > NodeAuthenticationScheme|Empty
|Node > ControllerAuthenticationScheme|Empty
|WebApi > [UseImpersonation](/run/setup/impersonation) (Controller)|true
|WebApi > [UseImpersonation](/run/setup/impersonation) (Node)|true

### Multi-Authentication on Controller, Windows Authentication on Node

![Multi Controller, Ntlm Node, Impersonation](/img/syn_auth_multictrl_ntlmnode_imp.png "Multi Controller, Ntlm Node with Impersonation")

This allows both Basic and Windows Authentication (Ntlm, Negotiate, or IntegratedWindowsAuthentication) to the Controller, but enforces Windows Authentication on the Node.  This setup is used to support applications that don't support sending Windows credentails in their web calls.  **Warning** : Basic authentication is not secure on its own.  The username and password are simply Base64 encoded in the Request header.  This should only be used over a secure connection.

1. Client passes user's (SANDBOX\guy) credentials to Controller either via Basic or Ntlm authentication.
2. Controller passes plan to Node using the user's credentails via Ntlm authentication (SANDBOX\guy).
3. Node executes handlers using the user's credentials. (SANDBOX\guy).
4. Node sends status updates to Controller using the user's credentails via Ntlm authentication (SANDBOX\guy).

#### Server Config File Setup Details

|Synapse Server Config|Value
|---------------------|-----
|WebApi > Authentication > Scheme (Controller)|Ntlm, Basic
|WebApi > Authentication > Scheme (Node)|Ntlm
|Controller > NodeAuthenticationScheme|Ntlm
|Node > ControllerAuthenticationScheme|Ntlm
|WebApi > [UseImpersonation](/run/setup/impersonation) (Controller)|true
|WebApi > [UseImpersonation](/run/setup/impersonation) (Node)|true

