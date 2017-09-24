# Configuring Runtime Security Context with RunAs

By default, Plans execute under the security context of the host process.  RunAs blocks provide an in-plan method to configure the runtime security context for Plans and Actions, and can be used in conjuction with Windows-Service-configured security context, as well.

## Settings

|Name|Description
|-|-
|Domain|The name of the domain or server whose account database contains the UserName account.   If this parameter is NULL, the user name must be specified in UPN format. If this parameter is ".", the function validates the account by using only the local account database.
|UserName|A string that specifies the name of the user. This is the name of the user account to log on to. If you use the user principal name (UPN) format, User@DNSDomainName, the Domain parameter must be NULL. 
|Password|The plaintext password for the user account specified by UserName.
|IsInheritable|Boolean value that indicates if this RunAs block's settings can be inherited by child descendant RunAs blocks.  Setting `IsInheritable = false` will stop propagation of the current RunAs block.
|BlockInheritance|Boolean value that indicates if this RunAs block allows inheritance of settings from an ascendant RunAs block.  Setting `BlockInheritance = true` is effectively a "deny" on inheritance propagation, terminated by the current block.
|Crypto|A local Crypto block for securing Domain/UserName/Password settings.


```yaml
RunAs:
  Domain: myDomain
  UserName: myUser
  Password: qbMVEA84ouEncryptedStringQ2eFbJ/fvs=
  Crypto:
    Key:
      Uri: 'C:\crypto\pubPriv.xml'
      ContainerName: myContainer
    Elements:
    - Password
  IsInheritable: true
  BlockInheritance: true
```

## Inheritance Chains Detail

|Declaration|Result
|-|-
|None|All Plan Actions execute under the security context of the host process.
|Plan-level|Applies to all Actions withing the Plan, unless an Action overrides with local settings.
|Action-level|Overrides host-process, Plan-level, or other inherited security context.
|IsInheritable|Prevents further propagation of the current security context.
|BlockInheritance|Prevents propagation of the acendant security context to the current RunAs block.
|UserName = null, no Inherited context|Defaults to host-process security context.

## Example

![Inheritance Example](/img/syn_runas_inheritance.png "Inheritance Example")