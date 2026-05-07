# 04. 컬렉션 (Collection)

<br>

## 1. 왜 컬렉션이 필요한가?

배열(`array`)만으로 데이터를 다루면 여러 한계가 있어요.

```java
// 배열의 한계
String[] names = new String[3]; // 크기를 미리 정해야 함
names[0] = "Alice";
names[1] = "Bob";
// 중간에 삽입? 삭제? → 직접 이동 로직 작성해야 함
// 크기가 부족하면? → 새 배열 만들고 복사해야 함
```

**배열의 문제점:**
- 크기가 고정 → 동적으로 늘리고 줄이기 불편
- 삽입/삭제 시 직접 인덱스 관리 필요
- 중복 제거, 정렬, 검색 등 기능이 없음

Java 컬렉션 프레임워크는 이런 문제를 해결하는 **자료구조 라이브러리**예요.

<br>

## 2. 어떻게 동작하는가?

### 컬렉션 계층 구조

```
Collection
├── List     → 순서 O, 중복 O
│   ├── ArrayList
│   └── LinkedList
├── Set      → 순서 X, 중복 X
│   ├── HashSet
│   ├── LinkedHashSet
│   └── TreeSet
└── Queue    → FIFO
    └── LinkedList

Map (Collection과 별도)
├── HashMap
├── LinkedHashMap
└── TreeMap
```

---

### 📋 List

**순서가 있고 중복을 허용하는 컬렉션**

#### ArrayList vs LinkedList

```java
// ArrayList — 내부적으로 배열 기반
List<String> arrayList = new ArrayList<>();
arrayList.add("A");       // O(1) — 끝에 추가
arrayList.get(2);         // O(1) — 인덱스로 바로 접근
arrayList.add(0, "X");    // O(n) — 앞에 삽입 시 뒤 요소 전부 이동

// LinkedList — 내부적으로 이중 연결 리스트 기반
List<String> linkedList = new LinkedList<>();
linkedList.add("A");      // O(1)
linkedList.get(2);        // O(n) — 처음부터 순회해야 함
linkedList.add(0, "X");   // O(1) — 포인터만 변경하면 됨
```

| 연산 | ArrayList | LinkedList |
|------|-----------|------------|
| get(index) | **O(1)** | O(n) |
| add(끝) | O(1) | O(1) |
| add(중간) | O(n) | **O(1)** |
| remove(중간) | O(n) | **O(1)** |
| 메모리 | 연속 공간 | 노드마다 포인터 추가 |

> **실무에서는 ArrayList를 훨씬 많이 써요.** 조회가 삽입/삭제보다 압도적으로 많기 때문이에요.

---

### 🔵 Set

**중복을 허용하지 않는 컬렉션**

```java
Set<String> set = new HashSet<>();
set.add("apple");
set.add("banana");
set.add("apple");   // 무시됨
System.out.println(set.size()); // 2

// 중복 제거에 자주 활용
List<String> listWithDuplicates = List.of("a", "b", "a", "c", "b");
Set<String> unique = new HashSet<>(listWithDuplicates);
// → {a, b, c}
```

#### HashSet vs LinkedHashSet vs TreeSet

```java
// HashSet — 순서 보장 없음, 가장 빠름 O(1)
Set<String> hashSet = new HashSet<>();

// LinkedHashSet — 입력 순서 보장
Set<String> linkedSet = new LinkedHashSet<>();
linkedSet.add("banana");
linkedSet.add("apple");
// 순회 시: banana → apple (입력 순서 유지)

// TreeSet — 정렬 순서 보장, O(log n)
Set<String> treeSet = new TreeSet<>();
treeSet.add("banana");
treeSet.add("apple");
// 순회 시: apple → banana (알파벳 순)
```

---

### 🗺 Map

**키-값 쌍으로 데이터를 저장 (키는 중복 불가)**

```java
Map<String, Integer> map = new HashMap<>();
map.put("apple", 3);
map.put("banana", 5);
map.put("apple", 10);   // 기존 값 덮어씀

map.get("apple");       // 10
map.containsKey("banana"); // true
map.getOrDefault("cherry", 0); // 없으면 기본값 반환

// 순회
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

#### HashMap vs LinkedHashMap vs TreeMap

| | HashMap | LinkedHashMap | TreeMap |
|--|---------|---------------|---------|
| 순서 | 없음 | 입력 순서 유지 | 키 정렬 |
| 속도 | **O(1)** | O(1) | O(log n) |
| null 키 | 허용 | 허용 | 불가 |

<br>

## 3. 다른 방법과 무엇이 다른가?

### HashMap 내부 동작 — 해시 충돌

```
put("apple", 10) 호출 시:

1. "apple".hashCode() 계산 → 해시값
2. 해시값 % 배열크기 → 버킷 인덱스 결정
3. 해당 인덱스에 저장

