# Use Kotlin Coroutines in your Android App

Created By: PilJu BAE
Last Edited: Aug 9, 2020 6:22 PM
Links: https://developer.android.com/codelabs/kotlin-coroutines?authuser=1&return=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fandroid-week5-languages%3Fauthuser%3D1%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fkotlin-coroutines#0
events: Code, android11

# Overview

- 비동기 프로그래밍을 단순화하기 위한 UI 및 WorkManager 작업에서 코 루틴을 Android 앱에 통합하는 방법,
- **`ViewModel`** 내부에서 코 루틴을 사용하여 네트워크에서 데이터를 가져 와서 메인 스레드를 차단하지 않고 데이터베이스에 저장하는 방법.
- **`ViewModel`** 완료 되면 모든 코 루틴을 취소하는 방법 .
- 코 루틴 base 코드를 테스트 하는 방법과 suspend 함수를 테스트에서 호출하는 방법

---

# Setting

build.gradle

- **kotlinx-coroutines-core** — Main interface for using coroutines in Kotlin
- **kotlinx-coroutines-android** — Support for the Android Main thread in coroutines
- **lifecycle-viewmodel-ktx** — a CoroutineScope to ViewModels that's configured to start UI-related coroutines

```
dependencies {
  ...
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:x.x.x"
  implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android:x.x.x"

  implementation "androidx.lifecycle : lifecycle-viewmodel-ktx : xxx"
}
```

---

# Coroutines in Kotlin

### The Callback pattern

- 주 스레드를 차단하지 않고 장기 실행 작업을 수행하는 한 가지 패턴
- 백그라운드 작업이 완료되면 콜백이 호출되어 주 스레드에 결과를 알려줌

## Using coroutines to remove callbacks

- Callback을 많이 사용하는 코드는 읽기 어렵고 추론하기가 어려움
- 예외와 같은 일부 언어기능의 사용을 허용하지 않음
- Coroutine 을 통해 Callback 기반 코드를 순차 코드로 변환
- **suspend**
    - Coroutine 에 사용할 수 있는 함수 또는 함수 유형을 표시
    - 해당 함수가 일반 함수 호출처럼 반환 될 때까지 차단하는 대신 결과가 준비될 때까지 실행을 중단한 다음 결과와 함께 중단된 지점에서 다시 시작
    - 결과를 기다리면서 일시 중단되는 동안 다른 함수나 Coroutine 이 실행될 수 있도록 실행중인 스레드를 차단 해제
    - Important: The suspend keyword doesn't specify the thread code runs on. Suspend functions may run on a background thread or the main thread.
- 참고 : async, await in other languages

    suspend 는 async 와 비슷

    Kotlin : await() is implicit suspend function

---

# Controlling the UI with coroutines

## Understanding CoroutineScope

- Kotlin 에서, 모든 코루틴은 CoroutineScope 안에서 동작
- Scope는 job 을 통해 Coroutine lifetime을 제어
- ViewModelScope

    Dispatchers.Main 에 바인딩, ViewModel 이 cleared 되는 시점에 자동으로 취소됨.

## Switch from threads to coroutines

백그라운드 스레드 사용

```kotlin
/**
* Wait one second then update the tap count.
*/
private fun updateTaps() {
   // TODO: Convert updateTaps to use coroutines
   tapCount++
   BACKGROUND.submit {
       Thread.sleep(1_000)
       _taps.postValue("$tapCount taps")
   }
}
```

Coroutine 사용

```kotlin
/**
* Wait one second then display a snackbar.
*/
fun updateTaps() {
   // launch a coroutine in viewModelScope
   viewModelScope.launch {
       tapCount++
       // suspend this coroutine for one second
       delay(1_000)
       // resume in the main dispatcher
       // _snackbar.value can be called directly from main thread
       _taps.postValue("$tapCount taps")
   }
}
```

1. ViewModelScope.launch

    ViewModelScope 에 전달한 job이 취소되면, 이 job/scope 안의 모든 Coroutine 이 취소됨

    Delay 전에 Activity를 떠난 경우 ViewModel.onCleared() 시점에 취소

