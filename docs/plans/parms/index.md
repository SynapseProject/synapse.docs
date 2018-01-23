# Synapse Plans: ParameterInfo Blocks

ParameterInfo blocks are configuration information used to initialize Handler modules (Handler->Config) and pass runtime invocation data to Handler methods (Action->Parameters).  Additionally, a ParameterInfo block declares the start-up configuration for SecurityContext modules (RunAs->Config).  For clarity, ParameterInfo block are used in the following contexts:

```yaml
Name: SamplePlan
Actions:
- Name: SampleAction
  Handler:
    Config:
      {ParameterInfo Block}
  Parameters:
    {ParameterInfo Block}
  RunAs:
    Config:
      {ParameterInfo Block}
```

## Overview

|Field | Description
|-|-
|`Name`|The name of the Config/Parameters block.  If supplied, the block will stored as a global variable.
|`Type`|The serialization format of the `Values` section.
|`InheritFrom`|References the `Name` of another, previously stored Config/Paramters block.  The inherited values are propagated to the current block.
|`Uri`|Retrieves data from a URI; propagates values to `Parameters`->`Values`.
|`Values`|A custom data structure as defined by the Handler.  All the other sections of a ParameterInfo block exist to define or modify the `Values` section, which is ultimately passed to the Handler->Execute method as runtime invocation data.
|`Dynamic`|Retrieves named values (`Source`) from the cmdline or URL and substitutes them into `Values` at the `Target` location.  Use `Options` to provide a predetermined set of values.
|`ParentExitData`|Retrieves the `Result`->`ExitData` from the parent Handler and propagates the values to `Parameters`->`Values`.  Use `TransformInPlace` for light data manipulation prior to `CopyToValues`.
|`ForEach`|Iterates `ForEach`->`Values`, substituting the current value into `Parameters`->`Values` at the given `Target`.  As ForEach is a list, multiple ForEach items will produce a Cartesian result across the `Values` sets.  Use `ParameterSource` to access the `ParentExitData` or another named Config/Parameters block's data, retrieving a list from the `Source` location to propagate to `Values`.
|`Crypto`|As with other section, use to encrypt selected values with the local Config/Parameters block.


```yaml
  Parameters:
    Name: NameSupportsInheritance
    Type: Yaml
    InheritFrom: APrecedingNamedParamInfo
    Uri: http://host/path
    Values: Custom values as defined by Handler/Provider
    Dynamic:
    - Source: URI parameter name
      Target: Element:IndexedElement[0]:Element
      Parse: true
      Replace: Regex Expression
      Encode: None | Base64
      Options:
      - Key: key
        Value: value
      - Key: key
        Value: value
    ParentExitData:
    - TransformInPlace:
        Source: Element:IndexedElement[0]:Element
        Target: Element:IndexedElement[0]:Element
        Parse: true
        Replace: Regex Expression
        Encode: None | Base64
      CopyToValues:
        Source: Element:IndexedElement[0]:Element
        Target: Element:IndexedElement[0]:Element
        Parse: true
        Replace: Regex Expression
        Encode: None | Base64
    ForEach:
    - ParameterSource:
        Name: Named ParameterInfo Block
        Source: Element:IndexedElement[0]:Element
        Parse: false
      Target: Element:IndexedElement[0]:Element
      Replace: Regex Expression
      Encode: None | Base64
      Values:
      - value0
      - value1
    Crypto:
      Key:
        Uri: Filepath to RSA key file; http support in future.
        ContainerName: RSA=supported container name
        CspFlags: NoFlags
      Elements:
      - Element:IndexedElement[0]:Element
      - Element:IndexedElement[1]:Element
```

### Example per SerializationType

|Serialization | Basic | ForEach
|-|-|-
|YAML | [Basic](/plans/parms/yaml/basic/) | [ForEach](/plans/parms/yaml/foreach/)
|JSON | [Basic](/plans/parms/json/basic/) | [ForEach](/plans/parms/json/foreach/)
|XML | [Basic](/plans/parms/xml/basic/) | [ForEach](/plans/parms/xml/foreach/)


## Detailed Field Description

### Name
The friendly name of the current ParameterInfo block.  This name is recorded in a dictionary at runtime and is used when referenced by other ParameterInfo block's `InheritFrom` field.  A ParameterInfo block Name should be *unique within a Plan*, or successive blocks of the same name will *replace* the dictionary pointer.

### Type
Dictates the serialization language of the Uri, Values, Dynamic, and ForEach fields.  Options are SerializationType: Yaml, Json, Xml.  If Type is omitted, the runtime engine defaults to Yaml.

