---
layout: default
title: Examples
---

The following examples demonstrate Javascript peer API usage and (protocol) messages.
They actually run in your browser. You can edit them on [codepen](http://codepen.io) if desired.
Most of them use the Jet Daemon hosted at [nodejitsu](http://nodejitsu.com) with the
Daemon URL: `ws://jet.nodejitsu.com` (Port 80). The nodejitsu Jet Daemon is public
and you may notice activity of other peers "playing" with it.

# Connect

This example creates a peer and reports back the connection status.

The most relevant code snippet is:

```javascript
var peer = new jet.Peer({
      url: 'ws://jet.nodejitsu.com:80',
      onOpen: function() {}
    });
```

<div data-height="345" data-theme-id="0" data-slug-hash="GEyuq" data-default-tab="js" class='codepen'><pre><code>var connect = function(url) {
   try {
    $(&#x27;#status&#x27;).text(&#x27;disconnected&#x27;);
    // create a Jet Peer, providing the Jet (Daemon) Websocket URL
    var peer = new jet.Peer({
      url: url,
      onOpen: function() {
        $(&#x27;#status&#x27;).text(&#x27;connected&#x27;);
      },
    });
  } catch(err) {
    $(&#x27;#status&#x27;).text(&#x27;error &#x27; + err);
  }
};

// try to (re-)connect when button is clicked
$(&#x27;button&#x27;).click(function(e) {
  connect($(&#x27;input&#x27;).val());
  e.preventDefault();
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});
</code></pre>
<p>See the Pen <a href='http://codepen.io/lipp/pen/GEyuq/'>Jet Connect</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>


# Add States

This example creates a Peer and adds two States with random. One, ignoring the
Daemon response, the other one logging the Daemon response to console. The message
traffic between the Daemon and the Peer is visible in the result window bottom.

The most relevant code snippet is:

```javascript
// create read-only state (no set callback provided)
// ignore the Daemon response by leaving out the callback object.
// the request is a notification.
peer.state({
  path: 'foo',
  value: 123
});

// create writable state,
// provide callback object with error and success handlers.
// the Daemon will send a response, as the request is not a
// notification.
var bar = 'hello';
peer.state({
  path: 'bar',
  value: bar,
  set: function(newVal) {
      bar = newVal;
  }
  }, {
  success: function() {},
  error: function(err) {}
});
```

<div data-height="870" data-theme-id="0" data-slug-hash="kLlfB" data-default-tab="js" class='codepen'><pre><code>var connect = function(url) {
   try {
    $(&#x27;#status&#x27;).text(&#x27;disconnected&#x27;);
    // create a Jet Peer, providing the Jet (Daemon) Websocket URL
    var peer = new jet.Peer({
      url: url,
      onOpen: function() {
        $(&#x27;#status&#x27;).text(&#x27;connected&#x27;);
      },
      onSend: function(message) {
        addLogEntry(&#x27;Out&#x27;,message);
      },
      onReceive: function(message) {
        addLogEntry(&#x27;In&#x27;,message);
      }
    });
     var random = &#x27;random&#x27; + new Date().getTime();
     // add as notification
     peer.state({
       path: random,
       value: 123
     });

     random = &#x27;random_2_&#x27; + new Date().getTime();
     // add as request
     peer.state({
       path: random,
       value: 123
     },{
       success: function() {
         console.log(&#x27;ok&#x27;);
       },
       error: function(err) {
         console.log(err);
       }
     });
  } catch(err) {
    $(&#x27;#status&#x27;).text(&#x27;error &#x27; + err);
  }
};



// try to (re-)connect when button is clicked
$(&#x27;button&#x27;).click(function(e) {
  connect($(&#x27;input&#x27;).val());
  e.preventDefault();
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});


var off;
var addLogEntry = function(direction, message) {
  var now = new Date().getTime();
  if (!off) {
    off = now;
  }
  var tr = $(&#x27;&lt;tr&gt;&lt;/tr&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + (now-off) + &#x27;&lt;/td&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + direction + &#x27;&lt;/td&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + message + &#x27;&lt;/td&gt;&#x27;);
  $(&#x27;#log tbody&#x27;).append(tr);
};

</code></pre>
<p>See the Pen <a href='http://codepen.io/lipp/pen/kLlfB/'>Jet Add State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>

# Fetch Simple

This examples creates a Peer, adds two States with random name ("random...")
and fetches everything with a path starting with "random".
The message traffic between the Daemon and the Peer is visible in the result
window bottom. Compare the (fetch) `id` of the `fetch` message with the incoming
messages' `method` field. The incoming messages can be considered __Passive__ as
they are Requests targeted at the Peer, embedding a Peer defined `method` field
value.

This is the most relevant snippet:

```javascript
// fetch and provide: 1) fetch rule, 2) callback for fetching
peer.fetch({path: {startsWith: 'random'}}, function(path, event, value){});
```

<div data-height="923" data-theme-id="0" data-slug-hash="Cglby" data-default-tab="js" class='codepen'><pre><code>var connect = function(url) {
   try {
    $(&#x27;#status&#x27;).text(&#x27;disconnected&#x27;);
    // create a Jet Peer, providing the Jet (Daemon) Websocket URL
    var peer = new jet.Peer({
      url: url,
      onOpen: function() {
        $(&#x27;#status&#x27;).text(&#x27;connected&#x27;);
      },
      onSend: function(message) {
        addLogEntry(&#x27;Out&#x27;,message);
      },
      onReceive: function(message) {
        addLogEntry(&#x27;In&#x27;,message);
      }
    });
     var random = &#x27;random&#x27; + new Date().getTime();
     // add as notification
     peer.state({
       path: random,
       value: 123
     });

     random = &#x27;random_2_&#x27; + new Date().getTime();
     // add as request
     peer.state({
       path: random,
       value: 123
     });

     peer.fetch({
       path: {
         startsWith: &#x27;random&#x27;
       }
     },function(path, event, value) {
       console.log(&#x27;fetch&#x27;, path, event, value);
     });
  } catch(err) {
    $(&#x27;#status&#x27;).text(&#x27;error &#x27; + err);
  }
};



// try to (re-)connect when button is clicked
$(&#x27;button&#x27;).click(function(e) {
  connect($(&#x27;input&#x27;).val());
  e.preventDefault();
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});


var off;
var addLogEntry = function(direction, message) {
  var now = new Date().getTime();
  if (!off) {
    off = now;
  }
  var tr = $(&#x27;&lt;tr&gt;&lt;/tr&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + (now-off) + &#x27;&lt;/td&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + direction + &#x27;&lt;/td&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + message + &#x27;&lt;/td&gt;&#x27;);
  $(&#x27;#log tbody&#x27;).append(tr);
};

</code></pre>
<p>See the Pen <a href='http://codepen.io/lipp/pen/Cglby/'>Jet Fetch State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>

# Change State

This examples creates a Peer, adds a "ticker" States with random name (so that at least
these State are available for fetching) and fetches everything with a path starting
with "ticker". Every second the ticker value is incremented by one.
The message traffic between the Daemon and the Peer is visible
in the result window bottom.

This is the most relevant snippet:

```javascript
var ticker = peer.state({
  path: 'ticker',
  value: 1
});

setInterval(function() {
  var old = ticker.value();
  ticker.value(++old); // post state change
}, 1000);
```

<div data-height="945" data-theme-id="0" data-slug-hash="eKBpG" data-default-tab="js" class='codepen'><pre><code>var tickTimer;
var connect = function(url) {
  try {
    $(&#x27;#status&#x27;).text(&#x27;disconnected&#x27;);
    // create a Jet Peer, providing the Jet (Daemon) Websocket URL
    var peer = new jet.Peer({
      url: url,
      onOpen: function() {
        $(&#x27;#status&#x27;).text(&#x27;connected&#x27;);
      },
      onSend: function(message) {
        addLogEntry(&#x27;Out&#x27;,message);
      },
      onReceive: function(message) {
        addLogEntry(&#x27;In&#x27;,message);
      }
    });
    var path = &#x27;ticker_#&#x27; + new Date().getTime();
    // add as notification
    var ticker = peer.state({
      path: path,
      value: 1
    });

    if (tickTimer) {
      clearInterval(tickTimer);
    }
    tickTimer = setInterval(function() {
      var old = ticker.value();
      ticker.value(++old);
    }, 1000);

    peer.fetch({
      path: {
        startsWith: &#x27;ticker_&#x27;
      }
    },function(path, event, value) {
      // console.log(&#x27;fetch&#x27;, path, event, value);
    });
  } catch(err) {
    $(&#x27;#status&#x27;).text(&#x27;error &#x27; + err);
  }
};



// try to (re-)connect when button is clicked
$(&#x27;button&#x27;).click(function(e) {
  connect($(&#x27;input&#x27;).val());
  e.preventDefault();
});

// try to (re-)connect when input field changed
$(&#x27;input&#x27;).change(function(e) {
  connect($(&#x27;input&#x27;).val());
});


var off;
var addLogEntry = function(direction, message) {
  var now = new Date().getTime();
  if (!off) {
    off = now;
  }
  var tr = $(&#x27;&lt;tr&gt;&lt;/tr&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + (now-off) + &#x27;&lt;/td&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + direction + &#x27;&lt;/td&gt;&#x27;);
  tr.append(&#x27;&lt;td&gt;&#x27; + message + &#x27;&lt;/td&gt;&#x27;);
  $(&#x27;#log tbody&#x27;).append(tr);
};

</code></pre>
<p>See the Pen <a href='http://codepen.io/lipp/pen/eKBpG/'>Jet Change State</a> by Gerhard Preuss (<a href='http://codepen.io/lipp'>@lipp</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
</div><script async src="//codepen.io/assets/embed/ei.js"></script>
