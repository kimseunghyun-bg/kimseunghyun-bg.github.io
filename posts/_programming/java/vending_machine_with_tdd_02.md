---
title: "자판기 프로젝트 02 (with.TDD)"
date: 2019-09-07
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

`자판기 프로젝트 01`에서는 TDD로 진행하면서 `자판기에 돈 넣기` 기능을 구현했다.

| 할 일 |
| :------------ |
| **자판기에서 음료수 뽑기** |
| **자판기에 음료수 넣기** |
| ~~자판기에 돈 넣기~~ |
| **자판기에서 거스름돈 나오기** |
| *체크카드로 음료수 뽑기(* 추후 과제)* |
| *신용카드로 음료수 뽑기(* 추후 과제)* |
| *메타데이터로 화폐정보 변경 해서 국제적인 자판기로 만들기(* 추후 과제)* |

이번편에서는 나머지 핵심이 되는 3가지 기능을 구현하겠다.
(쓱쓱 보시고 넘어가셔도 됩니다.)

***

`자판기에 음료수 넣기` 시작

| 할 일 |
| :------------ |
| 자판기에서 음료수 뽑기 |
| **자판기에 음료수 넣기** |
| ~~자판기에 돈 넣기~~ |
| 자판기에서 거스름돈 나오기 |

나는 능력이 안되니까. 할 일을 조금 더 작은 스텝으로 나누어서 진행하자.

| 할 일 |
| :------------ |
| 자판기에 음료수 넣기 |
| __자판기에 Coke 넣기__ |
| 자판기에 Sprite 넣기 |

동전 넣기와 유사하게 진행하자. Can 객체만 잘 만들면 될 것 같다.

( 리펙토링은 내일의 `김승현`이 해줄 것이다.)

```Java
public class MachineTest {

    @Test
    public void insertCan_InsertCoke_HasOneCoke() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertCan(Can.Coke());
        int actual = machine.stock.get(Can.Coke().getItemCode());
        // Then
        assertThat(actual, is(1));
    }
}
```
```Java
public class Can {
    private String itemCode;
    private String name;

    private Can(String itemCode, String name) {
        this.itemCode = itemCode;
        this.name = name;
    }

    public static Can Coke() {
        return new Can("001","Coke");
    }

    public String getItemCode() {
        return itemCode;
    }
}
```
```Java
public class Machine {
    private int cash;   // amount를 cash로 변경했다. 
                        //  접근제어자를 private으로 변경하고 getter추가, 테스트에도 getter로 변경.
    public Map<String, Integer> stock = new HashMap<>();
    public void insertCan(Can can) {
        stock.put(can.getItemCode(),1);
    }
}
```

***

| 할 일 |
| :------------ |
| 자판기에 음료수 넣기 |
| ~~자판기에 Coke 넣기~~ |
| __자판기에 Coke 2개 넣기__ |
| 자판기에 Sprite 넣기 |

```Java
public class MachineTest {
    @Test
    public void insertCan_InsertCokeTwoTimes_HasTwoCoke() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertCan(Can.Coke());
        machine.insertCan(Can.Coke());
        int actual = machine.stock.get(Can.Coke().getItemCode());
        // Then
        assertThat(actual, is(2));
    }
}
```
```Java
public class Machine {
    public void insertCan(Can can) {
        Integer quantity = Optional.ofNullable(stock.get(can.getItemCode())).orElse(0);
        stock.put(can.getItemCode(),quantity+1);
    }
}
```

***

| 할 일 |
| :------------ |
| 자판기에 음료수 넣기 |
| ~~자판기에 Coke 넣기~~ |
| ~~자판기에 Coke 2개 넣기~~ |
| __자판기에 Sprite 넣기__ |

```Java
public class MachineTest {
    @Test
    public void insertCan_InsertSprite_HasOneSprite() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertCan(Can.Sprite());
        int actual = machine.stock.get(Can.Sprite().getItemCode());
        // Then
        assertThat(actual, is(1));
    }
}
```

***

