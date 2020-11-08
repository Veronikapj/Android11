##### Android 11 Weeks 1 Week - Codelabs

# People: Conversations and Bubbles

[코드랩 링크](https://developer.android.com/codelabs/android-people?return=https%3A%2F%2Fdeveloper.android.com%2Fcourses%2Fpathways%2Fandroid-week1-people-identity%23codelab-https%3A%2F%2Fdeveloper.android.com%2Fcodelabs%2Fandroid-people#1)

## Codelab Overview

안드로이드 11에서 추가 된 대화 알림 표시 방식인 Bubbles에 대한 Codelab.

### Codelab을 통해 습득 할 사항

1. 대화 알림을 표시하는 방법
2. Bubbles를 지원하는 방법
3. Foreground 실행 중 Bubbles를 보여주는 방법

### 준비 사항

1. 기본 Kotlin 지식
2. 기본 notification 지식.
3. Android 11 Beta 1 이상의 기기 / 시뮬레이터
4. Android Studio 4.0.0 이상



## Do Codelabs

### 1. Clone Codelabs Repo

```git
git clone https://github.com/googlecodelabs/android-people.git
```


### 2. Update Codes

1. Useing MessageStyle

   대화형 UI 를 사용하기 위해 예시에 사용 된 setContentText 대신에 setStyle을 통해 MessageStyle 을 적용
   ```kotlin
    .setStyle ( 
      Notification.MessagingStyle (user) 
          .apply { 
              val lastId = chat.messages.last (). id 
              for (chat.messages의 메시지) { 
                  val m = Notification.MessagingStyle.Message ( 
                      message.text, 
                      message.timestamp, 
                      if (message.isIncoming) person else null 
                  ) .apply { 
                      if (message.photoUri! = null) { 
                          setData (message.photoMimeType, message.photoUri) 
                      } 
                  } 
                  if (message.id <lastId) { 
                      addHistoricMessage (m)
                  } else { 
                      addMessage (m) 
                  } 
              } 
          } 
          .setGroupConversation (false) 
    ) 
    .setWhen (chat.messages.last (). timestamp)
   ```

2. Create Dynamic shortcut

   동적 바로가기에 대한 버튼 대응을 위해 신규 코드 추가

   ```kotlin
   var shortcuts = Contact.CONTACTS.map { contact ->
    val icon = Icon.createWithAdaptiveBitmap(
        context.resources.assets.open(contact.icon).use { input ->
            BitmapFactory.decodeStream(input)
        }
    )
    ShortcutInfo.Builder(context, contact.shortcutId)
        .setLocusId(LocusId(contact.shortcutId))
        .setActivity(ComponentName(context, MainActivity::class.java))
        .setShortLabel(contact.name)
        .setIcon(icon)
        .setLongLived(true)
        .setCategories(setOf("com.example.android.bubbles.category.TEXT_SHARE_TARGET"))
        .setIntent(
            Intent(context, MainActivity::class.java)
                .setAction(Intent.ACTION_VIEW)
                .setData(
                    Uri.parse(
                        "https://android.example.com/chat/${contact.id}"
                    )
                )
        )
        .setPerson(
            Person.Builder()
                .setName(contact.name)
                .setIcon(icon)
                .build()
        )
        .build()
   }
   if (importantContact != null) {
    shortcuts = shortcuts.sortedByDescending { it.id == importantContact.shortcutId }
   }
   val maxCount = shortcutManager.maxShortcutCountPerActivity
   if (shortcuts.size > maxCount) {
    shortcuts = shortcuts.take(maxCount)
   }
   shortcutManager.addDynamicShortcuts(shortcuts)
   ```

3. Mapping Activity & Dynamic shortcut
   방금 생성한 동적 바로가기를 MainActivity에 Mapping
   ```xml
     <activity 
        android : name = "com.example.android.people.MainActivity" 
        ...> 
        <! -...- > 
        <meta-data 
            android : name = "android.app.shortcuts" 
            android : resource = "@ xml / 바로 가기"/> 
    </ activity>
   ```

4. Associate the notification with a shortcut.
   ```kotlin
 // ... 
            .setCategory (Notification.CATEGORY_MESSAGE) 
            .setShortcutId (chat.contact.shortcutId) 
            .setLocusId (LocusId (chat.contact.shortcutId))
   ```
   
5. Set up BubbleMeta

   Bubble을 알림채널로 보내기 위해 BubbleMate 를 설정하는 코드 입력

   ```koltin
   .setBubbleMetadata(
    Notification.BubbleMetadata
        .Builder(
            PendingIntent.getActivity(
                context,
                REQUEST_BUBBLE,
                Intent(context, BubbleActivity::class.java)
                    .setAction(Intent.ACTION_VIEW)
                    .setData(contentUri),
                PendingIntent.FLAG_UPDATE_CURRENT
            ),
            icon
        )
        .setDesiredHeightResId(R.dimen.bubble_height)
        .build()
)
   ```

6. Add BubbleActivity

   ```xml
   <activity
    android:name="com.example.android.people.BubbleActivity"
    android:allowEmbedded="true"
    android:documentLaunchMode="always"
    android:resizeableActivity="true">
    ...
   </activity>

   ```



항상 모든 코드랩이 따라 할 때는 괜찮은데, 막상 실전에 쓰려고 한다면 많은 연습과 사전 조사가 더욱 필요 할 것 같음.

우선 초심자라면 Notification에 대한 기초 코드랩 부터 차근차근 진행 후에 이 코드랩을 진행하는 것이 좋을 것 같음.

[Notification과 관련 Android Docs](https://developer.android.com/guide/topics/ui/notifiers/notifications)

[Notification과 관련 Codelabs](https://codelabs.developers.google.com/codelabs/advanced-android-kotlin-training-notifications/index.html?index=..%2F..index#0)

[Bubbles와 관련 Android Docs](https://developer.android.com/guide/topics/ui/bubbles)

[Original Git Repo](https://github.com/googlecodelabs/android-people)

[Fork Git Repo](https://github.com/CuroGom/android-people/tree/done)
