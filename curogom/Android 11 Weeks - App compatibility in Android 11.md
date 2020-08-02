# 이번 주의 메인 주제 : 호환성

Android에서는 각 릴리즈 업데이트 마다 호환성 관련 작업을 줄이기 위해 많은 노력을 기울이고 있는 중.

이번 Androind 11에서는  

- 동작 변경의 영향을 최소화
- 손쉬운 테스트와 디버깅 
- 비 SDK 인터페이스에 대한 제한
- 동적 리소스 로더 

등이 적용 되었음.

## Android 11에서의 테스트

기존 Android의 호환성 테스트 시에는 targetSDKVersion을 일일히 바꿔준 뒤 빌드하여 테스트 하던 반면,
토글 방식으로 테스트 할 변경사항들에 대해서 켜고 끌 수 있게 하여 
테스트 속도와 방식을 좀 더 원활하게 진행 할 수 있도록 변경 되었습니다.

![img](https://4.bp.blogspot.com/-YGDGz1JIPDI/XwKdbnvGZJI/AAAAAAAAJEg/D70tKKxbNZkkCiuzjkksquFQNNWTfWdgwCLcBGAsYHQ/s1600/toggle-changes.png)

해당 기능의 자세한 설명은 오늘 3번 항목에 대한 발표에서 자세하게 다뤄질 것으로 보입니다.
(관련 비디오 링크가 3번 영상과 연계되어있더라구요)

### 호환성 테스트를 위한 Android Studio의 신규 도구

 Android Studio 4.2 부터 테스트를 여러 물리적 혹은 가상 장치에서 병렬로 동시에 실행 할 수 있게 되었습니다.

![img](https://1.bp.blogspot.com/-qcsVjq6DqB0/XwKe0OC20GI/AAAAAAAAJEs/0_XMEzGEShwMjtESgGU4mYyjz8ewQ2JtgCLcBGAsYHQ/s1600/multipledevices.png)

이 기능은 개발 주기에서 가능한 빠르게 문제를 파악하고 빌드 간의 차이점을 비교 하는데에 초점이 맞춰져 있습니다.
또한 이 기능의 테스트 결과를 한 눈에 볼 수 있게 Test Matrix를 사용하여 조사할 수 있도록 별도 UI를 제공하고 있습니다.

![img](https://1.bp.blogspot.com/-uaXqHr7Uy44/XwKfBN5jNbI/AAAAAAAAJEw/YOX4TMx3gPUXHr_tWap600PQSoeOY2bPQCLcBGAsYHQ/s1600/testmatrix.png)

해당 기능의 자세한 설명은 오늘 2번 항목에 대한 발표에서 자세하게 다뤄질 것으로 보입니다.