---
layout: post
title: Progressive Web App - Step One
categories: pwa serviceworker
---
One of my long-term plans for this blog is to turn it into a [Progressive Web App][pwa], and
I hope to document the journey as I go.

For my first step, I have chosen to add a Web App Install Banner, as documented [here][banner].

According to the article, I need 3 things:

- My site needs to be served over HTTPS
- A Service Worker, and
- A Web App Manifest file.

Fortunately, the first item is easy. HTTPS is the new default for [Github Pages][github].

The service worker doesn't need to do anything (yet), so I will borrow a little code from
this [great tutorial][tutorial]:

##### index.js

    if (navigator.serviceWorker) {
      navigator.serviceWorker.register('/src/sw.js').then(function(registration) {
        console.log('ServiceWorker registration successful with scope:',  registration.scope);
      }).catch(function(error) {
        console.log('ServiceWorker registration failed:', errror);
      });
    }

##### sw.js

    console.log('I am a Service Worker!');

Finally, I need a [Web App Manifest][manifest] file.

    {
      "short_name": "Occasionally Frustrated",
      "icons": [

      ],
      "start_url": "/"
    }

[pwa]: https://developers.google.com/web/progressive-web-apps/
[banner]: https://developers.google.com/web/updates/2015/03/increasing-engagement-with-app-install-banners-in-chrome-for-android
[github]: https://github.com/blog/2186-https-for-github-pages/
[tutorial]: http://www.html5rocks.com/en/tutorials/service-worker/introduction/
[manifest]: https://developers.google.com/web/updates/2014/11/Support-for-installable-web-apps-with-webapp-manifest-in-chrome-38-for-Android
