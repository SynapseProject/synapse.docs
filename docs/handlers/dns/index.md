
# Overview

The Domain Name System (DNS) Handler provides a simple and programmatic way to interact with designated DNS servers and update DNS records.

The handler makes use of Windows Management Instrumentation (WMI) protocol to communicate with the DNS server. DNS administrator may need to grant the user, under which the handler runs, with sufficient permissions and do additional setup on the selected DNS server.

At a high level, the flow of actions is:  
1. **Initialize** - It will read the DNS server details from the Action Config values in the Synapse plan.  
2. **Execute** - It will parse the incoming Action Parameter data, validate and interact with a relevant DNS server accordingly. Result plan is returned upon completion advising the operation success or failure.
3. **Progress** - It will advertise the run-time status.

## Plan Details
### Config
The Config section of the plan specifies the one-to-one matching relationship between each domain suffix and its corresponding DNS server.

### Sample
<script src="https://gist.github.com/SynapseGists/9608a2da9fc0b0bfa88d726e78af153c.js"></script>

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|DnsServers|Key-value pairs string array|Yes|Parent element. Requires at least one child element key-value pair.
|DomainSuffix|String|Yes|Domain suffix. 
|ServerName|String|Yes|DNS server name. 

### Parameters
The Parameter section specifies what a client should send in during run-time.

### Sample
<script src="https://gist.github.com/SynapseGists/1144510317a908c70dfb0dd3f15d359a.js"></script>

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Action|String|Yes|Specify to "add" or "delete" a DNS record. Case-insensitive.
|RecordType|String|Yes|Specify the DNS record type. Valid values are "AType" and "PtrType". Case-insensitive.
|Hostname|String|Yes|Hostname of the server.
|IpAddress|String|Yes|Ip address of the server. Must match the hostname in the DNS record. 
|DnsZone|string|No|DNS zone. Only required when adding "PtrType" record.
|RequestOwner|string|Yes|Specify the request owner. This is captured for audit purpose.
|Note|string|Yes|Specify the purpose of the request. This is captured for audit purpose.

Multiple DNS actions are allowed.

## Sample Execution
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.

<script src="https://gist.github.com/SynapseGists/ef91ec47acfa55ffc96295100db157b7.js"></script>

Handler response contained in the `$result` PowerShell variable  may look like this in "json" format.

<script src="https://gist.github.com/SynapseGists/a1908e4c6c5b8aab9e275b9c1a712772.js"></script>

If the value of "ExitCode" is 0, it means the operation is successful. The value of -1 indicates the operation fails. "Note" will provide brief summary of the success or the reason of failure.