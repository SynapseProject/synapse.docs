# Example Parameters

## Parameters Example: URI, Static, Dynamic, and ForEach

Consider the following Plan layout:
```css
#single-node plan, with ForEach blocks in Config & Parms
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
      ForEach:
      - Path: CNode1
        Values:
        - CValue1_forach_0
        - CValue1_forach_1
      - Path: CNode2:CNode2_1
        Values:
        - CValue2_2_forach_0
        - CValue2_2_forach_1
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
      - PValue1_forach_0
      - PValue1_forach_1
    - Path: PNode2:PNode2_1
      Values:
      - PValue2_2_forach_0
      - PValue2_2_forach_1

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
The resulting Config/Parms for action0, before any ForEach processing is applied, is:
```css
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

## action0 ForEach Parameter Results
ForEach processing will take the number of options specified in Config times the number of options specified in Parameters and expand the matrix of all combinations, thus forming the cartesian product of Config and Parameters.  The resulting Config/Parms for action0, after ForEach processing is applied, is:
```css
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