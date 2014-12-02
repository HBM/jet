---
layout: default
title: Jet
---

# About

Jet is a lightweight message protocol for fast and simple realtime communication
between apps. These apps may run within browsers, on servers or even on embedded
systems with very limited ressources. It may be considered a realtime auto-sync
database like [Firebase](http://firebase.com) or a web-capable system bus variant
of [DBus](http://dbus.freedesktop.org).

With Jet you can implement IoT (Internet of Things) like apps, realtime chats, online
games or anything else where stuff is in flux and fast communication is a
key requirement. It is also intended to be a web-accessable configuration interface,
which is working multi-client and asynchronous as a connection based alternative to
REST based solutions.

It employs JSON-RPC 2.0 as message format and Websockets as transport. Other
transport can be easily added, as long as they are message based and pertain connection.
On top, Jet adds few simple concepts and a handfull of message definitions which
allow for efficient, flexible and transparent information flow. Implementations are
pretty small, e.g. the full-featured Javascript Peer for
[Browsers](https://github.com/lipp/jet-js/blob/master/peer.js) has < 700
lines of code (SLOC) and the production version requires < 2k bytes.

## Demo

Open [this page](http://jetbus.io) multiple times to see Jet in action. Every
connected client/browser adds a random "Tick" State, which can be reseted to 0.
The 3 clients with the highest tick value are displayed in order of their "Tick"
value.
Watch the instant update of all clients, try to reset client ticks or close
client windows to see them disappear. The demo works on all devices
(iPad/Android/etc) with Browser which support Websockets. As all the other
Live Examples, the code is available at [codepen.io](http://codepen.io/lipp/pen/wvkre).

<div id="demo">
  <div id="control">
    <input type="text" value="ws://jet.nodejitsu.com" id="url"></input>
    <button id="connect">Connect</button>
  </div>
  <ul id="tickers">
    <li>
      <span class="circle-text path"><div>--</div></span>
      <span class="circle-text count"><div>0</div></span>
      <span class="circle-text reset"><div>Reset</div></span>
    </li>
    <li>
      <span class="circle-text path"><div>--</div></span>
      <span class="circle-text count"><div>0</div></span>
      <span class="circle-text reset"><div>Reset</div></span>

    </li>
    <li>
      <span class="circle-text path"><div>--</div></span>
      <span class="circle-text count"><div>0</div></span>
      <span class="circle-text reset"><div>Reset</div></span>
    </li>
  </ul>
  <p class="info">You are: <span id="yourid"></span></p>
  <h2 id="status">
  </h2>
  <h5 id="help"><a href="http://websocketstest.com/">Check Websockets</a></h5>
</div>


# Jet in 10 Minutes

This is a quick start guide for writing Peers using the Javascript Peer API for
[Browsers](http://github.com/lipp/jet-js) and [Node.js](http://github.com/lipp/node-jet).
It may help you getting a basic understanding of Jet even if your are not
planing to use one of this Peer implementations.
The full Peer API documentation can be found [here](https://github.com/lipp/jet-js#api).

Note, that Node.js users have to require the Jet module.

```javascript
var jet = require('jet');
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
var machineName = 'Animal';
var machineNameState = peer.state({
  path: 'machine/name',
  value: machineName,
  set: function(newName) {
    machineName = newName;
  }
});
```

If a State does not provide a set callback function, it is considered read-only
and an appropriate error response will be emitted.
States may also change spontaneously at will, without someone trying to
**set** the State. E.g. one could add a "cpu-load" State, which changes very
frequently.

```javascript
// add a state
var cpuLoadState = peer.state({
  path: 'cpu/load',
  value: readCpuLoad()
});

// async post new value
setTimeout(function () {
  cpuLoadState.value(readCpuLoad());
},3000);
```

In opposite to [Firebase](http://firebase.com) there is no persistency layer.
Jet States are "runtime" States as they die with the Peer. Adding persistency
would be a task for the respective Peer to implement.

## Add Methods

Peers can also add Methods to the Daemon. Like States, Methods must have a
**unique path** and are visible to other Peers (by fetching). Whenever some Peer
calls the Method, the `call` callback function will be invoked.
Method arguments can be of any non-function type and the number of arguments
is flexible. The Method may return a value of any type or an exception might be
thrown during execution.

```javascript
// add a method
var greet = peer.method({
  path: 'greet',
  call: function (name) {
    if (name.first === 'John') {
      throw 'John is a bad guy!';
    }
    var greeting = 'Hello ' + name.first + ' ' + name.last;
    console.log(greeting);
    return greeting;
  }
});
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
    }, function (changes, n) {
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
peer.set('foo', 6271, {
  success: function() {
    console.log('great success');
  },
  error: function(err) {
    console.log('omg', err);
  }
});

//if you dont care about the result, leave the callbacks out
peer.set('foo', 416);
```

You must not make any assumptions about side-effects of a set call without error.
In particular, the actual value of a State can only be determined by fetching
it.

## Call Methods

Any Peer may (try) to call Methods. The return value is determined by the
implementation of the `call` callback function defined by the owning Peer of the
respective Method. The Daemon will route the request to the Peer which added the
Method (or reply with an error if the Method is not available).

```javascript
peer.call('greet', {first: 'Jim', last: 'White'}, {
  success: function() {
    console.log('great success');
  },
  error: function(err) {
    console.log('omg', err);
  }
});

//if you dont care about the result, leave the callbacks out
peer.call('greet', {first: 'John', last: 'Carter'});
```

## Remove States

States can be removed by the owning Peer. States are also implicitly removed by
the Daemon if the network connection to a Peer breaks. In any case, all fetching
Peers will be informed.

```javascript
var foo = peer.state({
  path: 'foo',
  value: 123
});
foo.remove();
```

## Remove Methods

Methods can be removed by the owning Peer. Methods are also implicitly removed by
the Daemon if the network connection to a Peer breaks. In any case, all fetching
Peers will be informed.

```javascript
var greet = peer.method({
  path: 'greet',
  call: function() {}
});
greet.remove();
```


# Concepts

Jet is built around some simple but powerful concepts:

- Daemon
- Peer
- States
- Methods
- Fetching

Each concept has its own sub-chapter. But let's start with a short overview, about
how these concepts work together and are related to each other.

There are two kinds of "players" involved in a Jet setup:

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
but just has to know where it is running (Websocket URL). A Daemon instance may
run everywhere, except within a Browser, since Browsers cannot accept incoming
network connection  request.

The Daemon is able to route messages if required and keeps track
of all Methods and States (and its associated values). It is much like a
phonebook as you can (try) to set a State or call a Method based on its unique
Path. The Daemon also manages ownership of States and Methods as no Peer may
remove a State or Method which was not added by the very same Peer in the first
place. Also, States and Methods are automatically removed if a Peer closes the
connection to the Daemon. Fetch rules are stored and processed as well.

There are implementations available for Lua ([lua-jet](http://github.com/lipp/lua-jet))
and for Node.js ([node-jet](http://github.com/lipp/node-jet)).

## Peer

Peers bring the Jet bus to life. There can be any number of Peers and there are
no restrictions to their "location". Peers can run on the same machine as the
Daemon or on remote machines - even within Browsers.
As a network connection to the Daemon is established, a Peer instance starts to
exist. As soon as the connection to the Daemon is closed, the Daemon frees all
ressources which have been associated with the respective Peer.

Peers can do any combination of the following things:

- Add / Remove States and Methods
- Call Methods
- Set States
- Fetch / Unfetch States and Methods

A Peer may do all the above things, or just use a subset.
E.g. a Peer, which just logs everything what is going on to console could look
like this:

```javascript
// the first argument is the "special" empty fetch rule,
// which just matches everything.
peer.fetch({}, function(path, event, value){
  console.log('Jet Event@' + new Date(), path, event, value);
});
```

## States

A State is made up a **unique path** and a value. The value of a State can
be any non-function (JSON) type, such as primitive Types (Boolean, Numbers, Strings) or nested
Objects. Peers can add States by sending an __add__ Request to the Daemon,
providing the State path and value.

Everytime a Peer calls **set** with a corresponding path, the Daemon will route
the Request to the Peer, who added the State. The Request's `method` field will
have the specified path and the parameters are simply forwarded.

The Peer, who added the State, is supposed to handle this routed message and
- if it is not a Notification - send back a Response with a "truish" `result` or
an `error` field defined. In this case the Daemon will forward the Response to
the Peer who made the initial Request.

The Peer should issue a **change** Request / Notification, if the State value
changed. This may happen immediately after a forwarded **set** Request has been
handled, or spontaneously at any time for any reason. This makes the State
change public and visible for all Peers interested (by fetching).

A State change is always a replacement. The Daemon manages a State cache and
stores the current (last known) value of all States currently added. This
enables the Daemon to provide the correct and complete information for fetchers
who join the party lately.

A Peer who __set__s a State to a new value without getting an error Reponse
must not make any assumptions about the State's "real" new value! The one and
only truth about State's value can be queried through a __fetch__. State
providers are explicitly allowed to "adjust" the value contained in the forwarded
__set__ message. E.g. a Peer may decide to the accept a new value of `5.1` but
actually sets the value to `5.0`, still returning a "truish" result.

Further, a State can be removed at any time by sending a __remove__ message
to the Daemon.


## Methods

A Method is made up a **unique path**. Peers can add Methods by sending an __add__ Request to the Daemon,
providing the Method path.

Everytime a Peer calls **call** with a corresponding path, the Daemon will route
the Request to the Peer, who added the Method. The Request's `method` field will
have the specified path and the parameters are simply forwarded.

The Peer, who added the Method, is supposed to handle this routed message and
- if it is not a Notification - send back a Response with any `result` or an
`error` field defined. In this case the Daemon will forward the Response to
the Peer who made the initial Request.

Further, a Method can be removed at any time by sending a __remove__ message
to the Daemon.


## Fetch

Fetching is the one and only mean for Peers to get information (States/Methods)
from the Daemon. In particular, there is no "get" method to prevent Peers from
polling and thus keeping network traffic and cpu load for the Daemon low.
Fetching is by far the most complex command, but it offers great possibilities.
Peers can setup up as many Fetches as they want. The Peer must provide
(per Peer) unique fetch ids to allow forwarding the Fetch Notification to the
correct message handler.

The Daemon may generate Fetch Notifications if one of three events
happen:

- a State or Method is added
- a State or Method is removed
- a State's value has changed

If one of these events happen and the event is considered "relevant" based on the
Fetch rule, the Daemon creates a Fetch Notification and sends it to the respective
Peer.

Depending on the Fetch rule, there are two types of Fetch Notifications:

- non-sorted
- sorted

The simpler ones are the non-sorted Fetch Notifications. They always provide
the same set of information:

- **path**: Path of the State / Method
- **event**: Can be either "add", "remove" or "change" (only for States)
- **value**: The current State value (undefined for Methods)

The more complicated ones are the sorted Fetch Notifications. To save bandwidth
the Notification contains only the relevant differences to the previous
Notification:

- **n**: The number of States/Methods matching
- **changes**: An array of changed States/Methods with entries like this:
  - **path**: Path of the State / Method
  - **index**: The index (within the specified from-to range)
  - **value**: The current State value (undefined for Methods)

This allows to just handle States/Methods which have actually changed in some way
(index or value) without having to figure out the differences manually. E.g.
just updating the display parts (Dom-Nodes, etc.) for the stuff which actually changed,
instead of replacing all of it.

Fetch rules can be tuned very fine-grained and are able to match against
paths and/or values.

### Path based

Available path matching predicates are:

- equals
- equalsNot
- endsWith
- startsWith
- contains
- containsNot
- containsAllOf (Array)
- containsOneOf (Array)
- startsNotWith
- endsNotWith
- equalsOneOf (Array)
- equalsNotOneOf (Array)

These predicates can be combined. If all predicates specified are truthy, the
path is considered matching. The path matching process can be made
case-insensitive by setting the option "caseInsensitive" to true.

### Value based

It is possible to match against an entire State value by defining a `value` field
within the fetch rule. For State values of type `Object`, it is also possible to
match against one or more sub fields, which are specified by an "index string"
(see below).


Available value matching predicates are:

- lessThan
- greaterThan
- equals
- equalsNot
- isType

E.g. imagine a person State which may look
like this:

```javascript
{
  "age": 25,
  "name": {
    "first": "John",
    "last": "Mitchell"
  }
}
```

One could match against the age:

```javascript
{
  "method": "fetch",
  "params": {
    "id": "asstw6", // some fetch id
    "valueField" : {
      "age": { // field name
        "lessThan": 21 // predicate
      }
    }
  }
}
```

Or against the last name:

```javascript
{
  "method": "fetch",
  "params": {
    "id": "asst61", // some fetch id
    "valueField" : {
      "name.last": { // field name
        "equals": "Mitchell" // predicate
      }
    }
  }
}
```

### special "fetch all"

If you want to fetch everything, just leave out the `path` and `value` fields
entirely.


# Protocol

Jet heavily relies on [JSON-RPC 2.0](http://www.jsonrpc.org/specification)
semantics for all of its messages. There are some minor changes to the spec,
however. To be able to follow the protocol details, the reader must be familiar
with the (JSON-RPC 2.0) terms: Request, Response, Notification, Error Object and Batch.

As a reminder: Don't be confused by the term Notification. A Notification is
simply a Request without an __id__  specified, thus indicating no Response to
the Request is expected.

There are two categories of Requests to distinguish:

-  __Active__: The Peer sends a Request to the Daemon
-  __Passive__: The Daemon sends a Request to the Peer

The __Active__ Request always originate from Peers and are send to the Daemon.
As a result of adding States/Methods or Fetching, the Daemon may send
__Passive__ Requests to Peers. For instance: If a Peer __adds__ a
State, another __fetching__ Peer with matching fetch rules is informed by means
of a __Passive__ Request.

The __Passive__ Requests are:

- Fetch based messages
- (Routed) Requests to set (change) a State
- (Routed) Requests to call a Method

The `method` field of __Passive__ Requests are Peer defined
(via __add__ and __fetch__), whereas the method name field of __Active__
Requests is always either __add__, __remove__, __fetch__, __unfetch__, __set__,
__call__, __change__ or __config__.

Note: Batches can contain __Active__ and/or __Passive__ Requests.

## Example for Active Message

In this example a Peer fetches all persons (all states and methods, where the
path starts with "person"). The side-effect of this message is that, the Daemon
may send a __Passive__ message with `"method":"personFetcher"` to the peer
whenever appropriate.

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

## Example for Passive Message

This is a __Passive__ message which is issued based on the fetch rule defined
above. Note the method name, which has been specified earlier in the fetch call.

```javascript
{
  "method": "personFetcher",
  "params": {
    "path": "person/asdlkjasdk92",
    "value": {
      "name": "Paul",
      "age": 63
    },
    "event": "add"
  }
}
```

## Differences to JSON-RPC 2.0

In opposite to the [JSON-RPC 2.0 spec](http://www.jsonrpc.org/specification) the
`"jsonrpc": "2.0"` field is considered optional and can be simply ignored.

Further __Batch__ Messages are not subject of any order requirements are may be
processed and send at any time. This allows to minimize message framing overhead (e.g. Websockets).

## Messages

This chapter lists an overview of all __active__ messsages by example.

### add

Use the __add__ message for adding States or Methods to the Daemon. The Daemon
will route all set/call messages to the Peer, changing the method to the
respective path.

```javascript
// add a simple state
{
  "method": "add",
  "params": {
    "path": "foo/bar", //unique path
    "value": 123 // any non-function value
  }
  "id": 41
}
// add a more complicate state
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
// add a method (just leave-out the value)
{
  "method": "add",
  "params": {
    "path": "addNumbers"
  }
  // Note that this is a Notification as there is no "id"
}

```

### remove

Use the __remove__ message for removing States or Methods from the Daemon. The
Daemon will stop to route set/call messages for the respective path.

```javascript
// remove a method or state
{
  "method": "remove",
  "params": {
    "path": "addNumbers"
  }
  // Note that this is a Notification as there is no "id"
}

// remove a method or state
{
  "method": "remove",
  "params": {
    "path": "persons/xyz"
  }
  "id": "ajsykw"
}

```

### set

Use the __set__ message for (trying) to set a State to a new value. The Daemon
will route the Request to the responsible Peer. Accepting the new value is solely
up to the Peer. In case of an error-less dispatching / assigning, the Peer
posts a __change__ Notification, thus informing all fetching Peers about the
actual new value.

```javascript
// set a state to a new value
{
  "method": "set",
  "params": {
    "path": "foo/bar",
    "value": 920
  }
  // Note that this is a Notification as there is no "id"
}

// set a state to a new value
{
  "method": "set",
  "params": {
    "path": "foo/bar",
    "value": { // the value can be any non-function type
      age: 22,
      gender: "female"
    }
  }
  "id": "92s"
}

```

### call

Use the __call__ message for calling a Method. The Daemon
will route the Request to the responsible Peer.

```javascript
// call a method
{
  "method": "call",
  "params": {
    "path": "logStuff",
    "args": ["WARN",{
      "system": "cpu",
      "category": "epicFail"
    }]
  }
  // Note that this is a Notification as there is no "id"
}

// call a method
{
  "method": "call",
  "params": {
    "path": "addNumbers",
    "args": [1,2]
  }
  "id": "91s"
}

// call a method with object args
{
  "method": "call",
  "params": {
    "path": "createPerson",
    "args": {
      "name": "Jefferson",
      "age": 22,
      "hobbies": ["soccer","stamps"]
    }
  }
  "id": "911s"
}


```

### change

Use the __change__ message to make a State value change public.

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

### fetch

Use the __fetch__ message to create a new fetch rule. The Daemon sends
fetch Notifications with the specified fetch id as method, whenever appropriate.

```javascript
// some path based case insensitive fetch
{
  "method": "fetch",
  "params": {
    "id": "f1000",  // a peer defined fetch id
    "path": {
      "startsWith": "abc",
      "contains": "foo"
    },
    "caseInsensitive": true
  },
  "id": 3412
}

// some path based fetch (setup as Notification)
{
  "method": "fetch",
  "params": {
    "id": "__321_f",  // a peer defined fetch id
    "path": {
      "endsWith": "abc"
    }  
  }
  // no id -> Notification, no Response if fetch setup is ok
}

// some path / value based fetch
{
  "method": "fetch",
  "params": {
    "id": "f123",  // a peer defined fetch id
    "path": {
      "endsWith": "/temperature"
    },
    "value": {
      "lessThan": 7
    }
  }
  "id": "pasdp3"
}

// some path / valuefield based fetch
{
  "method": "fetch",
  "params": {
    "id": "f1pq23",  // a peer defined fetch id
    "path": {
      "startsWith": "persons/"
    },
    "valueField": {
      "age": {
        "greaterThan": 20
      },
      "name.first": {
        "equals": "Micheal"
      }
    }  
  }
  "id": "pa223"
}

// some path based sorted fetch
{
  "method": "fetch",
  "params": {
    "id": "f1pq23",  // a peer defined fetch id
    "path": {
      "startsWith": "persons/"
    },
    "sort": {
      "from": 20, // paginating
      "to": 40,
      "byPath": true // this is default
    }
  }
  "id": "pa223"
}

// some path based sorted (byValueField) fetch
{
  "method": "fetch",
  "params": {
    "id": "f1pq23",  // a peer defined fetch id
    "path": {
      "startsWith": "persons/"
    },
    "sort": {
      "from": 1, // paginating
      "to": 10,
      "byValueField": {
        "age": "number" // type is required for sorting: "number", "string" or "boolean"
      },
      "descending": true
    }
  }
  "id": "pa123"
}

```

# Live Examples

The following examples demonstrate Javascript peer API usage and (protocol) messages.
They actually run in your browser. You can edit them on [codepen](http://codepen.io) if desired.
Most of them use the Jet Daemon hosted at [nodejitsu](http://nodejitsu.com) with the
Daemon URL: `ws://jet.nodejitsu.com` (Port 80). The nodejitsu Jet Daemon is public
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

<div data-height="345" data-theme-id="0" data-slug-hash="GEyuq" data-default-tab="js" class='codepen'><pre><code>var connect = function(url) {
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
  e.preventDefault();
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});
</code></pre>
<p>See the Pen <a href='http://codepen.io/lipp/pen/GEyuq/'>Jet Connect</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>


## Add States

This example creates a Peer and adds two States with random. One, ignoring the
Daemon response, the other one logging the Daemon response to console. The message
traffic between the Daemon and the Peer is visible in the result window bottom.

The most relevant code snippet is:

```javascript
// create read-only state (no set callback provided)
// ignore the Daemon response by leaving out the callback object.
// the request is a notification.
peer.state({
  path: 'foo',
  value: 123
});

// create writable state,
// provide callback object with error and success handlers.
// the Daemon will send a response, as the request is not a
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

<div data-height="870" data-theme-id="0" data-slug-hash="kLlfB" data-default-tab="js" class='codepen'><pre><code>var connect = function(url) {
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
       value: 123
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
  e.preventDefault();
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});


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

## Fetch Simple

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
peer.fetch({path: {startsWith: 'random'}}, function(path, event, value){});
```

<div data-height="923" data-theme-id="0" data-slug-hash="Cglby" data-default-tab="js" class='codepen'><pre><code>var connect = function(url) {
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
       value: 123
     });

     peer.fetch({
       path: {
         startsWith: &#x27;random&#x27;
       }
     },function(path, event, value) {
       console.log(&#x27;fetch&#x27;, path, event, value);
     });
  } catch(err) {
    $(&#x27;#status&#x27;).text(&#x27;error &#x27; + err);
  }
};



// try to (re-)connect when button is clicked
$(&#x27;button&#x27;).click(function(e) {
  connect($(&#x27;input&#x27;).val());
  e.preventDefault();
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});


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
<p>See the Pen <a href='http://codepen.io/lipp/pen/Cglby/'>Jet Fetch State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>

## Change State

This examples creates a Peer, adds a "ticker" States with random name (so that at least
these State are available for fetching) and fetches everything with a path starting
with "ticker". Every second the ticker value is incremented by one.
The message traffic between the Daemon and the Peer is visible
in the result window bottom.

This is the most relevant snippet:

```javascript
var ticker = peer.state({
  path: 'ticker',
  value: 1
});

setInterval(function() {
  var old = ticker.value();
  ticker.value(++old); // post state change
}, 1000);
```

<div data-height="945" data-theme-id="0" data-slug-hash="eKBpG" data-default-tab="js" class='codepen'><pre><code>var tickTimer;
var connect = function(url) {
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
    var path = &#x27;ticker_#&#x27; + new Date().getTime();
    // add as notification
    var ticker = peer.state({
      path: path,
      value: 1
    });

    if (tickTimer) {
      clearInterval(tickTimer);
    }
    tickTimer = setInterval(function() {
      var old = ticker.value();
      ticker.value(++old);
    }, 1000);

    peer.fetch({
      path: {
        startsWith: &#x27;ticker_&#x27;
      }
    },function(path, event, value) {
      // console.log(&#x27;fetch&#x27;, path, event, value);
    });
  } catch(err) {
    $(&#x27;#status&#x27;).text(&#x27;error &#x27; + err);
  }
};



// try to (re-)connect when button is clicked
$(&#x27;button&#x27;).click(function(e) {
  connect($(&#x27;input&#x27;).val());
  e.preventDefault();
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});


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
<p>See the Pen <a href='http://codepen.io/lipp/pen/eKBpG/'>Jet Change State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>
