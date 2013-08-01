---
template: architecture.jade
---

# Overview

A Jet Bus is made of one Daemon and an arbitrary number of Peers. The Peers
communicate with the Daemon through network sockets. Thus Peers may run on the same
machine or another machine as long as a network connection between
each Peer and the Daemon is available. Concerning the Jet Bus, Peers only communicate with the
Daemon and never (directly) with the Peers. The distribution /
deployment of the Peers is not visible from the outside, thus allowing
arbitrary service distribution.

# Daemon

A single Jet Bus must have one running Jet Daemon. At the moment there are two Daemon
implementations available:

 *    [lua-jet](http://github.com/lipp/lua-jet) for [Lua](http://www.lua.org)
 *    [node-jet](http://github.com/lipp/node-jet) for [Node.js](http://www.nodejs.org)

A Daemon serves these purposes:

 *     Routing Messages
 *     Caching Values
 *     Filtering
 *     Sorting

It is listening on two ports (one for Websockets and one for "raw" TCP
protocol) for new Peers.

# Peer

A single Jet Bus may have any number of Peers. By connecting to a daemon (either through
Websockets or through "raw" TCP) a Peer starts to exist. By using
Websockets, even a Browser may become a full featured Peer. A Peer may do
any of the following things:

 *    Call Methods
 *    Set States
 *    Fetch Values
 *    Add/Remove Methods
 *    Add/Remove States

If the socket connection closes, all resources (routes, fetch rules,
etc) associated with this Peer instance at the Daemon are freed. 

# Communication

There are two Message protocols a Jet Daemon implementation must
support (on different ports): 

 *    [Websockets](http://tools.ietf.org/html/rfc6455)
 *    "Raw" TCP Messages

The [Websocket Protocol](http://tools.ietf.org/html/rfc6455) is (very)
well defined and language bindings are broadly available. The "Raw"
TCP Message Protocol is a naive message framing protocol to allow more
primitive and faster Peer implementations.

## "Raw" TCP Format

```
Len := 32bit Unsigned Int Big Endian
Content := Bytes
RawMessage := Len Content
```