# Dependency Injection on Android with Hilt

Created By: PilJu BAE
Last Edited: Aug 13, 2020 4:29 PM
Links: https://medium.com/androiddevelopers/dependency-injection-on-android-with-hilt-67b6031e62d
events: android11

### [Dependency injection (DI)](https://developer.android.com/training/dependency-injection)

- By following DI principles, you lay the groundwork for good app architecture, greater code reusability, and ease of testing.

# [Hilt](https://developer.android.com/training/dependency-injection/hilt-android)

- [dagger](https://developer.android.com/training/dependency-injection/dagger-basics) 기반
- 컴파일 시간 정확성, 런타임 성능, 확장 성 및 Android Studio 지원
- Dagger and Hilt can coexist together
    - [migration guide](https://dagger.dev/hilt/migration-guide)

### automatically generates and provides

- **Components for integrating Android framework classes** with Dagger that you would otherwise need to create by hand.
- **Scope annotations** for the components that Hilt generates automatically.
- **Predefined bindings and qualifiers**.

## Setting

```groovy
//root build.gradle
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
    }
}
```

```groovy
// app/build.gradle
...
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}

dependencies {
    implementation "com.google.dagger:hilt-android:2.28-alpha"
    kapt "com.google.dagger:hilt-android-compiler:2.28-alpha"
}
```

## Hilt in action

```kotlin
// 1. application
@HiltAndroidApp
class MyApplication : Application() { ... }
```

```kotlin
// 2. Adapter @Inject 
class AnalyticsAdapter @Inject constructor() { ... }
```

```kotlin
// 3. Activity
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
  @Inject lateinit var analytics: AnalyticsAdapter
  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    // analytics instance has been populated by Hilt
    // and it's ready to be used
  }
}
```

Android 클래스에 @AndroidEntryPoint로 주석을 지정하면 이 클래스에 종속된 Android 클래스에도 주석을 지정해야 합니다. 예를 들어 프래그먼트에 주석을 지정하면 이 프래그먼트를 사용하는 활동에도 주석을 지정해야 합니다.

## [Jetpack Support](https://developer.android.com/training/dependency-injection/hilt-jetpack)

- support for ViewModel and WorkManager

    → @ViewModelInject

## Hilt Annotations

[hilt-annotations.pdf](res/hilt-annotations.pdf)