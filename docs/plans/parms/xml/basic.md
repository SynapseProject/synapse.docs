# Example Parameters

## Parameters Example: URI, Static, Dynamic, and Inheritance

Consider the following Plan layout:
#### Hierarchical-node plan, with Inheritance to child Action node in Config & Parms

```yaml
Name: plan0
Description: planDesc
IsActive: true
Actions:
- Name: action0
  Handler:
    Type: Synapse.Core:Synapse.Core.Runtime.EmptyHandler
    Config:
      Name: ConfigSet00
      Type: Xml
      Uri: file://C:/Synapse.UnitTests/Plans/Config/xml_in.xml
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
  Parameters:
    Name: ParamSet00
    Type: Xml
    Uri: file://C:/Synapse.UnitTests/Plans/Parms/xml_in.xml
    Values:
      <PXmlDoc>
          <PNode0 PAttr0="PAValue0_inline">PValue0_inline</PNode0>
          <PNode1>PValue1_inline</PNode1>
          <PNode3>
              <PNode3_1 PAttr0="PAValue0_inline">PValue3_1_inline</PNode3_1>
              <PNode3_2>PValue3_2_inline</PNode3_2>
          </PNode3>
      </PXmlDoc>
    Dynamic:
    - Source: pnode0Dynamic
      Target: /PXmlDoc[1]/PNode0[1]
    - Source: pnode2_1Dynamic
      Target: /PXmlDoc[1]/PNode2[1]/PNode2_1[1]
    - Source: pnode3_1Dynamic
      Target: /PXmlDoc[1]/PNode3[1]/PNode3_1[1]/@PAttr0
  Actions:
  - Name: action1
    Handler:
      Type: Synapse.Core:Synapse.Core.Runtime.EmptyHandler
      Config:
        Name: ConfigSet01
        InheritFrom: ConfigSet00
        Type: Xml
        Values:
          <CXmlDoc>
              <CNode0>CValue0_inline_1</CNode0>
              <CNode3>
                  <CNode3_1 CAttr3_1="CAValue3_1_inline_1">CValue3_1_inline_1</CNode3_1>
              </CNode3>
          </CXmlDoc>
    Parameters:
      Name: ParamSet01
      InheritFrom: ParamSet00
      Type: Xml
      Values:
        <PXmlDoc>
            <PNode1>PValue1_inline_1</PNode1>
            <PNode3>
                <PNode3_1 PAttr0="PAValue0_inline_1" />
                <PNode3_2>PValue3_2_inline_1</PNode3_2>
            </PNode3>
        </PXmlDoc>
```

####Where: Config/xml_in.xml and Parms/xml_in.xml contain:
#####Config:
```xml
<CXmlDoc>
    <CNode0 CAttr0="CAValue0_file">CValue0_file</CNode0>
    <CNode1>CValue1_file</CNode1>
    <CNode2>
        <CNode2_1>CValue2_1_file</CNode2_1>
        <CNode2_2>CValue2_2_file</CNode2_2>
    </CNode2>
</CXmlDoc>
```

#####Parms:
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

### action0 Parameter Results
The resulting Config/Parms for action0, before any Dynamic data is applied, is:
#####Config:
```xml
<CXmlDoc>
  <CNode0 CAttr0="CAValue0_file">CValue0_inline</CNode0>
  <CNode1>CValue1_inline</CNode1>
  <CNode2>
    <CNode2_1>CValue2_1_file</CNode2_1>
    <CNode2_2>CValue2_2_file</CNode2_2>
  </CNode2>
  <CNode3>
    <CNode3_1 CAttr3_1="CAValue3_1_inline">CValue3_1_inline</CNode3_1>
    <CNode3_2>CValue3_2_inline</CNode3_2>
  </CNode3>
</CXmlDoc>
```

