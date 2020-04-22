---
layout: post
title:  "Azure API Management CLI woes...and workarounds"
date:   2020-04-22 17:35:03 -0700
categories: azure apim azurecli cli api
--- 

## Are you having issues with az apim create?

Here it is a small work around for the current bug when adding managed identity.

```json
az apim create --name $name `
               --publisher-email $publisher.email `
               --publisher-name $publisher.name `
               -g $resourceGroup #--enable-managed-identity true
az apim update --name $name -g $resourceGroup --enabled-managed-identity true
```
