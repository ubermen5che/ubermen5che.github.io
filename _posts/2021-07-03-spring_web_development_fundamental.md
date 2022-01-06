---
layout : posts
comments : true
title: "[Web] 스프링 웹 개발 기초"
categories:
  - Java
  - Spring
tags:
  - Java
  - MVC
  - API
  - 템플릿 엔진
  - Template engine

---

# [Web] 스프링 웹 개발 기초

## 정적 컨텐츠

- 간단히 말해서 요청이 들어올 시 서버에서 웹 브라우저에 파일을 전송해주면 웹브라우저는 파일을 받아서 그대로 보여주는 방식
- 스프링에서 정적 컨텐츠는 project에 /static 경로를 찾아서 클라이언트에게 뿌려줌.
- 웹브라우저가 정적 페이지를 요청을 하면, Spring 내장 Tomcat서버를 거쳐 우선 스프링 컨테이너 안에 해당 요청에 관한 컨트롤러가 있는지 우선적으로 확인 후 관련 컨트롤러가 없다면 /static폴더내에 정적 컨텐츠가 존재하는지 확인한다. 있다면 그대로 클라이언트에게 제공해준다.

## MVC와 템플릿 엔진

- MVC : Model, View, Controller
- 예전과 달리 대부분의 웹 개발에서는 MVC패턴을 이용
- View에는 화면을 그려주는 기능에 집중하고, Controller에서는 비즈니스 로직 및 데이터 처리부분을 담당하고 Model에는 화면구성에 관련된 것들만 모아서 화면에 넘겨줌.
- PHP, JSP는 템플릿 엔진의 한 종류임
- 템플릿 엔진은 정적 컨텐츠와는 달리 서버에서 HTML을 동적으로 만들어주어 브라우저에 전송함.

## API

- JSON 포맷으로 클라이언트에 데이터를 전달해주는것
- 최근에 클라이언트는 JSON형식으로 받은 데이터를 사용자에게 보여줌
- API는 View를 따로 생성하지 않고 @ResponseBody를 이용해서 HTTP body에 객체 정보를 HttpMessageConverter가 적절한 형태(JSON, XML등)으로 변환하여 요청한 상대(browser, Android, server등)에게 데이터를 보내줌.
