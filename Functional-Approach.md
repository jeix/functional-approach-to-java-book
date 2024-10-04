# A Functional Approach to Java (2023)


# PART Ⅰ 함수형 기초

## CHAPTER 2 함수형 자바

### 2.1 자바 람다란?
### 2.1.1 람다 문법
### 2.1.2 함수형 인터페이스

- 람다 표현은 함수형 인터페이스의 하위 타입

- ```
  java.util.function.Predicate<T>
  . test(t : T) : boolean -- 단일 추상 메서드
  . and, or, negate       -- 디폴트 메서드
  . isEqual, not          -- 정적 메서드
  ```

### 2.1.3 람다와 외부 변수
### 2.1.4 익명 클래스는 무엇인가?

- - 익명 내부 클래스는 자체 스코프를 생성
  - 람다는 자신이 속한 스코프 내에 존재

### 2.2 람다의 실전 활용
### 2.2.1 람다 생성
### 2.2.2 람다 호출
### 2.2.3 메서드 참조

```java
////////////////////////////////////////////////////////////
// [ex-2-8] 메서드 참조와 스트림

// 람다
customers.stream()
         .filter(customer -> customer.isActive())
         .map(customer -> customer.getName())
         .map(name -> name.toUpperCase())
         .peek(name -> System.out.println(name))
         .toArray(count -> new String[count]);

// 메서드 참조
customers.stream()
         .filter(Customer::isActive)
         .map(Customer::getName)
         .map(String::toUpperCase) 
         .peek(System.out::println)
         .toArray(String[]::new);
```

```java
////////////////////////////////////////////////////////////
// method-references/Static.java

// 람다
Function<Integer, String> asLambda = i -> Integer.toHexString(i);

// 정적 메서드 참조
Function<Integer, String> asRef = Integer::tohexString;
```

```java
////////////////////////////////////////////////////////////
// method-references/BoundNonStatic.java

var now = LocalDate.now();

// 기존 객체를 기반으로 한 람다
Predicate<LocalDate> isAfterNowAsLambda = $ -> $.isAfter(now);

// 바운드 비정적 메서드 참조
Predicate<LocalDate> isAfterNowAsRef = now::isAfter;

// 반환값 바인딩
Predicate<LocalDate> isAfterNowAsRef = LocalDate.now()::isAfter;

// 정적 필드 바인딩
Function<Object, String> castToStr = String.class::cast;
```

```java
////////////////////////////////////////////////////////////
// method-references/BoundNonStaticThisSuper.java

// this
Function<String, String> thisWorker = this::doWork;

// super
Function<String, String> superWorker = SubClass.super::doWork;
```

```java
////////////////////////////////////////////////////////////
// method-references/UnboundNonStatic.java

// 람다
Function<String, String> toLowerCaseLambda = str -> str.toLowerCase();

// 언바운드 비정적 메서드 참조
Function<String, String> toLowerCaseRef = String::toLowerCase;
```

```java
////////////////////////////////////////////////////////////
// method-references/Constructor.java

// 람다
Function<String, Locale> newLocaleLambda = language -> new Locale(language);

// 생성자 참조
Function<String, Locale> newLocaleRef = Locale::new;
```

### 2.3 자바의 함수형 프로그래밍 개념
### 2.3.1 순수 함수와 참조 투명성

- Ehcache
- `@Cacheable`

```java
////////////////////////////////////////////////////////////
// [ex-2-9] Map#computeIfAbsent 로 구현한 메모이제이션

Map<String, Object> CACHE = new HashMap<>();

<T> T memoize(String identifier, Supplier<T> fn) {
    return (T) CACHE.computeIfAbsent(identifier, key -> fn.get());
}

Integer expensiveCall(String arg0, int arg1) {
    // ...
}

Integer memoizedCall(String arg0, int arg1) {
    var compoundKey = String.format("expensiveCall:%s-%d", arg0, arg1);
    return memoize(compoundKey, () -> expensiveCall(arg0, arg1));
}

var calculated = memoizedCall("hello, world!", 42);

var memoized = memoizedCall("hello, world!", 42);
```

- TTL, time-to-live

### 2.3.2 불변성
### 2.3.3 일급 객체
### 2.3.4 함수 합성

## CHAPTER 3 JDK의 함수형 인터페이스

### 3.1 네 가지 함수형 인터페이스

### 3.1.1 Function

- ```
  java.util.function.Function<T,R>
  . apply(t : T) : R
  ```

### 3.1.2 Consumer

- ```
  java.util.function.Consumer<T>
  . accept(t : T) : void
  ```

### 3.1.3 Supplier

- ```
  java.util.function.Supplier<T>
  . get() : T
  ```

### 3.1.4 Predicate

- ```
  java.util.function.Predicate<T>
  . test(t : T) : boolean
  ```

### 3.2 함수형 인터페이스 변형이 많은 이유
### 3.2.1 함수 아리티

- ```
  BiFunction<T,U,R> -- Function<T,R>
  BiConsumer<T,U>   -- Consumer<T>
  BiPredicate<T,U>  -- Predicate<T>
  ```

- ```
  UnaryOperaator<T> : Function<T,T>
  BinaryOperator<T> : Function<T,T,T>
  ```

- 타입호환성 vs 간결성

### 3.2.2 원시 타입

- ```
  Function
    . IntFunction<R> : Function<Integer, R>
    . IntUnary0perator : UnaryOperator<Integer>
    . IntBinary0perator : BinaryOperator<Integer>
    . ToIntFunction<T> : Function<T, Integer>
    . ToIntBiFunction<T, U> : BiFunction<T, U, Integer>
    . IntToDoubleFunction : Function<Integer, Double>
    . IntToLongFunction : Function<Integer, Long>

  Consumer
    . IntConsumer : Consumer<Integer>
    . ObjIntConsumer<T> : BiConsumer<T, Integer>

  Supplier
    . IntSupplier : Supplier<Integer>

  Predicate
    . IntPredicate : Predicate<Integer>
  ```

- ```
  Supplier
    . BooleanSupplier
  ```

### 3.2.3 함수형 인터페이스 브리징

```java
////////////////////////////////////////////////////////////
// bridging-functional-interfaces/BridgingFunctionalInterfaces.java

interface LikePredicate<T> {
    boolean test(T value);
}

LikePredicate<String> isNull = str -> str == null;

Predicate<String> wontCompile = isNull; // Error
Predicate<String> wontCompileEither = (Predicate<String>) isNull; // Error

Predicate<String> thisIsFine = isNull::test;
```

### 3.3 함수 합성

- ```
  Function<T,R>
  . compose(before : Function<? super V, ? extends T>) : Function<V,R>
  . andThen(after : Function<? super R, ? extends V>) : Function<T,V>
  ```

```java
////////////////////////////////////////////////////////////
// [ex-3-2] 함수 합성 방향

Function<String, String> removeLowerCaseA = str -> str.replace("a", "");
Function<String, String> upperCase = String::toUpperCase;

var input = "abcd";

String leftToRight = removeLowerCaseA.andThen(upperCase).apply(input);

String rightToLeft = upperCase.compose(removeLowerCaseA).apply(input);
```

- ```
  Predicate<T>
  . and
  . or
  . negate
  ```

- ```
  Consumer<T>
  . andThen
  ```

### 3.4 함수형 지원 확장
### 3.4.1 기본 메서드 추가
### 3.4.2 함수형 인터페이스 명시적으로 구현하기

```java
public interface TextEditorCommand extends Supplier<T> {
    String execute();

    default String get() {
        return execute();
    }
}
```

### 3.4.3 정적 헬퍼 생성하기

- ```
  compose(before : Supplier<T>, fn : Function<T, R>) : Supplier<R>
  compose(fn : Function<T, R>, after : Consumer<R>) : Consumer<T>
  ```

```java
////////////////////////////////////////////////////////////
// [ex-3-5] 함수형 컴포지터

public final class Compositor {

    public static <T, R> Supplier<R> compose(Supplier<T> before, Function<T, R> fn) {
        Objects.requireNonNull(before);
        Objects.requireNonNull(fn);

        return () -> {
            T result = before.get();
            return fn.apply(result);
        };
    }

    public static <T, R> Consumer<T> compose(Function<T, R> fn, Consumer<R> after) {
        Objects.requireNonNull(fn);
        Objects.requireNonNull(after);

        return (T t) -> {
            R result = fn.apply(t);
            after.accept(result);
        };
    }

    public static <T> Consumer<T> acceptIf(Predicate<T> predicate, Consumer<T> consumer) {
        Objects.requireNonNull(predicate);
        Objects.requireNonNull(consumer);

        return (T t) -> {
            if (!predicate.test(t)) {
                return;
            }
            consumer.accept(t);
        };
    }

    private Compositor() {
        // disallows direct instantiation
    }
}

////////////////////////////////////////////////////////////
// [ex-3-6] 함수형 컴포지터 활용

Function<String, String> removeLowerCaseA = str -> str.replace("a", "");
Function<String, String> upperCase = String::toUpperCase;

Function<String, String> stringOperations = removeLowerCaseA.andThen(upperCase);

Consumer<String> composedConsumer = Compositor.compose(stringOperations, System.out::println);
```


# PART Ⅱ 함수형 접근 방식


## CHAPTER 4 불변성

### 4.1 객체 지향 프로그래밍의 가변성과 자료 구조

### 4.2 함수형 프로그래밍의 불변성

### 4.3 자바 불변성 상태
### 4.3.1 java.lang.String
### 4.3.2 불변 컬렉션

### = unmodifiable collection

- ```
  java.util.Collections
  . unmodifiableCollection(Collection<? extends T c) : Collection<T>
  . unmodifiableSet(Set<? extends T> s) : Set<T>
  . unmodifiableList(List<? extends T> list) : List<T>
  . unmodifiableMap(Map<? extends K, ? extends V m) : Map<K,V>
  . unmodifiableSortedSet(SortedSet<T> s) : SortedSet<T>
  . unmodifiableSortedMap(SortedMap<K, ? extends V m) : SortedMap<K,V>
  . unmodifiableNavigableSet(NavigableSet<T> s) : NavigableSet<T>
  . unmodifiableNavigableMap(NavigableMap<K, V> m) : NavigableMap<K,V>
  ```

