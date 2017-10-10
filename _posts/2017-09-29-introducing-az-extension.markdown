---
layout: post
title:  "introducing az extension"
date:   2017-09-29 16:10:00 -0700
categories: azure cli extension python
---

Recently a powerful new feature was added to the [Azure CLI](https://github.com/Azure/azure-cli). It is called the 'extension' capability as in `az extension`.

An extension allows you to add new commands or override existing ones within Azure CLI from a separate package. Within an extension you are able to import any modules from the existing azure cli core namespace.

Meaning dev teams outside of the Azure CLI team can own code bases, tooling and other resources to build out new features for Azure CLI!

You can think of an extension like a plugin for an existing product. In this same vein, an extension depends on Azure CLI to work properly. It is **not** a standalone component.

Take a look at the [Azure IoT CLI Extension project](https://github.com/Azure/azure-iot-cli-extension) for a succint example of an available extension that expands existing IoT capabilities.

To access the extension command space, make sure you have [installed a recent version](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) of Azure CLI that supports extensions.

Let's take a look at some key features.

You can add a packaged (wheel format only) extension to Azure CLI by executing

```
az extension add --source <filepath.whl OR uri for .whl>
```

Meaning prior to adding the extension you should have the extension.whl locally or on some routable location ([Pypi](https://pypi.python.org/pypi/azure-cli-iot-ext) is an excellent choice).

If your extension package has dependencies declared in `setup.py` then they will be downloaded and installed during the add process.


To see which extensions are already installed in your Az CLI environment run

```
az extension list
```


To look in detail at an installed extension configuration run

```
az extension show --name extension_name
```

This will output key information such as dependencies, supported Python versions and declared metadeta used by Az CLI for installation such as `"azext.minCliCoreVersion": "2.0.16"`.



Then to delete an extension by name run

```
az extension remove --name extension_to_remove
```


References:
- [Az CLI Extension docs](https://github.com/Azure/azure-cli/tree/master/doc/extensions)
- [Azure IoT CLI Extension project](https://github.com/Azure/azure-iot-cli-extension)