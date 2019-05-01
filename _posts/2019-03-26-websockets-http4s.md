---
layout: post
title: WebSockets, http4s, and fs2
---

When I decided to experiment with functional streams and worked to build a WebSocket application using [http4s][http4s]. I chose that platform because [TypeLevel][typelevel] has a lot of velocity right now, especially with [fs2][fs2] (Functional Streams for Scala) in the [cats][cats] ecosystem.

I struggled at first, because the WebSocket examples were too simple for my use case. Fortunately, the contributors to http4s are very responsive on their [Gitter Channel][gitter] and quickly pointed me in the right direction.

To thank them, and to help the next round of adopters, I wrote a [detailed example application][chat] and shared the [code on GitHub][repo-chat].

[http4s]: https://http4s.org/
[typelevel]: https://typelevel.org
[fs2]: https://github.com/functional-streams-for-scala
[cats]: https://typelevel.org/cats/
[gitter]: https://gitter.im/http4s/http4s
[repo-chat]: https://github.com/MartinSnyder/http4s-chatserver

[chat]: {% link projects/chat.md %}
