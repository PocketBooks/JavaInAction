# 6장 스트림으로 데이터 수집

실용적인 파트 

collect는 스트림의 요소를 요약 결과로 누적하는 다양한 방법(컬렉터)을 인수로 갖는 **최종연산**

```jsx
질의 형태로 연산
lazy 반복자 (내부 반복)
```

- 중간연산: 스트림 파이프라인을 구성하며, 스트림의 요소를 소비하지 않는다 (4,5장)
- 최종연산: **스트림을 소비**해서 **최종 결과**를 도출

이 장에서 나가는 것들 

- 스트림 요소를 하나의 값으로 요약 → 리듀스
- groupingBy 로 스트림의 요소를 그룹화, partitioningBy로 스트림 요소를 분할
- 컬렉터는 다수준의 그룹화, 분할, 리듀싱 연산에 적합하게 설계
- 커스텀 컬렉터 개발 가능

## 문자열 연결: joining

```jsx
menu.stream().map(Dish::getName).collect(joining(", "));
```

## 범용 리듀싱

첫번째 인수: 리듀싱 연산의 **시작값**이거나 스트림에 인수가 없을 때는 반환값  → 초깃값

두번째 인수: 요리를 칼로리 정수로 반환할 때 사용한 변환 함수  → 대상

세번째 인수: 같은 종류의 두 항목을 하나의 값으로 더하는 BinaryOperator → 계산 

cf) 하나의 인수만 받는 reduce는 optional 로 넘어감(초깃값이 없음)

Collect Vs Reduce 

reduce로 collect 구현 가능 

```jsx
의미론적, 실용성 문제 발생 
collect -> 결과를 컨테이너로 
reduce는 하나로 도출하는 불변형 연산 
```

리듀싱 내부적으로 자신을 **그대로 반환하는 항등함수를  두번째 인수로** 받는 상황

그룹화로 스트림 항목 분류하는 과정 

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/24cf2b19-cec8-4567-aff9-2724a94db659/Untitled.png)

퀴즈

reducing: input 타입과 output 타입이 같아야한다. 

## 그룹화: Collectors.groupingBy

```jsx
Map<Dish.Type, List<Dish>> caloricDishesByType =
                menu.stream()
                        .collect(groupingBy(Dish::getType,
                                filtering(dish -> dish.getCalories() > 500, toList())));
```

collectingAndThen 으로 반환한 결과를 다른 형식으로 활용 

