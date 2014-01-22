---
layout: default
title: Jet
---

# Quick Start

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

Other Peers can fetch States and Methods. Fetching is like having a realtime query with automatic push notifications for changes. Fetch expressions can be based on paths and values. If desired, the fetched elements can also be delivered sorted.

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

## Set States

Other Peers may (try) to set States to other values. This operation may succeed or fail as determined by the implementation of the set callback function defined by the owning Peer of the respective State.

```javascript
peer.set('foo',6271);
```

## Call Methods

Other

# Concepts

## Daemon

## Peer

## States

## Methods

## Fetch

# Communication
