# How To Contribute

## Setup ##

1. Download the latest build from GitHub: <a href="https://github.com/SynapseProject/synapse.server.ui.core/releases" target="_blank">https://github.com/SynapseProject/synapse.server.ui.core/releases</a>

2. Extract the contents of the zip into a folder.

3. Open the solution.

4. Create a new project for your module. Use “ASP.NET Core Web Application” project template (or “Telerik ASP.NET Core MVC Application” project template if you are using Kendo UI controls) so you have the basic structure created for you. Name the project “Synapse.UI.Modules.[your module name]”. 

5. Set Target framework “.NET Standard 2.0”, Output type “Class Library”.

6. Remove the following files

    - `Views\_ViewImports.cshtml`
    - `Views\_ViewStart.cshtml`
    - `Views\Shared\_Layout.cshtml`
    - `appsettings.json`
    
7. Add the following folders

    - `Scripts` (if applicable) to store your scripts
    - `Styles` (if applicable) to store your css file
    
8.	Add the following line in the project file (`.csproj`) to embedded views and static contents into the resulting assembly.

    `<EmbeddedResource Include="Views\**;Styles\**;Scripts\**" />`


## Coding Guidelines ##

1. Place your stylesheets and scripts into the `Styles` and `Scripts` folders.

2. Set the `ViewBag.Title` property accordingly in the `.cshtml` file.

2. Use `@section` to add scripts and css. When referencing an embedded resource, replace the forward slash `'/'` after each folder with a dot `‘.’`. See examples below.
 
```html
    @section Styles {<link href="~/Styles.plan-execution.css" rel="stylesheet" />}
    @section Scripts {<script src="~/Scripts.plan-execution.js"></script>}
```

3. To register additional services to the service collection, implement the `IAddModuleService` interface. The interface is defined inside `Synapse.UI.Infrastructure` so you’ll need to add a reference to it in your project. Below is an example of the implementation.

```csharp
    public class AddTestService: IAddModuleService
        {
            public int Priority => 1000;
    
            public void Execute(IServiceCollection serviceCollection)
            {
                serviceCollection.AddTransient<ITestService, TestService>();
            }
        }
```

## Running The App

1. Build your project as you would normally do.

2. In `Synapse.UI.WebApplication` project,

    * Create a folder under the `Modules` folder. Copy your class library and any other dependent libraries to this folder. Libraries that are already referenced in `Synapse.UI.WebApplication` project can be excluded.
    
    * Edit `appsettings.json`. Add another entry under the `Include` section. Place it in the order of how you would like it appear on the navigation menu. Change the value of `SynapseControllerAPIURL` accordingly. Below is an example of the configuration settings.

```json
    {  
      "Modules": {
        "RootPath":  "\\Modules",
        "Include": [
          {
            "FolderName": "ModuleA",
            "FriendlyName": "Module A",
            "Url": "ModuleA"
          },
          {
            "FolderName": "ModuleB",
            "FriendlyName": "Module B",
            "Url": "ModuleB"
          },
          {
            "FolderName": "PlanExecution",
            "FriendlyName": "Plan Execution",
            "Url": "PlanExecution"
          }
        ]
      },
      "DefaultRoute": {
        "Controller": "PlanExecution",
        "Action": "Index"
      },
      "SynapseControllerAPIURL": "http://localhost:20000/synapse/execute/"
    }
```
|Name|Description
|-|-
|Modules:Include:FolderName|Name of the folder that contains assemblies for your module.
|Modules:Include:FriendlyName|Name given to the module. It is also the name displayed in the navigation menu.
|Modules:Include:Url| Relative URL to the controller.
|Default Route|Defines the default controller and action to use when the values are not provided.
|SynapseControllerAPIURL|Holds the URL to the Synapse Controller API.


