---
title: Best way to handle unknown enum values using Jackson
---

In this article I want to show you how to handle
enums using [Jackson]. It not as easy as it seems.
Jackson can't handle unknown enum values by default.
You have to decide, what to do. If you don't, your application
will fail, when external resource changes and new enum value is 
added.

Since Jackson 2.0, you can use [READ_UNKNOWN_ENUM_VALUES_AS_NULL] feature. Still you will have to handle `null`, when unknown value appears in deserialized enum.

Since 2.8 there is a better way. Use [READ_UNKNOWN_ENUM_VALUES_USING_DEFAULT_VALUE].
This feature allows you, to select a value that will be used by default. Let's see, how it will look.

Provided examples are in [Kotlin] and use [jackson-module-kotlin] 2.12+.

```kotlin
enum class Color {
    RED, GREEN, BLUE,
    @JsonEnumDefaultValue UNKNOWN
}

data class Pixel(val color: Color)
```

Don't forget to enable `READ_UNKNOWN_ENUM_VALUES_USING_DEFAULT_VALUE`,
when configuring `ObjectMapper`.

```kotlin
val objectMapper = jacksonMapperBuilder()
    .enable(READ_UNKNOWN_ENUM_VALUES_USING_DEFAULT_VALUE)
    .build()
```

And that's how to handle enums to avoid dealing with errors when
new enum values are extended.

```kotlin
when (objectMapper.readValue<Pixel>(jsonString).color) {
    Color.RED -> print("Handle red")
    Color.GREEN -> print("Handle green")
    Color.BLUE -> print("Handle blue")
    Color.UNKNOWN -> print("Handle unknown")
}
``` 

You can see full example [here].


[Jackson]: https://github.com/FasterXML/jackson

[READ_UNKNOWN_ENUM_VALUES_AS_NULL]: http://fasterxml.github.io/jackson-databind/javadoc/2.12/com/fasterxml/jackson/databind/DeserializationFeature.html#READ_UNKNOWN_ENUM_VALUES_AS_NULL

[READ_UNKNOWN_ENUM_VALUES_USING_DEFAULT_VALUE]: http://fasterxml.github.io/jackson-databind/javadoc/2.12/com/fasterxml/jackson/databind/DeserializationFeature.html#READ_UNKNOWN_ENUM_VALUES_USING_DEFAULT_VALUE

[Kotlin]: https://kotlinlang.org/

[jackson-module-kotlin]: https://github.com/FasterXML/jackson-module-kotlin

[here]: https://github.com/wpanas/jackson-default-enum