### = 불변 컬렉션 팩토리 메서드

- `List.of(E e1, ...) : List<E>`
- `Set.of(E e1, ...) : Set<E>`
- `Map.of(K k1, V v1, ... ) : Map<K, V>`

### = immutable copy

- `Set.copyOf(Collection<? extends E> coll) : Set<E>`
- `List.copyOf(Collection<? extends E> coll) : List<E>`
- `Map.copyOf(Ma<? extends K, ? extends V> map) : Map<K, V>`

### 4.3.3 원시 타입과 원시 래퍼
### 4.3.4 불변 수학
### 4.3.5 자바 시간 API (JSR-310)
### 4.3.6 enum
### 4.3.7 final 키워드
### 4.3.8 레코드

### 4.4 불변성 만들기
### 4.4.1 일반적인 관행


## CHAPTER 5 레코드

### 5.1 데이터 집계 유형
### 5.1.1 튜플
### 5.1.2 간단한 POJO

- `Objects.hash(...) : int`
- `Objects.equals(x, y) : boolean`

### 5.1.3 불변 POJO 만들기
### 5.1.4 POJO를 레코드로 만들기

### 5.2 도움을 주기 위한 레코드
### 5.2.1 내부 동작
### 5.2.2 레코드의 특징

### = 사용자 정의 생성자

- `Objects.requireNonNull(x) : T`

```java
////////////////////////////////////////////////////////////
// record-features/CompactConstructor.java

public record User(String username,
                   boolean active,
                   LocalDateTime lastLogin) {
    public User {
        Objects.requireNonNull (username);
        Objects.requireNonNull(lastLogin);
        username = username. tolowerCase();
    }
}
```

### = 리플렉션

- ```
  Class
  . getRecordComponents() : java.lang.reflect.RecordComponent[]
  ```

### 5.2.3 누락된 기능

### = 컴포넌트의 기본값과 컴팩트 생성자

- - 생성자 오버로드
  - 스태틱 팩토리 메서드
  - 스태틱 변수

```java
////////////////////////////////////////////////////////////
// (154)

// 인수가 없는 생성자는 상수를 활용하는 것이 적절
// 레코드는 불변하니까 단일 인스턴스면 됨

public record Origin(int x, int y) {
    public static Origin ZERO = new Origin(0, 0);
}
```

### = 단계별 생성

- 빌더 패턴

```java
////////////////////////////////////////////////////////////
// [ex-5-8] 내부 빌더

public record User(String username, boolean active, LocalDateTime lastLogin) {

    // NO BODY
    public static final class Builder {

        private final String username;

        private boolean active;
        private LocalDateTime lastLogin;

        public Builder(String username) {
            this.username = username;
            this.active = true;
        }

        public Builder active(boolean isActive) {
            if (this.active == false) {
                throw new IllegalArgumentException("...");
            }
            this.active = isActive;
            return this;
        }

        public Builder lastLogin(LocalDateTime lastLogin) {
            this.lastLogin = lastLogin;
            return this;
        }

        public User build() {
            return new User(this.username, this.active, this.lastLogin);
        }
    }
}
```

### 5.3 사용 사례와 일반적인 관행
### 5.3.1 레코드 유효성 검사 및 데이터 정제
### 5.3.2 불변성 강화

- `Collections.unmodifiableList(List<T>) : List`

- `List.copyOf(Collection) : List`

### 5.3.3 변형된 복사본 생성

### = wither 메서드

```java
////////////////////////////////////////////////////////////
// record-wither/WitherNested.java

public record Point(int x, int y) {
    public With with() {
        return new With(this);
    }
    public record With(Point source) {
        public Point x(int x) {
            return new Point(x, source.y());
        }
        public Point y(int y) {
            return new Point (source.(), y);
        }
    }
}
```

### = 빌더 패턴

```java
////////////////////////////////////////////////////////////
// record-use-cases-common-practices/BuilderPatternCopy.java

public record Point(int x, int y) {
    public static final class Builder {
        private int x;
        private int y;
        public Builder(Point point) {
            this.x = point.x();
            this.y = point.y();
        }
        public Builder (int ×) {
            this.x = x;
            return this;
        }
        public Builder y(int y) {
            this.y = y;
            return this;
        }
        public Point build() {
            return new Point(this.x, this.y);
        }
    }
}
```

### 5.3.4 명목상 튜플로서의 로컬 레코드

```java
////////////////////////////////////////////////////////////
// (167)

public List<String> filterAlbums(Map Integer, List<String> albums, int minimumYear) {
    return albums.entrySet()
                 .stream()
                 .filter(entry -> entry.getKey() >= minimumYear)
                 .sorted(Comparator.comparing(Map.Entry::getKey))
                 .map(Map.Entry::getValue)
                 .flatMap(List::stream)
                 .tolist();
}
```

```java
////////////////////////////////////////////////////////////
// [ex-5-9] 로컬 레코드를 사용한 스트림 파이프라인

private static List<String> filterAlbums(Map<Integer, List<String>> albums, int minimumYear) {

    record AlbumsPerYear(int year, List<String> titles) {

        AlbumsPerYear(Map.Entry<Integer, List<String>> entry) {
            this(entry.getKey(), entry.getValue());
        }

        static Predicate<AlbumsPerYear> minimumYear(int year) {
            return albumsPerYear -> albumsPerYear.year() >= year;
        }

        static Comparator<AlbumsPerYear> sortByYear() {
            return Comparator.comparing(AlbumsPerYear::year);
        }
    }

    return albums.entrySet()
                 .stream()
                 .map(AlbumsPerYear::new)
                 .filter(AlbumsPerYear.minimumYear(minimumYear))
                 .sorted(AlbumsPerYear.sortByYear())
                 .map(AlbumsPerYear::titles)
                 .flatMap(List::stream)
                 .toList();
}
```

### 5.3.5 Optional 데이터 처리

- `Objects.requireNonNull(x, msg) : T`

- `Optional.ofNullable(x) : Optional<T>`

### 5.3.6 레코드의 진화된 직렬화
### 5.3.7 레코드 패턴 매칭 (자바 19+)

```java
////////////////////////////////////////////////////////////
// (178)

// 이전 방식
if (obi instanceof String) {
    String str = (String) obj;
    ...
}

// 자바 16+
if (obj instanceof String str) {
    ...
}
```

```java
////////////////////////////////////////////////////////////
// (178)

// 스위치 패턴 매칭이 없는 경우
String formatted = "Unknown";
if (obi instanceof Integer i) {
    formatted = String.format("int %d", i);
} else if (obj instanceof Long l) {
    formatted = String.format("long %d", l);
} else if (obj instanceof String str) {
    formatted = String.format("String %s", str);
}

// 스위치 패턴 매칭을 적용한 경우
String formatted = switch (obj) {
    case Integer i -> String.format("int %d", i);
    case Long l -> String.format("long %d", l);
    case String s -> String.format("String %s", s);
    default -> "unknown";
};
```

```java
////////////////////////////////////////////////////////////
// (179)

record Point(int x, int y) {}

var point = new Point(23, 42);

if (point instanceof Point(int x, int y)) {
    System.out.println(x + y);
        // => 65
}

int result = switch (anyObject) {
    case Point(var x, var y) -> x + y;
    case Point3D(var x, var y, var z) -> x + y + z;
    default -> 0.0;
};
```

### 5.4 레코드를 마무리하며



## CHAPTER 6 스트림을 이용한 데이터 처리

### 6.1 반복을 통한 데이터 처리
### 6.1.1 외부 반복
### 6.1.2 내부 반복

### 6.2 함수형 데이터 파이프라인으로써의 스트림

```java
////////////////////////////////////////////////////////////
// [ex-6-2] 스트림을 활용한 책 찾기

List<Book> books = ...

List<String> result = books.stream()
                           .filter(book -> book.year() < 1970)
                           .filter(book -> book.genre() == Genre.SCIENCE_FICTION)
                           .map(Book::title)
                           .sorted()
                           .limit(3)
                           .collect(Collectors.toList());
```

### 6.2.1 스트림의 특성
### 6.2.2 스트림의 핵심, Spliterator

- `Spliterator<T>`

### 6.3 스트림 파이프라인 구축하기
### 6.3.1 스트림 생성하기
### 6.3.2 실습하기

### = 요소 선택

- ```
  Predicate.not(Predicate<T>)
  ```

- ```
  filter(Predicate<T>) : Stream
  dropWhile(Predicate<T>) : Stream
  takeWhile(Predicate<T>) : Stream
  limit(long) : Stream<T>
  skip(long) : Stream<T>
  distinct() : Stream<T>
  sorted() : Stream<T> -- Comparable or Comparator<T>
  ```

### = 요소 매핑

- ```
  map(Function<? super T, ? extends R>) : Stream<R>
  flatMap(Function<? super T, ? extends Stream<? extends R>>) : Stream<R>
  mapMulti(BiConsumer<? super T, ? super Consumer<R>>) : Stream
  ```

### = peek

- ```
  peek(Consumer<? super T>) : Stream
  ```

### 6.3.3 스트림 종료하기

### = 요소 축소

- ```
  reduce(T init, BinaryOperator<T> accum) : T
  reduce(BinaryOperator<T> accum) : Optional<T>
  ```

```java
////////////////////////////////////////////////////////////
// [ex-6-11] 인수 3개 짜리 reduce 對 인수 2개 짜리 reduce (+ map)

Integer reduceOnly = Stream.of("apple", "orange", "banana")
                            .reduce(0,
                                    (acc, str) -> acc + str.length(),
                                    Integer::sum);

int mapReduce = Stream.of("apple", "orange", "banana")
                      .mapToInt(String::length)
                      .reduce(0, (acc, length) -> acc + length);
```

