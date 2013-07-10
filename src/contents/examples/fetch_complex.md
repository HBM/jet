---
title: Advanced Fetch
story: Fetches all people, who play guitar in their freetime. The filter system is inspired by common REST-APIs
---

```json
{
  "method": "fetch",
  "params": {
    "id": "f76",
    "matches": ["^persons/.*"],
    "equals": {
      "hobby": "guitar"
    }
  }
}```
