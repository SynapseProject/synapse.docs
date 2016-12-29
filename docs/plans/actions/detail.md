# Action Blocks

An Action is a workflow process which is enabled runtime modules, called Handlers.  An Action is executed based on the StatusType of preceding Actions, or in the case of root Actions, 'Any' Status.  The Action Hander is executed at runtime with the Action Parameters, returning an ExecuteResult.  This result is used to determine if child ActionGroup/Actions are in turn executed down the tree.  Actions may be executed locally (on the receiving Synapse node), or remotely via the Proxy attribute.  Lastly, an Action is executed under a given SecurityContext, which is either specified directly or inherited from parent Actions.

## Detailed Field Description

#### Name
The friendly name of the current Action block.

#### Description
A friendly textual description for the current Action block.  This value is not used by the runtime engine, but it is emitted to the ResultPlan.

#### Proxy
The URI of a remote Synapse daemon, used under distributed execution models.

#### ExecuteCase
A list of StatusType values to match the ExecuteResult of a parent Action.

#### Handler
Declares the library to support executing the Action.

#### Parameters
Delares the ParameterInfo block used when invoking the Action.

#### ActionGroup
A grouping mechanism for a child branch of Actions. ActionGroup must complete before child Actions are executed; the ExecuteResult from ActionGroup will be used to filter child Actions.

#### Actions
The list of child Actions.

#### RunAs
The Action-level SecurityContext, overrides Plan-level declaration.

#### Result
Holds the post-execution result of the Action. Rolls-up child execution results to the highest StatusType. Includes runtime PId, Status, ExitData.

#### InstanceId
Local runtime identifier for an Action.