### = 컬렉터를 활용한 요소 집계

- ```
  Collectors

  . collect(Collector<T,A,R>)

  . toCollection(collectionFactory : Supplier<C>)
  . toList()
  . toSet()
  . toUnmodifiableList()
  . toUnmodifiableSet()

  . toMap(…)
  . toConcurrentMap(…)
  . toUnmodifiableMap(…)

  . groupingBy(...)
  . groupingByConcurrent(…)

  . partitionBy(...)

  . averafingInt(mapper : ToIntFunction<? super T>)
  . summingInt(mapper : ToIntFunction<? super T>)
  . summarizingInt(mapper : ToIntFunction<? super T>)
  . counting()
  . minBy(Comparator<? super T>)
  . maxBy(Comparator<? super T>)

  . joining(...)

  . reducing(...)
  . collectingAndThen(downstream : Collector<T,A,R), finisher : Function<R,RR>)
  . mapping(mapper Function<? super T, ? extends U>, downstream : Collector<? super U, A, R>)
  . filtering(Predicate>? super T>, downstream : Collector<? super T, A, R>)
  . teeing(downstream1 : Collector<T,?,R1>, downstream2 : Collector<T,?,R2>, nerger : BiFunction<? super R1, ? super R2, R>)
  ```

### = 요소 수집과 축소 비교

```java
////////////////////////////////////////////////////////////
// [ex-6-13] 스트림을 사용한 숫자의 불변 누적

var numbers = List.of(1, 2, 3, 4, 5, 6);

int total = numbers.stream()
                   .reduce(0,
                           Integer::sum);
```

```java
////////////////////////////////////////////////////////////
// [ex-6-14] reduce 와 collect 로 문자열 연결

// 스트림 리듀스

var reduced = strings.stream()
                    .reduce("", String::concat);

// 스트림 컬렉터 -- 사용자 정의

var joiner = strings.stream()
                    .collect(Collector.of(() -> new StringJoiner(""),
                             StringJoiner::add,
                             StringJoiner::merge,
                             StringJoiner::toString));

// 스트림 컬렉터 -- 사전 정의

var collectWithCollectors = strings.stream()
                                   .collect(Collectors.joining());
```

### = 요소를 직접 집계하기

- ```
  toList() : List<T>
  ```

```java
////////////////////////////////////////////////////////////
// (229)

String fruits = Stream.of("apple","orange","banana","peach")
                      .toArray(String[]::new);
```

### = 요소 찾기 및 매칭시키기

- ```
  findFirst : Optional<T>
  findAny : Optional<T>
  ```

- ```
  anyMatch(Predicate<? super T>) : boolean
  allMatch(Predicate<? super T>) : boolean
  noneMatch(Predicate<? super T>) : boolean
  ```

### = 요소 소비

- ```
  forEach(Consumer<? super T>) : void
  forEachOrdered(Consumer<? super T>) : void
  ```

### 6.3.4 연산 비용

```java
////////////////////////////////////////////////////////////
// [ex-6-15] 파이프라인 (naive)

Stream.of("ananas", "oranges", "apple", "pear", "banana")
      .map(String::toUpperCase)
      .sorted()
      .filter(s -> s.startsWith("A"))
      .forEach(System.out::println);
```

```java
////////////////////////////////////////////////////////////
// [ex-6-16] 개선된 파이프라인

Stream.of("ananas", "oranges", "apple", "pear", "banana")
      .filter(s -> s.startsWith("a"))
      .map(String::toUpperCase)
      .sorted()
      .forEach(System.out::println);
```

- 숏서킷 (잠재적 포함)
  - `limit`, `takeWhile`
  - `findAny`, `findFirst`, `anyMatch`, `allMatch`, `noneMatch`
  - `count`

```java
////////////////////////////////////////////////////////////
// short-circuit/Dropped.java

var result = Stream.of("apple", "orange", "banana", "melon")
                   .peek(str -> System.out.println("peek 1: " + str))
                   .map(str -> {
                       System.out.println("map: " + str);
                       return str.toUpperCase();
                   })
                   .peek(str -> System.out.println("peek 2: " + str))
                   .count();
```

```java
////////////////////////////////////////////////////////////
// short-circuit/NotDropped.java

var result = Stream.of("apple", "orange", "banana", "melon")
                   .filter(str -> str.contains("e"))
                   .peek(str -> System.out.println("peek 1: " + str))
                   .map(str -> {
                       System.out.println("map: " + str);
                       return str.toUpperCase();
                   })
                   .peek(str -> System.out.println("peek 2: " + str))
                   .count();
```

### 6.3.5 스트림 동작 변경하기

- ```
  parallel()
  sequential()
  unordered()
  onClose(handler : Runnable)
  ```

### 6.4 스트림 사용 여부 선택


## CHAPTER 7 스트림 사용하기

### 7.1 원시 스트림

- `IntStream`
- `LongStream`
- `DoubleStream`

### 7.2 반복 스트림

- ```
  iterate(seed : T, f : UnaryOperator<T>) : Stream<T>
  iterate(seed : int, f : IntUnaryOperator) : IntStream

  iterate(seed : T, hasNext : Predicate<T>, next : UnaryOperator<T>) : Stream<T>
  iterate(seed : int, hasNext : IntPredicate, next : IntUnaryOperator) : IntStream
  ```

```java
////////////////////////////////////////////////////////////
// streams-iteration/Java8.java
// streams-iteration/Java9.java

for (int idx = 1; idx < 5; idx++) {
    System.out.println(idx);
}

// Java 8

IntStream.iterate(1, idx -> idx + 1)
         .limit(4L)
         .forEachOrdered(System.out::println);

// Java 9

IntStream.iterate(1,
                  idx -> idx < 5,
                  idx -> idx + 1)
         .forEachOrdered(System.out::println);
```

- ```
  IntStream
  . range(beginInclusive : int, endExclusive : int) : IntStream
  . rangeClosed(beginInclusive : int, endInclusive : int) : IntStream
  ```

### 7.3 무한 스트림

- ```
  generate(Supplier<T>) : Stream<T>
  generate(IntSupplier) : IntStream
  ```

```java
////////////////////////////////////////////////////////////
// (246)

Stream<UUID> createStream(long count) {
    return Stream.generate(UUID::randomUUID)
                 .limit(count);
}
```

### 7.3.1 랜덤 숫자
### 7.3.2 메모리는 한정되어 있다

### 7.4 배열에서 스트림으로, 그리고 다시 배열로
### 7.4.1 객체 타입 배열

- ```
  Arrays
  . stream(T[], begin : int, endExcl : int) : Stream<T>
  ```

- ```
  Stream
  . toArray() : Object[]
  . toArray(String[]::new) : String[] -- generator : IntFunction<A[]>
  ```

### 7.4.2 원시 타입 배열

- ```
  Arrays
  . stream(int[], begin : int, endExcl : int) : IntStream
  ```

- ```
  IntStream
  . toArray() : int[]
  ```

### 7.5 저수준 스트림 생성

### 7.6 파일 I/O 사용하기
### 7.6.1 디렉토리 내용 읽기
### 7.6.2 깊이 우선 디렉토리 순회
### 7.6.3 파일 시스템 탐색하기
### 7.6.4 퍼알 헌 쥴씩 읽기
### 7.6.5 파일 I/O 스트림 사용 시 주의 사항

### = 스트림 닫기

```java
////////////////////////////////////////////////////////////
// (262)

try (Stream<String> stream = Files.lines(location)) {
    ...
}
```

### 7.7 날짜와 시간 처리하기
### 7.7.1 시간 타입 질의하기

```java
////////////////////////////////////////////////////////////
// streams-temporal-query/TemporalQuery.java

// TemporalQuery<Boolean> == Predicate<TemporalAccessor>
boolean isItTeaTime =
    LocalDateTime.now()
                 .query(temporal -> {
                     var time = LocalTime.from(temporal);
                     return time.getHour() >= 16;
                 });

// TemporalQuery<LocalTime> == Function<TemporalAccessor,Localtime>
LocalTime time = LocalDateTime.now()
                               .query(LocalTime::from);
```

### 7.7.2 LocalDate 범위 스트림

### 7.8 JMH를 활용하여 스트림 성능 측정하기

### 7.9 컬렉터 알아보기
### 7.9.1 다운스트림 컬렉터

### = 요소 변환

```java
////////////////////////////////////////////////////////////
// (268) streams-downstream-collectors/GroupBySimple.java

Map<String, List<User>> lookup =
    users.stream()
         .collect(Collectors.groupingBy(User::group));
```

```java
////////////////////////////////////////////////////////////
// (271) streams-downstream-collectors/GroupByMapping.java

var collectIdsToSet = Collectors.mapping(User::id,
                                         Collectors.toSet());

Map<String, Set<UUID>> lookup =
    users.stream()
         .collect(Collectors.groupingBy(User::group,
                                        collectIdsToSet));
```

### = 요소 축소

```java
////////////////////////////////////////////////////////////
// (272-273)

var summingUp = Collectors.reducing(0, Integer::sum);

var downstream =
        Collectors.mapping((user user) -> user.logEntries().size(), summingUp);

Map<UUID, Integer> logCountPerUserId =
        users.stream()
             .collect(Collectors.groupingBy(User::id, downstream));
```

```java
////////////////////////////////////////////////////////////
// (273) streams-downstream-collectors/ReduceSum.java

var downstream =
        Collectors.reducing(0,                                       // identity
                            (User user) -> user.logEntries().size(), // mapper
                            Integer::sum);                           // op

Map<UUID, Integer> logCountPerUserId =
        users.stream()
             .collect(Collectors.groupingBy(User::id, downstream));
```

```java
////////////////////////////////////////////////////////////
// (273-274)

var downstream =
        Collectors.summingInt((User user) -> user.logEntries().size());

Map<UUID, Integer> logCountPerUserId =
        users.stream()
             .collect(Collectors.groupingBy(User::id, downstream));
```

### = 컬렉션 평탄화

