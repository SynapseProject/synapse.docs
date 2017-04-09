#Architecture and Usage Patterns

## Synapse Workflow Engine

<p align="center">
<img alt="Synapse Engine" src="../img/syn_engine.png" />
</p>

The Synapse workflow engine processes actions by tasking to a Handler.  Each Handler is responsible only for its specific technology capability, whether that be local, or remote.  Handlers must also report status back to the calling engine.

### Engines as a Proxy

In order to simplify workflow processing, when considered as an end-to-end logical unit, a Synapse workflow engine can proxy Actions to other Synapse engine instances.  This is useful also when crossing network/firewall bourndaries to minimize exposing wide port ranges.  Under this model, Action subtrees are detached from the main Plan and forwarded to a remote engine for execution.

### Local Engine Processing Model

When invoked as a local process, such as through Synapse.cli, a Synapse workflow engine is processing work as an island, but still maintains the capability to proxy Actions to remote Synapse engines.  The primary difference is the request/response model does not participate in the Enterprise messaging mechanism.

## Synapse Enterprise

<p align="center">
<img alt="Synapse Enterprise" src="../img/syn_architecture.png" />
</p>

Synapse Enterpise is implemented to provide flexibility and scalability in high-volume environments.  The Enterprise service edits the RBAC and publishes Plans to the data access layer.  The Controller receives processing requests, validates the caller against the RBAC, fetches the Plan, and then forwards the request to available Synapse Node instances.  Nodes, in turn, return status back to the Controller for auditable logging.

### Components

| Component | Description
|--------|--------
|Synapse.Enterprise|Provides methods to edit the RBAC, the logical Plan container hierarchy, initiate Plan execution, and fetch Plan status.  Synapse.Enterpise is the "out of the box" implementation for enterprise-class installations.
|Synapse.Controller|Provides methods to initiate Plan execution, accept Plan status responses from Synapse.Node, and fetch Plan status.
|<p align="right">_Note:_|Synapse Enterpise and Controller form a single logical unit.  Developers seeking to integrate Synapse with existing applications should enter directly with Controller and consider utilizing the SDK for custom URI, RBAC, and data processing options.   
|Synapse.Node|A wrapper on the Synapse Workflow Engine, Synapse.Node is designed to be set up in groups for distributed parallel processing models.


