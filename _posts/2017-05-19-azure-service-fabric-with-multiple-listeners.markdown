---
layout: post
title:  "azure service fabric with multiple listeners"
date:   2017-05-19 16:31:03 -0700
categories: azure servicefabric kestrel
---

If you are building an Azure Service Fabric application and you are configuring your replica listeners
i.e. overriding (in the case of a Stateful Service) 

`protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners(){...}`

One quite common scenario you might run into is the addition of >1 listener.

Well in that case don't do this or you will have a bad time:

```cs
return new List<ServiceReplicaListener>()
{
    new ServiceReplicaListener( (context) => this.CreateServiceRemotingListener(context)),
    new ServiceReplicaListener( (context) => this.CreateServiceRemotingListener(context))
};
```

You will likely run into a difficult to debug scenario where your fabric app is unable to succesfully deploy into your cluster, your debugger is not attaching, and you are pulling your hair out trying to determine why!

IF you are paying attention to your cluster diagnostics events -- at somepoint (too long for my taste!) you could get a clue to why the cluster state is unhealthy.

Bingo:

```
Unhealthy event: SourceId='System.RA', Property='ReplicaOpenStatus', HealthState='Warning', ConsiderWarningAsError=false.
Replica had multiple failures in_Node_0 API call: IStatefulServiceReplica.ChangeRole(P); Error = System.Fabric.FabricElementAlreadyExistsException (-2146233088)
Unique Name must be specified for each listener when multiple communication listeners are used
   at Microsoft.ServiceFabric.Services.Communication.ServiceEndpointCollection.AddEndpointCallerHoldsLock(String listenerName, String endpointAddress)
   at Microsoft.ServiceFabric.Services.Communication.ServiceEndpointCollection.AddEndpoint(String listenerName, String endpointAddress)
   at Microsoft.ServiceFabric.Services.Runtime.StatefulServiceReplicaAdapter.d__27.MoveNext()`
```

With respect to the current version of the Azure Service Fabric SDK, make sure the optional [Name] parameter is set to some unique value i.e. for 2 listeners `Foo1` and `Foo2`. Also both listeners (and unique name) is specified in the ServiceManifest.xml Resources>Endpoints>Endpoint declaration.

Here is a more complete example using Kestrel with .NET Core:

```cs
/// <summary>
/// Optional override to create listeners (e.g., HTTP, Service Remoting, WCF, etc.) for this service replica to handle client or  user requests.
/// </summary>
/// <remarks>
/// For more information on service communication, see https://aka.ms/servicefabricservicecommunication
/// </remarks>
/// <returns>A collection of listeners.</returns>
protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new List<ServiceReplicaListener>()
    {
        new ServiceReplicaListener( (context) => this.CreateServiceRemotingListener(context),"ServiceEndpoint1" ),
        new ServiceReplicaListener(serviceContext =>
            new KestrelCommunicationListener(
                serviceContext,
                (url, listener) => new WebHostBuilder().UseKestrel().ConfigureServices(
                     services => services
                         .AddSingleton<StatefulServiceContext>(this.Context)
                         .AddSingleton<IReliableStateManager>(this.StateManager))
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                .UseStartup<Startup>()
                .UseUrls(url)
                .Build()),"ServiceEndpoint2")
    };
}
```

And respective snippet from ServiceManifest.xml

```xml
<Resources>
  <Endpoints>
    <!-- This endpoint is used by the communication listener to obtain the port on which to 
         listen. Please note that if your service is partitioned, this port is shared with 
         replicas of different partitions that are placed in your code. -->
    <Endpoint Protocol="http" Name="ServiceEndpoint1" />
    <Endpoint Protocol="http" Name="ServiceEndpoint2" />

    <!-- This endpoint is used by the replicator for replicating the state of your service.
         This endpoint is configured through a ReplicatorSettings config section in the Settings.xml
         file under the ConfigPackage. -->
    <Endpoint Name="ReplicatorEndpoint" />
  </Endpoints>
</Resources>
```


