---
layout: post
title:  "leveraging ubuntu trusty packages in xenial"
date:   2017-09-25 16:10:00 -0700
categories: ubuntu xenial apt-get aptitude boost docker
---

In this scenario we need to install an older version of an Ubuntu package containing popular Boost C++ libraries, **libboost1.54-all-dev**, which is not available 'out of the box' on our Ubuntu 16.04 Xenial environment.

Doing a quick search on [Ubuntu packages](https://packages.ubuntu.com/search?keywords=libboost1.54-all-dev&searchon=names&suite=all&section=all) we see that the target package components are in the **trusty** and **trusty-updates** suites. You will notice for both suites the package components are in the **universe** repository. 

Starting off a [fresh Xenial image](https://hub.docker.com/_/ubuntu/)...

First let us append this detail to `/etc/apt/sources.list`. It is worth pointing out you could also add seperate file based sources in `/etc/apt/sources.list.d/*` where the source file ends in `.list`

```bash
echo "deb http://archive.ubuntu.com/ubuntu/ trusty main universe" | tee -a /etc/apt/sources.list
echo "deb http://archive.ubuntu.com/ubuntu/ trusty-updates main universe" | tee -a /etc/apt/sources.list
```

**Note:** I am using a Docker image where root is the default user and `sudo` is not installed. Use typical `sudo` patterns where needed.

Now that your `sources.list` has the new sources, run `apt-get update` to update the local package cache.

After updating, try running `apt-get install -y libboost1.54-all-dev`. If you are lucky it may work, or like for me you will get an unmet dependency error. Using `apt-get install -y -f ...` produced the same error.

```bash
root@19ccd6b11032:/ apt-get install -y libboost1.54-all-dev
Reading package lists... Done
Building dependency tree
Reading state information... Done
Some packages could not be installed. This may mean that you have
requested an impossible situation or if you are using the unstable
distribution that some required packages have not yet been created
or been moved out of Incoming.
The following information may help to resolve the situation:

The following packages have unmet dependencies:
 libboost1.54-all-dev : Depends: libboost-mpi1.54-dev but it is not going to be installed
                        Depends: libboost-mpi-python1.54-dev but it is not going to be installed
E: Unable to correct problems, you have held broken packages.
```

We have a couple of options, on one hand we could go and resolve the dependency issues ourselves..._Or_ we could use a higher level tool called `aptitude` that has more sophisticated package management capabilities to make it easier on ourselves.

Install aptitude with apt-get...

`apt-get install -y aptitude`

After aptitude finishes installing, try installing boost

`aptitude install libboost1.54-all-dev`

You should get a number of 'solutions' as options to deal with the dependency issues. Here is the solution that made the most sense in my case (and notice the 'trusty' tag):

```bash
The following actions will resolve these dependencies:

     Install the following packages:
1)     libopenmpi-dev [1.6.5-8 (trusty)]
2)     openmpi-bin [1.6.5-8 (trusty)]
3)     openmpi-common [1.6.5-8 (trusty)]

     Keep the following packages at their current version:
4)     libopenmpi1.10 [Not Installed]

Accept this solution? [Y/n/q/?] Y
```
Now that the previously unmet dependencies are out of the way, aptitude should prompt for installing the original target package

```bash
  The following NEW packages will be installed:
  autotools-dev{a} binutils{a} cpp{a} cpp-5{a} file{a} gcc{a} gcc-4.8-base{a} gcc-5{a} icu-devtools{a} libasan0{a}
  ...
  ...
  manpages-dev{a} mime-support{a} mpi-default-bin{a} mpi-default-dev{a} ocl-icd-libopencl1{a} openmpi-bin{a}
  openmpi-common{a} python{a} python-minimal{a} python2.7{a} python2.7-minimal{a} sgml-base{a} xml-core{a}
0 packages upgraded, 129 newly installed, 0 to remove and 0 not upgraded.
Need to get 107 MB of archives. After unpacking 486 MB will be used.
Do you want to continue? [Y/n/?] Y
```

If you are wondering what the `{a}` means, you can type `?` in the prompt

```
 In the list of actions to be performed, some packages will be followed by one or more characters enclosed in braces;
 for instance: "aptitude{u}".  These characters provide extra information about the package's state, and can include
 any combination of the following:

 'a': the package was automatically installed or removed.
 'b': some of the package's dependencies are violated by the proposed changes.
 'p': the package will be purged in addition to being removed.
 'u': the package is being removed because it is unused.
 ```

 After continuing with installation our environment should be ready to use the Boost C++ libraries!


References:
- [sources.list man pages](http://manpages.ubuntu.com/manpages/zesty/man5/sources.list.5.html)
- [boost C++](https://en.wikipedia.org/wiki/Boost_C%2B%2B_libraries)