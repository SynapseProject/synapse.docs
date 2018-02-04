# Synapse Server Admin Web API

The Synapse Admin Web API manages AutoUpdate and provides an interface for viewing configuration and fetching log files.

## Executing Admin Functions

The Synapse Admin URI is: http://{host:port}/synapse/admin.  For detailed method invocation information, see the Swagger URI at: http://{host:port}/swagger/

|Verb|URI|Description
|-|-|-
|get|/synapse/admin/hello|Returns "Hello, World!"  Does not invoke RBAC or DAL, but does require authentication.
|get|/synapse/admin/hello/whoami|Returns a string of the authenticated user context.  Does not invoke RBAC or DAL.
|get|/synapse/admin/hello/about|Returns the server configuration (`synapse.server.config.yaml`) and an inventory of files for this server instance.
|get|/synapse/admin/update|Invokes AutoUpdate, which will stop the server, refresh the binaries, and then optionally restart the server.
|get|/synapse/admin/update/logs|Fetches a list of AutoUpdate logs.
|get|/synapse/admin/update/logs/{name}|Fetches a specific AutoUpdate log.
|get|/synapse/admin/logs|Fetches a list of runtime logs.
|get|/synapse/admin/logs/{name}|Fetches a specific runtime log.


