# Synapse Server QuickStart

If you want to try Synapse Server out, you may do so on any Windows server/workstation.  No installation is required, you can run it right from the DOS prompt.

### Setup

1. Download the latest build from GitHub: <a href="https://github.com/SynapseProject/synapse.server.net/releases" target="_blank">https://github.com/SynapseProject/synapse.server.net/releases</a>.
2. Extract the contents of the zip into two folders (by convention: .\Controller & .\Node) to run as separate processes.
3. Open two DOS-prompt windows, one in the .\Controller folder, the other in the .\Node folder.
4. From .\Controller, run `synapse.controller.cli service run`.  From .\Node, run `synapse.node.cli service run`.

```dos
C:\synapse\Controller>Synapse.Controller.Cli.exe service run
Starting Synapse.Server as Server: Press Ctrl-C/Ctrl-Break to stop.
2017-05-15 20:00:21,851|INFO |(1)|Starting
2017-05-15 20:00:23,178|INFO |(1)|Listening on http://localhost:20000
2017-05-15 20:00:23,910|INFO |(1)|Running


C:\synapse\Node>Synapse.Node.Cli.exe service run
Starting Synapse.Server as Node: Press Ctrl-C/Ctrl-Break to stop.
2017-05-15 20:02:15,013|INFO |(1)|Starting
2017-05-15 20:02:15,032|INFO |(1)|Initialized PlanScheduler, MaxThreads: 0
2017-05-15 20:02:15,215|INFO |(1)|Listening on http://localhost:20001
2017-05-15 20:02:15,239|INFO |(1)|Running
```

### Try it!

* **Smoke-test:** Open another DOS-prompt to .\Controller, execute `synapse.controller.cli hi`.  You may do similar for the Node.

```dos
C:\synapse\Controller>Synapse.Controller.Cli.exe hi
Calling Hello on http://localhost:20000/synapse/execute
"Hello from SynapseController, World!"
```

* **Run a Plan:** Execute `synapse.controller.cli s planName:sample dryRun:true`

```dos
C:\synapse\Controller>Synapse.Controller.Cli.exe s planName:sample dryRun:true
Calling StartPlan on http://localhost:20000/synapse/execute
636302779130701013
```

* **Check the result:** Look in the .\Controller\Dal\History folder to see the ResultPlan.  Open the file in any friendly YAML editor, such as Visual Studio Code or Notepad++.