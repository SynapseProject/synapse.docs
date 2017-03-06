# Synapse Server

Installing and configuring Synapse.Server is accompished via the CLI ([Controller](/cli/controller/ "Controller command-line"), [Node](/cli/node/ "Node command-line")).  In either case, the code distribution is the same - you're simply choosing to run either as a Controller or Node.  The important settings for determining role are in `Synapse.Server.config.yaml`.  Of note: coordinate the following settings:

### Settings
|Name|Description
|-|-
|ServerRole|`Controller` or `Node`
|WebApiPort|Select a port value.
|Controller --> NodeUrl|When configuring a Controller, populate the URI value for where this Controller sends work by default.
|Node --> ControllerUrl|When configuring a Node, populate the URI value as to what Controller this Node reports to.

### Additional Settings
|Name|Description
|-|-
|ServiceName & ServiceDisplayName|These values are only used when Installing Synapse.Server as a Windows Service.  Provide uniquely named values to run multiple instances of the services side-by-side.  The default values shown in the examples below will be modified to reflect Synapse.Controller or Synapse.Node (depending on role) under a default installation.  Alternatively, you may specify values directly.   _Note:_ Each unique instance of Synapse.Server should be distributed to an independent folder.  _Note:_ Do not modify these values _after_ installation.  You must first _uninstall_ the service, then modify and re-install.
|SignatureKeyFile|Filepath to RSA key file.  For a Controller instance, the keyfile must contain public and prvite key values.  For a Node instance, the keyfile must contain only the public key value.  If running both the Controller and Node from a single distribution, the keyfile must contain public and prvite key values. 
|SignatureKeyContainerName|The name of the container in which to look for key values.  A keyfile may contain multiple containers.
|SignatureCspProviderFlags|CspProviderFlags options.  See <a href="https://msdn.microsoft.com/en-us/library/system.security.cryptography.cspproviderflags(v=vs.110).aspx" target="_blank">MSDN</a> for details.
|Controller --> Dal|Specifies the type of Data Access Layer to invoke.
|Controller --> SignPlan|Specifies whether to sign the Plan with a hash value.
|Node --> MaxServerThreads|Limits concurrency level.  0 = unlimited.
|Node --> AuditLogRootPath|The path to which complete, per-Plan execution information is persisted.
|Node --> Log4NetConversionPattern|Format specifier for log information.
|Node --> SerializeResultPlan|Specifies that, in addition to the audit detail, the Plan.ResultPlan will serialize to a file.
|Node --> ValidatePlanSignature|Specifies whether to validate the Plan hash value.  Plans with an invalid hash will be rejected from executing. 

### Synapse.Server.config.yaml Controller Example:

```yaml
ServiceName: Synapse.[Controller/Node]
ServiceDisplayName: Synapse [Controller/Node]
ServerRole: Controller
WebApiPort: 20000
AuthenticationScheme: IntegratedWindowsAuthentication
SignatureKeyFile: .\Keys\publicAndPrivateKeys.xml
SignatureKeyContainerName: DefaultKeyContainer
SignatureCspProviderFlags: NoFlags
Controller:
  NodeUrl: http://localhost:20001/synapse/node
  Dal: Synapse.Controller.Dal.FileSystem:FileSystemDal
  SignPlan: true
Node:
  MaxServerThreads: 0
  AuditLogRootPath: .\Logs
  Log4NetConversionPattern: '%d{ISO8601}|%-5p|(%t)|%m%n'
  SerializeResultPlan: true
  ValidatePlanSignature: true
  ControllerUrl: http://localhost:20000/synapse/execute
```

### Synapse.Server.config.yaml Node Example:

```yaml
ServiceName: Synapse.[Controller/Node]
ServiceDisplayName: Synapse [Controller/Node]
ServerRole: Node
WebApiPort: 20001
AuthenticationScheme: IntegratedWindowsAuthentication
SignatureKeyFile: .\Keys\publicKeyOnly.xml
SignatureKeyContainerName: DefaultKeyContainer
SignatureCspProviderFlags: NoFlags
Controller:
  NodeUrl: http://localhost:20001/synapse/node
  Dal: Synapse.Controller.Dal.FileSystem:FileSystemDal
  SignPlan: true
Node:
  MaxServerThreads: 0
  AuditLogRootPath: .\Logs
  Log4NetConversionPattern: '%d{ISO8601}|%-5p|(%t)|%m%n'
  SerializeResultPlan: true
  ValidatePlanSignature: true
  ControllerUrl: http://localhost:20000/synapse/execute
```