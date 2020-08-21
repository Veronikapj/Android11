# Support for newer Java language APIs



## Using Java 8+ APIs on Android Devices

1. A lot of crashes on devices with the version < 26.
2. Why? The reason is just the **device didn’t ship the classes necessary for the newer Java API**.
3. Android 11 supports a number of APIs from newer OpenJDK. (up to 13)
4. **Android Gradle Plugin 4.0.0 and newer** one can enable us to use tons of new APIs from newer OpenJDK.
5. Developers can use some of these newer Java APIs in Android 11 for older devices (Basically through **backporting, and desugaring** on older devices where the Android platform doesn’t have the APIs on the runtime)



## D8/R8 Desugaring

1. The Android Gradle Plugin 4.0

   * Built-in support for using certain Java language APIs and 3rd-party libraries
   * With this, almost all versions of Android are supported.

2. Desugaring performed by D8/R8

   * Java Source code to Java bytecode [By Java Compiler]
   * Implementation of the new APIs [Bytecode Transformations] 
   * Bytecode to Dex Library (The necessary Java 8 runtime code is separate dex library)
   * This process is called **desugaring!** (a set of Java 8 APIs enabled on all existing devices, except parallel (supported from 21)

3. How to set up the desugaring in build.gradle. 

   * (Don’t forget to version up your AGP to 4.0 and above!)
   * If minSDK is 20 or lower, declare **multiDexEnabled as true**.

   ```groovy
   android {
   		defaultConfig {
   				multiDexEnabled true
   		}
   		
   		compileOptions {
   				coreLibraryDesugaringEnabled true
           sourceCompatibility JavaVersion.VERSION_1_8
           targetCompatibility JavaVersion.VERSION_1_8
   		}
   		
   		dependencies {
   				coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:1.0.5'
   		}
   }
   ```



## Added newer APIs

1. `java.time` - based on *JodaTime* library / no concurrency issues with immutability

   * If no need timezone data and Date and Time object

     * `LocalDate`
     * `LocalTime`

   * If requiring time zones

     * `ZoneDateTime` - immutable representation of `DateTime`

       * Hold `LocalDateTime` , `ZoneId` , ` ZoneOffset`
       * Useful when we need to store date and time (not relying on the device or app)
       * `ZoneOffsets` to calculate the difference between current one and UTC

     * `OffsetDateTime` - keeps the offset from Greenwich/UTC instead of `ZoneId`

       ```java
       ZoneDateTime currentDate = ZoneDateTime.now();
       System.out.println("the current zone is " + currentDate.getZone());
       // the current zone is America/Los_Angeles
       
       ZoneId anotherZone = ZoneId.of("Europe/Istanbul");
       ZoneDateTime currentDateInIstanbul = currentZone.withZoneSameInstant(anotherZone);
       ```

   * Summarize

     * `ZoneDateTime` is good for **displaying timezone-sensitive datetime data**
     * `OffsetDateTime` is best for **storing date time data** to a database / other use cases where the data needs to be **serialized**

   * `Period` and `Duration`

     ```java
     LocalDate new = LocalDate.now();
     LocalDate androidBeta = LocalDate.of(2020, 6, 10);
     Period period = Period.between(androidBeta, now);
     ```

2. `java.util.stream`

   * It allows us to perform **functional-style operations on collections**.

   * Intermediate operations / Terminal operations

   * It may perform better when working on large data sources.

     ```java
     List<Person> list = Arrays.asList(personArray);
     Stream<Person> stream1 = list.stream();
     
     // or
     Stream<Person> stream2 = Stream.of(personalArray);
     
     // or
     Stream.Builder<Person> streamBuilder = Stream.builder();
     for (Person p : personArray) {
     		streamBuilder.accept(p);
     }
     Stream<Person> stream3 = streamBuilder.build();
     ```

3. Other Java APIs

   1. Map, Collection, Comparator

      * Can be simple to handle Lambda expression

        ```java
        import static java.util.Comparator.comparing;
        
        comparing(Person::getFirstName).thenComparing(Person::getSurname);
        ```

   2. Optional and its counterpart for primitive types

      * `Optional<Object>`

      * `OptionalInt` , `OptionalLong` , `OptionalDouble`

      * Robust to null as well as Kotlin does.

        ```java
        Optional<String> opt = Optional.of("Java 8");
        if (opt.isPresent()) { // true
        		...
        }
        
        opt = Optional.ofNullable(this);
        if (opt.isEmpty()) {  // true
        	...
        }
        
        opt.ifPresent(str -> System.out.println(str)); // false, doesn't print
        ```

   3. Atomic classes in `java.util.concurrent.atomic` package

      * AtomicInteger`, `AtomicLong`, `AtomicReference`

   4. `ConcurrentHashMap` (Thread-safe alternative to HashMap)

      * ConcurrentHashMap on Android 21 and 22 had a bug.

      * Fixed version is in the desugared libraryAllows any number of thread to perform operations

      * **Performing thread must lock the particular segment** where the data is being processed when updating or inserting data.

      * Due to the synchronization, performance is slower than a HashMap.

        

## Full listups

1. https://developer.android.com/studio/write/java8-support-table