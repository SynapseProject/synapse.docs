# Overview
The Amazon Web Services (AWS) Handler provides a simple and programmatic way to interact with AWS resources.

## Plan Details
### Config
The config section of the plan specifies the one-to-one matching relationship between the cloud environment and the AWS credentials.

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|AwsEnvironmentProfile|Key-value pairs string array|Yes|Specify the exact matching of the AWS environment passed in from client request and the corresponding credential to access AWS resources.

### Dynamic Parameters
The dynamic section of the plan specifies the dynamic values a client may send in during run-time.

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|region|String|Yes|Dynamic parameter. Specify the region AWS resources are located, e.g. eu-west-1, us-east-2.
|cloudenvironment|String|Yes|Dynamic parameter. Specify the cloud environment, which is a tag value, the resources are tagged to.
|action|String|No|Dynamic parameter. Specify the action to be performed against the matching resources. Valid value supported is "none" at the moment.
|filters|Key-value pairs string array|Yes|Dynamic parameter. Specify the filters used to search for the resources.
|returnformat|string|Yes|Dynamic parameter. Valid values are "json", "xml" or "yaml".
|xslt|string|No|Dynamic parameter. Specify the v1.0 XSLT to transform the raw handler response. Can be empty.

### Sample Plan
````yaml
Name: ec2-matching-filters
Description: Query EC2 instances matching the specified filters.
Actions:
- Name: Synapse.Handlers.Aws:Ec2Handler
  Description: Resolved Handler from [Ec2Handler, Synapse.Handlers.Aws, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null].
  Handler:
    Type: Synapse.Handlers.Aws:Ec2Handler
    Config:
      Values:
        AwsEnvironmentProfile:
          EU_Customer_Production: XXXXXX
          EU_Customer_Test: XXXXXX
          EU_Customer_Development: XXXXXX
  Parameters:
    Dynamic:
    - Name: region
      Path: Region
    - Name: cloudenvironment
      Path: CloudEnvironment
    - Name: action
      Path: Action
    - Name: filters
      Path: Filters
      Parse: true
    - Name: returnformat
      Path: ReturnFormat
    - Name: xslt
      Path: Xslt
````

## Test Script
This test script can be modified to simulate a test request sent from client to Synapse to invoke the handler.

````powershell
$xslt = @"
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" version="1.0">
    <xsl:template match="/">
        <Servers>
            <xsl:for-each select="/Ec2Response/Ec2Instances/Ec2Instance">
                <Server>
                    <xsl:value-of select="PrivateDnsName" />
                </Server>
            </xsl:for-each>
        </Servers>
    </xsl:template>
</xsl:stylesheet>
"@

$filters = @(
	@{ "Name" = "tag:cloud-environment"; "Values" = @("xxxxxx") }
	@{ "Name" = "tag:Patch Group"; "Values" = @("Quarterly") }
	@{ "Name" = "instance-type"; "Values" = @("t2.medium") }
	@{ "Name" = "instance-state-name"; "Values" = @("running") }
)

$body = @{
	DynamicParameters  = @{
		"region"               = "eu-west-1"
		"cloudenvironment"     = "EU_Customer_Production" # Must match the config
		"action"               = "none"
		"xslt"                 = $xslt
		"filters"              = $filters | ConvertTo-Json -Compress
		"returnformat"         = "json" # Can be json, xml or yaml
	}
} | ConvertTo-Json -Compress

$result = Invoke-RestMethod http://localhost:20000/synapse/execute/ec2-matching-filters/start/sync -body $body -ContentType "application/json" -Method "post" -UseDefaultCredentials

````
Refer to <a href="http://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html" target="_blank">EC2 Filter List</a> for options available.

## Handler Exit Data Response
### Raw Response
Handler response contained in the "ExitData" field without any transformation may look like this in "xml" format.
````xml
<?xml version="1.0" encoding="utf-16"?>
<Ec2Response>
    <Count>5</Count>
    <Ec2Instances>
        <Ec2Instance>
            <Architecture>x86_64</Architecture>
            <AvailabilityZone>eu-west-1c</AvailabilityZone>
            <CloudEnvironment>xxxxxx</CloudEnvironment>
            <CostCentre />
            <InstanceId>i-xxxxxx</InstanceId>
            <InstanceState>running</InstanceState>
            <InstanceType>t2.medium</InstanceType>
            <LaunchTime>2016-08-11T09:17:29-04:00</LaunchTime>
            <Name>xxxxxx</Name>
            <PrivateDnsName>xxxxxx</PrivateDnsName>
            <PrivateIpAddress>10.xxx.xxx.46</PrivateIpAddress>
        </Ec2Instance>
        <Ec2Instance>
            <Architecture>x86_64</Architecture>
            <AvailabilityZone>eu-west-1c</AvailabilityZone>
            <CloudEnvironment>xxxxxx</CloudEnvironment>
            <CostCentre/>
            <InstanceId>i-xxxxxx</InstanceId>
            <InstanceState>running</InstanceState>
            <InstanceType>t2.medium</InstanceType>
            <LaunchTime>2017-10-17T07:10:18-04:00</LaunchTime>
            <Name>xxxxxx</Name>
            <PrivateDnsName>xxxxxx</PrivateDnsName>
            <PrivateIpAddress>10.xxx.xxx.50</PrivateIpAddress>
        </Ec2Instance>
    </Ec2Instances>
    <ExitCode>0</ExitCode>
    <Summary>Querying against AWS has been processed.</Summary>
</Ec2Response>

````
### Transformed Response
After the raw response is transformed using the specified "xslt", it may look like this in "xml".
````xml
<?xml version="1.0" encoding="utf-16"?>
<Servers>
    <Server>xxxxxx</Server>
    <Server>xxxxxx</Server>
</Servers>
````
Or like this in "json".
````json
{
   "Servers":{
      "Server":[
         "xxxxxx",
         "xxxxxx"
      ]
   }
}
````
Or like this in "yaml"
````yaml
Servers:
  Server:
    - xxxxxx
    - xxxxxx
````
