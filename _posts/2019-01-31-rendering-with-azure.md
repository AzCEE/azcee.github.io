---
layout: post
title:  "3D rendering using Microsoft Azure"
author: sergeype
tags: [ 3dsmax, vray, rendering, azure, batch, lowpriority ]
image: assets/users/sergeyperus/robot.png
featured: true

---
Imagine - you've created a 3d model of your object (interior, mechanism, construction plant etc.) using 3DMax for example. You can render it on you server, it tooks about 30 minutes in FHD resolution

Model and image can appear like this:
![image] (assets/users/sergeyperus/model-and-frame.png)

Now you need to create a short 5 minute movie. Depends on complexity of the model, desired quality and size of the outcome picture, it can take hours and days to render just one picture on a creator’s work server
With 25 fps, you have to render 25x60x5 = 7500 frames. It will take you 7500x30 = 156 `days`.
Not an option to do it on your own, right? And even you can double your equimpent - 2.5 month still far from acceptable

Of course, modern market is full of rendering service providers who can help you. 
This post is about how to build you own render farm, and in most cases it will cost you 30-70% less and many times faster


## How do create you own rendering farm

Of ccourse, i will record a video with all process. But now you can read it below

First of all, you have to have an Azure subscription. You can register trial with $200 credit here [https://azure.microsoft.com/en-us/offers/ms-azr-0044p/] or use your existing subscription
I aditionall to trial subscription, 

## Writing code blocks

There are two types of code elements which can be inserted in Markdown, the first is inline, and the other is block. Inline code is formatted by wrapping any word or words in back-ticks, `like this`. Larger snippets of code can be displayed across multiple lines using triple back ticks:

```
.my-link {
    text-decoration: underline;
}
```

If you want to get really fancy, you can even add syntax highlighting using Rouge.


![walking]({{ site.baseurl }}assets/users/sergeyperus/model-and-frame.png)

## Reference lists

The quick brown jumped over the lazy.

Another way to insert links in markdown is using reference lists. You might want to use this style of linking to cite reference material in a Wikipedia-style. All of the links are listed at the end of the document, so you can maintain full separation between content and its source or reference.

## Full HTML

Perhaps the best part of Markdown is that you're never limited to just Markdown. You can write HTML directly in the Markdown editor and it will just work as HTML usually does. No limits! Here's a standard YouTube embed code as an example:

<p><iframe style="width:100%;" height="315" src="https://www.youtube.com/embed/Cniqsc9QfDo?rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe></p>