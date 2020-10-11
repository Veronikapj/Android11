## Android 11 Beta 2 and Platform Stability

Android 11 두번째 베타는 플랫폼 안정성을 위한 중요한 시점에 도달하게 되며, Android 11의 API와 동작이 완성됨을 의미.
개발자는 호환성 작업을 진행 시작하며, 3분기 중에 이뤄질 공식 출시에 맞춰 공개 가능하게 됨.



### 플랫폼 안정성

베타 2에서는 플랫폼 안정성이 제공되며, 이는 Android 11에서의 앱의 화면과 동작이 확정 됨을 의미한다. 
SDK와 NDK API, 시스템 동작, 비공식 API 제한 목록의 최종 확정도 포함.

![Platform Stability timeline](https://4.bp.blogspot.com/-xv5CjfndWg8/XwTT2ykJSsI/AAAAAAAAJGI/rqxWH9MppHoIcLffVcmtnaBXwmspfeeCwCLcBGAsYHQ/s1600/timeline.png)

일정에 대한 자세한 정보: [링크](https://developer.android.com/preview/overview#timeline)

앱 개발자들은 최종 호환성 테스트를 실행 할수 있고,
모든 SDK, 라이브러리 및 게임 엔진 개발자는 지금 테스트를 시작하고 최대한 빨리 호환 업데이트를 진행



### 앱 호환성이 중요한 이유

**앱 호환성** 이란 앱이 플랫폼의 특정 버전 (보통 최신 버전)에서 정상적으로 동작함을 의미.
Android 11 기기 단말 or 에뮬레이터에 프로덕션 앱을 설치함으로 확인 가능하며, 모든 사용자 플로우와 기능을 테스트 했을때 앱이 정상 동작하면 된다.

간단해 보이지만, 개인 정보 보호와 보안을 개선하는 필수적인 변경 사항 뿐 아니라 OS 전반적으로 사용자 경험을 향상하는 변경 사항을 구현하고 있으므로, 이러한 **변경 사항이 앱에 영향을 미치는지 동작 변경 사항을 확인하고 테스트**를 해야한다.

호환성을 위해 우선적으로 기존 앱을 테스트하고 호환 업데이트를 출시하는 것부터 하길 바란다.

Android 11이 AOSP에 최종 출시 하는 즉시 픽셀과 다른 기기에 대한 업데이트가 시작될 것이다. 호환성 테스트를 지원하기 위해 픽셀과 일부 단말에 미리보기로 공개되어 있다.



### Android 11에서 보다 쉬워진 앱 호환성 지원

구글에서는 새로운 OS 버전이 출시될때마다 이에 맞춰 앱을 준비하는데 필요한 노력을 줄이고 있다.
이번 업데이트의 영향을 최소화 하기 위한 새로운 프로세스, 개발자 도구, 출시 일정을 추가하여 앱의 호환성을 보다 쉽게 확보할 수 있도록 했다.

- 동작 변경의 영향 최소화 : targetSdkVersion을 Android 11으로 설정할 때까지 최대한 플랫폼 변경 사항을 유예하도록 하였다. Google Play를 통해 앱을 배포할 경우 이런 변경 사항을 반드시 적용해야하는 시점까지 1년 이상이 확보 된다.

- 더욱 수월한 테스트와 디버깅 - 호환성 테스트에 도움이 되도록, 많은 breaking changes를 on/off 가능하도록 했다. 시스템 설정 > 개발자 옵션 이나 adb에서 변경 사항을 개별적으로 강제 활성화하거나 비활성화가 가능하다. 더 이상 기본적이 테스트를 위해 targetSdkVersion을 변경하거나 앱을 다시 컴파일 할 필요가 없다.

  [자세히 보기](https://developer.android.com/preview/test-changes)

![App compatibility toggles in Developer options](https://3.bp.blogspot.com/-vkoW40yNHlY/XwTUwuWo4mI/AAAAAAAAJGU/mrx1eNqeKAQXxtkM-lAFs0oY9Vuzu6HmQCLcBGAsYHQ/s1600/appcompatibilitychanges.png)

- 비공식 API 제한 - 개발자가 비공식 API 사용을 하지 않도록 비공식 인터페이스 제한 목록을 업데이트 했다. 공개 API에 대한 피드백과 요청은 언제나 환영한다.
- 동적 리소스 로더 - 비공식 API에서 이전하는 과정의 하나로, 개발자 커뮤니티에서 런타임 리소스와 자산을 동적 로드하는 공개 API를 요청 받았다. 이에 따라 Android 11에 Resource Loader Framework 를 추가 했다.
- 플랫폼 안정성 이정표 - 개발자에게 명확한 최종 변경 날짜를 알리기 위해 출시 프로세스에 새롭게 추가된 이정표. 플랫폼 안정성에는 최종 SDK, NDK API, 내부 API, 시스템 동작이 포함



### Android 11용으로 앱을 준비하세요!

앱 호환성 작업을 시작하세요.

![Android 11 compatibility flow chart](https://2.bp.blogspot.com/-c7S2OU24y-g/XwTVSX-OHJI/AAAAAAAAJGc/y678nfHADwoY8olKdEcSVnC4Pga9uG8tACLcBGAsYHQ/s1600/androi11-compat-flow.png)

모든 앱에 대한 동작 변경 사항을 확인해라.

- **TargetSdkVersion과 무관하게 적용**
  - 일회성 권한 (One-time permission) : 위치, 마이크, 카메라에 엑세스에 일회성 권한 부여가 가능하다.
  - 외부 저장소 접근 (External storage access) : 외부 저장소의 다른 앱 파일에 접근할 수 없다.
  - Scudo 강화 할당자 (Scudo hardened allocator) : 이제 앱에서 네이티브 코드용 힙 메모리 할당자를 지원한다.
  - 파일 디스크립터 새니타이저 (File descriptor sanitizer) :앱에서 네이티브 코드의 문제를 처리하는 파일 디스크립터를 탐지하는 기능이 기본적으로 활성화 된다.

**앱에서 라이브러리, SDK와의 호환성 테스트를 꼭 진행해라**

- **TargetSdkVersion 30 이상 에서만 적용**
  - 범위 지정 저장소 (Scoped storate) : 파일 읽기/쓰기 를 위한 새로운 저장소 제약, 동작 및 API
  - 백그라운드 위치 정보 (Backgound location) : 앱이 백그라운드 상태에서 위치를 요청하는 방법, 사용자 권한을 부여하는 방법 변경
  - 패키지 가시성 (Package visibility) : 앱이 다른 설치된 앱을 찾고 상호작용하는 방식 변경
  - 압축된 리소스 파일 (Compressed resource files) : 앱에 압축된 resources.arsc 파일이 있거나 파일이 4byte 단위로 맞춰 있지 않으면 앱을 설치하거나 업데이트 할 수 없다.
  - APK 서명 스킴 v2 (APK Signature Scheme v2) : APK Signature Scheme v2 이상을 사용하여 앱을 서명해야한다.
  - 힙 포인터 태깅 (Heap pointer tagging) - 64 비트 프로세스의 경우 네이티브 힙 할당은 포인터의 상위 바이트에 태그 세트가 있으며, 앱에서 수정하면 안된다.

제한된 비공게 API의 사용은 공개 SDK 인터페이스로 옮겨라.



### 새로운 기능 및 API 살펴보기

호완성 대응이 끝나면, Android 11에서 구현할 수 있는 새로운 경험을 찾아봐라.
Android Studio에는 대용량 APK를 더 빠르게 설치하기 위한 ADB 증분, 플랫폼 AP의 추가적인 null 허용 여부 주석 등 Android 11에서 생산성과 워크플로를 개선하기 위한 새로운 기능도 있다. 최신 Android Studio 베타나 Canary 버전을 다운로드 하여 살펴볼 수 있다.



### 베타 2는 어떻게 구할 수 있는가?

1. https://www.google.com/android/beta 에 기기 등록 (실기기)
2. Android Flash Tool 사용 (실기기)
3. Android Emulator 사용 (에뮬레이터)
4. GSI 이미지를 사용하여 지원되는 Treble 호환 기기



### Android 11 호환성 관련 자료

[#11 Weeks of Android](https://developer.android.com/11weeksofandroid) Chapter 4 에서 중점적으로 다룬다.