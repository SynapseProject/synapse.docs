# Example Parameters

## Parameters Example: URI, Static, Dynamic, and ForEach

Consider the following Plan layout:
#### Single-node plan, with ForEach blocks in Config & Parms

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
      Uri: file://C:/Devo/synapse/synapse.core.net/synapse.net/Synapse.UnitTests/Plans/Config/xml_in.xml
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
      ForEach:
      - CopyToValues:
        - Target: /CXmlDoc[1]/CNode1[1]
          Values:
          - CValue1_forach_0
          - CValue1_forach_1
        - Target: /CXmlDoc[1]/CNode2[1]/CNode2_1[1]
          Values:
          - CValue2_2_forach_0
          - CValue2_2_forach_1
  Parameters:
    Name: ParamSet00
    Type: Xml
    Uri: file://C:/Devo/synapse/synapse.core.net/synapse.net/Synapse.UnitTests/Plans/Parms/xml_in.xml
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
    ForEach:
    - CopyToValues:
      - Target: /PXmlDoc[1]/PNode1[1]
        Values:
        - PValue1_forach_0
        - PValue1_forach_1
      - Target: /PXmlDoc[1]/PNode2[1]/PNode2_1[1]
        Values:
        - PValue2_2_forach_0
        - PValue2_2_forach_1
```

####Where: Config/xml_in.xml and Parms/xml_in.xml contain:
####Config:
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

####Parms:
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
The resulting Config/Parms for action0, before any ForEach processing is applied, is:

####Config:
```yaml
CNode0: CValue0_inline
CNode1: CValue1_inline
CNode2:
  CNode2_1: CValue2_1_file
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_inline
  CNode3_2: CValue3_2_inline
```

####Parms:
```yaml
PNode0: PValue0_inline
PNode1: PValue1_inline
PNode2:
  PNode2_1: PValue2_1_file
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_inline
  PNode3_2: PValue3_2_inline
```

### action0 ForEach Parameter Results
ForEach processing will take the number of options specified in Config times the number of options specified in Parameters and expand the matrix of all combinations, thus forming the cartesian product of Config and Parameters.  The resulting Config/Parms for action0, after ForEach processing is applied, is:

```yaml
Yaml: Results as Config/Parms Set

#Set 0:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_0
CNode2:
  CNode2_1: CValue2_2_forach_0
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_0
PNode2:
  PNode2_1: PValue2_2_forach_0
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 1:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_0
CNode2:
  CNode2_1: CValue2_2_forach_1
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_1
PNode2:
  PNode2_1: PValue2_2_forach_0
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 2:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_1
CNode2:
  CNode2_1: CValue2_2_forach_1
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_0
PNode2:
  PNode2_1: PValue2_2_forach_1
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 3:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_1
CNode2:
  CNode2_1: CValue2_2_forach_0
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_1
PNode2:
  PNode2_1: PValue2_2_forach_0
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 4:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_0
CNode2:
  CNode2_1: CValue2_2_forach_1
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_0
PNode2:
  PNode2_1: PValue2_2_forach_1
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 5:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_1
CNode2:
  CNode2_1: CValue2_2_forach_1
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_1
PNode2:
  PNode2_1: PValue2_2_forach_1
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 6:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_0
CNode2:
  CNode2_1: CValue2_2_forach_1
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_1
PNode2:
  PNode2_1: PValue2_2_forach_1
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 7:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_1
CNode2:
  CNode2_1: CValue2_2_forach_0
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_0
PNode2:
  PNode2_1: PValue2_2_forach_1
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 8:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_1
CNode2:
  CNode2_1: CValue2_2_forach_0
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_1
PNode2:
  PNode2_1: PValue2_2_forach_1
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 9:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_1
CNode2:
  CNode2_1: CValue2_2_forach_1
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_0
PNode2:
  PNode2_1: PValue2_2_forach_0
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 10:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_0
CNode2:
  CNode2_1: CValue2_2_forach_1
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_0
PNode2:
  PNode2_1: PValue2_2_forach_0
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 11:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_0
CNode2:
  CNode2_1: CValue2_2_forach_0
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_1
PNode2:
  PNode2_1: PValue2_2_forach_0
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 12:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_1
CNode2:
  CNode2_1: CValue2_2_forach_0
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_0
PNode2:
  PNode2_1: PValue2_2_forach_0
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 13:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_0
CNode2:
  CNode2_1: CValue2_2_forach_0
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_0
PNode2:
  PNode2_1: PValue2_2_forach_1
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 14:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_0
CNode2:
  CNode2_1: CValue2_2_forach_0
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_1
PNode2:
  PNode2_1: PValue2_2_forach_1
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline

--------------------

#Set 15:
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_forach_1
CNode2:
  CNode2_1: CValue2_2_forach_1
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_forach_1
PNode2:
  PNode2_1: PValue2_2_forach_0
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline
```