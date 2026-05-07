# 02. 인터페이스 vs 추상클래스

<br>

## 1. 왜 둘 다 존재하는가?

OOP에서 추상화를 표현하는 방법은 두 가지입니다. 그런데 왜 Java는 `interface`와 `abstract class` 두 가지를 모두 제공할까요?

**문제 상황:**

```
Animal이라는 개념을 코드로 표현하고 싶다.
- 모든 동물은 "소리를 낸다"는 동작이 있다.
- 그런데 어떤 소리인지는 동물마다 다르다.
- 어떻게 틀만 정의하고, 구현은 자식에게 맡길까?
```

여기서 두 가지 요구사항이 충돌합니다.

| 요구사항 | 설명 |
|----------|------|
| **틀(계약)만 정의** | 구현 없이 "이 동작을 반드시 만들어야 해"를 강제하고 싶다 |
| **공통 구현 제공** | 자식 클래스들이 공통으로 쓸 코드를 미리 작성해두고 싶다 |

`interface`는 첫 번째, `abstract class`는 두 번째에 특화된 도구입니다.

<br>

## 2. 어떻게 동작하는가?

### 🔷 인터페이스 (Interface)

**"이 메서드를 반드시 구현해야 해"라는 계약서**

```java
interface Flyable {
    void fly(); // 구현 없음, 반드시 오버라이드해야 함
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {
    @Override
    public void fly() {
        System.out.println("오리가 날아간다");
    }

    @Override
    public void swim() {
        System.out.println("오리가 수영한다");
    }
}
```

**핵심 특징:**
- 다중 구현 가능 (`implements A, B, C`)
- 모든 메서드가 기본적으로 `public abstract`
- 필드는 `public static final` 상수만 가능
- Java 8부터 `default` 메서드로 기본 구현 제공 가능

```java
interface Greeting {
    // Java 8+: default 메서드로 기본 구현 제공 가능
    default void hello() {
        System.out.println("안녕하세요!");
    }

    void introduce(); // 이건 여전히 반드시 구현해야 함
}
```

---

### 🔶 추상클래스 (Abstract Class)

**"공통 기능은 내가 구현해줄게, 나머지만 네가 채워"**

```java
abstract class Animal {
    String name;

    Animal(String name) {
        this.name = name; // 생성자 가능
    }

    // 공통 구현 제공
    void breathe() {
        System.out.println(name + "이 숨을 쉰다");
    }

    // 이건 자식이 반드시 구현해야 함
    abstract void sound();
}

class Dog extends Animal {
    Dog(String name) {
        super(name);
    }

    @Override
    void sound() {
        System.out.println(name + ": 멍멍");
    }
}

class Cat extends Animal {
    Cat(String name) {
        super(name);
    }

    @Override
    void sound() {
        System.out.println(name + ": 야옹");
    }
}
```

**핵심 특징:**
- 단일 상속만 가능 (`extends`는 하나만)
- 일반 필드, 생성자, 일반 메서드 모두 가질 수 있음
- `abstract` 메서드가 하나라도 있으면 클래스 자체도 `abstract`

<br>

## 3. 인터페이스 vs 추상클래스 비교

| 항목 | 인터페이스 | 추상클래스 |
|------|-----------|-----------|
| **상속 수** | 다중 구현 가능 | 단일 상속만 |
| **필드** | 상수(`static final`)만 | 일반 필드 가능 |
| **생성자** | 없음 | 있음 |
| **접근 제어자** | `public`만 | 모두 가능 |
| **목적** | 타입(역할) 정의 | 공통 구현 공유 |
| **관계** | ~할 수 있다 (has-a 역할) | ~이다 (is-a 관계) |

### 언제 무엇을 쓸까?

```
✅ 인터페이스를 쓸 때
- 서로 관계없는 클래스들이 같은 동작을 해야 할 때
  (예: Bird도 Flyable, Plane도 Flyable → 둘은 관련 없음)
- 다중 구현이 필요할 때
- Spring에서 의존성 주입 타입을 정의할 때 (가장 흔한 사용)

✅ 추상클래스를 쓸 때
- 클래스 간에 명확한 "is-a" 관계가 있을 때
  (예: Dog is an Animal, Cat is an Animal)
- 공통 필드나 공통 구현 코드를 자식 클래스에 물려줄 때
- 템플릿 메서드 패턴처럼 "처리 흐름은 고정, 세부 구현만 다르게" 할 때
```

