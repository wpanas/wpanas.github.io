---
title: "This week I learned #5 - How to setup reactive client with OAuth in Spring Boot"
categories: learning
description: I will show you how to setup WebClient with OAuth. It is also the summary of last four weeks of learning.
---

# WebClient with OAuth

I started a new project with WebFlux and wanted to access API with OAuth2 authorization. 
The [official documentation for setting up OAuth2 with WebFlux](https://docs.spring.io/spring-security/site/docs/current/reference/html/webflux-oauth2.html)
was helpful, but not quite covered my case. 
Later, I found Baeldung's article 
-- [Spring Security OAuth Login with WebFlux](https://www.baeldung.com/spring-oauth-login-webflux#webclient). I tried using his solution, but 
`UnAuthenticatedServerOAuth2AuthorizedClientRepository` exception was thrown, when I booted the application up. 
I searched and found the final peace 
at [StackOverflow](https://stackoverflow.com/questions/56973321/spring-boot-reactive-webclient-serverwebexchange-must-be-null-when-spring-secu/57788475#57788475).

Below, I will show you, how to make WebClient 
for [Petfinder's public API](https://www.petfinder.com/developers/)
that will authorize itself.

First, you need to add configuration to `src/main/resources/application.yml`. In this example my client
will be distinguished by key `pet-finder`. This key is a registration id and will be needed later. You can have multiple OAuth configurations and those registration ids
will be used to separate them.

```yaml
spring.security.oauth2.client:
    provider.pet-finder.token-uri: "https://api.petfinder.com/v2/oauth2/token"
    registration.pet-finder.authorization-grant-type: "client_credentials"
```

Next, setup configuration with a `WebClient` bean. To authorize to Petfinder API,
`WebClient` needs to be extended with a filter -- `ServerOAuth2AuthorizedClientExchangeFilterFunction`.
Using registration id -- `pet-finder` -- setup previously configured token URI and authorization grand type.

```kotlin
@Configuration
class PetFinderConfiguration {
    @Bean
    WebClient petFinderClient(
        clientRegistrations: ReactiveClientRegistrationRepository, 
        clientRepository: ServerOAuth2AuthorizedClientRepository
    ) {
        val filter = serverOAuth2AuthorizedClientExchangeFilterFunction(
            clientRegistrations, 
            clientRepository
        )
        filter.setDefaultClientRegistrationId("pet-finder")

        return WebClient.builder()
            .filter(filter)
            .build()
    }
}
```

The last part is configuring your client id and secret. They should not
be included in your source code. 
Leaving secrets in code is asking yourself for trouble.
The best way to set them up is to pass them as arguments
when starting Spring application. With Maven it could look like:

```bash
./mvnw spring-boot:run -Dspring-boot.run.arguments=--spring.security.oauth2.client.registration.pet-finder.client-secret=YOUR_SECRET,--spring.security.oauth2.client.registration.pet-finder.client-id=YOUR_ID
```

That's all that is needed. Now you can use this `WebClient` to 
[fetch information about pets](https://www.petfinder.com/developers/v2/docs/#get-animals) 
that are waiting for adoption.

```kotlin
@Component
class AnimalsService(private val petFinderClient: WebClient) {
    fun fetch(): Flux<Animal> = petFinderClient
        .get()
        .uri("https://api.petfinder.com/v2/animals")
        .retrieve()
        .bodyToMono(Payload::class.java)
        .flatMapIterable { it.animals }
}

data class Payload(val animals: Set<Animal>)

data class Animal(val id: String, val name: String, val species: String)
```

# The summary of last 4 weeks and plans for future

I've learned a lot. I've read 
[Designing Event-Driven Systems](https://www.confluent.io/designing-event-driven-systems/)
and [Working Effectively with Legacy Code](https://www.goodreads.com/book/show/44919.Working_Effectively_with_Legacy_Code).
I've been at Poznan University of Technology performing my presentation about
Rest API. A lot has happened.

I will not be writing weekly articles next month. I want to take time and figure out
how I see my blog in future.
I feel like during this month, I wrote about many interesting but different
issues.
Software architecture -- especially architecture of distributed systems -- is the main
topic that I want to focus.
I want it to be the core, but not exclusive topic of my posts.

I wish I will not loose momentum and return in 2020 with many valuable articles
that will expand your and my knowledge!