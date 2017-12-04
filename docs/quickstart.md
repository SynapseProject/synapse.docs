# Synapse Server QuickStart

If you want to try Synapse Server out, you may do so on any Windows server/workstation.  No installation is required, you can run it right from the DOS prompt.

### 1. Setup

1. Download the latest build from GitHub: <a href="https://github.com/SynapseProject/synapse.server.net/releases" target="_blank">https://github.com/SynapseProject/synapse.server.net/releases</a>.
2. Extract the contents of the zip into two folders (by convention: .\Controller & .\Node) to run as separate processes.
3. Open two DOS-prompt windows, one in the .\Controller folder, the other in the .\Node folder.
4. From .\Controller, run `synapse.controller.cli service run`.  From .\Node, run `synapse.node.cli service run`.

**Note:** synapse.[controller/node].cli use `synapse.server.config.yaml` for understanding server bindings.  By default, each will establish a function-specific config, so running the clis from their respective folders is important (such that they have access to appropriate config settings).  This is very configurable: see [Options](/run/setup/options) for information on configuring `synapse.server.config.yaml` or for using alternate config files.

```dos
C:\synapse\Controller>Synapse.Controller.Cli.exe service run
Starting Synapse.Server as Server: Press Ctrl-C/Ctrl-Break to stop.
2017-05-15 20:00:21,831|INFO |(1)|Using SynapseServerConfig from [C:\synapse\Controller\Synapse.Server.config.yaml]
2017-05-15 20:00:21,851|INFO |(1)|Starting
2017-05-15 20:00:23,178|INFO |(1)|Listening on http://localhost:20000
2017-05-15 20:00:23,178|INFO |(1)|Authentication Scheme = [IntegratedWindowsAuthentication]
2017-05-15 20:00:23,910|INFO |(1)|Running


C:\synapse\Node>Synapse.Node.Cli.exe service run
Starting Synapse.Server as Node: Press Ctrl-C/Ctrl-Break to stop.
2017-05-15 20:02:15,001|INFO |(1)|Using SynapseServerConfig from [C:\synapse\Node\Synapse.Server.config.yaml]
2017-05-15 20:02:15,013|INFO |(1)|Starting
2017-05-15 20:02:15,215|INFO |(1)|Listening on http://localhost:20001
2017-05-15 20:02:15,215|INFO |(1)|Authentication Scheme = [IntegratedWindowsAuthentication]
2017-05-15 20:02:15,239|INFO |(1)|Running
```

### 2. Smoke-Test the Services

* Open another DOS-prompt to .\Controller, execute `synapse.controller.cli hi`.  You may do similar for the Node.

```dos
C:\synapse\Controller>Synapse.Controller.Cli.exe hi
Calling Hello on http://localhost:20000/synapse/execute
"Hello from SynapseController, World!"
```

### 3. Create a Plan, Run it

1.  Again in the .\Controller DOS-prompt window, execute:

    *  `synapse.cli.exe sample:Synapse.Core:EmptyHandler out:.\Dal\Plans\myPlan.yaml`
    *  This will create a very simple Synapse Plan based on the "EmptyHandler," placed into the .\Controller\Dal\Plans folder, and is thus ready to execute.

2.  To run the plan, execute `synapse.controller.cli s planName:myPlan dryRun:true`

```dos
C:\synapse\Controller>Synapse.Controller.Cli.exe s planName:myPlan dryRun:true
Calling StartPlan on http://localhost:20000/synapse/execute
636302779130701013
```

 3a. **Check the result:** Look in the .\Controller\Dal\History folder to see the ResultPlan.  Open the file in any friendly YAML editor, such as Visual Studio Code or Notepad++.

[or]

 3b. **Select the result** Use the "GetPlanElement" method to retrieve the result from the ResultPlan

```dos
C:\synapse\Controller>synapse.controller.cli ge planUniqueName:myPlan
                      planInstanceId:636302779130701013 elementPath:Actions[0]:Result:Status
Calling GetPlanElement on http://localhost:20000/synapse/execute
"Complete"
```