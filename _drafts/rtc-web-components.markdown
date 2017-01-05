---
layout: post
title: Polymer 2.0 and WebRTC
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

Note: I origonally intended this to be just one post, but as I started 
to write it, I realized it was going to be pretty large, so I have decided to break 
it into a small series of posts.

Caution: While the WebRTC API is pretty much
stable these days, it has not been finalized, and is subject to Change. As well,
Polymer 2.0 is a preview release, and will almost certainly change before release.
I did all my work in Chrome, so it's possible none of this will work in other browsers.

## step-02 (Chapter 5)

The new code consists of two files, index.html and step-02-app.html.

#### index.html

    <!DOCTYPE html>
    <html>
      <head>
        <title>Realtime communication with WebRTC</title>

The title was copied from the codelab's index.html file.

        <script src="bower_components/webcomponentsjs/webcomponents-lite.js"></script>
        <link href="step-02-app.html" rel="import">
      </head>
      <body>
        <step-02-app></step-02-app>
      </body>
    </html>

#### step-02-app.html

    <link rel="import" href="bower_components/polymer/polymer.html">

Standard for any Polymer app.

    <link rel="import" href="custom-elements/rtc-local-signaling-channel.html">

The Signaling Channel (more on this later)

    <link rel="import" href="bower_components/rtc-web-components/rtc-data-channel.html">
    <link rel="import" href="bower_components/rtc-web-components/rtc-media-stream.html">
    <link rel="import" href="bower_components/rtc-web-components/rtc-peer-connection.html">

The Web Components I wrote 

    <dom-module id="step-02-app">
      <template>
        <style>
          :host {
            font-family: sans-serif;
          }

          video {
            max-width: 100%;
            width: 320px;
          }
        </style>

This is just a straight import of the css/main.css file.

        <rtc-local-signaling-channel instance={{_signalingChannel}}></rtc-local-signaling-channel>

        <rtc-peer-connection id="one" peer="two" signaling=[[_signalingChannel]]>
            <rtc-media-stream in=[[_localStream]]></rtc-media-stream>
        </rtc-peer-connection>

        <rtc-peer-connection id="two" peer="one" signaling=[[_signalingChannel]]>
            <rtc-media-stream out={{_remoteStream}}></rtc-media-stream>
        </rtc-peer-connection>

The Magic stuff!

        <h1>Realtime communication with WebRTC</h1>

        <video id="localVideo" src-object=[[_localStream]] autoplay></video>
        <video id="remoteVideo" src-object=[[_remoteStream]] autoplay></video>

        <div>
        <button id="startButton" on-click="start">Start</button>
        <button id="callButton"  on-click="call" disabled>Call</button>
        <button id="hangupButton" on-click="hangup" disabled>Hang Up</button>
        </div>
    </template>

This section is almost verbatum copied from the Codelab, with the addition of
some Polymer bindings.














The code for this is available [here][step-02].



__step-03 (Chapter 6)__

The code for this is available [here][step-03].

__step-05 (Chapter 8)__

The code for this is available [here][step-05].

__step-06 (Chapter 9)__

The code for this is available [here][step-06].

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
[step-02]: https://github.com/abendigo/rtc-codelab/tree/master/step-02
[step-03]: https://github.com/abendigo/rtc-codelab/tree/master/step-03
[step-05]: https://github.com/abendigo/rtc-codelab/tree/master/step-05
[step-06]: https://github.com/abendigo/rtc-codelab/tree/master/step-06
