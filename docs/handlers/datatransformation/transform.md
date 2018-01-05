# Overview
The TransformHandler allows you to take YAML, JSON, or XML data and reformat it, or convert it from one format to another.  The TransformHandler uses XSLT to transform data sets, so all inbound data is first converted to XML for processing against the stylesheet.  You may choose to render your output as XML and allow the hander to automatically convert it back to the source format, or set `PreserveOutputAsIs = true` to retain the XSLT output without further automatic type conversion.  If  `PreserveOutputAsIs = false`, your XSLT output must be in XML in order to convert back to the source InputType.

### Basic XSL Transformation Workflow
1. Take YAML, JSON, or XML input
2. If YAML or JSON, convert input to XML
3. Apply XSLT
   - If `PreserveOutputAsIs = true`, return string output from transform, do not execute autoconvert back to original InputType.
   - If `PreserveOutputAsIs = false`, if InputType is YAML/JSON, autoconvert XML-output to original InputType

<p align="center">
<img alt="Synapse Concept" src="../../img/syn_transformHandler.png" />
</p>


## Plan Details
### Config

The Config section of the plan specifies the Input type of Parameters.Data and Output type of Handler.Result.ExitData.  To convert a given data object _without also_ executing a Transform action, specify the appropriate values for InputType/OutputType and omit any Parameters Transformations/Queries.

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

### Parameters

The Parameters section of the plan specifies the data to transform and the transform/query expressions to execute against the data.  For each transform/query that is executed, the result is merged into a result data set, which is then returned in the format specfied in Config.OutputType.

#### Sample
````yaml
  Handler:
    Type: Synapse.Handlers.DataTransformation:TransformHandler
  Parameters:
    Values:
      Data: '{ "Valid": "Data" }'
      XslTransformations:
      - Xslt: <xsl:stylesheet ... >
        PreserveOutputAsIs: true
      - Xslt: <xsl:stylesheet ... >
      RegexQueries:
      - Expression: ^\w+[\w-\.]*\@\w+((-\w+)|(\w*))\.[a-z]{2,3}$
        ExecuteLineByLine: true
        PreserveOutputAsIs: true
      JsonQueries:
      - Expression: $..Products[?(@.Price >= 50)].Name
        PreserveOutputAsIs: true
      - Expression: $.Manufacturers[?(@.Name == 'Acme Co')]
````

|Element|Type/Value|Required|Description
|-------|----------|--------|-----------
|Data|object|Yes|The data to transform.  Must match the data serialization format specified in Config.InputType.
|XslTransformations|List|No|
|RegexQueries|List|No|
|JsonQueries|List|No|