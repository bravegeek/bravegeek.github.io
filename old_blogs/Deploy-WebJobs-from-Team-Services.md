---
title: Deploy WebJobs from Team Services
date: 2016-12-03 08:59:04
tags:
    - WebJobs
    - Team Services
    - VSO
---
![](/content/images/2016/vsts.webjob.png)

## This is gonna be easy
So you just finished writing your awesome new WebJob that's gonna save you hours of work.
You just need to get it up to Azure, and Mission Accomplished.

Simple, right click on the Project (in Visual Studio) and select `Publish as Azure WebJob`, follow the wizard.

Check Azure, yup it's there.

## Wait a sec, not so fast
Then your teammate reminds you that it needs to automatically deploy from your CI. (Thanks Jeremy, really.)
Piece of cake, you're using Visual Studio Team Services, just add another Release task in your Release Definition.

And that's where the wheels fall off your little red wagon. (I don't hate wagons, just Jeremy's stupid red one.)

It turns out that Team Services doesn't have a canned task for deploying WebJobs.

## Hey, what's going on
Let's understand how WebJobs work in Azure.
1. You need an App Service for your WebJob to live under
2. The WebJob needs to be deployed to a specific directory

Number 1 is pretty easy, you probably already have an App Service in Azure. 
(I'm actually running an ASP.net Core app, and deploying a .net 4.52 WebJob.)

Number 2 isn't so obvious, so you might want to [read more about WebJobs](https://github.com/projectkudu/kudu/wiki/Web-Jobs)

_TLDR_
To deploy a triggered job copy your binaries to: d:\home\site\wwwroot\app_data\jobs\triggered\\{job name}
To deploy a continuous job copy your binaries to: d:\home\site\wwwroot\app_data\jobs\continuous\\{job name}

__The trick is getting your WebJob binaries into the correct folder on Azure.__

Let me say that again:
__The trick is getting your WebJob binaries into the correct folder on Azure.__

## The Trick
There are two key steps to getting this trick to work:
1. Add a `Copy Files` step to your Build definition.
2. Add an `Azure App Service Deploy` task to your Release definition.

### Copy Files (Build definition)
The thing to understand here is that when files are copied to Azure, they retain their original folder structure.
We're going to use that fact to setup the correct folder structure after we build.

(I've named your WebJob `WebJobTest` and it's continuous, you're welcome Jeremy.)

* In your Build definition, add a `Copy Files` step after the build step.
* Set these properties
    * Source Folder: src/WebJobTest/bin/$(BuildConfiguration)/
    * Contents: **
    * Target Folder: $(build.artifactstagingdirectory)\WebJobTest\App_Data\jobs\continuous\WebJobTest

![](/content/images/2016/build.def.webjob.png)

### Azure App Service Deploy (Release definition)
Now that the WebJob files are in the correct folder structure, we can just deploy that folder to Azure.

* In your Release definition, add an `Azure App Service Deploy` task.

![](/content/images/2016/release.def.webjob.png)

* Set `Package or Folder` to $(System.DefaultWorkingDirectory)//{YourBuildDefinition}/drop/WebJobTest 

![](/content/images/2016/release.def.webjob.folder.select.png)

That will copy everything below `WebJobTest`, starting with `App_Data`, which is the folder structure we need for WebJobs.

## That's all folks
Now your WebJobs are deploying automatically and Jeremy is happy about his red wagon.
(That job is totally not made up or photo-shopped in any way.)

Since you can have multiple WebJobs under an App Service, just lather, rinse, repeat to add more. 

![](/content/images/2016/WebJobs.azure.png)
