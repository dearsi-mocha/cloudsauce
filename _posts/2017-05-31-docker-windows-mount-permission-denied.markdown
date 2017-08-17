---
layout: post
title:  "docker on windows mount source path permission denied"
date:   2017-05-31 17:35:03 -0700
categories: docker windows mount permission
---

Recently I was leveraging Azure App Services to deploy my Docker packaged .NET Core app. My setup includes VS 2017 v15.2, Docker CE v17.03.1-ce-win12 (stable) and Windows 10 Enterprise (with Creators update).

My app ran fine locally without Docker but as soon as I tried deploying to a Linux container VS gave me a weird error:

```
error while creating mount source path permission denied
```

I figured there was something funky going on with my Docker settings. Navigating to `Docker Client -> Settings -> Shared Drives` none of my drives were shared (also weird since I am pretty sure I had set them up earlier). Maybe the Windows 10 Creators update had something to do with that? Anyways...

Re-sharing my local drive with Docker, I uncovered another error:

```
Error: A firewall is blocking file Sharing between Windows and the containers. See documentation for more info.
```

![docker error](/assets/dockerFirewall.PNG)

I tried a number of times to share, including resetting cached credentials, using local credentials etc. No dice :(

Of course looking at the documentation sent me down some bunny trails around making sure inbound firewall rules were setup correctly between the Host and VM processes.

After a chunk of time researching the issue and trying a bunch of things, I have a solution that worked for me - one that might save you time. The solution actually has nothing to do with the error displayed!


1. Make sure your target drive is unshared `Drive Properties > Sharing > Advanced > 'Share this folder' is unchecked`
2. As part of installing Docker you should have a DockerNAT interface setup. Uncheck the File and Printer sharing property and press OK. `Adapter Properties > Networking > Uncheck File and Printer Sharing for Microsoft Networks`
3. Now reverse what you did i.e. check the same file and printer sharing property and hit OK.

After the following the outlined steps above, I was able to share my target drive with Docker with no issues. Seems a bit voodoo no? I hope the tooling will improve to side step this issue altogether in the near future.

[ref](https://stackoverflow.com/questions/42203488/settings-to-windows-firewall-to-allow-docker-for-windows-to-share-drive)