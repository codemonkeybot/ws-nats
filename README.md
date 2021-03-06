# ws-nats
[![npm](https://img.shields.io/npm/v/ws-nats.svg)](https://www.npmjs.com/package/ws-nats)
[![License MIT](https://img.shields.io/npm/l/ws-nats.svg)](http://opensource.org/licenses/MIT)

A browser websocket client library for NATS

### Install

```bash
npm install --save ws-nats
```

### Usage

```html
<!DOCTYPE html>
<html lang="en">
<head></head>
<body>
  <script src="ws-nats.umd.js"></script>
  <script>
    var nats = NATS.connect({ url: 'ws://0.0.0.0:8080', json: true });

    nats.publish('foo', { message: 'Hello World!' });
  </script>
</body>
</html>
```

### Basic Example

```javascript
// var NATS = window.NATS;
var NATS = require('ws-nats');

var nats = NATS.connect({ url: 'ws://0.0.0.0:8080', json: true });

// Simple Publisher
nats.publish('foo', { message: 'Hello World!' });

// Simple Subscriber
nats.subscribe('foo', function(msg) {
  console.log('Received a message: ' + msg);
});

// "*" matches any token, at any level of the subject.
nats.subscribe('foo.*.baz', function(msg, reply, subject) {
  console.log('Msg received on [' + subject + '] : ' + msg);
});

nats.subscribe('foo.bar.*', function(msg, reply, subject) {
  console.log('Msg received on [' + subject + '] : ' + msg);
});

// Using Wildcard Subscriptions

// ">" matches any length of the tail of a subject, and can only be
// the last token E.g. 'foo.>' will match 'foo.bar', 'foo.bar.baz',
// 'foo.foo.bar.bax.22'
nats.subscribe('foo.>', function(msg, reply, subject) {
  console.log('Msg received on [' + subject + '] : ' + msg);
});


// Unsubscribing
var sid = nats.subscribe('foo', function(msg) {});
nats.unsubscribe(sid);

// Request Streams
var sid = nats.request('request', function(response) {
  console.log('Got a response in msg stream: ' + response);
});

// Request with Auto-Unsubscribe. Will unsubscribe after
// the first response is received via {'max':1}
nats.request('help', {}, {'max':1}, function(response) {
  console.log('Got a response for help: ' + response);
});

// Request for single response with timeout.
nats.requestOne('help', {}, {}, 1000, function(response) {
  // `NATS` is the library.
  if(response instanceof NATS.NatsError && response.code === NATS.REQ_TIMEOUT) {
    console.log('Request for help timed out.');
    return;
  }
  console.log('Got a response for help: ' + response);
});

// Replies
nats.subscribe('help', function(request, replyTo) {
  nats.publish(replyTo, 'I can help!');
});
```

> This library is compatible with all the API methods in [node-nats](https://github.com/nats-io/node-nats#basic-usage)

### Testing

To test `ws-nats`, you need to connect to a [NATS server](https://github.com/nats-io/gnatsd) using a Websocket-to-TCP relay such as [nats-relay](https://hub.docker.com/r/aaguilar/nats-relay/) or [ws-tcp-relay](https://github.com/isobit/ws-tcp-relay).

You can use Docker to run the `gnatsd` server and the Websockets to TCP relay:

```
# launch the gnatsd server
docker run -it --name=nats --rm -d -p 4222:4222 -p 8222:8222 nats -D -m 8222

# launch the relay (assumes we are using Linux!)
docker run -it --name=relay --rm -d -p 8080:8080 aaguilar/nats-relay -p 8080 $(hostname -i):4222

# then configure ws-nats to connect to the relay
var nats = NATS.connect({ url: 'ws://0.0.0.0:8080', json: true });
```

### Browser support

Tested in the following browsers versions:

* Google Chrome 53+
* Firefox 37+
* Internet Explorer 11
* Microsoft Edge 12+
* Safari 9+
* Mobile Safari 11+
* Opera 46+

### Limitations
* TLS connections to NATS server are not supported because we are using Websocket as transport
* The `nkeys` public-key signature system has been disabled from the codebase to reduce bundle size (we plan to port this at a later stage)
* The size of the library bundle has increased because we are using [sockjs-client](https://github.com/sockjs/sockjs-client), we will remove this dependency in future releases.

### Acknowledgements

This library is heavily inspired by [websocket-nats](https://github.com/isobit/websocket-nats) and re-uses the sames API methods from the original [node-nats](https://github.com/nats-io/node-nats#basic-usage) library. 
