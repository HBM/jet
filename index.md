---
layout: default
title: Jet
---

# Quick Start

This is a Quick Start for writing Peers using the or Javascript Peer API for [Browsers](http://github.com/lipp/jet-js) and [Node.js](http://github.com/lipp/node-jet). It may help you getting a basic understanding even if your are not planing to use this Peer API.

## Create a Peer

To interact with a Jet Daemon you need a Peer. A peer sets up a network connection to the Daemon and performs some configuration steps. Peers always communicate with the Daemon and never communicate directly.

```javascript
var jet = require('jet');
var peer = new jet.Peer();
```

## Add States

Peers can add States to the Daemon. A State must have a unique path and a value, which can be of any (non-function) type. They are visible to other Peers. Whenever someone tries to set a State through the Daemon to a new value, the corresponding set callback function will be invoked. States may also change at any time spontaneously.


```javascript
// add a state
var foo = peer.state({
  path: 'foo',
  value: 1234,
  set: function (newVal) {
    console.log('foo is now set to',newVal);
  }
});

// async set to a new value
setTimeout(function () {
  foo.value(6257);
},3000);
```


## Add Methods

Peers may also add Methods. Like States, Method must have a unique path and are visible to other Peers. Whenever someone calls the Method through the Jet Daemon, the call callback will be invoked. Method arguments can be of any (non-function) type and the number of arguments is flexible. The Method may return a value of any type or a exeception might be thrown during execution.

```javascript
// add a method
var greet = peer.method({
  path: 'greet',
  call: function (name) {
    if (name === 'John') {
      throw 'John was a bad guy!';
    }
    console.log('Hello',name);
  }
});
```

## Fetch States and Methods

Other Peers can fetch States and Methods. Fetching is like having a realtime query with automatic push notifications for changes. Fetch expressions can be based on paths and values.

```javascript
peer.fetch({
  path: {
    equalsOneOf: ['foo','addNumbers']
  }, function (notification) {
    console.log(notification);
	// will show:
	// {"path": "foo", "event": "add", "value": 1234}
	// {"path": "addNumbers", "event": "add"}
  }
});
```

If desired, the fetched elements can also be delivered sorted, e.g. querying the top ten players could look like:

```javascript
peer.fetch({ // first param is the fetch rule
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
    }
  }, function (sorted) { // second param is the fetch callback
    console.log(sorted);
	// will show:
	// { "n": 10,
        //   "changes": [
        //     {"path": "players/apXsdi", "event": "add", "value": { "nick": "BobTheNerd", "score": 999890} , index: 1},
        // {"path": "players/apiesda", "event": "add", "value": { "nick": "Superman", "score": 920} , index: 2}
        // ...
        //   ]
        // }
  }
});
```


## Set States

Other Peers may (try) to set States to other values. This operation may succeed or fail as determined by the implementation of the set callback function defined by the owning Peer of the respective State.

```javascript
peer.set('foo',6271);
```

## Call Methods

Other

# Concepts

## Daemon

The center of communication. All messages flow between a Peer and the Daemon. The Daemon is able to route messages if required and keeps track of all Methods and States (and its associated values). It is much like a phonebook as you can (try) to set a State or call a Method based on its unique Path. The daemon also manages ownership of States and Methods as no Peer may remove a State or Method which was not added by the very same Peer in the first place. Also States and Methods are automatically removed if a Peer closes the connection to the daemon. Fetch rules are stored and processed as well.

There are implementations available for Lua ([lua-jet](http://github.com/lipp/lua-jet)) and for Node.js ([node-jet](http://github.com/lipp/node-jet)).

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

# Protocol (Version 0.9)

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

In opposite to the [JSON-RPC 2.0 spec](http://www.jsonrpc.org/specification) the `"jsonrpc": "2.0"` field is considered optional and can be simply ignored.

Further __Batch__ Messages are not subject of any order requirements are may be processed and send at any time. This allows to minimize message framing overhead (e.g. Websockets).

# Live Examples

The following examples demonstrate Javascript peer API usage and (protocol) messages.
They actually run in your browser. You can edit them on [codepen](http://codepen.io) if desired.
Most of them use the Jet Daemon hosted at [nodejitsu](http://nodejitsu.com) with the
Daemon URL: `ws://jet.nodejitsu.com` (Port 80). The nodejitsu jet daemon is public
and you may notice activity of other peers "playing" with it.

## Connect

Creates a peer and reports back the connection status.

<div data-height="636" data-theme-id="0" data-slug-hash="GEyuq" data-default-tab="js" class='codepen'><pre><code>
var connect = function(url) {
try {
  $(&#x27;#status&#x27;).text(&#x27;disconnected&#x27;);

  var peer = new jet.Peer({
    url: url,
    onOpen: function() {
      $(&#x27;#status&#x27;).text(&#x27;connected&#x27;);
    }
  });

} catch(e) {
  $(&#x27;#status&#x27;).text(&#x27;error &#x27; + e);
  $(&#x27;#status&#x27;).style({color: &#x27;red&#x27;});
}
};

$(&#x27;button&#x27;).click(function(e) {
  connect($(&#x27;input&#x27;).val());
});

$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});

connect(&#x27;ws://jet.nodejitsu.com&#x27;);</code></pre>
<p>See the Pen <a href='http://codepen.io/lipp/pen/GEyuq/'>Jet Connect</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>

## Add State

Creates a peer and then adds a state with random path name (path must be unique on the jet bus).

<p data-height="650" data-theme-id="0" data-slug-hash="kLlfB" data-default-tab="js" class='codepen'>See the Pen <a href='http://codepen.io/lipp/pen/kLlfB/'>Jet Add State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//codepen.io/assets/embed/ei.js"></script>
