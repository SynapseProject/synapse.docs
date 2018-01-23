# Synapse Plans: Action Blocks

An Action is a workflow process which is enabled by runtime modules, called Handlers.  An Action is executed based on the Status of preceding Actions, or in the case of root Actions, 'Any' Status.  The Action Handler is executed at runtime with the Action Parameters, returning an ExecuteResult.  This result is used to determine if child ActionGroup/Actions are in turn executed down the tree.  Actions may be executed locally (on the receiving Synapse node), or remotely via the Proxy attribute.  Lastly, an Action is executed under a given SecurityContext, which is either specified directly or inherited from parent Actions.

```yaml
- Name: Sample Action
  Description: Sample Action friendly description.
  Proxy: 'Future-use: http://host:port/synapse/node'
  ExecuteCase: Complete
  SaveExitDataAs:
    Config: ConfigSet Name
    Parameters: ParameterSet Name
  Handler: {}
  RunAs: {}
  Parameters: {} 
  ActionGroup: {}
  Result: {}
```

## Overview

|Field | Description
|-|-
|`Name`|The friendly name of the Action.  This value is not used by the runtime engine, but it is emitted to the ResultPlan.
|`Description`|A friendly textual description for the current Action block.  This value is not used by the runtime engine, but it is emitted to the ResultPlan.
|`Proxy`|_Reserved for future use.  Will support distributed execution of Plans across daisy-chained Nodes._
|`ExecuteCase`|A filter for whether the current Action will execute; interrgated againt the parent Action's Result.Status.
|`SaveExitDataAs`|Persists this Action's Result.ExitData as a global variable in the Plan.
|`Handler`|Declaration for the technology-worker that fulfills the Action. [[More info.](/plans/handlers/ "Handlers")]
|`RunAs`|Provides a mechanism for an alternate security context for the current Action. [[More info.](/plans/runas/ "RunAs")]
|`Parameters`|The runtime invocation data to the Action.  [[More info.](/plans/parms/ "Parameters")]
|`ActionGroup`|A child tree of Actions that are executed as a single dependency chain and whose results are combined and returned as a single Action(Group).Result.Status.
|`Actions`|A child tree of Actions that are executed as independent items in parallel and whose results are returned as a individual Action.Result.Status values.  Each of these Results is ultimately combined into the currents Actions Result.BranchStatus.
|`Result`|The utimate result for the currect Action, including the result from the ActionGroup.


## Detailed Field Description

### Name
The friendly name of the current Action block.

### Description
A friendly textual description for the current Action block.  This value is not used by the runtime engine, but it is emitted to the ResultPlan.

### Proxy
*[Note: This feature is not yet available.]*

The URI of a remote Synapse daemon, used under distributed execution models.  Under Proxy execution, the Action subtree is detached and forwarded to the remote node, executed, and the summary status is returned.  Synapse nodes can be daisy-chained in strings longer than two nodes, or they can be executed in a hub/spoke model.  Logs/Status from remote noes are forwaded back to the orginal caller in breadcurmbs fashion such that total Plan continuity is maintained.

### ExecuteCase
A list of StatusType values to match the ExecuteResult of a parent Action.  ExecuteCase is evaluated based on the parent Action Result.

```yaml
Name: ExecuteCasePlan
Description: ExecuteCase Example
DefaultHandlerType: Synapse.Core:EchoHandler
Actions:
- Name: MyRoot
  Description: Root actions of Plans are always executed.
  Parameters:
    Values:
      MyValue: Action_Value0
  ActionGroup:
    Name: Group0
    Description: As ExecuteCase is 'Any', this will execute no matter the result of MyRoot
    ExecuteCase: Any
    Parameters:
      Values:
        MyValue: Group0_Value0
    Actions:
    - Name: Group0_Child0
      Description: Executes only if Group0.Result is 'Failed'
      ExecuteCase: Failed
      Parameters:
        Values:
          MyValue: Child0_Value0
    - Name: Group0_Child1
      Description: Executes only if Group0.Result is 'Complete'
      ExecuteCase: Complete
      Parameters:
        Values:
          MyValue: Child1_Value0
  Actions:
  - Name: MyRoot_Child0
    Description: Executes only if MyRoot.Result is 'Failed'
    ExecuteCase: Failed
    Parameters:
      Values:
        MyValue: MyRoot_Child0_Value0
  - Name: MyRoot_Child1
    Description: Executes only if MyRoot.Result is 'Complete'
    ExecuteCase: Complete
    Parameters:
      Values:
        MyValue: MyRoot_Child1_Value0
```

 - Represented as a flowchart, the above Plan looks like this:

