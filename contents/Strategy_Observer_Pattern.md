# Strategy Pattern(전략 패턴), Observer Pattern(옵저버 패턴)

## Strategy Pattern(전략 패턴)

### 1. Strategy Pattern 란?
- Strategy Pattern(전략 패턴)이란 Policy Pattern(정책 패턴)이라고도 한다.
- 객체의 행위를 바꾸고 싶은 경우 '직접' 수정하지 않고 전략이라고 부르는 '캡슐화한 알고리즘'을 컨텍스트안에서 바꿔주면서 상호 교체가 가능하게 만드는 패턴이다. 즉, 캡슐화한 알고리즘을 변경해줌으로써 유연하게 확장하는 방법이다.
  - `컨텍스트란? 프로그래밍에서의 컨텍스트는 상황, 맥락, 문맥을 의미하며 개발자가 어떠한 작업을 완료하는 데 필요한 모든 관련 정보를 말한다.`
- 런타임에 프로그램의 동작을 동적으로 변경할 수 있다.
- 전략 패턴을 활용하면, OCP(개방-폐쇄원칙)을 위반하지 않고 새로운 전략을 추가하는 것만으로 코드의 수정없이 기능을 추가할 수 있다는 장점을 가지고있다.
---


### 2. Strategy Pattern Code Example
```
  <가정>
  카페를 창업했다.
  현재, 신용카드 결제 시스템이 아직 적용되지 않아서 현재는 현금 결제만 가능하다.
  15일 뒤에 신용카드 결제 시스템이 도입된다.
  30일 뒤에는 Apple Pay도 도입 예정이다.
  ```
<br>

**[전략 패턴을 사용하기 전]**
```java
public interface PaymentService {
    void performPay(int money);
}
```

```java
public class PaymentServiceImpl implements PaymentService {
   @Override
   public void performPay(int money) {
      System.out.println("현금 결제 성공! 결제 금액은" + money + "원 입니다.");
   }
}
```

<br>
위와 같이 코드를 작성한 뒤 <br>
신용카드와 Apple Pay를 도입한다고 하면 아래와 같이 결제 수단이라는 변수를 추가하여 코드를 수정해야한다.
<br>

```java
public interface PaymentService {
   void performPay(int money, String method);
}
```

```java
public class PaymentServiceImpl implements PaymentService {
    @Override
    public void performPay(int money, String method) {
        if(method.equals("현금")) {
            System.out.println("현금 결제 성공! 결제 금액은 = " + money + "원 입니다.");
        } 
        
        if(method.equals("신용카드")) {
            System.out.println("신용카드 결제 성공! 결제 금액은 = " + money + "원 입니다.");
        }
    }
}
```

결제 수단이 추가될 때 마다 PaymentServiceImpl의 코드를 수정해야 한다. <br>
결제 수단이 추가될 때 마다 분기가 계속 늘어날 것이다. <br>
이렇게 되면 OCP 원칙을 위반하고 분기처리가 늘어남으로써 가독성이 떨어진다.
<br><br>

**[전략 패턴 사용]**

<br>
전략 패턴을 사용하기 위해서는 캡슐화된 알고리즘을 만들고, 해당 알고리즘군을 교체해서 사용할 수 있도록 해야한다.
<br><br>
먼저 PaymentStrategy 인터페이스를 만들어준다.

```java
public interface PaymentStrategy {
   void performPayment(int money);
}
```

<br>
PaymentStrategy 인터페이스를 구현한 다양한 결제 전략 클래스(구현체)를 만들어준다. <br>
총 3가지의 캡슐화된 알고리즘을 가진 전략 클래스를 만든 것이다.

```java
public class CardPaymentStrategy implements PaymentStrategy{
    @Override
    public void performPayment(int money) {
        System.out.println("신용카드 결제 성공! 결제 금액은 = " + money + "원 입니다.");
    }
}

public class CashPaymentStrategy implements PaymentStrategy{
   @Override
   public void performPayment(int money) {
      System.out.println("현금 결제 성공! 결제 금액은 = " + money + "원 입니다.");
   }
}

public class ApplePaymentStrategy implements PaymentStrategy{
   @Override
   public void performPayment(int money) {

      System.out.println("애플페이 결제 성공! 결제 금액은 = " + money + "원 입니다.");
   }
}
```

<br>
실행 시점에 Payment 클래스에 지정해주게되면, 각 구현체에 맞는 결제 전략으로 결제되도록 구현하게된다. <br>

```java
public class Payment {
    private PaymentStrategy paymentStrategy;

    public Payment(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void performPayment(int money) {
        this.paymentStrategy.performPayment(money);
    }
}
```

<br>
Payment 클래스에서 결제를 진행하지 않고, 결제가 이루어지는 부분을 PaymentStrategy에 위임하게 됨으로써 Payment 클래스는 어떤 방식으로 결제를 하게 되는지 알필요가 없다.