#####Parms:
```xml
<PXmlDoc>
  <PNode0 PAttr0="PAValue0_inline">PValue0_inline</PNode0>
  <PNode1>PValue1_inline</PNode1>
  <PNode2>
    <PNode2_1>PValue2_1_file</PNode2_1>
    <PNode2_2>PValue2_2_file</PNode2_2>
  </PNode2>
  <PNode3>
    <PNode3_1 PAttr0="PAValue0_inline">PValue3_1_inline</PNode3_1>
    <PNode3_2>PValue3_2_inline</PNode3_2>
  </PNode3>
</PXmlDoc>
```

The resulting Config/Parms for action0, after Dynamic data is applied, is:

```cs
//Key/Value pairs, as collected from an external source:
Dictionary<string, string> dynamicData = new Dictionary<string, string>();
dynamicData.Add( "cnode0Dynamic", "CValue0_dynamic" );
dynamicData.Add( "cnode2_1Dynamic", "CValue2_1_dynamic" );
dynamicData.Add( "cnode3_1Dynamic", "CValue3_1_dynamic" );
dynamicData.Add( "pnode0Dynamic", "PValue0_dynamic" );
dynamicData.Add( "pnode2_1Dynamic", "PValue2_1_dynamic" );
dynamicData.Add( "pnode3_1Dynamic", "PValue3_1_dynamic" );
```

#####Config:
```xml
<CXmlDoc>
  <CNode0 CAttr0="CAValue0_file">CValue0_dynamic</CNode0>
  <CNode1>CValue1_inline</CNode1>
  <CNode2>
    <CNode2_1>CValue2_1_dynamic</CNode2_1>
    <CNode2_2>CValue2_2_file</CNode2_2>
  </CNode2>
  <CNode3>
    <CNode3_1 CAttr3_1="CValue3_1_dynamic">CValue3_1_inline</CNode3_1>
    <CNode3_2>CValue3_2_inline</CNode3_2>
  </CNode3>
</CXmlDoc>
```

#####Parms:
```xml
<PXmlDoc>
  <PNode0 PAttr0="PAValue0_inline">PValue0_dynamic</PNode0>
  <PNode1>PValue1_inline</PNode1>
  <PNode2>
    <PNode2_1>PValue2_1_dynamic</PNode2_1>
    <PNode2_2>PValue2_2_file</PNode2_2>
  </PNode2>
  <PNode3>
    <PNode3_1 PAttr0="PValue3_1_dynamic">PValue3_1_inline</PNode3_1>
    <PNode3_2>PValue3_2_inline</PNode3_2>
  </PNode3>
</PXmlDoc>
```

## action1 Parameter Results
The resulting Config/Parms for action1, which inherits from its config/parms from action0, is:

#####Config:
```xml
<CXmlDoc>
  <CNode0 CAttr0="CAValue0_file">CValue0_inline_1</CNode0>
  <CNode1>CValue1_inline</CNode1>
  <CNode2>
    <CNode2_1>CValue2_1_file</CNode2_1>
    <CNode2_2>CValue2_2_file</CNode2_2>
  </CNode2>
  <CNode3>
    <CNode3_1 CAttr3_1="CAValue3_1_inline_1">CValue3_1_inline_1</CNode3_1>
    <CNode3_2>CValue3_2_inline</CNode3_2>
  </CNode3>
</CXmlDoc>
```

#####Parms:
```xml
<PXmlDoc>
  <PNode0 PAttr0="PAValue0_inline">PValue0_inline</PNode0>
  <PNode1>PValue1_inline_1</PNode1>
  <PNode2>
    <PNode2_1>PValue2_1_file</PNode2_1>
    <PNode2_2>PValue2_2_file</PNode2_2>
  </PNode2>
  <PNode3>
    <PNode3_1 PAttr0="PAValue0_inline_1">PValue3_1_inline</PNode3_1>
    <PNode3_2>PValue3_2_inline_1</PNode3_2>
  </PNode3>
</PXmlDoc>
```