# Synapse Server

## Installation

Installing and configuring Synapse.Server is accompished via the CLI ([Controller](/cli/controller/ "Controller command-line"), [Node](/cli/node/ "Node command-line"), [Server](/cli/server/ "Server command-line")).  In all cases, the code distribution is the same - you're simply choosing to run either as a Controller, Node, or both.  The important settings for determining role are in `Synapse.Server.config.yaml`.  When extracting Synapse Server for setup, no Synapse.Server.config.yaml is present by default.  Synapse will generate a new, complete Synapse.Server.config.yaml when you install the service, or by running the Controller/Node/Server CLIs.

### To install Synapse.Server:

1. Download the latest build from GitHub: <a href="https://github.com/SynapseProject/synapse.server.net/releases" target="_blank">https://github.com/SynapseProject/synapse.server.net/releases</a>.
2. Extract the contents of the zip into one folder for a "dual" install (Controller/Node running in a single process), or two folders (by convention: .\Controller & .\Node) to run as separate processes.  For a distributed-node deployment, extract the contents in remote locations, as desired.
3. Run `synapse.server install [run:true|false]` in the single folder, or `synapse.controller.cli install [run:true|false]` and `synapse.node.cli install [run:true|false]` in the separate folders.

### Notes:
* Optionally, you may execute `Synapse.Controller.setup.cmd`/`Synapse.Node.setup.cmd`.  These run the install commands as shown above, and also delete unnecessary files/folders in a split deployment.
* **Important**: Each CLI above will generate a Synapse.Server.config.yaml file if none is present, pre-configured for the installation option of choice.  However, if a Synapse.Server.config.yaml is already present, the installation will use the existing file, as-is, to install/execute the service.

## Configuration

Synapse.Server.config.yaml contains the settings that determine role and behavior of the installation.  Settings are read at server startup; if you change the settings, you need to restart Synapse.Server.exe for the changes to take effect.  The settings are divided into three sections, as shown below: Common (applies to both Controller and Node), Controller (-specific), and Node (-specific).

### Common Settings

```yaml
Service:
  Name: Synapse.Server
  DisplayName: Synapse Server
  Role: Server
WebApi:
  Host: localhost
  Port: 20000
  IsSecure: false
  Authentication:
    Scheme: IntegratedWindowsAuthentication
    Config: 
      {authentication provider initialization config}
Signature:
  KeyUri: .\path\RsaKeyFileName.xml
  KeyContainerName: 
  CspProviderFlags: NoFlags
```

|Name|Description
|-|-
|Service:Name & :DisplayName|These values are only used when installing Synapse.Server as a Windows Service.  Provide uniquely named values to run multiple instances of the services side-by-side.  The default values shown in the examples below will be modified to reflect Synapse.Controller or Synapse.Node (depending on role) under a default installation.  Alternatively, you may specify values directly.   **Note:** Each unique instance of Synapse.Server should be distributed to an independent folder.  **Note:** Do not modify these values _after_ installation.  You must first _uninstall_ the service, then modify and re-install.
|Service:Role|`Controller` or `Node` or `Server`
|Web:ApiPort|Select a port value.
|WebApi:IsSecure|Indicates if service is configured for SSL/TLS.
|Authentication:Scheme|Authentication  options.  See <a href="https://msdn.microsoft.com/en-us/library/system.net.authenticationschemes(v=vs.110).aspx" target="_blank">MSDN</a> for details.
|Authentication:Config|Proprietary configuration for specific authentication providers.  See [Configuring Basic Authentication] below.
|Signature:KeyUri|Filepath to RSA key file.  For a Controller instance, the keyfile must contain public and prvite key values.  For a Node instance, the keyfile must contain only the public key value.  If running both the Controller and Node from a single distribution, the keyfile must contain public and prvite key values.  To generate new RsaKeyFiles, use synapse.controller.cli *genkeys* option.  See CLI help for details.
|Signature:KeyContainerName|The name of the container in which to look for key values.  A keyfile may contain multiple containers.
|Signature:CspProviderFlags|CspProviderFlags options.  See <a href="https://msdn.microsoft.com/en-us/library/system.security.cryptography.cspproviderflags(v=vs.110).aspx" target="_blank">MSDN</a> for details.

#### Configuring Basic Authentication

```yaml
  Authentication:
    Scheme: Basic
    Config:
      LdapRoot: LDAP://...
      Domain: {Active Directory Domain}
```

|Name|Description
|-|-
|LdapRoot|Active Directory LDAP connection string.
|Domain|Active Directory domain name.

#### Authentication Subfolder

Binaries to support extended authentication paradigms are in the [controller-folder]/Authentication subfolder.

### Controller Settings

Use `synapse.controller.cli install [run:true|false]` to install.

 - If a `synapse.server.config.yaml` file is present, the installer will use the settings in the file, as-is. 
 - If no `synapse.server.config.yaml` file is present, the installer will create a file with default values.

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
|NodeUrl|[Optional] When configuring a Controller, populate the URI value for where this Controller sends work by default.  When executing Plans, the Start method allows a Node URL parameter, so it's possible to specify/override this setting per Start call.  **Note:** If you do not specify a default setting here, you must supply a NodeUrl per Plan execution.
|SignPlan|Specifies whether to sign the Plan with a hash value.
|Assemblies|A list of ApiControllers to dynamically load into the Synapse Controller execution space; provides for custom URI implementations.  Specify as library:classname.
|Dal|Specifies the type of Data Access Layer to invoke. See [Controller Data Access Layer](dal "Controller Data Access Layer") for details.

#### Assemblies Subfolder

Locate binaries to support custom ApiControllers in the [controller-folder]/Assemblies subfolder.  Create an additional subfolder per ApiController, as follows: [controller-folder]/Assemblies/[library name].

### Node Settings:

Use `synapse.node.cli install [run:true|false]` to install.

 - If a `synapse.server.config.yaml` file is present, the installer will use the settings in the file, as-is. 
 - If no `synapse.server.config.yaml` file is present, the installer will create a file with default values.

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

### Install Synapse Server as "Dual"

You may install Synapse Server under a single folder, supporting both the Controller and the Node, as follows:

Use `synapse.server.exe install [run:true|false]` to install.

 - If a `synapse.server.config.yaml` file is present, the installer will use the settings in the file, as-is. 
 - If no `synapse.server.config.yaml` file is present, the installer will create a file with default values.

