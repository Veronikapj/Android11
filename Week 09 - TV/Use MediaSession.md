# Use MediaSession

미디어 세션은 관리하는 플레이어와 함께 움직입니다. 
미디어 세션 및 관련 플레이어를 소유한 활동 또는 서비스의 `onCreate()` 메서드에서 미디어 세션을 만들고 초기화해야 합니다.


**참고:** 미디어 앱을 작성하는 가장 좋은 방법은 [media-compat 라이브러리](https://developer.android.com/guide/topics/media-apps/media-apps-overview?authuser=1#compat-library)를 사용하는 것입니다. 
이 페이지에서 '미디어 세션'은 `MediaSessionCompat`의 인스턴스를 의미하고 
'미디어 컨트롤러'`MediaControllerCompat`의 인스턴스를 의미합니다.

## 미디어 세션 초기화

새로 만든 미디어 세션에는 기능이 없습니다. 다음 단계에 따라 세션을 초기화해야 합니다.

- 미디어 세션이 미디어 컨트롤러 및 미디어 버튼에서 콜백을 수신할 수 있도록 플래그를 설정합니다.
- `PlaybackStateCompat`의 인스턴스를 만들고 초기화한 다음 세션에 할당합니다. 
  재생 상태는 세션 전체에서 변경되므로 재사용을 위해 `PlaybackStateCompat.Builder`를 캐시하는 것이 좋습니다.
- `MediaSessionCompat.Callback`의 인스턴스를 만들고 세션에 할당합니다.

세션을 소유한 [활동](https://developer.android.com/guide/topics/media-apps/video-app/building-a-video-player-activity?authuser=1) 또는 [서비스](https://developer.android.com/guide/topics/media-apps/audio-app/building-a-mediabrowserservice?authuser=1#init-session)의 `onCreate()` 메서드에서 미디어 세션을 만들고 초기화해야 합니다.

앱이 새로 초기화되거나 중지될 때 [미디어 버튼](https://developer.android.com/guide/topics/media-apps/mediabuttons?authuser=1)이 작동하려면 
미디어 버튼이 전송하는 인텐트와 일치하는 재생 작업이 `PlaybackState`에 포함되어 있어야 합니다. 
이러한 이유로 초기화 중에 `ACTION_PLAY`가 세션 상태에 할당됩니다. 자세한 내용은 [미디어 버튼에 응답](https://developer.android.com/guide/topics/media-apps/mediabuttons?authuser=1)을 참조하세요.

## 재생 상태 및 메타데이터 유지 관리

미디어 세션의 상태를 나타내는 두 가지 클래스가 있습니다.

`PlaybackStateCompat` 클래스는 플레이어의 현재 작동 상태를 설명합니다. 여기에는 다음이 포함됩니다.

- 전송 상태(재생 중/일시중지/버퍼링 등의 플레이어 상태. `getState()` 참조)
- 오류 코드 및 선택사항인 오류 메시지(해당되는 경우). (아래의 `getErrorCode()`를 참조하고 [상태 및 오류](https://developer.android.com/guide/topics/media-apps/working-with-a-media-session?authuser=1&return=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fandroid-week9-android-tv%3Fauthuser%3D1%23article-https%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fmedia-apps%2Fworking-with-a-media-session#errors)를 읽어보세요.)
- 플레이어 위치
- 현재 상태에서 처리할 수 있는 유효한 컨트롤러 작업

`MediaMetadataCompat` 클래스는 재생 중인 자료를 설명합니다.

- 아티스트, 앨범 및 트랙의 이름
- 트랙 길이
- 잠금 화면에 표시할 앨범 아트워크. 이미지는 최대 크기 320x320dp의 비트맵입니다(더 큰 경우 축소됨).
- 아트워크의 더 큰 버전을 가리키는 `ContentUris`의 인스턴스

플레이어 상태와 메타데이터는 미디어 세션의 수명 동안 변경될 수 있습니다. 
상태 또는 메타데이터가 변경될 때마다 각 클래스인 `PlaybackStateCompat.Builder()` 또는 `MediaMetadataCompat.Builder()`에 해당하는 빌더를 사용한 다음, 
`setPlaybackState()` 또는 `setMetaData()`를 호출하여 새 인스턴스를 미디어 세션에 전달해야 합니다. 
빈번한 작업으로 인한 전체 메모리 소비를 줄이려면 빌더를 한 번만 만들고 세션 수명 동안 재사용하는 것이 좋습니다.

### 상태 및 오류

`PlaybackState`는 *세션의 재생 상태* `getState()`와 필요한 경우 관련 *오류 코드* `getErrorCode()`에 관한 별도의 값을 포함하는 객체입니다. 

재생이 예기치 않게 중단될 때마다 심각한 오류가 발생합니다. 
전송 상태를 `STATE_ERROR`로 설정하고 `setErrorMessage(int, CharSequence)`와 관련된 오류를 지정하세요. 
오류에 의해 재생이 차단되는 한 `PlaybackState`는 계속해서 `STATE_ERROR` 및 오류를 보고해야 합니다.

앱이 요청을 처리할 수는 없지만 계속해서 재생할 수 있는 경우에는 심각하지 않은 오류가 발생합니다. 
전송은 '정상' 상태(예: `STATE_PLAYING`)로 유지되지만 `PlaybackState`는 오류 코드를 가지고 있습니다. 
예를 들어, 마지막 곡이 재생 중이고 사용자가 다음 곡으로 건너뛰기를 요청하면 재생을 계속할 수 있지만, 
오류 코드가 `ERROR_CODE_END_OF_QUEUE`인 새 `PlaybackState`를 만든 다음 `setPlaybackState()`를 호출해야 합니다. 
세션에 연결된 미디어 컨트롤러는 `onPlaybackStateChanged()` 콜백을 수신하고 
무슨 일이 일어났는지를 사용자에게 설명합니다. 
심각하지 않은 오류는 발생 시점에 한 번만 보고해야 합니다. 다음에 세션 업데이트 시 
`PlaybackState`는 심각하지 않은 동일한 오류를 다시 설정하지 않습니다 (새 요청의 응답으로 오류가 발생하지 않는 한).



## 미디어 세션 잠금 화면

Android 4.0(API 레벨 14)부터 시스템은 미디어 세션의 재생 상태와 메타데이터에 액세스할 수 있습니다. 이 방법으로 잠금 화면이 미디어 컨트롤과 아트워크를 표시할 수 있습니다. Android 버전에 따라 행동이 다릅니다.

### 앨범 아트워크

Android 4.0(API 레벨 14) 이상에서는 미디어 세션 메타데이터에 배경 비트맵이 포함된 경우에만 잠금 화면의 배경에 앨범 아트워크가 표시됩니다.

### 전송 컨트롤

Android 4.0(API 레벨 14)부터 Android 4.4(API 레벨 19)까지는 미디어 세션이 활성화되고 미디어 세션 메타데이터에 백그라운드 비트맵이 포함되면 잠금 화면에 전송 컨트롤이 자동으로 표시됩니다.

Android 5.0(API 레벨 21) 이상에서는 시스템이 잠금 화면에 전송 컨트롤을 제공하지 않습니다. 대신 전송 컨트롤을 표시하려면 [MediaStyle 알림](https://developer.android.com/guide/topics/media-apps/audio-app/building-a-mediabrowserservice?authuser=1#mediastyle-notifications)을 사용해야 합니다.

### 맞춤 작업 추가

`addCustomAction()`으로 맞춤 작업을 추가할 수 있습니다. 예를 들어 컨트롤을 추가하여 좋아요 작업을 구현하는 방법은 다음과 같습니다.

[KOTLIN Code]

```kotlin
    stateBuilder.addCustomAction(
            PlaybackStateCompat.CustomAction.Builder(
                    CUSTOM_ACTION_THUMBS_UP,
                    resources.getString(R.string.thumbs_up),
                    thumbsUpIcon
            ).run {
                setExtras(customActionExtras)
                build()
            }
    )
    
```

전체 예제는 [유니버설 음악 플레이어](https://github.com/android/uamp/blob/f3154af7ac972ee9b7b1fd32ca3c935e02268a18/mobile/src/main/java/com/example/android/uamp/playback/PlaybackManager.java#L150-L171)를 참조하세요.

`onCustomAction()`으로 작업에 응답합니다.

[KOTLIN Code]

```kotlin
    override fun onCustomAction(action: String, extras: Bundle?) {
        when(action) {
            CUSTOM_ACTION_THUMBS_UP -> {
                ...
            }
        }
    }
    
```

[유니버설 음악 플레이어](https://github.com/android/uamp/blob/f3154af7ac972ee9b7b1fd32ca3c935e02268a18/mobile/src/main/java/com/example/android/uamp/playback/PlaybackManager.java#L328-L346)도 참조하세요.

## 미디어 세션 콜백

기본 미디어 세션 콜백 메서드는 `onPlay()`, `onPause()` 및 `onStop()`입니다. 여기에서 플레이어를 제어하는 코드를 추가합니다.

런타임에(`onCreate()`에서) 세션의 콜백을 인스턴스화하고 설정하므로 앱에서 다른 플레이어를 사용하는 대체 콜백을 정의하고 기기 또는 시스템 수준에 따라 적절한 콜백/플레이어 조합을 선택할 수 있습니다. 앱의 나머지를 변경하지 않은 채 플레이어를 변경할 수 있습니다. 예를 들어 Android 4.1(API 레벨 16) 이상에서 실행할 경우 [ExoPlayer](https://developer.android.com/guide/topics/media/exoplayer?authuser=1)를 사용하고 그 이전 시스템에서는 [`MediaPlayer`](https://developer.android.com/guide/topics/media/mediaplayer?authuser=1)를 사용할 수 있습니다.

콜백은 플레이어를 제어하고 미디어 세션 상태 전환을 관리하는 것 외에도 앱의 기능을 사용 설정 및 사용 중지하고 앱이 다른 앱 및 기기 하드웨어와 상호작용하는 방식을 제어합니다. 자세한 내용은 [오디오 출력 제어](https://developer.android.com/guide/topics/media-apps/volume-and-earphones?authuser=1)를 참조하세요.