```java
public class test {
    public static void main(String[] args) {
        Payment payment = new Payment(new CashPaymentStrategy());
        payment.performPayment(5000);

        payment.setPaymentStrategy(new CardPaymentStrategy());
        payment.performPayment(8000);

        payment.setPaymentStrategy(new ApplePaymentStrategy());
        payment.performPayment(6500);
    }
}

[결과]
현금 결제 성공! 결제 금액은 = 5000원 입니다.
신용카드 결제 성공! 결제 금액은 = 8000원 입니다.
애플페이 결제 성공! 결제 금액은 = 6500원 입니다.
```

---
### 3. 장점과 단점
1. 장점
   1) 알고리즘을 정의하고 캡슐화하여 런타임 시에 알고리즘을 선택하는 데 사용
   2) 알고리즘을 쉽게 변경 및 대체할 수 있으므로 유연하다.
   3) 알고리즘 추가 및 수정을 할 때 코드 수정이 최소화되므로 확장성이 높아진다.
   4) 알고리즘을 캡슐화했기에 코드 재사용성이 좋다.
   5) 각각 알고리즘을 독립적으로 테스트할 수 있으므로 용이하다.

   
2. 단점
   1) 추가적인 클래스 및 인터페이스가 필요하기에 코드 복잡성이 증가될 수 있다.
   2) 런타임 시에 알고리즘을 선택하는 데 추가적인 오버헤드 발생이 가능하다.
   3) 전략패턴을 구현하는 것이 어려울 수 있으므로, 적절한 분석과 설계가 필요하다.

---
## Observer Pattern(옵저버 패턴)

### 1. Observer Pattern 란?
- Observer Pattern은 주체가 어떤 객체의 상태 변화를 관찰하다가 상태 변화가 있을 때마다 메서드 등을 통해 옵저버 목록에 있는 옵저버들에게 변화를 알려주는 디자인 패턴이다.
- 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들에게 연락이 가고 자동으로 정보가 갱신되는 1:N 관계를 정의한다.
- 주체란, 객체의 상태 변화를 보고 있는 관찰자이다.
- 옵저버들이란, 이 객체의 상태 변화에 따라 전달되는 메서드 등을 기반으로 '추가 변화 사항'이 생기는 객체들을 의미한다.
---

### 2. Observer Pattern Code Example
```java
// 옵저버들을 관리하는 인터페이스, Observable(발행자)이며, 관찰 대상자
public interface Subject {
    void register(Observer obj);

    void remove(Observer obj);

    void notifyObj();
}
```

<br>

```java
public class SubjectImpl implements Subject {

    // 관찰자들을 등록하여 담는 리스트
    List<Observer> objList = new ArrayList<>();

    // 관찰자를 리스트에 등록
    @Override
    public void register(Observer obj) {
        objList.add(obj);
        System.out.println(obj + " 님 구독 완료");
    }

    // 관찰자를 리스트에서 제거
    @Override
    public void remove(Observer obj) {
        objList.remove(obj);
        System.out.println(obj + "님 구독 해지");
    }

    // 관찰자에게 이벤트 발송
    @Override
    public void notifyObj() {
        for (Observer obj : objList) {
            obj.update();
        }
    }
}
```

<br>

```java
// 옵저버 == 관찰자/구독자
public interface Observer {
    void update();
}


class ObserverA implements Observer {

    @Override
    public void update() {
        System.out.println("ObserverA 로부터 이벤트 알림이 왔습니다.");
    }

    @Override
    public String toString() {
        return "ObserverA";
    }
}

class ObserverB implements Observer {

    @Override
    public void update() {
        System.out.println("ObserverB 로부터 이벤트 알림이 왔습니다.");
    }

    @Override
    public String toString() {
        return "ObserverB";
    }
}
```

<br>

```java
public class observerTest {
    public static void main(String[] args) {

        // 관찰 대상자 등록
        Subject subject = new SubjectImpl();

        // 관찰자들 리스트로 등록
        Observer obj1 = new ObserverA(); // 옵저버A 선언
        Observer obj2 = new ObserverB(); // 옵저버B 선언
        subject.register(obj1);
        subject.register(obj2);

        // 관찰자에게 이벤트 전파
        subject.notifyObj();

        // obj2가 구독 취소
        subject.remove(obj2);

        // 관찰자에게 이벤트 전파
        subject.notifyObj();

    }
}

[결과]
ObserverA 님 구독 완료
ObserverB 님 구독 완료
ObserverA 로부터 이벤트 알림이 왔습니다.
ObserverB 로부터 이벤트 알림이 왔습니다.
ObserverB님 구독 해지
```

---
### 3. 장점과 단점
1. 장점
   1) 객체의 상태 변경을 주기적으로 조회하지 않고 자동으로 감지할 수 있다.
   2) 관찰 대상자의 코드를 변경하지 않고도 새로운 옵저버 클래스를 도입할 수 있어 OCP 원칙을 준수한다.
   3) 런타임 시점에서에 관찰 대상자와 관찰자의 관계를 맺을 수 있다.
   4) 상태를 변경하는 객체와 변경을 감지하는 주체의 관계를 느슨하게 유지할 수 있다.


2. 단점
   1) 순서를 제어할수 없고, 무작위 순서로 알림을 받는다.
   2) 코드 복잡도가 증가한다.
   3) 옵저버 객체를 등록 이후에 해지하지 않으면 메모리 누수가 발생할 수 있다.