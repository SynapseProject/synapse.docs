# Synapse Plans: Action Handlers

## Handlers

A Handler declares the library to support executing the current Action.  A Handler declaration consists of naming the Handler Type and specifying its Config.  A detailed explanation of Handlers can be found in the `Plans` section of the docs, [here](/plans/handlers/ "Handlers").

This section of of the docs presents Handler-specific documentation.  Recall that you may use the `syanpse.cli` to generate a sample `Plan` for any given Handler, and then use the docs below to understand configuration options.

```dos
synapse.cli.exe sample:{handlerLib:handlerName,...} [out:{filePath}] [verbose:true|false]

C:\Synapse\Node>synapse.cli.exe sample:Synapse.Handlers.CommandLine:all

```