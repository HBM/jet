---
layout: default
title: Protocol
---

# Transport

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

# Messages

This chapter lists an overview of all __active__ messsages by example.

## add

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

## remove

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

## set

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

## call

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

## change

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

## fetch

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