<br>

## 4. 실제 코드에서는 어떻게 쓰는가?

### Spring에서의 인터페이스 활용 (가장 흔한 패턴)

```java
// 인터페이스로 역할 정의
public interface UserRepository {
    User findById(Long id);
    void save(User user);
}

// 실제 구현체
@Repository
public class JpaUserRepository implements UserRepository {
    @Override
    public User findById(Long id) {
        // JPA로 DB 조회
    }

    @Override
    public void save(User user) {
        // JPA로 저장
    }
}

// 사용하는 쪽 — 인터페이스 타입에만 의존
@Service
public class UserService {
    private final UserRepository userRepository; // 구현체가 뭔지 모름

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

> 이 구조 덕분에 `JpaUserRepository`를 `MongoUserRepository`로 바꿔도 `UserService` 코드는 전혀 수정하지 않아도 됩니다.

---

### 추상클래스 활용 — 템플릿 메서드 패턴

```java
// 처리 흐름은 고정, 세부 구현만 자식이 채움
abstract class DataExporter {

    // 템플릿 메서드: 흐름을 고정
    public final void export() {
        List<Object> data = fetchData();   // 1. 데이터 가져오기
        List<Object> filtered = filter(data); // 2. 필터링
        write(filtered);                   // 3. 출력
    }

    protected abstract List<Object> fetchData(); // 자식이 구현
    protected abstract void write(List<Object> data); // 자식이 구현

    // 공통 구현 제공
    protected List<Object> filter(List<Object> data) {
        return data.stream()
                .filter(d -> d != null)
                .collect(Collectors.toList());
    }
}

class CsvExporter extends DataExporter {
    @Override
    protected List<Object> fetchData() { /* DB에서 가져오기 */ }

    @Override
    protected void write(List<Object> data) { /* CSV로 쓰기 */ }
}

class ExcelExporter extends DataExporter {
    @Override
    protected List<Object> fetchData() { /* API에서 가져오기 */ }

    @Override
    protected void write(List<Object> data) { /* Excel로 쓰기 */ }
}
```

<br>

## 5. 면접 Q&A

**Q. 인터페이스와 추상클래스의 차이를 설명해주세요.**

> 인터페이스는 "무엇을 할 수 있는가"라는 역할/계약을 정의하는 도구로, 다중 구현이 가능하고 상수와 추상 메서드만 가질 수 있습니다. 추상클래스는 "공통 구현을 공유"하는 목적으로, 일반 필드와 메서드를 포함할 수 있지만 단일 상속만 됩니다. 관계로 보면 인터페이스는 "~할 수 있다(has-a 역할)", 추상클래스는 "~이다(is-a 관계)"에 해당합니다.

---

**Q. Java 8에서 인터페이스에 default 메서드가 추가된 이유가 뭔가요?**

> 기존 인터페이스에 새 메서드를 추가하면 그 인터페이스를 구현한 모든 클래스를 수정해야 하는 문제가 있었습니다. Java 8에서 `default` 메서드가 도입되면서 인터페이스에 기본 구현을 제공할 수 있게 되어, 기존 구현 클래스에 영향을 주지 않고 인터페이스를 확장할 수 있게 됐습니다. 대표적인 예로 `Collection` 인터페이스에 `forEach()`, `stream()` 등이 추가된 것이 있습니다.

---

**Q. 인터페이스를 다중 구현할 때 default 메서드가 충돌하면 어떻게 되나요?**

> 두 인터페이스에 동일한 시그니처의 `default` 메서드가 있을 경우 컴파일 에러가 발생합니다. 이때 구현 클래스에서 해당 메서드를 직접 오버라이드해서 충돌을 해결해야 합니다. 특정 인터페이스의 구현을 사용하고 싶다면 `InterfaceA.super.method()` 형태로 명시할 수 있습니다.

---

**Q. Spring에서 인터페이스를 사용하는 이유가 뭔가요?**

> Spring의 핵심인 DI(의존성 주입)와 AOP가 인터페이스 기반으로 동작하기 때문입니다. 서비스 클래스가 인터페이스 타입에만 의존하면, Spring이 런타임에 적절한 구현체를 주입해줍니다. 또한 JDK 동적 프록시는 인터페이스가 있어야 동작하므로, 트랜잭션 처리나 AOP 적용 시 인터페이스가 중요한 역할을 합니다.
