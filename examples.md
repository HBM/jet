---
layout: default
title: Examples
order: 3
---

The following examples demonstrate Javascript peer API usage and (protocol) messages.
They actually run in your browser. You can edit them on [codepen](http://codepen.io) if desired.
The examples use a public Jet Daemon, which also hosts a Todo-App. Thus
you may notice activity of other peers "playing" with it. 
To view everything in a nice debug-perspective, open <a href="radar.html" target="_blank">Radar</a>.

## Todo-App Tutorial

For a basic understanding of how to write complete "client/server" apps, 
check out the [Todo-App tutorial](https://github.com/lipp/node-jet/blob/master/examples/todo/README.md).

# Connect

This example creates a peer and reports back the connection status.
Daemons can run on the same machine as the Peer or on a remote site.

The most relevant code snippet is:

```javascript
var peer = new jet.Peer({
      url: 'ws://jet.nodejitsu.com:80'
});

peer.connect().then(function() {
  
});
```
<p data-height="268" data-theme-id="0" data-slug-hash="GEyuq" data-default-tab="result" data-user="lipp" class='codepen'>See the Pen <a href='http://codepen.io/lipp/pen/GEyuq/'>Jet Connect</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>


# Add States
 
This example creates a Peer and adds two States with random. One, ignoring the
Daemon response, the other one logging the Daemon response to console. The message
traffic between the Daemon and the Peer is visible in the result window bottom.

The most relevant code snippet is:

```javascript
// create read-only state (no set callback provided)
// ignore the Daemon response by leaving out the callback object.
// the request is a notification.
var foo = new jet.State('foo', 123);
peer.add(foo);

// create writable state,
// provide callback object with error and success handlers.
// the Daemon will send a response, as the request is not a
// notification.
var bar = new jet.State('bar', {age: 123});
bar.on('set', function(newVal) {
	if (newVal.age > this.value().age) {
	  console.log('bar', newVal);
	} else {
	  throw new Error('arg');
	}
});
```

<p data-height="609" data-theme-id="0" data-slug-hash="kLlfB" data-default-tab="result" data-user="lipp" class='codepen'>See the Pen <a href='http://codepen.io/lipp/pen/kLlfB/'>Jet Add State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>


# Fetch Simple

This examples creates a Peer, adds two States with random name ("random...")
and fetches everything with a path starting with "random".
The message traffic between the Daemon and the Peer is visible in the result
window bottom. Compare the (fetch) `id` of the `fetch` message with the incoming
messages' `method` field. The incoming messages can be considered __Passive__ as
they are Requests targeted at the Peer, embedding a Peer defined `method` field
value.

This is the most relevant snippet:

```javascript
// fetch and provide: 1) fetch rule, 2) callback for fetching
var randomStuff = new jet.Fetcher()
  .path('startsWith', 'random')
  .on('data', function(path, event, value) {
  });

peer.fetch(randomStuff);
```

<p data-height="701" data-theme-id="0" data-slug-hash="Cglby" data-default-tab="result" data-user="lipp" class='codepen'>See the Pen <a href='http://codepen.io/lipp/pen/Cglby/'>Jet Fetch State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

# Change State

This examples creates a Peer, adds a "ticker" States with random name (so that at least
these State are available for fetching) and fetches everything with a path starting
with "ticker". Every second the ticker value is incremented by one.
The message traffic between the Daemon and the Peer is visible
in the result window bottom.

This is the most relevant snippet:

```javascript
var ticker = new jet.State('ticker', 1);

peer.add(ticker).then(function() {
  setInterval(function() {
    var old = ticker.value();
    ticker.value(++old); // post state change
  }, 1000);
});
```

<p data-height="891" data-theme-id="0" data-slug-hash="eKBpG" data-default-tab="result" data-user="lipp" class='codepen'>See the Pen <a href='http://codepen.io/lipp/pen/eKBpG/'>Jet Change State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

