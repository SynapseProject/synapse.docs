# Synapse Core CommandLine

Synapse.cli provides a way to test Syanpse Plans locally and is the semantic equivalent of Synapse.Node.  Output from Synapse.cli is redirected to the prompt by default, instead of to a log.

Download the latest build of Synapse.cli from GitHub: <a href="https://github.com/SynapseProject/synapse.core.net/releases" target="_blank">https://github.com/SynapseProject/synapse.core.net/releases</a>.

Synapse.cli is a wrapper on Syanpse.Core, which is available to download as a NuGet package: <a href="https://www.nuget.org/packages/Synapse.Core.Signed" target="_blank">https://www.nuget.org/packages/Synapse.Core.Signed</a>.  Syanpse.Core is suitable for programmatic Synapse integration.

## CommandLine Help:

```dos
synapse.cli.exe, Version: 0.1.0.0

Syntax:
  synapse.cli.exe /plan:{filePath} [/dryRun:true|false]
    [/resultPlan:{filePath}|true] [dynamic parameters]

  /plan        - filePath: Valid path to plan file.
  /dryRun      Specifies whether to execute the plan as a DryRun only.
                 Default is false.
  /resultPlan  - filePath: Valid path to write ResultPlan output file.
               - [or]: 'true' will write to same path as /plan as *.result.*
  dynamic      Any remaining /arg:value pairs will passed to the plan
                 as dynamic parms for use in Plan Config/Param substitution.
```