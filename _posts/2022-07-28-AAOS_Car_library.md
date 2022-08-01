---
title:  "AAOS 용 androidx.car.app 라이브러리 사용"
excerpt: "AAOS car library"

# categories:
#   - android
tags:
  - [android, aaos]

toc: true
toc_sticky: true
 
date: 2022-07-25
last_modified_at: 2022-07-28

layout: post
summary: Using the Android for Cars App Library
author: changheum
category: [android]
# thumbnail: /assets/img/posts/code.jpg
keywords: [android, aaos]
usemathjax: true
permalink: /blog/aaos_car_library
---

https://developer.android.com/training/cars/apps?hl=ko

__자동차용 Android 앱라이브러리__ 를 사용하면, 내비게이션/주차/충전 앱을 자동차에서 사용할 수 있다.  

# 템플릿!!
<b>운전자 주의 분산 행동 표준을 충족하도록 설계된 일련의 템플릿을 제공한다.  </b> 
The Android for Cars App Library allows you to bring your navigation and point of interest (POI) apps to the car.  
It does so by providing a set of templates designed to meet driver distraction standards  
and taking care of details such as the variety of car screen factors and input modalities.  

# Key terms and concepts
## Models and Templates
유저인터페이스는 모델객체의 그래프로 표현됨.  
모델 객체의 그래프는 모델 객체가 속한 템플릿에서 허용하는 여러 방법으로 정렬 가능함.  
모델에는 사용자에게 전달될 정보(텍스트, 이미지형식)과 시각적 표시 측면을 구성하는 속성(텍스트 색상 또는 이미지 크기)가 포함.  
  
__호스트__ 가 모델을 운전자 주의 분산 행동 표준을 충족하는 뷰로 변환.  
__호스트__ 는 자동차 화면 요소 및 입력 모달리티와 같은 세부사항을 처리.  
  
## Host
앱이 자동차에 실행되도록, <b>라이브러리의 API에서 제공하는 기능</b>을 구현한 백엔드 구성요소  
호스트의 역할은 앱을 찾고, 수명주기를 관리하고, 모델을 뷰로 변환하고, 앱에 사용자 상호작용을 알림  
휴대기기에서, 이 <b>호스트</b>는 <b>Android Auto</b> 로 구현됨  

## Template restrictions
모델의 컨텐츠에 제한을 가하는 여러가지 템플릿이 있음.  
예를 들어, 목록 템플릿에는 항목 수에 제한이 있음.  
템플릿이 작업 흐름을 형성하기 위해 연결할 수 있는 방법에도 제한이 있음.  
예를 들어 앱은 화면 스택에 템플릿을 최대 5개만 푸시 가능.  
  
호스트는 주어진 작업에 표시할 템플릿 수를 최대 5개로 제한하고, 5개중 마지막 템플릿은 다음중 하나여야함.  
1. NavigationTemplate (public final class NavigationTemplate extends Object implements Template)  
2. PaneTemplate (public final class PaneTemplate extends Object implements Template)  
3. MessageTemplate (public final class MessageTemplate extends Object implements Template)  
  
## Screen
앱이 사용자에게 표시하는 사용자 인터페이스를 관리하기 위해 구현하는 클래스 (라이브러리에서 제공)  
수명 주기가 있으며, 앱에서 표시할 템플릿을 전송할 수 있는 메커니즘을 제공.  
화면 스택으로 푸시 될 수 있고, 팝 될수도 있으므로, 템플릿 제한 사항을 준수할 수 있음.  
![Screen 수명 주기](/assets/img/posts/2022-07-28_screen-lifecycle.png)

## CarAppService
호스트에서 검색/관리할 수 있도록 앱에서 구현해야 하는 추상 Service 클래스.  
CarAppService.createHostValidator 를 사용해서 호스트 연결을 신뢰할 수 있는지 검증한 후,  
CarAppService.onCreateSession 을 사용해서 각 연결에 Session 인스턴스를 제공하는 역할.  

