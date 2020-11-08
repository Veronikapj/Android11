# Privacy and Security

### One-time Permission

기존 Application 내 권한 Access시에는 
카메라 / 마이크 / 위치 정보 등에 대한 접근 권한이
1번 승인하게 되면 추후에 별도 조치를 취하지 않으면
무기한으로 권한이 해제되지 않고 유지되게 되어있습니다.

안드로이드 11에서는 해당 권한을 사용자 측면에서 세밀하게 관리 할 수 있게
일회성 권한 부여를 할 수 있도록 하여 
개인 정보 보호에 친화적인 앱을 만들 수 있습니다.

[Watch Video](https://youtu.be/MXlVj-EYgIQ)

### Background location

Android 10버전 까지는 백그라운드 위치 사용 시 알림을 통해
사용자가 앱이 백그라운드에서 기기 위치에 접근하는 시기를 알 수 있었는데,
해당 경우 75% 이상의 사용자가 
위치 권한을 다운그레이드 하거나, 거부 하는 경우가 발생하였습니다.

Android 11에서는 백그라운드 위치 권한 부여는 
런타임 프롬프트시 부여 외에 추가 단계를 두어
앱에 백그라운드 위치가 필요한 경우에 OS 시스템에서는 먼저
포 그라운드 위치를 요청 후, 앱에서 별도의 권한 요청을 통해 
백그라운드 위치 권한을 넓히는 방식으로 변경 됩니다.
이로 인해, 시스템은 권한 부여를 완료하기 위해서 사용자를 OS 설정 메뉴로 이동 시킵니다.

Google Play Developers 에서는 2월에 앱에서 백그라운드 위치에 접근 하는 경우
승인을 해야한다고 발표 하였으며[(관련 링크)](https://support.google.com/googleplay/android-developer/answer/9799150)
2021년 까지 해당 정책을 준수하여야 합니다.

[Watch Video](https://www.youtube.com/watch?v=xTVeFJZQ28c&feature=youtu.be)

### Permissions auto-reset

대다수의 사용자는 기기에 60개 이상의 앱을 설치하지만,
이 중 1/3 정도의 앱만 정기적으로 사용 된다는 통계가 있습니다.

Android 11에서는 앱을 사용하지 않는 경우,
OS 자체 내에서 권한을 자동으로 재설정 하여 사용자에게 알리고,
앱에서는 향후 다시 실행 될 경우 유저에게서 다시 권한을 요청 할 수 있도록 변경 되었습니다.

단, 합법적으로 권한을 보유해야만 하는 앱이 있는 경우에는
OS 설정 창 내에서 이 기능을 끌 수 있도록 설정에서 지원 하고 있습니다.

[Android Developers Link](https://developer.android.com/preview/privacy/permissions#auto-reset)

### Data access auditing API

민감성 데이터에 대한 접근에 대해 신뢰성을 높일 수 있도록,
API를 통해 시스템에서 앱이 개인 사용자 데이터에 대한 접근 기록 시 
앱에서 추적 할 수 있도록 지원합니다.

[Android Developers Medium](https://medium.com/androiddevelopers/new-android-11-tools-to-make-apps-more-private-and-stable-c9dcea0af415)

### Scoped Storage

Android 10은 외부 저장소에 접근 시 파일 및 미디어에 접근 할 수 있는
범위가 지정된 저장소 기능을 도입하였습니다.

이 기능은 사진, 비디오, 음악에 대한 읽기 권한 만 부여하도록 저장소에 권한을 변경하고,
여러 가지 방법으로 공유 저장소에 대한 접근을 제한함으로써 유저의 개인정보를 보호 합니다.

최초 업데이트 이후에 개발자 피드백을 통해 여러 업데이트가 진행되었으며,
Android 11에서는 API Target 30인 모든 Application에서는 
이 기능이 필수로 적용 되어있어야 합니다.

[Android Developer API](https://developer.android.com/training/data-storage#scoped-storage)
[Watch Video](https://www.youtube.com/watch?v=RjyYCUW-9tY&feature=youtu.be)
[Android11 Storage](https://developer.android.com/preview/privacy/storage)

### Google Play system updates

Google Play 시스템 업데이트는 이미 Android Project Mainline의 계획에 따라
Android 10에 도입이 되었습니다.
[Android Project Mainline](https://android-developers.googleblog.com/2019/05/fresher-os-with-projects-treble-and-mainline.html)

Android 11에서는 기존 업데이트에 신규 모듈을 추가하고, 기존 모듈에 대한
보안 속성을 유지 관리합니다.
예로, 암호화 기본 요소를 제공하는 Conscrypt는 
Android 11 에서도 FIPS 유효성 검증을 유지합니다.
[Conscrypt](https://source.android.google.cn/devices/architecture/modular-system/conscrypt?hl=ko)

### BiometricPrompt API

각종 생체인증이 보급되고 있는 상황에서,
Android 11에서는 이 API를 활용하여 필요한 생체 인증의 강도를 지정 할 수 있습니다.
해당 라이브러리는 이전 버전과의 호환을 위해 Jetpack Biometric Lib에 추가 될 예정이며,
추가 작업이 완료 될 경우 별도 업데이트 공지를 통해 공유 될 예정입니다.

[BiometricPromet API](https://developer.android.com/preview/features#biometric-auth)
[Jetpack Biometric Lib](https://developer.android.com/jetpack/androidx/releases/biometric)

### Identity Credential API

현재 한국에서 시행되고있는 PASS의 모바일 운전면허증 기능 처럼
모바일 운전면허증, 국가별 ID 등에 대한 케이스에 대한 지원을 준비 할 예정입니다.
해당 내용은 향후 추가 업데이트를 통해 좀 더 자세히 조사 할 필요가 있을 듯 합니다.

[Article](https://www.brookings.edu/techstream/privacy-preserving-credentials-for-smartphones-are-coming/)

