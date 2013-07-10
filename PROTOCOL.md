# Conventions

As described [here](http://www.jsonrpc.org/specification#conventions)

A __peer__ is a process which interacts with the jet daemon.

# Message wireformat

32bit big-endian denoting the payload length, followed by the message payload, which MUST be JSON.

# Message types

## Request Object

A message with payload as defined [here](http://www.jsonrpc.org/specification#request_object).

## Response Object

A message with payload as defined [here](http://www.jsonrpc.org/specification#response_object).

## Notification Object

A message with payload as defined [here](http://www.jsonrpc.org/specification#notification).

## Batch

A message with a JSON Array as payload, which contains an arbitrary sequence of Response/Request/Notification Objects.


# Jet Services

The jet daemon provides several services which may be used through:
- sending a __Request Object__; the daemon will process the message and reply with a corresponding __Response Object__
- sending a __Notification Object__; the daemon will process the message but will __NOT__ reply

To execute a jet service, set the Request's/Notification Message's field __method__ to the service name and set the field __params__ as required by the service.

## add

Adds an element to the jet bus. The element may have a __value__ field. After adding, the peer gets forwarded all __set__ (state) and __call__ (method) Requests / Notifications with a matching __path__ field. The peer is responsible for processing the message and - if the service message is a Request - reply with a Response Object. If there is already a state/method added with equal __path__, the peer will not get __set__ and __call__ Requests routed and - if the service message is a Request - an appropriate error Resonse is returned.

### path (String)

The element's path, '/' (forward-slash) for delimiting nodes.

### value (optional, any type)

Describes the state's current value. This will be cached by the daemon.

### Example 
```Javascript
{
        "id": 7384, // optional
        "method":"add",
        "params":{
                "path":"foo/bar/state",
                "value": 123,
        }  
        
}
```

### Sideeffects

The daemon MUST post a Notification for the newly added element, like:
```Javascript
{
        "method":...
        "params":{
                "event":"add",
                "path": "foo/bar/state",,
                "value": 123
        }
}
```

### Forwards

Note that the forwarded message may be a Notification (no 'id'). In this case, a Response MUST NOT be send.

#### set
__set__ service request will be forwarded as follows (imagine a state 'a/b/c' has been added):
```Javascript
var set_msg = 
{
        "id":231, // optional
        "method":"set",
        "params":{
                "path":"a/b/c",
                "value": 123
        }
}
var set_forward = 
{
        "id": 231, // optional
        "method":"a/b/c",
        "params":{
                "value":123
        }
}
```

__call__ service request will be forwarded as follows (imagine a state 'a/b/d' has been added):
```Javascript
var call_msg = 
{
        "id": 231, // optional
        "method":"call",
        "params":{
                "path":"a/b/d",
                "args": [1,2]
        }
}
var call_forward = 
{
        "id": 231, // optional
        "method":"a/b/c",
        "params":[1,2]
}
```


## remove

Removes the element with the specified path. __call__ and __set__ messages will no longer be forwarded.

### path (String)

The element's path, '/' (forward-slash) for delimiting nodes.

### Example 
```Javascript
{
        "id": 7384, // optional
        "method":"remove",
        "params":{
                "path":"foo/bar/state"
        }
}
```

### Sideeffects

The daenon MUST post a Notification for the newly removed element, like:
```Javascript
{
        "method":...
        "params":{
                "event":"remove",
                "path": "foo/bar/state",
                "value": 123
        }
}
```

## call

Calls a previously a added method with the specified arguments. The method may have been registered by the calling process or any other process connected to jet. 

The jet daemon will try to forward the request to a peer.

### path (String)

The element's path.

### args 

The arguments which will be forwarded to the peer as Array.

### Example 
```Javascript
var call_msg = {
        "id": 7384, // optional
        "method":"call",
        "params":{
                "path":"foo/bar/sum",
                "args":[1,2,3]
       }
}

var call_forward_msg = {
        "id": 7384, // optional
        "method":"foo/bar/sum",
        "params":[1,2,3]
}
```

## set [path,value]

Sets the element's value.

The jet daemon will try to forward the request to a peer.

### path (String)

The element's path, '/' (forward-slash) for delimiting nodes.

### value (any JSON)

The desired value's type cannot be determined previously and will be accepted / refused be the processing peer on message arrival.

### Example 
```Javascript
{
        "id": 7384, // optional
        "method":"set",
        "params":{
                "path":"foo/bar/state",
                "value": {"a":56.2,"b":33}
        }               
}
```

### Sideeffects

Even if the service returns with no error, the actual state's value might differ from the requested one. The peer SHOULD listen to state changes via fetch to retrieve the 'real' new value.

## post

Posts the specified notification to all 'fetchers', which match on path's value.

### path (String)

The element's path, '/' (forward-slash) for delimiting nodes.

### notification (Object)

MUST contain the fields:

- __event__: MUST be 'change' for now
- __path__: The effected elements path
- __data__: An Object containing all elements which have changed (up to now only supports 'value' and 'schema')


## fetch

### id (String)

An id which will be delivered back to the peer, whenever a notification is being posted, where the path is matched by matcher.

### match (Array)

An Array with Lua patterns. Length must be > 0.

### unmatch (Array,optional)

An Array with Lua patterns.

### Example 
```Javascript
{
        "id": 7384, // optional
        "method":"fetch",
        "params":{
           "id": "all_stuff",
           "match":[".*"]
        }   
}
```

```Javascript
{
        "id": 7384, // optional
        "method":"fetch",
        "params":{
                "id":"fency_stuff",
                "match":["a/b/.*","a/c/.*"],
                "unmatch": ["a/b/c/e"]
        }
}
```

### Sideeffects

The peer gets 'add' notifications for all existing matched nodes, states and methods as if they were just added. All future matched posted notifications are forwarded as well.

### Forwards

Imagine a state 'change' post, which is matched be the fetcher with id = 'foo'.
```Javascript
var incoming_post = 
{
        "method":"post",
        "params":{
                "path":"a/b/c",
                "event": "change",
                "data": {
                   "value": 2231
                }       
        }
}
var post_forward = 
{
        "method":"all_stuff",
        "params": {
                "path":"a/b/c",
                "event": "change",
                "data": {
                   "value": 2231
                }       
        }
}
```

## unfetch [id]









