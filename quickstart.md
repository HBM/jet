---
layout: default
title: Get Started
order: 2
---

This is a quick start guide for writing Peers using the Javascript Peer API for
[Browsers and Node.js](http://github.com/lipp/node-jet).
It may help you getting a basic understanding of Jet even if your are not
planing to use one of this Peer implementations.
The full Peer API documentation can be found [here](https://github.com/lipp/node-jet/blob/master/doc/peer.md#jetpeer-api).

## Todo-App Tutorial

For a basic understanding of how to write complete "client/server" apps, 
check out the [Todo-App tutorial](https://github.com/lipp/node-jet/blob/master/examples/todo/README.md).

## Load the jet module

```javascript
var jet = require('node-jet');
```

## Create a Peer

To interact with a Jet Daemon you need a Peer. The constructor sets up a network
connection to the Daemon. All further actions using the Peer will be based on
sending and receiving messages to/from the Daemon. If necessary, the Daemon routes
messages from Peer to Peer.

```javascript
var peer = new jet.Peer({
  url: 'ws://jet.nodejitsu.com:80'
});

peer.connect().then(function() {
  console.log('peer is connected to daemon');
});
```

## Add States
 
Peers can add States to the Daemon. A State must have a **unique path**
and a **value**, which can be of any non-function type. States are visible to other
Peers (by fetching) and any Peer may try to **set** the State to a new value.
Whenever someone tries to set a State to a new value, the corresponding `set`
callback function will be invoked. Per default, when the `set` callback does not throw an error,
a State change is posted automatically, thus keeping all other Peers and the
Daemon in sync.

```javascript
var machineName = new jet.State('machine/name', 'animal');
machineName.on('set', function(newName) {
  setMachineName(newName); // does something appropriate
});

peer.add(machineName).then(function() {
  console.log('machine/name has been added to daemon');
}).catch(function(err) {
  console.log('could not add machine/name to daemon', err);
});
```

If a State does not provide a set callback function, it is considered read-only
and an appropriate error response will be emitted.
States may also change spontaneously at will, without someone trying to
**set** the State. E.g. one could add a "cpu-load" State, which changes very
frequently.

```javascript
// add a state
var cpuLoad = new jet.State('cpu/load', readCpuLoad());

peer.add(cpuLoad).then(function() {
  // async post new value
  setTimeout(function () {
    cpuLoad.value(readCpuLoad());
  },3000);
});
```

In opposite to [Firebase](http://firebase.com) there is no persistency layer.
Jet States are "runtime" States as they die with the Peer. Adding persistency
would be a task for the respective Peer to implement and could involve
writing to disk, to a Database or setting hardware registers.

## Add Methods

Peers can also add Methods to the Daemon. Like States, Methods must have a
**unique path** and are visible to other Peers (by fetching). Whenever some Peer
calls the Method, the `call` callback function will be invoked.
Method arguments can be of any non-function type and the number of arguments
is flexible. The Method may return a value of any type or an exception might be
thrown during execution.

```javascript
// add a method
var greet = new jet.Method('greet');
greet.on('call', function(name) {
    if (name.first === 'John') {
      throw 'John is a bad guy!';
    }
    var greeting = 'Hello ' + name.first + ' ' + name.last;
    console.log(greeting);
    return greeting;
});

peer.add(greet);
```
 
## Fetch States and Methods

Other Peers can fetch States and Methods. Fetching is like having a realtime
query with automatic push notifications for changes. The fetch callback will be
invoked if:

- a State/Method is added
- a State/Method is removed
- a State's value has changed

Thus Peers are always in sync! Fetch expressions can be based on paths and/or
values. Note that the Daemon caches all States and Methods and thus is able to
"fake" add events for Peers joining the party lately. This feature allows Peers
to have hotplug-like behaviour, dependency based start-up sequences etc.

```javascript
var engineers = new jet.Fetcher()
  .path('startsWith', 'engineers/')
  .on('data', function(path, event, engineer) {
    if (path === 'add') {
	  console.log('hello engineer', engineer.name);
	} else if (path === 'change') {
	  console.log('has your salary been raised?', engineer.salary)
	} else if (path === 'remove') {
	  console.log('good bye', engineer.name);
	}
  });

peer.fetch(engineers);
```

If desired, the fetched elements can also be delivered sorted, e.g. querying the
top ten female players could look like:

```javascript
var topTenFemalePlayers = new jet.Fetcher()
  .path('startsWith', 'players/')
  .key('gender', 'equals', 'female')
  .sortByKey('score', 'number')
  .descending()
  .range(1, 10)
  .on('data', function(players) {
    console.log('scoreboard just changed:');
	console.log(players);
  });

peer.fetch(topTenFemalePlayers);
```

This saves the clients bandwidth as only a subset of information goes over the wire and
is easy to handle, since always a complete and correct set of information (Array) is
passed to the fetch "data" handler.

## Set States

Any Peer may (try) to set States to other values. This operation may succeed
or fail as determined by the implementation of the `set` callback function defined
by the owning Peer of the respective State. The Daemon will route the request
to the Peer which added the State (or reply with an error if the State is not
available).

```javascript
peer.set('foo', 6271)
  .then(function() {
    console.log('great success');
  }).catch(function(err) {
    console.log('omg', err);
  });
```

You must not make any assumptions about side-effects of a set call without error,
as the Peer implementing the State's `set` callback is free to do anything appropriate
with the requested new value (merging, adjusting, translating).
In particular, the actual value of a State can only be determined by fetching
it or by explicitly requesting it during the `set` call:

```javascript
peer.set('foo', 6271, {valueAsResult: true})
  .then(function(realNewValue) {
    console.log('the new "read" value is', realNewValue);
  }).catch(function(err) {
    console.log('omg', err);
  });
```
 
## Call Methods

Any Peer may (try) to call Methods. The return value is determined by the
implementation of the `call` callback function defined by the owning Peer of the
respective Method. The Daemon will route the request to the Peer which added the
Method (or reply with an error if the Method is not available).

```javascript
peer.call('greet', {first: 'Jim', last: 'White'})
  .then(function() {
    console.log('great success');
  }).catch(function(err) {
    console.log('omg', err);
  });
```

## Remove States

States can be removed by the owning Peer. States are also implicitly removed by
the Daemon if the network connection to a Peer breaks. In any case, all fetching
Peers will be informed.

```javascript
var foo = new jet.State('foo', 123); 

peer.add(foo).then(function() {
  foo.remove();
});
```

## Remove Methods
 
Methods can be removed by the owning Peer. Methods are also implicitly removed by
the Daemon if the network connection to a Peer breaks. In any case, all fetching
Peers will be informed.

```javascript
var greet = new jet.Method('greet'); 
greet.on('call', function() {});

peer.add(greet).then(function() {
  greet.remove();
});
```
