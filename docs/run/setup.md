# Synapse Server

Installing and configuring Synapse.Server is accompished via the CLI ([Controller](/cli/controller/ "Controller command-line"), [Node](/cli/node/ "Node command-line")).  In either case, the code distribution is the same - you're simply choosing to run either as a Controller or Node.  The important settings for determining role are in `Synapse.Server.config.yaml`.  When extracting Synapse Server for setup. no Synapse.Server.config.yaml is present by default.  Synapse will generate a new, complete Synapse.Server.config.yaml wen you install the service, or by running the Controller/Node CLIs (for any reason).

### Common Settings

```yaml
ServiceName: Synapse.Controller
ServiceDisplayName: Synapse Controller
ServerRole: Controller
WebApiPort: 20000
AuthenticationScheme: IntegratedWindowsAuthentication
AuthenticationConfig:
  {authentication provider initialization config}
SignatureKeyFile: .\path\RsaKeyFileName.xml
SignatureKeyContainerName: DefaultContainerName
SignatureCspProviderFlags: NoFlags
```

|Name|Description
|-|-
|ServiceName & ServiceDisplayName|These values are only used when Installing Synapse.Server as a Windows Service.  Provide uniquely named values to run multiple instances of the services side-by-side.  The default values shown in the examples below will be modified to reflect Synapse.Controller or Synapse.Node (depending on role) under a default installation.  Alternatively, you may specify values directly.   _Note:_ Each unique instance of Synapse.Server should be distributed to an independent folder.  _Note:_ Do not modify these values _after_ installation.  You must first _uninstall_ the service, then modify and re-install.
|ServerRole|`Controller` or `Node`
|WebApiPort|Select a port value.
|WebApiIsSecure|Indicates if service is configured for SSL/TLS.
|AuthenticationScheme|Authentication  options.  See <a href="https://msdn.microsoft.com/en-us/library/system.net.authenticationschemes(v=vs.110).aspx" target="_blank">MSDN</a> for details.
|AuthenticationConfig|Proprietary configuration for specific authentication providers.  See [Configuring Basic Authentication] below.
|SignatureKeyFile|Filepath to RSA key file.  For a Controller instance, the keyfile must contain public and prvite key values.  For a Node instance, the keyfile must contain only the public key value.  If running both the Controller and Node from a single distribution, the keyfile must contain public and prvite key values.  To generate new RsaKeyFiles, use synapse.controller.cli *genkeys* option.  See CLI help for details.
|SignatureKeyContainerName|The name of the container in which to look for key values.  A keyfile may contain multiple containers.
|SignatureCspProviderFlags|CspProviderFlags options.  See <a href="https://msdn.microsoft.com/en-us/library/system.security.cryptography.cspproviderflags(v=vs.110).aspx" target="_blank">MSDN</a> for details.

#### Configuring Basic Authentication

```yaml
AuthenticationScheme: Basic
AuthenticationConfig:
  LdapRoot:
  Domain: {Active Directory Domain}
```

|Name|Description
|-|-
|LdapRoot|Active Directory LDAP connection string.
|Domain|Active Directory domain name.

#### Authentication Subfolder

Binaries to support extended authentication paradigms are in the [controller-folder]/Authentication subfolder.

### Controller Settings

```yaml
Controller:
  NodeUrl: http://{host:port}/synapse/node
  SignPlan: false
  Assemblies:
  - {list of dynamically loaded assemblies}
  Dal:
    Type: {dal library}
    Config:
```

|Name|Description
|-|-
|NodeUrl|[Optional] When configuring a Controller, populate the URI value for where this Controller sends work by default.  When executing Plans, the Start method allows a Node URL parameter, so it's possible to specify/override this setting per Start call.
|SignPlan|Specifies whether to sign the Plan with a hash value.
|Assemblies|A list of ApiControllers to dynamically load into the Synapse Controller execution space; provides for custom URI implementations.  Specify as library:classname.
|Dal|Specifies the type of Data Access Layer to invoke. See [Controller Data Access Layer](dal "Controller Data Access Layer") for details.

#### Assemblies Subfolder

Locate binaries to support custom ApiControllers in the [controller-folder]/Assemblies subfolder.  Create an additional subfolder per ApiController, as follows: [controller-folder]/Assemblies/[library name].

### Synapse.Server.config.yaml Node Example:

```yaml
Node:
  MaxServerThreads: 0
  AuditLogRootPath: .\Logs
  Log4NetConversionPattern: '%d{ISO8601}|%-5p|(%t)|%m%n'
  SerializeResultPlan: true
  ValidatePlanSignature: true
  ControllerUrl: {optional fixed-URL, default responds to referrer}
```

|Name|Description
|-|-
|MaxServerThreads|Limits concurrency level.  0 = unlimited.
|AuditLogRootPath|The path to which complete, per-Plan execution information is persisted.
|Log4NetConversionPattern|Format specifier for log information.
|SerializeResultPlan|Specifies that, in addition to the audit detail, the Plan.ResultPlan will serialize to a file.
|ValidatePlanSignature|Specifies whether to validate the Plan hash value.  Plans with an invalid hash will be rejected from executing. 
|ControllerUrl|[Optional] When configuring a Node, you may populate the URI value as to what Controller this Node reports to.  By default, Nodes to reply to the referring Controller.


#### Handlers Subfolder

Locate binaries to support Handlers in the [node-folder]/Handlers subfolder.  Create an additional subfolder per Handler, as follows: [controller-folder]/Handlers/[library name].