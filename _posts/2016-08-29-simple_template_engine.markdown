---
layout: post
title: Simple Template Engine
comments: true
---
I was looking for a small templating library for a project I was doing for a course at [Udacity][udacity].
There are a lot of libraries available, but I couldn't find any that took advantage of the newish
[HTML Template Element][template].  So, I decided to write some code myself. Please note, what you are
about to see below is a huge horrible hack, and is probably slow as heck. But, it was a lot of fun to play
with.

So, how does the `<template>` tag work? There is plenty of good documentation online, but here is a quick example:

    <div id="main"></div>

    <template id="template">
      <ul>
        <li>One</li>
        <li>Two</li>
      </ul>
    </template>

    <script>
      let template = document.querySelector('#template');
      let clone = document.importNode(template.content, true);
      document.getElementById('main').appendChild(clone);
    </script>

Paste this into a html file, and load it. You should see a list with two elements loaded.

Not a bad start, but I want simple one way data binding. But I don't really want to write a
parser, and loading an external library defeats the purpose of doing it myself. The new es6
[Template literal][literal] looks promising!

After a little searching on Google, this [question/answer][construct] on StackOverflow showed
me how to construct a Template literal from a string. Now I just needed to load that string from
the `<template>`.

    <template id="bound">
      <h2>${title}</h2>
    </template>

    <script>
      let template = document.querySelector('#bound');
      let f = new Function('title', 'return `' + template.innerHTML + '`;');
      console.log(f('hello templates'))
    </script>

Paste this into an html file, and load it. In your javascript console, you should see:

    <h2>hello templates</h2>

The next step is to get that HTML back into the DOM. Another search, another [question/answer][dom]
on StackOverflow, and we have:

    <div id="main"></div>

    <template id="bound">
      <h2>${title}</h2>
    </template>

    <script>
      let template = document.querySelector('#bound');
      let f = new Function('title', 'return `' + template.innerHTML + '`;');

      let div = document.createElement('div');
      div.innerHTML = f('hello templates');
      document.getElementById("main").appendChild(document.importNode(div.firstChild, true))
    </script>

Paste that into an html file, and load it. Nothing. WTF? Turns out, the white space in front of and
after the h2 tags get added as text children. Lets add a .trim() to the template.innerHTML line, above.

    let f = new Function('title', 'return `' + template.innerHTML.trim() + '`;');

Great, now it works!

Now what good is a templating library without repeaters? Fortunately template literals can handle that:

    <div id="main"></div>

    <template id="list">
      <ul>
        ${data.map(function(next) {
          return `<li>${next}</li>`;
        }).join('')}
      </ul>
    </template>

    <script>
      let template = document.querySelector('#list');
      let f = new Function('data', 'return `' + template.innerHTML.trim() + '`;');

      let div = document.createElement('div');
      div.innerHTML = f(['first', 'second', 'third']);
      document.getElementById("main").appendChild(document.importNode(div.firstChild, true))
    </script>

Paste that into an html file, and load it. You should see a list with three items.

Hmm, what else can we do? Well, the return in the above template bother me. Lets
replace `function` with an [Arrow function][arrow]:

    <template id="list">
      <ul>
        ${data.map(next =>
          `<li>${next}</li>`
        ).join('')}
      </ul>
    </template>

Well, that looks better, but it doesn't work. Turns out the innerHTML does some encoding,
converting >, &, and < to `&gt;`, `&amp;` and `&lt;` respectively. You should see this in
the javscript console:

    VM170:4 Uncaught SyntaxError: Unexpected token &

We can fix that in our function:

    let f = new Function('data', 'return `' + template.innerHTML.trim().replace("&gt;", ">") + '`;');

One last thing I want to do with this for now, and that is use one template inside another, like this:

    <template id="item">
      <li>${data}</li>
    </template>

    <template id="list">
      <ul>
        ${data.map(next =>
          template.item(next)
        ).join('')}
      </ul>
    </template>

The first thing we need to do is read all the templates, and create functions that we can
call:

    <script>
      let templates = {};
      for (let next of document.querySelectorAll('template')) {
        templates[next.id] = new Function("data", "template", "return `" + next.innerHTML.trim().replace("&gt;", ">") + "`;");
      }

      let div = document.createElement('div');
      div.innerHTML = templates.list(['first', 'second', 'third'], templates);
      document.getElementById("main").appendChild(document.importNode(div.firstChild, true))
    </script>

Notice that we are [creating a function][function] that accepts two parameters, `data` and `template`. `data` is the data
we want to feed into our templates. This can be a string, number, object, or array. `template` is the set of templates we just
read in.

And there we have it, a templating library that handles one way data binding, and repeaters, all in less than 10 lines of javascipt.
I want to make two final tweaks, eliminating the join in the template, and wrapping the appendChild code in a function:

    <div id="main"></div>

    <template id="item">
      <li>${data}</li>
    </template>

    <template id="list">
      <ul>
        ${mapData(data, next => template.item(next))}
      </ul>
    </template>

    <script>
      const mapData = (data, func) => data.map(func).join('');
      const appendTemplate = (target, template) => {
        let div = document.createElement('div');
        div.innerHTML = template;
        document.getElementById(target).appendChild(document.importNode(div.firstChild, true))
      }

      let templates = {};
      for (let next of document.querySelectorAll('template')) {
        templates[next.id] = new Function("data", "template", "return `" + next.innerHTML.trim().replace("&gt;", ">") + "`;");
      }

      appendTemplate('main', templates.list(['always', 'learning', 'occasionally', 'frustrated'], templates));
    </script>


[udacity]: https://www.udacity.com/course/senior-web-developer-nanodegree-by-google--nd802
[template]: https://developer.mozilla.org/en/docs/Web/HTML/Element/template
[literal]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Template_literals
[construct]: http://stackoverflow.com/questions/29771597/how-can-i-construct-a-template-string-from-a-regular-string/29771751#29771751
[dom]: http://stackoverflow.com/questions/16816726/converting-an-html-string-to-a-dom-element
[arrow]: https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions
[function]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function