```java
////////////////////////////////////////////////////////////
// (275) streams-downstream-collectors/Flatten.java

var downstream =
        Collectors.flatMapping((User user) -> user.logEntries().stream(),
                               Collectors.toList());

Map<String, List<String>> result =
        users.stream()
             .collect(Collectors.groupingBy(User::group, downstream));
```

### = 요소 필터링

```java
////////////////////////////////////////////////////////////
// (275-276) streams-downstream-collectors/Filter.java

var startOfDay = LocalDate.now().atStartOfDay();

Predicate<User> loggedInToday =
        Predicate.not(user -> user.lastLogin().isBefore(startOfDay));

// 중간 필터 연산 사용
Map<String, Set<UUID>> todaysLoginsByGroupWithFilterOp =
        users.stream()
             .filter(loggedInToday)
             .collect(groupingBy(User::group,
                                 mapping(User::id,
                                         toSet())
             ));

// 컬렉터 필터 사용
Map<String, Set<UUID>> todaysLoginsByGroupWithFilteringCollector =
        users.stream()
             .collect(groupingBy(User::group,
                          filtering(loggedInToday,
                              mapping(User::id,
                                  toSet()
                              )
                          )
             ));
```

### = 합성 컬렉터

```java
////////////////////////////////////////////////////////////
// [ex-7-5] 로그인 날짜 최소 최대 찾기

UserStats result =
        users.stream()
             .collect(Collectors.teeing(Collectors.counting(),
                                        Collectors.filtering(user -> user.lastLogin() == null,
                                            Collectors.counting()
                                        ),
                                        UserStats::new
             ));
```

### 7.9.2 나만의 컬렉터 만들기


```java
////////////////////////////////////////////////////////////
// [ex-7-6] 문자열 요소를 연결하기 위한 사용자 정의 컬렉터

class Joinector implements Collector<CharSequence, // T
                                     StringJoiner, // A
                                     String> {     // R

    private final CharSequence delimiter;

    public Joinector(CharSequence delimiter) {
        this.delimiter = delimiter;
    }

    @Override
    public Supplier<StringJoiner> supplier() {
        return () -> new StringJoiner(this.delimiter);
    }

    @Override
    public BiConsumer<StringJoiner, CharSequence> accumulator() {
        return StringJoiner::add;
    }

    @Override
    public BinaryOperator<StringJoiner> combiner() {
        return StringJoiner::merge;
    }

    @Override
    public Function<StringJoiner, String> finisher() {
        return StringJoiner::toString;
    }

    @Override
    public Set<Characteristics> characteristics() {
        return Collections.emptySet();
    }
}
```

```java
////////////////////////////////////////////////////////////
// (283)

Collector<CharSequence, StringJoiner, String> joiner =
    Collector.of(() -> new StringJoiner(delimeter), // supplier
                 StringJoiner::add,                 // accumulator
                 StringJoiner::merge,               // combiner
                 StringJoiner::toString);           // finisher
```

### 7.10 (순차적인) 스트림에 대한 고찰


## CHAPTER 8 스트림을 활용한 병렬 데이터 처리

### 8.1 동시성 vs 병렬성

### 8.2 병렬 함수 파이프라인으로써의 스트림

### 8.3 병렬 스트림 활용

```java
////////////////////////////////////////////////////////////
// [ex-8-1] 순차 단어 집계

Map<String, Integer> wordCount =
    ...
    .collect(toMap(
        Function.identity(),
        word -> 1,
        Integer::sum
    ));
```

```java
////////////////////////////////////////////////////////////
// [ex-8-2] 병렬 단어 집계

Map<String, Integer> wordCount =
    ...
    .parallel()
    ...
    .collect(toConcurrentMap(
                 Function.identity(),
                 word -> 1,
                 Integer::sum
    ));
```

### 8.4 병렬 스트림 활용 시기와 주의할 점
### 8.4.1 적절한 데이터 소스 선택하기

- `IntStream.range`, `Arrays.stream` (primitive)
- `ArrayList`, `Arrays.stream` (object)
- `HashSet`, `TreeSet`
- `LinkedList`, `Stream.iterate`

### 8.4.2 요소의 개수

- - 작업의 수 N이 많을수록
  - 단일 작업의 비용 Q가 클수록

### 8.4.3 스트림 연산
### 8.4.4 스트림 오버헤드와 사용 가능한 자원
### 8.4.5 예시: 전쟁과 평화 (재분석)
### 8.4.6 예시: 랜덤 숫자
### 8.4.7 병렬 스트림 체크리스트


## CHAPTER 9 `Optional`을 사용한 `null` 처리

### 9.1 `null` 참조의 문제점

### 9.2 자바에서 `null`을 다루는 방법 (`Optional` 도입 전)

- JavaScript
  - `article?.authors?.[0]?.name`
  - `callback?.()`
  - `x ?? c`
- PHP
  - `$article?->author?->name`
  - `$x ?? c`
- Python
  - `x or c`
- TypeScript
  - `foo?.bar?.[0]?.baz()`

### 9.2.1 null 처리 모범 사례

### = `null` 값을 전달하거나 수용 또는 반환하지 않아야 한다

- `Objects.requireNonNull(x)`

### 9.2.2 도구를 활용한 `null` 검사
### 9.2.3 `Optional`과 같은 특별한 타입

### 9.3 `Optional` 알아보기
### 9.3.1 `Optional`이란 무엇인가?
### 9.3.2 `Optional` 파이프라인 구축하기

### = 필터링과 매핑

```java
////////////////////////////////////////////////////////////
// [ex-9-7] Optional 중간 연산 -- 권한 있는 액티브 관리자 찾기

record Permissions(List<String> permissions, Group group) {

    public boolean isEmpty() {
        return this.permissions.isEmpty();
    }
}

record Group(Optional<User> admin) { }

record User(boolean isActive) { }

User admin = new User(true);

Group group = new Group(Optional.of(admin));

Permissions permissions = new Permissions(List.of("A", "B", "C"), group);

boolean isActiveAdmin =
    Optional.ofNullable(permissions)
            .filter(Predicate.not(Permissions::isEmpty))
            .map(Permissions::group)
            .flatMap(Group::admin)
            .map(User::isActive)
            .orElse(Boolean.FALSE);
```

```java
////////////////////////////////////////////////////////////
// [ex-9-8] Optional 없이 -- 권한 있는 액티브 관리자 찾기

boolean isActiveAdmin = false;

if (permissions != null && !permissions.isEmpty()) {
    if (permissions.group() != null) {
        var group = permissions.group();
        var maybeAdmin = group.admin();

        if (maybeAdmin.isPresent()) {
            var admin = maybeAdmin.orElseThrow();
            isActiveAdmin = admin.isActive();
        }
    }
}
```

### 9.4 Optional과 스트림
### 9.4.1 Optional을 스트림 요소로 활용하기

```java
////////////////////////////////////////////////////////////
// [ex-9-9] 스트림 요소 Optional

List<User> activeUsers =
        permissions.stream()
                   .filter(Predicate.not(Permissions::isEmpty))
                   .map(Permissions::group)
                   .map(Group::admin)
                   .filter(Optional::isPresent)
                   .map(Optional::orElseThrow)
                   .filter(User::isActive)
                   .toList();
```

```java
////////////////////////////////////////////////////////////
// [ex-9-10] 스트림 요소 Optional 에 flatMap을 사용

List<User> activeUsers =
        permissions.stream()
                   .filter(Predicate.not(Permissions::isEmpty))
                   .map(Permissions::group)
                   .map(Group::admin)
                   .flatMap(Optional::stream)
                   .filter(User::isActive)
                   .toList();
```

### 9.4.2 스트림의 최종 연산

### 9.5 원시 타입용 Optional

### 9.6 주의 사항
### 9.6.1 Optional은 일반적인 타입이다
### 9.6.2 식별자에 민감한 메서드
### 9.6.3 성능 오버헤드

- 값의 존재 여부를 확인하거나 대체값을 제공하는 것 외에 추가적인 연산이 필요할 때만

```java
////////////////////////////////////////////////////////////
// (346)

// not recommended

String value = Optional.ofNullable(maybeNull).orElse(fallbackValue);

if (Optional.ofNullable(maybeNull).isPresent()) {
    // ...
}

// recommended

String value = maybeNull != null ? maybeNull : fallbackValue;

if (maybeNull != null) {
    // ...
}
```

- ```
  Objects.requireNonNullElse(x, default) : T
  Objects.requireNonNullElseGet(T, Supplier<T>) : T
  ```

### 9.6.4 컬렉션에 대한 고려 사항
### 9.6.5 `Optional`과 직렬화

### 9.7 `null` 참조에 대한 생각


## CHAPTER 10 함수형 예외 처리

### 10.1 자바 예외 처리

### 10.2 try-catch 블록

```java
////////////////////////////////////////////////////////////
// (353,354) try-catch/TryCatch.jav

try {
    doCalculation(BigDecimal.ZERO);
} catch (ArithmeticException | IllegalArgumentException e) {
    System.err.println("Calculation failed: " + e);
}

try (var fileReader = new FileReader(path.toFile());
     var bufferedReader = new BufferedReader(fileReader)) {
    
    var firstLine = bufferedReader.readLine();
    System.err.println(firstLine);
} catch (IOException e) {
    System.err.println("Couldn't read first line of " + path + ": " + e);
}
```

### 10.2.1 예외와 에러의 다양한 유형

### 10.3 람다에서의 체크 예외
### 10.3.1 안전한 메서드 추출

```java
////////////////////////////////////////////////////////////
// (360) [ex-10-1] 안전 메서드 -- 안에서 예외를 처리

static String safeReadString(Path path) {
    try {
        return Files.readString(path);
    } catch (IOException e) {
        return null;
    }
}

Stream.of(path1, path2, path3)
      .map(SafeMethodExtraction::safeReadString)
      .filter(Objects::nonNull)
      .count();
```

### 10.3.2 언체크 예외

