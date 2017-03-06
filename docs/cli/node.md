# Synapse Node CommandLine

Synapse.Node.cli provides a console interface to Synapse Node, whether Synapse Node is running locally or remotely, and whether it is running as a proper server daemon or console hosted process.  Synapse.Node.cli can be used as an administrative interface for runtime operations as a way to host Synapse Node for testing purposes.

Download the latest build of Synapse.Node.cli from GitHub: <a href="https://github.com/SynapseProject/synapse.server.net/releases" target="_blank">https://github.com/SynapseProject/synapse.server.net/releases</a>.

Synapse.Node.cli is a wrapper on Syanpse.Server.HttpClient, invoking the Node REST interface.  Synapse.Server.HttpClient is available to download as a NuGet package: <a href="https://www.nuget.org/packages/Synapse.Server.HttpClient" target="_blank">https://www.nuget.org/packages/Synapse.Server.HttpClient</a>.  Syanpse.Server.HttpClient is suitable for programmatic Synapse integration.

## CommandLine Help:

```dos
synapse.node.cli.exe, Version: 0.1.0.0

Syntax:
  synapse.node.cli.exe service {command} | {httpAction parm:value} |
       interactive|i [url:http://{host:port}/synapse/node]

  About URLs:  URL is an optional parameter on all commands except 'service'
               commands. Specify as [url:http://{host:port}/synapse/node].
               URL default is localhost:{port} (See WebApiPort in config.yaml)

  interactive  Run this CLI in interactive mode, optionally specify URL.
               All commands below work in standard or interactive modes.

  service      Install/Uninstall the Windows Service, or Run the Service
               as a cmdline-hosted daemon.
               - Commands: install|uninstall|run
               - Example:  synapse.controller.cli service run
               - Optional install args, use argname:value.  Defaults shown.
                 - ServiceName:Synapse.[Controller/Node]
                 - ServiceDisplayName:Synapse [Controller/Node]
                 - ServerRole:Controller
                 - WebApiPort:8008
                 - AuthenticationScheme:IntegratedWindowsAuthentication
                 - SignatureKeyFile:
                 - SignatureKeyContainerName:DefaultContainerName
                 - SignatureCspProviderFlags:NoFlags
                 - c.NodeUrl:http://localhost:8000/synapse/node
                 - c.Dal:Synapse.Controller.Dal.FileSystem:FileSystemDal
                 - c.SignPlan:True
                 - n.MaxServerThreads:0
                 - n.AuditLogRootPath:.\Logs
                 - n.Log4NetConversionPattern:%d{ISO8601}|%-5p|(%t)|%m%n
                 - n.SerializeResultPlan:True
                 - n.ValidatePlanSignature:True
                 - n.ControllerUrl:http://localhost:8008/synapse/execute
                 - Run:true  (Optionally Starts the Windows Service)


  httpAction   Execute a command, optionally specify URL.
               Parm help: synapse.node.cli {httpAction} help.

  - httpActions:
    - Hello|hi           Returns 'Hello, World!'.
    - WhoAmI|who         Returns NodeServer User Context.
    - Start|s            Start a new Plan Instance.
    - Cancel|c           Cancel a Plan Instance.
    - Drainstop|dst      Prevents the node from receiving incoming requests;
                         allows existing threads to complete. Optionally stops
                         the Service when queue is fully drained.
    - DrainStatus|dss    Returns true/false on whether queue is fully drained.
    - QueueDepth|qd      Returns the number of items remaining in the queue.
    - QueueItems|qi      Returns the list of items remaining in the queue.
    - Undrainstop|ust    Resumes normal request processing.

  Examples:
    synapse.node.cli hi url:http://somehost/synapse/node
    synapse.node.cli s help
    synapse.node.cli s planInstanceId:0 dryRun:false filePath:C:\planFile.yaml
    synapse.node.cli dst url:http://somehost/synapse/node
    synapse.node.cli i url:http://somehost/synapse/node
    synapse.node.cli i
```