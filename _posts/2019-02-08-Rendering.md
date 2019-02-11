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
In our case, rendering 1 frame takes 30 min

Model and image can appear like this:

![Model and render](/assets/users/sergeyperus/model-and-frame.png)

Now you need to create a short 5 minute movie. With 25 fps, you have to render 25x60x5 = 7500 frames. It will take you 7500x30 = 156 `days`.
Not an option to do it on your own, right? And even you can double your equimpent - 2.5 month still far from acceptable

Of cource, modern market is full of rendering service providers who can help you. 
This post is how to build you own render farm and why this is a good idea

## Why 

With Azure, you can engage thousands CPU cores with per-minute billing, automate rendering and control process by your own.
You can easily submit a few frames to estimate total cost and time. You even can use your own software and licenses, or rent it on a per-core per-minute rate (for 3DMax and V-Ray for example).
And, of course, you don't have to pay to 3rd party doing this job. So, let's rock!

## How

First of all, you need an Azure subscription. You can get free trial with $200 [here](https://azure.microsoft.com/en-us/offers/ms-azr-0044p/). In order to rent 3rd party licenses (like Autodesk or Chaos V-Ray), you can use your card, because you cannot pay for it with Microsoft credit.
And of course you can use your existing Azure subscription, many companies have it already - just ask your IT for it.
I will explain when and how much you have to pay

So, you have a subscription. Now you need to login to [Azure portal](https://portal.azure.com).
Here you can see your home screen or dashboard. On the left you can find you favourite service groups (2) and you can customize it with All services (1). 

![Azure portal](/assets/users/sergeyperus/AzPortal1.png)

Let's get Batch service to front and create Batch account

![Batch account](/assets/users/sergeyperus/Azportal2.png)

Here we should create:

 - *Resource group*. It will be used as a logical collection of assets for easy monitoring, access control, and for more effective management of their costs. Name it, for example, *Rendering*
 - *Storage account*. Choose Account kind as **StorageV2**. We will use it for storing our scene file/textures, as well as results.
 - *Location*. You can choose any region. But according to [Azure Price Calculator](https://azure.microsoft.com/en-us/pricing/calculator/#batch5c16533f-472c-43f3-b52d-3413fc585f11), our desired VM **Low Priority F32s V2** will cost less in EastUS datacenter

 ![Batch account creation](/assets/users/sergeyperus/batchaccount.png)

 Here i will stay for a while to describe price of resources and estimate cost of potential project

 Estimates for all and every Azure service are given on [Azure Calculator](https://azure.microsoft.com/en-us/pricing/calculator/). Actual prices may vary depending upon the date of purchase, currency of payment, and type of agreement you enter with Microsoft. 
 You should contact a Microsoft sales representative for additional information on pricing. I will use it for cost estimation.
 Almost every Azure resource billed by minute, and VM is not an exclusion. 

 Suppose, to render all frames of our scene, it will take 5 hours of operation of 50 virtual machines with 32 cores each, for a total of 1600 CPU cores.   
 In addition, we will rent 3rd party licenses (Autodesk® 3ds Max and Chaos Group V-Ray). Of course, you can use your own licenses and build your custom image with it among with all desired plugins.But I will simple scenario in this post - with license rent
 According to this information, the cost of compute resources can be about $505:  

	- $260.5 for 5 hours of work 50 x 32 vCPU VMs with WIndows Server license;  
	- $45 for 3ds Max licenses for this VMs;  
	- $200 for V-Ray licenses for this VMs;  

 ![Azure Low Priority Price](/assets/users/sergeyperus/AzlowpriorityUSD.png)