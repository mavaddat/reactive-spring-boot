This is the first step showing how to create a Reactive Spring Boot application in VS Code. This tutorial has been adapted from presentations of @trishagee.

# Creating a Reactive service with Kotlin

We will build a Spring Boot application that displays real-time
prices, using Spring, Kotlin and JavaFX.

[This tutorial is a series of
steps](https://blog.jetbrains.com/idea/tag/tutorial-reactive-spring/)
during which we will build a full [Spring
Boot](https://spring.io/projects/spring-boot) application featuring a
[Kotlin](https://kotlinlang.org/) back end, a
[Java](https://jdk.java.net/13/) client and a
[JavaFX](https://openjfx.io/) user interface.

This first step in the tutorial is to create a [Kotlin Spring Boot
Application](https://spring.io/guides/tutorials/spring-boot-kotlin/)
which serves as the back end of the application. We'll create a
[REST](https://en.wikipedia.org/wiki/Representational_state_transfer)
service that can be connected to in later parts of the tutorial.

### Create a Spring Boot service

Let's create a [new project for our Spring
Boot](https://www.jetbrains.com/help/idea/spring-boot.html#create-spring-boot-project)
service.

1.  Select [New
    Project](https://www.jetbrains.com/help/idea/new-project-wizard.html),
    either from the menu of IntelliJ IDEA or from the start screen.
2.  Select the Spring Initializr option on the left of the New Project
    window.
3.  We're using Java 13 as the SDK for this tutorial, although we're not
    using any of the Java 13 features (you can [download JDK
    13.0.1](http://jdk.java.net/13/) here, then [define a new IntelliJ
    IDEA SDK](https://www.jetbrains.com/help/idea/sdk.html#define-sdk)
    for it).
4.  Enter the group name for the project, and we'll use stock-server as
    the name.
5.  We can build this project with Maven or Gradle. We're going to
    create a Maven project which will generate the pom.xml and maven
    wrapper files that we need.
6.  Choose [Kotlin](https://kotlinlang.org/) as the language. We'll
    select Java 11 as the Java version as this is the [most recent Long
    Term
    Support](https://blog.jetbrains.com/idea/2018/09/using-java-11-in-production-important-things-to-know/)
    version for Java, but for the purposes of this project it makes no
    difference.
7.  The project name is automatically populated from the artifact name,
    we don't need to change this.
8.  Add a useful description for the project.
9.  We can optionally change the top level package if we need to.

Next we select the [Spring Boot
Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters)
we need.

1.  Choose which version of Spring Boot to use. We're going to use
    [2.2.0
    RC1](https://spring.io/blog/2019/10/03/spring-boot-2-2-0-rc1-has-been-released)
    for this tutorial because later we'll be using features that are
    only available in the release candidate.
2.  We can search for and select which Spring Boot starters we want to
    use. This is a reactive REST service, so select Reactive Web.
3.  We'll use the defaults for project name and location.

IntelliJ IDEA will use Spring Initializr to create the project and then
import it correctly into the IDE. [Enable auto-import on
Maven](https://www.jetbrains.com/help/idea/maven-importing.html#auto_import)
so when we make changes to the pom.xml file the project dependencies
will automatically be refreshed.

### The Spring Boot project

In the project window we see the structure of the project that has been
created, including a Kotlin directory and the standard default
Application class that Spring Boot creates.\

``` default
package com.mechanitis.demo.stockservice

import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class StockServiceApplication

fun main(args: Array<String>) {
    runApplication<StockServiceApplication>(*args)
}
```

\
IntelliJ IDEA Ultimate has [full support for Spring
applications](https://www.jetbrains.com/help/idea/spring-support.html),
which includes gutter icons making it easier to identify and navigate
between Spring elements like Spring Beans.

If we take a quick look at the generated pom.xml file, we see not only
the selected Spring Boot starters and Kotlin dependencies, we can also
see the [Spring compiler
plugin](https://kotlinlang.org/docs/reference/compiler-plugins.html#spring-support)
in the
[kotlin-maven-plugin](https://kotlinlang.org/docs/reference/using-maven.html).
This makes it a bit simpler to work with Spring in Kotlin.

[Run](https://www.jetbrains.com/help/idea/running-applications.html)
this basic application using the shortcut Ctrl+Shift+F10 for Windows or
Linux (ΓîâΓçºR for macOS) from inside the Application class file, or
double-press Ctrl ("run anything") and type "StockServiceApplication" to
run the application. It should start successfully, with
[Netty](https://netty.io/) running on port 8080.
[Stop](https://www.jetbrains.com/help/idea/stopping-and-pausing-applications.html)
it with Ctrl+F2 (ΓîÿF2).

### Create a REST Controller

Now we know it all works, we can get started on our own functionality.

1.  Create a class for our REST Controller. We'll put it in the same
    Kotlin file to keep things simple for now.
2.  We need to annotate this as a \@RestController.
3.  (Tip: we can use [Live
    Templates](https://www.jetbrains.com/help/idea/using-live-templates.html)
    to create code faster. We can type "fun1" and press tab to create a
    function that takes a single argument).
4.  Create a function "prices" which takes the symbol of the stock we
    want to get the prices for. This method will return
    [Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)\<StockPrice>,
    which will be a constant stream of prices.

``` default
@RestController
class RestController() {
    fun prices(@PathVariable symbol: String): Flux<StockPrice> {

    }
}
```

### Create a data class for stock price

1.  (Tip: we can get IntelliJ IDEA to create the StockPrice class by
    pressing Alt+Enter on the red StockPrice symbol and selecting
    "Create class StockPrice")
2.  Create a StockPrice class inside this same Kotlin file.
3.  This is going to be Kotlin [data
    class](https://kotlinlang.org/docs/reference/data-classes.html#data-classes).
    This is a compact way to declare a class with properties, and we
    simply declare what we want in the Constructor parameters. We want a
    symbol for the stock, which is a String, a price, which is a Double,
    and the time associated with that price, using Java 8's
    [java.time.LocalDateTime](https://docs.oracle.com/javase/8/docs/api/java/time/LocalDateTime.html).

``` java
data class StockPrice(val symbol: String,
                      val price: Double,
                      val time: LocalDateTime)
```

### Generate and return prices

Now we're going to define what the prices method returns. This method is
going to create a Flux which emits one randomly generated price every
second. We can do this with an interval with a Duration of one second.\

``` java
    fun prices(symbol: String): Flux<StockPrice> {
        return Flux.interval(Duration.ofSeconds(1))
    }
```

\
(note: this code will not compile yet)

Then we can create a new StockPrice for each of these seconds. Note that
in Kotlin we don't need the "new" keyword. The StockPrice needs the
symbol, a price, which for the purposes of this tutorial will be
randomly generated, and a time, which will be "now".\

``` java
    fun prices(symbol: String): Flux<StockPrice> {
        return Flux
                .interval(Duration.ofSeconds(1))
                .map { StockPrice(symbol, randomStockPrice(), LocalDateTime.now()) }
    }
```

\
(note: this code will not compile yet)

Create this randomStockPrice function (we can use Alt+Enter to
automatically create it). One way to create an arbitrary Double is using
ThreadLocalRandom and the nextDouble method. Let's generate a number
between zero and one hundred.\

``` java
    private fun randomStockPrice(): Double {
        return ThreadLocalRandom.current().nextDouble(100.0)
    }
```

\
Since we want to be able to access the prices method via an HTTP Get
call, we need to add the
[\@GetMapping](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/bind/annotation/GetMapping.html)
annotation to it. It needs a path ("/stocks/{symbol}"), and we need to
define what the response to this method looks like, we're going to use
[TEXT_EVENT_STREAM_VALUE](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/MediaType.html#TEXT_EVENT_STREAM_VALUE)
to create a [server-sent
events](https://en.wikipedia.org/wiki/Server-sent_events) streaming
endpoint. We need to also declare our symbol parameter as a
[\@PathVariable](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/PathVariable.html).

The whole RestController now looks like\

``` java
@RestController
class RestController() {
    @GetMapping(value = ["/stocks/{symbol}"],
                produces = [MediaType.TEXT_EVENT_STREAM_VALUE])
    fun prices(@PathVariable symbol: String): Flux<StockPrice> {
        return Flux
                .interval(Duration.ofSeconds(1))
                .map { StockPrice(symbol, randomStockPrice(), LocalDateTime.now()) }
    }

    private fun randomStockPrice(): Double {
        return ThreadLocalRandom.current().nextDouble(100.0)
    }
}
```

### Running the application

Run the application to see it start up correctly. Open up a web browser
and navigate to http://localhost:8080/stocks/DEMO, you should see the
events tick once a second, and see the stock price represented as a JSON
string:\

``` default
data:{"symbol":"DEMO","price":89.06318870033823,"time":"2019-10-17T17:00:25.506109"}
```

### Summary

We've created a simple Kotlin Spring Boot application that uses Reactive
Streams to emit a randomly generated stock price once a second.\

``` java
@SpringBootApplication
class StockServiceApplication

fun main(args: Array<String>) {
    runApplication<StockServiceApplication>(*args)
}

@RestController
class RestController() {
    @GetMapping(value = ["/stocks/{symbol}"], produces = [MediaType.TEXT_EVENT_STREAM_VALUE])
    fun prices(@PathVariable symbol: String): Flux<StockPrice> {
        return Flux.interval(Duration.ofSeconds(1))
                   .map { StockPrice(symbol, randomStockPrice(), LocalDateTime.now()) }
    }

    private fun randomStockPrice(): Double {
        return ThreadLocalRandom.current().nextDouble(100.0)
    }
}

data class StockPrice(val symbol: String, val price: Double, val time: LocalDateTime)
```

In the rest of the tutorial we'll show how to connect to this server to
retrieve prices, and how to create a chart that shows the price updates
in real time.

[Full code is available on
GitHub](https://github.com/trishagee/jb-stock-service).


&nbsp; | &nbsp; | &nbsp; | &nbsp;
----|----|----|----
[&#10094; Prev](./README.md)| &nbsp; | &nbsp; | &nbsp;[Next &#10095;](./STEPTWO.md)