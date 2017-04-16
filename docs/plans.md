#Synapse Plans

## Plans

A Synapse Plan is a declarative workflow that is based on execution-result branching.  Plans are comprised of a series of hierarchical Actions, where each Action optionally specifies an Execution Case.

### Fields

|Name|Description
|-|-
|Name|The friendly name of the Plan, used when reporting status.
|UniqueName|A globally unique name for the Plan to support database fetches.
|Description|A friendly Plan description.
|DefaultHandlerType|If declared, can be used to omit the Type setting under Handler, thus using the Plan DefaultHandlerType.
|IsActive|The boolean enabled/disabled state of the Plan.
|Actions|The list of child Actions.
|RunAs|The overriding, Plan-level SecurityContext.
|Result|Holds the post-execution result all child Actiona.  Rolls-up child execution results to the highest StatusType. Includes runtime PId, Status, ExitData.
|InstanceId|Local runtime identifier for a Plan.

### ResultPlan

The ResultPlan is the output of what Actions executed and their result.  The ResultPlan is useful for examining exact execution paths and validating Plan actions.  Any Actions *not* executed at runtime are omitted from the ResultPlan.  When testing Plans with Synapse.CLI, use the /resultPlan option to output the ResultPlan to a file.  Detailed information on Synapse.CLI can be found [here](/cli/ "Synapse.CLI").

### Example YAML

```yaml
Name: myPlan
UniqueName: myPlan012
Description: Runs important actions on nodes.
DefaultHandlerType: myLibrary.Utilities:myLibrary.Utilities.ServiceController
IsActive: true
Actions:
  {Actions}
RunAs:
  {SecurityContext}
Crypto:
  KeyFile: {RSA key}
  Elements:
  - {list of Plan elements to encrypt}
StartInfo:
  RequestNumber: 12345
  RequestUser: John Doe
Result:
  {runtime data, emitted to ResultPlan}
InstanceId: {runtime data, emitted to ResultPlan}
```

---

## Actions

An Action is a workflow process, which can essentially be anything.  Synapse is extended by creating/invoking new, custom processes as required via runtime modules.  Detailed information on Actions can be found [here](/plans/actions/detail/ "Actions").

### Fields

|Name|Description
|-|-
|Name|The friendly name of the Action, used when reporting status.
|Proxy|The URI of a remote Synapse daemon, used under distributed execution models.
|ExecuteCase|A list of StatusType values to match the ExecuteResult of a parent Action.
|Handler|Declares the library to support executing the Action.
|Parameters|Delares the ParameterInfo block used when invoking the Action.
|ActionGroup|A grouping mechanism for a child branch of Actions.  ActionGroup must complete before child Actions are executed; the ExecuteResult from ActionGroup will be used to filter child Actions.
|Actions|The list of child Actions.
|RunAs|The Action-level SecurityContext, overrides Plan-level declaration.
|Result|Holds the post-execution result of the Action.  Rolls-up child execution results to the highest StatusType.  Includes runtime PId, Status, ExitData.
|InstanceId|Local runtime identifier for an Action.

### Example YAML

```yaml
Name: Start Service
Proxy: http://remoteSynapseNodeUri
ExecuteCase: Success
Handler:
  Type: myLibrary.Utilities:myLibrary.Utilities.ServiceController
  Config: {ParameterInfo}
Parameters: {ParameterInfo}
RunAs: {SecurityContext}
ActionGroup: {Single-node subtree of child Actions}
Actions: {Multi-node subtree of child Actions}
Result:
  {runtime data, emitted to ResultPlan}
InstanceId: {runtime data, emitted to ResultPlan}
```

---

## ParameterInfo

ParameterInfo blocks declare start-up configuration for Handler modules, runtime invocation data for Handler methods, and start-up configuration for SecurityContext modules. ParameterInfo blocks can be inherited throughout Plans, and individual ParameterInfo Value settings can be overridden locally.  Detailed information on Parameters can be found [here](/plans/parms/detail/ "Parameters").

|Name|Description
|-|-
|Name|The friendly name of the ParameterInfo, used when inheriting the values.
|Type|SerializationType: Xml, Yaml, Json
|InheritFrom|The name of another ParameterInfo block.
|Uri|A file or http Uri from which to fetch values.
|Values|Locally declared values.
|Dynamic|List of Name/Path pairs used in value substitution.  Paths are declared in XPath for Xml serialization and colon-separated lists (root:node0:node1:...) for Yaml/Json.
|ForEach|Calculates the cartesian product of the declared blocks of values and expands the action into a set of actions for each result item.  ActionGroup and child Actions relationships are maintained and will be executed per result item, as well.  Of note, as both Action.Handler.Config and Action.Parameters can be declared with ForEach blocks, the total execution iterations for an Action is the carstesian product of the expanded Config and expanded Parameters.   

### Example YAML - YAML Values

```yaml
Name: myYamlParms
Type: Yaml
Uri: http://foo
Values:
  Custom: Data
  DefinedBy: Requirements
  Of:
    Handler: Module
    Expected: Input
Dynamic:
- Name: app
  Path: Custom
- Name: type
  Path: Of:Handler
ForEach:
- Path: DefinedBy
  Values:
  - x0
  - x1
  - x2
- Path: Of:Expected
  Values:
  - y0
  - y1
  - y2
```

### Example YAML - XML Values

```yaml
Type: Xml
Values: <xml attr="value1"><data>foo1</data></xml>
Dynamic:
- Name: app
  Path: /xml[1]/data[1]
- Name: type
  Path: /xml[1]/@attr
```

### Example YAML - JSON Values

```yaml
Type: Json
Uri: http://remoteSynapseNodeUri
Values:
  {
    "ApplicationName": "fooApp",
    "EnvironmentName": "dev1",
    "Tier": {
        "Name": "webserver1",
        "Type": "python1",
        "Version": "1.0"
    },
  }
Dynamic:
- Name: app
    Path: ApplicationName
    Options:
    - Key: 1
      Value: barApp
    - Key: 2
      Value: blahApp
- Name: type
  Path: Tier:Type
```