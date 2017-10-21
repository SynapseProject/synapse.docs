# Example YAML

## Comprehensive, Multi-type Plan

#### Hierarchical-node plan, with Inheritance to child Action node in Config & Parms

```yaml
Name: SamplePlan
UniqueName: Human-knowable name for database lookups
Description: Sample Plan, all features shown.
IsActive: true
DefaultHandlerType: Synapse.Handlers.CommandLine:CommandHandler
Actions:
- Name: Sample Action
  Description: Sample Action friendly description.
  Proxy: 'Future-use: http://host:port/synapse/node'
  ExecuteCase: Any
  Handler:
    Type: Synapse.Handlers.CommandLine:CommandHandler
    Config:
      Name: NameSupportsInheritance
      ForEach:
      - Path: Element:IndexedElement[0]:Element
        Values:
        - value0
        - value1
      InheritFrom: APrecedingNamedParamInfo
      Uri: http://host/path
      Values: Custom values as defined by Handler/Provider
      Dynamic:
      - Name: URI parameter name
        Path: Element:IndexedElement[0]:Element
        Parse: true
        Replace: Regex expression
        Encode: Base64
        Options:
        - Key: key
          Value: value
        - Key: key
          Value: value
      ParentExitData:
      - Source: Element:IndexedElement[0]:Element
        Destination: Element:IndexedElement[0]:Element
        Parse: true
        Replace: Regex expression
        Encode: Base64
      Crypto:
        Key:
          Uri: Filepath to RSA key file; http support in future.
          ContainerName: RSA=supported container name
        Elements:
        - Element:IndexedElement[0]:Element
        - Element:IndexedElement[1]:Element
  RunAs:
    Domain: AD Domain
    UserName: username
    Password: Use Crypto to Encrypt this value
    Provider: 'Reserved for future use: AD, AWS, Azure, etc.'
    IsInheritable: true
    BlockInheritance: true
    Crypto:
      Key:
        Uri: Filepath to RSA key file; http support in future.
        ContainerName: RSA=supported container name
      Elements:
      - Element:IndexedElement[0]:Element
      - Element:IndexedElement[1]:Element
  Parameters:
    Name: NameSupportsInheritance
    ForEach:
    - Path: Element:IndexedElement[0]:Element
      Values:
      - value0
      - value1
    InheritFrom: APrecedingNamedParamInfo
    Uri: http://host/path
    Values: Custom values as defined by Handler/Provider
    Dynamic:
    - Name: URI parameter name
      Path: Element:IndexedElement[0]:Element
      Parse: true
      Replace: Regex expression
      Encode: Base64
      Options:
      - Key: key
        Value: value
      - Key: key
        Value: value
    ParentExitData:
    - Source: Element:IndexedElement[0]:Element
      Destination: Element:IndexedElement[0]:Element
      Parse: true
      Replace: Regex expression
      Encode: Base64
    Crypto:
      Key:
        Uri: Filepath to RSA key file; http support in future.
        ContainerName: RSA=supported container name
      Elements:
      - Element:IndexedElement[0]:Element
      - Element:IndexedElement[1]:Element
  Actions: []
- Name: Synapse.Core:EmptyHandler
  Description: a friendly description.
  Handler:
    Type: synapse.core:EmptyHandler
    Config: {}
  Parameters:
    Values:
      SleepMilliseconds: 1000
      ReturnStatus: Complete
      ExitData: Sample Value
- Name: Sample_JSON_XML
  Handler:
    Type: Some.Library:SomeHandler
    Config:
      Type: Xml
      Uri: file://C:/Synapse/parms.xml
      Values: <xml attr="value1"><data>foo1</data></xml>
      Dynamic:
      - Name: app
        Path: /xml[1]/data[1]
      - Name: type
        Path: /xml[1]/@attr
  Parameters:
    Type: Json
    Uri: http://foo/parms.json
    Values:
      {
        "ApplicationName": "fooApp",
        "EnvironmentName": "dev1",
        "Tier": {
            "Name": "webserver1",
            "Type": "python1",
            "Version": "1.0"
        },
      }
    Dynamic:
    - Name: app
      Path: ApplicationName
      Options:
      - Key: 1
        Value: Something
      - Key: 2
        Value: Wonderful
    - Name: type
      Path: Tier:Type
RunAs:
  Domain: AD Domain
  UserName: username
  Password: Use Crypto to Encrypt this value
  Provider: 'Reserved for future use: AD, AWS, Azure, etc.'
  IsInheritable: true
  BlockInheritance: true
  Crypto:
    Key:
      Uri: Filepath to RSA key file; http support in future.
      ContainerName: RSA=supported container name
    Elements:
    - Element:IndexedElement[0]:Element
    - Element:IndexedElement[1]:Element
Crypto:
  Key:
    Uri: Filepath to RSA key file; http support in future.
    ContainerName: RSA=supported container name
StartInfo: {}
LastModified: 10/21/2017 2:28:54 PM
Signature: RSA Cryptographic signature, applied at runtime.
```