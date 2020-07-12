# What's new in System UI

Links: https://youtu.be/oLLUDOQxJS8


# Features

conversations

bubbles

device controls

media player

---

# Conversations

![Whats%20new%20in%20System%20UI%20/_______()_2-30_screenshot.png](What%20s%20new%20in%20System%20UI%20/_______()_2-30_screenshot.png)

1. 분리 공간 생성
2. 읽지 않은 메시지 
    1. 확장 지원성을 훨씬 쉽게 보고 적용할 수 있음
    2. 알림을 확장하면 즉각적으로 예측된 답을 하거나 앱 구동 가능
    3. 아바타를 더 크게 변경
    4. 길게 누르면 중요하다고 표시할 수 있음(현재 알림에도 적용되어 있는 기능)

---

## Using Conversations API

1. Use MessagingStyle
2. Add a shortcut ID

```kotlin
// 1. Person 객체 설정
// you have already done this if you're using MessagingStyle
val person = Person.Builder()
                .setName(...)
                . ...
                .build()
```

```kotlin
// 2. 알림을 위해서 단축키를 만들어 ShortcutManager 에 등록
// you have alread done this 
// if you have Lancher shortcuts or sharing shortcuts
val shortcut = ShortcutInfo.Builder(
                context, conversationShortcutId)
								// but these parts may be new
								// LongLived 단축키 제작 
								// 시스템이 이를 고수하여 다른 공간에서도 사용
                ***.setLongLived(true)** 
								// 방금 생성한 Person 객체를 첨부*
                ***.setPerson(person)***
                .build()
context.getSystemService(Context.SHORTCUT_SERVICE)
                .pushDynamicShortcut(shortcut)
```

```kotlin
// 단축키 ID가 있는지 확인하고, 알림 빌더를 사용해서 모두 결합 
val notification = Notification.Builder(
                context, conversationChannelId)
                .setStyle(Notification.MessagingStyle.Builder(person)
                        .addMessage(...)
                        . ...
                        .build())
                .setShortcutId(conversationShortcutId)
                .build()
```

---

# Bubbles

사용자가 실제로 참여하고 있는 대화를 바탕으로 휴대폰 화면의 상단에 버블 상태로 띄워 진다.

![Whats%20new%20in%20System%20UI%20/_______()_4-41_screenshot.png](What%20s%20new%20in%20System%20UI%20/_______()_4-41_screenshot.png)

BubbleMetadata 사용

- 대화 공간에 맞춰 결합시킨 같은 단축 ID 를 추가해야 함
- PendingIntent : 멀티 창의 크기를 조정할 수 있음, 크기는 미리 조정할 준비해야 함.

---

# Device Controls

![Whats%20new%20in%20System%20UI%20/_______()_5-4_screenshot.png](What%20s%20new%20in%20System%20UI%20/_______()_5-4_screenshot.png)

전원 메뉴를 새롭게 단장

## android .service.controls API

- Device-controls providers를 위한 새로운 api 출시
- 자바 9 Reactive Streams api 중 하나

![Whats%20new%20in%20System%20UI%20/_______()_6-15_screenshot.png](What%20s%20new%20in%20System%20UI%20/_______()_6-15_screenshot.png)

1. createPublisherForAllAvailable 
    - Reactive Stream에서 publisher는 변화에 따라 Object의 newInstance 생산
    - All Control objects list 사용자가 선택하려는 대상 포함
2. createPublisherFor
    - 시스템 UI가 사용
    - 사용하고자 하는 통제 대상을 알고 업데이트를 하길 원하는지 사용자에게 물어볼 때 사용.
3. Control ProviderService
    - 사용자가 언제 통제 버튼을 선택할 지 알기 위한 목적

        → Perform ControlAction

    - 클릭이나 드래그등 활동의 OK 반응을 전송해서 사용자가 반응을 인지 했음을 알려 준다.

### Code example

![Whats%20new%20in%20System%20UI%20/_______()_6-53_screenshot.png](What%20s%20new%20in%20System%20UI%20/_______()_6-53_screenshot.png)

control을 이끌어내는 방법을 시스템 UI에게 알려주며 마지막에 일부 상태를 보유한 control builder 에서 모든 사항을 함께 적용.

---

# Media Player

![Whats%20new%20in%20System%20UI%20/_______()_8-6_screenshot.png](What%20s%20new%20in%20System%20UI%20/_______()_8-6_screenshot.png)

1. MediaStyle Notification (Android Lollipop)
    - 미디어 재생을 위해 계속 remote view 를 사용하지 않아도 됨
2. Dedicated persistent space 생성
    - 빠른 설정에서 쉬운 접근으로 미디어 control 가능
    - lock screen 에도 알림을 추가
    - 미디어 플레이어에 output 선택 가능
3.  Media BrowserService
    - 멈췄던 곳에서 다시 재생

        재생이 중지된 몇 시간 후에, 또는 재부팅 후에도 가능