분류 함수로 원래 스트림을 분할 → 두번째 컬렉터로 각각의 서브 스트림을 **독립적**으로 처리 → (collectingAndThen //  리듀싱 컬렉터는 가장 칼라리가 높은 요리를 Optional로 감싸서 반환 → collectingAndThen 컬렉터는 Optional에서 값을 추출해서 반환 )
→ 그룹화 map으로 처리후 결과 반환 

## 분할

분할의 장점: **참, 거짓 두가지 요소의 스트림 리스트를 모두 유지** 

```jsx
Map<Boolean, List<Dish>> partitionedMenu =
                menu.stream()
										.collect(partitioningBy(Dish::isVegetarian));
```

다수준 분할 

퀴즈

→ partitioningBy는 predicate를 요구함 

```jsx
menu.stream().collect(partitioningBy(Dish::isVegetarian, partitioningBy(d -> d.getCalories() > 500)));
// menu.stream().collect(partitioningBy(Dish::isVegetarian, partitioningBy(Dish::getCalories))); compile fail
```

Collector 인터페이스 

```jsx
public interface Collector<T, A, R> {
    Supplier<A> supplier();

    BiConsumer<A, T> accumulator();

    BinaryOperator<A> combiner();

    Function<A, R> finisher();

    Set<Collector.Characteristics> characteristics();
}
```

- supplier: 새로운 결과 컨테이너 만들기 → 빈 **결과**로 이루어진 supplier를 반환해야함

```jsx
public Supplier<List<T>> supplier() {
	return () -> new ArrayList<T>();
}
```

- accumulator: 결과 컨테이너에 요소 추가 → 리듀싱 연산을 수행하는 함수를 반환 (요소가 남아있을 때까지 반복)

```jsx
public BiConsumer<List<T>, T> accumulator() {
	return (list, item) -> list.add(item);
}
```

- finisher: 최종 변환값을 결과 컨테이너로 적용하기

```jsx
public Function<List<T>, List<T>> finisher() {
	return Function.identity();
}
```

- combiner: 두 결과 컨테이너 병합  (병렬로 처리할 때 스트림 분할후 합칠때 사용) → Spliterator

```jsx
public BinaryOperator<List<T>> combiner() {
	return (list1, list2) -> {
			list1.addAll(list2);
			return list1;
	}
}
```

- Characteristics: 컬렉터 연산 형식을 정의 (enum)
    - UNORDERED: 리듀싱 결과는 스트림 요소의 방문 순서나 누적 순서에 영향을 받지 않는다
    - CONCURRENT: 다중 스레드에서 accumulator 함수를 동시에 호출할 수 있으며 이 컬렉터는 스트림의 병렬 리듀싱을 수행할 수 있다
    - IDENTITY_FINISH: 생략가능, finisher 메서드가 반환하는 함수는 단순히 identity를 적용할 뿐이므로 생략 가능

Custom Collector: Rank 넣기

```java
@Getter
@AllArgsConstructor
public class Rank<T> {
    @Setter
    int rank;
    T content;
}
```

```java
public static void addRank() {
  List<Batter> batters = Arrays.asList(
          new Batter("이승엽", 5),
          new Batter("김태균", 3),
          new Batter("이대호", 6),
          new Batter("나성범", 5)
  );

  Comparator<? super Batter> comparator = Comparator.comparingInt(Batter::getHr).reversed();
  List<Rank<Batter>> batterRanks = batters.stream()
          .sorted(comparator)
          .map(b -> new Rank<>(0, b))
          .collect(new RankCollector<>(comparator));

  batterRanks.forEach(batterRank -> System.out.println(batterRank.getRank() + " " + batterRank.getContent().getName()));
}
@ToString
@Getter
@AllArgsConstructor
public static class Batter implements Comparable<Batter> {
    String name;
    int hr; // 홈런

    @Override
    public int compareTo(Batter o) {
        return o.getHr() - this.hr;
    }
}
}
```

```java
public class RankCollector<T> implements Collector<T, List<Rank<T>>, List<Rank<T>>> {

    private static final Set<Characteristics> CHARACTERISTICS = Collections.unmodifiableSet(EnumSet.of(Characteristics.IDENTITY_FINISH));
    private final Comparator<? super T> comparator;

    public RankCollector(Comparator<? super T> comparator) {
        this.comparator = comparator;
    }

    @Override
    public Supplier<List<Rank<T>>> supplier() {
        return ArrayList::new;
    }

    @Override
    public BiConsumer<List<Rank<T>>, T> accumulator() {
        return (List<Rank<T>> list, T current) -> {
            if (list.isEmpty()) {
                list.add(new Rank<T>(1, current));
            } else {
                Rank<T> lastElement = list.get(list.size() - 1);
                int rank;
                if (comparator.compare(lastElement.getContent(), current) == 0) {
                    rank = lastElement.getRank();
                } else if (comparator.compare(lastElement.getContent(), current) < 0) {
                    rank = list.size() + 1;
                } else {
                    throw new RuntimeException(); // 정렬이 되어있어야함
                }
                list.add(new Rank<>(rank, current));
            }
        };
    }

    @Override
    public BinaryOperator<List<Rank<T>>> combiner() {
        return (List<Rank<T>> list1, List<Rank<T>> list2) -> {
            list1.addAll(list2);
            return list1;
        };
    }

    @Override
    public Function<List<Rank<T>>, List<Rank<T>>> finisher() {
        return Function.identity();
    }

    @Override
    public Set<Characteristics> characteristics() {
        return CHARACTERISTICS;
    }
}
```
