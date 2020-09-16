# What's new in Media

## Media Controls

기존 미디어 컨트롤러는 Notification 목록 중 최 상위에 위치했던 것에 반해, 
Android 11에서는 Quick Settings panel로 이동 됨.

Carousel이 적용 되어, 현재 미디어 세션은 물론 최근에 동작한 미디어 세션을 보여주며,
출력 전환도 가능하여 원하는 스피커에 연결 할 수 있도록 도와주는 역할을 함.

![videoplayback_Moment](res\videoplayback_Moment1.jpg)

해당 동작을 지원하기 위해 개발자는 두 가지를 지원해야 하는데,
첫 번째로는 MediaSession, 그리고 뷰를 꾸며 줄 MediaStyle(with Notification styled) 두 가지이다.
(해당 두 API는 이미 롤리팝에서부터 지원)

![videoplayback_Moment2](res\videoplayback_Moment2.jpg)

위의 코드처럼 MediaSession과 MediaControl을 연동하게 되면 addAction을 통해 
일시 중지 / 다음 트랙으로 이동 등과 같은 별도의 작업을 시행할 수 있게 됨.

### Media Resumption

해당 기능은 Android 11에서 추가 된 기능으로,
장치를 재부팅 한 후에도 플레이 중이던 미디어를 찾고 다시 재생할 수 있게 할 수 있음.

최대 5개의 최신 미디어 앱에 대한 미디어 컨트롤을 소유하여 위에 설명한 
MediaControl에서 Carousel 형태를 통해 제공 할 수 있음.

![videoplayback_Moment3](res\videoplayback_Moment3.jpg)

해당 기능을 지원하기 위해서는 앱에서 MediaBrowserService를 제공하여야 함.

![videoplayback_Moment4](C:\Users\ybw89\Documents\res\videoplayback_Moment4.jpg)

우선 시스템 UI를 실행하여 MediaBrowserServices의 onGetRoot 메서드를 호출하고,
EXTRA_RECENT 힌트를 통해 사용자가 중단한 지점에서 다시 재생 할 준비를 해야 하며,
재생을 할 시, 시스템 UI가 MediaSession에 연결 하여 onPlay Callback을 호출 하여 실행 할 수 있음.

### Seamless Transfer

Android 11은 미디어 세션을 재개할 수 있을 뿐만 아니라 디바이스에서 
다른 디바이스로 재생을 쉽게 전송 할 수 있도록 지원함.

출력 스위처를 통해 앱에서 사용 가능한 스피커 / 이어폰을 찾아내어
단순 조작을 통해 출력 디바이스를 손쉽게 변경 할 수 있는 기능.

해당 기능을 사용하기 위해선 MediaRouter Jetpack(1.2.0-alpha02 부터 지원) 라이브러리를 앱에 추가 후,
AndroidManifest에 TransferReceiver를 추가.

앱 내에서 MediaRouter 싱글톤 객체를 활용하여 사용 가능한 스피커 / 이어폰의 상태 및 연결 관리 할 수 있게 됨.

MediaControls는 Android 11 Beta 2.5 이상에서 사용 가능 함.

## Media Streaming & 5G

Android 11에서는 TEMPORARITLY_NOT_METED라는 새 네트워크 상수를 도입하여,
5G 대역폭에 따른 높은 Bitrate Content를 제공 받을 수 있게 됨.

단, 5G외 다른 대역폭의 네트워크로 다시 변경 될 경우 기존과 같은 스트림이 제공 됨.

## Oboe

NDK21 에서부터는 더 이상 OpenSLES가 더 이상 쓰이지 않으며, 
해당 라이브러리를 사용 중이었다면 Oboe로 마이그레이션 하여야 함.

Oboe는 API 16 이상에서부터 지원하고 있으며, 고성능과 낮은 지연율이 특징인 라이브러리.

Oboe를 지원 하는 기기에서는 최신 AAudio API를 사용하게 되며, 
지원하지 않는 기기에서는 자동으로 OpenSLES로 전환되어 사용 할 수 있게 됨.

기존 OpenSLES 보다 Oboe가 훨씬 단순하게 구성되어있으므로 마이그레이션이 간단한 편 (발표자 주장)
[Oboe 마이그레이션 안내서](https://goo.gle/sles-migration)

## MediaCodec API Update

입출력 버퍼 할당에 더 많은 제어권을 확보하여 인/디코딩 시 관련 된 메모리를 보다 효율적으로 관리 할 수 있게 됨.

게임이나 생방송 같은 라이브 콘텐츠를 스트리밍 할 때 유용한 저지연 디코딩 지원도 가능.