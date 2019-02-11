---
layout: post
title:  "3D rendering using Microsoft Azure"
author: sergeype
tags: [ 3dsmax, vray, rendering, azure, batch, lowpriority ]
image: assets/users/sergeyperus/pot.png
featured: true

---
	
Imagine - you've created a 3d model of your object (interior, mechanism, construction plant etc.) using 3DMax for example. After creating a model, you want to obtain a photorealistic image of it. This can be done by applying all textures, materials, light sources and camera position.
This image synthesis process is called rendering. Depends on complexity of the model, desired quality and size of the outcome picture, it can take hours and days to render just one picture on a creator’s work server.
In our example, rendering 1 frame takes 30 min

Model and image can appear like this:

![Model and render](/assets/users/sergeyperus/model-and-frame.png)

Now you need to create a short 5 minute movie. With 25 fps, you have to render 25x60x5 = 7500 frames. It will take you 7500x30 = 156 `days`.
Not an option to do it on your own, right? And even you can double your equimpent - 2.5 month still far from acceptable

Of cource, modern market is full of rendering service providers who can help you. 
This post is how to build you own render farm and why this is a good idea

## Why 

With Azure, you can engage thousands CPU cores with per-minute billing, automate rendering and control process by your own.
You can easily submit a few frames to estimate total cost and time. You even can use your own software and licenses, or rent it on a per-core per-minute rate (for 3DMax and V-Ray for example).
You can use two types of VM:  

**Dedicated**. This VMs will be available to you 100% time you need it
**Low Priority**. This VMs are not guaranteed. It is a surplus power, based on plans and demand, they can disapear/reapear at any time. And this VMs will cost you 2.5 times cheaper than **Dedicated**. When you need huge ammount of parallel vCPU, it is a perfect resource

And, of course, you don't have to pay to 3rd party provider doing this job. So, let's rock!

## How

This part will be divided on two - how to setup Azure part (probably you will need it once for all your rendering project) and Application part (you will use it every rendering project)

### Azure part 

First of all, you need an Azure subscription. You can get free trial with $200 [here](https://azure.microsoft.com/en-us/offers/ms-azr-0044p/). In order to rent 3rd party licenses (like Autodesk or Chaos V-Ray), you can use your card, because you cannot pay for it with Microsoft credit.
And of course you can use your existing Azure subscription, many companies have it already - just ask your IT for it.

After this part i will explain how much it will cost you


So, you have a subscription. Now you need to login to [Azure portal](https://portal.azure.com).
Here you can see your home screen or dashboard. On the left you can find you favourite service groups (2) and you can customize it with All services (1). 

![Azure portal](/assets/users/sergeyperus/AzPortal1.png)

Let's get Batch service to front and create Batch account

![Batch account](/assets/users/sergeyperus/Azportal2.png)

Here we should create:

 - *Resource group*. It will be used as a logical collection of assets for easy monitoring, access control, and for more effective management of their costs. Name it, for example, *Rendering*
 - *Storage account*. Select Account kind as **StorageV2**. We will use it for storing our scene file/textures, as well as results.
 - *Location*. You can choose any region. But according to [Azure Price Calculator](https://azure.microsoft.com/en-us/pricing/calculator/#batch5c16533f-472c-43f3-b52d-3413fc585f11), our desired VM **Low Priority F32s V2** will cost less in EastUS datacenter

 ![Batch account creation](/assets/users/sergeyperus/batchaccount.png)

 Leave othere parametrs by default and click **Create**  

Now its time to create support request. No, we did not break something already :). Issue is that by default everyone have a limit for available resources to prevent the use of huge resources unknowingly.
But we know what we are doing, right? Our ambitions are far below 100 default cores.  
Navigate to **Help + support** on the left. Click **New support request** and fill it:  

- *Issue type* - Service and subscription limits (quotas)
- *Subscription* - Select your subscription
- *Quota type* - Select **Batch**.  

Click Next.  

Click *Provide Details* and select location of your Batch account and **Low-priority cores** as *Resources*, as well as set *New Limit*  to **1600** cores. You can ask for more if you want, up to hundreds of thousands. Click *Save and continue* below    

Choose *Preferred contact method* and click *Next: Review + create >>*

![Service request](/assets/users/sergeyperus/AzSupport.png)  

In a few hours you can receive an email asked to provide **Batch Account Name** and **VM Type**. Reply with your Batch account name and VM type **FSv2** as per this example  

### Application part

Now it's time to render our first project. You can do it using simple native [Batch Explorer application](https://azure.github.io/BatchExplorer/). In this case you do not have to have 3ds Max installed, so you can render complex project even using tiny PC. 
Or you can install additional [plugin](https://github.com/Azure/azure-batch-rendering/tree/master/plugins/3ds-max) for 3ds Max application and submit you project in a native way. I will show the first option

After installing the Batch Explorer, you need to login to your Azure account and select your Batch account.  

### Creating compute pool

Navigate to *Pools* on the left, our pool list is empty. You can use Autopool for each rendering, but I prefer to have my own, with all plugins I need. Click **"+"** above and let's create out first pool  

- *ID* and *Display name* is basicaly a name of your pool
- *Scale* is a principle of pool scaling. In case of **Manual** mode you have to put number of VMs befor rendering and stop it after the process
Very important - `you have to stop VMs in pool after job, otherwise you will be billed for all running VMs in pool even job is finished!`  
To avoid extra charges we will use **autoscale function**. It will scale up VMs based on tasks ammount

		startingNumberOfVMs = 0;
		maxNumberofVMs = 50;
		pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(60 * TimeInterval_Second);
		pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(60 * TimeInterval_Second));
		$TargetLowPriorityNodes=min(maxNumberofVMs, pendingTaskSamples);
		//$TargetDedicatedNodes=min(maxNumberofVMs, pendingTaskSamples);  

In this formula you should change **maxNumberofVMs** parametr to desired number of VM. I will describe how to understand optimal number of VM later

## How much it will cost

Estimates for all and every Azure service are given on [Azure Calculator](https://azure.microsoft.com/en-us/pricing/calculator/). Actual prices may vary depending upon the date of purchase, currency of payment, and type of agreement you enter with Microsoft. 
 You should contact a Microsoft sales representative for additional information on pricing. I will use it for cost estimation.
 Almost every Azure resource billed by minute, and VM is not an exclusion. 

 Suppose, to render all frames of our scene, it will take 5 hours of operation of 50 virtual machines with 32 cores each, for a total of 1600 CPU cores.   
 In addition, we will rent 3rd party licenses (Autodesk® 3ds Max and Chaos Group V-Ray). Of course, you can use your own licenses and build your custom image with it among with all desired plugins. But I will use simple scenario in this post - with license rent  

 According to this information, the cost of compute resources can be about $505:  

	- $260.5 for 5 hours of working 50 x 32 vCPU VMs with WIndows Server license;  
	- $45 for 3ds Max licenses for this VMs;  
	- $200 for V-Ray licenses for this VMs;  

You can find this prices on mentioned [Azure Calculator](https://azure.microsoft.com/en-us/pricing/calculator/), try search "Batch"
 ![Azure Low Priority Price](/assets/users/sergeyperus/AzlowpriorityUSD.png)

In addition to compute resources, you will consume *Storage* and *Bandwidth*. Let's esimate it too:  

- Storage - our project (1GB) and rendered frames (20GB) will consume $0.43 per month. We need only 2 days, so about $0.03
- Bandwidth - you have to upload your project. Uploading to Azure is completly free. And to download results. It will cost you about $1.31