```java
////////////////////////////////////////////////////////////
// (362) [ex-10-2] ThrowingFunction

@FunctionalInterface
public interface ThrowingFunction<T, U> extends Function<T, U> {

    U applyThrows(T elem) throws Exception;

    @Override
    default U apply(T t) {
        try {
            return applyThrows(t);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static <T, U> Function<T, U> uncheck(ThrowingFunction<T, U> fn) {
        return fn::apply;
    }
}

// (363)

ThrowingFunction<Path,String> throwingFn = Files::readString;

Stream.of(path1, path2, path3)
      .map(ThrowingFunction.uncheck(Files::readString))
      .filter(Objects::nonNull)
      .count();
```

### 10.3.3 몰래 던지기

### 10.4 함수형으로 예외 다루기
### 10.4.1 예외를 발생시키지 않기

```java
////////////////////////////////////////////////////////////
// [ex-10-4] IOException 을 던지는 대신 Optional<String> 을 사용

Optional<String> safeReadString(Path path) {
    try {
        var content = Files.readString(path);
        return Optional.of(content);
    } catch (IOException e) {
        return Optional.empty();
    }
}
```

### 10.4.2 값으로서의 예러

```java
////////////////////////////////////////////////////////////
// [ex-10-5] 전통적인 결과 객체
// [ex-10-6] 리턴 타입으로 Result<V,E> 를 사용
// [ex-10-7] Result<V,E> 에 변환기 추가 -- mapSuccess, mapFailure, map
// result-reactions-fallback/ResultReactionsFallback.java -- ifSuccess, ifFailure, handle, orElse, orElseGet, orElseThrow

record Result<V, E extends Throwable> (V value, E throwable, boolean isSuccess) {

    static <V, E extends Throwable> Result<V, E> success(V value) {
        return new Result<>(value, null, true);
    }

    static <V, E extends Throwable> Result<V, E> failure(E throwable) {
        return new Result<>(null, throwable, false);
    }

    <R> Optional<R> mapSuccess(Function<V, R> fn) {
        return this.isSuccess
               ? Optional.ofNullable(this.value).map(fn)
               : Optional.empty();
    }

    <R> Optional<R> mapFailure(Function<E, R> fn) {
        return this.isSuccess
               ? Optional.empty()
               : Optional.ofNullable(this.throwable).map(fn);
    }

    <R> R map(Function<V, R> successFn, Function<E, R> failureFn) {
        return this.isSuccess
               ? successFn.apply(this.value)
               : failureFn.apply(this.throwable);
    }

    void ifSuccess(Consumer<? super V> action) {
        if (this.isSuccess) {
            action.accept(this.value);
        }
    }

    void ifFailure(Consumer<? super E> action) {
        if (!this.isSuccess) {
            action.accept(this.throwable);
        }
    }

    void handle(Consumer<? super V> successAction, Consumer<? super E> failureAction) {
        if (this.isSuccess) {
            successAction.accept(this.value);
        } else {
            failureAction.accept(this.throwable);
        }
    }

    V orElse(V other) {
        return this.isSuccess ? this.value : other;
    }

    V orElseGet(Supplier<? extends V> otherSupplier) {
        return this.isSuccess ? this.value : otherSupplier.get();
    }

    private <E extends Throwable> void sneakyThrow(Throwable e) throws E {
        throw (E) e;
    }

    V orElseThrow() {
        if (!this.isSuccess) {
            sneakyThrow(this.throwable);
            return null;
        }

        return this.value;
    }
}

static Result<String, IOException> safeReadString(Path path) {
    try {
        return Result.success(Files.readString(path));
    } catch (IOException e) {
        return Result.failure(e);
    }
}

// [ex-10-6] 리턴 타입으로 Result<V,E> 를 사용

var result = Stream.of(path1, path2, path3)
                   .map(ResultAsReturnType::safeReadString)
                   .filter(Result::isSuccess)
                   .count();

// [ex-10-7] Result<V,E> 에 변환기 추가 -- mapSuccess, mapFailure, map

// HANDLE ONLY SUCCESS CASE
var successOnly = Stream.of(path1, path2, path3)
                        .map(ResultTransformers::safeReadString)
                        .map(result -> result.mapSuccess(String::toUpperCase))
                        .flatMap(Optional::stream)
                        .toList();

// HANDLE BOTH CASES
var result = safeReadString(path)
                .map(
                    success -> success.toUpperCase(),
                    failure -> "IO-Error: " + failure.getMessage()
                );

// result-reactions-fallback/ResultReactionsFallback.java -- ifSuccess, ifFailure, handle, orElse, orElseGet, orElseThrow

safeReadString(path)
    .handle(
        success -> System.out.println("Success!"),
        failure -> System.out.println("IO-Error: " + failure.getMessage())
    );
```

### 10.4.3 Try/Success/Failure 패턴

```java
////////////////////////////////////////////////////////////
// [ex-10-9] 람다와 예외 핸들러를 받아들이는 미니멀 Try<T,R>
// [ex-10-10] Try<T,R> 에서 성공과 실패 처리하기
// [ex-10-11] Try<T,R> 에 값 적용하기
// [ex-10-12] Function<T, Optional<R>> 구현

class Try<T, R> implements Function<T, Optional<R>> {

    private final Function<T, R>                fn;
    private final Function<RuntimeException, R> failureFn;

    static <T, R> Try<T, R> of(ThrowingFunction<T, R> fn) {
        Objects.requireNonNull(fn);

        return new Try<>(fn, null);
    }

    Try(Function<T, R> fn, Function<RuntimeException, R> failureFn) {
        this.fn = fn;
        this.failureFn = failureFn;
    }

    Try<T, R> success(Function<R, R> successFn) {
        Objects.requireNonNull(successFn);

        var composedFn = this.fn.andThen(successFn);
        return new Try<>(composedFn, this.failureFn);
    }

    Try<T, R> failure(Function<RuntimeException, R> failureFn) {
        Objects.requireNonNull(failureFn);

        return new Try<>(this.fn, failureFn);
    }

    @Override
    public Optional<R> apply(T value) {
        try {
            var result = this.fn.apply(value);
            return Optional.ofNullable(result);
        } catch (RuntimeException e) {
            if (this.failureFn != null) {
                var result = this.failureFn.apply(e);
                return Optional.ofNullable(result);
            }
        }
        
        return Optional.empty();
    }
}

// [ex-10-10] Try<T,R> 에서 성공과 실패 처리하기

var trySuccessFailure =
        Try.<Path, String> of(Files::readString)
                          .success(String::toUpperCase)
                          .failure(str -> null);

// [ex-10-11] Try<T,R> 에 값 적용하기

Optional<String> result =
        Try.<Path, String> of(Files::readString)
                          .success(String::toUpperCase)
                          .failure(str -> null)
                          .apply(path);

// [ex-10-12] Function<T, Optional<R>> 구현

Function<Path, Optional<String>> fileLoader =
        Try.<Path, String> of(Files::readString)
                          .success(String::toUpperCase)
                          .failure(str -> null);

var result = Stream.of(path1, path2, path3)
                   .map(fileLoader)
                   .flatMap(Optional::stream)
                   .count();
```

### 10.5 함수형 예외 처리에 대한 고찰


## CHAPTER 11 느긋한 계산법 (지연 평가)

### 11.1 느긋함 vs 엄격함

### 11.2 자바는 얼마나 엄격한가?
### 11.2.1 단축 평가 계산
### 11.2.2 제어 구조
### 11.2.3 JDK에서의 느긋한 타입

### = 느긋한 Map

- ```
  Map
  . computeIfAbsent(K, mapping : Function<K,V>) : V
  . compute(K, remapping : BiFunction<K,V,V>) : V
  ```

### = Optional

- ```
  Optional
  . orElseGet(Supplier<T>) : T
  ```

### 11.3 람다와 고차 함수
### 11.3.1 열정적인 접근 방식

```java
////////////////////////////////////////////////////////////
// [ex-11-3] eager 메서드 인수를 사용해 사용자를 업데이트

User updateUser(User user, List<Role> availableRoles) {
    ...
}

List<Role> availableRoles = dao.loadAllAvailableRoles();

updateUser(user, availableRoles);
```

### 11.3.2 느긋한 접근 방식

```java
////////////////////////////////////////////////////////////
// (396)

User updateUser(User user, Dao roleDao) {
    ...
    List<Role> availableRoles = roleDao.loadAllAvailableRoles();
    ...
}

updateUser(user, dao);
```

### 11.3.3 함수형 접근 방식

```java
////////////////////////////////////////////////////////////
// [ex-11-4] 람다를 써서 사용자를 업데이트

User updateUser(User user, Supplier<List<Role>> availableRolesFn) {
    ...
    List<Role> availableRoles = availableRolesFn.get();
    ...
}

updateUser(user, dao::loadAllAvailableRoles);
```

### 11.4 썽크를 사용한 지연 실행

- memoization

### 11.4.1 간단한 썽크 만들기

```java
////////////////////////////////////////////////////////////
// [ex-11-5] 간단한 Thunk<T> 구현

public static class Thunk<T> implements Supplier<T> {

    private final Supplier<T> expression;
    private T result;

    private Thunk(Supplier<T> expression) {
        this.expression = expression;
    }

    @Override
    public T get() {
        if (this.result == null) {
            this.result = this.expression.get();
        }
        return this.result;
    }

    public static <T> Thunk<T> of(Supplier<T> expression) {
        if (expression instanceof Thunk<T>) {
            return (Thunk<T>) expression;
        }

        return new Thunk<>(expression);
    }
}

updateUser(user, Thunk.of(dao::loadAllAvailableRoles));
```

```java
////////////////////////////////////////////////////////////
// [ex-11-6] 함수형으로 확장한 Thunk<T>

public static class Thunk<T> implements Supplier<T> {

    ...

    public static <T> Thunk<T> of(T value) {
        return new Thunk<T>(() -> value);
    }

    public <R> Thunk<R> map(Function<T, R> mapper) {
        return Thunk.of(() -> mapper.apply(get()));
    }

    public void accept(Consumer<T> consumer) {
        consumer.accept(get());
    }
}
```

