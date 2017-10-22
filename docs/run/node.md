# Synapse Server Node Web API

The Synapse Node Web API manages Plan runtime execution.  For information about configuration, see [Controller/Node Setup](setup "Controller/Node Setup").

#### About Synapse Node Security

Generally, Synapse Nodes can run as Anonymous, but they should always implement the SignPlan/VerifyPlanSignature construct to guarantee Plans are invoked by authorized users from authorized Controllers.  You may apply additional security setup, such as higher levels of authentication or SSL/TLS, if desired. 

## Executing Plans

The Synapse Node URI is: http://{host:port}/synapse/node.  For detailed method invocation information, see the Swagger URI at: http://{host:port}/swagger/

|Verb|URI|Description
|-|-|-
|get|/synapse/node/hello|Returns "Hello, World!"  Does not invoke a Handler, but does require authentication.
|get|/synapse/node/hello/whoami||Returns a string of the authenticated user context.  Does not invoke a Handler.
|get|/synapse/node/hello/about|Returns the server configuration (`synapse.server.config.yaml`) and an inventory of files for this server instance.
|delete|/synapse/node/{planInstanceId}|Cancels a Plan by planInstanceId.  _Note:_ Plan Handlers must detect a request to Cancel and exit gracefully.  Synapse Node does not forcefully abort Plan execution at this time.
|post|/synapse/node/{planInstanceId}|Starts a Plan using the URI querystring for dynamic parameters.
|post|/synapse/node/{planInstanceId}/p|Starts a Plan with an http post, where dynamic parameters are specified in the http body.
|get|/synapse/node/drainstop|Pauses the Node from accepting new Plans for execution, allows the existing queue of Plans to complete.  Optionally Stops the Service when fully drainstopped.
|get|/synapse/node/drainstop/cancel|Returns the Node to normal execution.  This command is only effective before the queue is fully drained when drainstop->shutdown = true.
|get|/synapse/node/drainstop/iscomplete|Returns the queue drainstop status.
|get|/synapse/node/queue/count|Returns the number of items in the execution queue.
|get|/synapse/node/queue|Returns a list of Plans in the execution queue.
|get|/synapse/node/update|Invokes AutoUpdate, which will stop the server, refresh the binaries, and then optionally restart the server.
|get|/synapse/node/update/logs|Fetches a list of AutoUpdate logs.
|get|/synapse/node/update/logs/{name}|Fetches a specific AutoUpdate log.
