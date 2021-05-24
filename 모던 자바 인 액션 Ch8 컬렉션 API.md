# 모던 자바 인 액션 - Java 8,9,10 가이드

## Part 3. 스트림과 람다를 이용한 효과적 프로그래밍



## Chapter 8. 컬렉션 API

> 이 장의 목표
>
> 1. 컬렉션 팩토리
> 2. 리스트 및 집합에서 요소를 삭제하거나 바꾸는 관용 패턴 배우기
> 3. 맵 작업



### 8.1. 컬렉션 팩토리

> **컬렉션 리터럴**
>
> 변경할 수 없는 컬렉션



보통 리스트 만들기를 할때 다음과 같다.

```java
List<String> friends = new ArrayList<>();
friends.add("Raphael");
friends.add("Olivia");
friends.add("Thibaut");
```



#### 8.1.1. 리스트 팩토리

> List.of

```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
```



```java
List<String> friends = List.of("Raphael", "Olivia", "Thibaut");
friends.add("Chih-Chun"); // java.lang.UnsupportedOperationException 에러 발생
```

**List.of의 내부**

![image-20210524204809829](/Users/we/Library/Application Support/typora-user-images/image-20210524204809829.png)

최대 10개 파라미터까지 이렇게 정의한다.

아래와 같이 가변 인수로 받을 수도 있지만 배열을 할당하고 가비지 컬렉션을 하는 비용을 지불해야 하기 때문이다.

![image-20210524205005843](/Users/we/Library/Application Support/typora-user-images/image-20210524205005843.png)



#### 8.1.2. 집합 팩토리

바꿀 수 없는 집합 만들기

```java
Set<String> friends = Set.of("Raphael", "Olivia", "Thibaut");
```



#### 8.1.3. 맵 팩토리

> 키와 값 번갈아 쓰기

```java
Map<String, Integer> ageOfFriends
   = Map.of("Raphael", 30, "Olivia", 25, "Thibaut", 26);
```

> Map.ofEntries 사용

Map.entry는 Map.Entry 객체를 만드는 새로운 팩토리 메서드

```java
import static java.util.Map.entry;
Map<String, Integer> ageOfFriends
       = Map.ofEntries(entry("Raphael", 30),
                       entry("Olivia", 25),
                       entry("Thibaut", 26));
```



### 8.2. 리스트와 집합 처리

#### 8.2.1. removeIf

predicate를 만족하는 요소 제거

```java
transactions.removeIf(transaction ->
     Character.isDigit(transaction.getReferenceCode().charAt(0)));
```

#### 8.2.2. replaceAll

리스트의 각 요소를 새로운 요소로 바꾸기

```java
referenceCodes.replaceAll(code -> Character.toUpperCase(code.charAt(0)) +
     code.substring(1));
```



### 8.3. 맵 처리

#### 8.3.1. forEach 메서드

```java
for(Map.Entry<String, Integer> entry: ageOfFriends.entrySet()) {
   String friend = entry.getKey();
   Integer age = entry.getValue();
   System.out.println(friend + " is " + age + " years old");
}
```



```java
ageOfFriends.forEach((friend, age) -> System.out.println(friend + " is " +
     age + " years old"));
```



#### 8.3.2. 정렬 메서드

```java
Map<String, String> favouriteMovies
       = Map.ofEntries(entry("Raphael", "Star Wars"),
       entry("Cristina", "Matrix"),
       entry("Olivia",
       "James Bond"));

favouriteMovies
  .entrySet()
  .stream()
  .sorted(Entry.comparingByKey()) // Entry.comparingByValue도 가능
```



#### 8.3.3. getOrDefault 메서드

NullPointerException 방지

```java
Map<String, String> favouriteMovies
       = Map.ofEntries(entry("Raphael", "Star Wars"),
       entry("Olivia", "James Bond"));

System.out.println(favouriteMovies.getOrDefault("Olivia", "Matrix")); // "James Bond"
System.out.println(favouriteMovies.getOrDefault("Thibaut", "Matrix")); // "Matrix"
```

#### 8.3.4. 계산 패턴