### InheritFrom
The name of another ParameterInfo block within the current Plan.  Inheritable ParameterInfo blocks must *precede* the current ParameterInfo block in the Plan execution cycle.

### Uri
A file or http URI from which to fetch values.  URI-based values are suitable for storing commonly used, shared data sets and override any values gained from `InheritFrom`.  The format of the file/http payload should be just the plain value set (no Synapse Plan attributes), as in the following examples.

- **YAML Example**
    - **Uri: file://C:/Synapse.UnitTests/Plans/Parms/yaml_in.yaml**
    - The contents of the file `yaml_in.yaml` are:

```yaml
PNode0: PValue0_file
PNode1: PValue1_file
PNode2:
  PNode2_1: PValue2_1_file
  PNode2_2: PValue2_2_file
```

- **XML Example**
    - **Uri: file://C:/Synapse.UnitTests/Plans/Parms/xml_in.xml**
    - The contents of the file `xml_in.xml` are:

```xml
<PXmlDoc>
    <PNode0 PAttr0="PAValue0_file">PValue0_file</PNode0>
    <PNode1>PValue1_file</PNode1>
    <PNode2>
        <PNode2_1>PValue2_1_file</PNode2_1>
        <PNode2_2>PValue2_2_file</PNode2_2>
    </PNode2>
</PXmlDoc>
```

### Values
Locally declared values within a Plan.  Local `Values` are suitable for Plan-level/Plan-specific usage and override any values gained from `InheritFrom` and `Uri`.

- **YAML Example:**

```yaml
  Values:
    PNode0: PValue0_inline
    PNode1: PValue1_inline
    PNode3:
      PNode3_1: PValue3_1_inline
      PNode3_2: PValue3_2_inline
```

- **JSON Example:**

```json
  Values:
    {
      "CNode0": "CValue0_inline",
      "CNode1": "CValue1_inline",
      "CNode3": {
        "CNode3_1": "CValue3_1_inline",
        "CNode3_2": "CValue3_2_inline",
      }
    }
```

- **XML Example:**

```xml
  Values:
    <CXmlDoc>
        <CNode0>CValue0_inline</CNode0>
        <CNode1>CValue1_inline</CNode1>
        <CNode3>
            <CNode3_1 CAttr3_1="CAValue3_1_inline">CValue3_1_inline</CNode3_1>
            <CNode3_2>CValue3_2_inline</CNode3_2>
        </CNode3>
    </CXmlDoc>
```

### Dynamic
Dynamic suppors values which are provided dynamically at runtime, such as through CLI or URL parameters, which can then be mapped to Parameters->Values via Source/Target pairs.  Sources are declared in XPath for XML serialization and colon-separated lists (root:node0:node1:...) for YAML/JSON.  If the target path exists in the child data, the value updated.  If the target path does not exist, it will be created and seeded.  Of particular interest, YAML/JSON structures arriving as strings may optionally be Parsed and integrated into the Parameters Values structure iteslf.

```yaml
  Dynamic:
  - Source: URI parameter name
    Target: Element:IndexedElement[0]:Element
    Parse: true|false
    Replace: Regex expression
    Encode: None|Base64
    Options:
    - Key: key
      Value: value
    - Key: key
      Value: value
```

|Name|Type/Value|Required|Description
|-|-|-|-
|Source|String|Yes|The key name of the value in the dynamic values key-value pair collection.
|Target|String|Yes|The path the target location to add/update the value/structure. If the target path does not exist, it will be created and seeded.
|Parse|Boolean|No|Tries to parse the Source value as YAML/JSON and integrate the result into the Parameters Values structure.  This setting does not apply to XML data structures.
|Replace|String|No|Performs a Regular Expression replacement of the Target value with the value from Source, subject to the Regex pattern.
|Encoding|Enum|No|Specifies how to encode the value before replacement.  Options are _**None**_ and *Base64*.



- **YAML Example:**
    - A variable named `pnode0Dynamic` will replace the existing value for `PNode0`
    - A variable named `pnode3_1Dynamic` will replace the existing value for `PNode3` --> `PNode3_1`

```yaml
  Values:
    PNode0: PValue0_inline
    PNode1: PValue1_inline
    PNode3:
      PNode3_1: PValue3_1_inline
      PNode3_2: PValue3_2_inline
  Dynamic:
  - Source: pnode0Dynamic
    Target: PNode0
  - Source: pnode2_1Dynamic
    Target: PNode2:PNode2_1
  - Source: pnode3_1Dynamic
    Target: PNode3:PNode3_1
```

  - Assuming: `pnode0Dynamic = 'new PNode0 value'` and `pnode3_1Dynamic = 'new PNode3_1 value'`, the resulting Values will be:

