---
layout: post
title:  "Azure keyvault secret names do not support special characters.."
date:   2020-04-22 17:35:03 -0700
categories: azure keyvault kv azurecli cli devops
--- 

I was trying to deploy a certificate using azure cli with a valid password. If the password contains a special character, 
you get the following error when running import.

```powershell
  az keyvault certificate import --vault-name $vaultname `
                               -n $certificateName `
                               -f $pfxFilePath `
                               --password $env:PWD
                               
  az keyvault certificate import: error: argument --password: expected one argument
```

This is a current open issue [here](https://feedback.azure.com/forums/906355-azure-key-vault/suggestions/33931111-secret-names-do-not-support-special-characters). Go upvote it!
