---
title: Leader Only Spring Boot Starter
categories: tools
---

Two weeks ago I released a library [leader-only-spring-boot-starter].
It leverages [Apache Curator] to restrict computation to a single application node.
Underneath the library use [LeaderLatch].
All this is backed by [Apache Zookeeper]. If you want to use
it, you need to have Zookeeper up & running.
The library was created for applications that are developed
with [Spring Boot](https://spring.io/projects/spring-boot).

First of all you need to add dependency to your project. Copy right configuration for your build automation tool 
from [here](https://search.maven.org/artifact/pl.allegro.tech.boot/leader-only-spring-boot-starter/1.0.1/jar).

Then set up Zookeeper node by adding a property to Spring context.

```yaml
curator-leadership:
    connection-string: localhost:2181
```

A connection string has the following pattern: `IP1:PORT1,IP2:PORT2,...,IPN:PORTN`. As you can see you can use many Zookeeper nodes.

You can also configure authentication, timeouts & retries. 
All available configuration options are described [here](https://github.com/allegro/leader-only-spring-boot-starter#configuration).

The last step is to annotate a class and a method that you want
to execute only on single application node.

```kotlin
@Leader("unique-identifier")
class Example {

    @LeaderOnly
    fun operationThatIsExecutableOnlyOnLeaderNode() {
        TODO("implement")
    }

    fun operationThatIsExecutableOnAllNodes() {
        TODO("implement")
    }
}
```

`@Leader` annotation extends `@Component`. It creates a new `LeaderLatch` instance. 
You can have many latches each with a unique identifier.

`@LeaderOnly` annotation indicates a method that
will be executed on the selected leader node. Executing 
this method on nodes that weren't selected for the leader
will return `null`.

I hope you find it useful. If you want to see changes in
this library feel free to add a [pull request] or create
an [issue].


[leader-only-spring-boot-starter]: https://github.com/allegro/leader-only-spring-boot-starter
[Apache Curator]: https://curator.apache.org/
[LeaderLatch]: https://curator.apache.org/curator-recipes/leader-latch.html
[Apache Zookeeper]: https://zookeeper.apache.org/
[pull request]: https://github.com/allegro/leader-only-spring-boot-starter/pulls
[issue]: https://github.com/allegro/leader-only-spring-boot-starter/issues