```yaml
  Values:
    PNode0: new PNode0 value
    PNode1: PValue1_inline
    PNode3:
      PNode3_1: new PNode3_1 value
      PNode3_2: PValue3_2_inline
```

- **XML Example:**
    - A variable named `cnode0Dynamic` will replace the existing value for `CXmlDoc` --> `CNode0`
    - A variable named `cnode3_1Dynamic` will replace the existing value for `CXmlDoc` --> `CNode3` --> `CNode3_1` --> `@CAttr3_1`
    - *Note:* the indexes below are specified for clarity, but not strictly required in XPath syntax.

```xml
  Values:
    <CXmlDoc>
        <CNode0>CValue0_inline</CNode0>
        <CNode1>CValue1_inline</CNode1>
        <CNode3>
            <CNode3_1 CAttr3_1="CAValue3_1_inline">CValue3_1_inline</CNode3_1>
            <CNode3_2>CValue3_2_inline</CNode3_2>
        </CNode3>
    </CXmlDoc>
  Dynamic:
  - Source: cnode0Dynamic
    Target: /CXmlDoc[1]/CNode0[1]
  - Source: cnode2_1Dynamic
    Target: /CXmlDoc[1]/CNode2[1]/CNode2_1[1]
  - Source: cnode3_1Dynamic
    Target: /CXmlDoc[1]/CNode3[1]/CNode3_1[1]/@CAttr3_1
```

- Assuming: `cnode0Dynamic = 'new CNode0 value'` and `cnode3_1Dynamic = 'new CNode3_1 attr'`, the resulting Values will be:

```xml
  Values:
    <CXmlDoc>
        <CNode0>new CNode0 value</CNode0>
        <CNode1>CValue1_inline</CNode1>
        <CNode3>
            <CNode3_1 CAttr3_1="new CNode3_1 attr">CValue3_1_inline</CNode3_1>
            <CNode3_2>CValue3_2_inline</CNode3_2>
        </CNode3>
    </CXmlDoc>
```

### ParentExitData
Passing data from a parent Action to its children it accomplished with the ParentExitData section, which is an array of Source/Target pairs (see detail below).  If the Target path exists in the child data, the value updated.  If the Target path does not exist, it will be created and seeded.  As with Dynamic values, YAML/JSON structures arriving as strings may optionally be Parsed and integrated into the Parameters Values structure iteslf.

```yaml
  ParentExitData:
  - TransformInPlace:
      Source: Element:IndexedElement[0]:Element
      Target: Element:IndexedElement[0]:Element
      Parse: true
      Replace: Regex Expression
      Encode: None | Base64
    CopyToValues:
      Source: Element:IndexedElement[0]:Element
      Target: Element:IndexedElement[0]:Element
      Parse: true
      Replace: Regex Expression
      Encode: None | Base64
```

#### TransformInPlace

The TransformInPlace section provides a method for light data transformation _within_ the ParentExitData itself.  This is useful in scenarios where combining 2+ data elements is required _before_ copying data to the Values.

|Name|Type/Value|Required|Description
|-|-|-|-
|Source|String|No|The path to value/structure from the parent Action's ExitData.
|Target|String|No*|The path to the target location to add/update the value/structure _within_ the parent Action's ExitData.  This field is required if specifying Source.  If the target path does not exist, it will be created and seeded.

#### CopyToValues

The CopyToValues section copies values from ParentExitData to Parameters->Values.  


|Name|Type/Value|Required|Description
|-|-|-|-
|Source|String|Yes*|The path to value/structure from the parent Action's ExitData. -Note: When specifying TransformaInPlace->Source/Target, you may omit this Source setting if it is the same as TransformaInPlace->Target; Source will default to the TransformaInPlace->Target.
|Target|String|Yes|The path to the target location to add/update the value/structure in the current Action's Parameters Values.  If the target path does not exist, it will be created and seeded.

#### Settings with common definitions:

|Name|Type/Value|Required|Description
|-|-|-|-
|Parse|Boolean|No|Tries to parse the source value as YAML/JSON and integrate the result into the Parameters Values structure.
|Replace|String|No|Performs a Regular Expression replacement of the Target value with the value from Source, subject to the Regex pattern.
|Encoding|Enum|No|Specifies how to encode the value before replacement.  Options are _**None**_ and *Base64*.


- **YAML Example:**

