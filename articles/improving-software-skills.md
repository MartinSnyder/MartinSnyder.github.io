---
layout: page
title: Improving Skills as a Software Practitioner
---

One topic that comes up in a lot of my conversations is what techniques I recommend to improve as a software practitioner. The most common scenarios involve current students, early career job seekers, and early career professionals trying to "make the jump" to the higher levels, but there are others as well. My advice for all populations is the same. In every case that I've seen, the best practitioners write a lot of code, and are always learning. And, while there is much to learn and gain in academic environments, that's just not where the best skill sets are identified or refined.

I think this is because Programming, Software Engineering, and Compute Science are distinct fields and while they often overlap in practice, mastery of any one of them does not imply mastery of the others. This is one reason why people with similar experience levels and skills can very so much in overall effectiveness, and it's also a factor in why many consider programming to be more of a [craft][craft] than an engineering discipline. These observations in part fueled my [Musings on College][college], though I feel that article applies to all academic fields and not just the computer-related ones.

This comes up a lot in job seeking because while there are many elements that contribute to one being an impactful software practitioner, the raw programming skills are the easiest to assess and so there is an outsized focus on technical assessments during software interviews. I'm not aware of any field that has spawned a near-infinite number of [books on preparing for interviews][books] the same way that the software industry has.

So approach is to build something meaningful under the guidance of a master, akin to an apprenticeship. It can be a challenge to choose a project that is large enough to be meaningful but not so large as to be overwhelming. For people that have a novel project in mind, it can be an even greater challenge to focus on "making the software better" over "making the application better." For some time, I have thought the best approach to be to pick a popular, but simple website and recreate it. A great example target for this is [Twitter, or X or whatever][twitter]. Building a twitter competitor is hard, but building a website that does what twitter does is not particularly hard. The challenging things about twitter have to do with volume/scalability, community/moderation, and monetization. Another good target in this category is [Trello][trello], which implements flexible [kanban][kanban] boards.

Both sites (and many others) have the following characteristics that I provide great context for learning:
* Simple data models
* Manageable lists of functional requirements
* Data storage requirements which can be met using almost any technology
* User interface requirements which can be implemented using almost any technique and technology
* Could be implemented as a web application, desktop application, command-line interface (CLI) or raw API
* A reference implementation that can be studied


[craft]: https://en.wikipedia.org/wiki/Software_craftsmanship
[garden]: https://blog.codinghorror.com/tending-your-software-garden
[books]: https://www.google.com/search?q=technical+interviewing+books
[twitter]: https://x.com
[trello]: https://trello.com/
[kanban]: https://en.wikipedia.org/wiki/Kanban_board

[college]: {% link articles/dwytic.md %}