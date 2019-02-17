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
[static hosting][static-hosting]. It listed 3 options, App Engine, 
Firebase Hosting, and Netlify. As I had never used Netlify before, I decided to 
try it.

First, I forked the [PWA Starter Kit repo][starter-kit-repo]. Next, I signed up 
for [Netlify][netlify], and followed the [instructions][instructions].
Now we have a PWA deployed here: [elastic-khorana-7862f5.netlify.com][elastic-khorana]

As the next step, I needed to create the [Digital Asset Links][asset-links]. To do
that, I need my signing certificate fingerprint. After a little searching, I found
it in the Play Store console.

Ok, so back to Android Studio. I cloned [svgomg-twa][svgomg-twa] again, and 
edited `app/build.gradle`, and changed:
- `org.chromium.twa.svgomg` to `blog.frustraded.pwa.starterkit`.
- `svgomg.firebaseapp.com` to `elastic-khorana-7862f5.netlify.com` (4 places)
- `SVGOMG TWA` to `PWA StarterKit`

I built the bundle, then went to the Play Store console, and created a new 
application. After setting everything up like I did in 
[part 1][play-store-part-one], I clicked `App signing`, and there is the list
of certificate fingerprints.

I grabbed a copy of `assetlinks.json` from the [svgomg pwa][svgomg-pwa]

    [{
      "relation": ["delegate_permission/common.handle_all_urls"],
      "target": {
        "namespace": "android_app",
        "package_name": "org.chromium.twa.svgomg",
        "sha256_cert_fingerprints": ["82:04:C5:DB:19:A8:B9:8A:27:14:F0:3E:F5:23:2C:6B:B6:B9:63:10:F2:F9:CD:44:72:AA:C6:7E:09:E1:1C:47",
         "91:45:8F:34:E3:13:E4:58:1C:12:21:7A:FD:1E:BD:5C:BE:9B:DE:2C:1E:57:DC:0D:2B:0E:91:1D:A6:36:CA:E8"]}
    }]

I changed the `package_name` to `blog.frustraded.pwa.starterkit`, and copied 
the SHA256 fingerprint from the Play Store.

    [{
      "relation": ["delegate_permission/common.handle_all_urls"],
      "target": {
        "namespace": "android_app",
        "package_name": "blog.frustraded.pwa.starterkit",
        "sha256_cert_fingerprints": 
        ["C5:71:58:C5:F1:D0:3C:90:7B:CE:4C:8E:2B:51:90:5E:D3:D5:9D:84:61:6A:93:05:6F:29:64:2F:05:A1:52:DD"]
      }
    }]

The last trick was to get `polymer build` to copy the assetlinks.json file.
This turned out to be easy. First, I made a directory called `.well-known`
and saved assetlinks there. Then I edited `polymer.json`, and added
`".well-known/assetlinks.json"` to `extraDependencies`.

After I pushed my changes, Netlify built and published them. Then I went to 
the Play Store console, and rolled out the app. Then I waited. After about half
an hour, I got an opt-in link, and was able to join the internal test.
Just like before, I was able to install the app on my phone! Sweet!

Next, I plan to build a real PWA, and release it. Stay tuned!

[play-store-part-one]: /2019/02/09/pwa-play-store.html
[svgomg]: https://jakearchibald.github.io/svgomg/
[polymer-project]: https://www.polymer-project.org/
[pwa-starter-kit]: https://pwa-starter-kit.polymer-project.org/
[build-deploy]: https://pwa-starter-kit.polymer-project.org/building-and-deploying
[static-hosting]: https://pwa-starter-kit.polymer-project.org/building-and-deploying#static-hosting
[starter-kit-repo]: https://github.com/Polymer/pwa-starter-kit
[netlify]: https://app.netlify.com/signup
[instructions]: https://pwa-starter-kit.polymer-project.org/building-and-deploying#netlify
[elastic-khorana]: https://elastic-khorana-7862f5.netlify.com
[asset-links]: https://developers.google.com/digital-asset-links/v1/getting-started
[svgomg-twa]: https://github.com/GoogleChromeLabs/svgomg-twa/
[svgomg-pwa]: https://svgomg.firebaseapp.com/.well-known/assetlinks.json
