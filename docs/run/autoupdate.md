# Using AutoUpdate to Update Local Installations of Synapse.Server

Use the AutoUpdater to provide built-in server updates.  Invoking the AutoUpdater will stop the server, download/extract a patch, then optionally restart the server.

## Overview of the AutoUpdater Process

When initiating an autoupdate, Synapse.Server.AutoUpdater.exe follows this workflow:
1. Reads `Synapse.Server.AutoUpdater.yaml` and stops associated services.
2. Reads the `UpdateConfig.xml` sepcified at the `UpdateConfigUri`, gets the version for the current update, and compares to the version on the specified `AutoUpdater.RuntimeExe`.
3. If the `AutoUpdater.RuntimeExe` version is less than `UpdateConfig.CurrentVersion`, the file located at `UpdateConfig.PatchUri` is downloaded and extracted locally.
4. Following the update, services are (optionally) restarted.

<p align="center">
<img alt="Synapse Engine" src="/img/syn_autoupdater.png" />
</p>

## Configuring the AutoUpdater

### AutoUpdate Config File

`Synapse.Server.AutoUpdater.exe` sources `Synapse.Server.AutoUpdater.yaml` for its settings.

|Name|Type/Value|Required|Description
|-|-|-|-
|ServiceConfigs|Array of Strings|No|By default, the AutoUpdater will look for `..\Synapse.Server.config.yaml` to discover server settings.  If the server instance is Configured to use an alternately named configuration file, or if the server is executing in more than instance, list the paths to the config files in this setting.
|UpdateConfigUri|String|Yes|URL or UNC path to the update configuration file.  See [Update Config File](#update-config-file) below for more information.
|RuntimeExe|String|Yes|Path the runtime process.  The required value is `..\Synapse.Server.exe`.
|DownloadFolder|String|Yes|The path where the update files are cached during the update process.
|WaitForExitMillseconds|Integer|Yes|The maximum number of milliseconds to wait before terminating the AutoUpdater.
|StartServicesAfterInstall|Boolean|Yes|Indicates whether to restart the server after AutoUpdate is complete.

#### Example `Synapse.Server.AutoUpdater.yaml` File

```yaml
ServiceConfigs:
- ..\Synapse.Server.config.yaml
UpdateConfigUri:  http:\\ [or] \\UNC\path\to\updates\updateconfig.xml
RuntimeExe: ..\Synapse.Server.exe
DownloadFolder: patches
WaitForExitMillseconds: 30000
StartServicesAfterInstall: false
```

### Update Config File

|Name|Type/Value|Required|Description
|-|-|-|-
|CurrentVersion|String|Yes|The version of `Synapse.Server.exe` in the patch zip file.  This setting is used to compare to the `RuntimeExe` in `Synapse.Server.AutoUpdater.yaml` to determine if an update is necessary.
|IsMandatory|Boolean|No|Not applicable; the AutoUpdater treats all patches as mandatory, installing only when invoked.
|LastMandatoryVersion|String|Yes|Not applicable; the AutoUpdater treats all patches as mandatory, installing only when invoked.
|PatchUri|String|Yes|URL or UNC path to the update patch zip file.
|PatchSizeBytes|Integer|Yes|The size of the update patch zip file, in bytes.  Note: this is the 'Size' property, not the 'Size on Disk'.

Note: When running the Controller/Node from a single folder, the Synapse.Server zip file from GitHub (<a href="https://github.com/SynapseProject/synapse.server.net/releases" target="_blank">https://github.com/SynapseProject/synapse.server.net/releases</a>) may be used as-is.  If running the Controller and Node in separate folders, it may be useful to purge unnecessary files from the zip before listing as an available patch.

#### Example Update Config File

```xml
<UpdateConfig>
  <CurrentVersion>10.2.3.4</CurrentVersion>
  <IsMandatory>false</IsMandatory>
  <LastMandatoryVersion>1.2.3.0</LastMandatoryVersion>
  <PatchUri>http:\\ [or] \\UNC\path\to\update\zipFile.zip</PatchUri>
  <PatchSizeBytes>8675309</PatchSizeBytes>
</UpdateConfig>
```

## Invoking the AutoUpdater

You may run `Synapse.Server.AutoUpdater.exe` directly from the server installation, or invoke it from the Controller/Node URLs, as shown below.

|Verb|URI|Description
|-|-|-
|get|/synapse/[execute/node]/update|Invokes AutoUpdate, which will stop the server, refresh the binaries, and then optionally restart the server.
|get|/synapse/[execute/node]/update/logs|Fetches a list of AutoUpdate logs.
|get|/synapse/[execute/node]/update/logs/{name}|Fetches a specific AutoUpdate log.