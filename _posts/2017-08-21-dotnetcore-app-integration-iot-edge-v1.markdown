---
layout: post
title:  "dotnetcore app integration with iot edge v1"
date:   2017-08-21 16:10:00 -0700
categories: dotnetcore integration iot edge v1
---

This post is dedicated in stepping through key elements that you need to replicate in your app in order to module-ize and integrate with the IoT Edge v1 gateway.

The world of software changes fast, so keep in mind the versions of packages, modules and platform for this exercise. 

I am going to assume you have familiarity with what IoT Edge is. Otherwise please refer to the IoT Edge overview link near the bottom of the post.

We are going to focus on a dotnetcore v1.1 app - these steps are what worked for me and my team. The great thing is the outlined steps should be similar for other runtimes and languages.  


__Let's dive in...__

First create a new dotnet core project.

Add the following package reference, either directly to the .csproj or through the nuget interface

```xml
<PackageReference Include="Microsoft.Azure.Devices.Gateway.Module.NetStandard" Version="1.0.5" />
```
*Note*: Please use the latest version of the package available to you.

This package will bring in some dependencies like 

- Microsoft.Azure.Devices.Gateway.Native.Windows.x64
- Microsoft.Azure.Devices.Gateway.Native.Ubuntu.x64
- Microsoft.Azure.Devices.Gateway.Native.Debian.x64


Now lets build a concrete class implementing the edge inteface. We will build out the edge module lifecycle and message bus hooks here.

* Create a new .cs file in the root project directory. We will build out the edge gateway lifecycle and message bus hooks here.
    * Note the namespace
* Add `using Microsoft.Azure.Devices.Gateway;` 
* Create a new class, call it `ExampleAppEdge` for this exercise
    * This class needs to implement `IGatewayModule` and `IGatewayModuleStart`
    * i.e. `public class ExampleAppEdge : IGatewayModule, IGatewayModuleStart`
    * Stub out the interface methods that need to be implemented.

You should have a .cs file that looks similar to this

```cs
using Microsoft.Azure.Devices.Gateway;

public class ExampleAppEdge : IGatewayModule, IGatewayModuleStart
{
    private Broker broker;
    private string configuration;

    public void Create(Broker broker, byte[] configuration)
    {
    }

    public void Destroy()
    {
    }

    public void Receive(Message received_message)
    {
    }

    public void Start(){
    }
}
```
*Note*: `IGatewayModuleStart` is not required but it brings in the useful 'Start()' method which is called when the gateways message broker is ready to send and receieve messages to the target module.

To get the module-ized app to run, at a minimum we will need add some detail to the `Create(...)` method. This is where you would ideally bootstrap the application startup. Notice the `<Broker>` and `<byte[]>` types the interface method expects. The `Broker` type will be used if you are publishing outbound messages (for another module to receive). The `byte[]` type for this method is for configuration arguments when you pass in the declaritive JSON config to the gateway executable. We will look at the gateway JSON config a bit later. The `byte[]` encoding is UTF8, keep that in mind!

Because of the typical `IWebHost.Run()` method (where an `IWebHost` is returned from `WebHostBuilder.Build()`), the dotnet core module would run on the edge gateway in a foreground thread and block the calling thread - which is not great if you have other modules that need to spin up. They would be blocked until the dotnet core module is destroyed.

Let's circumvent that with a workaround which is to execute `IWebHost.Run()` in a background thead on the threadpool i.e. `Task.Run(() => Program.Main(null));`

So our Create now looks like 

```cs
public void Create(Broker broker, byte[] configuration)
{
    Console.WriteLine("ExampleApp.Create called!");
    this.broker = broker;
    this.configuration = Encoding.UTF8.GetString(configuration);
    Task.Run(() => Program.Main(null));
}
```

Now let's take a look at the `Receive(...)` interface method. The message broker will execute this method if the target module is specified in `links` as `sink` in the gateway config. Again more on the gateway config later.

You'll notice the `<Message>` type param the method expects. For receiving messages the two most important members of `<Message>` are `byte[] Content` and `Dictionary<string, string> Properties`. The purpose of Content is self explanatory but the Properties? You can think of Properties as metainfo about the Content. When your module publishes messages, you can add message properties for richer interaction downstream. For example, if your module is publishing sensor data and you have other modules publishing distinct data types why not add a property with key `'DataType'` and value `'Sensor'` or `'DataType'='OtherType'` for others?

Here is a basic implementation of Receive, also taking advantage of async

```cs
public async void Receive(Message received_message)
{
    Console.WriteLine("ExampleApp.Receive called!");
    string recMsg = Encoding.UTF8.GetString(received_message.Content, 0, received_message.Content.Length);
    Dictionary<string, string> props = received_message.Properties;

    if(props.ContainsKey("DataType") && props["DataType"].Equals("Sensor"))
    {
        var data = GetDataTokenFromJson<SensorData>(recMsg);
        await PersistenceManager.Instance.AddSensorDataAsync(data);
    }
    else {
        DoStuff();
    }

    DoSomeProcessing();
    DoMoreProcessing();
}
```

Thus far we have outlined steps for a mock application. We actually built a dotnetcore app that had exposed REST endpoints, a websocket implementation (for nice push notifications on the front end) AND integration into IotHub for just-in-time device creation and message simulation.

To learn more about device management and how you can bring it to your app with practical examples checkout [this blog post](http://katngov.com/2017/08/24/iot-device-management-web-app/)!


References:
- [iot edge overview](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-iot-edge-overview)
- [simulated ble module sample](https://github.com/Azure-Samples/iot-edge-samples/tree/master/dotnetcore/simulated_ble)
- [dotnet core module sample](https://github.com/Azure/iot-edge/tree/master/samples/dotnet_core_module_sample)

