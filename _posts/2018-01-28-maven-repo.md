---
title:  "Maven - Build your own projects repository"
date:   2018-01-28 13:55:01 +0100
categories: tools
---

# What is Maven?

[Apache Maven](https://maven.apache.org/) is a development tool that simplify building and managing dependencies of projects. It uses a XML file called [pom.xml](https://maven.apache.org/pom.html#What_is_the_POM) for configuration.

# <small>What is a Maven repository?</small>

A repository in Maven is a list of ready to use [artifacts](https://stackoverflow.com/a/2487511). If you add a repository to your `pom.xml` file, you can use any of its artifacts as a dependency for your project. You can also use projects that are locally stored on your computer without repository.
If you don't need to share libraries between workstations, there is no need to make one.

# <small>What I need to make a repository?</small>

First of all you need some space to store your projects. Then you need to transfer your project to this storage. To help you accomplish this, Maven created [wagons](https://maven.apache.org/wagon/) that handle this task for you. There are available many wagons that allow uploading your projects to a selected destination using different protocols like: SSH, HTTP, FTP or WebDAV.

# Wagon-SSH Example

# <small>Library example</small>

I will show you a simple Java project configured to deploy on a remote server using SSH protocol. Let's take a look at `pom.xml` file of this project.

```xml
<project>
    ...
    <properties>
        <server.user>user</server.user>
        <server.host>localhost</server.host>
        <server.path>repo</server.path>
    </properties>
    <build>
        <extensions>
            <extension>
                <groupId>org.apache.maven.wagon</groupId>
                <artifactId>wagon-ssh</artifactId>
                <version>2.7</version>
            </extension>
        </extensions>
    </build>
    <distributionManagement>
        <repository>
            <id>snapshots</id>
            <url>scp://${server.user}${server.host}/${server.path}</url>
        </repository>
    </distributionManagement>
</project>
```

If you don't want to pass your password during deployment, create SSH keys for yourself and copy your public key to the server.
You can check it [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2) how to do it. Remember that path in URL must be absolute.

When we have already had our project configured properly, we have to create very useful methods to share.

```java
package com.github.wpanas.maven.library;

public class Hello {
    public void world() {
        System.out.println("Hello world");
    }
}
```

To deploy a library to the server simple, put this line to a terminal and execute it. If you skip -D arguments, Maven will use default values stored in properties element. You can also ignore properties and type whole URL in `pom.xml` file.

```bash
mvn deploy -Dserver.user={user} -Dserver.host={host} -Dserver.path={path}
```

Remember to always change project version before deployment.

# <small>Usage example</small>

This project will be an example of using deployed library to our remote Maven repository. We have to add the repository to `pom.xml` file
and add library as a dependency.

```xml
<project>
    ...
    <properties>
        <server.host>localhost:8082</server.host>
        <server.path>repo</server.path>
    </properties>
    <dependencies>
        ...
        <dependency>
            <groupId>com.github.wpanas.maven</groupId>
            <artifactId>library</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
    </dependencies>
    <repositories>
        <repository>
            <id>snapshots</id>
            <name>My Maven Repository</name>
            <url>http://${server.host}/${server.path}</url>
        </repository>
    </repositories>
</project>
```

Remember that directory used for Maven repository needs to have autoindex enabled.

To use deployed library, import proper package and you are ready to use it.

```java
package com.github.wpanas.maven.usage;

import com.github.wpanas.maven.library.Hello;

public class Usage {
    public static void main(String[] args) {
        Hello hello = new Hello();
        hello.world();
    }
}
```

Here is the result.

```bash
Hello world

Process finished with exit code 0
```

You can avoid adding `repositories` element to your projects. It can be set globally in `~/.m2/settings.xml` file. You can look it up [here](https://maven.apache.org/settings.html).

# Summary

Maven allows you to create personal repositories with your projects. That way project can be shared with your colleagues/coworkers.
To make it simple, Maven's developers introduced wagons - small extensions that upload artifacts to a repository.

Projects created for this article are available on [GitHub](https://github.com/wpanas/code-snippets/tree/master/maven).