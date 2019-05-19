---
title: "접근 제어자"
date: 2019-05-20
tags:
  - Access modifier
  - 접근 제어자
  - Public
  - Protected
  - default
  - private
---

접근제어자는 Java를 처음 배울때 기초적으로 학습하게 됩니다.

- **접근 제어자의 종류: public, protected, default, private**

||<center>public</center> | <center>protected</center> | <center>default</center> | <center>private</center> |
|--------|--------|--------|--------|--------|
|같은 클래스| <center>O</center> | <center>O</center> | <center>O</center> |<center>O</center> |
|같은 패키지| <center>O</center> | <center>O</center> | <center>O</center> |<center>X</center> |
|상속관계| <center>O</center> | <center>O</center> | <center>X</center> |<center>X</center> |
|전체| <center>O</center> | <center>X</center> | <center>X</center> |<center>X</center> |

접근 제어자는 굉장히 간단합니다.

하지만 개발자의 작성의도를 `문서`나 `주석`을 사용하지 않고 코딩만으로 알려 줄 수 있는 중요한 부분입니다.

저처럼 설계에 익숙치 않은 개발자분들은 public을 남용하고 문서나 주석을 따로 남기기 보다, 접근제어자를 더 고민해서 사용하면 생산성을 높일 수 있을거라 생각됩니다.