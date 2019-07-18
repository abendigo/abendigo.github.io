---
layout: post
title: Svelte, Sapper, GraphQL, and Subscriptions
comments: true
---

I recently started a new hobby project, and so I decided to take a quick
look at the technologies that were available.

[Svelte 3.0][svelte] was recently [released][released], and it looks pretty
interesting. It has support for Server Side Rendering (for the SEO magic)
via a package called [Sapper][sapper].

I've been wanting to explore [GraphQL][graphql], and this seemed like a 
good opportunity. Real time updates are important for this project, so
I wanted to try GraphQL subscriptions.

Great, technologies decided. Lets just google for a tutorial, or sample 
app, and get started. And I ran into my first roadblock. I found a
[tweet][tweet] from [Rich Harris][rich], the mastermind behind Svelte 
and Sapper, mentioning [express-graphql][express-graphql].
I found a [blog post][blog] from [Hasura][hasura] mentioning 
[svelte-apollo][svelte-apollo]. But nothing doing the exact combo that I
am looking for. 

Ok, lets get started. The final code will be available on 
[github here][github].

First, I followed the instructions on the [Sapper][sapper] website
to create the template. I choose [Rollup][rollup] for building and
bundling, but you can also use [Webpack][webpack].

    npx degit "sveltejs/sapper-template#rollup" sapper-graphql-subscriptions
    cd sapper-graphql-subscriptions
    npm install
    npm run dev

Next, I added the `express-graphql`, '`graphql`, and 
`graphql-tools` modules as a dev dependencies, so they
will get bundled at build time.

    npm i -D express-graphql graphql graphql-tools

Restarting the development server, I saw some build errors. I fixed these
by moving all three in to the `dependencies`
section of `package.json`. For Sapper, this means that they will be
imported at run time, not build time. I am tryng to find the documentation
that explains this, and will update with a link here when I find it.

Now, adding GraphQL support was as easy as adding a `graphql.js` file
under `routes/`.

    import graphqlHTTP from 'express-graphql';
    import { makeExecutableSchema } from 'graphql-tools';

    const typeDefs = `
      type Query {
        random: Float
      }
    `;

    const resolvers = {
      Query: {
        random: () => Math.random(),
      }
    }

    const schema = makeExecutableSchema({typeDefs, resolvers});

    export const post = graphqlHTTP({
      schema,
      pretty: true
    });

You can test this using curl:

    curl 'http://localhost:3000/graphql' \
      -H 'content-type: application/json' \
      --data '{"query":"{random}"}'

You should see a response like:

    {
      "data": {
        "random": 0.482803512333974
      }
    }

Next step, consuming this data. I decied to work on the `About` page.
First, I modified `about.svelte`, adding this to the top:

    <script>
      let random = 0;
    </script>

and this at the end:

    <div>
      Random: {random}
    </div>

Next, I installed a GraphQL client, [Apollo][apollo], via `apollo-boost`
and some utilites via `graphql-tag`:

    npm i -D apollo-boost graphql-tag

and imported the `onMount` method from Svelte, the `gpl` method from
`graphql-tag` and `ApolloClient` from `apollo-boost`:

    import ApolloClient from 'apollo-boost';
    import gql from 'graphql-tag';
    import { onMount } from 'svelte';

and added this:

    onMount(() => {
      const client = new ApolloClient();

      client.query({
        query: gql`{ random }`
      }).then(result => {
        random = result.data.random;
      });
    });

When you load [About][about] page, you should see something like this:

![Sapper About Page](/assets/images/about.png)

Almost there! All that was left was the subscriptions. For this, I needed to 
use web sockets. Starting with the backend, I modified `server.js`.
I needed to switch from `polka` to `express`, to have a little more control 
of startup. I also needed to use `SubscriptionServer` from 
`subscription-transport-ws`. First, I installed the dependencies:

    npm i -D express subscriptions-transport-ws

Unfortunatley, this caused a build error, so I moved them both from
`devDependencies` to `dependencies`.

