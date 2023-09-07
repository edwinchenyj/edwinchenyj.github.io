---
title:  "Deploy static threejs site on Github Pages"
author: Ed
last_modified_at: 2023-09-07
categories:
        - Web
tags:
        - ThreeJS
        - ChatGPT
toc: true
toc_sticky: true
excerpt: "I leveraged ChatGPT to guide me in deploying a static ThreeJS site on Github Pages effectively"
---

I began my learning journey with [ThreeJS Journey](https://threejs-journey.com/) and have deployed the project sites to [Github Pages under my account](https://edwinchenyj.github.io/tjs-journey-preview/).

The course recommends deployment via Vercel. However, since I'm already familiar with Github, it felt like a more straightforward choice for me.

Yet, when I adhered to the [vite official instructions](https://vitejs.dev/guide/static-deploy.html), the deployed site appeared entirely black.
![png](/assets/images/three-js-deploy-black.png)

After attempting to debug on my own for a while, I decided to seek help from ChatGPT.
![png](/assets/images/learning-deployment-chatgpt.png)

Even though the answers didn't fix the issues I faced, I learned from Answer 1. ChatGPT that I need to fixed the asset paths. So I did exactly that, but not in index.html. 
There are a few line in `script.js` loading the assests, so I added the new base location to the Github Pages repo as indicated from the [vite official instructions](https://vitejs.dev/guide/static-deploy.html) in the `script.js` as well.

While the responses didn't directly resolve my issues, Answer 1 from ChatGPT made me realize that I needed to fix the asset paths. I proceeded to do exactly that. Instead of adjusting it in `index.html`, I made changes in `script.js`, which was responsible for loading the assets. I incorporated the new base location for the Github Pages repo, as suggested by the [vite official instructions](https://vitejs.dev/guide/static-deploy.html) into script.js.

*voilà*

![png](/assets/images/three-js-deploy-success.png)