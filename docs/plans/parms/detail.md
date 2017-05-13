# Synapse Plans: ParameterInfo Blocks

ParameterInfo blocks are configuration information used to initialize Handler modules and pass runtime invocation data to Handler methods.  Additionally, a ParameterInfo block declares the start-up configuration for SecurityContext modules.

## Example per SerializationType

|Serialization | Basic | ForEach
|-|-|-
|YAML | [Basic](/plans/parms/yaml/basic/) | [ForEach](/plans/parms/yaml/foreach/)
|JSON | [Basic](/plans/parms/json/basic/) | [ForEach](/plans/parms/json/foreach/)
|XML | [Basic](/plans/parms/xml/basic/) | [ForEach](/plans/parms/xml/foreach/)


## Detailed Field Description

#### Name
The friendly name of the current ParameterInfo block.  This name is recorded in a dictionary at runtime and is used when referenced by other ParameterInfo block's `InheritFrom` field.  A ParameterInfo block Name should be *unique within a Plan*, or successive blocks of the same name will *replace* the dictionary pointer.

#### Type
Dictates the serialization language of the Uri, Values, Dynamic, and ForEach fields.  Options are SerializationType: Yaml, Json, Xml.  If Type is omitted, the runtime engine defaults to Yaml.

#### InheritFrom
The name of another ParameterInfo block within the current Plan.  Inheritable ParameterInfo blocks must *precede* the current ParameterInfo block in the Plan execution cycle.

#### Uri
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

#### Values
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

#### Dynamic
Name/Path pairs which are provided dynamically at runtime, such as through CLI or URL parameters.  Paths are declared in XPath for XML serialization and colon-separated lists (root:node0:node1:...) for YAML/JSON.

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
    - Name: pnode0Dynamic
      Path: PNode0
    - Name: pnode2_1Dynamic
      Path: PNode2:PNode2_1
    - Name: pnode3_1Dynamic
      Path: PNode3:PNode3_1
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
      - Name: cnode0Dynamic
        Path: /CXmlDoc[1]/CNode0[1]
      - Name: cnode2_1Dynamic
        Path: /CXmlDoc[1]/CNode2[1]/CNode2_1[1]
      - Name: cnode3_1Dynamic
        Path: /CXmlDoc[1]/CNode3[1]/CNode3_1[1]/@CAttr3_1
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

#### ForEach
ForEach blocks calculate the cartesian product of the declared Path/Values and expands the Action into a set of Actions for each result item. ActionGroup and child Actions relationships are maintained and will be executed per result item, as well. Of note, as both Action.Handler.Config and Action.Parameters can be declared with ForEach blocks, the total execution iterations for an Action is the carstesian product of the expanded Config and expanded Parameters.

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
    - Path: PNode1
      Values:
      - PValue1_foreach_0
      - PValue1_foreach_1
    - Path: PNode2:PNode2_1
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
|5.|ForEach|Substitutes specified values into the resultant ParameterInfo block from above steps, calculating: ForEachValues( Result ).


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
    - Name: pnode0Dynamic
      Path: PNode0
    - Name: pnode2_1Dynamic
      Path: PNode2:PNode2_1
    - Name: pnode3_1Dynamic
      Path: PNode3:PNode3_1
    ForEach:
    - Path: PNode1
      Values:
      - PValue1_foreach_0
      - PValue1_foreach_1
    - Path: PNode2:PNode2_1
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
synapse.cli.exe /plan:sample.yaml /dryRun:true /pnode0Dynamic:PValue0_dynamic
                /pnode2_1Dynamic:PValue2_1_Dynamic /pnode3_1Dynamic:PValue3_1_Dynamic
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