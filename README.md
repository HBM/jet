# About

Jet is a connection-oriented protocol on top of JSON-RPC. By defining messages and behaviour, it allows building efficient, distributed, hot-plugable, asynchronous, event driven, all-buzzing applications in any language. 

This package provides the Jet protocol, a daemon (which happens to be written in Lua) and Lua peer bindings.
Jet prescribes simple concepts to allow building very complex distributed applications. It tries to be simpler and more transparent than DBus, while at the same time being more flexible.

## Installation

    $ git clone https://github.com/lipp/jet.git
    $ cd jet
    $ sudo luarocks make rockspecs/jet-scm-1.rockspec

## Build status

[![Build Status](https://travis-ci.org/lipp/jet.png?branch=master)](https://travis-ci.org/lipp/jet/builds)

# Concepts

Nodes, Methods and States  are the only conceptual elements viewed from the "outside". In short:

- Each Peer has one connection to the daemon and no direct connections to other Peers
- The daemon routes and repacks Messages between peers
- Nodes are managed by the daemon and can come and go at any time
- Peers can add or remove Methods and States at any time
- Their constraints (aka "schema") can change at any time
- States can be any JSON and change their value at any time
- The daemon caches the hierarchy and the states current values
- Current values and future changes of any kind can be fetched (aka "subscribed")

## Nodes

Nodes are required for structuring the Jet bus created by peers. They have at least one child. Childs can be Nodes, Methods or States. Peers cannot create Nodes. Instead they are created / deleted by the daemon if neccessary.

## Methods

Methods MAY be called by sending a __call__ Message to the daemon. The signature / parameters is not defined but MAY be descriped by a schema. Methods MAY be added by Peers by sending an apropriate __add__ Message to the daemon. After adding a Method, a Peer gets routed __call__ Request to this methods. The Peer SHOULD answer these Requests. The Peer MAY change a Method's schema at any time via sending a __post__ Message to the daemon. The Peer MAY remove the Method at any time and will not get any forwarded Request made to that Method.

## States

The value of states MUST be JSON. States MAY be requested to change by sending a __set__ Message to the daemon. They MAY be added by peers by sending an apropriate __add__ Message to the daemon. After adding a State, a Peer gets routed __set__ Request to this methods. The Peer SHOULD answer these Requests. The Peer MAY change a State's value or schema at any time via sending a __post__ Message to the daemon. The Peer MAY remove the State at any time and will not get any forwarded Request made to that State.

# Players (Processes)

## Daemon

The MUST be exactly one daemon process running, listening on port 33260 for incoming connections.

The jet daemon has three main jobs:

- Managing nodes and leaves
- Routing messages to the right peers
- Distributing Posts (aka "fetch/unfetch" or "publish/subscribe")

### Nodes and leaves (hierarchy)

Jet is built to allow distributing functionality between processes as granular and easy as possible.Thus peers are only responsible for their Jet leaves (methods and/or states). Nodes are managed by the daemon and created or deleted when appriopriate. In opposite to DBus, related functionality can be provided by different processes (aka peers, aka services). E.g. peer 1 could provide a state 'foo.bar.status' and peer 2 could provide a method 'foo.bar.fly'. A Jet peer using either of the functionality (__set__int a state or __call__ing a method) does not see peer boundaries though.

Therefor it is __easy__ to change service distribution, since other peers will not even notice a change.

The daemon manages creation and deletion of node. As soon as a new node is required, it is created (and posted/published to all peer interested). The other way around, if a node has no longer any child, it is deleted (and posted/published to all peer interested).

### Routing

Calls __set__

# What you get

- distributing states and methods with __arbitrary hierarchy__ among __arbitrary processes__
- events for creation, deletion and change of nodes, states and methods
- automatic node management
- hot-plugable infrastructure
- simple yet powerful concepts

# What you need

To implement your own daemon or client binding you will need:

- Sockets
- JSON parser
- Some event-loop or threads


# Definitions

## Leaves

Methods and States are considered leave elements, since they can not have any childs. All parent elements MUST be Nodes.

## Path

A path describes the hierarchical relation of Nodes, Methods and States and uniquely identify them. Paths are Strings, which use C Identifiers as Node and Leave names and a forward slash ('/') for sepeating Nodes and Leaves. E.g.

```Javascript
"foo/bar/test"
```