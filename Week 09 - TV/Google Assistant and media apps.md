# Google Assistant and media apps

* Link: https://developer.android.com/guide/topics/media-apps/interacting-with-assistant
* The Google Assistant lets us use voice command to control many devices, like Google Home, our phone and more, by built-in capabiliy to understand media commands. (It also supports media control such as pause, skip, etc.)
* **Media Session** can make the assistant communicate with Android media apps. (corresponding component: **Intent** and **Service** to launch our app and start playback.)

​	

## Use a media session

1. **Every audio and video app must implement a media session** so that the Assistant can run the transport controls once playback has started.

2. Enabling the media and transport controls by setting these flags.

   ```kotlin
   mediaSession.setFlags(
     MediaSessionCompat.FLAG_HANDLES_MEDIA_BUTTONS or
     MediaSessionCompat.FLAG_HANDLES_TRANSPORT_CONTROLS
   )
   ```

3. Using `setAction()` we have to declare the actions that our media session supports, and implement the corresponding media session callbacks.
4. [**Universal Android Music Player**](https://github.com/android/uamp) -> We can find the good example of how to set up a media session.

### Playback actions

1. What a media session must have.

   | Action                  | Callback             |
   | ----------------------- | -------------------- |
   | ACTION_PLAY             | `onPlay()`           |
   | ACTION_PLAY_FROM_SEARCH | `onPlayFromSearch()` |
   | ACTION_PLAY_FROM_URI(*) | `onPlayFromUri()`    |

2. **PREPARE** actions and their callbacks are required for our media session.

   | Action                     | Callback               |
   | -------------------------- | ---------------------- |
   | ACTION_PREPARE             | `onPrepare()`          |
   | ACTION_PREPARE_FROM_SEARCH | `onPrepareForSearch()` |
   | ACTION_PREPARE_FROM_URI(*) | `onPrepareFromUri()`   |

### Parse search queries

1. The `onPrepareFromSearch()` and `onPlayFromSearch()` callback receives a query parameter and extra bundle. (from user's query)

2. Sequence of app parsing the voice search query followed by 
   * Search query string and filter
   * Build a playback queue based on these results.
   * Play the most relevant item from the results.

3. `onPlayFromSearch()` method takes an extras parameter with more detailed information from the voice search.

   * These extras help us fine the content in our app for playback.
   *  If the search results are unable to provide this data, you can implement logic to parse the raw search query and play the appropriate tracks based on the query.

4. Supported extras in Android Automotive OS and Android Auto

   * EXTRA_MEDIA_ALBUM
   * EXTRA_MEDIA_ARTIST
   * EXTRA_MEDIA_GENRE
   * EXTRA_MEDIA_PLAYLIST
   * EXTRA_MEDIA_TITLE

5. Code snippet for how to override `onPlayFromSearch()` in `MediaSession.Callback` to parse the voice search query and begin playback.

   ```kotlin
   override fun onPlayFromSearch(query: String?, extras: Bundle?) {
       if (query.isNullOrEmpty()) {
           // The user provided generic string e.g. 'Play music'
           // Build appropriate playlist queue
       } else {
           // Build a queue based on songs that match "query" or "extras" param
           val mediaFocus: String? = extras?.getString(MediaStore.EXTRA_MEDIA_FOCUS)
           if (mediaFocus == MediaStore.Audio.Artists.ENTRY_CONTENT_TYPE) {
               isArtistFocus = true
               artist = extras.getString(MediaStore.EXTRA_MEDIA_ARTIST)
           } else if (mediaFocus == MediaStore.Audio.Albums.ENTRY_CONTENT_TYPE) {
               isAlbumFocus = true
               album = extras.getString(MediaStore.EXTRA_MEDIA_ALBUM)
           }
   
           // Implement additional "extras" param filtering
       }
   
       // Implement your logic to retrieve the queue
       var result: String? = when {
           isArtistFocus -> artist?.also {
               searchMusicByArtist(it)
           }
           isAlbumFocus -> album?.also {
               searchMusicByAlbum(it)
           }
           else -> null
       }
       result = result ?: run {
           // No focus found, search by query for song title
           query?.also {
               searchMusicBySongTitle(it)
           }
       }
   
       if (result?.isNotEmpty() == true) {
           // Immediately start playing from the beginning of the search results
           // Implement your logic to start playing music
           playMusic(result)
       } else {
           // Handle no queue found. Stop playing if the app
           // is currently playing a song
       }
   }
   ```

### Handle empty queries

1. `onPrepare()`, `onPlay()`, `onPrepareFromSearch()` or `onPlayFromSearch()` without a search query
   * Our media app should play the 'current' media.
   * If no current media: app should try to play something such as a song from the most recent playlist or a random queue.
   * The Google Assistant uses these APIs when a user asks to *play music on [our app name]*.
2. Cases of Android Automotive OS or Android Auto
   * When those get *'Play music on [our app name]'*, they attempted to launch our app and play audio by calling `onPlayFromSearch()`.
   * If no media name, those play a song in the most recent playlist or a random queue.

### Declare legacy support for voice actions

1. Edge cases that some systems require our app to contain and Intent filter for search.

```xml
<activity>
    <intent-filter>
        <action android:name=
             "android.media.action.MEDIA_PLAY_FROM_SEARCH" />
        <category android:name=
             "android.intent.category.DEFAULT" />
    </intent-filter>
</activity>
```

### Transport controls

1. The Assistant can issue voice commands to control playback and update media metadata. 
2. For the detail, please [see here](https://developer.android.com/guide/topics/media-apps/interacting-with-assistant). Official document has sample voice queries to try.

### Errors

1. The assistant handles errors from a media session when they occur, and reports them to users.
2. Once `getErrorCode()` returns error code, then Assistant can recognize.
3. **Commonly mishandled case**
   * User needs to sign in
   * User requests an unavailable action
   * User requests content not available in the app
   * User requests content where an exact match is unavailable (free user does to paid-tier plan)

## Playback with an intent

1. **Sending an intent with a deep** link can lead the assistant to launch an audio or video app.
2. Different sources
   * Starting a mobile app : use Google Search.
   * Starting a TV app
     * Our TV app should include a **TV Search Provider** to expose URIs for media content.
     * The Assistant sends a query to the content provider, and get an intent containing a URI for the deep link and optional actions.
     * If the query returns an action in the intent, the Assistant send that action and the the URI back to the app.
     * If the provider didn't specify an action, Assistant will add **ACTION_VIEW** to the intent.
3. The Assistant adds the extra **EXTRA_START_PLAYBACK** with value **true** to the intent, and our app should start playback when it receives an intent with **EXTRA_START_PLAYBACK**.



### Handling intents while active

1. When user ask the Assistant to play something while still playing content from a previous request.

2. Should override `onNewIntent()` to handle new requests.

3. When starting playback, the Assistant might add *additional flags* to the intent it sends to our app.

   * `FLAG_ACTIVITY_CLEAR_TOP` or `FLAG_ACTIVITY_NEW_TASK`

     

## Playback from a service

1. A **Media browser service** has been included in our app, the Assistant can start the app by communicating with the service's **Media session**.

2. `MediaSession.Token` has to be set when initialize the media browser service.

3. Remember to set the supported playback actions at all times, including during intialization.

   ![assistant-session-play](/Users/wonyoung/Desktop/assistant-session-play.png)

### Connecting to a MediaBrowserService

1. In order to use a service to start our app, the Assistant must be ablue to connect to the app's MediaBrowserService and retrieve its MediaSession.Token.

2. `onGetRoot()` can handle the connection request.

   * Accept all connection requests
   * Accept connection requests from the Assistant app only

3. Accept all connection request

   ```kotlin
   override fun onGetRoot(
           clientPackageName: String,
           clientUid: Int,
           rootHints: Bundle?
   ): BrowserRoot? {
   
       // To ensure you are not allowing any arbitrary app to browse your app's contents, you
       // need to check the origin:
       if (!packageValidator.isCallerAllowed(this, clientPackageName, clientUid)) {
           // If the request comes from an untrusted package, return an empty browser root.
           // If you return null, then the media browser will not be able to connect and
           // no further calls will be made to other media browsing methods.
           Log.i(TAG, "OnGetRoot: Browsing NOT ALLOWED for unknown caller. Returning empty "
                   + "browser root so all apps can use MediaController. $clientPackageName")
           return MediaBrowserServiceCompat.BrowserRoot(MEDIA_ID_EMPTY_ROOT, null)
       }
   
       // Return browser roots for browsing...
   }
   ```

4. Accept the Assistant app package and signature

   ```xml
   <signature name="Google" package="com.google.android.googlequicksearchbox">
       <key release="false">19:75:b2:f1:71:77:bc:89:a5:df:f3:1f:9e:64:a6:ca:e2:81:a5:3d:c1:d1:d5:9b:1d:14:7f:e1:c8:2a:fa:00</key>
       <key release="true">f0:fd:6c:5b:41:0f:25:cb:25:c3:b5:33:46:c8:97:2f:ae:30:f8:ee:74:11:df:91:04:80:ad:6b:2d:60:db:83</key>
   </signature>
   
   <signature name="Google Assistant on Android Automotive OS" package="com.google.android.carassistant">
       <key release="false">17:E2:81:11:06:2F:97:A8:60:79:7A:83:70:5B:F8:2C:7C:C0:29:35:56:6D:46:22:BC:4E:CF:EE:1B:EB:F8:15</key>
       <key release="true">74:B6:FB:F7:10:E8:D9:0D:44:D3:40:12:58:89:B4:23:06:A6:2C:43:79:D0:E5:A6:62:20:E3:A6:8A:BF:90:E2</key>
   </signature>
   ```

   