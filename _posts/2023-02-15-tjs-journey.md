---
title:  "Deploy static threejs site on Github Pages"
author: Ed
last_modified_at: 2023-02-15
categories:
        - Web
tags:
        - ThreeJS
        - ChatGPT
toc: true
toc_sticky: true
excerpt: "I used ChatGPT to help me learn how to deploy a static ThreeJS site correctly on Github Pages"
---

I started learning from [ThreeJS Journey](https://threejs-journey.com/) and I am deploying the project sites to another [Github Pages under my account](https://edwinchenyj.github.io/tjs-journey-preview/).

The course actually teaches how to deploy to Vercel. However, since I am already using Github, this is an more straight forward option for me.

However, when I followed the [vite official instructions](https://vitejs.dev/guide/static-deploy.html), the deployed site was all black.
![png](/assets/images/three-js-deploy-black.png)

After trying to debug by myself for a little while, I realized I should ask ChatGPT.
![png](/assets/images/learning-deployment-chatgpt.png)

Even though the answers didn't fix the issues I faced, I learned from Answer 1. ChatGPT that I need to fixed the asset paths. So I did exactly that, but not in index.html. 
There are a few line in `script.js` loading the assests, so I added the new base location to the Github Pages repo as indicated from the [vite official instructions](https://vitejs.dev/guide/static-deploy.html) in the `script.js` as well.

*vio la*

![png](/assets/images/three-js-deploy-success.png)