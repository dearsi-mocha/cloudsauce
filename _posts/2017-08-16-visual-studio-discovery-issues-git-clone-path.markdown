---
layout: post
title:  "visual studio project discovery issues due to git clone path"
date:   2017-08-16 16:10:00 -0700
categories: git clone path encoding visual studio dotnetcore
---

Here is a short and sweet headache causing issue with a resolution.

When you are git cloning a project to work on, particularly when you aren't providing a clone folder argument, **make sure you remove or replace special characters such as url encoding ones like %20 from the path.**

Otherwise visual studio will have issues discovering project artifacts...you know, important things like .cs files. 

Other side effects may be observed.

This happened to me using VS 2017 and Git 2.14.1. I had cloned a project with a space in the name not unlike the following path

`git clone https://proteam.visualstudio.com/_git/awesome%20project`

which dumped the project into root `awesome%20project`.

Opening the csproj or solution within the root in Visual Studio was a puzzling experience with code files missing, builds failing and no 'smoking gun' errors or exceptions provided.

Certainly VS was just as confused! ;) 
