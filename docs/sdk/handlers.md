# Custom Synapse Handlers

Synapse Handlers provide techncial capability to the Synapse Workflow Engine.  Creating a custom Handler is the primary means of extending Synapse for scenarios.

## "Free" Extensibility

Perhaps the easist way to add capability is the through the existing CommandHandler.  The CommandHandler can already execute any arbitrary command-line, and has special helper interfaces for scripts and batch files.  You can create new "technical modules" as scripts or whole logical processing workflows and integrate via the CommandHandler, as-is.  See the <a href="https://github.com/SynapseProject/synapse.examples/tree/develop/Plans/CommandLine" target="_blank">CommandHandler Examples (develop branch)</a>.  You can also [read the docs](/handlers/command/ "CommandHandler").

## Code-based Custom Handlers

If you want more control over your Handler, you may implement a compiled solution.  Synapse.Core contains all the interfaces and base classes necessary, so you only need the single dependency.  Wanna jump right in?  See the <a href="https://gist.github.com/SynapseProject/b8747c156843dad7119f6135d320f7bf" target="_blank">Custom Handler Example</a>.

### IHanderRuntime

To create a new Handler, create a new Class Library project (dll), and add a reference to <a href="https://www.nuget.org/packages/Synapse.Core.Signed" target="_blank">Synapse.Core from nuget.org</a>.

At a high level, the implementation flows as:

1. **Initialize**: receives the Action Config data, serialized as a string.  As an implementer, you must specify what serialization formats you accept (YAML, JSON, and XML).  Use this method override to execute any necessary setup code for your Handler.
2. **Execute**: receives the Action Parameter data, serialized as a string.  As with Config, you must specify what serialization formats you accept.  See the full `HandlerStartInfo` class for details on other members, but take care to implement the `IsDryRun` member so your Handler supports testing.  **ExecuteResult** is returned from Execute method upon workflow completion.
3. **Progress**: Advertises the runtime status of your internal workflow.  `HandlerProgressCancelEventArgs` implements a `Cancel` member to support external graceful-cancellation requests.
4. **LogMessage**: Override to log workflow detail during processing.  This is present to support general logging patterns as any application would.

```java
    public interface IHandlerRuntime
    {
        //implmement simple getter/setter; value set by the runtime engine
        string ActionName { get; set; }
        //implmement simple getter/setter; value set by the runtime engine
        string RuntimeType { get; set; }

        IHandlerRuntime Initialize(string config);
        ExecuteResult Execute(HandlerStartInfo startInfo);

        event EventHandler<HandlerProgressCancelEventArgs> Progress;
        event EventHandler<LogMessageEventArgs> LogMessage;
    }
```

### HandlerRuntimeBase

HandlerRuntimeBase provides a default implementation for `IHanderRuntime`, plus some helper methods for deserialzing Config/Parameters.  You may implement `IHanderRuntime` or inherit from `HandlerRuntimeBase` as you see fit.

## Example Implementation Code

See the examples on GitHub.

- <a href="https://github.com/SynapseProject/synapse.examples/tree/develop/Plans/CommandLine" target="_blank">CommandHandler Examples (develop branch)</a>
- <a href="https://gist.github.com/SynapseProject/b8747c156843dad7119f6135d320f7bf" target="_blank">Custom Handler Example</a>