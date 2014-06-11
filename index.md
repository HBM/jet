---
layout: default
title: Jet
---

# About

Jet is a lightweight message protocol for realtime communication between apps.
These apps may run within browsers, on servers or even on embedded systems with
very limited ressources. It may be considered a realtime auto-sync database like
[Firebase](http://firebase.com) or as a web-enabled system bus like
[DBus](dbus.freedesktop.org).

It employs Websockets as transport and JSON-RPC 2.0 as message format.
On top, Jet adds few simple concepts and a handfull of message definitions which
allow efficient, flexible and transparent information flow. Implementations are
pretty small, e.g. the full-featured Javascript Peer for
[Browsers](https://github.com/lipp/jet-js/blob/master/peer.js) has < 700
lines of code (SLOC) and uses minified and gzipped < 2k bytes.

# Jet in 10 Minutes

This is a quick start for writing Peers using the or Javascript Peer API for
[Browsers](http://github.com/lipp/jet-js) and [Node.js](http://github.com/lipp/node-jet).
It may help you getting a basic understanding even if your are not planing to use
one of this Peer implementations. The full documentation can be found
[here](#API).

Note that Node.js users have to require the jet module.

```javascript
var jet = require('jet');
```

## Create a Peer

To interact with a Jet Daemon you need a Peer. The constructor sets up a network
connection to the Daemon. Peers always communicate with the Daemon and never
communicate directly.

```javascript
var peer = new jet.Peer();
```

## Add States

Peers can add States to the Daemon connected. A State must have a **unique path**
and a **value**, which can be of any (non-function) type. States are visible to other
Peers (by fetching) and any Peer may try to **set** the State to a new value.
Whenever someone tries to set a State to a new value, the corresponding `set`
callback function will be invoked. If a State does not provide a set callback
function, it is considered read-only and an appropriate error response will be emitted.
Per default, when the `set` callback does not throw an error, a State change is
posted automatically, thus keeping all other Peers and the Daemon in sync.

States may also change at any time spontaneously.

```javascript
// add a state
var fooVal = 1234;
var foo = peer.state({
  path: 'foo',
  value: fooVal,
  set: function (newVal) {
    fooVal = newVal;
  }
});

// async set to a new value
setTimeout(function () {
  fooVal = 627;
  foo.value(fooVal);
},3000);
```

## Add Methods

Peers may also add Methods. Like States, Methods must have a **unique path** and
are visible to other Peers (by fetching). Whenever someone calls the Method
(through the Jet Daemon), the `call` callback will be invoked.
Method arguments can be of any (non-function) type and the number of arguments
is flexible. The Method may return a value of any type or an exception might be
thrown during execution.

```javascript
// add a method
var greet = peer.method({
  path: 'greet',
  call: function (name) {
    if (name === 'John') {
      throw 'John is a bad guy!';
    }
    console.log('Hello', name);
  }
});
```

## Fetch States and Methods

Other Peers can fetch States and Methods. Fetching is like having a realtime
query with automatic push notifications for changes. The fetch callback will be
invoked if:
 - a State/Method is added
 - a State/Method is removed
 - a State changed its value
Fetch expressions can be based on paths and/or values. Note the Daemon caches all
States and Methods and thus is able to "fake" add events for Peers joining the
party lately. This feature allows Peers to have hotplug-like behaviour.

```javascript
peer.fetch({
  path: {
    equalsOneOf: ['foo','addNumbers']
  }, function (path, event, value) {
      // the value is undefined for methods
  }
});
```

If desired, the fetched elements can also be delivered sorted, e.g. querying the
top ten players could look like:

```javascript
peer.fetch({
    path: {
      startsWith: 'players/'
    },
    sort: {
      from: 1,
      to: 10,
      descending: true,
      byValueField: {
        score: 'number'
      }
    }, function (sorted) {
  }
});
```

## Set States

Any Peer may (try) to set States to other values. This operation may succeed
or fail as determined by the implementation of the `set` callback function defined
by the owning Peer of the respective State. The Daemon will route the request
to the Peer which added the State (or reply with an error if the State is not
available).

```javascript
peer.set('foo', 6271);
```

## Call Methods

Any Peer may (try) to call Methods. The return value is determined by the
implementation of the `call` callback function defined by the owning Peer of the
respective Method. The Daemon will route the request to the Peer which added the
Method (or reply with an error if the Method is not available).

```javascript
peer.call('greet', 'Rupert');
```

## Remove State

States can be removed by the owning Peer. States are also implicitly removed by
the Daemon if the network connection to a Peer breaks. In any case, all fetching
Peers will be informed.

```javascript
foo.remove();
```

## Remove Method

Methods can be removed by the owning Peer. Methods are also implicitly removed by
the Daemon if the network connection to a Peer breaks. In any case, all fetching
Peers will be informed.

```javascript
greet.remove();
```


# Concepts

Jet is built around some simple but powerful concepts. There are two kinds of
"players" involved in a Jet setup:
- A Daemon
- Any Number of Peers
All Peers which are able to connect to the Daemon's Jet Websocket service can join
the party. Whereas the Daemon provides the infrastructure, the Peers provide the
content:
- States
- Methods
Either one must have a unique path to avoid name clashes
(which would introduce ambiguous routes) and can be considered
services/information offered by the Peer. Once setup the Daemon takes care of
the message routing and keeps track of a cache with all the stuff added. When the
Daemon loses network connection to a Peer, every ressource (fetch rules, States,
Methods, outstanding messages) are properly cleaned up.

The one and only mean to query information about States and Methods from the Daemon
is by using the powerful fetch command. By design there is no "get" command,
which may tempt the user to poll. This keeps the traffic and the work for the Daemon
low, but at the same time enables fast-as-possible push messages for relevant
events.


## Daemon

The Daemon is the center of all communication. All messages flow between Peers
and the Daemon. Anyhow, in most cases a Peer has not to be aware of the Daemon inner workings,
but just has to know where it is running (Websocket URL).

The Daemon is able to route messages if required and keeps track
of all Methods and States (and its associated values). It is much like a
phonebook as you can (try) to set a State or call a Method based on its unique
Path. The daemon also manages ownership of States and Methods as no Peer may
remove a State or Method which was not added by the very same Peer in the first
place. Also, States and Methods are automatically removed if a Peer closes the
connection to the daemon. Fetch rules are stored and processed as well.

There are implementations available for Lua ([lua-jet](http://github.com/lipp/lua-jet))
and for Node.js ([node-jet](http://github.com/lipp/node-jet)).

## Peer

There can be any number of Peers. Peers bring the Jet bus to life. They can do any of the following things:

- Add / Remove States and Methods
- Call Methods
- Set States
- Fetch / Unfetch States and Methods

## States

A State is made up a unique Path and an optional value. The value of a State can be any valid JSON, as primitive Types (Boolean, Numbers, Strings) or nested Objects are. Peers can add States to the bus by calling __add__, e.g.:

```javascript
{
  "method": "add",
  "params": {
    "path": "foo/bar",
    "value": 123
  }
  "id": 41
}
```

```javascript
{
  "method": "add",
  "params": {
    "path": "person/Xop",
    "value": {
      "name": "Bob",
      "age": 26,
      "hobbies": ["Hiking", "Swimming"]
    }
  }
  // Note that this is a Notification as there is no "id"
}
```

As time goes by, States are most probably subject to change. States are allowed (or even expected) to change at any time to any new value. The Peer has just to inform the daemon (and thus eventually other Peers) by calling __change__ making a state change public, e.g.

```javascript
{
  "method": "change",
  "params": {
    "path": "foo/bar",
    "value": false // yes, state values may change type
  }
  "id": 41
}
```

```javascript
{
  "method": "change",
  "params": {
    "path": "person/Xop",
    "value": {
      "name": "Bob",
      "age": 27,
      "hobbies": ["Computer Games", "Climbing"]
    }
  }
  // Note that this is a Notification as there is no "id"
}
```

A State change is always a replacement. The Daemon manages a State cache and stores the current (last known) value of all States currently added. This enables the daemon to provide the correct and complete information for fetchers who join the party lately.

## Methods

## Fetch

# Protocol

Jet heavily relies on [JSON-RPC 2.0](http://www.jsonrpc.org/specification) semantics for all of its messages. There are some minor changes to the spec, however. To be able to follow the protocol details, the reader must be familiar with the (JSON-RPC 2.0) terms: Request, Response, Notification, Error Object and Batch.

As a reminder: Don't be confused by the term Notification. A Notification is simply a Request without an __id__  specified, thus indicating no Response to the Request is expected.

## Architecture

- There must be one Daemon.
- There can be any number of Peers.


Peers never communicate directly. Peers always communicate with a Daemon. Peer can (indirectly) interact between each other through a Daemon. The Daemon may send messages to other Peers when one of two things happen:

- A Peer sends a message to the Daemon
- A Peer connection closes (and thus States and Methods are removed)


## Active / Passive

There are two different kinds of message flows:

-  __Active__: The Peer sends a Request to the Daemon
-  __Passive__: The Daemon sends a Request to the Peer


The __Active__ messages always originate from Peers and are send to the Daemon. Eventually the Daemon may send __Passive__ messages to one or more Peers as a result of processing an __Active__ message. For instance: If a Peer __adds__ a State, another __fetching__ Peer with matching fetch rules may be informed by __Passive__ messages.

The __Passive__ messages are:

- Fetch based messages
- (Routed) Requests to set (change) a State
- (Routed) Requests to call a Method

The method name field of __Passive__ messages are Peer defined (via __add__ and __fetch__), whereas the method name field of __Active__ messages is always on of __add__, __remove__, __fetch__, __unfetch__, __set__, __call__, __change__ or __config__.

### Example for Active Message

In this example a Peer fetches all persons (all states and methods, where the path starts with "person"). The side-effect of this message is that, the Daemon may send a __Passive__ message with `"method":"personFetcher"` to the peer whenever appropriate.

```javascript
{
  "method": "fetch",
  "params": {
    "id": "personFetcher",
    "path": {
      "startsWith": "person"
    }
  },
  "id": 762
}
```

### Example for Passive Message

This is a __Passive__ message which is issued based on the fetch rule defined above. Note the method name, which has been specified earlier in the fetch call.

```javascript
{
  "method": "personFetcher",
  "params": {
    "path": "person/asdlkjasdk92",
    "value": {
      "name": "Paul",
      "age": 63
    }
  },
  "id": 762
}
```

## Differences to JSON-RPC 2.0

In opposite to the [JSON-RPC 2.0 spec](http://www.jsonrpc.org/specification) the
`"jsonrpc": "2.0"` field is considered optional and can be simply ignored.

Further __Batch__ Messages are not subject of any order requirements are may be
processed and send at any time. This allows to minimize message framing overhead (e.g. Websockets).

# Live Examples

The following examples demonstrate Javascript peer API usage and (protocol) messages.
They actually run in your browser. You can edit them on [codepen](http://codepen.io) if desired.
Most of them use the Jet Daemon hosted at [nodejitsu](http://nodejitsu.com) with the
Daemon URL: `ws://jet.nodejitsu.com` (Port 80). The nodejitsu jet daemon is public
and you may notice activity of other peers "playing" with it.

## Connect

This example creates a peer and reports back the connection status.

The most relevant code snippet is:

```javascript
var peer = new jet.Peer({
      url: 'ws://jet.nodejitsu.com:80',
      onOpen: function() {}
    });
```

<div data-height="619" data-theme-id="0" data-slug-hash="GEyuq" data-default-tab="js" class='codepen'><pre><code>var connect = function(url) {
   try {
    $(&#x27;#status&#x27;).text(&#x27;disconnected&#x27;);
    // create a Jet Peer, providing the Jet (Daemon) Websocket URL
    var peer = new jet.Peer({
      url: url,
      onOpen: function() {
        $(&#x27;#status&#x27;).text(&#x27;connected&#x27;);
      },
    });
  } catch(err) {
    $(&#x27;#status&#x27;).text(&#x27;error &#x27; + err);
  }
};

// try to (re-)connect when button is clicked
$(&#x27;button&#x27;).click(function(e) {
  connect($(&#x27;input&#x27;).val());
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});

// initially try to reach the jet daemon hosted at nodejitsu.com (which listens on port 80)
connect(&#x27;ws://jet.nodejitsu.com&#x27;);</code></pre>
<p>See the Pen <a href='http://codepen.io/lipp/pen/GEyuq/'>Jet Connect</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>


## Add States

This example creates a Peer and adds two States with random. One, ignoring the
Daemon response, the other one logging the Daemon response to console. The message
traffic between the Daemon and the Peer is visible in the result window bottom.

The most relevant code snippet is:

```javascript
// create read-only state (no set callback provided)
// ignore the daemon response by leaving out the callback object.
// the request is a notification.
peer.state({
  path: 'foo',
  value: 123
});

// create writable state,
// provide callback object with error and success handlers.
// the daemon will send a response, as the request is not a
// notification.
var bar = 'hello';
peer.state({
  path: 'bar',
  value: bar,
  set: function(newVal) {
      bar = newVal;
  }
  }, {
  success: function() {},
  error: function(err) {}
});
```

<div data-height="591" data-theme-id="0" data-slug-hash="kLlfB" data-default-tab="js" class='codepen'><pre><code>var connect = function(url) {
   try {
    $(&#x27;#status&#x27;).text(&#x27;disconnected&#x27;);
    // create a Jet Peer, providing the Jet (Daemon) Websocket URL
    var peer = new jet.Peer({
      url: url,
      onOpen: function() {
        $(&#x27;#status&#x27;).text(&#x27;connected&#x27;);
      },
      onSend: function(message) {
        addLogEntry(&#x27;Out&#x27;,message);
      },
      onReceive: function(message) {
        addLogEntry(&#x27;In&#x27;,message);
      }
    });
     var random = &#x27;random&#x27; + new Date().getTime();
     // add as notification
     peer.state({
       path: random,
       value: 123
     });

     random = &#x27;random_2_&#x27; + new Date().getTime();
     // add as request
     peer.state({
       path: random,
       value: 123,
       set: function(newValue) {
         random = newValue;
       }
     },{
       success: function() {
         console.log(&#x27;ok&#x27;);
       },
       error: function(err) {
         console.log(err);
       }
     });
  } catch(err) {
    $(&#x27;#status&#x27;).text(&#x27;error &#x27; + err);
  }
};



// try to (re-)connect when button is clicked
$(&#x27;button&#x27;).click(function(e) {
  connect($(&#x27;input&#x27;).val());
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});

// initially try to reach the jet daemon hosted at nodejitsu.com (which listens on port 80)
connect(&#x27;ws://jet.nodejitsu.com&#x27;);
var off;
var addLogEntry = function(direction, message) {
  var now = new Date().getTime();
  if (!off) {
    off = now;
  }
  var tr = $(&#x27;&lt;tr&gt;&lt;/tr&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + (now-off) + &#x27;&lt;/td&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + direction + &#x27;&lt;/td&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + message + &#x27;&lt;/td&gt;&#x27;);
  $(&#x27;#log tbody&#x27;).append(tr);
};

</code></pre>
<p>See the Pen <a href='http://codepen.io/lipp/pen/kLlfB/'>Jet Add State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>
