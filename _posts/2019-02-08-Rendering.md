---
layout: post
title:  "3D rendering using Microsoft Azure"
author: sergeype
tags: [ 3dsmax, vray, rendering, azure, batch, lowpriority ]
image: assets/users/sergeyperus/robot.png
featured: true

---
	
Imagine - you've created a 3d model of your object (interior, mechanism, construction plant etc.) using 3DMax for example. After creating a model, you want to obtain a photorealistic image of it. This can be done by applying all textures, materials, light sources and camera position
This image synthesis process is called rendering. Depends on complexity of the model, desired quality and size of the outcome picture, it can take hours and days to render just one picture on a creator’s work server


Model and image can appear like this:

![Model and render](/assets/users/sergeyperus/robot_result.png)

Now you need to create a short 5 minute movie. With 25 fps, you have to render 25x60x5 = 7500 frames. It will take you 7500x30 = 156 `days`
Not an option to do it on your own, right? And even you can double your equimpent - 2.5 month still far from acceptable

Of cource, modern market is full of rendering service providers who can help you. 
This post is how to build you own render farm, and in most cases it will cost you 30-70% cheaper and many times faster

If you already have a full Ruby development environment with all headers and RubyGems installed (see Jekyll’s requirements), you can create a new Jekyll site by doing the following:







```ruby
# Install Jekyll and Bundler gems through RubyGems
gem install jekyll bundler

# Create a new Jekyll site at ./myblog
jekyll new myblog

# Change into your new directory
cd myblog

# Build the site on the preview server
bundle exec jekyll serve

# Now browse to http://localhost:4000
```