<p align="center">
<img alt="Synapse Action ExecuteCase Flowchart" src="../../../img/action_executeCaseFlowchart.png" />
</p>

### SaveExitDataAs
Persists this Action's Result.ExitData as a global variable in the Plan, thus making it available to descendent Actions' Config/Parameters sections via ParameterInfo.InheritFrom or ForEach.ParameterSource.

```yaml
  SaveExitDataAs:
    Config: ConfigSet Name
    Parameters: ParameterSet Name
```

|Name|Type/Value|Required|Description
|-|-|-|-
|Config|String|No|The key name for ExitData values when persisted in the global Configs collection.
|Parameters|String|No|The key name for ExitData values when persisted in the global Parameters collection.

### Handler
Declares the library to support executing the Action.  Detailed information on Handlers can be found [here](/plans/handlers/ "Handlers").

### Parameters
Declares the ParameterInfo block used when invoking the Action.  Detailed information on Handlers can be found [here](/plans/parms/ "Parameters").

#### Discovering Handler Config/Parameters Layout
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

### ActionGroup
A grouping mechanism for a child branch of Actions. For a given parent Action, its ActionGroup must complete before child Actions are executed; the ExecuteResult from ActionGroup will be used to filter child Actions.  ActionGroups provide a mechanism for ensuring a subset of actions complete before continuing to further descendent Actions.  Any Action, including the ActionGroup itself, can host an ActionGroup.

In the graphic above, MyRoot hosts an ActionGroup named Group0.  The execution order will be MyRoot --> Group0 --> Group0_Child[x] (filtered on Group0 Result) ==> MyRoot_Child (filtered on the status of MyRoot, as propagated upwardly from Group0_Child[x] --> Group0 --> MyRoot).

### Actions
The list of child Actions.  Sibling Action nodes are executed in parallel and carry independent status (node status).  The highest status value of a sibling set is propagated upward as the Result.BranchStatus.

### RunAs
The Action-level SecurityContext, overrides Plan-level declaration.  Detailed information on RunAs can be found [here](/plans/runas/ "RunAs").

### Result
Holds the post-execution result of the Action. Rolls-up child execution results to the highest StatusType. Includes runtime PId, Status, ExitData.

- Status: The local, node status.  This status is used for filtering execution of child Actions.
- BranchStatus: The highest status value from descendent nodes on the branch.  This value is carried primarily for reporting clarity.

```yaml
Result:
  PId: 12345
  Status: Complete
  ExitCode: 1
  ExitData: Custom data as returned from Handler
  BranchStatus: CompletedWithErrors
  Sequence: 1
  Message: Custom exit message from Handler
  SecurityContext: runtime user
```

- In the diagram below, the Status (node status) is shown on the lower left of the Action nodes with arrows pointing downward.  The BranchStatus is shown on the upper right of the Action nodes with arrows pointing upward.
    - The Status path shows which nodes are executed: blue nodes always execute, green nodes execute based on parent node status.
    - The BranchStatus values are gained by upward propagation from child node execution.
    - Execution of Group0 completes first, propagating its status upward to MyRoot.  Execution of MyRoot_Child1 follows on the 'Complete' path.
    - The MyRoot BranchStatus value of 'Completed' is ultimately overriden by 'CompletedWithErrors', as 'CompletedWithErrors' has a higher value than 'Complete'.
    - Lastly, note that the Propagation path of 'CompletedWithErrors' is directly from MyRoot_Child1 --> MyRoot; the status would not traverse Group0_Child1 or Group0 as they are not on the ascending branch path.

<p align="center">
<img alt="Synapse Action Result.Status Propagation" src="../../../img/action_ResultStatusPropagation.png" />
</p>

### InstanceId
Local runtime identifier for an Action, used for tracking status in the local database.

