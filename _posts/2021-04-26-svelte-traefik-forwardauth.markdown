---
layout: post
title: Implementing ForwardAuth for Traefik using SvelteKit
comments: true
---

In my [previous post][blue_green], I said I would build a simple app that shows you which service was selected.
While working on it, I wanted to hide it behid a login screen. Rather than build an entire login process, I
decided to leverage the options [Traefik][traefik] provided me and start with [BasicAuth][basicauth]. In this
example, `whoami` is substituting for the blue-green status app.

`docker-compose.yml`:

    proxy:
      image: traefik:v2.4
      command: --api.insecure=true --providers.docker
      ports:
        - "80:80"
        - "8080:8080"
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock
      labels:
        - "traefik.http.middlewares.auth.basicauth.users=test:$$apr1$$H6uskkkW$$IgXLP6ewTrSuBkTrqE8wj/,test2:$$apr1$$d9hr9HBB$$4HxwgUir3HP4EsggP/QNo0"
        - "traefik.http.middlewares.auth.basicauth.realm=Frustrated.blog"

      whoami:
        image: traefik/whoami
        labels:
          - "traefik.http.routers.whoami.rule=Host(`whoami.docker.localhost`)"
          - "traefik.http.routers.whoami.middlewares=auth@docker"

Spin up the services: `docker-compose up`, and you can visit the [Traefik dasbhboard][dashboard] and
the protected service, [whoami.docker.localhost][whoami]. When you do, you'll see something like this
(thuse username and password are both `test`):

![Chrome Sign in Dialog](/assets/images/signin.png)

Great start! But I wanted soemething less intrusive, and more flexable. A little
further down in the [docs][documentation], I found [ForwardAuth][forwardauth].

I've been playing with [Svlete][svelte], and [Svelte Kit][sveltekit] recently entered
public beta, so I figured this would be a great time to build something!

Starting in the same directory as the docker-compose.yml file, I ran
`npm init svelte@next`. I chose `template`, and then ran `npm init`.

Run `npm run dev` and visit the site, and you should see something like:

![Welcome to SvelteKit](/assets/images/sveltekit.png)

Note: If you are running in WSL, like me, you may need to run `npm run dev -- --host`.

Update `docker-compose.yml` and add the following service:

    auth:
      image: node:14
      ports:
        - 3000:3000
        - 24678:24678
      command: npm run dev -- --host
      working_dir: /usr/src/app
      volumes:
        - ./:/usr/src/app
      labels:
        - "traefik.http.routers.auth.rule=Host(`auth.docker.localhost`)"

You can now visit the Svelte tmplate at [auth.docker.localhost][auth].
Port `24678` allows live reload, so your changes are reflected instantly to the site!

According the the docs for [ForwardAuth][forwardauth], the middleware does a `get` to
our endpoint, and expects either a `2xx` for success, or anything else for failure. Adding
an endpoint to SvelteKit is [easy][endpoint]. Just add a file, and implement a `get` method.

`src/routes/auth.js`:

    export async function get() {
      return {
        status: 200,
        body: 'OK'
      };
    }

Update `docker-compose.yml`, removing the labels from `proxy`, adding this label to auth:

    - "traefik.http.middlewares.auth.forwardauth.address=http://auth:3000/auth"

Now, when you open [whoami.docker.localhost][whoami], the page should show, and any logging
you added to `auth.js` should show as well. (`docker-compose logs -f auth`)

If we make a small change, and return `status: 401` instead, we will be denied access. Awesome!

Alright, to turn this into something useful, we need a couple things. We need a Login page, and a way to
record the login status. SveletKit passes a `context` as part of the parameters object to the `get` method.
The context is created inside a `getContext` method in the [hooks][hooks] file. We are going to use that
method to look for a session id cookie, and retrieve or create the context. We will then use the companion
`handle` method to write the cookie to the browser.

`src/hooks.js`:

    import * as cookie from 'cookie';

    const SESSION_COOKIE = 'sid';
    const contexts = {};

    export async function getContext({ headers }) {
      const cookies = cookie.parse(headers.cookie || '');

      let sid = cookies[SESSION_COOKIE];
      let context = contexts[sid];

      if (!context) {
        const sid = Date.now(); // Don't do this in a real site!!
        context = contexts[sid] = { sid };
      }

      return context;
    }

    export async function handle( { request: { context, ...request }, render }) {
      const { headers, ...response } = await render({ ...request, context });

      if (context.sid) {
        headers['set-cookie'] = [cookie.serialize(SESSION_COOKIE, context.sid, {
          httpOnly: true, domain: 'docker.localhost', path: '/'
        })];
      }

      return { ...response, headers };
    }

Now, in a real site, we would need to worry about expireing the session, generating a
more secure session id, checking ip addresses, etc, but this will do for now.

Ok, lets add a Login page. Here is one that I copied from [CodeShack][codeshack].

