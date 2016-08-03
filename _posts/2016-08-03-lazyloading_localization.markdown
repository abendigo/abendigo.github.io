---
layout: post
title: Lazy Loading Localization data in Polymer
comments: true
---
We are currently working on a [Polymer][polymer] app, and my most recent task has been to add localization.
As the language files tend to get quite large, and we expect only a small percentage of users to change languages
after the app has started, we want to only load the appropriate data, and fetch more as required.

The Polymer Project has a section in their [Toolbox][toolbox] on [localization][localize] so that seems
like a good place to start. They even provide a nice [behavior][behavior] called
[Polymer.AppLocalizeBehavior][applocalizebehavior]. This might be easy!

Ok, lets use the [Polymer CLI][cli] to generate a quick app to experiment with:

    mkdir example && cd example
    polymer init application

Answer the questions, or accept the defaults. When it is has finished generating the app, and
installing the dependencies, serve it up:

    polymer serve

Open it in your browser, [here][localhost]. You should see something like this:

    Hello example-app

Next, we need to install the behavior as a dependency:

    bower install --save PolymerElements/app-localize-behavior

Open example-app.html, and import the behavior, and update the code, following the example in the
documentation for app-localize-behavior:

    {% raw %}
    <link rel="import" href="../../bower_components/polymer/polymer.html">
    <link rel="import" href="../../bower_components/app-localize-behavior/app-localize-behavior.html">

    <dom-module id="example-app">
      <template>
        <div>{{localize('hello', 'name', 'Batman')}}</div>
      </template>

      <script>
        Polymer({

          is: 'example-app',

          // include the behavior
          behaviors: [
            Polymer.AppLocalizeBehavior
          ],

          properties: {
            language: {
              value: 'en'
            },
          },

           // load localized messages
          attached: function() {
            this.loadResources(this.resolveUrl('locales.json'));
          }
        });
      </script>
    </dom-module>
    {% endraw %}

Now, we need to create the file locales.json in the same directory as example-app.html:

    {
      "en": { "hello": "My name is {name}." },
      "fr": { "hello": "Je m'apelle {name}." }
    }

Reload the app, and you should see:

    My name is Batman.

Change the value of language to 'fr', and reload the app. You should see:

    Je m'apelle Batman.

Awesome! Ok, to solve our requirement of only loading the data we need, lets break locales.json into
two files, en.json:

    {
      "en": { "hello": "My name is {name}." }
    }

and fr.json:

    {
      "fr": { "hello": "Je m'apelle {name}." }
    }

and lets change the attached method:

    attached: function() {
      this.loadResources(this.resolveUrl(this.language + '.json'));
    }

Reload the app. It should still work. Change the language, and reload again. It still works.
Thats awesome, we are now only loading the data we need. But, we can't change the language.

Lets add a little code to allow the user to switch languages. Add the following markup inside the template:

    <div id="languages">
      <span id="en">en</span>&nbsp<span id="fr">fr</span>
    </div>

And add the following ready function inside the script section:

    ready: function() {
      document.getElementById('languages').addEventListener('click', function(event) {
        this.language = event.target.id;
      }.bind(this));
    }

Reload the page, and click 'fr'. Hmm, that's weird, we got an error!

    app-localize-behavior.html:213 Uncaught TypeError: Cannot read property 'hello' of undefined

Turns out, there is what I consider to be a bug in the behavior. Great! A chance to submit a
[pull request][pullrequest], and give back! Until that is accepted, we can continue using my
[fork][fork]:

    bower install --save https://github.com/oanda/app-localize-behavior.git#1cadd14a7e3d1c7a3cb2a177018cde94a886a5b3

Now, reload the app, and click on 'fr'. The text disappears. That's because we haven't loaded fr.json yet.

Add an observer to the language property, like this:

    properties: {
      language: {
        value: 'en',
        observer: '_languageChanged'
      }
    }

Remove the attached function, and add this function:

    _languageChanged: function(newValue, oldValue) {
       this.loadResources(this.resolveUrl(newValue + '.json'));
     }

Reload the app, click 'fr'. It works! Click 'en'. It works too! We accomplished both the goals we set out above, and it wasn't that hard!

Here is the final version of example-app.html:

    {% raw %}
    <link rel="import" href="../../bower_components/polymer/polymer.html">
    <link rel="import" href="../../bower_components/app-localize-behavior/app-localize-behavior.html">

    <dom-module id="example-app">
      <template>
        <div id="languages">
          <span id="en">en</span>&nbsp<span id="fr">fr</span>
        </div>
        <div>{{localize('hello', 'name', 'Batman')}}</div>
      </template>

      <script>
        Polymer({

          is: 'example-app',

          // include the behavior
          behaviors: [
            Polymer.AppLocalizeBehavior
          ],

          properties: {
            language: {
              value: 'en',
              observer: '_languageChanged'
            },
          },

          ready: function() {
            document.getElementById('languages').addEventListener('click', function(event) {
              this.language = event.target.id;
            }.bind(this));
          },

          _languageChanged: function(newValue, oldValue) {
            this.loadResources(this.resolveUrl(newValue + '.json'));
          }
        });
      </script>
    </dom-module>
    {% endraw %}


[polymer]: https://www.polymer-project.org/1.0/
[toolbox]: https://www.polymer-project.org/1.0/toolbox/
[localize]: https://www.polymer-project.org/1.0/toolbox/localize
[behavior]: https://www.polymer-project.org/1.0/docs/devguide/behaviors
[applocalizebehavior]: https://elements.polymer-project.org/elements/app-localize-behavior
[cli]: https://www.polymer-project.org/1.0/tools/polymer-cli
[localhost]: http://localhost:8080
[pullrequest]: https://github.com/PolymerElements/app-localize-behavior/pull/42
[fork]: https://github.com/oanda/app-localize-behavior
