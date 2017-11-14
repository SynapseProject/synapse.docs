# Overview
The Amazon Web Services (AWS) Handler provides a simple and programmatic way to interact with AWS resources.

At a high level, the flow of actions is:  
1. **Initialize** - It will read the AWS environment details from the Action Config values in the Synapse plan.  
2. **Execute** - It will parse the incoming Action Parameter data, validate and interact with AWS accordingly. Result plan is returned upon completion. If a valid Extensible Stylesheet Language Transformation (XSLT) document is supplied, a transformed result will be returned instead.  
3. **Progress** - It will advertise the run-time status.  

## Plan Details
### Config
The Config section of the plan specifies the one-to-one matching relationship between the user-friendly profile names and actual AWS profiles.

### Sample
<script src="https://gist.github.com/SynapseGists/d8b12857398f46eb5fc945e7328b5295.js"></script>

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|AwsEnvironmentProfile|Key-value pairs string array|Yes|The key name can be anything. The corresponding value must be an existing AWS profile handler has access to. Each AWS profile has its access id and secret key. See <a href="http://docs.aws.amazon.com/cli/latest/userguide/cli-multiple-profiles.html" target="_blank">AWS Named Profiles</a> for more information.

### Parameters
The Parameter section specifies what a client should send in during run-time.

### Sample
<script src="https://gist.github.com/SynapseGists/80b41c5dc82fcb0b091b8ce5140187fd.js"></script>

<script src="https://gist.github.com/SynapseGists/3cba0895c9d1d8aab9be27c78186eb0f.js"></script>
|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Region|String|Yes|Specify the region AWS resources are located, e.g. eu-west-1, us-east-2.
|AwsEnvironmentProfile|String|Yes|Specify the AWS environment profile which must match one of the key names in AwsEnvironmentProfile from the Config section.
|Action|String|No|Specify the action to be performed against the matching resources. Valid value supported is "none" at the moment.
|Filters|Key-value pairs string array|Yes|Specify the filters used to search for the resources. Refer to <a href="http://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html" target="_blank">EC2 Filter List</a> for options available. Note that it is *AND* logic for all the filters but *OR* logic for each value in a filter. For example, the above examples instructs the handler to find all matching EC2s, which have tag "Cost Code" of value "xxxxxx" *OR* "yyyyyy" *AND* tag "Patch Group" of value "Quarterly" *AND* "instance-type" of value "t2.medium" *AND* "instance-state-name" of value "running".
|ReturnFormat|string|Yes|Dynamic parameter. Valid values are "json", "xml" or "yaml".
|Xslt|string|No|Specify the v1.0 XSLT to transform the raw handler response. Response will not be transformed if the value is null or empty.

## Sample Execution
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.
<script src="https://gist.github.com/SynapseGists/5401ad6dc00353155d8f0c085ba0300a.js"></script>

Handler response without any transformation contained in the `$result` PowerShell variable  may look like this in "xml" format.
<script src="https://gist.github.com/SynapseGists/7d370891bee0c99b863747f066d78902.js"></script>

After the raw response is transformed using the specified "xslt", it may look like this in "xml".
<script src="https://gist.github.com/SynapseGists/0d0b8115c046608a86f002676a9f1d88.js"></script>

Or like this in "json".
<script src="https://gist.github.com/SynapseGists/6edd4919d73b9fc2d2104f4a8d544871.js"></script>

Or like this in "yaml"
<script src="https://gist.github.com/SynapseGists/abf3247b54c1078c742cda89a78748a8.js"></script>