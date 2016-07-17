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
      "name": "Occasionally Frustrated",
      "short_name": "Occasionally Frustrated",
      "icons": [
        {
          "src": "launcher-icon.png",
          "sizes": "144x144",
          "type": "image/png"
        }
      ],
      "start_url": "/"
    }

Ok, that looks like it should work. Lets get out an Android device, and disable the
engagement check ([chrome://flags/#bypass-app-banner-engagement-checks](chrome://flags/#bypass-app-banner-engagement-checks))

## WTF, it doesn't work!

If only there was an easier way to test and debug this. Well, fortunately, there is! [This page][criteria] lists the criteria.
and has a handy section titled "Testing the App Install Banner". Basically, you enable two flags in Chrome on your desktop:

- [chrome://flags/#bypass-app-banner-engagement-checks](chrome://flags/#bypass-app-banner-engagement-checks)
- [chrome://flags/#enable-add-to-shelf](chrome://flags/#enable-add-to-shelf)

Now, when I load the page, I see the following errors in my JavaScript console:

    App banner not shown: 192px square icon is required, but no supplied icon is at least this size
    App banner not shown: manifest display property must be set to 'standalone' or 'fullscreen'

Ok, lets update the manifest with a larger icon, and set display to "standalone".

    App banner not shown: no matching service worker detected. You may need to reload the page, or check that the service worker for the current page also controls the start URL from the manifest

Right, Service Workers have scope. We'll probably get more into that when we make the service worker do
some real work, but for now, lets move it back to the root folder.

## Awesome, it worked!

Here are the final versions of index.js

    if (navigator.serviceWorker) {
      navigator.serviceWorker.register('/sw.js').then(function(registration) {
        console.log('ServiceWorker registration successful with scope:',  registration.scope);
      }).catch(function(error) {
        console.log('ServiceWorker registration failed:', error);
      });
    }

 and manifest.json:

    {
      "name": "Occasionally Frustrated",
      "short_name": "Occasionally Frustrated",
      "icons": [
        {
          "src": "launcher-icon.png",
          "sizes": "192x192",
          "type": "image/png"
        }
      ],
      "start_url": "/",
      "display": "standalone"
    }



[pwa]: https://developers.google.com/web/progressive-web-apps/
[banner]: https://developers.google.com/web/updates/2015/03/increasing-engagement-with-app-install-banners-in-chrome-for-android
[github]: https://github.com/blog/2186-https-for-github-pages/
[tutorial]: http://www.html5rocks.com/en/tutorials/service-worker/introduction/
[manifest]: https://developers.google.com/web/updates/2014/11/Support-for-installable-web-apps-with-webapp-manifest-in-chrome-38-for-Android
[criteria]: https://developers.google.com/web/fundamentals/engage-and-retain/app-install-banners/web-app-install-banners?hl=en
