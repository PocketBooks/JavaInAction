## Chapter4. 스트림 소개

### 스트림?

> 데이터 처리 연산을 지원하도록 소스에서 추출된 연속된 요소

- 선언형으로 컬렉션을 처리 할 수 있다.
  - 루프나 조건문 등이 아닌 제어 블록을 사용해 동작 수행을 지정한다.
- 데이터를 처리하는 코드 구현대신 질의로 표현 할 수 있다.
  - filter, sorted, map, collect와 같은 빌딩 블록 연산을 이용해 데이터 처리 파이프 라인을 만든다.
- 멀티스레드 코드를 구현하지 않아도 데이터를 투명하게 병렬로 처리할 수 있다.
  - 고수준 빌딩블록으로 이루어져 있어 특정 스레딩 모델에 제한받지 않는다.
- 스트림 사용 시 장점
  - 선언형 : 간결하고 가독성이 좋음
  - 조립할 수 있음 : 유연성
  - 병렬화 : 성능 향상
- 특정 요소 형식으로 이루어진 연속된 값 집합의 인터페이스

### 스트림 특징

- 파이프라이닝
  - 스트림 연산끼리 연결해 커다란 파이프 라인을 구성한다.
  - 이를 통해 게으름, 쇼트서킷같은 최적화를 얻을 수 있다.
- 내부 반복

### 살펴보기

```java
List<String> threeHighCaloriesDishNames = 
  menu.stream() <- 스트림 획득
  	  .filter(dish -> dish.getCalories() > 30) <- 30칼로리 이상 추출
      .map(Dish::getName) <- 요리 이름 추출
      .limit(3) <- 선착순 3개
  		.collect(toList()); <- 다른 리스트로 저장
```

### 스트림과 컬렉션

- 두 형식 모두 연속된 요소 형식의 값을 저장하는 자료구조의 인터페이스 제공
- 순서에 상관없이 아무값에 접근이 아닌 순차적인 접근
- 차이
  - 컬렉션의 저장되는 모든 요소는 연산 수행시 메모리에 저장되어 있으며 미리 계산되어 있어야 함
    - 생산자 중심(supplier-driven)
  - 스트림은 요청할 때만 요소를 계산하는 고정된 자료구조
    - 요청 중심(demand-driven)
    - 더보기 버튼을 생각하면 이해하기 쉬움
- 딱 한 번만 탐색 가능
- 외부 반복(컬렉션) vs 내부 반복(스트림)
  - 외부 반복 : 사용자가 직접 요소를 반복한다.
  - 내부 반복 : 반복을 알아서 처리하고(추상화) 결과 스트림값을 저장한다.
  - 내부 반복의 장점
    - 동시 처리 (한 손에는 인형을, 다른 손에는 장난감을)
    - 일괄 처리 (모든 장난감을 상자 가까이 이동 후, 상자에 넣음)
  - 외부 반복은 병렬성을 스스로 관리해야 함
    - 병렬성 포기
    - synchronized

### 스트림 연산

- 중간 연산
  - 연산을 통해 파이프 라인을 생성
  - 다른 연산과 연결되는 연산
  - 결과를 생성할 수 없음
- 최종 연산
  - 스트림을 닫는다
  - 스트림이 아닌 결과를 반환받을 때 사용
- `쇼트서킷`
  - `limit()`
- `루프 퓨전`
  - `filter()`, `map()`
- 게으른 연산



## Chapter5. 스트림 활용

### 필터링

- 스트림의 요소를 선택하는 방법

- 프리디케이트를 인수로 받아, 프리디케이트와 일치하는 모든 스트림 반환
- `distinct()` : 중복제거

### 슬라이싱

- 스트림의 요소를 선택하거나 스킵하는 방법
- `takeWhile()` : 조건이 참 일때까지 반복
- `dropWhile()` : 조건이 거짓 일때까지 반복
- `limit()` : 갯수 설정
- `skip()` : 처음 n개 요소를 제외한 스트림 반환

### 매핑

- 특정 데이틀 선택하는 방법
- `map()` : stream 요소를 이용해 새로운 요소 추출
- `flatmap()`
  - `Stream<String[]` 과 같은 내부 요소가 배열로 이루어진 경우 해당 배열요소를 `평면화`(통합) 함

### 검색과 매칭

- 특정 속성이 데이터 집합에 있는지 여부를 검색하는 방법
- `anyMatch()` : 적어도 한개는 일치하지?
- `allMatch()` : 모든 요소가 일치하지?
- `noneMatch()` : 일치하는 놈 없지?
- 해당 기능은 쇼트서킷으로, 자바의 &&, || 과 같은 연산의 기능을 한다.
- `findAny()` : 요소 검색이며 `Optional`을 return 한다.
- `findFirst()` : 첫 번째 요소를 찾는다. 병렬 실행에서는 사용하기 어렵다.

### 리듀싱

- 스트림 요소를 조합해 더 복잡한 질의를 표현하는 방법
- `리듀싱 연산` : 모든 스트림 요소를 처리해서 값으로 도출하는 법으로 `폴드`라고도 한다.
- `reduce(a, b)`
  - a : 초기값, 없을 경우 return 값이 Optional로 제공
  - b : 계산될 람다

### 상태

- 각 연산에 따라 해당 상태를 저장할 필요가 있다.
- map, filter의 경우 해당 요소를 바로 return 하므로 상태 저장이 필요 없음
- reduce, sum, max같은 경우 해당 요소를 저장하여 다음 steam요소에 사용하므로 결과를 누적할 내부 상태가 필요
- 내부 상태의 크기는 한정되어 있으므로 스트림의 크기가 무한이라면 문제가 생길 수 있음

### 숫자형 스트림

- stream의 결과를`sum()`과 같은 함수로 저장한다면 호출 할때 언박싱 비용이 필요

- 기본형 특화 스트림

  - `IntSteam`, `DoubleStream`, `LongStream`
  - `mapToInt()`, `mapToDouble()`,  `mapToLong()`

- 객체 스트림 복원

  - `boxed()`

    ```java
    Stream<Integer> stream = IntStream.boxed();
    ```

- 숫자 범위

  - 특정 범위의 숫자 생성

    ```java
    IntStream numbers = Instream.rangeClosed(1, 100); // 1-100의 숫자 생성
    IntStream numbers = Instream.range(1, 100); // 2-99의 숫자 생성
    ```

### 스트림 만들기

- `Arrays.asList()` 와 같이 스트림도 정적 메서드를 이용해 스트림 생성

  ```java
  Stream<String> stream = Stream.of("Modren", "Java", "In", "Action");
  Stream<String> emptyStream = Stream.empty()
  ```

- null이 가능한 stream 생성

  ```java
  Stream.of("Config", "home", "user")
              .flatMap(key -> Stream.ofNullable(System.getProperty(key)))
              .forEach(System.out::println);
  
  // 아무 값도 출력되지 않음, 해당하는 property가 system에 존재 하지 않음
  ```

- 배열 스트림

  ```java
  int[] numbers = {2, 3,5, 7, 11, 13};
  int sum = Arrays.stream(numbers).sum();
  ```

- 무한 스트림

  - 크기가 고정되지 않은 스트림, `limt()`과 함께 활용
  - 요청할 때 마다 값을 생산하는 무한한 스트림, 언바운드 스트림
  - `Stream.interate()` : 생산된 값을 연속적으로 사용
  - `Stream.generate()` : 생산된 각 값을 연속적으로 계산하지 않음

  