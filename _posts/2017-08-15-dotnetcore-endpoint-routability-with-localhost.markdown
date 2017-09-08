---
layout: post
title:  "dotnetcore 1.x endpoint routability with localhost binding"
date:   2017-08-15 16:10:00 -0700
categories: dotnetcore kestrel endpoint binding
---

Quite recently, I was integrating some components for a platform project I was involved with. Among the components was a dotnetcore web app which was exposing an API endpoint. The endpoint was binding and serving requests properly locally, but not when it was deployed in a container on the remote host.

The solution to this issue was an `ohh of course...` (or `/sigh`) moment but it did take a chunk of time to resolve - so maybe some others out there could use the info, or at least recache and consider it a refresher.
 
Here was the WebHostBuilder() problem code...

```
    public static void Main(string[] args)
    {
        var host = new WebHostBuilder()
            .UseKestrel()
            .UseContentRoot(Directory.GetCurrentDirectory())
            .UseStartup<Startup>()
            .UseUrls("http://localhost:8081")
            .Build();

        host.Run();
    }
```

Lets focus in on the `UseUrls(...)` method. If you don't provide any urls .NET core will bind to http://localhost:5000. For my case I had to override this behavior.

UseUrls takes params string[], so you could pass in a collection of endpoints.

Example:

`UseUrls("http://localhost:8081","http[s]://127.0.0.1","http://[::1]")`

(note [::1] is the IPv6 loopback)

or you can pass a single string with semi-colon delimited urls i.e.

`UseUrls("http://localhost:8081;http[s]://127.0.0.1")`

OK thats great, but why is the endpoint not serving requests when it is deployed?

The answer is _because of the loopback address_. This causes the server to only service requests coming from localhost.

This behavior is not unique to dotnet.

In fact, typically you'd want your firewall to drop packets with destination 127.0.0.1 for security reasons.

The key here is to set your address
- to be the wildcard char `*` e.g. `http://*:8081` (bind to any IP address or host)
- `0.0.0.0` like `http://0.0.0.0:8081` (bind to all IPv4 addresses)
- `[::]` like `http://[::]:8081` (bind to all IPv6 addresses)


Assuming your port mapping rules are correct (if you have any) that should solve the problem.
  
  
References:
- [dotnetcore hosting](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/hosting?tabs=aspnetcore1x)
- [kestrel config](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/servers/kestrel?tabs=aspnetcore1x#endpoint-configuration)
