---
title: Fetch
template: command.jade
---

# Overview

Fetching is __the__ mechanism to efficiently get data from the Jet
Bus. It works similar to the publish-subscribe pattern. As this is
Jet's most powerful command, it needs some more explanation.

A Peer can have many different fetches at the same time. The Peer must supply fetch parameters, which specifiy which data you are
interested in and what 'method' the forwarded fetch notifications should
have ('fetchId'). The Daemon first scans its cache for matching elements
and publishes them to the Peer. Afterwards, whenever some Peer adds, removes
or changes elements on the Jet Bus, the Daemon applies the fetch rules
and forwards matching messages to the fetching Peer. There are two main categories of fetches: 

-    Non-sorting
-    Sorting

# Non-sorting Fetches

If the 'sort' field of the feth parameters is not specified, the fetch
is considered to be non-sorting:

```JSON
{
  "id": <id>, // optional
  "method": "fetch",
  "params": {
    "matches": [...], // optional, Array with match patterns 
    "unmatches": [...], // optional, Array with unmatch patterns
    "where": { // optional, filter by value
      ...
    }    
  }
}
```

For non-sorting fetches the Daemon
sends messages with this content:

```JSON
{
  "method": <fetchId> // as supplied 
  "params": {
    "event": <event> // 'add','change' or 'remove',
    "path": <elementPath>,
    "value": <elementValue> // optional, any type (also Object or Arrays)
  }
}
```

## Example 1: Fetch all stuff
