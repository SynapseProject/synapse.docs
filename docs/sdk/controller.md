# Custom Synapse Controller Interface

Synapse.Controller accepts configuration specifying custom ApiController libraries, thereby allowing you apply custom data processing models, URIs, and RBAC implentations.  We consider this a "normal" integration pattern such that Synapse can feel like _part of your application_, not an adjunct entity.

## Synapse.Server.Extensibility

The easiest way to implement a custom ApiController is to create a new Class Library project (dll), and add a reference to <a href="https://www.nuget.org/packages/Synapse.Server.Extensibility" target="_blank">Synapse.Server.Extensibility from nuget.org</a>.  This will add all the necessary Web Api dependencies, and Synapse.Server.Extensibility contains a few helper utilities to aid in your dev.  See a <a href="https://gist.github.com/SynapseProject/0f345c4fa60cdb53ae8d3585cde24513" target="_blank">Custom ApiController using Synapse.Server.Extensibility on Github.com</a>.

## Native ApiControllers

You may choose to implement a regular .NET ApiController, and Synapse will load it just fine.  The Synapse.Server.Extensibility library mentioned above is simply a helper lib.  Working sans Synapse.Server.Extensibility just means you need to bridge into Synapse.Controller on your own, whereas Synapse.Server.Extensibility provides a utility helper class to do so for you.

## Declare your Custom Controller Interface

After you implement the custom interface, specify it in Synapse.Server.config.yaml, as follows:

```yaml
# Configure the 'Assemblies' node of Synapse.Server.config.yaml

ServiceName: Synapse.Controller
ServiceDisplayName: Synapse Controller
ServerRole: Controller
WebApiPort: 20000
AuthenticationScheme: IntegratedWindowsAuthentication
SignatureKeyFile: 
SignatureKeyContainerName: DefaultContainerName
SignatureCspProviderFlags: NoFlags
Controller:
  NodeUrl: http://localhost:20001/synapse/node
  SignPlan: false
  Assemblies:
  - Synapse.CustomController0
  - Synapse.CustomController1
  - Synapse.CustomController2
```

## Synapse.Server.Extensibility Example

- <a href="https://gist.github.com/SynapseProject/0f345c4fa60cdb53ae8d3585cde24513" target="_blank">Custom ApiController using Synapse.Server.Extensibility on Github.com</a>.
