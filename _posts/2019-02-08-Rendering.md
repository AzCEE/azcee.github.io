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

First of all, you need an Azure subscription. You can get free trial with $200 here. In order to rent 3rd party licenses (like Autodesk or Chaos V-Ray), you can use your card, because you cannot pay for it with Microsoft credit.
And of course you can use your existing Azure subscription, many companies have it already - just ask your IT for it.
I will explain when and how much you have to pay

So, you have a subscription. Now you need to login to [Azure portal](https://portal.azure.com)
Here you can see your home screen or dashboard. On the left you can find you favourite service groups (2) and you can customize it with All services (1). 

![Azure portal](/assets/users/sergeyperus/AzPortal1.png)

Let's get Batch service to front and create it 

![Batch account](/assets/users/sergeyperus/Azportal2.png)

Now we need to create 

