# Custom Synapse Controller Interface

Synapse.Controller accepts configuration specifying custom ApiController libraries, thereby allowing you apply custom data processing models, URIs, and RBAC implentations.  We consider this a "normal" integration pattern such that Synapse can feel like _part of your application_, not an adjunct entity.

## Synapse.Server.Extensibility

The easiest way to implement a custom ApiController is to create a new Class Library project (dll), and add a reference to <a href="https://www.nuget.org/packages/Synapse.Server.Extensibility" target="_blank">Synapse.Server.Extensibility from nuget.org</a>.  This will add all the necessary Web Api dependencies, and Synapse.Server.Extensibility contains a few helper utilities to aid in your dev.  See a <a href="https://gist.github.com/SynapseProject/0f345c4fa60cdb53ae8d3585cde24513" target="_blank">Custom ApiController using Synapse.Server.Extensibility on Github.com</a>.

## Native ApiControllers

You may choose to implement a regular .NET ApiController, and Synapse will load it just fine.  The Synapse.Server.Extensibility library mentioned above is simply a helper lib.  Working sans Synapse.Server.Extensibility just means you need to bridge into Synapse.Controller on your own, whereas Synapse.Server.Extensibility provides a utility helper class to do so for you.

## Declare your Custom Controller Interface

After you implement the custom interface, specify it in Synapse.Server.config.yaml in the Controller->Assemblies section, as shown below.  The syntax is simply to list the assembly name, as opposed to `assembly:{namepsace.}class`, as in Handler declaration.  Synapse Controller will discover all classes that inherit from `System.Web.Http.ApiController` (<a href="https://msdn.microsoft.com/en-us/library/system.web.http.apicontroller(v=vs.118).aspx" target="_blank">MSDN<a/>).

```yaml
# Configure the 'Assemblies' node of Synapse.Server.config.yaml

Service:
  Name: Synapse.Controller
  DisplayName: Synapse Controller
  Role: Server
WebApi:
  Host: localhost
  Port: 20000
  IsSecure: false
  Authentication:
    Scheme: Anonymous
    Config: 
Signature:
  KeyUri: 
  KeyContainerName: DefaultContainerName
  CspProviderFlags: NoFlags
Controller:
  NodeUrl: http://localhost:20001/synapse/node
  SignPlan: false
  Assemblies: 
  - Synapse.CustomController0
  - Synapse.CustomController1
  - Synapse.CustomController2
  Dal:
    ...
```

## Synapse.Server.Extensibility Example

- <a href="https://gist.github.com/SynapseProject/0f345c4fa60cdb53ae8d3585cde24513" target="_blank">Custom ApiController using Synapse.Server.Extensibility on Github.com</a>.

## Debugging

In order to test your custom ApiController in the Visual Studio debugger, edit the the project properties: click on the project in the Solution Explorer window, and press Alt-Enter on the keyboard. From the Project Properties window, choose the Debug option from the left menu/option-list.

1. <a href="https://github.com/SynapseProject/synapse.server.net/releases" target="_blank">Download</a> the latest build of Synapse.Server and extraxt it to the .\bin folder of your custom controller (note: not the .\bin\Debug folder, or, if you're using a custom build location, extract local to that folder).
2. Choose “Start external program” and browse to either 1) `synapse.controller.cli.exe`, or 2) `synapse.server.exe` in the extract location from step 1.
    - If using `synapse.controller.cli.exe`, then under “Start Options,” fill in the “Command line arguments” with a `service run`.
    - Use `synapse.controller.cli.exe` to debug against _only_ a Controller instance, and use `synapse.server.exe` if you want a Controller _and_ and Node option (making sure to configure `synapse.server.config.yaml` appropriately ([detail](..\run\setup\options)).
3. See [Configuration Options](..\run\setup\options#relocating-the-assemblies-authorization-dal-and-handlers-folders) for assembly resolution, and add the Debug & Release folders to the .exe.config `probing privatePath` setting.
4. Press F5 to launch a debug session.

<p align="center">
<img alt="Synapse Handler" src="../../img/syn_controllerDebug_.png" />
</p>