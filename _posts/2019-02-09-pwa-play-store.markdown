---
layout: post
title: PWAs and the Google Play Store (part 1)
comments: true
---

Recently, I read an article called [Google Play Store now open for Progressive Web Apps][pwa-play-store]. This got me
very excited, as the number one argument I hear against Web Apps is "People look for apps in the App Store".
I decided to see how easy it is. I am writing this entry as I go through the exercise, so hopefully I'll capture
all the steps, pains, etc.

To keep things simple, I decided to start with the demo app mentioned in the article, 
[Google Chrome Labs' svgomg-twa][svgomg-twa].

First, I needed [Android Studio][studio]. After downloading and installing it, I used `git clone` to download the 
svgomg-twa repo. Then, I imported it into Android Studio, and built it. This took a little time, as
gradle, and some other API's needed to be downloaded.

Next, I tried to run the built app in the emulator. Unfortunately, that didn't go as smoothly as I 
hoped. 

    Emulator: ERROR: x86 emulation currently requires hardware acceleration!
    Please ensure Intel HAXM is properly installed and usable.
    CPU acceleration status: HAX kernel module is not installed!

A quick google, and a helpful [StackOverflow][stackoverflow] article later, and I had an Android
VM running. I tried running the app again, and still, no luck. I checked the version of Chrome 
in the VM, and it was 68! But, the article above mentioned the TWA feature was only available in
Chrome 72+. Unfortunately, the VM I had did not include the Play Store, so I was unable to upgrade
Chrome. I created a new VM, selecting an image that included the Play Store, botted it up, 
logged into the Play Store (why did I make my password so hard to type!), and updated Chrome.

And finally, I was able to run the app! Sweet!

Ugh. That was painful. I remember now why I gave up Android and Java development in favour of the web!
Of course, if you do Android development regularly, you will have already resolved these issues.
If you follow my blog, you know I use docker to help write these posts.
Unfortunatey, HAXM and HyperV don't work together, which caused extra
headaches. I recently saw an
article on running Android VM's with HyperV, which I'll explore later.

Now, to publish this thing! First, I needed to create a 
[Play Store developer publisher account][playstore-account]. $25 later, that was done.

In the Google Play Console, I clicked `Create Application`. I answered a few questions, (title, description,
etc.) and clicked `Save Draft`. Then, in the left hand menu, I selected `App releases`, scrolled down to 
`Internal test track`, and clicked `Manage`. In there, I clicked `Create Release`. I decided to let
Google Play create and manage my signing keys, so I clicked `Continue`. 

Now, I needed a bundle to upload, so I returned to Android Studio, clicked `build`, then 
`Generate Signed Bundle/APK`. In the pop-up, I left APK
unchecked, and clicked `next`. It prompted me to generate a signing key, so I filled in the blanks, 
and clicked `next`. Then I chose `release`, and clicked `Finsih`.

Back to the Play Store console, under `Android App Bundles and APKs to Add`, I clicked `BROWSE FILES`, 
and chose the bundle I just created. (it was under `app/release/app.aab`).

Oh no, an error: 

    Upload failed
    You need to use a different package name because "org.chromium" is restricted.

I grepped the source files, and made a change in `app/build.gradle`. I changed `org.chromium` to
`blog.frustrated`, and tried again.

I rebuilt the bundle, uploaded it, and it was accepted! I updated the release notes, and was able to save.
I clicked `Review`, and got a warning that I had not configured any users to be able to access it.
I added myself to the list of users, and was finally able to click `Review`.

The Rollout option didn't show up. After a little searching, I realized I needed to fill out the `Store listing`,
`Content Rating` and `Pricing & distribution` sections.

All in all, I needed to find 4 images (2 screenshots, 1 icon, and one 
feature graphic). I just grabed them from Google, as
no one other than myself will ever see them.

I returned to Internal Test, clicked `review`, and was finally presented 
with `START ROLLOUT TO INTERNAL TEST`.

And then I waited... I checked back in an hour or so, and the Opt-in URL
was available! I loaded the URL, joined the test program, and clicked 
`download it on Google Play`. 

Sweet! It's a Google Play store page! I clicked `install`, choose my 
device, and installed the app! I grabbed my phone, and a minute or so 
later, the icon appeared on my home screen! I clciked it, and the app 
launched!

Sorry for all the exclamation marks, but this is pretty exciting for me!
We published a PWA, in the Play Store! Of course, it's not our PWA, but
we will take care of that in Part 2!

[pwa-play-store]: https://medium.com/@firt/google-play-store-now-open-for-progressive-web-apps-ec6f3c6ff3cc
[svgomg-twa]: https://github.com/GoogleChromeLabs/svgomg-twa/
[studio]: https://developer.android.com/studio/
[stackoverflow]: https://stackoverflow.com/questions/29136173/emulator-error-x86-emulation-currently-requires-hardware-acceleration
[playstore-account]: https://developer.android.com/distribute/console/