### 11.4.2 안전한 스레드 썽크

```java
////////////////////////////////////////////////////////////
// [ex-11-7] 동기화된 평가를 이용한 Thunk<T>

public static class Thunk<T> implements Supplier<T> {

    private static class Holder<T> implements Supplier<T> {

        private final T value;

        Holder(T value) {
            this.value = value;
        }

        @Override
        public T get() {
            return this.value;
        }
    }

    private Supplier<T> holder;

    private Thunk(Supplier<T> expression) {
        this.holder = () -> evaluate(expression);
    }

    public static <T> Thunk<T> of(Supplier<T> expression) {
        return new Thunk<>(expression);
    }

    private synchronized T evaluate(Supplier<T> expression) {
        if ((this.holder instanceof Holder) == false) {
            var evaluated = expression.get();
            this.holder = new Holder<>(evaluated);
        }
        return this.holder.get();
    }

    @Override
    public T get() {
        return this.holder.get();
    }
}
```

- see also
  - `java.util.concurrent.atomic.Atomic*`
  - `ConcurrentHashMap#computeIfAbsent`

### 11.5 느긋함에 대한 고찰


## CHAPTER 12 재귀

### 12.1 재귀란 무엇인가?
### 12.1.1 머리 재귀와 꼬리 재귀
### 12.1.2 재귀와 호출 스택

### 12.2 더 복잡한 예시
### 12.2.1 반복적인 트리 순회
### 12.2.2 재귀적인 트리 순회

- > 이 접근법은 반복적인 접근법에 비해 간결하고 함수적이며 더 쉽게 이해할 수 있습니다.
  > 그럼에도 불구하고, 루프를 사용하는 것에도 장점이 있습니다.
  > 가장 큰 장점은 성능 차이인데 필요한 스택 공간을 사용 가능한 힙heap 공간으로 바꾸는 것입니다. [419]

### 12.3 재귀와 유사한 스트림

```java
////////////////////////////////////////////////////////////
// stream-recursion/stream-recursion.md

@FunctionalInterface
public interface RecursiveCall<T> {

    // Represents the recursive call
    RecursiveCall<T> apply();

    // The wrapper needs to know when the call chain is
    // complete by reaching its base condition
    default boolean isComplete() {
        return false;
    }

    // The final result of the recursive call
    default T result() {
        return null;
    }

    // Starts the Stream pipeline
    default T run() {
        return Stream.iterate(this, RecursiveCall::apply)
                     .filter(RecursiveCall::isComplete)
                     .findFirst()
                     .get()
                     .result();
    }

    // A convenience method for creating a lambda representing
    // a reached base condition and containing the actual result
    static <T> RecursiveCall<T> done(T value) {

        return new RecursiveCall<T>() {

            @Override
            public boolean isComplete() {
                return true;
            }

            @Override
            public T result() {
                return value;
            }

            @Override
            public RecursiveCall<T> apply() {
                return this;
            }
        };
    }
}

RecursiveCall<Long> sum(Long total, Long summand) {
    if (summand == 1) {
        return RecursiveCall.done(total);
    }

    return () -> sum(total + summand, summand - 1L);
}

var result = sum(1L, 4000L).run();
```

### 12.4 재귀에 대한 고찰


## CHAPTER 13 비동기 작업

### 13.1 동기 vs 비동기

### 13.2 자바의 `Future`

```java
////////////////////////////////////////////////////////////
// [ex-13-1] Future<T> 실행 흐름

ExecutorService execSvc = Executors.newFixedThreadPool(10);

Callable<Integer> task = () -> {
    System.out.println("enter task");
    TimeUnit.SECONDS.sleep(2);
    Systen.out.println("exit task");
    return 42;
};

System.out.println("before submit");
Future<Integer> future = execSvc.submit (task);
System.out.println("after submit");
try {
    int result = future.get();
    System.out.println("blocking ends & got " + result);
} catch (InterruptedException | ExecutionException e) {
    throw new RuntimeException(e);
}
```

### 13.3 `CompletableFutures`로 비동기 파이프라인 구축
### 13.3.1 값에 대한 프로미스
### 13.3.2 `CompletableFuture` 생성

- ```
  CompletableFuture<T>
  + runAsync(Runnable) : CompletableFuture<Void>
  + supplyAsync(Supplier<U>) : CompletableFuture<U>
  ```

```java
////////////////////////////////////////////////////////////
// [ex-13-2] CompletableFuture 생성

// Future<T>

ForkJoinPool execSvc = ForkJoinPool.commonPool();

ForkJoinTask<?> futureRunnable = execSvc.submit(() -> System.out.println("..."));

ForkJoinTask<String> futureCallable = execSvc.submit(() -> "...");

// CompletableFuture<T>

CompletableFuture<Void> completableFutureRunnable = CompletableFuture.runAsync() -> System.out.println("..."));

CompletableFuture<String> completableFutureSupplier = CompletableFuture.supplyAsync(() -> "...");
```

### 13.3.3 작업 합성 및 결합

### = 작업 합성

- ```
  CompletableFuture<T>
  . thenAccept(Consumer<T>) : CompletableFuture<Void>
  . thenApply(Function<T,U>) : CompletableFuture<U>
  . thenRun(Runnable) : CompletableFuture<Void>
  ```

```java
////////////////////////////////////////////////////////////
// [ex-13-3] 비동기 북마크 관리자 워크플로

var task = CompletableFuture.supplyAsync(() -> this.downloadService.get(url))
                            .thenApply(this.contentCleaner::clean)
                            .thenRun(this.storage::save);
```

### = 작업 결합

- ```
  CompletableFuture<T>
  . thenCombine(other : CompletionStage<U>, BiFunction<T,U,V>) : CompletableFuture<V>
  
  . thenCompose(Function<T, CompletionStage<U>>) : CompletableFuture<U>

  . applyToEither(other : CompletionStage<T>, Function<T,U>) : CompletableFuture<U>

  . thenAcceptBoth(other : CompletionStage<U>, action : BiConsumer<T,U>) : CompletableFuture<Void>
  . acceptEither(other : CompletionStage<T>, action : Consumer<T>) : CompletableFuture<Void>

  . runAfterBoth(other : CompletionStage<?>, action : Runnable) : CompletableFuture<Void>
  . runAfterEither(other : CompletionStage<?>, action : Runnable) : CompletableFuture<Void>
  ```

```java
////////////////////////////////////////////////////////////
// [ex-13-4] 중첩된 단계 풀기

CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 42);
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(() -> 23);

BiFunction<Integer, Integer, CompletableFuture<Integer>> task =
        (lhs, rhs) -> CompletableFuture.supplyAsync(() -> lhs + rhs);

CompletableFuture<Integer> combined = future1.thenCombine(future2, task)
                                             .thenCompose(Function.identity());
```

### = 동시에 여러 CompletableFuture 인스턴스 실행하기

- ```
  CompletableFuture<T>
  + allOf(CompletableFuture<?>...) : CompletableFuture<Void>
  + anyOf(CompletableFuture<?>...) : CompletableFuture<Object>
  ```

### 13.3.4 예외 처리

- ```
  CompletableFuture<T>
  . exceptionally(Function<Throwable, T>) : CompletionStage<T>
  . handle(BiFunction<T, Throwable, U>) : CompletableFuture<U>
  . whenComplete(action : BiConsumer<T, Throwable>) : CompletableFuture<T>
  ```

### = 거부된 either 작업

```java
////////////////////////////////////////////////////////////
// [ex-13-5] either 연산과 거부된 단계

CompletableFuture<String> notFailed =
        CompletableFuture.supplyAsync(() -> "Success!");

CompletableFuture<String> failed =
        CompletableFuture.supplyAsync(() -> { throw new RuntimeException(); });

// 이전 스테이지가 실패이기 때문에 출력 없음
var rejected = failed.acceptEither(notFailed, System.out::println);

// 이전 스테이지가 정상이기 때문에 출력됨
var resolved = notFailed.acceptEither(failed, System.out::println);
```

### 13.3.5 종료 연산

- ```
  CompletableFuture<T>
  . get() : T                         // InterruptedException, ExecutionException
  . get(timeout : long, TimeUnit) : T // InterruptedException, ExecutionException, TimeoutException
  . getNow(valueIfAbsent : T) : T
  . join() : T
  ```

- ```
  CompletableFuture<T>
  . cancel(mayInterruptIfRunning : boolean) : boolean
  . isCancelled() : boolean
  . isDone() : boolean
  . isCompletedExceptionally() : boolean
  ```

### 13.3.6 `CompletableFuture` Helper 생성

기능 추가 -- eachOf(CompletableFuture<T>...) : CompletableFuture<List<T>>

### = CompletableFuture 헬퍼 개선하기

```java
////////////////////////////////////////////////////////////
// [ex-13-6] eachOf 구현
// [ex-13-7] CompletableFutures 리팩토링

private final static Predicate<CompletableFuture<?>> EXCEPTIONALLY =
        Predicate.not(CompletableFuture::isCompletedExceptionally);

private static <T> Function<Void, List<T>> gatherResultsFn(CompletableFuture<T>... cfs) {

    return unused -> Arrays.stream(cfs)
                            .filter(Predicate.not(CompletableFutures.EXCEPTIONALLY))
                            .map(CompletableFuture::join)
                            .toList();
}

public static <T> CompletableFuture<List<T>> eachOf(CompletableFuture<T>... cfs) {
    return CompletableFuture.allOf(cfs)
                            .thenApply(gatherResultsFn(cfs));
}

public static <T> CompletableFuture<List<T>> bestEffort(CompletableFuture<T>... cfs) {
    return CompletableFuture.allOf(cfs)
                            .exceptionally(ex -> null)
                            .thenApply(gatherResultsFn(cfs));
}
```

### 13.4 수동 생성 및 수동 완료
### 13.4.1 수동 생성

