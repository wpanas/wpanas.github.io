---
title: "This week I learned #1 - Property-based testing, usefulness of HATEOAS and Victor Rentea awesomeness"
categories: learning
description: I went to SoftwareMill's 10 years anniversary.
---

Every week I try to learn something new and grow as a software developer.
I want to journal my discoveries and share them with community.

# Property-based testing

This week I attended my local JUG. [SoftwareMill](https://twitter.com/softwaremill) was celebrating it's 10 year anniversary. 
They organized two talks for us. First one was about property-based testing.
You can read the [slides](https://slides.com/magdastozek/property-based-testing#/), but I warn you that they are in polish.

I've never given a much thought about property-based testing. I think
that I misunderstood them. Thanks to [Magda Stożek](https://twitter.com/magdastozek) I know that it was mistake.

Before this meeting, I viewed property-based testing as a parametrized
unit tests. It's partially true, but the essence of property-based testing is exploring a domain and defining properties.

I will give you an example. Let's say we want to make function that sorts list of numbers. You can write a unit test like that:

```
# given
list = [7, 5, 6]

# when
result = sort(list)

# then
assert result == [5, 6, 7]
```

You can even parametrize it and check multiple lists yourself.
So why property-based testing is different. First of all, you need to define a property. 
The property of the sorted list of numbers is that the next number in the list is greater than or equal to the previous number. To test it, you could write a property-based test similar to this:

```
# given
list = generateListOfNumbers()

# when
result = sort(list)

# then
assert result[0] <= result[1]
assert result[1] <= result[2]
```

In my pseudo-code I used function `generateListOfNumbers`. This is the next important factor of property-based testing.
Frameworks for property-based testing will test your code with thousands of random generated lists, trying to break it.

In unit tests random values are considered anti-pattern. In property-based test they are very much needed. Using random list of numbers allows us to check if property is always true. 

If something goes wrong and a test fails, the framework that runs your tests, will take care for you to save that case.
This will allow you to rerun that particular case and write a unit test to verify, if you fixed that bug.

You should consider property-based tests as an extension of unit tests. It is not a replacement.

If you are interested in the topic, check out those resources.
- [jqwik](https://jqwik.net/)
- [ScalaCheck](https://www.scalacheck.org/)
- [PropEr Testing](https://propertesting.com/toc.html)
- [Examples from Magda's presentation](https://github.com/magdzikk/property-based-testing)
- [An introduction to property-based testing](https://fsharpforfunandprofit.com/posts/property-based-testing)

# Victor Rentea is awesome!

You have to watch [Victor Rentea](https://twitter.com/VictorRentea)'s [talk](https://youtu.be/tMHO7_RLxgQ)
about hexagonal architecture from Voxxed Days Bucharest 2019.
I discovered Victor's talks last week. I enjoy his energy and passion
for programming. 

This is one of my favorite talks of 2019. It's pure knowledge.
Just watch it!

# HATEOAS is very much needed

I am preparing myself to give a talk at my local university about REST API.
For inspiration I've watched talks at YouTube. I stumbled 
at [Olivier Drotbohm](https://twitter.com/odrotbohm)'s
talk [REST beyond the Obvious](https://youtu.be/Z9E7sDhCn5U) from GOTO Amsterdam 2019.

He was talking about how to prevent coupling in designing REST API.
I've never considered HATEOAS as a solution, but he gave a very good [example](https://github.com/odrotbohm/spring-restbucks). 

Before his talk I considered HATEOAS as a extra feature, but not necessarily needed. Now I know, that HATEOAS helps with decoupling 
client and server application logic.

My talk will be an introduction to REST API. I thought I would just mention HATEOAS, but now I would like it to be the major part of my presentation.