충돌 발생 시 (다른 키가 같은 인덱스):
Java 8 이전 → LinkedList로 체이닝
Java 8 이후 → 8개 초과 시 Red-Black Tree로 전환 (O(n) → O(log n))
```

---

### ArrayList 내부 동작 — 동적 크기 확장

```
초기 용량: 10
요소 추가 시 용량 초과 → 기존 배열의 1.5배 크기로 새 배열 생성 후 복사

→ 대량 데이터 미리 알면 new ArrayList<>(예상크기) 로 초기 용량 지정 권장
```

---

### List.of() vs new ArrayList()

```java
// List.of() — 불변 리스트 (수정 불가)
List<String> immutable = List.of("a", "b", "c");
immutable.add("d"); // UnsupportedOperationException!

// new ArrayList() — 가변 리스트
List<String> mutable = new ArrayList<>(List.of("a", "b", "c"));
mutable.add("d"); // OK

// 언제 쓰나?
// List.of() → 변경 없는 상수 데이터, 메서드 인자로 넘길 때
// new ArrayList() → 이후에 추가/삭제가 필요한 경우
```

<br>

## 4. 실제 코드에서는 어떻게 쓰는가?

### Spring 서비스에서 자주 쓰는 패턴

```java
@Service
public class UserService {

    // 1. 중복 제거
    public List<String> getUniqueTags(List<Post> posts) {
        return posts.stream()
                .flatMap(p -> p.getTags().stream())
                .distinct()
                .collect(Collectors.toList());
    }

    // 2. 그룹핑 (Map 활용)
    public Map<String, List<User>> groupByDepartment(List<User> users) {
        return users.stream()
                .collect(Collectors.groupingBy(User::getDepartment));
    }

    // 3. 빠른 조회를 위한 Map 변환
    public Map<Long, User> getUserMap(List<User> users) {
        return users.stream()
                .collect(Collectors.toMap(User::getId, u -> u));
        // 이후 userMap.get(id) → O(1) 조회
    }
}
```

---

### N+1 문제 해결 시 Map 활용

```java
// N+1 문제: 유저 100명의 주문 각각 조회 → DB 쿼리 101번
for (User user : users) {
    List<Order> orders = orderRepository.findByUserId(user.getId()); // 매번 쿼리
}

// 해결: 한 번에 조회 후 Map으로 변환
List<Order> allOrders = orderRepository.findByUserIdIn(userIds); // 쿼리 1번

Map<Long, List<Order>> orderMap = allOrders.stream()
        .collect(Collectors.groupingBy(Order::getUserId));

for (User user : users) {
    List<Order> orders = orderMap.getOrDefault(user.getId(), List.of()); // O(1)
}
```

<br>

## 5. 면접 Q&A

**Q. ArrayList와 LinkedList의 차이를 설명해주세요.**

> ArrayList는 내부적으로 배열을 사용해 인덱스 기반 조회가 O(1)로 빠르지만, 중간 삽입/삭제 시 요소를 이동해야 해 O(n)이 걸립니다. LinkedList는 이중 연결 리스트로 중간 삽입/삭제가 O(1)이지만, 특정 인덱스 조회 시 처음부터 순회해 O(n)이 걸립니다. 실무에서는 조회가 많은 경우 ArrayList를, 큐/덱처럼 앞뒤 삽입/삭제가 빈번한 경우 LinkedList를 선택합니다.

---

**Q. HashMap의 내부 동작 원리를 설명해주세요.**

> HashMap은 키의 `hashCode()`를 계산해 배열의 인덱스(버킷)를 결정하고 값을 저장합니다. 서로 다른 키가 같은 인덱스를 가리키는 해시 충돌이 발생하면, Java 8 이전에는 LinkedList로 체이닝했고, Java 8부터는 같은 버킷에 8개 이상 쌓이면 Red-Black Tree로 전환해 최악의 경우 O(n)에서 O(log n)으로 개선됩니다.

---

**Q. HashSet은 어떻게 중복을 판단하나요?**

> HashSet은 내부적으로 HashMap을 사용합니다. `add(element)` 시 먼저 `hashCode()`로 버킷을 찾고, 같은 버킷 안에서 `equals()`로 동등성을 비교합니다. 두 조건이 모두 같으면 중복으로 판단해 저장하지 않습니다. 따라서 커스텀 객체를 Set에 넣으려면 `hashCode()`와 `equals()`를 반드시 함께 오버라이드해야 합니다.

---

**Q. HashMap과 HashTable, ConcurrentHashMap의 차이는 무엇인가요?**

> HashMap은 동기화를 지원하지 않아 멀티스레드 환경에서 안전하지 않지만 가장 빠릅니다. HashTable은 모든 메서드에 `synchronized`가 걸려 스레드 안전하지만 성능이 낮고, null 키/값을 허용하지 않습니다. ConcurrentHashMap은 Java 5에서 도입된 현대적 대안으로, 전체 잠금 대신 버킷 단위로 잠금을 걸어 멀티스레드 환경에서 성능과 안전성을 모두 확보합니다. 실무에서 멀티스레드 환경이라면 ConcurrentHashMap을 선택합니다.
