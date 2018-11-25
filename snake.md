---
layout: page
title: Snake
permalink: /snake/
---

When I first experimented with Elm, I wrote this simple implementation of the classic Snake game. The source code is available on [GitHub][snake-repo].

Use your mouse or finger to point the snake to point the snake in the direction to move. Gobble the green prizes to score points. The snake will grow as it eats and you will need to avoid your tail in order to keep going!

<pre>
<div id="elm" width="100%"></div>
<script type="text/javascript" src="elm-pep.js"></script>
<script type="text/javascript" src="snake.js"></script>
<script>
    var app = Elm.Main.init({
        node: document.getElementById('elm')
    });
</script>
</pre>

[snake-repo]: https://github.com/MartinSnyder/elm-snake