```java
////////////////////////////////////////////////////////////
// (447)

CompletableFuture<String> unsettled = new CompletableFuture<>();
```

### 13.4.2 수동 완료

- ```
  CompletableFuture<T>
  . complete(value : T) : boolean
  . completeExceptionally(Throwable) : boolean

  . completeAsync(Supplier<T>) : CompletableFuture<T>
  . completeAsync(Supplier<T>, Executor) : CompletableFuture<T>
  . completeOnTimeout(value : T, timeout : long, TimeUnit) : CompletableFuture<T>

  + completedFuture(value : U) : CompletableFuture<U>
  + failedFuture(Throwable) : CompletableFuture<U>
  + completedStage(value : U) : CompletionStage<U>
  + failedStage(Throwable) : CompletionStage<U>
  ```

### 13.4.3 수동으로 생성 및 완료된 인스턴스 활용 사례

### = CompletableFuture를 반환값으로 사용하기

```java
////////////////////////////////////////////////////////////
// [ex-13-8] CompletableFuture를 사용한 캐시된 날씨 서비스

public CompletableFuture<WeatherInfo> check(int zipCode) {
    return cacheLookup(zipCode)
            .map(CompletableFuture::completedFuture)
            .orElseGet(() -> restCall(zipCode));
}

private Optional<WeatherInfo> cacheLookup(int zipCode) {
    return Optional.ofNullable(this.cache.get(zipCode));
}

private CompletableFuture<WeatherInfo> restCall(int zipCode) {
    Supplier<WeatherInfo> restCall = () -> apiGetWeatherInfo(zipCode);
    return CompletableFuture.supplyAsync(restCall)
            .thenApply(this::storeInCache);
}

private WeatherInfo apiGetWeatherInfo(int zipCode) {
    return new WeatherInfo(zipCode);
}

private WeatherInfo storeInCache(WeatherInfo info) {
    return this.cache.put(info.zipCode(), info);
}
```

### = 보류중인 CompletableFuture 파이프라인

```java
////////////////////////////////////////////////////////////
// [ex-13-9] 미확정 상태의 CompletableFuture를 활용한 이미지 프로세서 설계

public record Task(
    CompletableFuture<Path> start,
    CompletableFuture<InputStream> end) {
}

public Task createTask(int maxHeight,
                        int maxWidth,
                        boolean keepAspectRatio,
                        boolean trimWhitespace) {
    
    var start = new CompletableFuture<Path>();

    var end = unsettled.thenApply(...)
                        .exceptionally(...)
                        .thenApply(...)
                        .handle(...);
    
    return new Task(start, end);
}

// (453)

// 느긋한 작업 생성
var task = this.imageProcessor.createTask(800, 600, false, true);

// 작업 실행
var path = Path.of("path/to/foobar.png");
task.start().complete(path);

// 결과값에 접근
var processed = task.end().get();
```

### 13.5 스레드 풀과 타임 아웃의 중요성

- > 적절한 타임아웃을 설정하는 것은 영원히 차단되는 스레드 만드는 것을 예방하는 데 도움이 됩니다.
  > 그러나 타임아웃을 사용한다는 것은 TimeeoutException도 처리해야 한다는 것을 의미합니다. [455]

- ```
  CompletableFuture<T>
  . completeOnTimeout(value : T, timeout : long, TimeUnit) : CompletableFuture<T>
  . orTimeout(timeout : long, TimeUnit) : CompletableFuture<T>
  . get(timeout : long, TimeUnit) : T // InterruptedException, ExecutionException, TimeoutException

  . cancel(mayInterruptIfRunning : boolean) : boolean
  ```

### 13.6 비동기 작업에 대한 고찰


## CHAPTER 14 비함수형 디자인 패턴

### 14.1 디자인 패턴이란?

### 14.2 (함수형) 디자인 패턴
### 14.2.1 팩토리 패턴

### = 객체 지향 접근 방식

```java
////////////////////////////////////////////////////////////
// factory-pattern-oo/*.java

public class ShapeFactory {

    public static Shape newShape(ShapeType type, Color color) {

        return switch (type) {
            case CIRCLE -> new Circle(color);
            case TRIANGLE -> new Triangle(color);
            case SQUARE -> new Square(color);
            case PENTAGON -> new Pentagon(color);
        };
    }
}

Shape triangle = ShapeFactory.newShape(ShapeType.TRIANGLE, Color.RED);
```

### = 더 함수적인 접근 방식

```java
////////////////////////////////////////////////////////////
// factory-pattern-fp/*.java

public enum ShapeType {
    CIRCLE(Circle::new),
    TRIANGLE(Triangle::new),
    SQUARE(Square::new),
    PENTAGON(Pentagon::new);

    public final Function<Color, Shape> factory;

    ShapeType(Function<Color, Shape> factory) {
        this.factory = factory;
    }

    public Shape newInstance(Color color) {
        return this.factory.apply(color);
    }
}

Shape triangle = ShapeType.TRIANGLE.newInstance(Color.RED);
```

### 14.2.2 데코레이션 패턴

### = 객체 지향 접근 방식

```java
////////////////////////////////////////////////////////////
// decorator-pattern-oo/*.java

public interface CoffeeMaker {
    List<String> getIngredients();
    Coffee prepare();
}

public class BlackCoffeeMaker implements CoffeeMaker {

    @Override
    public List<String> getIngredients() {
        return List.of("Robusta Beans", "Water");
    }

    @Override
    public Coffee prepare() {
        return new BlackCoffee();
    }
}

public abstract class Decorator implements CoffeeMaker {

    private final CoffeeMaker target;

    public Decorator(CoffeeMaker target) {
        this.target = target;
    }

    @Override
    public List<String> getIngredients() {
        return this.target.getIngredients();
    }

    @Override
    public Coffee prepare() {
        return this.target.prepare();
    }
}

public class AddMilkDecorator extends Decorator {

    private final MilkCarton milkCarton;

    public AddMilkDecorator(CoffeeMaker target, MilkCarton milkCarton) {
        super(target);

        this.milkCarton = milkCarton;
    }

    @Override
    public List<String> getIngredients() {
        var newIngredients = new ArrayList<>(super.getIngredients());
        newIngredients.add("Milk");
        return newIngredients;
    }

    @Override
    public Coffee prepare() {
        var coffee = super.prepare();
        coffee = this.milkCarton.pourInto(coffee);
        return coffee;
    }
}

{
    // Café con Leche

    CoffeeMaker coffeeMaker = new BlackCoffeeMaker();
    CoffeeMaker decoratedCoffeeMaker = new AddMilkDecorator(coffeeMaker, new MilkCarton());

    Coffee cafeConLeche = decoratedCoffeeMaker.prepare();

    // Add Sugar

    CoffeeMaker lastDecoratedCoffeeMaker = new AddSugarDecorator(decoratedCoffeeMaker);
}
```

### = 더 함수적인 접근 방식

```java
////////////////////////////////////////////////////////////
// decorator-pattern-fp/*.java

public static CoffeeMaker decorate(CoffeeMaker coffeeMaker,
                                   Function<CoffeeMaker, CoffeeMaker> decorator) {

    return decorator.apply(coffeeMaker);
}

{
    CoffeeMaker decoratedCoffeeMaker =
            Barista.decorate(new BlackCoffeeMaker(),
                             coffeeMaker -> new AddMilkDecorator(coffeeMaker,
                                                                 new MilkCarton()));

    CoffeeMaker finalCoffeeMaker =
            Barista.decorate(decoratedCoffeeMaker,
                             AddSugarDecorator::new);
}

public static CoffeeMaker decorate(CoffeeMaker coffeeMaker,
                                    Function<CoffeeMaker, CoffeeMaker>... decorators) {

    Function<CoffeeMaker, CoffeeMaker> reducedDecorations =
            Arrays.stream(decorators)
                  .reduce(Function.identity(),
                          Function::andThen);

    return reducedDecorations.apply(coffeeMaker);
}

{
    CoffeeMaker multiDecoratedCoffeeMaker =
            Barista.decorate(new BlackCoffeeMaker(),
                             coffeeMaker -> new AddMilkDecorator(coffeeMaker,
                                                                  new MilkCarton()),
                             AddSugarDecorator::new);
}

public final class Decorations {

    public static Function<CoffeeMaker, CoffeeMaker> addMilk(MilkCarton milkCarton) {
        return coffeeMaker -> new AddMilkDecorator(coffeeMaker, milkCarton);
    }

    public static Function<CoffeeMaker, CoffeeMaker> addSugar() {
        return AddSugarDecorator::new;
    }
}

{
    CoffeeMaker maker = Barista.decorate(new BlackCoffeeMaker(),
                                         Decorations.addMilk(new MilkCarton()),
                                         Decorations.addSugar());
}
```

### 14.2.3 전략 패턴

### = 객체 지향 접근 방식

```java
////////////////////////////////////////////////////////////
// strategy-pattern-oo/*.java

public interface ShippingStrategy {
    void ship(Parcel parcel);
}

public interface ShippingService {
    default void ship(Parcel parcel, ShippingStrategy strategy) {
        strategy.ship(parcel);
    }
}

public class StandardShipping implements ShippingStrategy {
    ...
}

public class ExpeditedShipping implements ShippingStrategy {

    private final boolean signatureRequired;

    public ExpeditedShipping(boolean signatureRequired) {
        this.signatureRequired = signatureRequired;
    }

    ...
}

{
    ShippingService service = new ShippingService() {};
    Parcel parcel = new Parcel();
    ShippingStrategy strategy = new ExpeditedShipping(true);

    service.ship(parcel, strategy);
}
```

### = 더 함수적인 접근 방식

- 행동의 매개변수화behavioral parameterization

- - 람다와 메서드 참조
  - 부분 적용 함수
  - 구체적인 구현

