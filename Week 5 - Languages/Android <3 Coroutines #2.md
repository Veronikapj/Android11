# Android <3 Coroutines #2

Link: https://youtu.be/6manrgTPzyA



## Overview

Goal: Learn why coroutines are the recommended solution for async code



## Kotlin Flows

		1. Model streams of the data, not the single values.

2. Built upon the foundation of coroutines and suspend function

   * Cancellation

   * Structure Concurrency

   * Exception Transparency

   * Natural back pressure handling

   * Here, once the database is changed, it automatically updates the UI in the reactive way. (Observing the flow lets **the app react to changes** when the DB emits new data)

     ```kotlin
     @Dao
     interface UsersDao {
     		@Query("SELECT * FROM users")
     		fun getUsers(): Flow<List<Users>>
     }
     ```

3. Comparison to the `RxJava` and the `LiveData`

   * Conceptually it's not different from the RxJava.

   * To `LiveData` 

     * Short Answer: little difference

     * Full Answer: more complicated

       

4. `LiveData` vs `Flow`

   ```kotlin
   // LiveData
   MutableLiveData<T>:
   	var value:T
   	fun postValue(value:T)
   		
   LiveData<T>:
   	val value:T
   
   // Flow
   flow { emit(...) }
   ```

   * **State holder** vs **cold data stream**

   * Conceptually `LiveData` is a value holder, so the most important thing is to observe the current state.

   * As UI only sees the latest result, hence it’s fine to use `LiveData`.

   * `Flow` observes the ‘current’ of the data, so it’s more appropriate for the lower layer of the app

     

5. `StateFlow`

   ```kotlin
   // LiveData
   MutableLiveData<T>:
   	var value:T
   	fun postValue(value:T)
   		
   LiveData<T>:
   		val value:T
   
   // StateFlow
   MutableStateFlow<T>:
   	val value:T
   	
   StateFlow<T>:
   	val value:T
   ```

   * **State holder** vs  **...a Flow, for states**

   * Found in the latest coroutine version
   * It holds the current value like LiveData
   * But LiveData has some drawbacks like being able to be observed only in the main thread.
   * Trying converting one over the other, as converting is easy from regular Flow to LiveData or vice versa.

6. `ViewModelScope` ,  ` CoroutineScope`

   ```kotlin
   // Observing a LiveData in an Activity
   myFlow.asLiveData().observe( ... ) 
   // cancels when there are no active observers
   
   // Collection a Flow in an Activity
   lifecycleScope.launchWhenStarted { myFlow.collect { ... }}
   // pauses when not STARTED, cancels when Activity is destroyed
   ```

   * `LifecycleScope` for Activities and Fragments
     * When to cancel the livedata and flow?

7. `callbackFlow`,  **Hot source, cold flow**

   ```kotlin
   fun FusedLocationProviderClient.locationFlow() = callbackFlow<Location>{
   		val callback = object : LocationCallback() {
   				override fun onLocationResult(result: LocationResult?) {
   						result ?: return
   						try { offer(result.lastLocation)} catch (e: Exception) {}
   				}
   		}
   		requestLocationUpdates(createLocationRequest(), callback, Looper.getMainLooper())
   				.addOnFailureListener { e ->
   						close(e) // in case of exception, close the Flow
   				}
   		// clean up when Flow collection ends
   		awaitClose {
   				removeLocationUpdates(callback)
   		}
   }
   ```

   * A builder function that lets us convert callback or listener-based API to Flow.
   * Hot source: Can it be understood as Hot Observable (?)
   * Cold flow: the code block will not be executed until the flow is collected. 
   * So, even if the activity stays onStop, producers are still alive, which means it causes battery problems.

   | MyActivity | myFlow.asLiveData()  | lifecycleScope.launchWhenStarted { ... } |
   | ---------- | -------------------- | ---------------------------------------- |
   | onStart    | Collector: Active    | Collector: Active                        |
   |            | Producer: Active     | Producer: Active                         |
   | onStop     | Collector: Cancelled | Collector: Paused                        |
   |            | Producer:  Cancelled | Producer: Active                         |

   8. `SharedFlow` and **.sharedIn()** for cold Flows

   ```kotlin
   fun <T> Flow<T>.shareIn(
   		scope: CoroutineScope,
   		replay: Int,
   		started: SharingStarted = SharingStarted.Eagerly,
   		initialValue: T = NO_VALUE as T
   ): SharedFlow<T>
   
   // Only keep upstream active when someone's listening
   SharingStarted.WhileSubscribed
   ```

   * It can take the flow, and be shared among multiple subscribers.
   * Useful when the flow is expensive to create
   * WhileSubscribed option: free up the upstream producer whenever no subscribers.

