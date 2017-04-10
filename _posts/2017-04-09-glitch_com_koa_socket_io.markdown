---
layout: post
title: Glitch.com, KOA, Socket.io
comments: true
---

Recently, [Fog Creek Software][fogcreek] announced thier beta version of [Glitch.com][glitch].
I was experimenting with a simple [socket.io][socket.io] based [nodejs][nodejs] server
to use as a signaling backend for some [Web RTC stuff][rtc]. I needed
a place to host it, and this seemed like a good chance to try out Glitch.

I went to the web site, clicked "Sign in", and linked my Github account. After looking 
around for a bit, I clicked "Edit Code", and it started a fresh project for me. There are
a lot of sample and starter apps, you'll figure it out pretty qucik.

My two favorite features are the auto restart of the server, and the auto reload of the web pages.
Although, the autosave can be a little disconcerting at first. The server crashes a lot
when you are in the middle of typing some code. 

The code that they give you to start with uses [Express][express], but, if you know me,
you know I never seem to do things the easy way. I wanted to try [Koa][koa]. So, in the 
Glitch editor, I clicked the "Logs" button, and opened the console. Then, I clicked the 
package.json file, and replaced express with koa in the dependecies section:

     "dependencies": {
       "koa": "^2.2.0"
     }

In the console section, I saw some errors as I typed, but once I finished editing the file,
I saw the packages get installed. Sweet!

I added [koa-static][static], middleware to server static assets, and
socket.io while I was here, ending up with: 

    "dependencies": {
      "koa": "^2.2.0",
      "koa-static": "^3.0.0",
      "socket.io": "^1.7.3"
    }

Next, I started to edit the server.js file. I replaced the current contents with:

    const Koa = require('koa');
    const staticServer = require('koa-static');

    const app = new Koa();

    app.use(staticServer('public'));

    const server = app.listen(process.env.PORT, function() {
        console.log('Koa is listening:', server.address());
    });

In the front-end section, I added public/index.html:

    <h3>
      Hello There!
    </h3>

Then I clicked the "Show Live" button, and got to see my greeting!

Ok, lets add socket.io support to the server. Turns out, thats dead simple. 
I added the following lines at the end of server.js:

    const sio = require('socket.io');
    const io = new sio(server);

    io.on('connection', socket => {
        console.log('connection', socket.id);
    });

Next, I created a simple Polymer app. public/index.html became:

    <!doctype html>
    <html>
      <head>
        <link rel="import" href="dirt-owner.html">

      </head>
      <body>
        <dirt-owner></dirt-owner>
      </body>
    </html>

I called the app dirt-owner, as that was the name of the instance Glitch gave me.
Next, I added public/dirt-owner.html:

    <link rel="import" href="https://polygit.org/polymer+2.0.0-rc.3/components/polymer/polymer-element.html">

    <dom-module id="dirt-owner">
      <template>
        <h3>
          Welcome, Dirt Owner!
        </h3>
      </template>
      <script>
        class DirtOwner extends Polymer.Element {
          static get is() {
            return "dirt-owner";
          }
        }
        
        customElements.define(DirtOwner.is, DirtOwner);
      </script>
    </dom-module>

When I finished editing the code, the page updated, and I was greeted again!
I am really starting to like Glitch.com!

Next, lets hook in socket.io. At the head of public/dir-owner.html, I added:

    <script src="/socket.io/socket.io.js"></script>
    
Then, In the Web Component connectedCallback, I initialized the socket.io connection:

    connectedCallback() {
        super.connectedCallback();

        this.socket = io();
    }

Almost immediatly, I see a connection being logged by the server! 

Ok, lets make this do something usefull. I will display a number of 
connections currently live, and the total number of connections since the server
restarted. This is what the final server.js file looks like:

    const Koa = require('koa');
    const staticServer = require('koa-static');

    const app = new Koa();

    app.use(staticServer('public'));

    const server = app.listen(process.env.PORT, function() {
        console.log('Koa is listening:', server.address());
    });

    const sio = require('socket.io');
    const io = new sio(server);

    let totalConnections = 0;
    let liveConnections = 0;

    io.on('connection', socket => {
        console.log(`new connection: ${socket.id}`);
    
        totalConnections += 1;
        liveConnections += 1;

        io.sockets.emit('marko', {total: totalConnections, live: liveConnections});

        socket.on('disconnect', reason => {
            console.log(`${socket.id} disconnected because: ${reason}`);
            liveConnections -= 1;
        
            io.sockets.emit('count', {total: totalConnections, live: liveConnections});
        });
    });


And this is the final public/dirt-owner.html:

    <script src="/socket.io/socket.io.js"></script>

    <link rel="import" href="https://polygit.org/polymer+2.0.0-rc.3/components/polymer/polymer-element.html">

    <dom-module id="dirt-owner">
      <template>
        <h3>
          Welcome, Dirt Owner!
        </h3>
        Live Connections: [[liveConnections]]<br />
        Total Connections: [[totalConnections]]<br />
      </template>
      <script>
        class DirtOwner extends Polymer.Element {
          static get is() {
            return "dirt-owner";
          }
        
          static get properties() {
            return {
              totalConnections: Number,
              liveConnections: Number
            };
          }

          connectedCallback() {
            super.connectedCallback();

            let socket = io();
            
            socket.on('count', data => {
              this.liveConnections = data.live;
              this.totalConnections = data.total;
            });
          }
        }
        
        customElements.define(DirtOwner.is, DirtOwner);
      </script>
    </dom-module>


As a final cleaneup, 
I deleted the following files that my app didnt use: public/client.js, public/style.css, and 
views/index.html.

To see it in it's final glory, you can visit it at [https://dirt-owner.glitch.me/][dirt].



[fogcreek]: http://www.fogcreek.com/
[glitch]: https://glitch.com/
[socket.io]: https://socket.io/
[nodejs]: https://nodejs.org/en/
[rtc]: https://abendigo.github.io/2017/01/09/rtc-web-components.html
[express]: https://expressjs.com/
[koa]: http://koajs.com/
[static]: https://github.com/koajs/static
[dirt]: https://dirt-owner.glitch.me/
