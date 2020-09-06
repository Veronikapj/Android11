##### Android 11 Weeks - 9 Week Android TV (Codelabs)

# Controlling media through MediaSession

[Codelabs Link](https://developer.android.com/codelabs/supporting-mediasession?authuser=1&return=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fandroid-week9-android-tv%3Fauthuser%3D1%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fsupporting-mediasession#0)



## 0. Introduce

### Video를 Play 할 때 MediaSession을 사용해야하는 이유?

​	MediaSession은 Android Platform과 Media Application 간의 필수 링크로,
​	Media가 재생 중인 것을 Android 에 알리고, Media 작업을 올바른 세션으로 전달 하고, 
​	Platform에 재생중인 항목과 제어 방법을 알려줍니다.

아래는 MediaSession을 활용하여 사용자에게 제공하는 대표적인 예시입니다.

#### - Google 어시스턴트

​	사용자는 Google 어시스턴트를 통해 '일시중지', '재시작', '다음' 과 가은 음성 명령을 통해
​	앱의 미디어와 쉽게 상호 작용 할 수 있으며,
​	미디어의 메타 데이터를 통하여 현재 재생중인 항목에 대해 답변을 얻을 수도 있습니다.

#### - Android TV

​	대형 화면 환경에서 Android TV  Application은 기존 TV 리모컨을 통해 '재생' / '일시정지' 등과 같은
​	간단한 미디어 컨트롤을 제공합니다.

#### - On-Screen media controls

​	MediaSession의 재생 상태 및 메타 데이터에 엑세스 하여 디바이스가 잠금화면 상태에서도
​	미디어의 컨트롤과 아트워크 등을 표시 할 수 있도록 지원합니다.

#### - Background media

​	미디어를 재생하는 앱이 백그라운드에서 실행 되는 경우에도 미디어를 제어할 수 있는 기능을 제공합니다.

#### - Ambient computing

​	재생중인 미디어 및 제어 방법에 대한 데이터로, 사용자가 즐기는 다양한 방식으로 상호작용 할 수 있도록
​	장치간의 상호 연결을 지언합니다.



### Codelabs Goal

- MediaSession이 사용자에게 더 풍부한 경험을 제공하는 이유에 대해 알아보기
- MediaSession을 만들고 상태를 관리하는 방법
- MediaSession을 Android의 대표 MediaPlayer인 ExoPlayer에 연결하는 방법
- MediaSession의 재생 대기열에 콘텐츠의 메타 데이터를 포함하는 방법
- 사용자 지정 작업을 추가하는 방법



## 1. Create Media Session & Manage State

해당 코드랩은 [ExoPlayer의 Demo Project](https://github.com/google/ExoPlayer)로 진행 됩니다.

우선, Player를 구현하는 필드에 아래와 같은 코드를 입력합니다.

![image-20200906195948169](image-20200906195948169.png)

단, 해당 외부 라이브러리 중 `MediaSessionConnector` 의 경우 
gradle에 `extension-mediasession` 이 추가 되어 있어야 합니다.

이제, 플레이어가 최초 생성 될 때, MediaSession을 연결 해 주게 되는데,
해당 커넥션은 아래의 코드로 작업 합니다.

![image-20200906200414841](image-20200906200414841.png)

또한 미디어 플레이어가 Release 될 경우 MediaSession도 같이 Release 되어야 하므로,
아래의 코드를 작업 합니다.

![image-20200906200518366](image-20200906200518366.png)

이제 연결과 해제가 완료 되었으니,
해당 미디어 세션의 상태를 관리 하는 방법에 대한 코드입니다.

우선, 미디어 플레이어의 재생이 시작 될 때, MediaSession 역시 활성화 작업을 시작해야 합니다.
해당 작업은 아래의 코드로 이루어 집니다.

![image-20200906200629986](image-20200906200629986.png)

또한, 더 이상 사용자가 미디어 플레이어를 필요로 하지 앟늘 경우, 미디어 세션을 비활성화 하는 작업이 필요한데, 
해당 작업은 아래의 코드로 이루어집니다.

![image-20200906200709696](image-20200906200709696.png)

## 2. Including metadata of items in the playback queue

ExoPlayer의 경우, 미디어 구조를 타임 라인으로 나타내는데, 
작동 방식에 대한 자세한 내용은 ExoPlayer 내의 [Timeline 객체](https://exoplayer.dev/doc/reference/com/google/android/exoplayer2/Timeline.html)에 대한 문서를 참조 바랍니다.

해당 구조를 활용 하게 되면, 콘텐츠가 변경 될 때 알림을 받고, 
요청 시 현재 재생중인 콘텐츠의 메타 데이터를 노출 할 수 있습니다.

해당 구조를 사용하기 위해 `initializePlayer()` Method 내에 아래의 코드를 추가하여 
`TimelineQueueNavigator` 객체를 추가합니다.

![image-20200906201144225](image-20200906201144225.png)

이 후 사용자는 Google 어시스턴트 같은 Application을 통해 '지금 무엇을 재생하고 있어?' 등의 질문에 
위에 설정한 'Title', 'Description', 'Subtitle'등을 알맞게 응답 받을 수 있습니다.

## 3. Customizing actions

개발자는 MediaSession을 통해 다양한 작업을 지원 할 수 있습니다.

각각의 동작의 지원을 위해 아래와 같은 작업 정보를 코드로 입력 해 주어야 합니다.

![image-20200906203806413](image-20200906203806413.png)

또한 커스텀 기능도 제공 할 수 있는데, 해당 코드랩에서는 사용자가 자막을 실행 할 수 있는 기능을 소개하고 있습니다.

![image-20200906204031773](image-20200906204031773.png)

