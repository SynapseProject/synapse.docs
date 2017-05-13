# Synapse Controller CommandLine

Synapse.Controller.cli provides a console interface to Synapse Controller, whether Synapse Controller is running locally or remotely, and whether it is running as a proper server daemon or console hosted process.  Synapse.Controller.cli can be used as an administrative interface for runtime operations as a way to host Synapse Controller for testing purposes.

Download the latest build of Synapse.Controller.cli from GitHub: <a href="https://github.com/SynapseProject/synapse.server.net/releases" target="_blank">https://github.com/SynapseProject/synapse.server.net/releases</a>.

Synapse.Controller.cli is a wrapper on Syanpse.Server.HttpClient, invoking the Controller REST interface.  Synapse.Server.HttpClient is available to download as a NuGet package: <a href="https://www.nuget.org/packages/Synapse.Server.HttpClient" target="_blank">https://www.nuget.org/packages/Synapse.Server.HttpClient</a>.  Syanpse.Server.HttpClient is suitable for programmatic Synapse integration.

## Service Runtime Support

|Command|Description
|-|-
|install|Installs Syanpse.Server pre-configured as Controller
|uninstall|Uninstalls this instance of Syanpse.Server
|run|Runs Syanpse.Server as cmdline daemon, pre-configured as Controller

## CommandLine Help:

```dos
synapse.controller.cli.exe, Version: 0.1.0.0

Syntax:
  synapse.controller.cli.exe service {command} | {httpAction parm:value} |
       interactive|i [url:http://{host:port}/synapse/execute]

  About URLs:  URL is an optional parameter on all commands except 'service'
               commands. Specify as [url:http://{host:port}/synapse/execute].
               URL default is localhost:{port} (See WebApiPort in config.yaml)

  interactive  Run this CLI in interactive mode, optionally specify URL.
               All commands below work in standard or interactive modes.

  service      Install/Uninstall the Windows Service, or Run the Service
               as a cmdline-hosted daemon.
               - Commands: install [run:true|false] | uninstall | run
               - Example:  synapse.controller.cli service install run:false
                           synapse.controller.cli service run

  keygen       Generate RSA key for signing Plans.
               - keyContainerName:  Key values storage Container.
               - filePath:          Path and filename to store key values.

  httpAction   Execute a command, optionally specify URL.
               Parm help: synapse.controller.cli {httpAction} help.

  - httpActions:
    - Hello|hi           Returns 'Hello, World!'.
    - WhoAmI|who         Returns ControllerServer User Context.
    - List|l             Get a list of Plans.
    - ListInstances|li   Get a list of Plans Instances.
    - Start|s            Start a new Plan Instance.
    - GetStatus|gs       Get the Status for a Plan Instance.
    - SetStatus|ss       Set the Status for a Plan Instance.
    - Cancel|c           Cancel a Plan Instance.

  Examples:
    synapse.controller.cli l url:http://somehost/synapse/execute
    synapse.controller.cli li help
    synapse.controller.cli li planName:foo url:http://somehost/synapse/execute
    synapse.controller.cli li planName:foo
    synapse.controller.cli i url:http://somehost/synapse/execute
    synapse.controller.cli i
```