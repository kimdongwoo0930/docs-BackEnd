# 01. OOP (객체지향 프로그래밍)

<br>

## 1. 왜 객체지향이 필요한가?

초기 프로그래밍은 **절차지향(Procedural)** 방식이었습니다.
코드가 위에서 아래로 순서대로 실행되고, 데이터와 함수가 분리된 채로 존재했어요.

**절차지향의 문제점:**

- 프로그램이 커질수록 코드 간의 의존성이 뒤엉킴
- 데이터를 여러 함수가 공유하다 보니 어디서 값이 바뀌는지 추적하기 어려움
- 기능 하나를 수정하면 연관된 코드 전체를 손봐야 함

```
// 절차지향적 사고
String name = "홍길동";
int age = 25;

void printUser() { ... }
void updateAge() { ... }
// 데이터(name, age)와 동작(함수)이 따로 놀고 있음
```

**객체지향은 이 문제를 이렇게 해결합니다:**

> “연관된 데이터와 동작을 하나의 단위(객체)로 묶자”

```java
// 객체지향적 사고
class User {
    String name;
    int age;

    void printUser() { ... }
    void updateAge() { ... }
    // 데이터와 동작이 User라는 단위 안에 함께 있음
}
```

<br>

## 2. OOP 4대 원칙

### 🔒 캡슐화 (Encapsulation)

**한 줄 정의:** 객체의 내부 데이터를 외부에서 직접 접근하지 못하게 숨기고, 정해진 방법(메서드)으로만 접근하게 한다.

**왜 필요한가?**
외부에서 데이터를 직접 건드리면 객체가 의도하지 않은 상태가 될 수 있어요.

```java
// 캡슐화 없음 → 문제 발생
class BankAccount {
    int balance = 1000;
}

BankAccount account = new BankAccount();
account.balance = -99999; // 잔액이 음수가 돼버림, 막을 방법이 없음
```

```java
// 캡슐화 적용
class BankAccount {
    private int balance = 1000; // 외부 직접 접근 차단

    public void deposit(int amount) {
        if (amount > 0) balance += amount; // 검증 로직을 여기서 통제
    }

    public int getBalance() {
        return balance;
    }
}
```

**핵심:** `private`으로 숨기고, `public` 메서드로만 접근 허용 → 객체가 항상 유효한 상태를 유지

---

### 🧬 상속 (Inheritance)

**한 줄 정의:** 부모 클래스의 속성과 동작을 자식 클래스가 물려받아 재사용한다.

**왜 필요한가?**
공통 기능을 매번 중복해서 작성하지 않기 위해서예요.

```java
class Animal {
    String name;

    void eat() {
        System.out.println(name + "이 먹는다");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println(name + "이 짖는다");
    }
}

class Cat extends Animal {
    void meow() {
        System.out.println(name + "이 운다");
    }
}
// Dog, Cat 모두 eat()을 따로 작성하지 않아도 사용 가능
```

**주의할 점:** 상속은 강한 결합을 만들어요. 부모 클래스가 바뀌면 자식 클래스 전체에 영향이 가기 때문에, 실무에서는 **상속보다 조합(Composition)을 선호**하는 경향이 있어요.

---

### 🎭 다형성 (Polymorphism)

**한 줄 정의:** 같은 타입(인터페이스나 부모 클래스)으로 다양한 객체를 다룰 수 있다.

**왜 필요한가?**
코드가 구체적인 구현에 의존하지 않고, 역할(타입)에만 의존하게 만들어 유연성을 높여요.

```java
class Animal {
    void sound() {
        System.out.println("...");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("멍멍");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("야옹");
    }
}

// 다형성의 힘
Animal animal = new Dog(); // Animal 타입으로 Dog 객체를 가리킴
animal.sound(); // "멍멍" → 실제 객체(Dog)의 메서드가 호출됨

animal = new Cat();
animal.sound(); // "야옹"
```

```java
// 실무에서 자주 보이는 패턴
List<Animal> animals = List.of(new Dog(), new Cat(), new Dog());

for (Animal a : animals) {
    a.sound(); // 타입을 몰라도 각자 맞는 동작을 수행
}
```