| 할 일 |
| :------------ |
| 자판기에 음료수 넣기 |
| ~~자판기에 Coke 넣기~~ |
| ~~자판기에 Coke 2개 넣기~~ |
| ~~자판기에 Sprite 넣기~~ |
| __자판기 재고 확인 메소드 추가하기__ |
| 자판기에 Stock 접근권한 변경 |

자판기에 Stock 권한을 변경하고싶다.

그전에 재고를 확인 할 수 있는 메소드가 필요해보인다.
먼저 추가하자

*(테스트 코드는 필요하면 복붙하는게 좋습니다. 단, 나중에 리펙토링 필수)*
```Java
public class MachineTest {
    @Test
    public void insertCan_InsertCoke_HasOneCoke() {
        // Given
        Machine machine = new Machine();
        // When
        machine.insertCan(Can.Coke());
        int actual = machine.getQuantityOfCan(Can.Coke());
        // Then
        assertThat(actual, is(1));
    }
}
```
```Java
public class Machine {
    public int getQuantityOfCan(Can can) {
        return Optional.ofNullable(stock.get(can.getItemCode())).orElse(0);
    }
}
```

***

| 할 일 |
| :------------ |
| 자판기에 음료수 넣기 |
| ~~자판기에 Coke 넣기~~ |
| ~~자판기에 Coke 2개 넣기~~ |
| ~~자판기에 Sprite 넣기~~ |
| ~~자판기 재고 확인 메소드 추가하기~~ |
| __자판기에 Stock 접근권한 변경__ |

접근권한을 수정하면 컴파일 에러가 막 뜬다. 컴파일 오류가 난 부분을 잡아주자.

Machine의 getQuantityOfCan와 insertCan에 중복된 코드가 있다. 

**리펙토링!**

```Java
public class Machine {

    private int cash;
    private Map<String, Integer> stock = new HashMap<>();

    public void insertCan(Can can) {
        int quantity = getQuantityOfCan(can);
        stock.put(can.getItemCode(),quantity+1);
    }

    public int getQuantityOfCan(Can can) {
        return Optional.ofNullable(stock.get(can.getItemCode())).orElse(0);
    }
}
```

여기서 잠깐.
테스트 케이스를 작성하다보면 특정 객체나 메소드에 의존적인 상황이 벌어지기 마련이다.

아직 많은 경험이 없어서 잘 모르지만 `getter`, `setter`정도는 의존해도 큰 무리는 없어 보인다.

| 할 일 |
| :------------ |
| 자판기에 음료수 넣기 |
| ~~자판기에 Coke 넣기~~ |
| ~~자판기에 Coke 2개 넣기~~ |
| ~~자판기에 Sprite 넣기~~ |
| ~~자판기 재고 확인 메소드 추가하기~~ |
| ~~자판기에 Stock 접근권한 변경~~ |


| 할 일 |
| :------------ |
| 자판기에서 음료수 뽑기 |
| ~~자판기에 음료수 넣기~~ |
| ~~자판기에 돈 넣기~~ |
| 자판기에서 거스름돈 나오기 |

`자판기에 음료수 넣기`에 요구했던 기능도 대략 끝난거 같다.

***

| 할 일 |
| :------------ |
| __자판기에서 음료수 뽑기__ |
| ~~자판기에 음료수 넣기~~ |
| ~~자판기에 돈 넣기~~ |
| 자판기에서 거스름돈 나오기 |

`자판기에서 음료수 뽑기`를 할 수 있을 것 같다.
돈하고 음료수를 자판기에 넣었으니 이제 골라서 사기만 하면 된다.

```Java
public class MachineTest {
    @Test
    public void selectCan_SelectCoke_reduceStockAndCash() {
        // Given
        Machine machine = new Machine();
        machine.insertCoin(KRWCoin.fiveHundredCoin());
        machine.insertCan(Can.Coke());
        machine.insertCan(Can.Sprite());
        // When
        machine.selectCan(Can.Coke());
        int actualQuantityOfCoke = machine.getQuantityOfCan(Can.Coke());
        int actualQuantityOfSprite = machine.getQuantityOfCan(Can.Sprite());
        int actualCash = machine.getCash();
        // Then
        assertThat(actualQuantityOfCoke, is(0));
        assertThat(actualQuantityOfSprite, is(1));
        assertThat(actualCash, is(0));
    }
}
```

