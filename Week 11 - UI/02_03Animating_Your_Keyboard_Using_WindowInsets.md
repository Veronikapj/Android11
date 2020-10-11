# Animating your Keyboard - Part1

## New WindowInsets APIs for checking the keyboard (IME) visibility and size

Android 11 의 WindowInsets API 많은 개선 사항으로 키보드를 열고 닫을 때 앱의 화면이 원활히 전환 된다.

![Image for post](https://miro.medium.com/max/2160/1*yc4QV2vZiA7ve3PVSrsW4w.gif)

[Android 11 키보드 애니메이션의 두 가지 예 : Google 검색 앱 (왼쪽), 메시지 (오른쪽)]



위와 같은 경험을 주기위해서는 아래의 3단계가 필요

1. edge-to-edge 를 해야한다.
2. 앱이 inset 애니메이션에 반응한다.
3. 위의 단계가 앱에 적합한 경우, inset 애니메이션을 제어하고 이끈다.

각 단계를 블로그 게시물에서 자세히 다루며 이번 블로그는 1번 단계를 다룬다.



## Going edge-to-edge

Android 10 부터 새로운 제스처 탐색을 최대한 활용하는 [edge to edge](https://medium.com/androiddevelopers/gesture-navigation-going-edge-to-edge-812f62e4e83e) 개념을 도입

![Image for post](https://miro.medium.com/max/1600/1*g8oz_UWkJ_1cFXNnEBGjNA.gif)

간단히 요약하면, 앱이 시스템 표시 줄 뒤에 그려진다.

작년의 문구를 인용하면

> *에지 투 에지를 이용하면 앱이 시스템 막대 뒤에 배치됩니다. 이는 앱 콘텐츠를 빛나게하여 사용자에게보다 몰입감있는 경험을 제공하기위한 것입니다.*



#### edge to edge와 키보드의 연관성?

edge to edge가 잘 동작하려면 상태바, 네이게이션바 뒤에 그려지는 것 이상이 필요하다. 앱과 겹칠수 있는 시스템 UI 부분을 처리하는 것은 앱의 역할이다. 

키보드, IME 또한 인식해야할 시스템 UI 의 일부다.



## How do apps go edge to edge?

작년 가이드라인으로 돌아가보자면, edge to edge 는 3가지 작업으로 구성

1. 시스템바 색상 변경
2. 전체화면 레이아웃 요청
3. 시각적 충돌 처리 

2, 3단계만 변경사항이 있어서 다룬다.



#### #2: 전체 화면 레이아웃 요청

전체 화면을 요청하기 위해 `systemUiVisibility` 에 여러 플래그를 함께 사용했다.

```kotlin
view.systemUiVisibility = 
    // Tells the system that the window wishes the content to
    // be laid out at the most extreme scenario. See the docs for
    // more information on the specifics
    View.SYSTEM_UI_FLAG_LAYOUT_STABLE or
    // Tells the system that the window wishes the content to
    // be laid out as if the navigation bar was hidden
    View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION
```



complie SDK version을 30 으로 업데이트 한 경우 이러한 모든 API는 더이상 사용되지 않는다.

`setDecorFitsSystemWindows()` 로 대체 되었다.

```kotlin
// Tell the window that we (the app) want to handle/fit any system
// windows (and not the decor)
window.setDecorFitsSystemWindows(false)

// OR you can use WindowCompat from AndroidX v1.5.0-alpha02
WindowCompat.setDecorFitsSystemWindows(window, false)
```

많은 플래그 대신 Boolean 값을 전달한다. 
앱이 시스템 윈도에 맞춤 처리한다면 false로 한다. (그리고 전체화면으로 이동한다.)

android.core v1.5.0-alpha2에 포함된 WindowCompat 메소드에서 사용할수있다.

이 부분이 2단계 업데이트이다.



#### #2: 시작적 충돌 처리

System UI와의 충돌을 피하기 위해 Android 의 `WindowInsets` 클래스나 AndroidX의 `WindowInsetsCompat` 을 사용한다.

API 30 업데이트 전 `WindowInsets` 을 살펴보면, 일반적으로 시스템 윈도우 insets를 사용한다. 이 것으로 상태바, 네비게이션바, 키보드 보임을 커버할수 있다.

`WindowInsets` 을 사용하려면 일반적으로 `OnApplyWindowInsetsListener` 를 사용한다.

```kotlin
ViewCompat.setOnApplyWindowInsetsListener(view) { v, insets ->
    v.updatePadding(bottom = insets.systemWindowInsets.bottom)
    // Return the insets so that they keep going down the view hierarchy
    insets
}
```

이 부분에서 insets으로 뷰의 패딩을 업데이트한다.

Android 10에서 최근에 추가된 제스처 insets 을 포함하여 많은 insets [타입](https://developer.android.com/reference/kotlin/androidx/core/view/WindowInsetsCompat#getsystemwindowinsets)이 있다.

```kotlin
ViewCompat.setOnApplyWindowInsetsListener(v) { view, windowInsets ->
    val sysWindow = windowInsets.systemWindowInsets
    val stable = windowInsets.stableInsets
    val systemGestures = windowInsets.systemGestureInsets
    val tappableElement = windowInsets.tappableElementInsets
}
```



하지만, Insets 을 쿼리하기 위해 많은 API는 더 이상 사용되지 않는다.

- `getInsets(type: Int)` 주어진 유형에 대해 보이는 insets 을 반환합니다.
- `getInsetsIgnoringVisibility(type: Int)` 는 표시 여부에 관계없이 insets을 반환합니다.
- `isVisible(type: Int)` 로 주어진 유형이 표시되면  `true`  를 반환 합니다.

> 비트 단위로 여러 type을 결합 할수 있다.

`WindowInsetsCompat` 에 모든 API가 추가 되어 API 14에서도 안전하게 사용할수 있다. 



방금 예제를 새로운 API로 업데이트하면 아래와 같다.

```kotlin
ViewCompat.setOnApplyWindowInsetsListener(...) { view, insets ->
-    val sysWindow = insets.systemWindowInsets
+    val sysWindow = insets.getInsets(Type.systemBars() or Type.ime())

-    val stable = insets.stableInsets
+    val stable = insets.getInsetsIgnoringVisibility(Type.systemBars())

-    val systemGestures = insets.systemGestureInsets
+    val systemGestures = insets.getInsets(Type.systemGestures())

-    val tappableElement = insets.tappableElementInsets
+    val tappableElement = insets.getInsets(Type.tappableElement())
}
```



## The IME type ⌨️

드디어 10년 전 [StackOverflow 질문](https://stackoverflow.com/questions/2150078/how-to-check-visibility-of-software-keyboard-in-android) 인 키보드 가시성을 확인하는 방법에 대해 답변을 할수 있게 됐다. 

현재 키보드 가시성을 얻기 위해서는 root window insets을 가져온 후 `isVisible()` 메소드를 호출하여 IME type을 전달 했었다.

비슷하게 높이를 알고 싶다면 아래와 같이 한다.

```kotlin
val insets = ViewCompat.getRootWindowInsets(view)
val imeVisible = insets.isVisible(Type.ime())
val imeHeight = insets.getInsets(Type.ime()).bottom
```

키보드의 변경 사항을 감지해야한다면 `OnApplyWindowInsetsListener` 를 사용한다.

```kotlin
ViewCompat.setOnApplyWindowInsetsListener(view) { v, insets ->
    val imeVisible = insets.isVisible(Type.ime())
    val imeHeight = insets.getInsets(Type.ime()).bottom
}
```



## 키보드 보이기/숨기기

11년전 [StackOverflow 질문](https://stackoverflow.com/questions/1109022/how-do-you-close-hide-the-android-soft-keyboard-using-java)인 "키보드를 어떻게 종료하는지" 에 대해 새로운 답변을 하려고 한다.

 [WindowInsetsController](https://developer.android.com/reference/android/view/WindowInsetsController) 의 Android 11의 새로운 API를 소개한다.
앱은 어떤 뷰에서든지 IME type을 넘김으로써 키보드를 보이거나 숨길수 있다.

```kotlin
val controller = view.windowInsetsController

// Show the keyboard (IME)
controller.show(Type.ime())

// Hide the keyboard
controller.hide(Type.ime())
```

하지만 키보드를 숨기고 표시하는 것이 컨트롤러가 할 수 있는 전부는 아니다.



## WindowInsetsController

앞선 내용에서 일부 `View.SYSTEM_UI_*` 플래그가 Android 11에서 더 이상 사용되지 않고 새로운 API로 대체 되었다고 했다. 하지만 언급하지 않은 아래와 같은 플래그들이 있다.

- `View.SYSTEM_UI_FLAG_LAYOUT_HIDE_NAVIGATION`
- `View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN`
- `View.SYSTEM_UI_FLAG_LAYOUT_STABLE`
- `View.SYSTEM_UI_FLAG_LOW_PROFILE`
- `View.SYSTEM_UI_FLAG_FULLSCREEN`
- `View.SYSTEM_UI_FLAG_HIDE_NAVIGATION`
- `View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY`
- `View.SYSTEM_UI_FLAG_IMMERSIVE`
- `View.SYSTEM_UI_FLAG_VISIBLE`
- `View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR`
- `View.SYSTEM_UI_FLAG_LIGHT_NAVIGATION_BAR`

모두 API 30 에서 depreated 됐으며 `WindowInsetsController` 의 API로 대체됐다.



모든 플래그에 대한 마이그레이션을 진행하는 대신 몇가지 시나리오를 다루고 업데이트 하는 방법을 살펴보자.



## 몰입형 모드

그리기시 앱의 공간을 최대화 하기위해 시스템 UI를 숨기는 그리기 앱을 볼 수 있다.

![Image for post](https://miro.medium.com/max/2160/1*Yvt16ghD3hIf1LF77SJ83w.gif)

WindowInsetsController를 사용하여 구현하려면 이전과 같은 hide () 및 show () 메소드를 사용하지만, 이번에는 시스템바 타입을 전달한다.

```kotlin
val controller = view.windowInsetsController

// When we want to hide the system bars
controller.hide(Type.systemBars())

// When we want to show the system bars
controller.show(Type.systemBars())
```

또한 앱은 [몰입 형 모드](https://developer.android.com/training/system-ui/immersive#immersive)를 사용하여 사용자가 시스템바를 숨긴 후 다시 스와이프 할 수 있다. WindowInsetsController를 사용하여 이를 구현하기 위해 숨김, 표시 동작을 `BEHAVIOR_SHOW_BARS_BY_SWIPE` 로 변경한다.

```kotlin
val controller = view.windowInsetsController

// Immersive is now...
controller.setSystemBarsBehavior(
    WindowInsetsController.BEHAVIOR_SHOW_BARS_BY_SWIPE
)

// When we want to hide the system bars
controller.hide(Type.systemBars())
```

마찬가지로 [고정 몰입 형 모드를](https://developer.android.com/training/system-ui/immersive#sticky-immersive) 사용하는 경우 `BEHAVIOR_SHOW_TRANSIENT_BARS_BY_SWIPE` 를 사용한다.

```kotlin
val controller = view.windowInsetsController

// Sticky Immersive is now ...
controller.setSystemBarsBehavior(
    WindowInsetsController.BEHAVIOR_SHOW_TRANSIENT_BARS_BY_SWIPE
)

// When we want to hide the system bars
controller.hide(Type.systemBars())
```



## 상태바 컨텐츠 색상

![Image for post](https://miro.medium.com/max/3840/1*TaeLKEFEIerlPz3aWcD4DQ.png)

`setSystemBarAppearance()` 에 [APPEARANCE_LIGHT_STATUS_BARS](https://developer.android.com/reference/android/view/WindowInsetsController#APPEARANCE_LIGHT_STATUS_BARS) 를 설정하여 우측과 같이 어두운 내용이 포함된 밝은 상태바 배경을 설정할 수 있다.

```kotlin
val controller = view.windowInsetsController

// Enable light status bar content
controller.setSystemBarsAppearance(
    APPEARANCE_LIGHT_STATUS_BARS, // value
    APPEARANCE_LIGHT_STATUS_BARS // mask
)
```

어두운 상태바를 설정하려면 value를 clear하는 대신 `0` 설정한다.

> 테마를 코드상 구현하지 않고 *android:windowLightStatusBar* 어트리뷰트를 설정하여 동일한 효과를 줄수 있다. 값이 변경되지 않으면 이 방법을 추천한다.

 `APPEARANCE_LIGHT_NAVIGATION_BARS` 에도 동일한 기능을 제공 하는 플래그를 사용할 수 있다.

이 API의 Jetpack 버전은 현재 개발중...



---

# Animating your Keyboard - Part2 

## Reacting to WindowInset animations

![Image for post](https://miro.medium.com/max/2560/1*9wELaIXfyzEQrHlsU2jkug.gif)

Android11 에서는 **앱이 키보드와 함께 이동**하여 더 좋은 경험을 제공한다.



## WindowInsetsAnimation

[WindowInsetsAnimation](https://developer.android.com/reference/android/view/WindowInsetsAnimation) 클래스는 insets을 포함하는 애니메이션을 캡슐화한 클래스다. [WindowInsetsAnimation.Callback](https://developer.android.com/reference/android/view/WindowInsetsAnimation.Callback) 를 사용하여 애니메이션 이벤트를 수신 할 수 있다.

```kotlin
val cb = object : WindowInsetsAnimation.Callback(DISPATCH_MODE_STOP) {
    // TODO
}

view.setWindowInsetsAnimationCallback(cb)
```



`WindowInsetsAnimation.Callback` 이 제공하는 메소드

키보드가 닫혀있는 상태에서 사용자가 EditText를 선택하여 시스템이 키보드를 표시하기 시작하는 경우 아래와 같이 수신한다.

```kotlin
val cb = object : WindowInsetsAnimation.Callback(DISPATCH_MODE_STOP) {

    override fun onPrepare(animation: WindowInsetsAnimation) {
        // #1: First up, onPrepare is called which allows apps to record any
        // view state from the current layout
    }
  
    // #2: After onPrepare, the normal WindowInsets will be dispatched to
    // the view hierarchy, containing the end state. This means that your
    // view's OnApplyWindowInsetsListener will be called, which will cause
    // a layout pass to reflect the end state.

    override fun onStart(
        animation: WindowInsetsAnimation,
        bounds: WindowInsetsAnimation.Bounds
    ):  WindowInsetsAnimation.Bounds {
        // #3: Next up is onStart, which is called at the start of the animation.
        // This allows apps to record the view state of the target or end state.
        return bounds
    }

    override fun onProgress(
      insets: WindowInsets,
      runningAnimations: List<WindowInsetsAnimation>
    ): WindowInsets {
        // #4: Next up is the important call: onProgress. This is called every time
        // the insets change in the animation. In the case of the keyboard, which
        // would be as it slides on screen.
        return insets
    }

    override fun onEnd (animation: WindowInsetsAnimation) {
        // #5: And finally onEnd is called when the animation has finished. Use this
        // to clear up any old state.
    }
}
```



이제, 이 이론을 시나리오에 적용



## 예제 구현

### onPrepare()

우선, 레이아웃이 변경하기전 bottom coordinate를 기록하고 `onPrepare()` 메소드를 재정의한다.

![Image for post](https://miro.medium.com/max/3200/1*jtS-M93Qjv0lhQegXsPgDg.png)

```kotlin
val view = binding.conversationList

val cb = object : WindowInsetsAnimation.Callback(DISPATCH_MODE_STOP) {
    var startBottom = 0
    var endBottom = 0

    override fun onPrepare(animation: WindowInsetsAnimation) {
        // #1: First up, onPrepare is called which allows apps to record any
        // view state from the current layout. We record the bottom of the view
        // in the window
        startBottom = view.calculateBottomInWindow()
    }
}
```



### Insets dispatch

이 시점에서 최종 상태 insets 이 전달되고 `OnApplyWindowInsetsListener` 가 호출된다. 리스너는 컨테이너 뷰의 패딩을 업데이트하여 콘텐츠를 밀어올린다.

아래에서 볼 수 있듯이 사용자는 이것을 볼 수 없다.

![Image for post](https://miro.medium.com/max/3200/1*qw75S_SOJ8_HZN40LsBOgA.png)



### onStart()

`onStart()` 는 view의 **끝(end)** 위치 를 기록 할 수 있다.

또한 사용자가 아직은 종료 상태를 보지 않도록하기 때문에, `translationY`를 사용하여 뷰를 원래 위치로 시각적으로 다시 이동한다. 시스템은 위의 삽입 패스에서 트리거 된 모든 레이아웃이 onStart ()와 동일한 프레임에서 호출되도록 보장하므로 사용자에게 깜박임이 표시되지 않는다.

![Image for post](https://miro.medium.com/max/3200/1*_EYNFB4cNL61AhviJSiBmw.gif)



```kotlin
val view = binding.conversationList

val cb = object : WindowInsetsAnimation.Callback(DISPATCH_MODE_STOP) {
    var startBottom = 0
    var endBottom = 0

    override fun onStart(
        animation: WindowInsetsAnimation,
        bounds: WindowInsetsAnimation.Bounds
    ):  WindowInsetsAnimation.Bounds {
        // #3: Next up is onStart, which is called at the start of the animation.
      
        // We record the bottom of the view within the window
        endBottom = view.calculateBottomInWindow()
        
        // And then we translate the view back down, so it is visually
        // in the start position
        view.translationY = startBottom - endBottom
      
        // We don't alter the bounds so just pass the value given to us
        return bounds
    }
}
```



### onProgress()

마지막으로 키보드가 들어가면 뷰를 업데이트 할 수 있도록 `onProgress()` 재정의 한다.

`translationY` 를 다시 사용하여 키보드와 화면이 일제히 변경 되도록 한다.

![Image for post](https://miro.medium.com/max/2400/1*dAYsHps6Q3BOsFpel1B0kA.gif)

```kotlin
val view = binding.conversationList

val cb = object : WindowInsetsAnimation.Callback(DISPATCH_MODE_STOP) {
    var startBottom = 0
    var endBottom = 0

    override fun onProgress(
      insets: WindowInsets,
      runningAnimations: List<WindowInsetsAnimation>
    ): WindowInsets {
        // #4: Next up is the important call: onProgress. This is called every time
        // the insets change in the animation.
      
        // We calculate the the offset for our view by linearly interpolating from
        // the start position, to the end position, using the animation's fraction
        val offset = lerp(
            startBottom - endBottom,
            0,
            animation.interpolatedFraction
        )
        // ...which we then set using translationY
        view.translationY = offset

        return insets
    }
}
```



## Keyboard synergy

구현을 보고 싶으면  [**WindowInsetsAnimation**](https://github.com/android/user-interface-samples/tree/master/WindowInsetsAnimation)  샘플 참고

다음 블로그에서는 앱이 키보드를 제어하는 방법과 목록 스크롤(list scrolling) 같은 동작으로 키보드를 자동 열수 있게 할 것 이다.



### View clipping

자신의 뷰에서 이를 구현하려고 시도하면 이 블로그 게시물에서 설명한 기술이 애니메이션으로 표시 될 때 뷰가 잘릴 수 있다. 이는 OnApplyWindowInsetsListener의 레이아웃 변경을 통해 크기가 조정 되었을 수있는 뷰를 translating하기 때문입니다.

해결 방법은 향후 게시물에서 살펴볼테지만, 지금은 이를 방지하는 한가지 방법이 포함된  [**WindowInsetsAnimation**](https://github.com/android/user-interface-samples/tree/master/WindowInsetsAnimation)  샘플 참조해라.





---

# Animating your Keyboard - Part3 

## Control inset animations

Sample project를 보면 키보드를 제어하는 헬퍼 클래스를 만들었다.

```kotlin
class SimpleImeAnimationController {
  // Start a request to control the IME
  fun startControlRequest()
  
  // Whether a request was successful and
  // we're in control of an IME animation
  fun isInsetAnimationInProgress(): Boolean
  
  // Inset (move) the IME by dy
  fun insetBy(dy: Int)
  
  fun finish()
  fun cancel()
}
```

Text 입력 부분을 드래그 하여 키보드를 보이기/숨기기 ([이미지 참고](https://github.com/android/user-interface-samples/tree/master/WindowInsetsAnimation))

```kotlin
val controller = SimpleImeAnimationController()

fun onTouch(ev: MotionEvent) = when(ev.action) {
  MotionEvent.ACTION_MOVE -> {
    // TODO : Check the scroll > touch slop
    
    val dy = ...
    
    if (!controller.isInsetAnimationInProgress()) {
      controller.startControlRequest()
    } else {
      controller.insetBy(dy)
    }
  }
  MotionEvent.ACTION_UP -> controller.finish()
  MotionEvent.ACTION_CANCEL -> controller.cancel()
}
```

