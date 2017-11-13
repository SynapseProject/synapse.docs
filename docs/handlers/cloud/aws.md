# Overview
The Amazon Web Services (AWS) Handler provides a simple and programmatic way to interact with AWS resources.

At a high level, the flow of actions is:  
1. **Initialize** - It will read the AWS environment details from the Action Config values in the Synapse plan.  
2. **Execute** - It will parse the incoming Action Parameter data, validate and interact with AWS accordingly. Result plan is returned upon completion. If a valid Extensible Stylesheet Language Transformation (XSLT) document is supplied, a transformed result will be returned instead.  
3. **Progress** - It will advertise the run-time status.  

## Plan Details
### Config
The Config section of the plan specifies the one-to-one matching relationship between the cloud environment and the AWS credentials.

### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.Aws:Ec2Handler
    Config:
      Values:
        AwsEnvironmentProfile:
          profile_name_1: XXXXXX
          profile_name_2: XXXXXX
          profile_name_3: XXXXXX
````

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|AwsEnvironmentProfile|Key-value pairs string array|Yes|Specify the exact matching of the AWS profile name passed in from client request and the corresponding credential to access AWS resources.

### Parameters
The Parameter section specifies what a client should send in during run-time.

### Sample
````yaml
  Parameters:
    Name: 
    Type: Yaml
    ForEach: 
    InheritFrom: 
    Uri: 
    Values:
      Region: eu-west-1
      CloudEnvironment: profile_name_0
      Action: none
      Filters:
      - Name: tag:Cost Code
        Values:
        - xxxxxx
      - Name: tag:Patch Group
        Values:
        - Quarterly
      - Name: instance-type
        Values:
        - t2.medium
      - Name: instance-state-name
        Values:
        - running
      ReturnFormat: xml
      Xslt: ''
````

````yaml
  Parameters:
    Name: 
    Type: Yaml
    ForEach: 
    InheritFrom: 
    Uri: 
    Values:
      Region: eu-west-1
      CloudEnvironment: profile_name_0
      Action: none
      Filters:
      - Name: tag:Cost Code
        Values:
        - xxxxxx
      - Name: tag:Patch Group
        Values:
        - Quarterly
      - Name: instance-type
        Values:
        - t2.medium
      - Name: instance-state-name
        Values:
        - running
      ReturnFormat: json
      Xslt: >-
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
````
|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Region|String|Yes|Specify the region AWS resources are located, e.g. eu-west-1, us-east-2.
|CloudEnvironment|String|Yes|Specify the AWS environment value which must match one of the values in AwsEnvironmentProfile from the Config section.
|Action|String|No|Specify the action to be performed against the matching resources. Valid value supported is "none" at the moment.
|Filters|Key-value pairs string array|Yes|Specify the filters used to search for the resources. Refer to <a href="http://docs.aws.amazon.com/cli/latest/reference/ec2/describe-instances.html" target="_blank">EC2 Filter List</a> for options available.
|ReturnFormat|string|Yes|Dynamic parameter. Valid values are "json", "xml" or "yaml".
|Xslt|string|No|Specify the v1.0 XSLT to transform the raw handler response. Response will not be transformed if the value is null or empty.

## Sample Execution
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
	@{ "Name" = "tag:Cost Code"; "Values" = @("xxxxxx") }
	@{ "Name" = "tag:Patch Group"; "Values" = @("Quarterly") }
	@{ "Name" = "instance-type"; "Values" = @("t2.medium") }
	@{ "Name" = "instance-state-name"; "Values" = @("running") }
)

$body = @{
	DynamicParameters  = @{
		"region"               = "eu-west-1"
		"cloudenvironment"     = "profile_name_0" # Must match the config
		"action"               = "none"
		"xslt"                 = $xslt
		"filters"              = $filters | ConvertTo-Json -Compress
		"returnformat"         = "json" # Can be json, xml or yaml
	}
} | ConvertTo-Json -Compress

$result = Invoke-RestMethod http://localhost:20000/synapse/execute/ec2-matching-filters/start/sync -body $body -ContentType "application/json" -Method "post" -UseDefaultCredentials
````

Handler response without any transformation contained in the `$result` PowerShell variable  may look like this in "xml" format.
````xml
<?xml version="1.0" encoding="utf-8"?>
<Ec2Response>
    <Count>5</Count>
    <Ec2Instances>
        <Ec2Instance>
            <Architecture>x86_64</Architecture>
            <AvailabilityZone>eu-west-1c</AvailabilityZone>
            <CloudEnvironment>xxxxxx</CloudEnvironment>
            <CostCentre>xxxxxx<CostCentre> 
            <InstanceId>i-xxxxxx</InstanceId>
            <InstanceState>running</InstanceState>
            <InstanceType>t2.medium</InstanceType>
            <LaunchTime>2016-08-11T09:17:29-04:00</LaunchTime>
            <Name>xxxxxx</Name>
            <PrivateDnsName>xxxxxx</PrivateDnsName>
            <PrivateIpAddress>xxx.xxx.xxx.xxx</PrivateIpAddress>
        </Ec2Instance>
        <Ec2Instance>
            <Architecture>x86_64</Architecture>
            <AvailabilityZone>eu-west-1c</AvailabilityZone>
            <CloudEnvironment>xxxxxx</CloudEnvironment>
            <CostCentre>xxxxxx<CostCentre> 
            <InstanceId>i-xxxxxx</InstanceId>
            <InstanceState>running</InstanceState>
            <InstanceType>t2.medium</InstanceType>
            <LaunchTime>2017-10-17T07:10:18-04:00</LaunchTime>
            <Name>xxxxxx</Name>
            <PrivateDnsName>xxxxxx</PrivateDnsName>
            <PrivateIpAddress>xxx.xxx.xxx.xxx</PrivateIpAddress>
        </Ec2Instance>
    </Ec2Instances>
    <ExitCode>0</ExitCode>
    <Summary>Querying against AWS has been processed.</Summary>
</Ec2Response>

````

After the raw response is transformed using the specified "xslt", it may look like this in "xml".
````xml
<?xml version="1.0" encoding="utf-8"?>
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