테스트를 하면서 생각이 났다.

**나는 sellPrice를 입력하지 않았다.** 진행하며 추가해보자.

```Java
public class Machine {
    public void selectCan(Can can) {
        int quantity = getQuantityOfCan(can);
        stock.put(can.getItemCode(),quantity-1);
        cash = can.getSellPrice();  // can에 sellPrice를 추가해야 한다.
    }
}
```
**sellPrice를 추가해보자**
```Java
public class Can {
    private String itemCode;
    private String name;
    private int sellPrice;

    private Can(String itemCode, String name, int sellPrice) {
        this.itemCode = itemCode;
        this.name = name;
        this.sellPrice = sellPrice;
    }

    public static Can Coke() {
        return new Can("001","Coke", 500);
    }

    public static Can Sprite() {
        return new Can("002","Sprite", 600);
    }

    public int getSellPrice() {
        return sellPrice;
    }
}
```

휴... 공짜로 음료수를 줄뻔했다.

***

`자판기에서 음료수 뽑기 완성!` 은 아니다.

| 할 일 |
| :------------ |
| ~~자판기에서 음료수 뽑기~~ |
| __500원으로 600원짜리 음료수 뽑으면 lackOfCash 에러 던지기__ |
| 재고가 없으면 SoldOut 에러 던지기 |
| 거스름돈 돌려주기 |

- 이 자판기는 아직도 공짜로 음료수를 줄 수 있다.
- 그리고 음료 재고가 없어도 계속 뽑을 수 있다.
- 거스름돈문제도 있다. (거스름돈은 refund 기능을 추가하면서 같이 처리하는게 좋을 것 같은 예감이 든다.)

하나씩 하나씩 스탭을 밟아가면서 할 수 있는게 TDD의 매력같다.

```Java
public class MachineTest {
    @Rule
    public ExpectedException lackOfCashExceptionRule = ExpectedException.none();

    @Test
    public void selectCan_Insert500KRWSelectSprite_LackOfCashExceptionThrown() throws LackOfCashException {
        // Given
        Machine machine = new Machine();
        machine.insertCoin(KRWCoin.fiveHundredCoin());
        machine.insertCan(Can.Coke());
        machine.insertCan(Can.Sprite());
        // When
        lackOfCashExceptionRule.expect(LackOfCashException.class);
        machine.selectCan(Can.Sprite());
       // Then
    }
}
```
```Java
public class LackOfCashException extends Exception {
}
```
```Java
public class Machine {
    public void selectCan(Can can) throws LackOfCashException{
        if (cash - can.getSellPrice() < 0) {
            throw new LackOfCashException();
        }
        
        int quantity = getQuantityOfCan(can);
        stock.put(can.getItemCode(),quantity-1);
        cash -= can.getSellPrice();
    }
}
```

***

| 할 일 |
| :------------ |
| ~~자판기에서 음료수 뽑기~~ |
| ~~500원으로 600원짜리 음료수 뽑으면 lackOfCash 에러 던지기~~ |
| __재고가 없으면 SoldOut 에러 던지기__ |
| 거스름돈 돌려주기 |

```Java
public class MachineTest {
    @Rule
    public ExpectedException soldOutCanExceptionRule = ExpectedException.none();

    @Test
    public void selectCan_SelectSoldOutCoke_SoldOutExceptionThrown() throws LackOfCashException, SoldOutCanException {
        // Given
        Machine machine = new Machine();
        machine.insertCoin(KRWCoin.fiveHundredCoin());
        // When
        soldOutCanExceptionRule.expect(SoldOutCanException.class);
        machine.selectCan(Can.Coke());
        // Then
    }
}
```
```Java
public class SoldOutCanException extends Exception {
}
```
```Java
public class Machine {
    public void selectCan(Can can) throws LackOfCashException, SoldOutCanException{
        if (cash - can.getSellPrice() < 0) {
            throw new LackOfCashException();
        }
        int quantity = getQuantityOfCan(can);
        if (quantity < 1) {
            throw new SoldOutCanException();
        }

        stock.put(can.getItemCode(),quantity-1);
        cash -= can.getSellPrice();
    }
}
```

