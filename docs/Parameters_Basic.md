# Example Parameters

## Parameters Example: URI, Static, Dynamic, and Inheritance

Consider the following Plan layout:

```yaml
#hierarchical-node plan, with Inheritance to child Action node in Config & Parms:
Name: plan0
Description: planDesc
IsActive: true
Actions:
- Name: action0
  Handler:
    Type: Synapse.Core:Synapse.Core.Runtime.EmptyHandler
    Config:
      Name: ConfigSet00
      Type: Yaml
      Uri: file://C:/Synapse.UnitTests/Plans/Config/yaml_in.yaml
      Values:
        CNode0: CValue0_inline
        CNode1: CValue1_inline
        CNode3:
          CNode3_1: CValue3_1_inline
          CNode3_2: CValue3_2_inline
      Dynamic:
      - Name: cnode0Dynamic
        Path: CNode0
      - Name: cnode2_1Dynamic
        Path: CNode2:CNode2_1
      - Name: cnode3_1Dynamic
        Path: CNode3:CNode3_1
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
  Actions:
  - Name: action1
    Handler:
      Type: Synapse.Core:Synapse.Core.Runtime.EmptyHandler
      Config:
        Name: ConfigSet01
        InheritFrom: ConfigSet00
        Type: Yaml
        Values:
          CNode0: CValue0_inline_1
          CNode3:
            CNode3_1: CValue3_1_inline_1
    Parameters:
      Name: ParamSet01
      InheritFrom: ParamSet00
      Type: Yaml
      Values:
        PNode1: PValue1_inline_1
        PNode3:
          PNode3_2: PValue3_2_inline_1

#Where: Config/yaml_in.yaml and Parms/yaml_in.yaml contain:
#Config:
CNode0: CValue0_file
CNode1: CValue1_file
CNode2:
  CNode2_1: CValue2_1_file
  CNode2_2: CValue2_2_file

#Parms:
PNode0: PValue0_file
PNode1: PValue1_file
PNode2:
  PNode2_1: PValue2_1_file
  PNode2_2: PValue2_2_file
```

## action0 Parameter Results
The resulting Config/Parms for action0, before any Dynamic data is applied, is:

```yaml
#Config:
CNode0: CValue0_inline
CNode1: CValue1_inline
CNode2:
  CNode2_1: CValue2_1_file
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_inline
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_inline
PNode1: PValue1_inline
PNode2:
  PNode2_1: PValue2_1_file
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_inline
  PNode3_2: PValue3_2_inline
```

The resulting Config/Parms for action0, after Dynamic data is applied, is:

```java
//Key/Value pairs, as collected from an external source:
Dictionary<string, string> dynamicData = new Dictionary<string, string>();
dynamicData.Add( "cnode0Dynamic", "CValue0_dynamic" );
dynamicData.Add( "cnode2_1Dynamic", "CValue2_1_dynamic" );
dynamicData.Add( "cnode3_1Dynamic", "CValue3_1_dynamic" );
dynamicData.Add( "pnode0Dynamic", "PValue0_dynamic" );
dynamicData.Add( "pnode2_1Dynamic", "PValue2_1_dynamic" );
dynamicData.Add( "pnode3_1Dynamic", "PValue3_1_dynamic" );
```

```yaml
#Config:
CNode0: CValue0_dynamic
CNode1: CValue1_inline
CNode2:
  CNode2_1: CValue2_1_dynamic
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_dynamic
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_dynamic
PNode1: PValue1_inline
PNode2:
  PNode2_1: PValue2_1_dynamic
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_dynamic
  PNode3_2: PValue3_2_inline
```

## action1 Parameter Results
The resulting Config/Parms for action1, which inherits from its config/parms from action0, is:

```yaml
#Config:
CNode0: CValue0_inline_1
CNode1: CValue1_inline
CNode2:
  CNode2_1: CValue2_1_file
  CNode2_2: CValue2_2_file
CNode3:
  CNode3_1: CValue3_1_inline_1
  CNode3_2: CValue3_2_inline

#Parms:
PNode0: PValue0_inline
PNode1: PValue1_inline_1
PNode2:
  PNode2_1: PValue2_1_file
  PNode2_2: PValue2_2_file
PNode3:
  PNode3_1: PValue3_1_inline
  PNode3_2: PValue3_2_inline_1
```