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

한국의 `둥근 돼지코` 모양의 콘센트(220V)와 일본의 `납작 돼지코`모양의 110V 콘센트(110V)의 어댑터를 코드로 나타낸 코드입니다.

예제
---
### KoreaPlugStandard.java
```java
package io.github.kimseunghyun_bg.adapter.plug;

public interface KoreaPlugStandard {
    String VOLT = "220v";
    String getVolt();
}
```
### KoreaPlug.java
```java
package io.github.kimseunghyun_bg.adapter.plug;

public class KoreaPlug implements KoreaPlugStandard {

    @Override
    public String getVolt() {
        return VOLT;
    }
}
```
### JapanPlugStandard.java
```java
package io.github.kimseunghyun_bg.adapter.plug;

public interface JapanPlugStandard {
    String VOLT = "110v";
    String getVolt();
}
```
### JapanPlug.java
```java
package io.github.kimseunghyun_bg.adapter.plug;

public class JapanPlug implements JapanPlugStandard{
    public String getVolt() {
        return VOLT;
    }
}
```
### Main.java
```java
package io.github.kimseunghyun_bg.adapter;

import io.github.kimseunghyun_bg.adapter.plug.JapanPlug;
import io.github.kimseunghyun_bg.adapter.plug.KoreaPlug;
import io.github.kimseunghyun_bg.adapter.socket.KoreaSocket;

public class Main {
    public static void main(String[] args) {
        KoreaSocket koreaSocket = new KoreaSocket();
        koreaSocket.plugin(new KoreaPlug());
    }
}

// 출력 결과
// Volt: 220v
```
### PlugAdapter.java
```java
package io.github.kimseunghyun_bg.adapter;

import io.github.kimseunghyun_bg.adapter.plug.JapanPlug;
import io.github.kimseunghyun_bg.adapter.plug.KoreaPlugStandard;

public class PlugAdapter implements KoreaPlugStandard {
    JapanPlug japanPlug;

    public PlugAdapter(JapanPlug japanPlug) {
        this.japanPlug = japanPlug;
    }

    @Override
    public String getVolt() {
        return japanPlug.getVolt();
    }
}
```
### Main.java
```java
package io.github.kimseunghyun_bg.adapter;

import io.github.kimseunghyun_bg.adapter.plug.JapanPlug;
import io.github.kimseunghyun_bg.adapter.plug.KoreaPlug;
import io.github.kimseunghyun_bg.adapter.socket.KoreaSocket;

public class Main {
    public static void main(String[] args) {
        KoreaSocket koreaSocket = new KoreaSocket();
        koreaSocket.plugin(new KoreaPlug());

        koreaSocket.plugin(new PlugAdapter(new JapanPlug()));
    }
}

// 출력 결과
// Volt: 220v
// Volt: 110v
```
우리가 일상에서 흔히 접할 수 있는 콘센트 어댑터를 통해 어댑터패턴을 구현해 보았습니다.
어댑터 패턴의 2가지 유형 (1.클래스 상속 & 인터페이스 구현, 2.인터페이스 구현 & 의존성 주입) 중에 2번째를 예시로 작성하였습니다.
<!-- 더보기 -->