| 할 일 |
| :------------ |
| ~~자판기에서 음료수 뽑기~~ |
| ~~500원으로 600원짜리 음료수 뽑으면 lackOfCash 에러 던지기~~ |
| ~~재고가 없으면 SoldOut 에러 던지기~~ |

`자판기에서 음료수 뽑기`를 마쳤다.

***

| 할 일 |
| :------------ |
| ~~자판기에서 음료수 뽑기~~ |
| ~~자판기에 음료수 넣기~~ |
| ~~자판기에 돈 넣기~~ |
| 자판기에서 거스름돈 나오기 |

`자판기에서 거스름돈 나오기`를 할 차례다.

| 할 일 |
| :------------ |
| 자판기에서 거스름돈 나오기 |
| __천원 넣고 600원짜리 음료 뽑으면 거스름돈으로 100원 객체 4개 담긴 Map얻기__ |
| 천원 넣고 600원짜리 음료 뽑으면 거스름돈으로 500원 객체 4개 담긴 Map이랑 비교해서 실패하기 |


```Java
public class MachineTest {
    @Test
    public void cashReturn_Insert1000KRWAndSelect600KRWCan_ShouldReturnFour100KRWCoins() throws LackOfCashException, SoldOutCanException {
        // Given
        Map<KRWCoin, Integer> expected = new HashMap<>();
        expected.put(KRWCoin.oneHundred(), 4);
        
        Machine machine = new Machine();
        machine.insertBill(KRWBill.oneThousandBill());
        machine.insertCan(Can.Sprite());
        machine.selectCan(Can.Sprite());
        // When
        Map<KRWCoin, Integer> actual = machine.cashReturn();
        // Then
        assertThat(actual.entrySet(), equalTo(expected.entrySet()));
    }
}
```
```Java
public class Machine {
    public Map<KRWCoin, Integer> cashReturn() {
        // 바로 구현할 자신이 없어서 stub으로 만들었다.
        Map<KRWCoin, Integer> stub = new HashMap<>();
        stub.put(KRWCoin.oneHundredCoin(), 4);
        return stub;
    }
}
```
Equals를 재정의 해줄 필요가 있어 보인다.

Intellij의 도움을 받아서 Equals와 hashCode를 만들었다.
편하지만 어떤 의미인지 한번씩 코드리뷰는 하고 넘어가자. 

본인의 의도와 다르게 비교를 할 수 있다.
```Java
public class KRWCoin {
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof KRWCoin)) return false;
        KRWCoin krwCoin = (KRWCoin) o;
        return getAmount() == krwCoin.getAmount();
    }

    @Override
    public int hashCode() {
        return Objects.hash(getAmount());
    }
}
```

***

| 할 일 |
| :------------ |
| 자판기에서 거스름돈 나오기 |
| ~~천원 넣고 600원짜리 음료 뽑으면 거스름돈으로 100원 객체 4개 담긴 Map얻기~~ |
| __천원 넣고 500원짜리 음료 뽑으면 거스름돈으로 500원 객체 1개 담긴 Map얻기__ |

```Java
public class MachineTest {
    @Test
    public void cashReturn_Insert1000KRWAndSelect500KRWCan_ShouldReturnOne500KRWCoins() throws LackOfCashException, SoldOutCanException {
        // Given
        Map<KRWCoin, Integer> expected = new HashMap<>();
        expected.put(KRWCoin.fiveHundredCoin(), 1);

        Machine machine = new Machine();
        machine.insertBill(KRWBill.oneThousandBill());
        machine.insertCan(Can.Coke());
        machine.selectCan(Can.Coke());
        // When
        Map<KRWCoin, Integer> actual = machine.cashReturn();
        // Then
        assertThat(actual.entrySet(), equalTo(expected.entrySet()));
    }
}
```

