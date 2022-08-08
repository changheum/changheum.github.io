---
title:  "Spring Boot 구조"
excerpt: ""

# categories:
#   - android
tags:
  - [springboot]

toc: true
toc_sticky: true
 
date: 2022-08-02
last_modified_at: 2022-08-02

layout: post
summary: springboot
author: changheum
category: [springboot]
# thumbnail: /assets/img/documents.png
keywords: [springboot]
usemathjax: true
permalink: /blog/springboot_architecture
---


  
# Spring과의 차이점  
1. Embed Tomcat을 사용한다.  
2. Starter로 dependency가 자동화 되어있다.   
3. XML 설정을 하지 않아도 된다.  
4. jar file을 이용하여 자바 옵션만으로 배포가 가능하다.  
   - Spring Actuator를 이용한 어플리케이션 모니터링과 관리를 제공한다.  
  
# Spring Boot Starter  
`spring-boot-starter-*`
메이븐 pom.xml 이나, 그래들 build.gradle 에서 라이브러리를 받아올 수 있다.  

스프링부트는 미리 설정된 스타터 프로젝트를 제공한다.  
스프링부트는 자동설정(AutoConfiguration) 을 이용하고, 모든 내부 Dependency를 관리함  
- 스프링의 jar 파일이 클래스 패스에 있을 경우, Dispatcher Servlet 으로 자동 구성함.   
- Hibernate의 jar가  클래스 패스에 있을 경우, datasource로 자동 설정함.  
  
