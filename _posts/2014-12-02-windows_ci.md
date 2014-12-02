---
layout: post
title:  "Windows, Jenkins, and Continuous Integration"
date:   2014-12-01 10:30:00
author: apierce
categories: devops continuous integration delivery jenkins
---

In 2011, I vowed never to install Windows again. I had more than a couple
disparaging opinions about the operating system, chief of which was that it was
difficult to use. Finding and installing development packages? A pain. Having a
terminal without VI keybindings and all those helpful tools that make up most
Unix operating systems? A tragedy!

Imagine my dismay, then, when I discovered a client of ours had an application
built on (and served by) Windows and the .NET stack, and I was to be cast back
into the World of Windows I swore I'd leave behind. However, the Buddha tells
us:

> The root of suffering is attachment

And so it was that I stopped clinging to the safety and familiarity of Unix,
and approached the task at hand with an open mind. Enlightenment dawned quickly
as I discovered:

## Windows is Not Difficult to Work With

To all the Windows developers who live in this world already, this post will
probably not teach you anything new. But to my fellow Knights of the Unix Order,
I say to you: disparage not the Windows Operating System! You may be surprised
by what you discover.

This post is going to discuss how easy it is to set up a [Jenkins](http://jenkins-ci.org/)
Continuous Integration server on a Windows machine, and why we would do such a
thing when there are perfectly fine ways to install Jenkins in Unix.

# Why Jenkins on Windows?
We need to install Jenkins on Windows because we need to build and deploy .NET
projects. To do this, we are using the following tools:

+ [Windows PowerShell](http://en.wikipedia.org/wiki/Windows_PowerShell)
+ [MSBuild](http://msdn.microsoft.com/en-us/library/wea2sca5%28v=vs.90%29.aspx)
+ [MSDeploy](http://www.iis.net/downloads/microsoft/web-deploy)

We're using PowerShell to automate the build and deploy tasks in Jenkins
(referencing MSBuild and MSDeploy in our scripts) whenever code changes are
pushed to git. While PowerShell has a Unix equivalent (shell scripting),
MSBuild and MSDeploy unfortunately do not. This is why we need to install
Jenkins on a Windows machine, a process which turned out to be much easier
than expected!

# Chocolatey: A Windows Package Manager
Yes, Virginia, there IS a package manager for Windows! And its name is
[Chocolatey](https://chocolatey.org/) (a play on the NuGet package manager used
by VisualStudio for managing development packages).

Once you've installed Chocolatey on your Windows system, installing Jenkins
(or any other package for that matter) is simple. Simple!

# Installing Jenkins In One Command
```
C:\> choco install jenkins
```

# Setting up Build and Deploy in Jenkins
Once Jenkins is installed (wasn't that simple?), now comes the "hard part" of
setting up our build and deploy scripts. Fortunately, Jenkins on Windows comes
pre-configured to run PowerShell scripts in Build and Post-build Actions, so
you can automate your builds like so:

+ Manage Jenkins -> Download Plugins:
    - Git Plugin(s)
    - MSBuild plugin
+ Configure Project:
    - [Source Code Management] Configure your git repository
    - [Build Triggers] "Build when a change is pushed to GitHub"
+ Project "Build" Section:
    - Batch command to run `NuGet.exe restore` on the `.sln` file we just checked out
    - "Build a Visual Stuido project or solution using MSBuild"

## Continuous Integration PowerShell Scripts
+ Batch command to build the project package:

```
& $msbuild $projectdir$project /target:Package /p:Configuration=Release /P:PackageLocation=$packageLocation
```

+ Batch command to deploy the package we just built:

```
& $msdeploy -verb:sync -source:"package=$contentPath\caservice.zip" -dest:"auto,computerName=https://$($destination):8172/msdeploy.axd,authType=Basic,userName=$username,password='$password'"
```

+ Batch command to run some [JMeter](http://jmeter.apache.org/) tests to validate the deployment
+ Post-Build Section:
    - JMeter performance metrics

The Continuous Integration magic is mostly in the (abbreviated) PowerShell
scripts above that invoke MSBuild and MSDeploy; just add your own variables!

# Conclusion
Fellow Knights of the Unix Order, the evidence is clear: between package managers,
scripting, and familiar tools like Jenkins, Windows has proven itself as an
operating system capable of handling the best-practice tasks you may require of
it. I say again: disparage not the Windows Operating System. It can get the job
done easier than you might expect!

<span style='font size:"0.2em";'>I still prefer Unix for basically everything, though.</span>
