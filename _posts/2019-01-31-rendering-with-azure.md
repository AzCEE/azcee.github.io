---
layout: post
title:  "3D rendering using Microsoft Azure"
author: sergeype
tags: [ 3dsmax, vray, rendering, azure, batch, lowpriority ]
image: assets/users/sergeyperus/robot.png
featured: true
hidden: false
---
Imagine - you've created a 3d model of your object (interior, mechanism, construction plant etc.) using 3DMax for example. After creating a model, you want to obtain a photorealistic image of it. This can be done by applying all textures, materials, light sources and camera position
This image synthesis process is called rendering. Depends on complexity of the model, desired quality and size of the outcome picture, it can take hours and days to render just one picture on a creator’s work server


Model and image can appear like this:

![Model and render](/assets/users/sergeyperus/robot_result.png)

Now you need to create a short 5 minute movie. With 25 fps, you have to render 25x60x5 = 7500 frames. It will take you 7500x30 = 156 `days`
Not an option to do it on your own, right? And even you can double your equimpent - 2.5 month still far from acceptable

Of cource, modern market is full of rendering service providers who can help you. 
This post is how to build you own render farm, and in most cases it will cost you 30-70% cheaper and many times faster




As with the last post about the editor, you'll want to be actually editing this post as you read it so that you can see all the Markdown code we're using.


## Special formatting

As well as bold and italics, you can also use some other special formatting in Markdown when the need arises, for example:

+ ~~strike through~~
+ ==highlight==
+ \*escaped characters\*


## Writing code blocks

There are two types of code elements which can be inserted in Markdown, the first is inline, and the other is block. Inline code is formatted by wrapping any word or words in back-ticks, `like this`. Larger snippets of code can be displayed across multiple lines using triple back ticks:

```
.my-link {
    text-decoration: underline;
}
```

If you want to get really fancy, you can even add syntax highlighting using Rouge.

[//]: ![walking]({{ site.baseurl }}assets/users/sergeyperus/model-and-frame.png)

## Reference lists

The quick brown jumped over the lazy.

Another way to insert links in markdown is using reference lists. You might want to use this style of linking to cite reference material in a Wikipedia-style. All of the links are listed at the end of the document, so you can maintain full separation between content and its source or reference.

## Full HTML

Perhaps the best part of Markdown is that you're never limited to just Markdown. You can write HTML directly in the Markdown editor and it will just work as HTML usually does. No limits! Here's a standard YouTube embed code as an example:

<p><iframe style="width:100%;" height="315" src="https://www.youtube.com/embed/Cniqsc9QfDo?rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe></p>