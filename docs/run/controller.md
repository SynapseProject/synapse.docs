# Synapse Server Controller Web API

The Synapse Controller Web API manages enforcing the RBAC, starting, cancelling, and recording/fetching Plan status.  For information about configuration, see [Controller/Node Setup](setup "Controller/Node Setup").

## Executing Plans

The Synapse Controller URI is: http://{host:port}/synapse/execute.  For detailed method invocation information, see the Swagger URI at: http://{host:port}/swagger/

|Verb|URI|Description
|-|-|-
|get|/synapse/execute/hello|Returns "Hello, World!"  Does not invoke RBAC or DAL, but does require authentication.
|get|/synapse/execute/hello/whoami|Returns a string of the authenticated user context.  Does not invoke RBAC or DAL.
|get|/synapse/execute|Returns a list of Plans.
|get|/synapse/execute/{planUniqueName}|Returns a list of Plan Instance Ids.
|get|/synapse/execute/{planUniqueName}/start|Execute a Plan using the URI querystring for dynamic parameters. Returns planInstanceId.
|post|/synapse/execute/{planUniqueName}/start|Execute a Plan with an http post, where dynamic parameters are specified in the http body. Returns planInstanceId.
|delete|/synapse/execute/{planUniqueName}/{planInstanceId}|Cancel a Plan by planInstanceId.
|get|/synapse/execute/{planUniqueName}/{planInstanceId}|Get the status of a Plan by planInstanceId.
|post|/synapse/execute/{planUniqueName}/{planInstanceId}|Update the status of the entire Plan, by planInstanceId.
|post|/synapse/execute/{planUniqueName}/{planInstanceId}/action|Update the status of an individual Action within a Plan, by planInstanceId.
|get|/synapse/execute/{planUniqueName}/{planInstanceId}/part|Select an individual element from within a Plan, specifing the desired return data serialization format.  See [Using the Part Interface] below for details.
|post|/synapse/execute/{planUniqueName}/{planInstanceId}/part|Select one or more individual elements from within a Plan, specifing the desired return data serialization format.  See [Part] below for details.


## Using the Part Interface

The Synapse Controller /part/ interface is designed to allow selection of one or more individual Plan elements, serialized into a specific format.  This is useful when using Synapse a data service, or when seeking efficiency of data interrogation.

### Syntax

Selecting Plan elements requires a rooted path, expressed asshown below.  If an element is part of a list, use a index specifier.

- element:element:element:etc.
- element[index]:element:element[index]:element:etc.

Examples:

- Result:BranchStatus
- Actions[0]:Result:ExitData

### Serialization Format

To return "native" data formats from the Synapse Controller API, specify the serialization format as:

- YAML
- JSON
- XML
- Unspecified

YAML/JSON/XML will attempt to parse the element data into the specified format before returing it.  "Unspecified" will return the data, as-is.