## Session
앱이 CarAppService.onCreateSession 을 사용하여 구현하고 반환해야 하는 추상 클래스.  
자동차 화면에 정보를 표시하는 진입점 역할.  
앱이 표시되거나 숨겨질 때와 같은 앱의 현재 상태를 자동차 화면에 알려주는 수명 주기를 가지고 있음.  
  
Session 이 시작되면, 호스트에서는 Session.onCreateScreen 메서드를 사용하여 표시할 초기 Screen을 요청함.  
  
![Session 수명 주기](/assets/img/posts/2022-07-28_carappservice-session-lifecycle.png)  

# Install the library
앱에 라이브러리를 추가하는 방법  
https://developer.android.com/jetpack/androidx/releases/car-app#declaring_dependencies  
'''Groovy
dependencies {
    implementation "androidx.car.app:app:1.1.0"

    // For Android Auto specific functionality
    implementation "androidx.car.app:app-projected:1.1.0"

    // For Android Automotive specific functionality
    implementation "androidx.car.app:app-automotive:1.1.0"

    // For testing
    testImplementation "androidx.car.app:app-testing:1.2.0-rc01"
}
'''

# Configure your app’s manifest files
```XML
<application>
    ...
   <service
       ...
        android:name=".MyCarAppService"
        android:exported="true"
        android:label="@string/my_app_name"
        android:icon="@drawable/my_app_icon">
      <intent-filter>
        <action android:name="androidx.car.app.CarAppService"/>
        <category android:name="androidx.car.app.category.PARKING"/>
      </intent-filter>
    </service>
    ...
<application>
```  
## Declare your CarAppService
호스트는 CarAppService 구현을 통해서 앱에 연결됨.  
매니페스트에서 CarAppService 를 선언하여 호스트에서 앱을 검색/연결 할 수 있음.  
  
`<action android:name="androidx.car.app.CarAppService"/>`  

## Supported App Categories
자동차 앱 서비스를 선언할 때 인텐트 필터에서 다음 지원되는 카테고리 값 중 하나 이상을 추가하여 앱의 카테고리를 선언합니다.  
   
androidx.car.app.category.NAVIGATION: 세부 경로 내비게이션 안내를 제공하는 앱입니다.  
androidx.car.app.category.PARKING: 주차 위치 찾기와 관련된 기능을 제공하는 앱입니다.  
androidx.car.app.category.CHARGING: 전기자동차 충전소 찾기와 관련된 기능을 제공하는 앱입니다.  
  
`<category android:name="androidx.car.app.category.PARKING"/>`  

## Specify the app name and icon
호스트가 시스템UI에서 앱을 표시하는데 사용할 수 있는 앱 이름과 아이콘을 지정  
(service 요소에서 라벨이나 아이콘이 선언되지 않으면 호스트는 애플리케이션(`<`application`>`엘리먼트)에 지정된 값으로 대체합니다.)  
```
android:label="@string/my_app_name"
android:icon="@drawable/my_app_icon"  
```

## Set a custom theme
자동차앱(Car app)의 테마를 커스텀 하고 싶을 때는, manifest 파일의 `<`meta-data`>` Element를 이용함  
```
<meta-data
    android:name="androidx.car.app.theme"
    android:resource="@style/MyCarAppTheme />
```
그 다음, style resource 를 선언하여 커스텀 앱테마에 다음 속성을 설정
```
<resources>
  <style name="MyCarAppTheme">
    <item name="carColorPrimary">@layout/my_primary_car_color</item>
    <item name="carColorPrimaryDark">@layout/my_primary_dark_car_color</item>
    <item name="carColorSecondary">@layout/my_secondary_car_color</item>
    <item name="carColorSecondaryDark">@layout/my_secondary_dark_car_color</item>
    <item name="carPermissionActivityLayout">@layout/my_custom_background</item>
  </style>
</resources>
```