```java
////////////////////////////////////////////////////////////
// strategy-pattern-fp/*.java

public final class ShippingStrategies {

    public static ShippingStrategy expedited(boolean requiresSignature) {
        return new ExpeditedShipping(requiresSignature);
    }

    public static ShippingStrategy standard() {
        return new StandardShipping();
    }
}

{
    ShippingService service = new ShippingService() {};
    Parcel parcel = new Parcel();

    service.ship(parcel, ShippingStrategies.expedited(true));
}
```

### 14.2.4 빌더 패턴

### = 객체 지향 접근 방식

```java
////////////////////////////////////////////////////////////
// builder-pattern-oo/*.java

public record User(String email,
                   String name,
                   List<String> permissions) {

    public User {
        if (email == null || email.isBlank()) {
            throw new IllegalArgumentException("'email' must be set.");
        }

        if (permissions == null) {
            permissions = Collections.emptyList();
        }
    }

    public static class Builder {

        private String email;
        private String name;
        private final List<String> permissions = new ArrayList<>();

        public Builder email(String email) {
            this.email = email;
            return this;
        }

        public Builder name(String name) {
            this.name = name;
            return this;
        }

        public Builder addPermission(String permission) {
            this.permissions.add(permission);
            return this;
        }

        public User build() {
            return new User(this.email,
                            this.name,
                            this.permissions);
        }
    }

    public static Builder builder() {
        return new Builder();
    }
}

{
	var builder = User.builder()
                      .email("john@doe")
                      .name("John Doe");

	// 다른 작업을 하고 빌더를 전달

	var user = builder.addPermission("create")
                      .addPermission("edit")
                      .build();
}
```

### = 더 함수적인 접근 방식

```java
////////////////////////////////////////////////////////////
// builder-pattern-fp/*.java

public record User(String email, String name, List<String> permissions) {

    ...

    public static class Builder {

        private Supplier<String> emailSupplier;
        private Supplier<String> nameSupplier;
        private final List<String> permissions = new ArrayList<>();

        public Builder email(String email) {
            return email(() -> email);
        }

        public Builder email(Supplier<String> emailSupplier) {
            this.emailSupplier = emailSupplier;
            return this;
        }

        public Builder name(String name) {
            return name(() -> name);
        }

        public Builder name(Supplier<String> nameSupplier) {
            this.nameSupplier = nameSupplier;
            return this;
        }

        public Builder addPermission(String permission) {
            this.permissions.add(permission);
            return this;
        }

        public User build() {
            return new User(this.emailSupplier.get(),
                            this.nameSupplier.get(),
                            this.permissions);
        }
    }

    public static Builder builder() {
        return new Builder();
    }
}

{
	var builder = User.builder()
                      .name(() -> "Ben Weidig")
                      .email("ben@example.com");

    var user = builder.addPermission("create")
                      .addPermission("edit")
                      .build();
}

public record User(String email, String name, List<String> permissions) {

    public String email;
    public String name;
    private final List<String> permissions = new ArrayList<>();

    public static class Builder {

        ...

        public Builder with(Consumer<Builder> builderFn) {
            builderFn.accept(this);
            return this;
        }

        public Builder withPermissions(Consumer<List<String>> permissionsFn) {
            permissionsFn.accept(this.permissions);
            return this;
        }

        ...
    }

    ...
}

{
	var user2 = User.builder()
                    .with(with -> {
                        with.email = "ben@example.com";
                        with.name = "Ben Weidig";
                    })
                    .withPermissions(permissions -> {
                        permissions.add("create");
                        permissions.add("view");
                    })
                    .build();
}
```

### 14.3 함수형 디자인 패턴에 대한 고찰


## CHAPTER 15 자바를 위한 함수형 접근 방식

### 15.1 객체 지향 프로그래밍과 함수형 프로그래밍 원칙 비교

- 객체 지향
  - 데이터와 동작의 캡슐화, 다형성, 추상화
  - 메타포 기반metaphor-based
- 함수형
  - 수학적 원리(?)를 사용하여 문제를 해결
  - 선언적 코드 스타일
  - 고차원 추상화

- 함수형이
  - 객체 지향의
    - 정교한, 표현력 있는, 도메인 기반 접근
  - ...을 저해하게 될까?
    - 메서드 시그니처의 표현력

### 15.2 함수형 사고방식
### 15.2.1 함수는 일급 객체다
### 15.2.2 사이트 이펙트 피하기

### = 순수 함수

```java
////////////////////////////////////////////////////////////
// (490-491)

public String buildGreeting(User user) {
    String greeting;
    if (LocalTime.now().getHour() < 12) {
        greeting = "Good morning";
    } else {
        greeting = "Hello"
    }
    return String.format("%s, %s", greeting, user.name());
}

public String buildGreeting(User user, LocalTime time) {
    ...
}
```

### = 사이드 이펙트 격리

- - 더 작고 순수한 함수로 분할해서
  - 사이드 이펙트를 격리하고 캡슐화
  - `processFile(Path) : String` 대신
  - `loadFile(Path) : Optional<String>`과 `process(content : String) : String`로 분리

### = 문장보다는 표현식을 사용하기

```java
////////////////////////////////////////////////////////////
// (496)

public String buildGreeting(User user, LocalTime time) {
    String greeting = time.getHour() < 12 ? "Good Morning"
                                          : "Hello";
    return String.format("%s, %s", greeting, user.name());
}
```

### = 불변성을 향해 나아가기

- - immutable 컬렉션
  - `BigInteger`, `BigDecimal`
  - `record`
  - 날짜 및 시간 API

- 불변성을 나중에 추가하는 것보다 (부분적으로) 빼는 것이 더 쉽다

### 15.2.3 맵/필터/리듀스를 활용한 데이터 처리

```java
////////////////////////////////////////////////////////////
// (499)

List<User> usersToNotify = new ArrayList<>();
for (var users : availableUsers) {
    if (!user.hasValidSubscription()) {
        continue;
    }
    usersToNotify.add(user);
}
notify(usersToNotify);

////////////////////////////////////////////////////////////
// (500)

List<User> usersToNotify = availableUsers.stream()
                                         .filter(User::hasValidSubscription)
                                         .toList();
notify(usersToNotify);
```

### 15.2.4 추상화 가이드 구현

- - (객체 지향에서) 요구 사항의 변화가 초래한 메타포 불일치
  - (함수형에서) (워낙) 고차원 추상화

### 15.2.5 함수형 연결고리 만들기

### = 메서드 참조 친화적 시그니처

- - `Function<T,R>` / `apply(T) : R` / map
  - `Predicate<T>`  / `test(T) : boolean` / filter
  - `Comparator<T>` / `compare(T, T) : int` / sort

- - `s -> s.toLowerCase()`
  - `String::toLowerCase`

### = 기존의 함수형 인터페이스 사용

- - `java.lang.Comparable<T>`
  - `java.lang.Runnable`
  - `java.util.Comparator<T>`
  - `java.util.concurrent.Callable<V>`

```java
////////////////////////////////////////////////////////////
// (503)

// AS ANONYMOUS CLASS

users.sort(new Comparator<User>() {

    @Override
    public int compare(User lhs, User rhs) {
        return lhs.email().compareTo(rhs.email());
    }
});

// AS LAMBDA

users.sort((lhs, rhs) -> lhs.email().compareTo(rhs.email()));

// AS METHOD REFERENCE

users.sort(Comparator.comparing(User::email));
```

### = 일반적인 작업을 위한 람다 팩토리

```java
////////////////////////////////////////////////////////////
// (504)

public class ProductCategory {

    public String localizedDescription(Locale locale) {
        return "...";
    }

    // LAMBDA FACTORY
    public static Function<ProductCategory, String> localizedDescriptionMapper(Locale locale) {
        return category -> category.localizedDescription(locale);
    }
}

var locale = Locale.GERMAN;

List<ProductCategory> categories = List.of(...);

// EXPLICIT LAMBDAS

List<String> descriptionsViaExplicitLambdas =
    categories.stream()
              .map(category -> category.localizedDescription(locale))
              .toList();

// LAMBDA FACTORY

List<String> descriptionsViaLambdaFactory =
    categories.stream()
              .map(ProductCategory.localizedDescriptionMapper(locale))
              .toList();
```

### = 명시적으로 함수형 인터페이스 구현하기

```java
////////////////////////////////////////////////////////////
// (506)

interface VideoConverterJob extends Function<Path, Path> {

    Path convert(Path sourceFile);

    default Path apply(Path sourceFile) {
        return convert(sourceFile);
    }
}
```

### = 함수형 `null` 처리를 위한 `Optional` 사용

```java
////////////////////////////////////////////////////////////
// (508)

public Optional<User> tryLoadUser(long id) {
  // ...
}

boolean isAdminUser =
  tryLoadUser(23L).map(User::getPermissions)
                  .filter(Predicate.not(Permissions::isEmpty))
                  .map(Permissions::getGroup)
                  .flatMap(Group::getAdmin)
                  .map(User::isActive)
                  .orElse(Boolean.FALSE);
```

- 간단한 `null` 검사를 위해 `Optional`을 쓰지는 말 것

```java
////////////////////////////////////////////////////////////
// (509)

// BAD

var nicknameOptional = Optional.ofNullable(customer.getNickname())
                               .orElse("Anonymous");

// BETTER

var nicknameTernary = customer.getNickname() != null ? customer.getNickname()
                                                     : "Anonymous";
```

- 대안

```java
////////////////////////////////////////////////////////////
// (509)

var nickname = Objects.requireNonNullElse(customer.getNickname(), "Anonymous");

var nicknameWithSupplier = Objects.requireNonNullElse(customer.getNickname(),
                                                      () -> "Anonymous");
```

### 15.2.6 병렬성과 동시성 쉽게 다루기
### 15.2.7 잠재적인 오버헤드에 주의하기

### 15.3 명령형 세계의 함수형 아키텍처

- functional core, imperative shell

### 15.3.1 객체에서 값으로
### 15.3.2 관심사 분리
### 15.3.3 FC/IS 아키텍처의 다양한 규모
### 15.3.4 FC/IS 테스팅

### 15.4 자바에서 함수형 접근에 대한 고찰


:wq