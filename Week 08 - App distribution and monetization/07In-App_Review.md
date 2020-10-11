## [Leverage the In-App Review API for your Google Play reviews](https://android-developers.googleblog.com/2020/08/in-app-review-api.html)

앱 평점과 리뷰

- 사용자 측면 : 적합한 앱과 게임을 선택하는데 도움
- 개발자 측면 : 사용자의 니즈를 파악하는데 도움



이를 위해 Google Play 홈페이지에서 리뷰를 남길수 있고, 앱에 리뷰페이지로 리뷰를 중앙 집중식 관리를 할수 있게 했다.

**개발자가 가장 많이 요청한 기능 중 하나인 앱에서 바로 리뷰를 남길수 있는 In-App Review API를 출시함.**



### 앱 리뷰는 적시에 요청하라

앱 리뷰 요청은 사용자에게 유용한 피드백을 받을 수 있을만큼 충분히 앱을 사용했을때 나오는게 가장 적절하다. 단, 사용자의 플로우 중간에 리뷰를 요청해서 흐름을 방해하지 않아야 한다.

![User ratings for app image](https://4.bp.blogspot.com/-MSM5VeZfLAU/Xym0OR_huqI/AAAAAAAAPaA/2u9CsMLiQoE4cx3fmQPf0coIH0TuTWiSwCLcBGAsYHQ/s1600/image2.jpg)



In-App review API는 beta 버전일때 공개/비공개 리뷰를 지원한다.

In-App review API는 Java/Kotlin, C++, Unit 용으로 배포되는 Play Core Library 에 포함되어 있다. 
사용자가 앱에서 review를 요청하고 review 플로우를 시작할 수 있도록 lightweight API를 제공한다.

통합은 4개의 메인 단계로 구성.

1. 검토를 요청하기 위한 조건과 최적의 장소 정의
2. API에 대한 검토 플로우 요청
3. 적절한 순간에 리뷰 시작
4. 리뷰가 완료된 후 플로우를 계속함

중요한것은 사용자의 흐름을 변경하지 않아야함.

[샘플](https://github.com/android/app-bundle-samples/tree/master/PlayCoreKtx)에서 인앱 리뷰 API를 확인할 수 있으며, Play Core Kotlin Extension(KTX) Libary 를 통해 API를 호출하고, 인앱 업데이트 및 주문 기능 모듈과 같은 다른 API도 볼수 있다.



#### 베스트 피드백 수집

인앱 리뷰 API 호출하면 리뷰를 받기위한 추가 작업은 필요없다. 그리고, 사용자가 리뷰를 남기지 않기로 선택한다면, 사용자에게 과도한 메시지가 표시되지 않도록 한도를 설정했다.

개발자는 Google Play Console 에서 사용자 리뷰와 등급을 분석하고 답변 달 수 있다.



추가 조사

인앱 리뷰 플로우 (4단계)

![in app review flow](https://developer.android.com/images/google/play/in-app-review/iar-flow.jpg)

In-App Review API 요구 사항

장치

- Google Play 스토어가 설치된 Android 5.0 (API 레벨 21) 이상을 실행하는 Android 기기 (휴대 전화 및 태블릿).
- Google Play 스토어가 설치된 Chrome OS 기기.

Play Core Library

- 앱에 인앱 리뷰를 통합하려면 앱에서 [Play Core 라이브러리](https://developer.android.com/guide/playcore) 버전 1.8.0 이상을 사용해야합니다 .



## 인앱 검토를 요청하는 경우

사용자로부터 인앱 리뷰를 요청할시기를 결정하려면 다음 지침을 따르십시오.

- 사용자가 유용한 피드백을 제공하기에 충분한 앱을 경험 시킨 후 인앱 리뷰 플로우를 요청하라.
- 사용자에게 과도하게 리뷰를 요청하지 말라.
- 앱은 사용자의 의견에 대한 질문 (ex : "앱이 마음에 드십니까?") 또는 예측 질문 (ex : "이 앱을 별 5 개로 평가 하시겠습니까? ”) 을 하지 말라.



## 디자인 지침

앱에서 인앱 리뷰를 통합하는 방법을 결정할 때 다음 지침을 따르십시오.

- 크기, 불투명도, 모양 또는 기타 속성을 포함하여 기존 디자인을 변경하거나 수정하지 않고 카드를 있는 그대로 표시합니다.
- 카드 상단이나 주변에 오버레이를 추가하지 마십시오.
- 카드와 카드의 배경은 최상위 레이어에 있어야합니다. 카드가 표면에 나타나면 프로그래밍 방식으로 카드를 제거하지 마십시오. 카드는 사용자의 명시적인 작업 또는 내부 Play 스토어 메커니즘에 따라 자동으로 삭제됩니다.



## 할당량

Google Play는 사용자에게 리뷰 대화 상자를 표시 할 수있는 빈도에 할당량을 적용하기 때문에 항상 `launchReviewFlow` 를 호출한다고 하여 인앱 리뷰 팝업이 뜨지는 않는다.  그러므로 버튼 같은 동작으로 인앱 메시지를 항상 뜨는것처럼 요청하면 안된다.