# Car App API level
Car App Library는 자체적인 API 레벨을 정의함. 이를 통해, 차량의 Template host 가 어떤 라이브러리를 지원하는지 알수 있음.   
CarContext 의 getCarAppApiLevel() 함수로 호스트에서 지원하는 가장 높은 Car App API Level을 알 수 있음.  
(public class CarContext extends ContextWrapper)  

앱에서 요구하는 최소의 Car App API Level 을 AndroidManifest.xml 파일에 선언해야 함.  
```
<manifest ...>
    <application ...>
        <meta-data
            android:name="androidx.car.app.minCarApiLevel"
            android:value="1"/>
    </application>
</manifest>
```

# Create your CarAppService and Session
앱은 __CarAppService__ 클래스를 상속해야 하고, __CarAppService.onCreateSession()__ 함수를 구현해야함.  
이 함수는, 현재 host 로 연결되어있는 Session 인스턴스를 반환해야 함.  
```java
public final class HelloWorldService extends CarAppService {
    ...
    @Override
    @NonNull
    public Session onCreateSession() {
        return new HelloWorldSession();
    }
    ...
}
```
  
Session 인스턴스는, 앱이 처음 시작될 때 사용할 Screen 인스턴스를 반한해야 함.  
```java
public final class HelloWorldSession extends Session {
    ...
    @Override
    @NonNull
    public Screen onCreateScreen(@NonNull Intent intent) {
        return new HelloWorldScreen();
    }
    ...
}
```
앱이 홈이나 랜딩화면이 아닌 화면에서 시작하는 시나리오를 처리하려면, (ex. 딥링크(앱의 컨텐츠로 연결되는 링크) 처리)  
__onCreateScreen__ 으로 부터 리턴을 받기 전에  
__ScreenManager.push__ 를 사용하여 screen 의 back stack을 pre-seed 할 수 있다.  
Pre-seeding 을 사용하면, 사용자가 첫화면에서 이전 스크린으로 이동할 수 있다.  

# Create your start screen
Screen 클래스를 확장하는 클래스를 정의하고, 자동차 화면에 표시할 UI 상태를 나타내는 Template 인스턴스를 반환하는
__Screen.onGettemplate__ 메서드를 구현하여, 앱에서 표시하는 화면을 만든다.  
  
__PaneTemplate__ 템플릿을 사용한 간단한 Screen 선언 예제
```java
public class HelloWorldScreen extends Screen {
    @NonNull
    @Override
    public Template onGetTemplate() {
        Row row = new Row.Builder().setTitle("Hello world!").build();
        Pane pane = new Pane.Builder().addRow(row).build();
        return new PaneTemplate.Builder(pane)
            .setHeaderAction(Action.APP_ICON)
            .build();
    }
}
```

# The CarContext class
__CarContext__ 클래스 : __ContextWrapper__ 의 서브 클래스.  
__Session__ 및 __Screen__ 인스턴스에서 액세스할 수 있으며, <b>자동차 서비스 액세스 권한</b>을 제공.  
예를 들어,  
- 화면 스택 관리용 ScreenManager  
- 일반 앱 관련 기능(예를 들면 내비게이션 앱의 지도를 그리는 Surface 객체 액세스)을 위한 AppManager
- 세부 경로 안내 내비게이션 앱에서 내비게이션 메타데이터 및 기타 내비게이션 관련 이벤트를 호스트에 전달하는 NavigationManager 
등이 있음. 
  
그 외에 제공하는 기능..    
car screen에서 설정을 이용해서 drawable resource를 로딩할수 있게 하고,  
인텐트를 이용해서 자동차의 app을 시작하고,  
내비게이션 앱에서 지도를 dark mode로 표시해야 하는지 알릴 수 있는 기능도 제공함.  

# Implement screen navigation
화면 내비게이션 구현!!!!!  
앱은 여러개의 서로다른 스크린을 보여주기도 한다.  
각 스크린은 서로다른 여러가지 Template을 활용할 수 있다.  
사용자가 화면의 인터페이스와 상호 작용할 때 탐색할 수 있는 여러 화면을 표시하는 경우가 많음.  