```yaml
Name: ParentExitDataTest
DefaultHandlerType: Synapse.Core:EmptyHandler
Actions:
- Name: Parent
  Parameters:
    Values:
      SleepMilliseconds: 1000
      ReturnStatus: Complete
      ExitData:
        Something:
          Wonderful:
            Stars: 2001
        Foo:
          Bar: Tombstoned
        ListMember: [1,2,3]
  Actions:
  - Name: Child
    Parameters:
      ParentExitData:
        CopyToValues:
        - Source: Something:Wonderful:Stars
          Target: SleepMilliseconds
        - Source: Foo:Bar
          Target: ReturnStatus
        - Source: ListMember
          Target: ExitData
          Parse: True
```

- **XML Example:**

```xml
Name: ParentExitDataTest
DefaultHandlerType: Synapse.Core:EmptyHandler
Actions:
- Name: Parent
  Parameters:
    Type: Xml
    Values:
      <EmptyHandlerParameters>
        <SleepMilliseconds>1000</SleepMilliseconds>
        <ReturnStatus>Complete</ReturnStatus>
        <ExitData>
          <Something>
            <Wonderful>
              <Stars>2001</Stars>
            </Wonderful>
            <Foo>
              <Bar>Tombstoned</Bar>
            </Foo>
            <ListMember>
              <Servers>
                <Server>localhost0</Server>
                <Server>localhost1</Server>
                <Server>localhost2</Server>
              </Servers>
            </ListMember>
          </Something>
        </ExitData>
      </EmptyHandlerParameters>
  Actions:
  - Name: Child
    Parameters:
      Type: Xml
      ParentExitData:
        CopyToValues:
        - Source: /Something/Wonderful/Stars
          Target: /EmptyHandlerParameters/SleepMilliseconds[1]
        - Source: /Something/Foo/Bar
          Target: /EmptyHandlerParameters/ReturnStatus
        - Source: /Something/ListMember
          Target: /EmptyHandlerParameters/ExitData
```


### ForEach
ForEach blocks calculate the Cartesian product of the declared Target/Values and expands the Action into a set of Actions for each result item. ActionGroup and child Actions relationships are maintained and will be executed per result item, as well. Of note, as both Action.Handler.Config and Action.Parameters can be declared with ForEach blocks, the total execution iterations for an Action is the carstesian product of the expanded Config and expanded Parameters.

```yaml
  ForEach:
  - ParameterSource:
      Name: Named ParameterInfo Block
      Source: Element:IndexedElement[0]:Element
      Parse: false
    Target: Element:IndexedElement[0]:Element
    Replace: Regex Expression
    Encode: None | Base64
    Values:
    - value0
    - value1
```

|Name|Type/Value|Required|Description
|-|-|-|-
|Source|String|Yes|The path to value/structure from the Action's Values.
|Target|String|Yes|The path to the target location to add/update the value/structure in the current Action's Parameters Values.  If the target path does not exist, it will be created and seeded.


- **YAML Example**

```yaml
  Values:
    PNode0: PValue0_file
    PNode1: PValue1_file
    PNode2:
      PNode2_1: PValue2_1_file
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_inline
      PNode3_2: PValue3_2_inline
  ForEach:
    CopyToValues:
  - Target: PNode1
    Values:
    - PValue1_foreach_0
    - PValue1_foreach_1
  - Target: PNode2:PNode2_1
    Values:
    - PValue2_2_foreach_0
    - PValue2_2_foreach_1
```
 - The block above is processed like a nested for loop (psuedocode):

```cs
foreach( string value0 in valueSet0 )
    foreach( string value1 in valueSet1 )
        //substitute value0 in at PNode1, value1 at PNode2:PNode2_1
```

  - Resulting in four distinct sets of Values, as follows:
```yaml
  Values:
    PNode0: PValue0_file
    PNode1: PValue1_foreach_0
    PNode2:
      PNode2_1: PValue2_2_foreach_0
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_inline
      PNode3_2: PValue3_2_inline

  Values:
    PNode0: PValue0_file
    PNode1: PValue1_foreach_0
    PNode2:
      PNode2_1: PValue2_2_foreach_1
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_inline
      PNode3_2: PValue3_2_inline

  Values:
    PNode0: PValue0_file
    PNode1: PValue1_foreach_1
    PNode2:
      PNode2_1: PValue2_2_foreach_0
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_inline
      PNode3_2: PValue3_2_inline

  Values:
    PNode0: PValue0_file
    PNode1: PValue1_foreach_1
    PNode2:
      PNode2_1: PValue2_2_foreach_1
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_inline
      PNode3_2: PValue3_2_inline
```

---

## Precedence for value resolution
A ParameterInfo block will fetch and resolve values, in order, as follows:

