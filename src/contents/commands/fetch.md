---
title: Fetch
template: command.jade
---

Fetching is __the__ mechanism to efficiently get data from the Jet
Bus. It works similar to the publish-subscribe pattern, but also
generates notifications for "past" events using the cache. As this is
Jet's most powerful command, it needs some more explanation.

A Peer can have many different fetches at the same time. The Peer must supply fetch parameters, which specifiy which data you are
interested in and what 'method' the forwarded fetch notifications should
have ('fetchId'). The Daemon first scans its cache for matching elements
and publishes them to the Peer. Afterwards, whenever some Peer adds, removes
or changes elements on the Jet Bus, the Daemon applies the fetch rules
and forwards matching messages to the fetching Peer. There are two
main categories of fetches, sorting and non-sorting. 

A fetch message always looks like this:

```JSON
// Peer --> Daemon
{
  "id": <id>, // optional, if the peer wants confirmation for setup the fetch rule
  "method": "fetch",
  "params": {
    "id": <fetchId>, // This will be the 'method' field in fetch notifications
    "matches": [...], // optional, Array with match patterns 
    "unmatches": [...], // optional, Array with unmatch patterns
    "where": { // optional, filter by value
      ...
    },
    "sort": { // optional sorting criteria
    }    
  }
}
```

The forwarding rule created by the 'fetch' call can be destroyed by
calling 'unfetch' with the respective 'fetchId'.

```JSON
// Peer --> Daemon
{
  "id": <id>, // optional, if the peer wants confirmation for cleanup the fetch rule
  "method": "unfetch",
  "params": {
    "id": <fetchId> // Must have been previously specified in 'fetch' call
  }
}
```

## Non-sorting Fetches

If the 'sort' field of the fetch parameters is not specified, the fetch
is considered to be non-sorting. For non-sorting fetches the Daemon
sends messages with this content:

```JSON
// Daemon --> Peer
{
  "method": <fetchId> // as supplied by the Peer (params.id)
  "params": {
    "event": <event> // 'add','change' or 'remove',
    "path": <elementPath>,
    "value": <elementValue> // optional, any type (also Object or Arrays)
  }
}
```

## Sorting Fetches

If the 'sort' field of the fetch parameters is specified, the fetch is
considered sorting. For sorting fetches the Daemon send messages with
this content:

```JSON
// Daemon --> Peer
{
  "method": <fetchId> // as supplied by the Peer (params.id)
  "params": {
    "n": <someNumber> // The number of currently matching elements in [to:from]
    "changes": [ // an Array containing the changes elements
      {
        "index": <indexInRange>, // the position/index in [to:from]
        "path": <elementPath>,
        "value": <elementValue> // optional, any type (also Object orArrays)
      },
  }
}
```

### Example 1: Fetch all stuff

Define the wildcard pattern '.*' in the 'matches' fetch parameter to
get all states and methods available at the Jet Bus. This is pretty
much like subscribing to anything.

```JSON
// Peer --> Daemon
{
  "method": "fetch",
  "params": {
    "id": "fetchAllStuff",
    "matches": [".*"]
  }
}
```

### Example 2: Fetch all persons and cars

Use multiple 'matches' Array entries plus the wildcard pattern '.*'.

```JSON
// Peer --> Daemon
{
  "method": "fetch",
  "params": {
    "id": "fetchPersonsAndCars",
    "matches": ["persons/.*","cars/.*"]
  }
}
```

### Example 3: Fetch all persons but not the person with id 25162

Use 'unmatches' Array to exclude entries which are matched by
'matches' pattern. 

```JSON
// Peer --> Daemon
{
  "method": "fetch",
  "params": {
    "id": "fetchPersonsButNot25162",
    "matches": ["persons/.*"],
    "unmatches": ["person/25162"]
  }
}
```

### Fetch Notification Scenario

The Daemon will scan its internal cache and forward all matching
entries. Even if the elements already have been added, the event is
set to 'add'. This is to make virtually no difference for the fetcher if the
state/method was already added or not.

```JSON
// Daemon --> Peer

// message 1, add state foo
{
  "method": "f521", // the fetchId specified by the Peer
  "params": {
    "event": "add",
    "path": "foo",
    "value": "bar"
  }
}

// message 2, add state persons/fatty
{
  "method": "f521", // the fetchId specified by the Peer
  "params": {
    "event": "add",
    "path": "persons/fatty",
    "value": {
      "age": 35,
      "name": "John Foo"
    }
  }
}

// message 3, add state persons/slowdude
{
  "method": "f521", // the fetchId specified by the Peer
  "params": {
    "event": "add",
    "path": "persons/slowdude",
    "value": false
  }
}

// message 3, add method reboot
{
  "method": "f521", // the fetchId specified by the Peer
  "params": {
    "event": "add",
    "path": "reboot"
  }
}
```

If (later on) another state is added, removed or changed, this is also forwarded:

```JSON
// Daemon --> Peer

// message 4, add state brother
{
  "method": "f521", // the fetchId specified by the Peer
  "params": {
    "event": "add",
    "path": "brother",
    "value": "hi there!"
  }
}

// message 5, remove state foo
{
  "method": "f521", // the fetchId specified by the Peer
  "params": {
    "event": "remove",
    "path": "foo",
    "value": "hi there!" // the last known value
  }
}

// message 5, change persons/fatty
{
  "method": "f521", // the fetchId specified by the Peer
  "params": {
    "event": "change",
    "path": "persons/fatty",
    "value": {
      "age": 36,
      "name": "John Foo"
    }   
  }
}
```


