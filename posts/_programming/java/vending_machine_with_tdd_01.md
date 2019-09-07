---
title: "자판기 프로젝트 01 (with.TDD)"
date: 2019-09-05
tags:
  - TDD
  - Vending Machine
---

1. [자판기 프로젝트 01 (with.TDD)](https://kimseunghyun-bg.github.io/programming/java/vending_machine_with_tdd_01/)
2. [자판기 프로젝트 02 (with.TDD)](https://kimseunghyun-bg.github.io/programming/java/vending_machine_with_tdd_02/)

## 발단
문득 자판기를 만들고 싶었다.

## 기술 목표
- TDD (Junit4)

프로젝트가 진행되면서 기술은 하나씩 추가.

~~(용두사미가 특기라서 미리 다 안써논거다. 썼다가 못하면 창피)~~

## 시작합니다.

TDD책에서 켄트백 형이 여러 이야기를 했지만, TDD방법은 이렇게 하라고 했다.
1. <span style="color:CORNFLOWERBLUE">할 일 작성</span>
2. <span style="color:CORNFLOWERBLUE">테스트 작성</span>
3. <span style="color:SALMON">빨간 막대기</span> 보기 or 컴파일 오류
4. <span style="color:SEAGREEN">초록 막대기</span> 보기 (Stub을 써서라도)
5. <span style="color:CORNFLOWERBLUE">리펙토링</span>
6. __1번 ~ 5번 반복__

***

- <span style="color:CORNFLOWERBLUE">할 일 작성</span>

| 할 일 |
| :------------ |
| 자판기에서 음료수 뽑기 |
| 자판기에 음료수 넣기 |
| 자판기에 돈 넣기 |
| 자판기에서 거스름돈 나오기 |

이 중에 `자판기에 돈 넣기`를 해보자. 난 좀 서투르니까 작은 스탭으로 쪼개서 진행하자.

| 할 일 |
| :------------ |
| 자판기에 돈 넣기 |
| __자판기에 돈 100원 넣기__ |
| 자판기에 돈 100원 2번 넣기 |

- <span style="color:CORNFLOWERBLUE">테스트 작성</span>

Given/When/Then 양식을 따라하면 참 좋다고 한다. 해보니 좋은거 같다. 따라하자.
```java
public class MachineTest {

    @Test
    public void insertMoney_Insert100_MachineShouldHave100() {
        // Given
        // When
        // Then
        assertThat(actual, is(100));
    }
}
```
테스트를 작성 할때 assert를 먼저 작성하고, 역으로 Given으로 올라가라고 켄트백 형이 말했다. 묻지도 따지지도 않고 시키는대로 하자.

~~따지긴 해야지!~~
이 테스트로 얻고자 하는 바를 처음부터 분명히 하고, 그 목표에 맞춰서 테스트를 작성 할 수 있어서 그런거다.

테스트를 마저 작성한다.
```java
public class MachineTest {

    @Test
    public void insertMoney_Insert100_MachineShouldHave100() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertMoney(100);
        int actual = machine.amount;
        // Then
        assertThat(actual, is(100));
    }
}
```

- <span style="color:SALMON">빨간 막대기</span>

짜잔!!! 컴파일 오류가 발생했습니다.

코드 구현
```java
public class Machine {

    public int amount = 100;

    public void insertMoney(int amount) {

    }
}
```
- <span style="color:SEAGREEN">초록 막대기</span>

일단 stub을 써서 초록 막대기를 보자

```java
public class Machine {

    public int amount;

    public void insertMoney(int amount) {
        this.amount = amount;
    }
}
```
- <span style="color:CORNFLOWERBLUE">리펙토링</span>

<span style="color:CORNFLOWERBLUE">리펙토링</span>하고 다시 <span style="color:SEAGREEN">초록막대기</span>가 되는지 확인을 하면 된다.


#### 짜잔! 이 프로세스를 무한반복하면서 프로젝트를 진행하면 됩니다.
***

| 할 일 |
| :------------ |
| 자판기에 돈 넣기 |
| ~~자판기에 돈 100원 넣기~~ |
| __자판기에 돈 100원 2번 넣기__ |


```java
public class MachineTest {

    @Test
    public void insertMoney_Insert100TwoTimes_MachineShouldHave200() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertMoney(100);
        machine.insertMoney(100);
        int actual = machine.amount;
        // Then
        assertThat(actual, is(200));
    }
}
```

```java
public class Machine {

    public int amount;

    public void insertMoney(int amount) {
        this.amount += amount;
    }
}
```

***

| 할 일 |
| :------------ |
| 자판기에 돈 넣기 |
| ~~자판기에 돈 100원 넣기~~ |
| ~~자판기에 돈 100원 2번 넣기~~ |
| __자판기에 돈 2원을 넣어서 실패하기__ |

자판기에 100원이 아니라 2원을 넣을 수 없다.

2원이라는 권종은 없으니 amount가 늘어나면 안된다.

복붙 해서 조금만 고치는게 빨라요~
```java
public class MachineTest {

    @Test
    public void insertMoney_InsertTwo_MachineShouldHaveZero() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertMoney(2);
        int actual = machine.amount;
        // Then
        assertThat(actual, not(2));
        assertThat(actual, is(0));
    }
}
```

잠시 생각해보면(1초)

현실을 생각해보면 화폐 권종을 넣는 것이지, 숫자를 넣지 않는다. (* 할 일이 추가 되었습니다.)

| 할 일 |
| :------------ |
| *insert의 파라미터를 int에서 Coin으로 바꾸기* |

```java
public class Machine {
  
    public void insertMoney(int amount) {
        if (amount == 100)
            this.amount += amount;
    }
}
```
그런데 사실 이것도 최악의 코드는 아닌거 같다. (그러니까 코드짠 사람을 천국으로 당장 보내고 싶다던가 하는 기분이 드는 코드)

~~우리나라 화폐권종은 10원부터 50000원까지 8개밖에 안되니까 조건문을 8개만 걸면 된다.~~

그래도 int말고 화폐로 바꿔보자. 리디노미네이션이나 외국에 자판기를 수출하면서 큰 혼란을 피하고 싶기 때문이다.

(단, 이 생각은 언제든지 바뀔 수 있다. '실용주의 프로그래머'를 보면 '적당히 괜찮은 소프트웨어'라는 부분이 있다. 모든걸 고려한 완벽한 프로그램은 어려우니 고객의 요구조건에 맞는 적당히 현실적인 프로그램 산출물로 타협을 보라는 내용이다.)

***

| 할 일 |
| :------------ |
| 자판기에 돈 넣기 |
| ~~자판기에 돈 100원 넣기~~ |
| ~~자판기에 돈 100원 2번 넣기~~ |
| ~~자판기에 돈 2원을 넣어서 실패하기~~ |
| insert의 파라미터를 int에서 Coin으로 바꾸기 |
| __자판기에 돈 100원 Coin객체로 넣기__ |

insertMoney의 파라미터를 int에서 Coin으로 바꾸자.
InsertCoin 메소드 추가. (*진행을 빨리 나가겠다.)

```java
public class MachineTest {

    @Test
    public void insertCoin_InsertKRW100Coin_Has100() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertCoin(new KRW100Coin());
        int actual = machine.amount;
        // Then
        assertThat(actual, is(100));
    }
}
```
```java
public class KRW100Coin {
    public static final int AMOUNT = 100;
}
```
```java
public class Machine
    public void insertCoin(KRW100Coin coin) {
        this.amount += coin.AMOUNT;
    }
```

***

| 할 일 |
| :------------ |
| 자판기에 돈 넣기 |
| ~~자판기에 돈 100원 넣기~~ |
| ~~자판기에 돈 100원 2번 넣기~~ |
| ~~자판기에 돈 2원을 넣어서 실패하기~~ |
| insert의 파라미터를 int에서 Coin으로 바꾸기 |
| ~~자판기에 돈 100원 Coin객체로 넣기~~ |
| __자판기에 돈 500원 Coin객체로 넣기__ |


```java
public class MachineTest {

    @Test
    public void insertCoin_InsertKRW500Coin_Has500() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertCoin(new KRW500Coin());
        int actual = machine.amount;
        // Then
        assertThat(actual, is(500));
    }
}
```

```java
public class KRW500Coin {
    public static final int AMOUNT = 500;
}
```

Machine.insertCoin에서 컴파일 오류 -> 추상화 필요 적어두자

| 할 일 |
| :------------ |
| *Coin객체 추상화* |

오버로딩으로 초록막대기 생성
```java
public class Machine {
    public void insertCoin(KRW500Coin coin) {
        this.amount += coin.AMOUNT;
    }
}
```

***

| 할 일 |
| :------------ |
| 자판기에 돈 넣기 |
| ~~자판기에 돈 100원 넣기~~ |
| ~~자판기에 돈 100원 2번 넣기~~ |
| ~~자판기에 돈 2원을 넣어서 실패하기~~ |
| insert의 파라미터를 int에서 Coin으로 바꾸기 |
| ~~자판기에 돈 100원 Coin객체로 넣기~~ |
| ~~자판기에 돈 500원 Coin객체로 넣기~~ |
| __Coin객체 추상화__ |

추상화 클래스를 만들어서 올리자.
```java
// 1단계 - 상속 (1/2 테스트 통과)
public class KRWCoin {
    public int amount = 100;    // AMOUNT를 amount로 바꿔준다
}
public class KRW100Coin extends KRWCoin{
    //    public static final int AMOUNT = 100;
}
public class KRW500Coin extends KRWCoin{
    //    public static final int AMOUNT = 500;
}
```
```java
// 2 단계 - 상속받은 객체 생성자 구현 (2/2 테스트 통과)
public class KRWCoin {
    public int amount;
}
public class KRW100Coin extends KRWCoin{
    public KRW100Coin() {
        amount = 100;
    }
}
public class KRW500Coin extends KRWCoin{
    public KRW500Coin() {
        amount = 500;
    }
}
```
```java
// 오버로딩 메소드 통합
public class Machine {
    public void insertCoin(KRWCoin coin){
            this.amount += coin.amount;
        }
}
```

KRW100Coin 클래스와 KRW500Coin클래스는 KRWCoin클래스에 넣을 수 있을 것 같다.

```java
public class KRWCoin {
    public int amount;

    private KRWCoin(int amount) {
        this.amount = amount;
    }

    public static KRWCoin oneHundredCoin() {
        return new KRWCoin(100);
    }
}
```

테스트도 수정해준다.
```java
public class MachineTest {

    @Test
    public void insertCoin_InsertKRW100Coin_Has100() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertCoin(KRWCoin.oneHundredCoin());
        int actual = machine.amount;
        // Then
        assertThat(actual, is(100));
    }
}
```

KRW500Coin도 동일한 방법으로 제거해준다.

이제 Coin이 추가된다고 Class가 추가될 일은 없다. 

응집도가 아주 조금 높아졌다 (만족)

Coin클래스의 amount의 접근 제어자를 수정해주자

```java
public class KRWCoin {
    private int amount;

    public int getAmount() {
        return amount;
    }
}
```

```java
public class Machine {
    public void insertCoin(KRWCoin coin) {
        this.amount += coin.getAmount();
    }
}
```
나는 리펙토링에 안정감을 주기때문에 TDD가 너무 좋다. 흐흐흐흐

***

| 할 일 |
| :------------ |
| 자판기에 돈 넣기 |
| ~~자판기에 돈 100원 넣기~~ |
| ~~자판기에 돈 100원 2번 넣기~~ |
| ~~자판기에 돈 2원을 넣어서 실패하기~~ |
| insert의 파라미터를 int에서 Coin으로 바꾸기 |
| ~~자판기에 돈 100원 Coin객체로 넣기~~ |
| ~~자판기에 돈 500원 Coin객체로 넣기~~ |
| ~~Coin객체 추상화~~ |

Machine 클래스에 insertMoney(int amount), insertCoin(KRWCoin coin) 두가지가 생겼다. insertCoin이 insertMoney를 대체해야한다.

```java
public class Machine {
    public void insertMoney(int amount) {
        if (amount == 100) {
            insertCoin(KRWCoin.oneHundredCoin());
        } else if (amount == 500) {
            insertCoin(KRWCoin.fiveHundredCoin());
        }
    }
}
```

Machine클래스의 insertMoney 메소드를 insertCoin을 단순 호출하는 기능으로 수정했다. 이렇게 해서 기존의 테스트가 통과한다면 insertCoin메소드가 insertMoney에 구현된 기능을 완벽히 대체한다고 볼 수 있겠다. (100원 + 100원 = 200원 확인하려고 저렇게 수정하였다.)

초록막대기! 성공이다!

insertMoney를 삭제하기전에 100원을 두번 넣는 테스트로 수정해준다.
```java
public class MachineTest {
    @Test
    public void insertCoin_InsertKRW100CoinTwoTimes_Has200() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertCoin(KRWCoin.oneHundredCoin());
        machine.insertCoin(KRWCoin.oneHundredCoin());
        int actual = machine.amount;
        // Then
        assertThat(actual, is(200));
    }
}
```
다시 테스트를 해보고, 성공했으니 insertMoney 메소드와 테스트를 삭제해준다.

할 일 목록은 본인 취향대로 관리하면 될 꺼 같다.

| 할 일 |
| :------------ |
| 자판기에 돈 넣기 |
| ~~insert의 파라미터를 int에서 Coin으로 바꾸기~~ |
| ~~자판기에 돈 100원 Coin객체 2번 넣기~~ |
| ~~자판기에 돈 100원 Coin객체로 넣기~~ |
| ~~자판기에 돈 500원 Coin객체로 넣기~~ |
| ~~Coin객체 추상화~~ |

***

| 할 일 |
| :------------ |
| 자판기에 돈 1000원 Bill객체 2번 넣기 |
| 자판기에 돈 1000원 Bill객체로 넣기 |
| 자판기에 돈 5000원 Bill객체로 넣기 |

지폐를 넣는 과정도 비슷하겠다.

동전 테스트를 복사해서 하나씩 바꿔서 똑같이 진행하면 된다.

```java
public class MachineTest {

    @Test
    public void insertBill_InsertKRW1000Bill_Has1000() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertBill(KRWBill.oneThousandBill());
        int actual = machine.amount;
        // Then
        assertThat(actual, is(1000));
    }

    @Test
    public void insertBill_InsertKRW5000Bill_Has5000() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertBill(KRWBill.fiveThousandBill());
        int actual = machine.amount;
        // Then
        assertThat(actual, is(5000));
    }

    @Test
    public void insertBill_InsertKRW1000BillTwoTimes_Has2000() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertBill(KRWBill.oneThousandBill());
        machine.insertBill(KRWBill.oneThousandBill());
        int actual = machine.amount;
        // Then
        assertThat(actual, is(2000));
    }
}
```
```java
public class Machine {
    public void insertBill(KRWBill bill) {
        this.amount += bill.getAmount();
    }
}
```
```java
public class KRWBill {
    private int amount;

    private KRWBill(int amount) {
        this.amount = amount;
    }

    public static KRWBill oneThousandBill() {
        return new KRWBill(1000);
    }

    public static KRWBill fiveThousandBill() {
        return new KRWBill(5000);
    }

    public int getAmount() {
        return amount;
    }
}
```

Coin과 Bill 특성만 따로 빼서 확장성을 부여하고싶은 마음이 일고 있다.
지금은 참고 넘어가고, 나중에 국제적인 자판기를 만들면서 추상화 하도록하자.

| 할 일 |
| :------------ |
| 자판기에서 음료수 뽑기 |
| 자판기에 음료수 넣기 |
| ~~자판기에 돈 넣기~~ |
| 자판기에서 거스름돈 나오기 |

`자판기에 돈 넣기`에 요구했던 기능은 끝난거 같다.

카드결제 기능도 추가하고싶다.
각종 화폐 지원 자판기로 확장도 하고싶다.

| 할 일 |
| :------------ |
| 자판기에서 음료수 뽑기 |
| 자판기에 음료수 넣기 |
| ~~자판기에 돈 넣기~~ |
| 자판기에서 거스름돈 나오기 |
| 체크카드로 음료수 뽑기 |
| 신용카드로 음료수 뽑기 |
| 메타데이터로 화폐정보 변경 해서 국제적인 자판기로 만들기 |

되는데 까지만 해보자.

***

정리
---
코드를 보면 엉망이다. ~~잘 알고 있다. 일부러 엉망인 코드로 시작했다~~

리펙토링을 하며 TDD가 가지는 장점 중 하나를 보여주고 싶어서 이렇게 시작했다.

켄트백형이 TDD에 쓴 예제가 눈으로 쓱쓱 넘어가기엔 어려운 감이 있다. (Expression을 사용한 메타포의 강력한 효능을 보여주려는건 알겠다.) 지금은 TDD에 대해서만 이야기 하고 싶고, 눈으로 쓱쓱 쉽게 읽히는 코드가 더 좋을 것 같다는 생각이다.

***
### 환경
1. Language: Java 1.8
2. Framework: Spring-Boot-2.1.7, Gradle-5.6
3. Library: Junit-4.12