# APW - Apollo Phoenix Websocket

<a href="https://travis-ci.org/vic/apollo-phoenix-websocket"><img src="https://travis-ci.org/vic/apollo-phoenix-websocket.svg"></a>
[![help maintain this lib](https://img.shields.io/badge/looking%20for%20maintainer-DM%20%40vborja-663399.svg)](https://twitter.com/vborja)


This node module implements an [Apollo GraphQL Network Layer] using [Phoenix Channels]

Since version `0.6.0`, all Apollo operations are supported: queries, mutations, watchQueries (pooling) and
subscriptions.

Using the [Apollo Client], queries and mutations resolve to promises, and watchQueries and
subscriptions resolve to observables.

See the Apollo client [documentation][Apollo Client] for more info on how to invoke your gql backend.

For those wondering why to use Apollo instead of directly executing your GQL queries via a simple
websocket, Apollo has an internal store that let's you optimize the queries and only ask for what it
needs, much like Facebook's Relay does.


## Installation

```shell
npm install --save apollo-phoenix-websocket
```

## Usage

Just import the `createNetworkInterface` from APW and use it to create an ApolloClient.

The `networkInterface` function takes an options object, the only required
property is `uri` which specifies your endpoint websocket address.

```javascript
import ApolloClient from 'apollo-client'
import {createNetworkInterface} from 'apollo-phoenix-websocket'

// Nothing to configure if you are using an Absinthe backend
// Otherwise take a look at the Options section.
const networkInterface = createNetworkInterface({uri: 'ws://localhost:4000/socket'})

const apollo = new ApolloClient({networkInterface})
```

## Options

Most likely, (as you are looking for a phoenix-websocket transport) you might be using
the [Absinthe] library to implement your Elixir GQL server. APW is configured by default
to work out of the box with an [Absinthe backend].

But if need araises, you can supply some advanced options to customize how it works.
Here's is a commented example of the options that you can set for APW:


```javascript
createNetworkInterface({

  // The websockets endpoint to connect to, like wss://example.com:4000/socket
  uri: WS_URI,

  // how to send queries and mutations
  channel: {
    topic: '__absinthe__:control',
    event: 'doc',
  },

  // for using websocket subscriptions
  subscription: (subscriptionResponse) => ({
    topic: subscriptionResponse.subscriptionId,
    event: 'subscription:data',

    // extract the data from the event payload
    map: payload => payload.result.data,

    // what to do when unsubscribing
    off: controlChannel => {
      controlChannel.push('unsubscribe', subscriptionResponse.subscriptionId)
    }
  }),

  // If you want to reuse an existing Phoenix Socket, just provide a function
  // for APW to get it. By default, it will use the Phoenix Socket module.
  Socket: options => new Socket(options),
})
```

## Middlewares

You can use middlewares with `use` just like with
the standard apollo network interface.

```javascript
networkInterface.use([{
  applyMiddleware({request, options}, next) {
    // Here you can modify the interface options, for example
    // you can change the socket/channel that will handle the request
    // For example for a channel expecting authenticated queries
    options.channel.topic = "gql:secured"
    options.channel.params = {token: "phoenix_token"}

    // Or Modify the request
    request.variables = {name: 'Luke'}

    next()
  }
}])
```

## Afterware

You can use afterwares with `useAfter` just like the standard
apollo network interface. An example use-case is for error handling:

```javascript
networkInterface.useAfter([{
  applyAfterware({response, options}, next) {
    // throw an error that will trigger your error handler
    if (response.error) {
      throw new Error(response.error)
    }
    next();
  }
}])
```

## Absinthe backend

[Absinthe] is an amazing project (kudos to @benwilson512 et al.). It's actually very
simple to create a GQL backend with it.

Take a look at the following places for more information:

- The [Absinthe guide][Absinthe] itself for getting started.
- [Absinthe Phoenix] for implementing websocket subscriptions on your endpoint.
- [Absinthe.Schema#subscription/2][Absinthe Subscription] docs for how to setup your schema



# Made with love <3

If you want to provide feedback or even better if you want to contribute some code feel free to open a [new issue].
Possible thanks to the awesome work of [our contributors].


[Apollo Client]: http://dev.apollodata.com/core/apollo-client-api.html
[Apollo GraphQL Network Layer]: http://dev.apollodata.com/core/network.html
[Phoenix Channels]: http://www.phoenixframework.org/docs/channels
[Absinthe]: http://absinthe-graphql.org/
[new issue]: https://github.com/vic/apollo-phoenix-websocket/issues
[our contributors]: https://github.com/vic/apollo-phoenix-websocket/graphs/contributors
[Absinthe Phoenix]: https://github.com/absinthe-graphql/absinthe_phoenix
[Absinthe Subscription]: https://hexdocs.pm/absinthe/1.4.0-beta.1/Absinthe.Schema.html#subscription/2