__ScreenManager__ 클래스는 뒤로가기로 보여줄 수있는 화면을 푸시하는데 사용하는 <b>화면 스택</b>을 제공한다.  
  
(예) 메시지 템플릿에 뒤로 가기 작업을 추가하는 방법과, 사용자가 선택할 때 새 화면을 푸시하는 작업
```java
MessageTemplate template = new MessageTemplate.Builder("Hello world!")
    .setHeaderAction(Action.BACK)
    .addAction(
        new Action.Builder()
            .setTitle("Next screen")
            .setOnClickListener(
                () -> getScreenManager().push(new NextScreen(getCarContext())))
            .build())
    .build();
```
__Action.BACK__ 객체는 __ScreenManager.pop__ 을 자동으로 호출하는 표준 __Action__ 임.  
이 동작은 __CarContext__ 에서 사용 가능한 __OnBackPressedDispatcher__ 인스턴스를 사용하여 재정의 가능하다.  

화면스택에는 화면이 최대 5개 있을 수 있다.(템플릿 제한 사항, 앱의 안전한 동작을 위해서)   
템플릿 제한 사항 링크 https://developer.android.com/training/cars/apps#template-restrictions  

# Refresh the contents of a template
앱에서 __Screen.invalidate__ 함수를 호출하여 __Screen__ 의 컨텐츠를 무효화(to be invalidated) 가능.  
그럼 호스트는 __Screen.onGetTemplate__ 함수를 다시 호출하여 새컨텐츠가 있는 템플릿을 검색할 것임.  

Screen 을 새로고침할 때, 업데이트 가능한 템플릿의 특정 컨텐츠를 파악하는 것이 중요하다.  
호스트가 새 템플릿을 템플릿 할당량(the template quota)에 포함하지 않도록...?  
  
When refreshing a Screen, it is important to understand the specific content in the template that can be updated so that the host will not count the new template against the template quota. See Template restrictions for more details.  
  
# Interact with the user
앱이 모바일 앱과 유사한 패턴을 사용해서 유저와 상호작용할 수 있다.  

