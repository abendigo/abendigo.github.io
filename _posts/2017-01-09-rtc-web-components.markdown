---
layout: post
title: Polymer 2.0 and WebRTC - Introduction
comments: true
---

Over the Christmas holidays, there was [discusion on Hacker News][news]
about an article on [WebRTC and the future of web games][article].

This got me interested in exploring [WebRTC][webrtc] again, and in particular,
the [Data Channel API][rtcdatachannel].

I quickly found a [Google Codelab][rtccodelab], and after a half hour or so,
realized I really enjoyed what I was seeing, and started to think about
creating some [Web Components][webcomponents] using [Polymer][polymer].

Earlier in the week, I had completed another [Google Codelab][polymercodelab],
building an Image Carousel with [Polymer 2.0][polymer2], so naturally I decided to
use this version.

What I present here is a recreation of the code for step-02 [(Chapter 5)][chapter5],
step-03 [(Chapter 6)][chapter6], step-05 [(Chapter 8)][chapter8], and step-06 [(Chapter 9)][chapter9] of the
Google Codelab, using the Web Components I created.

>Note: I originally intended this to be just one post, but as I started
to write it, I realized it was going to be pretty large, so I have decided to break
it into a small series of posts.

<div style="background-color:rgba(190, 190, 0, 0.8); text-align:center; vertical-align: middle; padding:10px 10px;">
Caution: While the WebRTC API is pretty much
stable these days, it has not been finalized, and is subject to Change. As well,
Polymer 2.0 is a preview release, and will almost certainly change before release.
I did all my work in Chrome, so it's possible none of this will work in other browsers.
</div>


[news]: https://news.ycombinator.com/item?id=13264952
[article]: https://getkey.eu/blog/5862b0cf/webrtc:-the-future-of-web-games
[webrtc]: https://webrtc.org/
[rtcdatachannel]: https://developer.mozilla.org/en/docs/Web/API/RTCDataChannel
[rtccodelab]: https://codelabs.developers.google.com/codelabs/webrtc-web/
[chapter5]: https://codelabs.developers.google.com/codelabs/webrtc-web/#4
[chapter6]: https://codelabs.developers.google.com/codelabs/webrtc-web/#5
[chapter8]: https://codelabs.developers.google.com/codelabs/webrtc-web/#7
[chapter9]: https://codelabs.developers.google.com/codelabs/webrtc-web/#8
[webcomponents]: http://webcomponents.org/
[polymer]: https://www.polymer-project.org/
[polymercodelab]: https://codelabs.developers.google.com/codelabs/polymer-2-carousel/index.html#0
[polymer2]: https://www.polymer-project.org/2.0/docs/about_20
