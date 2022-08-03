---
title:  "[Effective Java] 1. 생성자 대신 정적 팩터리 메서드를 고려하라"
excerpt: "Static Factory Method"

# categories:
#   - android
tags:
  - [java, pattern]

toc: true
toc_sticky: true
 
date: 2022-07-25
last_modified_at: 2022-07-28

layout: post
summary: Static Factory Method
author: changheum
category: [java]
thumbnail: /assets/img/documents.png
keywords: [java, pattern]
usemathjax: true
permalink: /blog/effective_java_1
---

# 생성자 대신 정적 팩터리 메서드를 고려하라
정적 팩토리 메서드(static factory method) : 클래스의 인스턴스를 반환    
```java
// (ex) boolean 기본타입의 박싱 클래스(boxed class) 의 일부
public static Boolean valueOf(boolean b) {
  return b ? Boolean.TRUE : Boolean.FALSE;   
}
```

# 장점
1. 이름을 가질 수 있다. (반환 객체의 특성을 설명하기 좋음)  
  
2. 호출시, 매번 새로운 인스턴스를 생성할 필요가 없다. (캐싱, 재활용 가능)  
  -> 싱글턴, 플라이웨이트 패턴 등에서 사용  
  -> 불변값 클래스에서 동치인 인스턴스를 하나뿐인것으로 보장 가능(a==b 일때만 a.equals(b) 가 성립)  
  -> 열거 타입에서 인스턴스가 하나만 만들어짐을 보장  

3. 반환 타입의 하위 객체를 반환할 수 있다.  
  -> 구현 클래스를 공개 하지 않고도 객체를 반환할 수 있다. (item 28 인터페이스 기반 프레임워크의 핵심).  
      자바8 부터 인터페이스가 정적 메서드를 가질 수 없다는 제한이 풀렸다.  
      자바9 에서는 private 정적 메서드까지 허락하게 되었다.  
  
4. 입력 매개변수에 따라 다른 클래스의 객체를 반환할 수 있다.  
  -> 반환 타입의 하위 타입이기만 하면 어떤 클래스의 객체를 반환하든 상관이 없음.  
  
    (예를 들어,) 
    EnumSet 클래스(item 36) 에서는 public 생성자 없이 오직 정적 팩토리만 제공하는데,  
    OpenJDK 에서는 원소의 수에 따라 두 가지 하위 클래스중 하나의 인스턴스를 반환한다.   
    원소가 64개 이하이면, 원소들을 long 변수 하나로 관리하는 RegularEnumSet의 인스턴스를,  
    그 이상은 JumboEnumSet의 인스턴스를 반환한다.   
  
5. 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.  
   클라이언트를 구현체로 부터 분리해줌  
  
    서비스 제공자 프레임워크를 만드는 근간이된다. 대표적으로 JDBC가 있다.  
    서비스 제공자 프레임워크는, 3가지로 구성  
  - 서비스 인터페이스 : 구현체의 동작을 정의  
  - 제공자 등록 API : 제공자가 구현체를 등록할 때 사용  
  - 서비스 접근 API : 클라이언트가 서비스의 인스턴스를 얻을 때 사용  
  + 서비스 제공자 인터페이스 : 서비스 인터페이스의 인스턴스를 생성하는 팩토리 객체를 설명  
  
    JDBC 에서는,  
    Connection 이 서비스 인터페이스 역할  
    DriverManager.registerDriver 가 제공자 등록 API 역할  
    DriverManager.getConnection 이 서비스 접근 API 역할  
    Driver 가 서비스 제공자 인터페이스 역할  

    서비스 제공자 프레임워크 패턴에는 여러 변형이 존재한다.  
  
# 단점
1. 상속을 하려면 public 이나 protected 생성자가 필요하다   
  -> 정적 팩토리 메서드 만드로는 하위 클래스를 만들 수 없다.  

2. 프로그래머가 찾기 어렵다. 
  -> 네이밍 규칙을 잘 따라야함.  
  - from : 매개 변수를 하나 받아서 해당 타입 인스턴스를 반환  
    ```java
    Date d = Date.from(instant);
    ```
  - of : 여러 매개변수를 받아서 인스턴스를 반환
    ```java
    Set<Rank> faceCards = EnumSet.of(jack, queen, king);
    ```
  - valueOf : from 과 of의 더 자세한 버전
    ```java
    BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
    ```
  - instance 또는 getInstance : 같은 인스턴스임을 보장하지 않음. 
    ```java
    StackWalker = luke = StackWalker.getInstance(options);
    ```
  - create 또는 newInstance : 매번 새로운 인스턴스를 생성해 반환
    ```java
    Object newArray = Array.newInstance(classObject, arrayLen);
    ```
  - getType : getInstance와 같으나, 생성할 클래스가 아닌 클래스의 팩토리 메서드를 정의할 때 씀.  
              "Type"은 팩토리 메서드가 만환할 객체의 타입.
    ```java
    FileStore fs = Files.getFileStore(path);
    ```
  - newType : newInstance와 같으나, 생성할 클래스가 아닌 다른 클래스의 팩토리 메서드를 정의할 때 씀.
    ```java
    BufferedReader br = Files.newBufferedReader(path);
    ```
  - type : getType과 newType 의 간결한 버전
    ```java
    List<Complaint> litany = Collections.list(legacyLitany);
    ```
  
 
  
