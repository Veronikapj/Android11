# Using Hilt in your Android App (2)

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



## Qualifiers

We can define an interface and implement it  when we want to abstract the common attribute. 
But it's obvious that the implementations have different functionalities under same name of methods. 
How can we handle this in Hilt?

```kotlin
// Common interface for Logger data sources.
interface LoggerDataSource {
    fun addLog(msg: String)
    fun getAllLogs(callback: (List<Log>) -> Unit)
    fun removeLogs()
}
```

```kotlin
@Singleton
class LoggerLocalDataSource @Inject constructor(
    private val logDao: LogDao
) : LoggerDataSource {
    ...
    override fun addLog(msg: String) { ... }
    override fun getAllLogs(callback: (List<Log>) -> Unit) { ... }
    override fun removeLogs() { ... }
}
```

```kotlin
@ActivityScoped
class LoggerInMemoryDataSource @Inject constructor() : LoggerDataSource {

    private val logs = LinkedList<Log>()

    override fun addLog(msg: String) {
        logs.addFirst(Log(msg, System.currentTimeMillis()))
    }

    override fun getAllLogs(callback: (List<Log>) -> Unit) {
        callback(logs)
    }

    override fun removeLogs() {
        logs.clear()
    }
}
```

```kotlin
@Qualifier
annotation class InMemoryLogger

@Qualifier
annotation class DatabaseLogger

@InstallIn(ApplicationComponent::class)
@Module
abstract class LoggingDatabaseModule {

    @Singleton
    @Binds
    abstract fun bindDatabaseLogger(impl: LoggerLocalDataSource): LoggerDataSource
}

@InstallIn(ActivityComponent::class)
@Module
abstract class LoggingInMemoryModule {

    @ActivityScoped
    @Binds
    abstract fun bindInMemoryLogger(impl: LoggerInMemoryDataSource): LoggerDataSource
}
```

`@Binds` methods must have the scoping annotations if the type is scoped, so that's why the functions above are annotated with `@Singleton` and `@ActivityScoped`.



So we can annotate this implementation of `LoggerDataSource` like below.

```kotlin
@AndroidEntryPoint
class LogsFragment : Fragment() {
  
    @InMemoryLogger
    @Inject lateinit var logger: LoggerDataSource
    ...
}
```

Or if we use Database module instead, see below.

```kotlin
class LogsFragment : Fragment() {
  
    @DatabaseLogger
    @Inject lateinit var logger: LoggerDataSource
    ...
}
```



## @EntryPoint annotation

`@EntryPoint` is to inject dependencies into classes which is not supported by Hilt. 

Use case this codelab provides is that making our database accessible from outside the process of this app. It means we should use `ContentProvider`.



```kotlin
interface LogDao {
    ...
    @Query("SELECT * FROM logs ORDER BY id DESC")
		fun selectAllLogsCursor(): Cursor

    @Query("SELECT * FROM logs WHERE id = :id")
    fun selectLogById(id: Long): Cursor?
}
```

Define the contentProvider.

```kotlin
private const val LOGS_TABLE = "logs"

/** The authority of this content provider.  */
private const val AUTHORITY = "com.example.android.hilt.provider"

/** The match code for some items in the Logs table.  */
private const val CODE_LOGS_DIR = 1

/** The match code for an item in the Logs table.  */
private const val CODE_LOGS_ITEM = 2

/**
 * A ContentProvider that exposes the logs outside the application process.
 */
class LogsContentProvider: ContentProvider() {

    private val matcher: UriMatcher = UriMatcher(UriMatcher.NO_MATCH).apply {
        addURI(AUTHORITY, LOGS_TABLE, CODE_LOGS_DIR)
        addURI(AUTHORITY, "$LOGS_TABLE/*", CODE_LOGS_ITEM)
    }

    override fun onCreate(): Boolean {
        return true
    }

    /**
     * Queries all the logs or an individual log from the logs database.
     *
     * For the sake of this codelab, the logic has been simplified.
     */
    override fun query(
        uri: Uri,
        projection: Array<out String>?,
        selection: String?,
        selectionArgs: Array<out String>?,
        sortOrder: String?
    ): Cursor? {
        val code: Int = matcher.match(uri)
        return if (code == CODE_LOGS_DIR || code == CODE_LOGS_ITEM) {
            val appContext = context?.applicationContext ?: throw IllegalStateException()
            val logDao: LogDao = getLogDao(appContext)

            val cursor: Cursor? = if (code == CODE_LOGS_DIR) {
                logDao.selectAllLogsCursor()
            } else {
                logDao.selectLogById(ContentUris.parseId(uri))
            }
            cursor?.setNotificationUri(appContext.contentResolver, uri)
            cursor
        } else {
            throw IllegalArgumentException("Unknown URI: $uri")
        }
    }

    override fun insert(uri: Uri, values: ContentValues?): Uri? {
        throw UnsupportedOperationException("Only reading operations are allowed")
    }

    override fun update(
        uri: Uri,
        values: ContentValues?,
        selection: String?,
        selectionArgs: Array<out String>?
    ): Int {
        throw UnsupportedOperationException("Only reading operations are allowed")
    }

    override fun delete(uri: Uri, selection: String?, selectionArgs: Array<out String>?): Int {
        throw UnsupportedOperationException("Only reading operations are allowed")
    }

    override fun getType(uri: Uri): String? {
        throw UnsupportedOperationException("Only reading operations are allowed")
    }
  
	  private fun getLogDao(appContext: Context): LogDao {
      	val hiltEntryPoint = EntryPointAccessors.fromApplication(
            appContext,
            LogsContentProviderEntryPoint::class.java
        )
        return hiltEntryPoint.logDao()
    }
}	
```

`@EntryPoint` is an interface with an accessor method for each binding type we want. Also the interface has to be annotated with `@InstallIn` to specify the component in which to install the entry point.
