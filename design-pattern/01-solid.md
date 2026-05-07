# SOLID 원칙

SOLID는 객체지향 설계에서 자주 언급되는 5가지 설계 원칙의 약자이다.

- 단일 책임 원칙 (SRP)
- 개방-폐쇄 원칙 (OCP)
- 리스코프 치환 원칙 (LSP)
- 인터페이스 분리 원칙 (ISP)
- 의존성 역전 원칙 (DIP)

> 모두 변경에 유연하게 대응할 수 있는 코드 구조를 만들기 위한 도구이다.

SOLID의 목적은 단순히 클래스를 많이 나누는 것이 아니라, 변경이 생겼을 때 수정 범위를 작게 만들고 코드의 의도를 더 분명하게 만드는 것이다.

---

## 단일 책임 원칙 (SRP)

`단일 책임 원칙`은 "하나의 클래스는 하나의 책임만 가져야 한다"는 원칙이다.

여기서 책임이란 단순히 기능 하나를 뜻한다기보다, "변경되는 이유"라고 볼 수 있다. 즉, 클래스가 변경되어야 하는 이유가 여러 개라면 책임이 여러 개일 가능성이 높다.

- 여러 가지 역할을 가지고 있는 클래스는 한 기능을 변경할 때 다른 영역까지 영향을 받을 수 있기 때문에 유지보수에 취약하다.
- 비즈니스 로직, 저장 로직, 출력 로직, 검증 로직 등이 한 클래스에 섞이면 SRP를 위반하기 쉽다.

Before:

```java
class User {
    private String name;
    private String email;

    public void saveToFile(String filename) {
        // 파일 저장
    }

    public void sendEmail() {
        // 이메일 발송
    }
}
```

`User` 클래스가 사용자 정보 관리, 파일 저장, 이메일 발송까지 담당하고 있다.

After:

```java
class User {
    private String name;
    private String email;

    // 사용자 정보 관련 기능
}

class FileManager {
    public void saveToFile(User user, String filename) {
        // 파일 저장
    }

    public User loadFromFile(String filename) {
        // 파일 읽기
        return null;
    }
}

class EmailService {
    public void sendEmail(User user) {
        // 이메일 발송
    }
}
```

이런 식으로 역할을 나누면 파일 저장 방식이 변경될 때는 `FileManager`만 수정하면 되고, 이메일 발송 방식이 변경될 때는 `EmailService`만 수정하면 된다.

---

## 개방-폐쇄 원칙 (OCP)

`개방-폐쇄 원칙`이란 "확장에는 열려 있고, 변경에는 닫혀 있어야 한다"는 원칙이다.

- 새로운 기능을 추가할 때 기존 코드를 계속 수정하지 않고도 확장할 수 있어야 한다.
- `if`, `switch`가 계속 늘어나는 구조는 OCP 위반을 의심해 볼 수 있다.
- 인터페이스, 추상 클래스, 다형성을 활용하면 기존 코드를 덜 건드리고 기능을 확장할 수 있다.

Before:

```java
public class PaymentService {
    public void processPayment(String method, double amount) {
        if (method.equals("CREDIT_CARD")) {
            // 카드 결제
        } else if (method.equals("PAYPAL")) {
            // 페이팔 결제
        } else if (method.equals("KAKAO_PAY")) {
            // 카카오페이 결제
        }
    }
}
```

결제 방법이 추가될수록 계속 조건문을 추가해야 하고, `PaymentService` 클래스를 수정해야 한다.

After:

```java
public interface PaymentMethod {
    void pay(double amount);
}

public class CreditCard implements PaymentMethod {
    public void pay(double amount) {
        // 카드 결제
    }
}

public class PayPal implements PaymentMethod {
    public void pay(double amount) {
        // 페이팔 결제
    }
}

public class PaymentService {
    private final PaymentMethod paymentMethod;

    public PaymentService(PaymentMethod paymentMethod) {
        this.paymentMethod = paymentMethod;
    }

    public void process(double amount) {
        paymentMethod.pay(amount);
    }
}
```

이런 식으로 만들면 새로운 결제 수단이 추가되더라도 `PaymentService` 코드는 건드리지 않고 새로운 클래스를 만들어 `PaymentMethod` 인터페이스를 구현하면 된다.

---

## 리스코프 치환 원칙 (LSP)

`리스코프 치환 원칙`이란 "서브 타입은 언제나 자신의 기반 타입으로 교체할 수 있어야 한다"는 원칙이다.

어떤 객체가 부모 클래스 타입으로 정의되어 있다면, 그 자리에 자식 클래스 객체를 넣어도 기존 동작에 문제가 없어야 한다.

> 이 원칙은 상속을 사용할 때 반드시 지켜야 하는 조건이다.

LSP를 위반하면 컴파일은 되지만 실제 동작이 부모 타입을 사용할 때의 기대와 달라진다.

Before:

```java
class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

class Square extends Rectangle {
    public void setWidth(int width) {
        this.width = width;
        this.height = width;
    }

    public void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}

class RectangleResizer {
    public void resize(Rectangle rect) {
        rect.setWidth(5);
        rect.setHeight(10);
    }
}
```

`resize` 메서드는 가로와 세로가 각각 독립적인 사각형을 가정한다. 그런데 `Square` 객체를 파라미터로 넘기면 가로와 세로가 항상 같아지기 때문에 의도와 다르게 동작한다.