||Field|Behavior
|-|-|-
|1.|InheritFrom|Will copy the named ParameterInfo block from the runtime dictionary.  Inheritable ParameterInfo blocks must precede the current ParameterInfo block in the Plan execution cycle.
|2.|Uri|Will read the value set from the URI location.  If InheritFrom was declared, URI values will be merged on top of any existing values from InheritFrom as: Result = InheritFrom + Uri.
|3.|Values|Any directly declared data values will be merged on top of existing values as: Result = InheritFrom + Uri + Values, or: Result = Result + Values.
|4.|Dynamic|Any dynamically provided values will be merged on top of existing values as: Result = InheritFrom + Uri + Values + Dynamic, or: Result = Result + Dynamic.
|5.|ParentExitData|Any values mapped from the Parent Action's ExitData will be merged on top of existing values as: Result = InheritFrom + Uri + Values + Dynamic + ParentExitData, or: Result = Result + ParentExitData.
|6.|ForEach|Substitutes specified values into the resultant ParameterInfo block from above steps, calculating: ForEachValues( Result ).


## Tying it Together
A complete example, showing Uri, Values, Dynamic, and ForEach processing follows below.

- **YAML Example**

```yaml
  Parameters:
    Name: ParamSet00
    Type: Yaml
    Uri: file://C:/Synapse.UnitTests/Plans/Parms/yaml_in.yaml
    Values:
      PNode0: PValue0_inline
      PNode1: PValue1_inline
      PNode3:
        PNode3_1: PValue3_1_inline
        PNode3_2: PValue3_2_inline
    Dynamic:
    - Source: pnode0Dynamic
      Target: PNode0
    - Source: pnode2_1Dynamic
      Target: PNode2:PNode2_1
    - Source: pnode3_1Dynamic
      Target: PNode3:PNode3_1
    ForEach:
      CopyToValues:
      - Target: PNode1
        Values:
        - PValue1_foreach_0
        - PValue1_foreach_1
      - Target: PNode2:PNode2_1
        Values:
        - PValue2_2_foreach_0
        - PValue2_2_foreach_1
```
  - Assuming the contents of the `yaml_in.yaml` are:

```yaml
PNode0: PValue0_file
PNode1: PValue1_file
PNode2:
  PNode2_1: PValue2_1_file
  PNode2_2: PValue2_2_file
```
  - And, parameters to synapse.cli.exe include:
```dos
synapse.cli.exe plan:sample.yaml dryRun:true pnode0Dynamic:PValue0_dynamic
                pnode2_1Dynamic:PValue2_1_Dynamic pnode3_1Dynamic:PValue3_1_Dynamic
```

#### Step 1 - Retrieve the file URI, resulting in:
```yaml
  Values:
    PNode0: PValue0_file
    PNode1: PValue1_file
    PNode2:
      PNode2_1: PValue2_1_file
      PNode2_2: PValue2_2_file
```

#### Step 2 - Merge Locally declared Value with the file URI contents, resulting in:
```yaml
  Values:
    PNode0: PValue0_inline
    PNode1: PValue1_inline
    PNode2:
      PNode2_1: PValue2_1_file
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_inline
      PNode3_2: PValue3_2_inline
```

#### Step 3 - Merge Dynamic values into current Values, resulting in:
```yaml
  Values:
    PNode0: PValue0_dynamic
    PNode1: PValue1_inline
    PNode2:
      PNode2_1: PValue2_1_Dynamic
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_Dynamic
      PNode3_2: PValue3_2_inline
```

#### Step 4 - Process the ForEach block, expanding the current Values into four new blocks:
```yaml
  Values:
    PNode0: PValue0_dynamic
    PNode1: PValue1_foreach_0
    PNode2:
      PNode2_1: PValue2_2_foreach_0
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_Dynamic
      PNode3_2: PValue3_2_inline

  Values:
    PNode0: PValue0_dynamic
    PNode1: PValue1_foreach_0
    PNode2:
      PNode2_1: PValue2_2_foreach_1
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_Dynamic
      PNode3_2: PValue3_2_inline

  Values:
    PNode0: PValue0_dynamic
    PNode1: PValue1_foreach_1
    PNode2:
      PNode2_1: PValue2_2_foreach_0
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_Dynamic
      PNode3_2: PValue3_2_inline

  Values:
    PNode0: PValue0_dynamic
    PNode1: PValue1_foreach_1
    PNode2:
      PNode2_1: PValue2_2_foreach_1
      PNode2_2: PValue2_2_file
    PNode3:
      PNode3_1: PValue3_1_Dynamic
      PNode3_2: PValue3_2_inline
```

The resulting four Value sets will each be passed to the Action.Handler as runtime parameters (as four independently executed Actions).