---
title: "Adapter Pattern"
date: 2019-06-13
tags:
  - Design Pattern
  - 디자인 패턴
  - Adapter Pattern
  - 어댑터 패턴
  - Wrapper Pattern
  - 래퍼 패턴
---
> ### Convert the interface of a class into another interface that clients expect. The adapter pattern lets classes work together that couldn’t otherwise because of incompatible interfaces. - GoF
>
> ### 어느 한 클래스의 인터페이스를 사용자(client)가 원하는(expect) 인터페이스로 변환해준다. 어댑터 패턴은 호환되지 않는 인터페이스(incompatible interfaces)로 인해서 함께 사용할 수 없는 클래스들을 사용할 수 있게 해준다.

![Adapter UML Diagram](../../../assets/images/design_pattern/Adapter_UML_class_diagram.jpg){: width="100%" height="100%"}<center>Adapter UML Diagram</center>


장점
---
1. 기존에 존재하는 소스코드를 재사용 하면서 신규개발보다 안정적으로 빠르게 개발이 가능하다.
2. 버전업을 할 때 어댑터 패턴을 이용하여 신규 라이브러리를 adaptee로서 관리하면 구버전과 신버전의 유지보수에 용이하다.
3. 시스템에서 기존에 사용하던 기능을 서드파티 라이브러리로 교체할 경우에 adapter 부분만 수정하면 전체시스템의 수정 없이 교체가 가능하다.

래퍼 패턴(Wrapper Pattern) 이라고도 한다.

<!-- 예제 -->

<!-- 더보기 -->