Solution:

```java
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private int width;
    private int height;

    public int getArea() {
        return width * height;
    }
}

class Square implements Shape {
    private int side;

    public int getArea() {
        return side * side;
    }
}
```

상속 관계로 억지로 묶기보다, 공통 동작만 인터페이스로 분리하면 타입의 기대 동작을 깨지 않을 수 있다.

---

## 인터페이스 분리 원칙 (ISP)

`인터페이스 분리 원칙`이란 "클라이언트는 자신이 사용하지 않는 인터페이스에 의존하지 않아야 한다"는 원칙이다.

> 하나의 거대한 인터페이스보다 용도에 맞게 나누어진 여러 개의 인터페이스가 낫다는 철학을 바탕으로 한다.

- 인터페이스가 너무 크면 구현 클래스가 필요 없는 메서드까지 구현해야 한다.
- 필요 없는 메서드에 `throw new UnsupportedOperationException()` 같은 처리가 늘어난다면 ISP 위반을 의심할 수 있다.

Before:

```java
public interface MultiFunctionDevice {
    void print();
    void scan();
    void fax();
}
```

만약 복합기가 아니라 일반 프린터라면 `scan`, `fax` 메서드를 억지로 처리해야 하기 때문에 좋지 않다.

After:

```java
public interface Printer {
    void print();
}

public interface Scanner {
    void scan();
}

public interface Fax {
    void fax();
}

public class BasicPrinter implements Printer {
    public void print() {
        System.out.println("문서 출력");
    }
}

public class AdvancedDevice implements Printer, Scanner, Fax {
    public void print() {
        System.out.println("문서 출력");
    }

    public void scan() {
        System.out.println("문서 스캔");
    }

    public void fax() {
        System.out.println("팩스 전송");
    }
}
```

이런 식으로 나누면 불필요한 예외 처리가 없어지고, 필요한 기능에만 의존해서 사용할 수 있다.

---

## 의존성 역전 원칙 (DIP)

`의존성 역전 원칙`이란 "고수준 모듈이 저수준 모듈에 직접 의존하지 말고, 둘 다 추상화된 인터페이스에 의존하게 설계하라"는 원칙이다.

- 고수준 모듈: 비즈니스 정책이나 핵심 흐름을 담당하는 코드
- 저수준 모듈: DB, 파일 시스템, 외부 API, 프레임워크처럼 세부 구현을 담당하는 코드

DB나 외부 API에 직접 엮여 있으면 기능 확장이 어렵다. 그러나 인터페이스나 추상 클래스를 통해 의존성을 분리하면 구조가 더 유연해진다.

Before:

```java
public class Service {
    private final MySql database = new MySql();

    public void doSomething() {
        database.connect();
        // 기타 로직
    }
}

class MySql {
    public void connect() {
        // MySQL 연결
    }
}
```

이 코드에서는 `Service` 클래스가 `MySql`이라는 구체 클래스에 의존하고 있어서 다른 데이터베이스를 사용하려면 `Service` 클래스도 같이 수정해야 한다.

After:

```java
public interface Database {
    void connect();
}

public class MySql implements Database {
    public void connect() {
        // MySQL 연결
    }
}

public class Oracle implements Database {
    public void connect() {
        // Oracle 연결
    }
}

public class Service {
    private final Database database;

    public Service(Database database) {
        this.database = database;
    }

    public void doSomething() {
        database.connect();
        // 기타 로직
    }
}
```

이런 식으로 만들면 `Service`는 `Database`라는 인터페이스에만 의존하기 때문에 MySQL을 사용하든 Oracle을 사용하든 자유롭게 변경할 수 있다. 테스트할 때도 Mock 객체를 주입하기 쉬워진다.

---

## 요약

| 원칙 | 핵심 의미 | 위반 신호 | 개선 방향 |
| --- | --- | --- | --- |
| SRP | 하나의 클래스는 하나의 변경 이유만 가져야 한다. | 한 클래스가 저장, 검증, 출력, 통신 등 여러 역할을 함께 처리한다. | 역할별로 클래스를 분리한다. |
| OCP | 기존 코드는 닫혀 있고, 새 기능 확장에는 열려 있어야 한다. | 새 기능이 생길 때마다 기존 조건문이나 클래스를 계속 수정한다. | 인터페이스와 다형성으로 확장 지점을 만든다. |
| LSP | 자식 타입은 부모 타입을 대체해도 정상 동작해야 한다. | 부모 타입을 기대한 코드에 자식 타입을 넣으면 결과가 달라진다. | 잘못된 상속을 피하고 공통 계약을 명확히 한다. |
| ISP | 사용하지 않는 기능에 의존하지 않아야 한다. | 구현 클래스가 필요 없는 메서드까지 억지로 구현한다. | 큰 인터페이스를 역할별 작은 인터페이스로 나눈다. |
| DIP | 구체 구현이 아니라 추상화에 의존해야 한다. | 서비스 코드가 DB, API, 파일 시스템 같은 구현체를 직접 생성한다. | 인터페이스를 두고 생성자 주입 등으로 구현체를 전달한다. |

SOLID는 객체지향 설계의 핵심이자 유지보수성을 높이는 지침이다. 다만 원칙 자체가 목적은 아니며, 변경 가능성이 높거나 복잡도가 커지는 부분에서 코드의 의존성과 책임을 정리하기 위한 기준으로 사용하는 것이 좋다.
