# What's New in Jetpack

### Overview

1. Suites of libraries to make developers build app easily

2. Since Google launched the Jetpack, the adoption rate has increased up to 47 percent (surveyed on top 1000 apps, at least one or two libraries of jetpack has been used)

   

### Hilt

1. Dependency Injection Library built on the Dagger

2. Why DI?

   * Greater code reusability due to decoupling between components.
   * And ease of testing
   * 49 % of developers asked for a DI solution

3. Why Dagger?

   * 74% of top 10K apps adopted Dagger.
   * Steep learning curve -> background of Hilt.

4. Hilt

   * Designed for Android, it knows Android components like ViewModel and its scope

   * Provides new annotations, and pre-defined scope especially about Android

   * Installing a Dagger Module is also available. Hilt will discover it automatically.

     ```kotlin
     @AndroidEntryPoint
     class SearchFragment:Fragment(){
         @Inject
         lateinit var foo:Foo
         val viewModel:SearchViewModel by viewModels<>()
     }
     
     class SearchViewModel @ViewModelInject constructor(
       private val database:MyDatabase
     ): ViewModel()
     
     @AndroidEntryPoint
     class MainActivity: AppCompatActivity()
     
     @AndroidEntryPoint
     class MyService: Service()
     
     @HiltAndroidApp
     class MyApplication: Application()
     ```

     ```kotlin
     @InstallIn(ApplicationComponent::class)
     @Module
     object AppModule {
         @Provides
         fun provideDb(app:Application):MyDatabase {
             ...
         }
     }
     ```

   * Hilt in Actions - e.g Google I/O app. Google team saw the great amount has been reduced from DI code by using Hilt. (75%)
   * Integrated with Jetpack(ViewModel, Fragment, and WorkManager), Scopes for the Android, Box of test Apis, and android studio integration
   * Alpha stage

### App Startup

1. Faster application initialization, to reduce negative effect on launch performance
2. Content Provider to initialize automatically, including jetpack libraries like workmanager and lifecycle.
3. Provides a straightforward and performant way.
4. Single ContentProvider shared between all these initializers reducing the application startup time.
5. Automatically added Trace Point to figure out the performance.

```kotlin
class WorkManagerInitializer : Initializer<WorkManager> {
    override fun create(context:Context):WorkManager {
        val configuration = Configuration.Builder()
    		.setMinimumLoggingLevel(Log.DEBUG)
    		.build()
    	WorkManager.initialize(context, configuration)
        return WorkManager.getInstance(context)
    }
	
    override fun dependencies(): List<Class<out Initializer<*>> = emptyList()
}
```



### Android Game SDK

1. Gaming support now in Jetpack!
2. Two important modules in Gaming SDK

   * Frame pacing API - to maintain a steady frame rate and lower input latency (detect the expected frame rate and auto-adjust frame presentation times)

   * Performance Tuner - performance in Android Vital

     

### Benchmark 1.1

1. Support for CPU profiling benchmarks
2. Memory allocation developer makes



### Paging 3

1. Kotlin / Coroutines and Flow
2. Headers, Footers, and Separator
3. Loading State and RetryCompatible with Paging 2
4. PagingSource, Pager, and Presenting
5. Alpha, and First Kotlin-based library



### CameraX

1. Beta Stage, focusing on reliability and documentation
2. Runs on 400M Devices in use. 
3. PreviewView
   * camera preview, interactions with lifecycle.
   * Less buffering, and better power efficiency
4. YUV to RGB conversation to do image analysis



### WorkManager

1. Allowing us to run deferrable background job
2. Not relying on Jobscheduler, but in-process scheduler
3. Now supports Delayed workers and periodic work requests
4. No longer imposes scheduling limits -> improves throughput of work requests.
5. Supports long-running work that should be kept alive by OS. <- Using Foreground Service. (more than 10 minutes. But have to show notification.)
6. Diagnostics
   * New Diagnostics Api that we can invoke with ADB.
   * Can dump diagnostics to logcat

### Navigation

1. allows us to navigate between different screens of the app easily
2. Navigation 2.3

   * Supports Dynamic feature modules, with corresponding classes annotated.

   * Deep Linking Improvement - by using one parameter in the graph.

   * Returning Result

     * Each screen in the app has a NavBackStackEntry.
     * It gives us access to the same state of that entry as well.
     * Navigation uses the SavedStateHandle class to pass data between screens, to ensure that results are kept even over configuration changes.
     * Our fragment can access the previous fragment SaveStateHandle by using the previous BackStackEntry.
     * Once we obtain the saveStateHandle from the previous entry, we can set the result values on the SavedState.
     * To observe the result, we can get the same value from the SavedState of the currentBackStackEntry and observe the value in livedata. It means observation is lifecycle aware.

     ```kotlin
     val savedStateHandle = navController.previousBackStackEntry?.savedStateHandle
     // Set the result
     savedStateHandle?.set("key",  result)
     
     override fun onViewCreated(view:View, savedInstanceState:Bundle?){
     	val savedStateHandle = findNavController().currentBackStackEntry?.savedStateHandle
     	savedStateHandle?.getLiveData<String>("key")?.observe(viewLifecycleOwner) {
         // Do something with the result
       }
     }
     ```

   

### Permissions and Activity

1. ActivityResultContracts API - Activity 1.2.0 alpha2
   * Replacing startActivityForResult
   * Type safe contracts for common intent
   * Not only requesting permission (single or multiple), but also replacing some jobs using intent such as taking pictures, querying the contacts.



### AppCompat

1. Lint Rule
   * Library-specific Lint rule
     * AS automatically shows the warning for users to replace the attribute to the one available in AppCompat. (in xml)
   * AppCompat and Dark Mode
     * More reliable Dark Theme
     * Configuration Override API for users to customize their theme