## Handle user input
앱은 (리스너를 지원하는..)모델에 적절한 리스너를 전달하여 사용자 입력에 응답할 수 있다.  
다음 스니펫은 Action 모델을 만드는 방법을 보여주며, 이 모델은 앱 코드에서 정의된 메서드를 다시 호출하는 OnClickListener 를 설정한다.  
```java
Action action = new Action.Builder()
    .setTitle("Navigate")
    .setOnClickListener(this::onClickNavigate)
    .build();
```
__onClickNavigate()__ 함수는 __CarContext.startCarApp__ 을 이용해서 default navigation car app 을 실행 할 수 있다!
```java
private void onClickNavigate() {
    Intent intent = new Intent(CarContext.ACTION_NAVIGATE, Uri.parse("geo:0,0?q=" + address));
    getCarContext().startCarApp(intent);
}
```
인텐트로 자동차앱 시작(https://developer.android.com/training/cars/apps#start-car-app)에서  
__ACTION_NAVIGATE__ 의 인텐트 형식을 포함하여, 앱을 시작하는 여러가지 방법을 볼 수 있음.  
  
  
사용자가 휴대 기기에서 상호작용을 계속하도록 안내하는 일부 작업은 <b>자동차가 주차된 때만 허용</b>됨.  
__ParkedOnlyOnclickListener__ 를 사용하여 이런 작업을 구현 가능.  
자동차가 주차되어 있지 않으면, 허용되지 않는 작업이라는 메시지를 표시함.  
자동차가 주차되어 있으면, 코드가 정상적으로 실행됨.  
  
(예제) __ParkedOnlyOnClickListener__ 를 사용해서 휴대기기에서 설정 화면 열기
```java
Row row = new Row.Builder()
    .setTitle("Open Settings")
    .setOnClickListener(ParkedOnlyOnClickListener.create(this::openSettingsOnPhone))
    .build();
```

## Display notifications
휴대기기로 전송된 알림은 __CarAppExtender__ 로 확장된 경우에만 자동차 Screen 에 표시됨.  
자동차의 Screen 에 표시될 때, CarAppExtender 에서 일부속성(컨텐츠 제목, 텍스트, 아이콘, 작업actions) 은 재정의 될 수 있음.  
  
```java
Notification notification = new NotificationCompat.Builder(context, NOTIFICATION_CHANNEL_ID)
    .setContentTitle(titleOnThePhone)
    .extend(
        new CarAppExtender.Builder()
            .setContentTitle(titleOnTheCar)
            ...
            .build())
    .build();
```

알림(Notification)은 아래의 유저 인터페이스에 영향을 미칠 수 있음.  
- 헤드업 알림(HUN)은 사용자에게 표시될 수 있습니다.
- 알림 센터의 항목(An entry in the notification center)은 추가될 수 있고 선택적으로 배지(badge)가 레일(rail)에 표시됩니다.
- 내비게이션 앱의 경우 세부 경로 안내 알림(Turn-by-turn notification)에 설명된 대로 알림이 레일 위젯(rail widget)에 표시될 수 있습니다.

(참고: Android Automotive OS에서 OEM은 Turn-by-turn HUN을 표시하지 않도록 선택할 수 있습니다.)  
  
앱은 notification's priority를 이용하여 유저 인터페이스에 영향을 미치는 각각의 notification 을 구성하는 방법을 설정할 수 있다.  
(CarAppExtender documentation 참고 : https://developer.android.com/reference/androidx/car/app/notification/CarAppExtender)  

예를 들어, __NotificationCompat.Builder.setOnlyAlertOnce__ 를 true 값으로 호출하면 우선순위가 높은 알림이 __HUN__ 으로 __한 번__ 만 표시됩니다.  
(참고: 라이브러리는 <b>휴대기기가 아닌 자동차 화면에만</b> 알림을 전송하는 기능을 <b>제공하지 않습니다.</b>)  


# Show toasts
앱에서는 <b>CarToast</b>를 사용하여 토스트 메시지를 표시
```java
CarToast.makeText(getCarContext(), "Hello!", CarToast.LENGTH_SHORT).show();
```

# Request permissions
Permission 이 필요할 때, <b>CarContext.requestPermissions()</b> 를 사용한다.  
(안드로이드 권한 표준 규칙 : https://developer.android.com/guide/topics/permissions/overview?hl=ko)  
- 표준 Android API 를 사용하지 않으면, 권한 대화상자를 생성하기 위해 자체 Acitivy를 실행하지 않아도 되는 장점이 있음.  
- 플랫폼에 종속된 흐름을 만들지 않아도, Android Auto와 Android Automotive OS에서 동일한 코드를 사용 가능.  
  
Android Auto 에서는, 권한 다이얼로그(permissions dialog)가 폰에서 보여짐.  
기본적으로, 다이얼로그 뒤에는 background가 없다.  
  
custom background 를 설정하기 위해서는,  
__AndroidManifest.xml__ 파일에 __car app theme__ 를 선언하고,  
자동차 앱 테마(car app theme)에 __carPermissionActivityLayout__ 속성을 설정해 주어야 함.  
  
(1) __car app theme__ 를 선언  
```xml
<meta-data
    android:name="androidx.car.app.theme"
    android:resource="@style/MyCarAppTheme />
```

(2) __carPermissionActivityLayout__ 속성을 설정    
```xml
<resources>
  <style name="MyCarAppTheme">
    <item name="carPermissionActivityLayout">@layout/my_custom_background</item>
  </style>
</reso7urces>
```

# Start a car app with an intent
__CarContext.startCarApp__ 메서드를 호출하여, 다음 중 하나의 작업을 실행할 수 있다.  
- 다이얼러를 열어 __전화__  
- __기본 내비게이션 자동차 앱__ 을 사용하여 세부 경로 안내 내비게이션을 시작
- 인텐트로 자체 앱을 시작  

# Template restrictions
템플릿 제한 사항.  
호스트는 주어진 작업에 대해 보여줄수 있는 템플릿을 최대 5개로 제한한다.  
또한, 템플릿은 아래 3가지 타입 중 하나 여야 한다.  

1. NavigationTemplate  
2. PaneTemplate  
3. MessageTemplate  

이 제한 사항은 __Template__ 의 갯수에 대한 제한이고, Stack 내 __Screen__ 의 instance 갯수에 대한 제한은 아니다.  
예를 들어, 앱이 Screen A 에서 Template을 2개 전송하고, Screen B를 Push 한다면, Template을 3개 더 전송 할 수 있다.  
또는, 각 화면이 단일 템플릿을 전송하는 경우, 앱은 화면 인스턴스 5개를 __ScreenManager__ 스택으로 푸시할 수 있다.  
  
(주의)  
템플릿 할당량이 소진된 상태에서 앱이 새 템플릿을 전송하려고 하면 호스트는 앱을 종료하기 전에 사용자에게 오류 메시지를 표시합니다.  
   

(참고)  
```
Android Auto로 개발할때는,  
개발자 모드를 사용으로 설정하고 설정에서 디버그 오버레이 사용 설정(Enable debug overlay)을 선택하여  
화면에 오버레이된 현재 단계 수를 확인할 수 있다.  
AAOS(Android Automotive OS)에서는, `adb shell setprop log.tag.CarApp.H.Dis VERBOSE` 를 실행한 후에  
logcat 의 아웃 풋에서 현재 스텝의 숫자를 볼 수 있다.  
```

# Template refreshes
특정 콘텐츠 업데이트는 템플릿 제한에 포함되지 않습니다.  
일반적으로, 앱이 이전 템플릿과 유형이 동일하고 기본 콘텐츠도 동일한 새 템플릿을 푸시한다면 새 템플릿은 할당량에 포함되지 않습니다. 
예를 들어, <b>ListTemplate에서 행의 전환 상태를 업데이트</b>해도 할당량에 포함되지 않습니다.  
새로고침으로 간주할 수 있는 콘텐츠 업데이트 유형에 관한 자세한 내용은 개별 템플릿 문서를 참고  

## Back operations
작업 내에서 하위 흐름을 사용 설정하기 위해 호스트는 앱이 ScreenManager 스택에서 Screen을 표시할 때를 감지하고 앱이 뒤로 이동할 템플릿 수에 따라  
나머지 할당량을 업데이트합니다.  
  
예를 들어, Screen A에서 앱이 템플릿을 2개 전송하고, Screen B를 푸시한 후 템플릿을 2개 더 전송하면 앱에 남은 할당량은 하나입니다.  
앱이 다시 화면 A로 돌아오면 호스트는 할당량을 3으로 재설정합니다. 앱이 템플릿 2개만큼 뒤로 이동했기 때문입니다.  
  
화면으로 다시 돌아올 때 앱은 이 화면에서 마지막으로 전송한 것과 동일한 유형의 템플릿을 전송해야 합니다.  
다른 템플릿 유형을 전송하면 오류가 발생합니다.  
  
그러나 뒤로 작업 시 유형이 동일하게 유지되는 한 앱은 할당량에 영향을 주지 않고 템플릿의 콘텐츠를 자유롭게 수정할 수 있습니다.  

## Reset operations
특정 템플릿에는 작업의 끝을 나타내는 특수한 의미 체계(special semantics)가 있습니다.  
    
예를 들어, __NavigationTemplate__ 은  
Screen에 계속 유지되고, 새로운 세부 경로 안내가 계속 새로고침 될 것으로 예상되는 뷰입니다.  
이러한 템플릿 중 하나에 도달하면 호스트는 템플릿 할당량을 재설정하여 템플릿을 새 작업의 첫 번째 단계인 것처럼 간주하므로  
앱에서 새 작업을 시작할 수 있습니다. 호스트에서 재설정을 트리거하는 템플릿은 개별 템플릿 문서를 참고하세요.  
  
호스트가 인텐트를 수신하여 알림 작업이나 런처에서 앱을 시작하면 할당량도 재설정됩니다.  
이 메커니즘으로 앱은 알림에서 새 작업 흐름을 시작할 수 있고 앱이 이미 바인딩되고 포그라운드에 있더라도 마찬가지입니다.  
  
앱 알림(app’s notifications)을 자동차 화면에 표시하는 방법 [Display notifications](https://developer.android.com/training/cars/apps#display-notifications)  
앱 알림에서 앱을 시작하는 방법  [Start a car app with an intent](https://developer.android.com/training/cars/apps#start-car-app)  
  
# Connection API
CarConnection API를 사용하여 앱이 Android Auto 또는 Android Automotive OS에서 실행되고 있는지 확인할 수 있습니다.  
```java
new CarConnection(getCarContext()).getType().observe(this, this::onConnectionStateUpdated);
```
다음과 같이, observer 에서 연결 상태의 변경 사항에 반응할 수 있다.  
``` java
private void onConnectionStateUpdated(int connectionState) {
  String message;
  switch(connectionState) {
    case CarConnection.CONNECTION_TYPE_NOT_CONNECTED:
      message = "Not connected to a head unit";
      break;
    case CarConnection.CONNECTION_TYPE_NATIVE:
      message = "Connected to Android Automotive OS";
      break;
    case CarConnection.CONNECTION_TYPE_PROJECTION:
      message = "Connected to Android Auto";
      break;
    default:
      message = "Unknown car connection type";
      break;
  }
  CarToast.makeText(getCarContext(), message, CarToast.LENGTH_SHORT).show();
}
```
# Constraints API
각각의 차량들은 한번에 보여줄 수 있는 Item 의 갯수가 다르다.  
ConstraintManager를 사용하면, Template에서 적절한 수의 Item 제한을 체크할 수 있다.  
  
__CarContext__ 로부터 __ConstraintManager__ 를 가져올 수 있다.
```java
ConstraintManager manager = getCarContext().getCarService(ConstraintManager.class);
```

관련된 컨텐츠 제한 사항에 대해 __ConstraintManager__ 를 쿼리할 수 있다.  
예를 들어, 그리드에 표시할 수 있는 Item 수를 얻으려면,
```java
int gridItemLimit = manager.getContentLimit(ConstraintManager.CONTENT_LIMIT_TYPE_GRID);
```


# Add a sign-in flow
## Use AccountManager

# Add text string variants

# Car Hardware APIs
## Requirements
## CarInfo
## CarSensors
## Testing

# The CarAppService, Session and Screen Lifecycles
__LifesycleOwner__ 인터페이스를 구현하여, __Session__ 및 __Screen__ 클래스를 구현한다.  
사용자가 앱과 상호작용하는 수명 주기 콜백 다이어그램은 아래와 같다.  
  
## The lifecycles of a CarAppService and a Session
자세한 내용은 [Session.getLifecycle 메서드 문서](https://developer.android.com/reference/androidx/car/app/Session?hl=ko#getLifecycle())를 참고  
![Session 수명 주기](/assets/img/posts/2022-07-28_carappservice-session-lifecycle.png){style="background-color:white"}  

## The lifecycle of a Screen
자세한 내용은 [Screen.getLifecycle 메서드 문서](https://developer.android.com/reference/androidx/car/app/Screen#getlifecycle)를 참고  
![Screen 수명 주기](/assets/img/posts/2022-07-28_carappservice-session-lifecycle.png){: .class #custom-id-3 style="padding-top:0" key="value"} 