2. Default dispatcher = Dispatchers.Main

    메인 스레드에서 시작

3. delay() 는 suspend 함수

    메인 스레드에서 실행되지만 delay 1초 동안 스레드를 차단하지 않음. 대신 코루틴이 다음 명령문에서 1초 안에 재개되도록 예약

## How to Test

```kotlin
   @get:Rule
   val coroutineScope =  MainCoroutineScopeRule()
```

- Dispatchers.Main 을 사용하도록 구성
- 테스트를 위한 vitual-clock 구성하여 테스트
- coroutineScope.advanceTimeBy(1000)
- Dispatchers.Main 에 값이 설정될 때 까지 1초를 기다릴 필요가 없음

## Exceptions in suspend functions

- 일반 function exception 처럼 작동
- 호출자에게 오류를 전달하기 때문에 일반 try/catch 블록을 사용
- Coroutine 에서 예외를 던지면 해당 coroutine 이 기본적으로 부모를 취소 → 관련된 작업을 함께 취소하는 것이 쉬움
- uncaught exception

    CoroutineScope 의 uncaught exception handler 로 전달

---

# Making main-safe functions from blocking code

## Calling blocking calls from coroutines

- 큰 목록의 정렬 및 필터링 또는 디스크에서 읽기와 같은 코루틴 내부에서 차단 또는 CPI의 집약적인 작업을 수행해야 할때 이 패턴 사용
- 가능하면 Room 또는 Retrofit 과 같은 라이브러리의 일반 suspend 함수를 사용하는 것이 좋음

### Kotlin Coroutine Dispatcher

- IO dispatcher : 네트워크 또는 디스크 읽기와 같은 IO 작업에 최적화
- Default : CPU 집약적인 작업에 최적화
- Main

```kotlin
suspend fun refreshTitle() {
   // interact with *blocking* network and IO calls from a coroutine
   withContext(Dispatchers.IO) {
       val result = try {
           // Make network request using a blocking call
          //네트워크 요청 실행 
					network.fetchNextTitle().execute()
       } catch (cause: Throwable) {
           // If the network throws an exception, inform the caller
           throw TitleRefreshError("Unable to refresh title", cause)
       }
      
       if (result.isSuccessful) {
           // Save it to database 성공하면 데이터베이스에 저장
           titleDao.insertTitle(Title(result.body()!!))
       } else {
           // If it's not successful, inform the callback of the error
           throw TitleRefreshError("Unable to refresh title", null)
       }
   }
}
```

