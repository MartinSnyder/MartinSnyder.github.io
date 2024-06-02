---
layout: page
title: Improving Skills as a Software Practitioner
---

One topic that often comes up in my conversations is what are my recommended techniques to improve as a software practitioner. The most common scenarios involve current students, early career job seekers, and early career professionals trying to "make the jump" to the higher levels, but there are others as well. My advice for all populations is the same. In every case that I've seen, the best practitioners write a lot of code, and are always learning. And, while there is much to learn and gain in academic environments, that's just not where the best skill sets are identified or refined.

I think this is because Programming, Software Engineering, and Compute Science are distinct fields and while they often overlap in practice, mastery of any one of them does not imply mastery of the others. Some of these elements can be learned in a classroom, but others are best learned through application. This is one reason why people with similar experience levels and skills can very so much in overall effectiveness, and it's also a factor in why many consider programming to be more of a [craft][craft] than an engineering discipline. These observations in part fueled my [Musings on College][college], though I feel that article applies to all academic fields and not just the computer-related ones.

Many people learn tremendous amounts though their employment, which is great. After a certain amount of time though, the rate of learning there will slow down 

This topic dominates job seeking because while there are many elements that contribute to being an impactful software practitioner, the raw programming skills are the easiest to assess and so there is an outsized focus on technical assessments during software interviews. I'm not aware of any field that has spawned a comparable number of [books on preparing for interviews][books] the same way that the software industry has. Note that if you only care about the interview process and technical assessments, rather than follow my advice, you probably want to read one of those books while working through a puzzler site like [Project Euler][euler] or [Advent of Code][advent].

My recommendation for improvement is to build something meaningful under the guidance of a mentor, akin to an apprenticeship. Don't choose your own project though, because that introduces a distraction in refining the idea itself, rather than the implementation. Instead, pick a popular but simple website known to both the student and the mentor and recreate it. [Twitter, or X or whatever][twitter] is one of the simplest social media sites. [Trello][trello] implements powerful yet flexible [kanban][kanban] boards. Building a competitor to one of these products is hard, but building a personal implementation is not, mainly because you get to choose which aspects to take on, and your implementation will likely never have any real users.

If you want to pick a different implementation target, look for the following characteristics that provide fertile context for learning:
* Simple data models
* Manageable lists of functional requirements
* Data storage requirements which can be met using almost any technology
* Simple user interfaces
* A reference implementation that can be studied

Finding a mentor is easier for some than others. If you are struggling to find ont, consider reaching out on Social Media sites like [LinkedIn][linkedin]. If you are a student, see if you can send a request to the alumni of your organization to find someone willing to spend time with you. If all of that fails, don't ask for a long-term mentor, and instead ask people to look at what you have written in isolation. It's also possible to do this without a mentor, though that path is more challenging, exceptionally more so for people who are still seeking their first job.

Build any part of the target site as best you can, regardless of how poor the result may be. Print out G.K. Chesterton's quote "[Anything worth doing is worth doing badly][badly]" and tape it to your wall above your computer screen. It really doesn't matter how clumsy the initial implementation is, how simplified your interpretation, or how narrow the scope. Iterate with your mentor on the code you are actually writing. [Version control everything][github]. Improve your implementation and work to understand what improved fundamentally. A key part of the craft of programming is forming your own opinions as to the right and wrong way of doing things.

It's possible to iterate in this fashion for months or even years. When you feel you have hit a wall in terms of improving the code you have written and you and your mentor have run out of ideas, consider the following exercises:

Beginner:
1. Write the data model for the document. Establish what information constitutes the data model and how to best represent it.
2. Write functional requirements for your application. Work to clearly delineate in your own mind what the application does versus how it does it.

Intermediate:
1. Deploy your application to a cloud provider. Consider a platform as a service (PAAS) like [Render][render] (note Render still has a free tier) or [Heroku][heroku]. This will open your mind to an entirely different category of the field
2. Examine and work to understand the HTTP traffic using your browser's debugger. Try to replicate browser requests using a tool like [Postman][postman].
3. Adopt a UI framework like [Material UI][material] or [Bootstrap][bootstrap] for your application. Adopting a framework usually forces you to organize and conceptualize your application in a very specific way. One is not necessarily better than the other, but learning about different ways to think about the same problem will be useful.

Advanced:
1. Replace a key part of your application such as the database or storage mechanism or application framework
2. Adopt a new programming language, especially one with different properties than the one you initially chose
3. Build a CI/CD pipeline to automatically update a server when you push changes
4. Use a tool like [Apache JMeter][jmeter] to apply a user load to your application, and investigate whatever breaks!


[craft]: https://en.wikipedia.org/wiki/Software_craftsmanship
[garden]: https://blog.codinghorror.com/tending-your-software-garden
[books]: https://www.google.com/search?q=technical+interviewing+books
[euler]: https://projecteuler.net
[advent]: https://adventofcode.com
[twitter]: https://x.com
[trello]: https://trello.com
[kanban]: https://en.wikipedia.org/wiki/Kanban_board
[linkedin]: https://linkedin.com
[badly]: https://www.psychologytoday.com/us/blog/second-wind/201206/anything-worth-doing-is-worth-doing-badly
[github]: https://github.com
[render]: https://render.com
[heroku]: https://www.heroku.com
[postman]: https://www.postman.com
[material]: https://en.wikipedia.org/wiki/Material_Design
[bootstrap]: https://github.com/twbs/bootstrap
[jmeter]: https://jmeter.apache.org

[college]: {% link articles/dwytic.md %}