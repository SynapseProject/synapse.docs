# Overview
The TransformHandler allows you to take YAML, JSON, or XML data and reformat it, or convert it from one format to another.  Transform options include XSLT, Regex, or JsonQuery.  In the cases of XSLT and JsonQuery, the input data is first converted to the necessary type, XML or JSON, respectively, and then optionally converted back to the source type.  As a function of Config->OutputType, the data may be additionally converted as specified.

When executing multiple Transormations/Queries, each successive result set is combined into a total, composite result.  As such, each result must of the same format type.


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
|InputType|None, Yaml, **Json**, Xml|No|Indicates the serialization format of the Parameters.Data member.  Defaults to `Json` if not specified.
|OutputType|**None**, Yaml, Json, Xml|No|Indicates the serialization format of the Handler ExitData.  Defaults to `None` if not specified.  A value of `None` assumes OutputType = InputType, meaning, return the ExitData in the same serialization format as the InputData.  If specifying a format different than InputType, then a format-type conversion will occur.

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

## XslTransformations Detail

When using XSLT to transform data sets, all inbound data is first converted to XML for processing against the stylesheet.  You may choose to render your output as XML and allow the hander to automatically convert it back to the source format, or set `PreserveOutputAsIs = true` to retain the XSLT output without further automatic type conversion.  If  `PreserveOutputAsIs = false`, your XSLT output must be in XML in order to convert back to the source InputType.

### Basic XSL Transformation Workflow
1. Take YAML, JSON, or XML input
2. If YAML or JSON, convert input to XML
3. Apply XSLT
   - If `PreserveOutputAsIs = true`, return string output from transform, do not execute autoconvert back to original InputType.
   - If `PreserveOutputAsIs = false`, if InputType is YAML/JSON, autoconvert XML-output to original InputType

<p align="center">
<img alt="Synapse Concept" src="../../img/syn_transformHandler_xml.png" />
</p>

## RegexQueries Detail

This will execute a Regex Match and return the result.  Be sure to return a complete subset of the format type when combing result sets.

Complete Subset Examples:
- `<sometag>data</sometag>`
- `{ "sometag": "data" }`
- `sometag: data`

## JsonQueries Detail

JsonQueries are executed via Json.net's SelectTokens capability.  For details on syntax, see [this article](https://www.newtonsoft.com/json/help/html/QueryJsonSelectTokenJsonPath.htm).  As with XSLT tranformations, JsonQueries require conversion of source data to JSON before processing the query expression, then the result may be optionally converted back to the original source format (set `PreserveOutputAsIs = true|false` accordingly).

### Basic XSL Transformation Workflow
1. Take YAML, JSON, or XML input
2. If XML or JSON, convert input to JSON
3. Execute JsonQuery
   - If `PreserveOutputAsIs = true`, return string output from transform, do not execute autoconvert back to original InputType.
   - If `PreserveOutputAsIs = false`, if InputType is XML/JSON, autoconvert JSON-output to original InputType

<p align="center">
<img alt="Synapse Concept" src="../../img/syn_transformHandler_json.png" />
</p>