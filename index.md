---
layout: default
title: Jet
title_hidden: true
sections:
 - hero.html
 - all-in.html
 - free.html
 - bus.html
 - todo.html
order: 0
---

<div id="hero">
  <div id="logo">
    <a href="/index.html">
      <h1 id="brand">Jet</h1>
    </a>
    <div>
      <h2 id="tagline">Lightweight, Realtime Message Bus for the Web</h2>
    </div>
  </div>
</div>

    <div class="highlights">
      <ul>
        <li>
          <img src="/images/ic_search_48px.svg"/>
          <div>
          <h3>Filter & Sorting</h3>
          <p>Powerful filter and sorting mechanisms help you getting only the relevant data,
            pagination included.
            Let the Jet Daemon do the heavy work and save bandwidth.
          </p>
        </div>
        </li>
                <li>
          <img src="/images/ic_public_48px.svg"/>
          <div>
          <h3>Web Native</h3>
          <p>Your Data is JSON. The API is JSON. Everything is traveling over WebSockets.
          </p>
        </div>
        </li>
        <li>
          <img src="/images/ic_sort_48px.svg"/>
          <div>
          <h3>Easy Integration</h3>
          <p>Jet Daemon can be very easily integrated into your server environment.
            <i>Any</i> Webserver is suitable, since the WebSocket communication optionally may use another port
            than your webserver.
          </p>
        </div>
        </li>
        <li>
          <img src="/images/ic_autorenew_48px.svg"/>
          <div>
          <h3>Sync</h3>
          <p>"Dont call us. We call you!". Strictly publish/subscribe based mechanisms at work.
            Stay in sync when states or methods gets added, removed or changed.
          </p>
        </div>
        </li>
        <li>
          <img src="/images/ic_attachment_48px.svg"/>
          <div>
          <h3>Small</h3>
          <p>Powerfull things kept simple. Complete Javascript for Browser implementation
            has less than 700 lines of code and is less than 9k in production.
          </p>
        </div>
        </li>
        <li>
          <img src="/images/ic_memory_48px.svg"/>
          <div>
          <h3>SoC</h3>
          <p>Runs very nice on RaspberryPi-class SoCs. Jet was designed
            as a realtime configration and status Bus for embedded devices.
          </p>
        </div>
        </li>

      </ul>
    </div>

        <div class="downloads">
          <ul>
            <li>
          <a href="http://github.com/lipp/jet-js">
            <img src="images/HTML5_Badge.svg" alt="HTML5 Powered with Connectivity / Realtime, and Device Access" title="HTML5 Powered with Connectivity / Realtime, and Device Access">
          </a>
        </li>
          <li>
          <a href="http://github.com/lipp/node-jet">
            <img src="images/Node.js_Logo.svg" alt="Node.js" title="node-jet"></img>
          </a>
        </li>
          <li>
          <a href="http://github.com/lipp/lua-jet">
            <img src="images/Lua-logo-nolabel.svg" alt="Lua" title="lua-jet"></img>
          </a>
        </li>

        </ul>
    </div>

    <iframe id="todo" src="/todo/index.html"></iframe>
