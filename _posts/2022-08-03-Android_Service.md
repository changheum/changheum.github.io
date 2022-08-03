---
title:  "Android Service"
excerpt: ""

# categories:
#   - android
tags:
  - [android, service]

toc: true
toc_sticky: true
 
date: 2022-08-03
last_modified_at: 2022-08-03

layout: post
summary: android service class
author: changheum
category: [android]
thumbnail: /assets/img/documents.png
keywords: [android, service]
usemathjax: true
permalink: /blog/android_service
---

( 참고 출처 : 밍나.log https://velog.io/@alsgk721/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-16-Service)  
( 참고 출처 : 기록, 성장, 공유 - Android  https://nosorae.tistory.com/entry/Android-Service-%EC%99%80-App-Life-cycle-%EC%9D%84-%EC%97%B0%EA%B4%80%EC%A7%80%EC%96%B4-%EC%83%9D%EA%B0%81%ED%95%B4%EB%B3%B4%EA%B8%B0 )  
( 참고 출처 : 블랜디 - https://qmffosel77.tistory.com/49)
```kotlin
class MyService : Service() {
	override fun onBind(intent: Intent): IBinder? {
    	return null
    }
}
```
안드로이드 4대 컴포넌트 중 하나. (액티비티, 서비스, 브로드캐스트 수신자, 콘텐츠 제공자)  
서비스 클래스는 싱글톤으로 동작 함.  

- 백그라운드에서 Thread 나 Couroutine 등을 이용하여 동작이 가능
- 화면 출력 기능이 없다

# 서비스의 타입
- Foreground  
유저의 눈에 보이지 않지만 인지할 수 있는 작업 수행  
반드시 Notification 을 보여주어야. (그래야 유저가 서비스가 실행중임을 알 수 있으니까)  
서비스가 만들어지고 5초이내에 startForeground 호출 해야함  
- Background  
유저가 인지하지 못하는 작업들을 수행. 예를들면 저장소 압축  
- Bound  
다른 앱 컴포넌트가 서비스에 bindService 메소드를 호출해서 바인드 됐을 때만 실행  
여러 컴포넌트가 동시에 바인드될 수 있고 모두 unbind 되면 서비스가 파괴됨  
client/server 인터페이스를 제공하여 바인드한 컴포넌트가 요청하고 결과를 받는 상호작용을할 수 있음  
이것으로 IPC 까지 가능  

# 서비스 구현 및 사용
(1) Service를 상속받아 구현  
  - onBind()는 필수로 구현  

```java
public abstract class CarAppService extends Service {
  @Override
      @CallSuper
      @NonNull
      public final IBinder onBind(@NonNull Intent intent) {
          return requireNonNull(mBinder);
      }
}

public final class NavigationCarAppService extends CarAppService {
  @Override
  @NonNull
  public Session onCreateSession() {
    NavigationSession session = new NavigationSession();

    return session;
  }
}
```

(2) AndroidManifest.xml 에 등록
- name 생략이 불가능
- __Intent__ 로 실행이 되며, 명시적 인텐트로 실행시에는 name만 써도 되지만,  
  암시적 인텐트로 실행하려면 __intent filter__ 작성이 필요하다.  

```xml
<service
  android:name="androidx.car.app.sample.navigation.common.car.NavigationCarAppService"
  android:foregroundServiceType="location"
  android:exported="true">
    <intent-filter>
      <action android:name="androidx.car.app.CarAppService" />
      <category android:name="androidx.car.app.category.NAVIGATION"/>
    </intent-filter>
</service>
```
  
(3) Callback Method 구현  
- onBind  
`public abstract IBinder onBind(Intent intent)`
다른 컴포넌트가 서비스에 바인드할 때 시스템이 bindService 를 호출해서 onBind 를 호출  
앞서 Bound 타입 서비스는 client/server 구조를 가진다고 했는데  
개발자는 여기서 리턴해주는 IBinder 로 clients 가 서비스와 소통할 수 있게 해준다.  
  
- onStartCommand  
`public @StartResult int onStartCommand(Intent intent, @StartArgFlags int flags, int startId)`
다른 컴포넌트가 서비스를 동작시키면서 시스템이 startService 를 호출해서 onStartCommand 를 호출  
일단 시작하면 백그라운드에서 계속 실행될 수 있기 때문에  
개발자는 시스템 리소스나 배터리 낭비를 막기 위해 (binding 만 원하는 게 아니라면)  
서비스 내에서 stopSelf 나 서비스를 싲가한 다른 컴포넌트에서 stopService 를 호출하여 실행을 정지시킬 필요가 있다.  
같은 서비스를 여러 번 실행시키는 상황에서는 stopService 를 하나만 호출해도 여러 개가 모두 죽기 때문에  
stopSelf 에 id 를 전달해서 하나만 종료시킬 수 있다.  
onStartCommand 에서 반환하는 Int 값으로 서비스가 죽었을 때 다시시작하는 방식을 설정해줄 수 있다.  
  
- onCreate  
서비스가 초기에 만들어질 때 한 번 호출됨. 여기서 리소스 초기화  
  
- onDestroy  
서비스가 더이상 사용되지 않고 파괴될 때, 여기서 리소스 해제  

(4) startService()에 의해 Service 실행
```java
val intent = Intent(this, MyService::class.java)
startService(intent)
```

(5) stopService()에 의해 Service 종료
```java
val intent = Intent(this, MyService::class.java)
stopService(intent)
```


# 서비스 라이프 사이클
![Android Service](/assets/img/posts/2022-08-03-Android_Service.png)  
(1) startService() 로 구동  
  startService() 함수로 서비스를 실행하려면, 해당 서비스를 Intent에 담아서 매개변수로 전달한다.  
  외부 앱의 서비스라면 암시적 인텐트로 실행해야 하므로, setPackage() 함수를 이용해 앱의 패키지명을 명시한다.  
```kotlin
val intent = Intent(this, MyService::class.java)
intent.setPackage("com.example.test_outer")
startService(intent)
```
stopService()함수로 서비스가 종료되면 바로 전에 onDestroy()함수가 호출된다.
```kotlin
val intent = Intent(this, MyService::class.java)
stopService(intent)
```
![startService](/assets/img/posts/2022-08-03-Android_Service_startService.png)  
데이터를 전달받는 곳에 __BroadcastReceiver__ 를 정의하고,  
데이터 전달이 필요할 때, BroadcastReceiver를 실행시키면 Broadcast Intent 에 Extra 데이터로 전달  

(2) bindService() 로 구동시,  
onCreate() -> onBind()함수가 호출되고 서비스가 실행  
실행되고 있는 상태에서 bindService() 함수로 실행하면 onBind() 함수만 다시 호출  
  
bindService()함수로 서비스를 실행하려면, 먼저 ServiceConnection 인터페이스를 구현한 객체를 준비해야 한다.
```kotlin
val connection: ServiceConnection = object : ServiceConnection {
  override fun onServiceConnected(name: ComponentName?, service: IBinder?) { } 
  //bindService()함수로 서비스를 구동할 때 자동으로 호출
  override fun onServiceDisconnected(name: ComponentName?) { } 
  //unbindService() 함수로 서비스를 종료할 때 자동으로 호출
}

val intent = Intent(this, MyService::class.java)
bindService(intent, connection, Context.BIND_AUTO_CREATE)
//BIND_AUTO_CREATE : 서비스가 실행상태가 아니더라도 객체를 생성해서 실행   
```
unbindService()함수로 서비스를 종료하면 onUnbind() -> onDestroy() 함수까지 실행된다.  
```kotlin
unbindService(connection)
```
![bindService](/assets/img/posts/2022-08-03-Android_Service_bindService.png)  
- bindService() 는 서비스가 자신을 실행시킨 곳에 객체로 바인딩한다는 의미  
- 서비스를 구동시킨 결과를 실행시킨 곳에 객체로 넘겨준다.(바인딩한다)  
- 서비스를 구동시킨 곳에 객체를 넘겨주고,  
  객체의 함수를 호출하여 매개변수 또는 리턴값으로 데이터를 주고 받을 수 있다.  