Then, I changed the startup to use `express` and the `http` package:

    import express from 'express';
    import http from 'http';

    const app = express() // You can also use Express
      .use(
        compression({ threshold: 0 }),
        sirv('static', { dev }),
        sapper.middleware()
      );

    const server = http.createServer(app);

    server.listen(PORT, err => {
      if (err) console.log('error', err);
    });

in the listen callback, I created the `SubscriptionServer`:

    import { execute, subscribe } from 'graphql';
    import { SubscriptionServer } from 'subscriptions-transport-ws';

    new SubscriptionServer({execute, subscribe, schema},
      {
        server: server,
        path: '/subscriptions'
      });

But, we are missing one thing, the `schema`. I extracted it from 
`graphql.js` into `_schema.js`:

    import { makeExecutableSchema } from 'graphql-tools';

    const typeDefs = `
      type Query {
        random: Float
      }
    `;

    const resolvers = {
      Query: {
        random: () => Math.random(),
      }
    }

    export const schema = makeExecutableSchema({typeDefs, resolvers});

and imported it into `server.js':

    import { schema } from './_schema';

Finally, I defined a subscription in `_schema.js`:

    type Subscription {
      randoms: Float
    }

    Subscription: {
      randoms: {
        subscribe: async function* asyncRandomNumbers() {
          while (true) {
            yield { randoms: Math.random() };
            await sleep(1000);
          }
        }
      }
    }

Quick note. A subscription server side is implemented as an `AsyncIterator`.
I made a quick async generator function that sleeps for 1 second after
returning a random number.

Back to the `About` page. Client initialization is a little more 
complicated, so I moved it out to a helper file, after installing
more modules:

    npm i -D apollo-client apollo-cache-inmemory apollo-link \
      apollo-link-http apollo-link-ws apollo-utilities

`_client.js`:

    import { ApolloClient } from 'apollo-client';
    import { InMemoryCache } from 'apollo-cache-inmemory';
    import { split } from 'apollo-link';
    import { HttpLink } from 'apollo-link-http';
    import { WebSocketLink } from 'apollo-link-ws';
    import { getMainDefinition } from 'apollo-utilities';

    export const createApolloClient = () => {
      const httpLink = new HttpLink({
        uri: 'http://localhost:3000/graphql'
      });
      const wsLink = new WebSocketLink({
        uri: `ws://localhost:3000/subscriptions`,
        options: {
          reconnect: true
        }
      });

      const link = split(
        ({ query }) => {
          const definition = getMainDefinition(query);
          return (
            definition.kind === 'OperationDefinition' &&
            definition.operation === 'subscription'
          );
        },
        wsLink,
        httpLink
      );

      return new ApolloClient({link, cache: new InMemoryCache()});
    }

And, in `About.svelte`:

    let randoms = 0;
    ...
    client.subscribe({
      query: gql`subscription { randoms }`
    }).subscribe(result => {
     randoms = result.data.randoms;
    });
    ...
    Randoms: {randoms}

Load the [About][about] page, and you should see your results! 

[svelte]: https://svelte.dev/
[released]: https://svelte.dev/blog/svelte-3-rethinking-reactivity
[sapper]: https://sapper.svelte.dev/
[graphql]: https://graphql.org/
[tweet]: https://twitter.com/rich_harris/status/987132513943420928
[rich]: https://github.com/Rich-Harris
[express-graphql]: https://github.com/graphql/express-graphql
[blog]: https://blog.hasura.io/build-and-deploy-svelte-js-3-apps-using-graphql/
[hasura]: https://hasura.io/
[svelte-apollo]: https://github.com/timhall/svelte-apollo
[github]: https://github.com/abendigo/sapper-graphql-subscriptions
[rollup]: https://rollupjs.org
[webpack]: https://webpack.js.org/
[playground]: https://github.com/prisma/graphql-playground
[apollo]: https://github.com/apollographql/apollo-client
[about]: http://localhost:3000/about