**핵심:** `Animal`이라는 타입 하나로 Dog도, Cat도 다룰 수 있음 → 새로운 동물이 추가돼도 반복문 코드는 바뀌지 않음

---

### 🗂 추상화 (Abstraction)

**한 줄 정의:** 복잡한 내부 구현을 숨기고, 필요한 기능의 인터페이스(틀)만 드러낸다.

**왜 필요한가?**
사용하는 쪽에서 내부 동작을 알 필요 없이, “무엇을 할 수 있는가”만 알면 되게 해줘요.

```java
// 추상화의 예 — 인터페이스로 틀을 정의
interface PaymentService {
    void pay(int amount); // "결제한다"는 기능만 정의, 어떻게 하는지는 숨김
}

class KakaoPay implements PaymentService {
    @Override
    public void pay(int amount) {
        // 카카오페이 내부 결제 로직 (복잡해도 외부에서는 안 보임)
        System.out.println("카카오페이로 " + amount + "원 결제");
    }
}

class NaverPay implements PaymentService {
    @Override
    public void pay(int amount) {
        System.out.println("네이버페이로 " + amount + "원 결제");
    }
}

// 사용하는 쪽
class OrderService {
    private PaymentService paymentService;

    public OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }

    public void order(int amount) {
        paymentService.pay(amount); // 어떤 결제 수단인지 몰라도 됨
    }
}
```

**캡슐화와의 차이:**

- 캡슐화 → “내부 데이터를 숨긴다” (필드 레벨)
- 추상화 → “복잡한 구현을 숨기고 인터페이스만 노출한다” (설계 레벨)

<br>

## 3. 4대 원칙 한눈에 보기

| 원칙   | 핵심 질문                                       | 주요 키워드              |
| ------ | ----------------------------------------------- | ------------------------ |
| 캡슐화 | 데이터를 어떻게 보호할 것인가?                  | `private`, getter/setter |
| 상속   | 공통 기능을 어떻게 재사용할 것인가?             | `extends`, `@Override`   |
| 다형성 | 하나의 타입으로 여러 객체를 어떻게 다룰 것인가? | 업캐스팅, `@Override`    |
| 추상화 | 복잡성을 어떻게 감출 것인가?                    | `interface`, `abstract`  |

<br>

## 4. 면접 Q&A

**Q. 객체지향의 4대 원칙을 설명해주세요.**

> 캡슐화는 객체 내부 데이터를 외부에서 직접 접근하지 못하게 숨기고 메서드로만 접근하게 하는 것입니다. 상속은 부모 클래스의 속성과 동작을 자식이 물려받아 재사용하는 것이고, 다형성은 같은 타입으로 다양한 객체를 다룰 수 있는 것입니다. 추상화는 복잡한 구현을 숨기고 필요한 인터페이스만 노출하는 것입니다.

---

**Q. 캡슐화와 추상화의 차이가 뭔가요?**

> 캡슐화는 객체의 필드(데이터)를 외부로부터 보호하는 것으로 `private` 접근 제어자와 getter/setter가 대표적입니다. 추상화는 한 단계 위의 개념으로, 복잡한 구현 자체를 숨기고 “무엇을 할 수 있는가”라는 인터페이스만 노출하는 설계 차원의 이야기입니다.

---

**Q. 상속보다 조합(Composition)을 선호하는 이유가 뭔가요?**

> 상속은 부모-자식 간 강한 결합을 만듭니다. 부모 클래스가 변경되면 자식 클래스 전체에 영향이 가고, 자식은 부모의 모든 것을 물려받기 때문에 불필요한 기능까지 가져오게 됩니다. 반면 조합은 필요한 기능을 가진 객체를 내부에 포함하는 방식이라 결합도가 낮고 유연합니다. Java에서도 “상속보다 인터페이스와 조합을 우선하라”는 원칙이 있습니다.

---

**Q. 다형성이 실무에서 왜 중요한가요?**

> 다형성이 없으면 새로운 타입이 추가될 때마다 사용하는 쪽의 코드를 전부 수정해야 합니다. 다형성을 활용하면 인터페이스 타입 하나로 다양한 구현체를 다룰 수 있어서, 새로운 구현체가 추가돼도 기존 코드를 건드리지 않아도 됩니다. 이는 OCP(개방-폐쇄 원칙)와도 연결됩니다.
