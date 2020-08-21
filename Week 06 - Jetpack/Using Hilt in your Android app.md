# Using Hilt in your Android App

codelab link: [https://developer.android.com/codelabs/android-hilt#0](https://developer.android.com/codelabs/android-hilt#0)

useful link: https://developer.android.com/training/dependency-injection



## Overview

* Introducing Hilt to create a solid and extensible application that scales to large projects.

* How to apply Hilt to the existing project in order to make it scalable and boilerplate-free, by solving codelab set

  

## Prerequisites

* Experience with Kotlin syntax
* Understanding DI
* Further reading list
  * [Fundamentals of dependency injection](https://developer.android.com/training/dependency-injection)
  * [Manual dependency injection in Android](https://developer.android.com/training/dependency-injection/manual)



## Introducing Hilt

* Why DI?
  * Reusability of code
  * Ease of refactoring
  * Ease of testing
* Hilt...
  * Is an opinionated dependency injection library for Android that reduces the boilerplate of using manual DI in our project.
  * Provides a standard way to do DI in our application by providing **containers to every Android component**.

## Setup

Open the root **build.gradle** and put the code below.

```groovy
buildscript {
    ...
    ext.hilt_version = '2.28-alpha'
    dependencies {
        ...
        classpath "com.google.dagger:hilt-android-gradle-plugin:$hilt_version"
    }
}
```

Open the **app/build.gradle** and specify the plugins for the Hilt.

```groovy
...
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
    ...
}
```

Place Hilt dependencies in same **app/build.gradle** file.

```groovy
...
dependencies {
    ...
    implementation "com.google.dagger:hilt-android:$hilt_version"
    kapt "com.google.dagger:hilt-android-compiler:$hilt_version"
}
```



## Hilt in Application

```kotlin
@HiltAndroidApp
class LogApplication: Application() {
    ...
}
```

We assume the servicelocator is attached to the app's lifecycle, so can annotate the application class.

`@HiltAndroidApp` triggers Hilt's code generation. 

The application container is the parent container of the app.



## Setup Entry Point

```kotlin
@AndroidEntryPoint
class LogsFragment : Fragment() { ... }
```

` @AndroidEntryPoint`  creates a dependencies container which follows the Android class lifecycle.

* Hilt supports **Application** (`@HiltAndroidApp`) , **Activity**, **Fragment**, **View**,  **Service** and **BroadcastReceiver**.
* Note that activities extending **FragmentActivity** (like **AppCompatActivity**) and fragment extending the Jetpack Library **Fragment**, not the platform one!



## Field Injection

```kotlin
@AndroidEntryPoint
class LogsFragment : Fragment() {

    @Inject lateinit var logger: LoggerLocalDataSource
    @Inject lateinit var dateFormatter: DateFormatter
  
    ... 
}

class DateFormatter @Inject constructor() {
  ...
}

class LoggerLocalDataSource @Inject constructor(private val logDao:LogDao) {
  ...
}
```

Initially the code to initiliaze and populate the dependency was in the  ```onAttach()``` , but after annotating the fields with ```@Inject```, we don't need to populate by ourselves any more.

Secondly we need to let Hilt know where to inject instance to, so we will add ```@Inject ``` to the constructor of the class we want to be injected.

* Binding: The information about how to provide instances of different types.
* So This ```LogsFragment```  has two bindings.

Hilt automatically creates and destroys instance of generated component classes following the lifecycle of the corresponding Android classes.

| Generated Component       | Created At             | Destroyed At            |
| ------------------------- | ---------------------- | ----------------------- |
| ApplicationComponent      | Application#onCreate() | Application#onDestroy() |
| ActivityRetainedComponent | Activity#onCreate()    | Activity#onDestroy()    |
| ActivityComponent         | Activity#onCreate()    | Activity#onDestroy()    |
| FragmentComponent         | Fragment#onAttach()    | Fragment#onDestroy()    |
| ViewComponent             | View#super()           | View destroyed          |
| ViewWithFragmentComponent | View#super()           | View destroyed          |
| ServiceComponent          | Service#onCreate()     | Service#onDestroy()     |



## Scoping instances to containers

As Hilt generates different containers that have different lifecycles, there are different annotations that scopes to those containers.

```kotlin
@Singleton
class LoggerLocalDataSource @Inject constructor(private val logDao: LogDao) {
    ...
}
```

| Android class                             | Generated component       | Scope                  |
| :---------------------------------------- | :------------------------ | :--------------------- |
| Application                               | ApplicationComponent      | @Singleton             |
| ViewModel                                 | ActivityRetainedComponent | @ActivityRetainedScope |
| Activity                                  | ActivityComponent         | @ActivityScoped        |
| Fragment                                  | FragmentComponent         | @FragmentScoped        |
| View                                      | ViewComponent             | @ViewScoped            |
| View annotated with @WithFragmentBindings | ViewWithFragmentComponent | @ViewScoped            |
| Service                                   | ServiceComponent          | @ServiceScoped         |



## Hilt Modules

``` kotlin
@InstallIn(ApplicationComponent:class)
@Module
object DatabaseModule {
    @Provides
    @Singleton // It has to be unique instance throughout the app
    fun provideDatabase(@ApplicationContext appContext: Context): AppDatabase {
        return Room.databaseBuilder(
            appContext,
            AppDatabase::class.java,
            "logging.db"
        ).build()
    }
  
    @Provides
    fun provideLogDao(database: AppDatabase): LogDao {
        return database.logDao()
    }
}
```

A Hilt module is a class annotated with `@Module ` and `@InstallIn` .

* `@Module` tells Hilt this is a module.
* `@InstallIn` tells Hilt in which containers the bindings are available by specifying a Hilt Component.

We can think of a Hilt Component as a container, and find the mapping table between Component and Injector below.

| Hilt component            | Injector for                                 |
| :------------------------ | :------------------------------------------- |
| ApplicationComponent      | Application                                  |
| ActivityRetainedComponent | ViewModel                                    |
| ActivityComponent         | Activity                                     |
| FragmentComponent         | Fragment                                     |
| ViewComponent             | View                                         |
| ViewWithFragmentComponent | View  annotated with `@WithFragmentBindings` |
| ServiceComponent          | Service                                      |

Each Hilt container comes with a set of default bindings that can be injected as dependencies into your custom bindings. This is the case of the `applicationContext`: to access it, you need to annotate the field with `@ApplicationContext`.

| Android component         | Default bindings                      |
| :------------------------ | :------------------------------------ |
| ApplicationComponent      | Application                           |
| ActivityRetainedComponent | Application                           |
| ActivityComponent         | Application, Activity                 |
| FragmentComponent         | Application, Activity, Fragment       |
| ViewComponent             | Application, Activity, View           |
| ViewWithFragmentComponent | Application, Activity, Fragment, View |
| ServiceComponent          | Application, Service                  |



# Providing interfaces with @Binds

```kotlin
@InstallIn(ActivityComponent::class)
@Module
abstract class NavigationModule {

    @Binds
    abstract fun bindNavigator(impl: AppNavigatorImpl): AppNavigator
}
```

To tell Hilt what implementation to use for an interface, you can use the `@Binds` annotation on a function inside a Hilt module.

* `@Binds` must annotate an abstract function.
* Functions with `@Binds` annotated takes the implementation of the interface we want to provide, and returns the interface itself.

```kotlin
class AppNavigatorImpl @Inject constructor(
    private val activity: FragmentActivity
) : AppNavigator {
    ...
}
```

```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject lateinit var navigator: AppNavigator

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        if (savedInstanceState == null) {
            navigator.navigateTo(Screens.BUTTONS)
        }
    }

    ...
}
```



We've covered the fundamentals for Hilt!
