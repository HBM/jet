---
layout: default
title: Concepts
sections:
 - daemon.html
 - peer.html
 - state.html
 - method.html
 - fetching.html
order: 1
---

# Overview

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


# Daemon

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

# Peer

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

# States

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


# Methods

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


# Fetch

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

## Path based

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

## Value based

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

## special "fetch all"

If you want to fetch everything, just leave out the `path` and `value` fields
entirely.
