---
layout: default
title: Jet
---

# Jet in 5 Minutes

## Create a Peer

To interact with a Jet Daemon you need a Peer.

```javascript
var jet = require('jet');
var peer = new jet.Peer();
```

## Add States and Methods

Peers can create States or Methods which must have a unique path. 
Other peers can __set__ States to new values or __call__ Methods. 
The value of a State can be of any (non-function) type.

```javascript
// add a state
peer.state({
  path: 'foo',
  value: 1234,
  set: function (newVal) {
    console.log('foo is now set to',newVal);
  }
});

// add a method
peer.method({
  path: 'addNumbers',
  call: function (a,b) {
    return a+b;
  }
});
```

## Fetch States and Methods

Other Peers can fetch States and Methods. Fetch expressions can be based on paths and values. If desired, the
fetched elements can also be delivered sorted.

```javascript
peer.fetch({
  path: {
    equalsOneOf: ['foo','addNumbers']
  }, function (notification) {
    console.log(notification);
	// will show:
	// {"path": "foo", "event": "add", "value": 1234}
	// {"path": "addNumbers", "event": "add"}
  }
});

```

## Set States and Call Methods

### Test Inset

Hello

<div id="home">
  <h1>Blog Posts 2</h1>
  <ul class="posts">
    {% for post in site.posts %}
      <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
</div>
