---
layout: post
title:  "iot edge v1 config analysis"
date:   2017-08-21 16:10:00 -0700
categories: dotnet core app integration iot edge v1
---

Let's take a look at how the IoT Edge v1 config works.

The gateway_config.json (arbitrary file name) has 3 primary aspects which manifest as collections within a JSON object. This config file is passed to the gateway exe.

First there is the 'loaders' collection. Each element of this collection will identify a language type.

You are telling edge you want to load a module and that module will be executed using the target language binding.
The binding allows the Edge runtime to execute your module code written in a supported language.

Note you do not have to specify the native language binding.

IoT Edge v1 as of this post date supports .NET Core, .NET, Java, Node.js and C.

As an example, here I am defining two language bindings

```json
{
    "loaders": [
        {
            "type": "node",
            "name": "node",
            "configuration": {
                "binding.path": "nodejs_binding.dll"
            }
        },
        {
            "type": "dotnetcore",
            "name": "dotnetcore",
            "configuration": {
                "binding.path": "dotnetcore.dll",
                "binding.coreclrpath": "C:\\Program Files\\dotnet\\shared\\Microsoft.NETCore.App\\1.1.2\\coreclr.dll",
                "binding.trustedplatformassemblieslocation": "C:\\Program Files\\dotnet\\shared\\Microsoft.NETCore.App\\1.1.2\\"
            }
        }
    ]
    
    // Next section
    // ...
}
```

Next there is the 'modules' collection. This is where you define your modules, module arguments, and module language bindings for Edge to load and execute.

Here is an example where I am defining 3 modules:

- The native IoT Hub module with args pointing to my IoT hub.
- The native identity map which will translate device mac addresses to an Azure device.
- A custom http data intake API. In my particular use case the devices are able to make http calls, so I need a module to provide the data receive endpoint.

```json
{
    // Previous section
    // ...

    "modules": [
        {
            "name": "IotHub",
            "loader": {
                "name": "native",
                "entrypoint": {
                    "module.path": "iothub.dll"
                }
            },
            "args": {
                "IoTHubName": "thehubname",
                "IoTHubSuffix": "azure-devices.net",
                "Transport": "HTTP"
            }
        },
        {
            "name": "mapping",
            "loader": {
                "name": "native",
                "entrypoint": {
                    "module.path": "identity_map.dll"
                }
            },
            "args": [
                {
                    "macAddress": "01:01:01:01:01:01",
                    "deviceId": "sensor-id-2302192",
                    "deviceKey": "ABCDEFGHIJKLMNOP="
                },
                {
                    "macAddress": "02:02:02:02:02:02",
                    "deviceId": "sensor-id-7948391",
                    "deviceKey": "ABCDEFGHIJKLMNOP="
                }
            ]
        },
        {
            "name": "dataapi",
            "loader": {
                "name": "node",
                "entrypoint": {
                    "main.path": "nodejs_modules/dataapi.js"
                }
            },
            "args": null
        }
    ]

    // Next section
    // ...
}
```

The final component of the Edge gateway config is the 'links' collection.

This is describing input and output streams of target modules. Each object in links will contain a 'source' and a 'sink'. Think of it like this, for each object the 'source' is describing the data a module produces, will be consumed by the 'sink' module.

Example:

```json
{
    // Previous section
    // ...

    "links": [
        {
            "source": "dataapi",
            "sink": "mapping"
        },
        {
            "source": "mapping",
            "sink": "IotHub"
        }
    ]
}
```

In the links example, data produced by the 'dataapi' (what is published to the broker) i.e in Javascript:

```javascript
    broker.publish({
        // payload.properties.macAddress == '01:01:01:01:01:01'
        properties: payload.properties,
        content: 
            // Published content is in the form of a byte array.
            new Uint8Array(Buffer.from(
                JSON.stringify(
                    [{'name':'data',"content": [payload.data]}], 'utf8', this.UInt8ArrayFormatter
                )
            ))
    });
```

will be received by 'mapping' which will process the message and publish a new one (the new message meta data should include the azure device id and key)...this will be received by 'IotHub' which will push the message to the target Azure IoT Hub.

One other cool thing to mention is that you can apply Advanced Stream Analytics modules to your Edge deployment. For example if you need to filter out junk data.

That should cover key aspects of the IoT Edge v1 gateway config. Hope you enjoyed reading and happy IoT'ing! 


References:
- [iot edge overview](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-iot-edge-overview)
- [simulated ble module sample](https://github.com/Azure-Samples/iot-edge-samples/tree/master/dotnetcore/simulated_ble)
- [dotnet core module sample](https://github.com/Azure/iot-edge/tree/master/samples/dotnet_core_module_sample)

