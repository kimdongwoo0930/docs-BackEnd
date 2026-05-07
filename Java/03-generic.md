# 03. 제네릭 (Generic)

<br>

## 1. 왜 제네릭이 필요한가?

제네릭이 없던 시절(Java 1.4 이하), 컬렉션에 여러 타입을 담으려면 모든 것을 `Object`로 처리해야 했어요.

```java
// 제네릭 없을 때
List list = new ArrayList();
list.add("hello");
list.add(123);       // 컴파일 OK → 문제의 시작

String value = (String) list.get(1); // 런타임 에러! ClassCastException
```

**문제점:**
- 꺼낼 때마다 타입 캐스팅 필요
- 잘못된 타입이 들어가도 **컴파일 시점에 잡을 수 없음**
- 런타임에 `ClassCastException` 터짐

제네릭은 **"타입을 나중에 지정할 수 있게"** 하면서, **컴파일 시점에 타입을 검사**해주는 기능이에요.

```java
// 제네릭 적용
List<String> list = new ArrayList<>();
list.add("hello");
list.add(123);       // 컴파일 에러! → 런타임 전에 잡힘

String value = list.get(0); // 캐스팅 불필요
```

<br>

## 2. 어떻게 동작하는가?

### 기본 문법

```java
// 클래스에 제네릭 적용
class Box<T> {          // T: 타입 파라미터 (이름은 관례상 T, E, K, V 등 사용)
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}

// 사용
Box<String> strBox = new Box<>();
strBox.set("hello");
String s = strBox.get(); // 캐스팅 불필요

Box<Integer> intBox = new Box<>();
intBox.set(123);
Integer n = intBox.get();
```

---

### 제네릭 메서드

클래스 전체가 아닌 **메서드 하나에만** 제네릭을 적용할 수도 있어요.

```java
class Util {
    // 메서드에 제네릭 적용 — 반환타입 앞에 <T> 선언
    public static <T> T pick(T a, T b, boolean first) {
        return first ? a : b;
    }
}

String result = Util.pick("hello", "world", true);  // "hello"
Integer num = Util.pick(1, 2, false);               // 2
```

---

### 타입 파라미터 관례

| 이름 | 의미 | 주로 쓰는 곳 |
|------|------|-------------|
| `T` | Type | 일반적인 타입 |
| `E` | Element | 컬렉션 원소 |
| `K` | Key | Map의 키 |
| `V` | Value | Map의 값 |
| `N` | Number | 숫자 타입 |

---

### 타입 소거 (Type Erasure)

제네릭은 **컴파일 시점에만 존재**하고, 런타임에는 타입 정보가 지워져요.

```java
List<String> strList = new ArrayList<>();
List<Integer> intList = new ArrayList<>();

// 런타임에는 둘 다 그냥 List — 타입 정보가 사라짐
System.out.println(strList.getClass() == intList.getClass()); // true
```

> JVM은 제네릭을 모르고, 컴파일러가 타입 검사 후 캐스팅 코드를 자동 삽입하는 방식이에요.

<br>

## 3. 다른 방법과 무엇이 다른가?

### 와일드카드 (`?`)

제네릭 타입이 정해지지 않은 상황에서 **유연하게 받아야 할 때** 사용해요.

```java
// 비제한 와일드카드: 어떤 타입이든 OK
public void printList(List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
}

printList(new ArrayList<String>());
printList(new ArrayList<Integer>());
```

---

### 상한 경계 (`<? extends T>`) — 읽기 전용

```java
// Number 또는 Number의 하위 타입만 허용 (Integer, Double 등)
public double sum(List<? extends Number> list) {
    double total = 0;
    for (Number n : list) {
        total += n.doubleValue();
    }
    return total;
}

sum(List.of(1, 2, 3));          // Integer → OK
sum(List.of(1.1, 2.2));         // Double → OK
sum(List.of("hello"));          // String → 컴파일 에러
```

---