ex. 캐싱할때 사용

> computeIfAbsent

제공된 키에 해당하는 값이 없으면(empty or null) 키를 이용해 새 값을 계산하고 맵에 추가

```java
lines.forEach(line ->
   dataToHash.computeIfAbsent(line,
                              this::calculateDigest));
```

#### 8.3.5. 삭제 패턴

```java
favouriteMovies.remove(key, value);
```

#### 8.3.6. 교체 패턴

> replaceAll

```java
favouriteMovies.replaceAll((friend, movie) -> movie.toUpperCase());
```

#### 8.3.7. Merge

> putAll

```java
Map<String, String> family = Map.ofEntries(
   entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
   entry("Raphael", "Star Wars"));

Map<String, String> everyone = new HashMap<>(family);

everyone.putAll(friends);
System.out.println(everyone); // {Cristina=James Bond, Raphael=Star Wars, Teo=Star Wars}
```

> merge

```java
Map<String, String> family = Map.ofEntries(
    entry("Teo", "Star Wars"), entry("Cristina", "James Bond"));
Map<String, String> friends = Map.ofEntries(
    entry("Raphael", "Star Wars"), entry("Cristina", "Matrix"));

Map<String, String> everyone = new HashMap<>(family);
friends.forEach((k, v) ->
   everyone.merge(k, v, (movie1, movie2) -> movie1 + " & " + movie2)); // 중복된 키가 있으면 두 값을 연결
System.out.println(everyone); // Outputs {Raphael=Star Wars, Cristina=James Bond & Matrix, Teo=Star Wars}

```



merge로 초기화 검사

```java
Map<String, Long> moviesToCount = new HashMap<>();
String movieName = "JamesBond";
long count = moviesToCount.get(movieName);
if(count == null) {
   moviesToCount.put(movieName, 1);
}
else {
   moviesToCount.put(moviename, count + 1);
}
```

위 코드를 아래와 같이 구현할 수 있다.

```java
moviesToCount.merge(movieName, 1L, (key, count) -> count + 1L); // 3번째 인자: 중복된 key가 있으면 count + 1, 없으면 1
```



### 8.4. 개선된 ConcurrentHashMap

동시성 + HashMap -> 동시 추가, 갱신 작업 허용 (스레드 안정성 제공)

#### 8.4.1. 리듀스와 검색

- forEach: 각 (키, 값) 쌍에 주어진 액션을 실행
- reduce: 모든 (키, 값) 쌍을 제공된 리듀스 함수를 이용해 결과로 합침
- search: null이 아닌 값을 반환할 때까지 각 (키, 값) 쌍에 함수를 적용



- 키, 값으로 연산 (forEach, reduce, search)
- 키로 연산 (forEachKey, reduceKeys, searchKeys)
- 값으로 연산 (forEachValue, reduceValues, searchValues)
- Map.Entry 객체로 연산 (forEachEntry, reduceEntries, search-Entries)

⚠️ 위 연산은 ConcurrentHashMap의 상태를 잠그지 않고 연산을 수행하기 때문에 이 연산에 제공되는 함수는 계산이 진행되는 동안 바뀔 수 있는 객체, 값, 순서 등에 의존하지 않아야 한다.



위의 연산에 병렬성 기준값을 지정해야 한다.

- threshold=1: 공통 스레드 풀을 이용해 병렬성 극대화
- threshold=Long.MAX_VALUE: 한 개의 스레드로 연산 실행

```java
ConcurrentHashMap<String, Long> map = new ConcurrentHashMap<>();
long parallelismThreshold = 1;
Optional<Integer> maxValue =
   Optional.ofNullable(map.reduceValues(parallelismThreshold, Long::max)); //맵의 최대값
```



#### 8.4.2. 계수

맵의 매핑 개수를 반환하는 mapppingCount 메서드 제공



#### 8.4.3. 집합뷰

집합 뷰로 반환하는 keySet 메서드

![스크린샷 2021-05-24 오후 10.00.34](/Users/we/Desktop/스크린샷 2021-05-24 오후 10.00.34.png)