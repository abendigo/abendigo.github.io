---
layout: post
title: PWAs and the Google Play Store (part 2)
comments: true
---

In [part 1][play-store-part-one], I took the [SVGOMG pwa][svgomg], and released it 
to the Play Store. Exciting, but not very useful. In this post, I am going to 
take a different PWA, and release it.

I chose the [Polymer Project][polymer-project] [PWA Starter Kit][pwa-starter-kit].
I also needed a place to deploy it. The starter kit docs had a page titled
[Building and deploying][build-deploy], and a section for 
[static hosting][static-hosting]. It listed 3 opotions, App Engine, 
Firebase Hosting, and Netlify. As I had never used Netlify before, I decided to 
try it.

First, I forked the [PWA Starter Kit repo][starter-kit-repo]. Next, I signed up 
for [Netlify][netlify], and followed the [instructions][instructions].
Now we have a PWA deployed here: [elastic-khorana-7862f5.netlify.com][elastic-khorana]


[play-store-part-one]: /2019/02/09/pwa-play-store.html
[svgomg]: https://jakearchibald.github.io/svgomg/
[polymer-project]: https://www.polymer-project.org/
[pwa-starter-kit]: https://pwa-starter-kit.polymer-project.org/
[build-deploy]: https://pwa-starter-kit.polymer-project.org/building-and-deploying
[static-hosting]: https://pwa-starter-kit.polymer-project.org/building-and-deploying#static-hosting
[pwa-starter-repo]: https://github.com/Polymer/pwa-starter-kit
[netlify]: https://app.netlify.com/signup
[instructions]: https://pwa-starter-kit.polymer-project.org/building-and-deploying#netlify
[elastic-khorana]: https://elastic-khorana-7862f5.netlify.com
