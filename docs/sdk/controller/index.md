# Custom Synapse Controller Interface

Synapse.Controller accepts configuration specifying custom ApiController libraries, thereby allowing you apply custom data processing models, URIs, and RBAC implentations.  We consider this a "normal" integration pattern such that Synapse can feel like _part of your application_, not an adjunct entity.

## Synapse.Server.Extensibility

There are two ways to implement a custom ApiController: you may use the Synapse.CustomContoller utility to auto-generate a code file and dll based on a YAML template, or you can code a Controller by hand in Visual Studio.

- Read about [Synapse.CustomController Commandline Utility here](util).

- Read about [coding a Controller in Visual Studio here](vs).


## Declare your Custom Controller Interface

After you implement the custom interface and compile it to a dll, you need to specify it in Synapse.Server.config.yaml in the Controller->Assemblies section, as shown below.  The syntax is simply to list the assembly name, as opposed to `assembly:{namepsace:}class`, as in Handler declaration.  Synapse Controller will discover all classes that inherit from `System.Web.Http.ApiController` (<a href="https://msdn.microsoft.com/en-us/library/system.web.http.apicontroller(v=vs.118).aspx" target="_blank">MSDN<a/>).

```yaml
# Configure the 'Assemblies' node of Synapse.Server.config.yaml

Service:
  Name: Synapse.Controller
  DisplayName: Synapse Controller
  Role: Controller
...
Controller:
  ...
  Assemblies: 
  - Name: Synapse.CustomController0
    Config:
      {any required config}
  - Name: Synapse.CustomController1
    Config:
      {any required config}
  - Name: Synapse.CustomController2
    Config:
      {any required config}
  Dal:
    ...
```


## Debugging

In order to test your custom ApiController in the Visual Studio debugger, edit the the project properties: click on the project in the Solution Explorer window, and press Alt-Enter on the keyboard. From the Project Properties window, choose the Debug option from the left menu/option-list.

1. <a href="https://github.com/SynapseProject/synapse.server.net/releases" target="_blank">Download</a> the latest build of Synapse.Server and extraxt it to the .\bin folder of your custom controller (note: not the .\bin\Debug folder, or, if you're using a custom build location, extract local to that folder).
2. Choose “Start external program” and browse to either 1) `synapse.controller.cli.exe`, or 2) `synapse.server.exe` in the extract location from step 1.
    - If using `synapse.controller.cli.exe`, then under “Start Options,” fill in the “Command line arguments” with a `service run`.
    - Use `synapse.controller.cli.exe` to debug against _only_ a Controller instance, and use `synapse.server.exe` if you want a Controller _and_ and Node option (making sure to configure `synapse.server.config.yaml` appropriately ([detail](..\run\setup\options)).
3. See [Configuration Options](..\run\setup\options#relocating-the-assemblies-authorization-dal-and-handlers-folders) for assembly resolution, and add the Debug & Release folders to the .exe.config `probing privatePath` setting.
4. Press F5 to launch a debug session.

<p align="center">
<img alt="Synapse Handler" src="../../img/syn_controllerDebug.png" />
</p>