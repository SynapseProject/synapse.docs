
# Synapse Custom Controller from Visual Studio

If you have a more complex requirement than is serviced from the Synapse.CustomController commandline utility, you may choose to code an ApiController by hand in Visual Studio.  The easiet way to get started is to create a new Class Library project (dll), and add a reference to <a href="https://www.nuget.org/packages/Synapse.Server.Extensibility" target="_blank">Synapse.Server.Extensibility from nuget.org</a>.  This will add all the necessary Web Api dependencies, and Synapse.Server.Extensibility contains a few helper utilities to aid in your dev.


#### A Note on Native ApiControllers

You may choose to implement a "regular" .NET ApiController, that is, a Controller sans any Synapse references, and Synapse Server will load it just fine.  The Synapse.Server.Extensibility library mentioned above is simply a helper lib.  Working without Synapse.Server.Extensibility just means you need to bridge into Synapse.Controller on your own, whereas Synapse.Server.Extensibility provides utility helper classes to do so for you.

## Example of manually-coded Custom Synapse Controller

<script src="https://gist.github.com/SynapseProject/0f345c4fa60cdb53ae8d3585cde24513.js"></script>
