---
layout: post
title:  "Azure function VNet integration and Azure private DNS Zones"
date:   2020-04-22 17:35:03 -0700
categories: azure keyvault kv azurecli cli devops
--- 

Recently I was trying to lock down my azure functions to only consume services within a private VNet. 
I thought the process would be very simple but it took me sometime to figure it out as there was no documentation
on my specific scenario.

[this sample shows how to set up vnet integration on azure functions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-vnet) 

I was able to get through the sample but ideally I do not want to have IP addresses on all my azure functions. So I need a DNS to resolve 
these names for me. The recommendation by Azure is to use Azure private DNS Zones.
This was the goal:
[![](https://mermaid.ink/img/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG5BenVyZSBGdW5jdGlvbi0-PkludGVncmF0aW9uIFN1Ym5ldDogSSB3YW50IHRvIGdvIHRvIGh0dHBzOi8vcHJpdmF0ZS52bmV0LmNvbVxuTm90ZSByaWdodCBvZiBJbnRlZ3JhdGlvbiBTdWJuZXQ6IDooIG5vdCB3b3JraW5nIC4uLlxuSW50ZWdyYXRpb24gU3VibmV0LT4-IEF6dXJlIHByaXZhdGUgRE5TOiBwcml2YXRlLnZuZXQuY29tP1xuQXp1cmUgcHJpdmF0ZSBETlMgLT4-IEludGVncmF0aW9uIFN1Ym5ldDogaGVyZSBpcyB0aGUgSVBcbkludGVncmF0aW9uIFN1Ym5ldCAtLT4-IFByaXZhdGUgU2VydmljZSBlbmRwb2ludDogZmluYWxseSBjYW4gY2FsbCBwcml2YXRlIGVuZHBvaW50IGZyb20gZnVuY3Rpb24hXG5cbiIsIm1lcm1haWQiOnsidGhlbWUiOiJkZWZhdWx0In0sInVwZGF0ZUVkaXRvciI6ZmFsc2V9)](https://mermaid-js.github.io/mermaid-live-editor/#/edit/eyJjb2RlIjoic2VxdWVuY2VEaWFncmFtXG5BenVyZSBGdW5jdGlvbi0-PkludGVncmF0aW9uIFN1Ym5ldDogSSB3YW50IHRvIGdvIHRvIGh0dHBzOi8vcHJpdmF0ZS52bmV0LmNvbVxuTm90ZSByaWdodCBvZiBJbnRlZ3JhdGlvbiBTdWJuZXQ6IDooIG5vdCB3b3JraW5nIC4uLlxuSW50ZWdyYXRpb24gU3VibmV0LT4-IEF6dXJlIHByaXZhdGUgRE5TOiBwcml2YXRlLnZuZXQuY29tP1xuQXp1cmUgcHJpdmF0ZSBETlMgLT4-IEludGVncmF0aW9uIFN1Ym5ldDogaGVyZSBpcyB0aGUgSVBcbkludGVncmF0aW9uIFN1Ym5ldCAtLT4-IFByaXZhdGUgU2VydmljZSBlbmRwb2ludDogZmluYWxseSBjYW4gY2FsbCBwcml2YXRlIGVuZHBvaW50IGZyb20gZnVuY3Rpb24hXG5cbiIsIm1lcm1haWQiOnsidGhlbWUiOiJkZWZhdWx0In0sInVwZGF0ZUVkaXRvciI6ZmFsc2V9)

I easily created a private DNS Zone and added an A name for my private endpoint... Unfortunately my azure function was unable to resolve the name.
> I did some more digging and eventually found that you can enable this by adding WEBSITE_DNS_SERVER andWEBSITE_VNET_ROUTE_ALL  app settings to your function.

##### Azure DNS Private Zones
After your app integrates with your VNet, it uses the same DNS server that your VNet is configured with. By default, your app won't work with Azure DNS Private Zones. To work with Azure DNS Private Zones you need to add the following app settings:

  - WEBSITE_DNS_SERVER with value 168.63.129.16
  - WEBSITE_VNET_ROUTE_ALL with value 1
 
These settings will send all of your outbound calls from your app into your VNet in addition to enabling your app to use Azure DNS private zones.

And Voila that solved my issue, Domain name is resolved.
