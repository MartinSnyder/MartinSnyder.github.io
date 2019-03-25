---
layout: page
title: http4s Chat Server
permalink: /chat
---

This is a pure functional chat application written using WebSockets and Scala/http4s. It was written to provide a non-trivial WebSocket example for http4s. Of note, it uses fs2 "Topics" and a publish/subscribe paradigm to bind the websockets of all connected users into a single conversation. The source code is [available on github][chat-repo].

<iframe width="100%" height="400px" src="https://http4s-chatserver.herokuapp.com/" frameborder="0" scrolling="no"></iframe>

[chat-repo]: https://github.com/MartinSnyder/http4s-chatserver
