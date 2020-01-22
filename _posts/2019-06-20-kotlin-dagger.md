---
title: "Using Dagger 2 in Android and Kotlin"
categories: programming
description: Quick guide how to configure Dagger 2 with Kotlin
---

# What made me do this article?

I started programming Android applications few months ago. I went through fundamental and more advanced guides. None of them mentioned dependency injection. Without it I struggled to make proper unit tests. I decided that I have to make it work.

I looked around and found [Dagger 2](https://dagger.dev). The project developed by Google that provides dependency injection for Android applications. Sadly, I had a hard time making it work with Kotlin. 

I have read many articles, but they had only scraps of necessary information. Combining them together I made it work. This post is a summary of my small research. 
It won't be about Dagger 2 basics. If you want to know more, read official [user's guide](https://dagger.dev/users-guide). 
Those are articles that inspired me:

- [Setup Dagger 2.11 on Kotlin Project](https://medium.com/@malinitin/setup-dagger-2-11-on-kotlin-project-2257ad84ad7c)
- [How to use Android Injector for Activity and Fragment objects through New Dagger 2 (with Kotlin)](https://medium.com/@serapbercin001/how-to-use-android-injector-for-activity-and-fragment-objects-through-new-dagger-2-with-kotlin-704eb8a98c43)
- [New Android Injector with Dagger 2 — part 3](https://android.jlelse.eu/new-android-injector-with-dagger-2-part-3-fe3924df6a89)
- [Dependency Injection on Android with Dagger 2 (and Kotlin) — Part 1](https://medium.com/@rachitmishra/dagger-2-with-kotlin-59d3c2440f4f)

# How to setup Dagger 2 with Kotlin?


First of all, add dependencies to `app/build.gradle` and enable `kapt`.

```bash
...

apply plugin: 'kotlin-kapt'

android {
    ...
}

dependencies {
    ...
    implementation 'com.google.dagger:dagger-android:2.x'
    implementation 'com.google.dagger:dagger-android-support:2.x'
    kapt 'com.google.dagger:dagger-android-processor:2.x'
    kapt 'com.google.dagger:dagger-compiler:2.x'
}
```

# Provide modules to inject

Organize your code in modules that provide dependencies. Activities and fragments are dependencies too. They need to be in special abstract modules that use `@ContributesAndroidInjector` annotation on methods.

```kotlin
@Module
abstract class ActivityProvider {
    @ContributesAndroidInjector
    abstract fun mainActivity(): MainActivity
}
```

For other objects write modules with methods annotated with `@Provides`.

```kotlin
@Module
class CoffeeModule {
    @Provides
    @Singleton
    fun coffeeService() = CoffeeService()
}

class CoffeeService {
    fun prepareCoffee() = "Here! Espresso for you"
    fun fetchCoffee() = "espresso"
}
```

`OrderService` depends on `CoffeeService`. You don't need to use `@Inject` annotation to do this with Dagger.

```kotlin
@Module
class OrderModule {
    @Provides
    @Singleton
    fun orderService(coffeeService: CoffeeService)
     = OrderService(coffeeService)
}

class OrderService(private val coffeeService: CoffeeService) {
    fun askForOrder() = "Would you like some ${coffeeService.fetchCoffee()}?"
}
```

# Prepare your Application

Next add `AppComponent` that extends `AndroidInjector<DaggerApplication>`. It is vital that one of the included modules is `AndroidSupportInjectionModule`.

```kotlin
@Singleton
@Component(modules = [AndroidSupportInjectionModule::class, ActivityProvider::class, CoffeeModule::class, OrderModule::class])
interface AppComponent : AndroidInjector<DaggerApplication>
```

Add your own `DaggerApplication` implementation. This will be the starting point of your Android application.

```kotlin
class InjectionExampleApplication : DaggerApplication() {
    override fun applicationInjector(): AndroidInjector<out DaggerApplication> {
        val appComponent = DaggerAppComponent.builder()
            .build()

        appComponent.inject(this)

        return appComponent
    }
}
```

Don't forget to add attribute `android:name` in `AndroidManifest.xml`.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.github.wpanas.injectionexample">

    <application
        android:name=".InjectionExampleApplication"
        ...
    >
        ...
    </application>

</manifest>
```
From now on, dependencies registered by modules in `AppComponent` will be available for injection.

# Inject dependencies

The last step is injecting those dependencies into your activities. Change parent class of your `MainActivity` from `AppCompatActivity` to `DaggerAppCompatActivity`. Then add your dependencies as `lateinit var` properties with `@Inject` annotation.

```kotlin
class MainActivity : DaggerAppCompatActivity() {

    @Inject
    lateinit var coffeeService: CoffeeService

    @Inject
    lateinit var orderService: OrderService

    ...
}
```

The same you can do with all activities and fragments. Add prefix `Dagger-` to theirs parents class names. This will ensure that dependencies are injected. Don't forget to add those activities and fragments to module like in `ActivityProvider`.

# Summary

Those simple steps enables you to make Dagger 2 work with Kotlin. All this code is captured in this [repository](https://github.com/wpanas/InjectionExample). For more complex example you can also check out this [project](https://github.com/wpanas/StateCounter) of mine.