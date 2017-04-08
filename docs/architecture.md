#Architecture and Usage Patterns

## Synapse Node

<p align="center">
<img alt="Synapse Node" src="../img/syn_engine.png" />
</p>

The Synapse Node processes workflow actions by tasking to a Handler.  Each Handler is responsible only for its specific technology capability, whether that be local, or remote.  Handlers must also report status back to the calling Node.

### Nodes as a Proxy

In order to simplify workflow processing, when considered as an end-to-end logical unit, a Synapse Node can proxy Actions to other Synapse Node instances.  This is useful also when crossing network/firewall bourndaries to minimize exposing wide port ranges.  Under this model, Action subtrees are detached from the main Plan and forwarded to a remote Node for execution.

### Local Node Processing Model

When invoked as a local process, such as through Synapse.cli, a Synapse Node is processing work as an island, but still maintains the capability to proxy Actions to remote Synapse Nodes.  The primary difference is the request/response model does not participate in the Enterprise messaging mechanism.

## Synapse Enterprise

<p align="center">
<img alt="Synapse Enterprise" src="../img/syn_architecture.png" />
</p>

Synapse Enterpise is implemented to provide flexibility and scalability in high-volume environments.  The Enterprise service edits the RBAC and publishes Plans to the data access layer.  The Controller receives processing requests, validates the caller against the RBAC, fetches the Plan, and then forwards the request to available Synapse Node instances.  Nodes, in turn, return status back to the Controller for auditable logging.