`src/routes/login.svelte`:

    <style>
      .login-form {
        width: 300px;
        margin: 0 auto;
        font-family: Tahoma, Geneva, sans-serif;
      }
      .login-form h1 {
        text-align: center;
        color: #4d4d4d;
        font-size: 24px;
        padding: 20px 0 20px 0;
      }
      .login-form input[type="password"],
      .login-form input[type="text"] {
        width: 100%;
        padding: 15px;
        border: 1px solid #dddddd;
        margin-bottom: 15px;
        box-sizing:border-box;
      }
      .login-form input[type="submit"] {
        width: 100%;
        padding: 15px;
        background-color: #535b63;
        border: 0;
        box-sizing: border-box;
        cursor: pointer;
        font-weight: bold;
        color: #ffffff;
      }
    </style>

    <head>
      <meta charset="utf-8">
      <title>Login Form</title>
    </head>

    <div class="login-form">
      <h1>Login Form</h1>
      <form method="POST">
        <input type="text" name="username" placeholder="Username" required>
        <input type="password" name="password" placeholder="Password" required>
        <input type="submit">
      </form>
    </div>

The form will `POST` to `/login` when you press enter, so:

`src/routes/login.js`:

    export const post = async ({ body, context }) => {
      const username = body.get('username');
      const password = body.get('password');

      // High Security!!
      if (username === 'test' && password === 'test') {
        context.authenticated = true;

        return {
          status: 200,
          body: 'OK'
        };
      }

      return {
        status: 401,
        body: 'Invalid Username/Password'
      }
    }

Upon succesful login, we set the `authenticated` boolean in the context. Lets update the `auth`
endpoint to check that. If it's not set, redirect the user to the login page.

`/src/routes/auth.js`:

    export async function get({ context }) {
      if (context.authenticated === true) {
        return {
          status: 200,
          body: 'OK'
        };
      }

      return {
        status: 302,
        headers: { location: 'http://auth.docker.localhost/login' },
        body: 'OK'
      };
    }

Ok, open a browser window, and vist [whoami.docker.localhost][whoami]. You should get
redirected to the login page. Enter the super secret username and password, and reload
[whoami.docker.localhost][whoami]. This time it should work!

Pretty sweet, but we can make it just a little bit better! If we pass the original destination,
either through the session, or as part of the URL, login can redirect the user back to the
requested destination. This is passed to the auth endpoint as a set of headers.
Here is the updated auth endpoint.

`/src/routes/auth.js`:

    export async function get({ headers, context }) {
      const forwardedHost = headers['x-forwarded-host'];
      const forwardedUri = headers['x-forwarded-uri'];

      if (context.authenticated === true) {
        return {
          status: 200,
          body: 'OK'
        };
      }

      return {
        status: 302,
        headers: { location: `http://auth.docker.localhost/login?dest=${forwardedHost}${forwardedUri}` },
        body: 'OK'
      };
    }

The login POST handler needs to be updated to get the destination, and redirect to it.

`src/routes/login.js`:

    export const post = async ({ body, query, context }) => {
      const username = body.get('username');
      const password = body.get('password');

      const dest = query.get('dest');

      // High Security!!
      if (username === 'test' && password === 'test') {
        context.authenticated = true;

        return {
          status: 302,
          headers: {
            location: `http://${dest}`
          },
          body: 'OK'
        };
      }

      return {
        status: 401,
        body: 'Invalid Username/Password'
      }
    }

Give it a try!

One final note, if you want to protect access to the Traefik Dashboard this way,
you need to cover both the dashboard and api routes. Here are the labels you need:

    labels:
      - "traefik.http.routers.dashboard.rule=Host(`traefik.docker.localhost`) && PathPrefix(`/`)"
      - "traefik.http.routers.dashboard.middlewares=auth@docker"
      - "traefik.http.routers.dashboard.service=dashboard@internal"
      - "traefik.http.routers.api.rule=Host(`traefik.docker.localhost`) && PathPrefix(`/api`)"
      - "traefik.http.routers.api.middlewares=auth@docker"
      - "traefik.http.routers.api.service=api@internal"

The final version of the code used here can be found at github:
[https://github.com/abendigo/forwardAuth][github]


[blue_green]: /2021/03/16/traefik_blue_green.html
[traefik]: https://traefik.io/traefik/
[basicauth]: https://doc.traefik.io/traefik/v2.4/middlewares/basicauth/
[forwardauth]: https://doc.traefik.io/traefik/v2.4/middlewares/forwardauth/
[dashboard]: http://localhost:8080
[whoami]: http://whoami.docker.localhost
[documentation]: https://doc.traefik.io/traefik/v2.4/
[svelte]: https://svelte.dev/
[sveltekit]: https://kit.svelte.dev/
[auth]: http://auth.docker.localhost
[endpoint]: https://kit.svelte.dev/docs#routing-endpoints
[hooks]: https://kit.svelte.dev/docs#hooks
[codeshack]: https://codeshack.io/basic-login-system-nodejs-express-mysql/
[github]: https://github.com/abendigo/forwardAuth
