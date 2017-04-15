# Synapse SDK Overview

The Synapse SDK provides extensibility and integration opportunities, so you can meet new technical and business requirements as they arise.

## Custom Handlers

Handlers are the basic building blocks of Synapse technical capability.  By creating new Handlers, you can tightly control the interface between automation data and technology.  This provides you direct access to manipulate your data _as_ data, and invoke your technology per your specific requirements.  For example, by creating a new Handler, you may choose to consume a 3rd party API or SDK within the Handler, thus providing a method to evalute and mutate parameter data ahead of invoking the 3rd-party technology, and also affords advanced error handling techniques.

### Handler Processing Pipeline

Creating a custom Handler is intentionally a straighforward, easy to understand process.  The Handler processing pipeline consists of only four parts:

1. Initialize with custom Handler Config data (optional step; omit if not required)
2. Execute
3. Raise "Progress" events, Log process detail
4. Return "Exit Data" (could simply be exit/summary data, or any complex data-return requirement)

<p align="center">
<img alt="Synapse Handler" src="../../img/syn_handler.png" />
</p>


For more detailed information and links to code expamples, see the [Handlers SDK page](/sdk/handlers/ "Handlers SDK page").

## Custom Controllers

### Method 1: Synapse Controller Service + Your Custom ApiController

Custom ApiControllers are for extending the "look & feel" of Synapse's API, or for introducing custom business logic.  The out-of-the-box Controller interface expects GET/POST operations taking Synapse Plans as input, and returns Synapse Actions & Plans as output.  These Synapse unit act as envelopes for your technical & business data, thereby abstracting external processing requirements.

If you want to use the Synapse Controller capability, but you want a more natural feel to your REST API, you may choose to inject a custom Microsoft Web Api ApiController into the Synapse Controller.  Custom ApiControllers participate in the REST transaction pipeline independent from the native Synapse Controller interfaces, thus enabling complete control of input data, processing, and return data.

<p align="center">
<img alt="Synapse Custom ApiContoller" src="../../img/syn_customContoller.png" />
</p>

For more detailed information and links to code expamples, see the [Controllers SDK page](/sdk/controller/ "Controllers SDK page").

### Method 2 - Roadmap: Your Web Api Application + Synapse Controller

In a future release of the Synapse Controller, we will provide Synapse.Server as a library that you can hook via a standard compile-time dependency.  Under this model, your application must provide the runtime execution hosting (such as with IIS or OWIN self-hosted), but you get the advantage of direct integration with your existing codebase.  This model is appropriate when custom runtime hosting requirements arise, or if you already have an established application design and you simply want to augment your capability with Synapse.