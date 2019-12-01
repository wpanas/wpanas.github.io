---
title: "This week I learned #4 - Architecture Guilds, Event-driven systems and talk at University"
categories: learning
description: I read best book about Event Sourcing. I had a talk at University.
---

# Architecture Guilds

[Jakub Nabrdalik](https://twitter.com/jnabrdalik)
had a [talk about mistakes made when moving to microservices](https://youtu.be/6H3mSY1AJ1k) at GeeCon Prague 2019. 
You can watch it also [in polish](https://youtu.be/jo46-CP6ywU) from [Confitura 2019](https://www.youtube.com/playlist?list=PLVbNBx5Phg3AwVti8rYNqx7965pgfMZWO).

He did a thesis that majority of companies that migrated to 
microservices from a legacy monolith made similar mistakes.
From his experience as a consultant, he described what was done 
poorly and how to do it better.

Big part of talk was about software architecture and how it was enforced in companies. My favorite quote from this part is:

> Architecture is the shared understanding of people on the project.

My conclusions that came from this quote:
- all developers create the software architecture consciously or not
- sharing knowledge is the best way to enforce architecture changes 
- all developers must have a single baseline to compare their individual understanding of architecture

Jakub Nabrdalik introduced a solution that covers part of these conclusions -- architecture guilds.
Guilds are a way for spreading and cultivate the knowledge.
To be effective, they need to be open and share there work with whole company. Members of the guild have to guide and teach.

To cover the rest of these conclusions Jakub Nabrdalik was convincing
the audience to implement [C4 model](https://c4model.com/) 
of theirs software architecture in open for all developers repository. 
This way every developer could introduce changes
as pull requests that would be reviewed. 
This repo would be the baseline for everyone to compare.

Jakub Nabrdalik recommended books by [Simon Brown](https://twitter.com/simonbrown)
-- [Software Architecture for Developers](https://leanpub.com/b/software-architecture).

He covered other topics of migration from monolith to microservices.
I will not cover them, so if you want to know more go watch it.


# Event-Driven Systems & CQRS

Jakub Nabrdalik in his talk also recommended another book
-- [Designing Event-Driven Systems](http://www.benstopford.com/2018/04/27/book-designing-event-driven-systems/) by [Ben Stopford](https://twitter.com/benstopford). 
You can download it right now for free from Ben Stopford's site.
I did it and read it in less than a week.

Ben Stopform is CTO of [Confluent](https://www.confluent.io/) --
a company that describe itself as *The Complete Event Streaming Platform For Apache Kafka*. 

No one will be surprised that his book about the event streaming 
is based on [Apache Kafka](https://kafka.apache.org/).
Fortunately, this book is not just an advertisement for technology. 
Examples are made in Apache Kafka, but the knowledge is universal.

I like the fact that this book introduces more and more difficult concepts one by one. Each concept is break down with an example.
All examples are about the same use-case, so it easy to compare them.

I like this book, because it gave me confidence on how and when
to use the event streaming. 
Here on of my favorite quotes from this book.
I hope they will encourage you to read it.

> Centralize an immutable stream of facts.
> Decentralize the freedom to act, adapt and change.

> Event Sourcing ensures every state change in system is recorded,
> much like a version control system.

> If an event stream is source of truth, you can have as many
> different views in as many different shapes (...) as you may need.

> If you squint a bit, you can see the whole of your organization's
> systems and data flows as a single distributed database. 
> Jay Kreps, 2013

> The more mutable copies, the more data will diverge over time.

# Talk at Poznan University of Technology

On Monday I had a presentation at [put.net](https://www.facebook.com/put.net/).
It is the science club at Poznan University of Technology.
Their main focus is C#, but they talk about many other aspects 
of developing software. 
[Jakub Riegel](https://www.linkedin.com/in/jakubriegel/),
who takes care of Web Development section, invited me to talk
about REST API. I was very happy to help and share my experience
with students.

I introduced them to common REST API implementation with HTTP and JSON. [Oliver Drotbohm](https://twitter.com/odrotbohm)'s 
presentation inspired me to show students 
usage of [HATEOAS](https://restfulapi.net/hateoas/). 
Here you can [read the slides from my presentation](https://www.slideshare.net/secret/jUTgKcb5ifOB7O).

# Bookmarks drop

- [Hermitage](https://github.com/ept/hermitage) - project that 
verifies how isolation levels are **actually implemented** in different database systems.
[Dr Martin Kleppmann](https://twitter.com/martinkl) wrote it as a background for his book -- [Designing Data-Intensive Applications](https://dataintensive.net/). If you don't believe Dr Martin Kleppmann, you can check them yourself. He left instructions to manually verify his summary. Currently project contains tests for: 
[PostgreSQL](https://www.postgresql.org/), 
[MySQL](https://www.mysql.com/),
[Oracle DB](https://www.oracle.com/pl/database/),
[MS SQL Server](https://www.microsoft.com/pl-pl/sql-server/sql-server-2019)
and [FoundationDB](https://www.foundationdb.org/)
- [Awesome Conference Practices](https://github.com/kitze/awesome-conference-practices) by [Kitze](https://twitter.com/thekitze ) sums up what you need to do if you want to organize a conference
- [Awesome EventStorming](https://github.com/mariuszgil/awesome-eventstorming) 
by [Mariusz Gil](https://twitter.com/mariuszgil). 
It is a collection of resources -- books, articles, videos,
presentation slides, people to follow on Twitter,
projects and communities
-- about Event Storming. If you are interested in the topic,
you should star this repo and contribute.
- Microsoft released article -- [Reasons to move to Java 11](https://docs.microsoft.com/en-us/azure/java/jdk/reasons-to-move-to-java-11)
-- that covers changes between Java 8 and Java 11.
- [Claes Redestad](https://twitter.com/cl4es) from Oracle compared
[startup times for OpenJDK](https://cl4es.github.io/2019/11/20/OpenJDK-Startup-Update.html)
from Java 8 to 14. 
I am impressed with changes that came with new Java versions.
Read it and see for yourself.
- [Angie Jones](https://twitter.com/techgirl1908) 
and [Dr Nicole Forsgren](https://twitter.com/nicolefv) did a webinar
-- [Test Automation as a Key Enabler for High-performing Teams](https://applitools.com/blog/test-automation-key-enabler-for-success) -- this week. It is based on book [Accelerate](https://itrevolution.com/book/accelerate/) and revelations that came out of it. 
- Article [The Service Mesh: What Every Software Engineer Needs to Know about the World's Most Over-Hyped Technology](https://servicemesh.io/)
by [William Morgan](https://twitter.com/wm) debunks myths
about service mesh. It describes what is services mesh and what isn't.
If you want to know better how service mesh could be helpful
and when to use them, you should read it.
- If you missed, [ToughtWorks](https://www.thoughtworks.com/) released [Tech Radar vol. 21](https://assets.thoughtworks.com/assets/technology-radar-vol-21-en.pdf) this week. I wish to cover it in detail in the next week article.
- Article [How to hire sane engineers 2.0](https://vas3k.com/notes/hiring/)
by [vas3k](https://twitter.com/vas3kcom). It is suited for HR,
but for me it is always interesting to see things from the other side. 
I also like to compare and benchmark myself by measures in those 
kinds of articles.