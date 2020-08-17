# Android <3 Coroutines

Link: https://youtu.be/6manrgTPzyA



## Overview

Goal: Learn why coroutines are the recommended solution for async code



## The alternative of AsyncTask

  1. AsyncTask is finally deprecated in Android 11.

  2. Android official document says to use Concurrent Class of Java or Kotlin concurrency utilites (Kotlin Coroutines)

     

## Why coroutines?

1. The unique properties of coroutines -> make it apps well-suited

2. Structured Concurrency

   * Helps developers scope their works to application’s component
   * prevent memory leaks

3. Callback-free code

   * Higher readability and understandability

4. Cancellation Support and natural exception handling

5. Lightweight

   * Many coroutines run on the single thread, suspending their work instead of blocking

   * Making them fast and lightweight

     

## Practical Examples

1. `WorkManager` with synchronous job
   * But we can’t stop the work at any moment.

```kotlin
override fun doWork(): ListenableWorker.Result {
		try {
				val dbData = readFromDb()
				val serverData = uploadToServer(data)
				writeToDb(serverData)
		} catch (e: IOException) {
				return ListenableWorker.Result.failure()
		}
		
		return ListenableWorker.Result.success()
}
```

2. WorkManager` with code handling 'stopped case'
   * But how to pass the signal to stop?	

```kotlin
override fun doWork(): ListenableWorker.Result {
		try {
				val dbData = readFromDb()
	      if (isStopped()) return ListenableWorker.Result.retry()
				val serverData = uploadToServer(data)
      	if (isStopped()) return ListenableWorker.Result.retry()
				writeToDb(serverData)
		} catch (e: IOException) {
				return ListenableWorker.Result.failure()
		}
		
		return ListenableWorker.Result.success()
}
```

​	

3. The other way to do asynchronous request using `ListenableFuture`
   * Cancellation, and error propagation
   * But it's hard to find where the actual work is being done.

```kotlin
override fun startWork(): ListenableFuture<Result> {
		val futureDb: ListenableFuture<Data> = executor.submit { readFromDb()}
		val futureSrv = Futures.transformAsync(futureBd, { uploadToServer(it) }, executor)
  	return Futures.transformAsync(
    	futureSrv, {
        	writeToDb(it)
        	ListenableWorker.Result.success()
      },
    	executor
    )
```



4. A case using `RxWorker`
   * We still need a Java way, chaining calls.
   * Have to understand the operators of RxJava
   * Less readability

```kotlin
override fun createWork(): Single<Result> {
		return Single.just(readFromDb())
				.subscribeOn(Schedulers.io())
				.flatMap { dbData -> uploadToServer(dbData) }
				.flatMap { serverData -> writeToDatabase(serverData) }
				.map {
						Result.success()
				}
}
```

5. `CoroutineWorker` backed by Coroutines.
   * Everything looks sequential and is readable
   * Error propagation is more natural
   * Try-catch if needed

```kotlin
override suspend fun doWork(): Result {
		val dbData = readFromDb()
		val serverData = uploadToServer(data)
		writeToDb(serverData)
		return Result.success()
}
```



6. Use case in `ROOM` database of Android Jetpack

* Assuming that these operations shouldn’t be run in the main thread.
* Adopting the same approaches in Room and other Jetpack Libraries, and 3rd party libraries such as Retrofit.

```kotlin
@Dao
interface UsersDao {
		@Query("SELECT * FROM users")
		suspend fun getUsers(): List<User>
}
```



7. Suspend function is `main-safe`
   * safe to call from the main thread or the main dispatcher
   * Non-blocking the thread

```kotlin
suspend fun mySuspendFunction() = withContext(Dispatchers.IO) {
		// you can do blocking stuff here
}

// usage:
viewModelScope.launch {
		...
		mySuspendFunction()
		...
}
```



8. Executor's rule is almost identical to the coroutine Dispatchers.  
   So, There are methods to translate between them.

```kotlin
RoomDatabase.Builder.setQueryExecutor(executor: Executor)
RoomDatabase.Builder.setTransactionExecutor(executor: Executor)

// in coroutine library:
fun Executor.asCoroutineDispatcher(): CoroutineDispatcher
fun CoroutineDispatcher.asExecutor(): Executor
```



9. Use case of Dispatchers Injection

```kotlin
// provide a mechanism to configure the dispatcher somewhere
var dispatcher = Dispatchers.IO

suspend fun mySuspendFunction() = withContext(dispatcher) {
		// you can do blocking stuff here
}
```

10. Use case of common future APIs

```kotlin
// Awaits for completion of the completion stage without blocking a thread
suspend fun <T> CompletionStage<T>.await(): T // (for CompletableFuture)

// Awaits for completion of the ListenableFuture without blocking a thread
suspend fun <T> ListenableFuture<T>.await(): T

// Awaits for completion of this task without blocking a thread
suspend fun <T> Task<T>.await(): T
```



11. This `suspendCancellableCoroutine` is **to wrap** the callback-based libraries with **suspend functions**. It transforms callback functions to coroutine corresponding.

```kotlin
suspend fun <T> Task<T>.await(): T = 
		suspendCancellableCoroutine { continuation ->
				addOnSuccessListener { result ->
						continuation.resume(result)
				}
				addOnFailureListener { exception ->
						continuation.resumeWithException(exception)
				}
		}
```

```kotlin
suspend fun <T> Task<T>.await(): T {
		if (isComplete) { // eagerly resume coroutine if Task is ready
				if (isSuccessful) {
						return result
				} else {
						throw exception!!
				}
		}
		return suspendCancellableCoroutine { continuation ->
				addOnSuccessListener { result ->
						continuation.resume(result)
				}
				addOnFailureListener { exception ->
						continuation.resumeWithException(exception)
				}				
		}
}
```

