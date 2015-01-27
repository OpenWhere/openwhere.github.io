---
layout: post
title:  "Automatically Deploying .NET applications to EC2 Instances with Jenkins"
date:   2015-01-28 10:30:00
author: apierce
categories: devops continuous integration delivery jenkins ec2 webdeploy .net
---

The Internet is full of articles trying to convince you of the [benefits of
continuous delivery](http://lmgtfy.com/?q=Benefits+of+continuous+delivery).
Presumably, you've had the opportunity to partake of the Kool-Aid, and now you
want to learn how to make it happen.

This article will walk you through the process of setting up a continuous
delivery pipeline for your .NET web applications using Jenkins, AWS EC2
instances, and Microsoft WebDeploy.

## Assumptions
Before you start, please note that this article makes the following assumptions

+ You already have a .NET project you want to build and deploy to the cloud
+ You keep your source code in a git repository
+ You already have an [AWS account](http://aws.amazon.com)
+ You've already stood up [Jenkins](http://jenkins-ci.org/) on an EC2 instance.
This Jenkins server should be running on a Windows host so that you have access
to [PowerShell](https://technet.microsoft.com/library/dn425048.aspx) scripting.

## 1. Install MSBuild on your Jenkins Server
Jenkins will handle checking your code out of GitHub and building it for
deployment. Building the code, however, requires the presence of [MSBuild](https://msdn.microsoft.com/en-us/library/ms171452(v=vs.90).aspx).
The simplest way to ensure MSBuild is installed on your Jenkins box is to
download and install [Microsoft Build Tools](http://www.microsoft.com/en-us/download/confirmation.aspx?id=40760)
on your Jenkins server.

Once the install is complete, double-check that `MSBuild.exe` exists in

    C:\Program Files (x86)\MSBuild\12.0\Bin

## 2. Install NuGet on your Jenkins Server
NuGet is a package manager. It's what we'll use to download the dependencies
your project needs.

Follow the instructions for setting up NuGet from the [Official Website](http://docs.nuget.org/docs/start-here/installing-nuget).

The remainder of this tutorial will assume that NuGet executable lives here:

    C:\NuGet\NuGet.exe

## 3. Create a New Jenkins Job
Name the Jenkins Job after your application

![Image]

## 4. Set up Source Code Management in the Job
Set up Source Code Management to pull from Git. This is an involved step that
others have solved before, so I will link
[additional](http://blog.loadimpact.com/2013/11/29/bootstrap-your-ci-with-jenkins-and-github/)
[resources](http://fourword.fourkitchens.com/article/trigger-jenkins-builds-pushing-github)
for how to step through this process.

![Image]

## 5. Set up NuGet to run on your project
After the project is pulled down from Git, running `NuGet restore` will download
all of the dependencies we do not yet have on the Jenkins box, enabling us
to build the project.

Select `Execute Windows Batch Command` from the `Add Build Step` dropdown.
Then, paste the following script in the box:

    C:\NuGet\NuGet.exe restore <YourSolution.sln>

... where `<YourSolution.sln>` is the name of your .NET solution file.

![Image]

## 6. Build the Project
In Jenkins, hit "Apply," and then (on the left-hand side) "Build Now."
This will create the project and attempt to check the code out of Git.

![Image]

If the build fails, resolve any problems before continuing on to the next step.

If the build succeeds, you should now have a directory containing your project
in the following (or a very similar) directory:

    C:\Program Files (x86)\Jenkins\jobs\AuditService\workspace\<YourApplicationName>

## 7. Run MSBuild on Your Source Code
Back to our Jenkins job! From the `Add Build Step` dropdown, select
`Windows PowerShell`. If this option is not available to you, download the
[PowerShell Plugin](https://wiki.jenkins-ci.org/display/JENKINS/PowerShell+Plugin)
from the "Manage Plugins" section of the Jenkins Management Console.

In the new Windows PowerShell text area, paste the following script:
```
# Build a project package

$msbuild = "C:\Program Files (x86)\MSBuild\12.0\Bin\MSBuild.exe"
$projectPath = 'AuditService\AuditService.csproj'
$packageLocation = "C:\builds\auditservice.zip"

& $msbuild $projectPath /target:Package /p:Configuration=Release /P:PackageLocation=$packageLocation
```
... where `$projectPath` is the relative path to your `.csproj` file from the
workspace directory path identified in Step 6, and `$packageLocation` is the
path to which the project should be built.

![Image]

## 8. Provision a new EC2 Instance
For our application, we're going to provision a new `Windows Server 2012 R2 Base`
EC2 instance.

Choose the instance type that best suits your needs; in our case, we're
deploying a MicroService, so we're going to help ourselves to a `t2.micro`.

**NOTE:** Save yourself some time later, and add the following ports to the
security group:
    + 3389      (RDP)
    + 80        (HTTP)
    + 443       (HTTPS)
    + 8172      (WebDeploy)

![Image]

It is **ESPECIALLY IMPORTANT** that you ensure that the WebDeploy port (8172)
is accessible from our Jenkins server! Otherwise, you will not be able to
perform remote builds whenever the code is updated.

## 9. Install WebDeploy on the new EC2 Instance
Using RDP, connect to the server we just provisioned.

Follow [this guide](http://www.iis.net/learn/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later)
(kindly provided by Microsoft) to get WebDeploy up and running on our new instance.

The short version is:
+ Search for "Recommended"
+ Install "Recommended Server Configuration for Web Hosting Providers"
+ Search for "IIS"
+ Install "IIS: ASP.NET 4.5"

Note that your application may require other dependencies, but you can always
go back and run the Web Platform Installer again. Just remember to restart
IIS afterwards!

## 10. Create a new user and configure WebDeploy

### Create User
In order to deploy builds securely, we're going to create a new user `Jenkins`,
and grant this user permissions to use WebDeploy.

In Control Panel -> User Accounts -> User Accounts -> Manage Accounts,
click `Add a user account`.

Username is `Jenkins`

Enter a Password

### Configure WebDeploy
If you followed Step 9 by the book, this might already be done. The guide
referenced will have detailed screenshots starting from the section entitled
`USING POWERSHELL TO CONFIGURE WEB DEPLOY FOR A NON-ADMINISTRATOR`. If you have
not yet followed this section of the guide, the short version is:

+ Open IIS
+ Expand the Computer Name -> Sites -> Default Web Site on the left-hand side
+ Double-click on IIS Manager Permissions
+ Add the Jenkins user
+ Right-click on Default Web Site, select Deploy, select "Configure Web Deploy Publishing"
+ Select the Jenkins user from the top dropdown
+ Click Setup

### Re-register ASP.NET:
Into a command prompt, paste:

    c:\Windows\Microsoft.NET\Framework\v4.0.30319\aspnet_regiis.exe -i

## 11. Add Remote Deployment to the Jenkins Job
In our Jenkins Job, from the `Add Build Step` dropdown, select `Windows
PowerShell`. In the resulting text area, paste the following script, changing
all variables beginning with `YOUR_...`

If you want to deploy to multiple machines, add multiple lines to the
`deploymentBoxes` array.

```
# Deploy the package we just built

# Define destination boxes
$deploymentBoxes = @(
    @{"destination"="YOUR_IP_ADDRESS"; "username"="Jenkins"; "password"="YOUR_PASSWORD"
);


function deploy($destination, $username, $password) {
    $webServerName = "https://$($destination):8172/msdeploy.axd,authType=Basic,userName=$username,password='$password'"

    Write-Host "Deploying AuditService to $($webServerName)"

    $iisName = "IIS Web Application Name"
    $deploySite = "Default Web Site\YOUR_SITE_NAME"

    $msdeploy = "C:\Program Files\IIS\Microsoft Web Deploy V3\msdeploy.exe"
    $contentPath = "C:\YOUR_PACKAGE_PATH"

    $msdeployArguments = [string[]]@(
        "-verb:sync",
        "-source:package=$contentPath\YOUR_PACKAGE_NAME.zip",
        "-dest:auto,computerName=$webServerName",
        "-allowUntrusted",
        "-setParam:name='IIS Web Application Name',value='$($deploySite)'")

    Start-Process $msdeploy -ArgumentList $msdeployArguments -NoNewWindow -Wait
}

# Deploy to each of the boxes
foreach ($box in $deploymentBoxes.GetEnumerator()) {
   &deploy $box.destination $box.username $box.password
}
```

# 12. Build! Enjoy! You're done!
If you've configured your Git hooks correctly, Jenkins will automatically
check out and build the project every time code is merged to the branch
identified in step 4.

This is the part where you add additional steps, like configuring [Apache
JMeter](http://jmeter.apache.org/) to run functional tests against your newly
deployed project.
