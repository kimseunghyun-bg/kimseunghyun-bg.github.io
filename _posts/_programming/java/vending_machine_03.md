---
title: "자판기 프로젝트 03 (with.JPA)"
date: 2019-09-11
tags:
  - TDD
  - Vending Machine
---

1. [자판기 프로젝트 01 (with.TDD)](https://kimseunghyun-bg.github.io/programming/java/vending_machine_with_tdd_01/)
2. [자판기 프로젝트 02 (with.TDD)](https://kimseunghyun-bg.github.io/programming/java/vending_machine_with_tdd_02/)
3. [자판기 프로젝트 03 (with.JPA)](https://kimseunghyun-bg.github.io/programming/java/vending_machine_with_tdd_03/)

## 발단
문득 자판기를 만들고 싶었다.

## 기술 목표
- ~~TDD (Junit4)~~
- **Spring JPA**

프로젝트가 진행되면서 기술은 하나씩 추가.

~~(용두사미가 특기라서 미리 다 안써논거다. 썼다가 못하면 창피)~~

## 시작합니다.

`자판기 프로젝트 01`에서는 TDD로 진행하면서 `자판기에 돈 넣기` 기능을 구현했다.

| 할 일 |
| :------------ |
| ~~자판기에서 음료수 뽑기~~ |
| ~~자판기에 음료수 넣기~~ |
| ~~자판기에 돈 넣기~~ |
| ~~자판기에서 거스름돈 나오기~~ |
| **자판기 금전함 기능** |
| *체크카드로 음료수 뽑기(\* 추후 과제)* |
| *신용카드로 음료수 뽑기(\* 추후 과제)* |
| *메타데이터로 화폐정보 변경 해서 국제적인 자판기로 만들기(\* 추후 과제)* |

***

`자판기 금전함 기능`(aka. 돈 통) 시작

천원을 넣고, 500원짜리 음료수를 하나 먹었다.

그러면 자판기 금전함(aka. 돈 통)에는 `+1 천원 지폐` 그리고 `-1 500원 동전`이 되야 한다.

지금은 천원은 허공으로 사라지고, 500원은 시공간을 찢고 나와서 거슬러준다. 아... 음... 

### DB 선택하기

굳이 선택하는 과정을 적을 필요는 없을 것 같다.

- Production: RDB - H2 - file
- Test: RDB - H2 - InMemory

H2는 작은 프로젝트를 진행하기에 적절한 성능을 보여준다.

대용량처리를 할때 H2 쓰면 골로간다 (몇 번 갔다 와봤다)

자판기 금전함 (~~돈통~~, CashBox) 는 영속성(Persistance)가 있어야 되니까 H2의 file DB기능을 사용할 것이고, 테스트 할때는 영속성이 있으면 테스트에 방해가 되니까 InMemory로 테스트 돌릴때마다 내부 데이터를 날려버릴거다.

### 금전함 테이블 스키마
  
| CashBox | |
| :------------ | :------------ |
| faceValue (PK) | Varchar |
| currency | Varchar |
| type | Varchar |
| remaining_amt | Integer |



### Gradle dependencies 추가

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}
```

***

| 할 일 |
| :------------ |
| 자판기 금전함 기능 |
| __DB에 Insert 하기__ |

테스트 케이스를 작성한다.

```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class CashBoxRepositoryTest {

    @Resource
    private CashBoxRepository cashBoxRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    public void cashBoxSave_10KRWCoinTen_ShouldInsert() {
        // Given
        CashBox cashBox = new CashBox("10", "KRW", "Coin", 10);

        // When
        cashBoxRepository.save(cashBox);

        // Then
        Optional<CashBox> actual = cashBoxRepository.findById("10");
        assertThat(actual.get().getRemainingAmount(), is(10));
    }
}
```
컴파일 에러, 필요한 클래스들을 만들어주자

```java
@Entity(name = "cash_box")
public class CashBox {

    @Id @Column(name = "face_value")
    private String faceValue;
    private String currency;
    private String type;
    @Column(name = "remaining_amt")
    private Integer remainingAmount;

    public CashBox() {
    }

    public CashBox(String faceValue, String currency, String type, Integer remainingAmount) {
        this.faceValue = faceValue;
        this.currency = currency;
        this.type = type;
        this.remainingAmount = remainingAmount;
    }

    public String getFaceValue() {
        return faceValue;
    }

    public void setFaceValue(String faceValue) {
        this.faceValue = faceValue;
    }

    public String getCurrency() {
        return currency;
    }

    public void setCurrency(String currency) {
        this.currency = currency;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public Integer getRemainingAmount() {
        return remainingAmount;
    }

    public void setRemainingAmount(Integer remainingAmount) {
        this.remainingAmount = remainingAmount;
    }
}
```

```java
@Repository
public interface CashBoxRepository extends CrudRepository<CashBox, String> {
}
```
갑자기 어노테이션이 등장하고 혼란스러울 것 같다.

이전 포스트 까지는 POJO였는데... 갑자기 스프링입니다. 갑작스러워서 죄송합니다.

어쨌든 테스트를 해보면 성공으로 나온다.

| 할 일 |
| :------------ |
| 자판기 시제 기능 |
| ~~DB에 Insert 하기~~ |

JPA없이 DAO를 만들면 Connection가져오고, Prepared Statement를 만드는 과정이 코딩되어야 하는데 JPA와 Spring을 쓰면서 그 과정이 모두 사라졌다. 생산성이 역시 갑이다. 메뉴얼 쿼리도 지원하기때문에 운영하다가 성능상 필요한 부분은 메뉴얼 쿼리를 넣어서 처리하면 된다.

(* 나중에 리펙토링을 하기 위해 지금은 일부러 괴물같은 스파게티 코드를 만들고 있습니다. 이렇게 코딩하지 마세요.)

***

| 할 일 |
| :------------ |
| 자판기 시제 기능 |
| ~~DB에 Insert 하기~~ |
| **200원 반환할 때 금전함(~~돈통~~, Cash Drawer)에서 100원 -2 하기** |

테스트
```java
// Mock객체를 쓰는게 이번 테스트에 원활 할것 같다.
@RunWith(MockitoJUnitRunner.class)
public class MachineTest {

    // Mock 객체
    @Mock
    private CashBoxRepository cashBoxRepository;

    // Mock 객체(CashBoxRepository)를 멤버로 가질 객체
    @InjectMocks
    private Machine machine;

    @Test
    public void updateCashBoxRemainingAMT_100KRWMinusTWO_AmountShouldZero() {
        // Given
        CashBox cashBox = new CashBox("100", "KRW", "Coin", 2);
        when(cashBoxRepository.findById("100")).thenReturn(Optional.of(cashBox));
        when(cashBoxRepository.save(any())).thenAnswer((Answer<CashBox>) invocation -> {
            Object[] args = invocation.getArguments();
            return (CashBox) args[0];
        });
        // When
        CashBox actual = machine.updateCashBoxRemainingAMT(KRWCoin.oneHundred(), -2);
        // Then
        assertThat(actual.getRemainingAmount(), is(0));
    }
}
```
구현 코드
```java
@Service
public class Machine {

    @Resource
    private CashBoxRepository cashBoxRepository;

    public CashBox updateCashBoxRemainingAMT(KRWCoin krwCoin, int change) {
        String faceValue = "10";
        if (krwCoin.equals(KRWCoin.oneHundred())) {
            faceValue = "100";
        }
        CashBox cashBox = cashBoxRepository.findById(faceValue).get();
        cashBox.setRemainingAmount(cashBox.getRemainingAmount()+change);
        return cashBoxRepository.save(cashBox);
    }
}
```

초록 막대기

| 할 일 |
| :------------ |
| 자판기 시제 기능 |
| ~~DB에 Insert 하기~~ |
| ~~200원 반환할 때 금전함(~~돈통~~, Cash Drawer)에서 100원 -2 하기~~ |

죄악을 마음껏 저질러보자

```java
public class Machine {
    public CashBox updateCashBoxRemainingAMT(KRWCoin krwCoin, int change) {
        String faceValue = "0";
        if (krwCoin.equals(KRWCoin.ten())) {
            faceValue = "10";
        } else if (krwCoin.equals(KRWCoin.fifty())) {
            faceValue = "50";
        } else if (krwCoin.equals(KRWCoin.oneHundred())) {
            faceValue = "100";
        } else if (krwCoin.equals(KRWCoin.fiveHundred())) {
            faceValue = "500";
        }
        CashBox cashBox = cashBoxRepository.findById(faceValue).get();
        cashBox.setRemainingAmount(cashBox.getRemainingAmount()+change);
        return cashBoxRepository.save(cashBox);
    }
}
```
if문이 무려 4개!!! 하드코딩의 끝판왕

잠깐!!

아직 지폐가 남았소!

***

| 할 일 |
| :------------ |
| 자판기 시제 기능 |
| ~~DB에 Insert 하기~~ |
| ~~200원 반환할 때 금전함(~~돈통~~, Cash Drawer)에서 100원 -2 하기~~ |
| **2000원 반환할 때 금전함(~~돈통~~, Cash Drawer)에서 1000원 -2 하기** |

```java
public class MachineTest {
    @Test
    public void updateCashBoxRemainingAMT_1000KRWMinusTWO_AmountShouldZero() {
        // Given
        CashBox cashBox = new CashBox("1000", "KRW", "Coin", 2);
        when(cashBoxRepository.findById("1000")).thenReturn(Optional.of(cashBox));
        when(cashBoxRepository.save(any())).thenAnswer((Answer<CashBox>) invocation -> {
            Object[] args = invocation.getArguments();
            return (CashBox) args[0];
        });
        // When
        CashBox actual = machine.updateCashBoxRemainingAMT(KRWBill.oneThousand(), -2);
        // Then
        assertThat(actual.getRemainingAmount(), is(0));
    }
}
```

```java
public class Machine {
    public CashBox updateCashBoxRemainingAMT(KRWBill krwBill, int change) {
        String faceValue = "0";
        if (krwBill.equals(KRWBill.oneThousand())) {
            faceValue = "1000";
        } else if (krwBill.equals(KRWBill.fiveThousand())) {
            faceValue = "5000";
        } else if (krwBill.equals(KRWBill.tenThousand())) {
            faceValue = "10000";
        } else if (krwBill.equals(KRWBill.fiftyThousand())) {
            faceValue = "50000";
        }
        CashBox cashBox = cashBoxRepository.findById(faceValue).get();
        cashBox.setRemainingAmount(cashBox.getRemainingAmount()+change);
        return cashBoxRepository.save(cashBox);
    }
}
```
여기도 if문이 무려 4개!!! 하드코딩의 끝판왕

| 할 일 |
| :------------ |
| 자판기 시제 기능 |
| ~~DB에 Insert 하기~~ |
| ~~200원 반환할 때 금전함(~~돈통~~, Cash Drawer)에서 100원 -2 하기~~ |
| ~~2000원 반환할 때 금전함(~~돈통~~, Cash Drawer)에서 1000원 -2 하기~~ |

***

정리
---
TDD에서 좀 벗어난 코딩을 하고있다.

하나의 `할 일`을 한 단계, 한 단계 진행 하면서 끝에는 꼭 리펙토링을 해줘야 한다.

그렇지만 난 하지 않았다!!

CI / CD 까지 구축해서 실시간 배포환경에서 리펙토링하며 버그를 잡는 위엄을 느끼고 싶었는데, 도저히 안되겠다.

일단 다음 포스트부터는 앞에 했던 부분들 리펙토링하는 시간을 가져보자.

***
### 환경
1. Language: Java 1.8
2. Framework: Spring-Boot-2.1.7, spring-boot-starter-test-2.1.7, spring-boot-starter-data-jpa-2.1.7, Gradle-5.6
3. Library: Junit-4.12, Mockito-2.23.4