1. execute() 및 insertTitle() 은 [Dispatcher.IO](http://dispatcher.IO) 의 스레드 중 하나를 차단 
2. Dispatcher.Main 은 withContext 가 완료될 때까지 일시 중단
3. Callback 으로 구현 했을 때와 비교
    - withContext 를 호출한 Dispatcher.Main 으로 결과 반환
    - 호출자는 이 함수에 콜백을 전달할 필요가 없음
    - result 또는 Exception 을 얻기 위해 suspend 또는 resume 에 의존
4. withContext
    - this withContext block only calls *blocking* calls it will **not be cancelled** until it returns from withContext.
    - yield() 를 정기적으로 호출해서 취소 여부를 확인할 수 있음.

---

# Coroutines in Room & Retrofit

Room 과 Retrofit 은 자동으로 main-safe 한 suspend function 을 생성 → Dispatcher.Main 에서 호출 가능 

Room 과 Retrofit 은 [Dispatchers.I](http://dispatchers.ID)O 를 사용하지 않고 커스텀 dispatcher 를 사용

### To use suspend functions with Retrofit

1. 함수에 suspend modifier 추가
2. Remove the **Call** wrapper from the return type

```kotlin
suspend fun refreshTitle() {
   try {
       // Make network request using a blocking call
       val result = network.fetchNextTitle()
       titleDao.insertTitle(Title(result))
   } catch (cause: Throwable) {
       // If anything throws an exception, inform the caller
       throw TitleRefreshError("Unable to refresh title", cause)
   }
}
```

- withContext 가 제거됨.
- main-safe suspending function 을 부르는데 withContext 를 사용할 필요가 없음
- 컨벤션에 따라 suspend 함수가 main-safe 한 것을 보장 하기만 하면 됨, Dispatchers.Main 에서 호출 하더라도 안전하다.

## How to Test

```kotlin
@Test
fun whenRefreshTitleSuccess_insertsRows() {
   val subject = TitleRepository(
       MainNetworkFake("OK"),
       TitleDaoFake("title")
   )
	 //suspend 함수이므로 바로 테스트 할 수 없음
   1. subject.refreshTitle()

   // launch starts a coroutine then immediately returns
   2. GlobalScope.launch {
       // since this is asynchronous code, this may be called *after* the test completes
       subject.refreshTitle()
   }
   // test function returns immediately, and
   // doesn't see the results of refreshTitle
}
```

- 테스트는 코루틴이 반환되기 전에 완료될 때까지 실행해야 함.
- 테스트 기능이 반환되면 테스트가 종료
- 2번과 같이 코루틴 launch 로 시작된 비동기 코드는 향후 어느 시점에서 완료될 수 있음
- launch 는 비 차단 호출이기 때문에 즉시 반환되고 함수가 반환된 후에도 코루틴을 계속 실행할 수 있음
- 때때로 실패하므로 테스트로 사용 불가
- refreshTitle 에 예외가 발생하면 테스트 호출스택에서 발생하지 않고 GlobalScope 의 uncaught exception handler 로 전달

```kotlin
@Test
fun whenRefreshTitleSuccess_insertsRows() = runBlockingTest {
   val titleDao = TitleDaoFake("title")
   val subject = TitleRepository(
           MainNetworkFake("OK"),
           titleDao
   )

   subject.refreshTitle()
   Truth.assertThat(titleDao.nextInsertedOrNull()).isEqualTo("OK")
}
```

### runBlockingTest

- suspend 함수를 부르는 동안 block 함
- new coroutine 이나 suspend function 을 기본적으로 즉시 호출
- uncaught exception 을 다시 던져준다
- 일반적인 함수 호출과 마찬가지로 runBlockingTest 는 caller 를 항상 블럭
- 코루틴은 같은 스레드에서 동기적으로 실행
- 코루틴에 runBlocking block 인터페이스를 제공하는데 사용
- 코루틴 테스트가 완료된 후 코루틴이 누출(leak)되지 않도록 한다.

    만약 테스트가 끝날 때, 끝나지 않은 코루틴이 있으면 테스트가 실패하게 된다

- TestCoroutineDispatcher, TestCoroutineScope

    Coroutine 을 단일 스레드로 만드는 효과가 있으며 테스트에서 모든 코루틴을 명시적으로 제어할 수 있는 기능을 제공

---

# Refactoring

```kotlin
//suspend lambda
private fun launchDataLoad(**block: suspend () -> Unit**): Job {
   return viewModelScope.launch {
       try {
           _spinner.value = true
           block()
       } catch (error: TitleRefreshError) {
           _snackBar.value = error.message
       } finally {
           _spinner.value = false
       }
   }
}

fun refreshTitle() {
   launchDataLoad {
       repository.refreshTitle()
   }
}
```

# Using coroutines with WorkManager

## WorkManager

- Uploading logs
- Applying filters to images and saving the image
- Periodically syncing local data with the network

---

# More

- [Advanced Coroutines with Kotlin Flow and LiveData](https://developer.android.com/codelabs/advanced-kotlin-coroutines/index.html?authuser=1#0)
- Article series
    - [Part 1: Coroutines](https://medium.com/androiddevelopers/coroutines-first-things-first-e6187bf3bb21)
    - [Part 2: Cancellation in coroutines](https://medium.com/androiddevelopers/cancellation-in-coroutines-aa6b90163629)
    - [Part 3: Exceptions in coroutines.](https://medium.com/androiddevelopers/exceptions-in-coroutines-ce8da1ec060c)
- [Coroutines guides](https://github.com/Kotlin/kotlinx.coroutines/blob/master/docs/coroutines-guide.md)
- [Improve app performance with Kotlin coroutines](https://developer.android.com/kotlin/coroutines?authuser=1)