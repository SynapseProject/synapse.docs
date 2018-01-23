# Synapse Plans: Action Handlers

## Handlers

A Handler declares the library to support executing the current Action.  A Handler declaration consists of naming the Handler Type and specifying its Config, which is a [ParameterInfo block](/plans/parms/ "Parameters").

### Fields

|Name|Type|Required|Description
|-|-|-|-
|Type|string|Yes|Declares the library and class to support executing the current Action.
|Config|[ParameterInfo](/plans/parms/ "Parameters")|No|Delares the ParameterInfo block used when invoking the current Action.

### Example YAML

```yaml
Handler:
  Type: myLibrary.Utilities:myLibrary.Utilities.ServiceController
  Config: {ParameterInfo}
```

#### Type

The Handler Type is specified in two parts, separated by a colon: 1) library name, and 2) class name.  The library name is the simple name; do not specify the full path, and do not include the file extension.

Handler Type declaration supports several forms of full and abbreviated syntax, as shown in the table below.  Additionally, you may even omit Type and use the Plan default type.  On Windows systems, the default handler type is `Synapse.Handler.CommandLine:CommandLineHandler`.  You may override the Plan default handler type by setting Plan.DefaultHandlerType, in which case the declared value will be used in place of omitted Action.Handler.Type values.

|Style|Description
|-|-
|Full|Library:FullyQualifiedClassName
|FullyQualifiedClassName|Omits the Library name.
|SimpleClassName|Omits the Library name, and omits the Class namespace.
|AbbreviatedClassName|Omits the Library name, the Class namespace, and the 'Handler' suffix from the Class.
|(None)|Omits the entire Type declaration; uses Plan.DefaultHandlerType.

### Example YAML - Handler Type Declaration

In the Plan below, all declarations result in the same Type being invoked: Synapse.Core:Synapse.Core.Runtime.EmptyHandler.

```yaml
Name: plan0
Description: Load handlers with full or partial names.
DefaultHandlerType: Synapse.Core:Synapse.Core.Runtime.EmptyHandler
Actions:
- Name: FullName-Library:FullyQualifiedClassName
  Handler:
    Type: Synapse.Core:Synapse.Core.Runtime.EmptyHandler
- Name: FullyQualifiedClassName
  Handler:
    Type: Synapse.Core.Runtime.EmptyHandler
- Name: SimpleClassName
  Handler:
    Type: EmptyHandler
- Name: AbbreviatedClassName
  Handler:
    Type: Empty
- Name: NoHandlerSpecified-Use:Plan.DefaultHandlerType
```

## Discovering Handler Config/Parameters Layout
Using [Synapse.Core CLI](/cli/core/ "Synapse CLI") to discover Handler Config/Parameters is accomplished via the `sample` parameter, as follows:

```dos
 synapse.cli.exe sample:{handlerLib:handlerName,...} [out:{filePath}]
   [verbose:true|false]

  - Create a sample Plan with the specified Handler(s).

  sample       - A csv list of handlerLib:handlerName pairs.
  out          - filePath: Optional output filePath.
               If [out] not specified, will output to screen.
  verbose      - If true, adds example values for all Plan options.
```

### Example - Discovering Handler Config/Parameters
```dos
C:\synapse\>synapse.cli.exe sample:Synapse.Core:all out:mySample.yaml verbose:true
```