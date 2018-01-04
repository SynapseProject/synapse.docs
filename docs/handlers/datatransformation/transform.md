# Overview
The TransformHandler allows you to take YAML, JSON, or XML data and reformat it, or convert it from one format to another.  The TransformHandler uses XSLT to transform data sets, so all inbound data is first converted to XML for processing against the stylesheet.  You may choose to render your output as XML and allow the hander to automatically convert it back to the source format, or set `PreserveOutputAsIs = true` to retain the XSLT output without further automatic type conversion.  If  `PreserveOutputAsIs = false`, your XSLT output must be in XML in order to convert back to the source InputType.

### Basic Transformation Workflow
1. Take YAML, JSON, or XML input
2. If YAML or JSON, convert input to XML
3. Apply XSLT
   - If `PreserveOutputAsIs = true`, return string output from transform, do convert back to original InputType.
   - If `PreserveOutputAsIs = false`, if InputType is YAML/JSON, convert XML-output to InputType

<p align="center">
<img alt="Synapse Concept" src="../../img/syn_transformHandler.png" style="width: 431px; height: 361px;" />
</p>


## Plan Details
### Config

The config section of the plan specifies what 

#### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.DataTransformation:TransformHandler
    Config:
      Values:
        InputType: Json
        OutputType: Yaml
````

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|InputType|None, YAML, **JSON**, XML|No|Indicates the serialization format of the Parameters.Data member.  Defaults to `JSON` if not specified.
|OutputType|**None**, YAML, JSON, XML|No|Indicates the serialization format of the Handler ExitData.  Defaults to `None` if not specified.  A value of `None` assumes OutputType = InputType, meaning, return the ExitData in the same serialization format as the InputData.  If specifying a format different than InputType, then a format-type conversion will occur.

## Plan Details
### Config

The config section of the plan specifies Input/Output type.  Can execute convert-only with no parms. 

#### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.DataTransformation:TransformHandler
  Parameters:
    Values:
      Data: '{ "Valid": Data }'
      XslTransformations:
      - Xslt: <xsl:stylesheet ... >
        PreserveOutputAsIs: true
      - Xslt: <xsl:stylesheet ... >
````

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Data|object|Yes|The data to transform
|XslTransformations|List|No|