### 하한 경계 (`<? super T>`) — 쓰기 전용

```java
// Integer 또는 Integer의 상위 타입만 허용 (Number, Object 등)
public void addNumbers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
}

List<Number> numList = new ArrayList<>();
addNumbers(numList); // OK
```

---

### PECS 원칙

```
Producer → Extends (읽을 때)
Consumer → Super   (쓸 때)

데이터를 꺼내서 쓸 거라면 → <? extends T>
데이터를 넣을 거라면      → <? super T>
```

<br>

## 4. 실제 코드에서는 어떻게 쓰는가?

### Spring에서 흔히 보이는 제네릭 패턴

```java
// 공통 API 응답 래퍼
public class ApiResponse<T> {
    private int status;
    private String message;
    private T data;         // 응답 데이터 타입을 유연하게

    public static <T> ApiResponse<T> success(T data) {
        return new ApiResponse<>(200, "OK", data);
    }

    public static <T> ApiResponse<T> fail(String message) {
        return new ApiResponse<>(400, message, null);
    }
}

// 사용
ApiResponse<UserResponse> response = ApiResponse.success(userResponse);
ApiResponse<List<PostResponse>> listResponse = ApiResponse.success(posts);
```

---

### 제네릭 Repository 패턴

```java
// 공통 CRUD 인터페이스
public interface BaseRepository<T, ID> {
    T findById(ID id);
    void save(T entity);
    void delete(ID id);
}

// 구현
public class UserRepository implements BaseRepository<User, Long> {
    @Override
    public User findById(Long id) { ... }

    @Override
    public void save(User user) { ... }

    @Override
    public void delete(Long id) { ... }
}
```

<br>

## 5. 면접 Q&A

**Q. 제네릭을 사용하는 이유가 무엇인가요?**

> 제네릭은 컴파일 시점에 타입 안전성을 보장하고, 불필요한 타입 캐스팅을 제거하기 위해 사용합니다. 제네릭 없이 Object 타입으로 처리하면 잘못된 타입이 들어가도 런타임에서야 ClassCastException이 발생하지만, 제네릭을 사용하면 컴파일러가 타입을 미리 검사해 에러를 조기에 발견할 수 있습니다.

---

**Q. 타입 소거(Type Erasure)란 무엇인가요?**

> Java 제네릭은 컴파일 시점에만 타입 정보를 사용하고, 컴파일 후 바이트코드에서는 타입 파라미터가 `Object`로 대체되는 방식으로 동작합니다. 이를 타입 소거라고 합니다. 덕분에 제네릭을 도입하면서도 기존 Java 버전과의 하위 호환성을 유지할 수 있었습니다. 단점으로는 런타임에 제네릭 타입 정보를 알 수 없어 `instanceof List<String>` 같은 검사가 불가능합니다.

---

**Q. `<? extends T>`와 `<? super T>`의 차이를 설명해주세요.**

> `<? extends T>`는 T와 T의 하위 타입만 허용하는 상한 경계로, 데이터를 읽는(생산하는) 용도에 적합합니다. `<? super T>`는 T와 T의 상위 타입만 허용하는 하한 경계로, 데이터를 쓰는(소비하는) 용도에 적합합니다. 이 원칙을 PECS(Producer Extends, Consumer Super)라고 합니다. 예를 들어 `Collections.copy(dest, src)`에서 `src`는 `extends`, `dest`는 `super`를 사용합니다.

---

**Q. 제네릭 클래스와 제네릭 메서드의 차이는 무엇인가요?**

> 제네릭 클래스는 클래스 선언 시 `class Box<T>`처럼 타입 파라미터를 지정해 클래스 전체에서 해당 타입을 사용합니다. 제네릭 메서드는 메서드 반환 타입 앞에 `<T>`를 선언해 해당 메서드 내에서만 타입 파라미터를 사용합니다. 제네릭 메서드는 정적 메서드에도 적용할 수 있어 유틸리티 메서드 작성 시 자주 활용됩니다.
