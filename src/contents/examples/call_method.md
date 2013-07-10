---
title: Call Method
story: Call "person/26317/run". Arguments can be any JSON Object or JSON Array. The request is routet to the peer who added the method. If an *id* is specfied, the response it routed back as well.
---

```json
{
  "method": "call",
  "params": {
    "path": "person/26317/run",
    "args": {
      "fast": true
    }
  },
  "id": 123918
}
```
