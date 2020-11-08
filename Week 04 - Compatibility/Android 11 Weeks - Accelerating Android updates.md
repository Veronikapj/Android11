# 안드로이드 OS 업데이트의 가속화

지난 몇 년간 Android 의 버전 업데이트를 균일하고 신속하며 효율적으로 제공 할 수 있도록
각 버전에 새로운 기능들이 포함 되어있었습니다.

- Oreo에서는 **Treble**의 등장으로 OEM 및 SoC 종속성을 명확하게 분리하기 위한 노력이 이루어졌습니다.
  [관련 링크](https://android-developers.googleblog.com/2017/05/here-comes-treble-modular-base-for.html)
  해당 기능의 도입으로 안드로이드 Pie에 대한 전환은 2.5배 빨라졌으며, 
  Google Play 스토어를 사전 예약 형태로 사용하는 모든 Android 기기는 
  이 시점부터 Treble를 준수하도록 되어있습니다.
- Pie에서는 개발자가 실제 하드웨어에서 앱 호환성 테스트에 사용할 수 있도록 GSI 개시를 시작하였습니다.
  Treble를 준수 함으로써, 모든 Android 장치가 GSI와 함께 제공되지 않더라도 GSI와 호환되게 되었으며,
  주요 파트너들과 함께 [OEM 개발자 미리보기 프로그램](https://android-developers.googleblog.com/2018/05/faster-adoption-with-project-treble.html)을 시작하였으며, 
  결과적으론 Android 10의 채택률이 1.5배 상승하였습니다.
- Android 10에서는 Google Play 시스템 업데이트 [(Project Mainline)](https://android-developers.googleblog.com/2019/05/fresher-os-with-projects-treble-and-mainline.html)를 통해 
  OS 구성요소를 직접 업데이트 하기 시작하였으며, 
  Google Play를 통해 앱과 유사한 방식으로 OS에 대한 보안 및 개인 정보 업데이트를 제공하고 있습니다.
  가장 최근의 배포에서는 보안 취약점에 대한 수정 사항으로 
  2억 9천대에 가까운 장비를 직접 업데이트를 진행하였습니다.
- 또한 Google Play는 인증 / 푸시알림 및 Google Play Protect와 같은 
  중요한 프로그램 및 서비스를 업데이트 하고 있습니다. 
  특히 Notification API의 업데이트는 COVID-19 상황에 공중 보건 기능을 높이는데 주로 사용되었으며, 
  Project Mainline 덕분에 단 4주만에 20억 대 이상의 디바이스에 배포 될 수 있었습니다.

위와 같은 여러 노력 덕분에 각각의 Android Release 의 채택 시점이 이전 버전들에 비해 점점 빨라지고 있으며,
가장 최근에 Release 된 Android 10의 경우에는 출시 후 약 5개월 동안 1억 대의 디바이스에서 실행 되었으며,
기존 Pie보다는 28% 가량 빠른 걸로 집계 되어지고 있습니다.

![img](https://3.bp.blogspot.com/-Sl_jjA1fGVE/Xwc8VcxcdlI/AAAAAAAAJHE/NKZ_6uyg_WIBFrmVmRH-jvyZR4VHE1BMgCLcBGAsYHQ/s1600/androidgraph.png)

## 그렇다면 Android 11은 과연?

Android 11에서도 빠른 채택을 위해 여러 기능들이 추가가 되었습니다. 하나하나 확인 해 보자면,

- OEM 개발자 미리보기
  Pie에서도 진행되었듯이 Android 11에서도 OEM 개발자들에게 Android 11에 관한 미리보기가 시행되고 있습니다.
  현재 7개의 OEM 업체가 13종의 디바이스에 Developer Preview 빌드를 출시하여 앱 개발자가 호환성을 테스트 할 때 다양한 하드웨어를 제공하고 있습니다. 

- Google Play 시스템 업데이트
  Android 10에 이어 Mainline의 추가 업데이트가 진행 되면서, Andorid의 다양한 장치와 정보의 업데이트가 가능

- 일반적인 커널 이미지
  지속적인 업데이트 작업은 6 년 LTS 지원과 같은 이니셔티브를 통해 Linux 커널 자체로 확장됩니다. 
  Android 11에서는 모든 Android 기기에서 작동하는 일반적인 커널 이미지 (GKI)를 생성하고보다 
  빠른 보안 배포를 지원하기 위해 Android Linux 커널에서 공통 코드를 추가로 격리합니다.

- 가상의 A / B
  대부분의 OS 업데이트는 Google Play를 통해 제공되지 않습니다. 
  대신 다양한 OEM간에 다른 별도의 타사 OTA (Over-the-Air) 서비스를 사용합니다. 
  이러한 서비스는 공간 효율성이 매우 높지만 적용 속도가 느려서 
  장치를 지속 기간 동안 사용할 수 없게하는 메커니즘을 사용합니다. 

  이 문제를 해결하기 위해 Android Nougat에서 
  "A / B OTA"(일명 [Seamless Updates](https://source.android.com/devices/tech/ota/ab?hl=en) ) 라는 메커니즘을 시작했습니다 . 
  A / B OTA는 백그라운드에서 적용되고 다음에 다시 부팅 할 때 활성화되므로 
  사용자의 관점에서 거의 즉각적인 것처럼 보일 수 있습니다. 
  그러나 OS 자체에 예약 된 스토리지 양을 두 배로 늘려 OEM의 채택을 제한했습니다.

  Android 11에서는 이전의 두 가지 장점을 결합한 새로운 OTA 메커니즘 (Virtual A / B)이 적용되었습니다.
  즉, 사용자 관점에서 매끄럽게 유지하면서 적은 스토리지를 요구합니다. 
  우리는 OEM 파트너와 긴밀히 협력하여 
  Android 11 기기에서 Virtual A / B 구현을 시작하여 가능한 한 마찰이없는 OTA를 만듭니다. 
  앞으로 Virtual A / B는 Android에서 유일하게 지원되는 OTA 메커니즘이 될 것입니다.
