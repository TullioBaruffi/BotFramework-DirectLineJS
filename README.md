# Microsoft Bot Framework Direct Line library for JavaScript

Client library for the [Microsoft Bot Framework](http://www.botframework.com) [DirectLine](https://docs.botframework.com/en-us/restapi/directline3/) protocol.

Used by [WebChat](https://github.com/Microsoft/BotFramework-WebChat) and thus (by extension) [Emulator](https://github.com/Microsoft/BotFramework-Emulator), WebChat channel, and [Azure Bot Service](https://azure.microsoft.com/en-us/services/bot-service/).

## FAQ

### *Who is this for?*

Anyone who is building a Bot Framework JavaScript client who does not want to use [WebChat](https://github.com/Microsoft/BotFramework-WebChat). (WebChat already includes DirectLine so there is no reason to include this package too).

### *What is Rx?*

**R**eactive E**x**tensions are a set of libraries for different languages implementing Reactive Functional Programming a.k.a. the Observable pattern. This library uses [RxJS](https://github.com/reactivex/rxjs/).

### *Can I use [TypeScript](http://www.typescriptlang.com)?*

You bet.

### How ready for prime time is this library?

This is an official Microsoft-supported library, and is considered largely complete. Future changes (aside from supporting future updates to the Direct Line protocol) will likely be limited to bug fixes, performance improvements, tutorials, and samples. The big missing piece here is unit tests.

That said, the public API is still subject to change.

## How to build from source

0. Clone this repo
1. `npm install`
2. `npm run build` (or `npm run watch` to rebuild on every change)

## How to include in your app

There are several ways:

1. Build from scratch and include either `/directLine.js` (webpacked with rxjs) or `built/directline.js` in your app
2. Use the unpkg CDN, e.g. `<script src="http://unpkg.com/botframework-directlinejs/directLine.js"/>`
3. `npm install botframework-directlinejs`

## How to create and use a directLine object

### First, obtain security credentials for your bot:

1. If you haven't already, [register your bot](https://dev.botframework.com/bots/new).
2. Add a DirectLine (**not WebChat**) channel, and generate a Direct Line Secret. Make sure to Direct Line 3.0 is enabled.
3. For testing you can use your Direct Line Secret as a security token, but for production you will likely want to exchange that Secret for a Token as detailed in the Direct Line [documentation](https://docs.botframework.com/en-us/restapi/directline3/).

### Second, create a DirectLine object:

    var directLine = new DirectLine({
        secret: /* put your Direct Line secret here */,
        token: /* or put your Direct Line token here (supply secret OR token, not both) */,
        domain: /* optional: if you are not using the default Direct Line endpoint, e.g. if you are using a region-specific endpoint, put its full URL here */
        webSocket: /* optional: true if you want to use WebSocket to receive messages. Currently defaults to false. */,
    });

### Post activities to the bot:

    directLine.postActivity({
        from: { id: 'myUserId', name: 'myUserName' }, // required (from.name is optional)
        type: 'message',
        text: 'a message for you, Rudy'
    }).subscribe(
        id => console.log("Posted activity, assigned ID ", id),
        error => console.log("Error posting activity", error)
    );

You can also post messages with attachments, and non-message activities such as events, by supplying the appropriate fields in the activity.

### Listen to activities sent from the bot:

    directLine.activity$
    .subscribe(
        activity => console.log("received activity ", activity)
    );

You can use RxJS operators on incoming activities. To see only message activities:

    directLine.activity$
    .filter(activity => activity.type === 'message')
    .subscribe(
        message => console.log("received message ", message)
    );

Direct Line will helpfully send your client a copy of every sent activity, so a common pattern is to filter incoming messages on `from`:

    directLine.activity$
    .filter(activity => activity.type === 'message' && activity.from.id !== 'yourBotHandle')
    .subscribe(
        message => console.log("received message ", message)
    );

### Monitor connection status

Subscribing to either `postActivity` or `activity$` will start the process of connecting to the bot. Your app can monitor the current connection status and react appropriately :

    directLine.connectionStatus$
    .subscribe(connectionStatus =>
        switch(connectionStatus) {
            case ConnectionStatus.Uninitialized:    // the status when the DirectLine object is first created/constructed
            case ConnectionStatus.Connecting:       // currently trying to connect to the conversation
            case ConnectionStatus.Online:           // successfully connected to the converstaion. Connection is healthy so far as we know.
            case ConnectionStatus.ExpiredToken:     // last operation errored out with an expired token. Your app should supply a new one.
            case ConnectionStatus.FailedToConnect:  // the initial attempt to connect to the conversation failed. No recovery possible.
            case ConnectionStatus.Ended:            // the bot ended the conversation
        }
    );

### Reconnecting to a conversation

If your app created your DirectLine object by passing a token, DirectLine will refresh that token every 15 minutes.
Should your client lose connectivity (e.g. close laptop, fail to pay Internet access bill, go under a tunnel), `connectionStatus$`
will change to `ConnectionStatus.ExpiredToken`. Your app can request a new token from its server, which should call
the [Reconnect](https://docs.botframework.com/en-us/restapi/directline3/#reconnecting-to-a-conversation) API. 
The resultant Conversation object can then be passed by the app to DirectLine:

    var conversation = /* a Conversation object obtained from your app's server */;
    directLine.reconnect(conversation);

## Copyright & License

© 2017 Microsoft Corporation

[MIT License](/LICENSE)