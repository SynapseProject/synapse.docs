# Configuration Options for Arranging Synapse Server Content

## Synapse.Server.Config.yaml

The default name for the server runtime config is `Synapse.Server.Config.yaml`.  Running any of the server exes (synapse.server.exe, synapse.controller.cli.exe, or synapse.node.cli.exe) will generate a Synapse.Server.Config.yaml file if none is already present. Further, the exes will look for a Synapse.Server.Config.yaml by default, as well.  The table below shows the config generation/consumption per exe.  These behaviors are meant to facilitate easy setup/usage of the product for the most common distribution patterns.  You may choose, however, to use Synapse.Server.Config.yaml under a different name/location.

|Exe|YAML Config Content
|-|-|-
|Synapse.Server|Shared + Controller + Node
|Synapse.Controller.cli|Shared + Controller
|Synapse.Node.cli|Shared + Node

### Use-Cases

Generating a custom config supports flexibility in bin re-use or file distribution.  For example, you may choose to:
- Execute multiple processes from a single binary distribution, each using a custom config.
- Locate custom configs outside the application directory, thus simplifying code upgrades.
- Share config across multiple binary distributions.

### Create a new Synapse.Server.Config.yaml

To generate new config file, run any of the exes with the `genconfig` option.  As the table above indicates, the relevant sections of the config will be populated with default values.  Of note, you may specify a new, custom filename when creating the config file.

- `synapse.server genconfig filepath:{path\filename}`
- `synapse.controller.cli genconfig filepath:{path\filename}`
- `synapse.node.cli genconfig filepath:{path\filename}`

### Consuming a custom Synapse.Server.Config.yaml for Install/Uninstall

Synapse.Server accepts a custom filename/path as an install/uninstall parameter, as follows:

- `synapse.server install synapseConfig:{path\filename}`
- `synapse.server uninstall synapseConfig:{path\filename}`

Synapse.Controller and Synapse.Node install/uninstall/run syntax is:

- Controller:
  - `synapse.controller.cli service install synapseConfig:{path\filename}`
  - `synapse.controller.cli service uninstall synapseConfig:{path\filename}`
  - `synapse.controller.cli service run synapseConfig:{path\filename}`
- Node:
  - `synapse.node.cli service install synapseConfig:{path\filename}`
  - `synapse.node.cli service uninstall synapseConfig:{path\filename}`
  - `synapse.node.cli service run synapseConfig:{path\filename}`

For reference, the install action results in a Windows Service runtime similar to the below, where {path\filename} represent a synapseConfig file.

- `synapse.server {path\filename}`

### Consuming a custom Synapse.Server.Config.yaml for Synapse.Controller/Node.cli HttpAction Execution

- Controller:
  - `synapse.controller.cli {httpAction} {parm:value} synapseConfig:{path\filename}`
- Node:
  - `synapse.node.cli {httpAction} {parm:value} synapseConfig:{path\filename}`


## Relocating the Assemblies, Authorization, Dal, and Handlers Folders

To relocate the Assemblies, Authorization, Dal, or Handlers folders, open **each** of the exe .NET config files below and edit the `probing privatePath` setting, updating paths as desired.

- **Important**: you must edit each file below and match the *probing privatePath* setting.

|Exe|.NET Config File
|-|-
|Synapse.Server|Synapse.Server.exe.config
|Synapse.Controller.cli|Synapse.Controller.cli.exe.config
|Synapse.Node.cli|Synapse.Node.cli.exe.config
|Synapse.cli|Synapse.cli.exe.config


### Default Synapse binary resolution configuration:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  ...
  <runtime>
    <assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
      <probing privatePath="Dal;Handlers;Authentication;Assemblies" />
    </assemblyBinding>
    ...
  </runtime>
</configuration>
```