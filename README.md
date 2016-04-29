
## ff-devtools-libs

This is an experimental repo whose sole purpose is to allow developing
Firefox Developer Tools code in a normal HTML context. It originated
from the [devtools.html](https://github.com/joewalker/devtools.html)
experiment.

The most important piece is a WebSocket transport that allows the page
to connect to a remote Firefox debugger instance. This lets a normal
HTML page access a real live debugging environment.

A lot of other dependencies are provided as well. **If you develop
against this, your code should run without changes just fine in the
Firefox DevTools context as well**.

Install it via npm:

```
npm install ff-devtools-libs
```

Connecting a remote is as easy as (see below for aliasing `devtools`
to `ff-devtools-libs`):

```js
const { DebuggerClient } = require('devtools/shared/client/main');
const { DebuggerTransport } = require('devtools/transport/transport');
const { TargetFactory } = require("ff-devtools-lib/client/framework/target");
const socket = new WebSocket("ws://localhost:9000");
const transport = new DebuggerTransport(socket);
const client = new DebuggerClient(transport);
```

And then you can make whatever API calls you need:

```js
yield client.connect();
const response = yield client.listTabs();
const tab = response.tabs[response.selected];
const options = { form: tab, client, chrome: false };
const target = yield TargetFactory.forRemoteTab(options);

target.activeTab.attachThread({}, (res, threadClient) => {
  threadClient.resume();
});
```

You will need to create aliases for the top-level `devtools` and `sdk`
paths. Example webpack config:

```js
{
  resolve: {
    alias: {
      "devtools": "ff-devtools-libs",
      "sdk": "ff-devtools-libs/sdk"
    }
  }
}
```

Lastly, you need to run a separate proxy for this to work. This sets
up a websocket connection that forwards everything to the native
devtools TCP connection.

```
firefox-proxy [--web-socket-port 9000] [--tcp-port 6080]
```

`firefox-proxy` is installed with this module, and it exists at
`node_modules/.bin/firefox-proxy`. It is recommend to run it in
whatever `run` task you have in your npm scripts or gulpfile. It's a
simple JS file in the `bin` directory, so you may run `node
node_modules/ff-devtools-libs/bin/firefox-proxy` as well.