
## Overview  
The Representational State Transfer (REST) Handler provides a simple and programmatic way to interact with remote HTTP services and returns the response result to the client.

Currently the handler supports:  
1. REST actions - DELETE, GET, POST, PUT  
2. Authentication - None, Basic, NTLM (Windows) 

At a high level, the flow of actions is:  
1. **Initialize** - It will read the handler configuration from the Action Config values in the Synapse plan.  
2. **Execute** - It will parse the incoming Action Parameter data, validate and interact with designated HTTP services accordingly. Result plan is returned upon completion advising the operation success or failure.
3. **Progress** - It will advertise the run-time status.

## Plan Details
### Config
None. 

### Sample
None

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
|ContentType|string|No|For "post" or "put" method, it is mandatory.


## Sample Execution
### Sample 1 - Anonymous GET
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.

<script src="https://gist.github.com/SynapseGists/4c4f0e38f06eab89cdaab0dd59c9d87c.js"></script>

Handler response, contained in the "ExitData" field and captured in the `$result` PowerShell variable  may look like this in "json" format.

<script src="https://gist.github.com/SynapseGists/4748af8d42ca6cd150b94fb7614a90d6.js"></script>

If the value of "ExitCode" is 0, it means the operation is successful. The value of -1 indicates the operation fails. "Message" will provide brief summary of the success or the reason of failure.

### Sample 2 - Anonymous POST
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.

<script src="https://gist.github.com/SynapseGists/20f1bfd1583759c4518f36aecae7c167.js"></script>

Handler response, contained in the "ExitData" field and captured in the `$result` PowerShell variable  may look like this in "json" format.

<script src="https://gist.github.com/SynapseGists/5efafb6e6ec596d909ca6d6f946d862a.js"></script>

If the value of "ExitCode" is 0, it means the operation is successful. The value of -1 indicates the operation fails. "Message" will provide brief summary of the success or the reason of failure.


### Sample 3 - Anonymous PUT
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.

<script src="https://gist.github.com/SynapseGists/2441c70ac104b92b97c88bdac16a2b64.js"></script>

Handler response, contained in the "ExitData" field and captured in the `$result` PowerShell variable  may look like this in "json" format.

<script src="https://gist.github.com/SynapseGists/999aff646e1bb634fe93bf91ce2a3867.js"></script>

If the value of "ExitCode" is 0, it means the operation is successful. The value of -1 indicates the operation fails. "Message" will provide brief summary of the success or the reason of failure.

### Sample 4 - Anonymous DELETE
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.

<script src="https://gist.github.com/SynapseGists/2cc04ccb2900ac68e3ce8a0d4e81e595.js"></script>

Handler response, contained in the "ExitData" field and captured in the `$result` PowerShell variable  may look like this in "json" format.

<script src="https://gist.github.com/SynapseGists/b635cb7fb2b11faade7dda9b17b38d69.js"></script>

If the value of "ExitCode" is 0, it means the operation is successful. The value of -1 indicates the operation fails. "Message" will provide brief summary of the success or the reason of failure.

### Sample 5 - GET with Basic Authentication
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.

<script src="https://gist.github.com/SynapseGists/e6c681b3d63fce9a949593fd9d881c09.js"></script>

Handler response, contained in the "ExitData" field and captured in the `$result` PowerShell variable  may look like this in "json" format.

<script src="https://gist.github.com/SynapseGists/18f6a9d0cf4fe43c063bcc8dcbf64b4c.js"></script>

If the value of "ExitCode" is 0, it means the operation is successful. The value of -1 indicates the operation fails. "Message" will provide brief summary of the success or the reason of failure.

### Sample 6 - GET with NTLM (Windows) Authentication
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.

<script src="https://gist.github.com/SynapseGists/137e51b5a2d17e2097ccd69083bcc2b8.js"></script>

Handler response, contained in the "ExitData" field and captured in the `$result` PowerShell variable  may look like this in "json" format.

<script src="https://gist.github.com/SynapseGists/a2b303be729a2373154df732a2bec0ba.js"></script>

If the value of "ExitCode" is 0, it means the operation is successful. The value of -1 indicates the operation fails. "Message" will provide brief summary of the success or the reason of failure.