Stub으로 만들어둔 Machine의 cashReturn 메소드를 제대로 구현하자.

```Java
public class Machine {
    public Map<KRWCoin, Integer> cashReturn() {
        Map<KRWCoin, Integer> result = new HashMap<>();
        while (cash != 0) {
            if (cash / KRWCoin.fiveHundred().getAmount() > 0) {
                result.put(KRWCoin.fiveHundred(), cash / KRWCoin.fiveHundred().getAmount());
                cash %= KRWCoin.fiveHundred().getAmount();
            } else if (cash / KRWCoin.oneHundred().getAmount() > 0) {
                result.put(KRWCoin.oneHundred(), cash / KRWCoin.oneHundred().getAmount());
                cash %= KRWCoin.oneHundred().getAmount();
            }
        }
        return result;
    }
}
```

성능에 문제가 있어 보이지만, 리펙토링은 나중에 해도 좋다.

***

| 할 일 |
| :------------ |
| 자판기에서 거스름돈 나오기 |
| ~~천원 넣고 600원짜리 음료 뽑으면 거스름돈으로 100원 객체 4개 담긴 Map얻기~~ |
| ~~천원 넣고 500원짜리 음료 뽑으면 거스름돈으로 500원 객체 1개 담긴 Map얻기~~ |
| __1500원 넣고 600원짜리 음료 뽑으면 거스름돈으로 500원 객체 1개, 100원 객체가 4개 담긴 Map얻기__ |

테스트 두개를 합치는게 나을 것 같다는 생각이 들었다.

```Java
public class MachineTest {
    @Test
    public void cashReturn_Insert1500KRWAndSelect600KRWCan_ShouldReturnOne500KRWCoinAndFour100KRWCoins() throws LackOfCashException, SoldOutCanException {
        // Given
        Machine machine = new Machine();
        machine.insertBill(KRWBill.oneThousandBill());
        machine.insertCoin(KRWCoin.fiveHundredCoin());
        machine.insertCan(Can.Sprite());
        machine.selectCan(Can.Sprite());
        // When
        Map<KRWCoin, Integer> actual = machine.cashReturn();
        // Then
        assertThat(actual, IsMapContaining.hasEntry(KRWCoin.fiveHundredCoin(), 1));
        assertThat(actual, IsMapContaining.hasEntry(KRWCoin.oneHundredCoin(), 4));
    }
}
```
먼저 만들었던 두개의 테스트는 중복되기 때문에 삭제해준다.

| 할 일 |
| :------------ |
| 자판기에서 거스름돈 나오기 |
| ~~1500원 넣고 600원짜리 음료 뽑으면 거스름돈으로 500원 객체 1개, 100원 객체가 4개 담긴 Map얻기~~ |

***

| 할 일 |
| :------------ |
| ~~자판기에서 음료수 뽑기~~ |
| ~~자판기에 음료수 넣기~~ |
| ~~자판기에 돈 넣기~~ |
| ~~자판기에서 거스름돈 나오기~~ |
| 체크카드로 음료수 뽑기(* 추후 과제) |
| 신용카드로 음료수 뽑기(* 추후 과제) |
| 메타데이터로 화폐정보 변경 해서 국제적인 자판기로 만들기(* 추후 과제) |

기본 기능은 대략적으로 다 구현한 것 같다.

***

정리
---
설계를 처음부터 하고 시작한 코드가 아니어서 많은 허점이 보인다.
- KRWCoin, Bill이 다른 객체일 이유가 있을까?
- insertCoin, insertBill 메소드를 분리시켜서 사용자에게 혼란을 주지 않을까?
- 돈을 넣고, 거스름돈을 돌려주는데. 은행 ATM도 현금보유량이 있는데 이건 마법의 자판기인가?

이런 문제가 정말 많이 남아있다.

이것도 할일 목록에 적어두고 한편 한편 완성시켜 나가도록 하겠다.

***
### 환경
1. Language: Java 1.8
2. Framework: Spring-Boot-2.1.7, Gradle-5.6
3. Library: Junit-4.12