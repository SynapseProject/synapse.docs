
## Overview  
The Representational State Transfer (REST) Handler provides a simple and programmatic way to interact with remote HTTP services and returns the response result to the client.

Currently the handler supports:  
* REST actions - DELETE, GET, POST, PUT
* Authentication - None, Basic, NTLM

At a high level, the flow of actions is:  
1. **Initialize** - It will read the handler configuration from the Action Config values in the Synapse plan.  
2. **Execute** - It will parse the incoming Action Parameter data, validate and interact with designated HTTP services accordingly. Result plan is returned upon completion advising the operation success or failure.
3. **Progress** - It will advertise the run-time status.

## Plan Details
### Config
TO BE ADDED.

### Sample
TO BE ADDED.

### Parameters
The Parameter section specifies what a client should send in during run-time.

### Sample
<script src="https://gist.github.com/SynapseGists/6bd5c22910d9072b403750db349457a6.js"></script>

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Authentication|String|Yes|Valid values are "none", "basic", "ntlm". "oauth1" and "oauth2" are to be added later.
|Username|String|No|Username is not required for authentication "none". But it is required for the others.
|Password|String|No|Password is not required for authentication "none". But it is required for the others.
|Domain|String|No|It is only required for authentication "ntlm".
|Url|string|Yes|Destination url. 
|Body|string|No|Request body if any to be sent to remote site.
|Method|string|Yes|Valid values are "delete", "get", "post" and "put".


## Sample Execution
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.

<script src="https://gist.github.com/SynapseGists/4c4f0e38f06eab89cdaab0dd59c9d87c.js"></script>

Handler response contained in the `$result` PowerShell variable  may look like this in "json" format.

<script src="https://gist.github.com/SynapseGists/4748af8d42ca6cd150b94fb7614a90d6.js"></script>

If the value of "ExitCode" is 0, it means the operation is successful. The value of -1 indicates the operation fails. "Note" will provide brief summary of the success or the reason of failure.