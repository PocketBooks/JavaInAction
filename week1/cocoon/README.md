## Chapter1. 자바 8,9, 10, 11 : 무슨 일이 일어나고 있는가?

자바 8 이전에는 멀티코어에서 환경을 제대로 활용하기 위해서는 쓰레드를 활용한 방법이 있었다.

하지만, 쓰레드를 사용해 본 사람들이라면 느낄 수 있지만 쓰레드를 관리하는 일은 어려우며 예상치 못한 많은 문제들을 직면할 수 있기 때문에 쓰레드를 활용한 코딩은 많은 생각을 가지고 설계가 필요하다.

그렇기 때문에 자바는 계속해서 병렬 실행 환경을 쉽게 관리하기 위해 진화해 왔다.

- 자바 1.0 : 쓰레드와 락
- 자바 5 : 쓰레드 풀, 병렬 실행 컬렉션
- 자바 7 : 포크/조인 프레임워크

이렇게 자바는 병렬 실행을 쉽고 간편하게 사용하기 위해 많은 기능들을 추가해 왔고 자바 8에 이르러서는 새로운 방식으로 병렬 실행을 단순한 방식으로 제공할 수 있도록 했다. 또한 자바 9에서는 RxJava를 표준적으로 적용한 리액티브 프로그래밍을 제공한다.

자바 8의 새로 추가된 기법들을 요약해서 살펴보면 아래와 같다.

- 스트림 API
- 메서드에 코드를 전달하는 기법
- 인터페이스의 디폴트 메서드

### 스트림 API

스트림이란 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임이다. 입력 스트림에서 데이터를 한 개씩 읽어 들여 출력 스트림으로 데이터를 한 개씩 기록한다. 자바 8에서는 `java.util.stream` 패키지에 스트림 API가 추가 됐다.

- Stream<T>

이 스트림 API는 파이프 라인을 만드는데 많은 메서드를 제공한다. 파이프 라인을 통해 기존에는 한 번에 한 항목을 처리했지만, 이제 자바 8에서는 하려는 작업을 고수준으로 추상화하여 일련의 스트림으로 만들어 처리할 수 있다. 또한 해당 스트림 파이프라인을 이용해 입력 부분을 여러 CPU 코어에 할당하여 쓰레드라는 복잡한 작업을 거치지 않고 병렬 작업을 쉽게 할 수도 있다.

한 예로 한 CPU는 리스트의 앞부분을 한 CPU는 리스트의 뒷부분을 처리하도록 요청할 수 있다.이를 `포킹단계`라고 한다. 



### 동작파라미터화로 메서드에 코드 전달

자바 8 이전에는 메서드를 다른 메서드에 전달할 방법이 없었다. 

이러한 말도 안되는 방법을 `동작 파라미터화` 라고 부른다. 연산의 동작을 파라미터화 하여 코드를 전달한다는 스트림 API의 사상이 해당 기능을 탄생하도록 하였다.



이 기능을 이해하기 위해선 알아야할 개념이 한가지 있다.

동작파라미터화를 이용해 메서드에 전달을 한다고 했는데 다시 말하자면 동작이 서술된 함수를 파라미터로 넣어준다는 뜻이며 함수를 값으로 취급한다는 소리이다. 여태껏 자바는 기본 값(int, double 등), 혹은 인스턴스 등을 파라미터로 넘겨 값을 전달했다. 근데 갑자기 함수를 넘긴다니 이건 무슨 뚱단지 같은 소리인가?

이번 자바8 의 핵심은 이 값에 대한 패러다임을 바꾸는 것이다.

- 일급 시민
- 이급 시민

기존의 파라미터로 넘길 수 있는 값들은 일급(first-class) 값(혹은 시민)이라 불렀으며, 그 이외에 전달할 수없는 구조체들은 이급값(혹은 시민)이라고 불렸다. 이 패러다임을 바꾼것이다. 

여기서 자바 8은 메서드를 일급시민으로 변경하여 메서드를 값으로 취급할 수 있게끔 했다. 이 개념을 `메서드 참조` 라고 한다.  또한 람다(익명함수)를 포함하여 함수도 값으로 취급할 수 있게 되었다. 

하지만 메서드를 값으로 전달하기 위해 계속해서 메서드를 선언하는 일은 상당히 귀찮다. 그렇기 때문에 람다를 이용해 한번 사용할 메서드를 간결하게 넘겨 줄 수 있다.

```java
public boolean isSame(Apple a) {
  return GREEN.equals(a.getColor);
}
filterApples(inventory, isSame());

  
filterApples(inventory, (Apple a) -> GREEN.equals(a.getColor()));
```

해당 코드를 보면 메서드를 정의하지 않고 아주 간단히 코드를 넘기는 것을 볼 수 있다.



### 인터페이스의 디폴트 메서드

동작이 서술된 인터페이스를 본적이 있는가? 아마 자바8 이전에는

```java
void printf();
void println();  
```

이러한 형태의 인터페이스만 보았을 것이다.

자바 8은 구현 클래스에서 구현하지 않아도 되는 메서드를 인터페이스에 추가할 수 있는 기능을 제공한다.

메서드의 본문은 클래스의 구현이 아닌, 인터페이스의 일부로 포함된다.

```java
default void sort(COmparator<? super E> c) {
  Collections.sort(this, c);
}
```



## Chap2. 동작 파라미터화 코드 전달하기

chap1 을 통해 동작 파라미터화 코드에 대해서 배웠다. 동작 파라미터화 코드는 아직은 어떻게 실행할지 결정하지 않은 코드를 의미한다. 파라미터로 코드를 넘겨 줬을 뿐이지 나중에 프로그램을 통해 호출된다.

즉, 코드의 실행은 나중으로 미뤄진다는 뜻이다. 그렇다면 어떠한 동작을 코드로 넘길 때 특정 동작만 하는것이 아닌 인수에 의해 여러가지 동작을 할수 있다면 조금더 효율적이지 않을까? 이렇게 변화하는 요구사항에 유연하게 대응할 수 있게 동작 파라미터화를 통해 유연성 있게 구현할 수 있다.

### 변화하는 요구사항에 대응

```java
public static List<Apple> filterApples(List<Apple> inventory, Color color,
                                       int weight, boolean flag) {
  List<Apple> result = new List<>();
  for (Apple apple : inventory) {
    if (apple.getWeight() > weight) {
      result.add(apple);
    }
  }
}  
```

위와 같이 어떠한 조건에 의해 결과 list에 담기는 Apple class가 있다고 보자

해당 `if()` 안의 조건이 계속해서 변경된다고 했을 때 해당 내부 조건만을 코드로 넘겨준다면? 계속해서 재사용성을 가지고 해당 메서드를 사용할 수 있을 것이다.

```java
public class AppleHeavyPredicate implements ApplePredicate {
  public boolean test(Apple apple) {
    return apple.getWeight() > 150;
  }
}

public static List<Apple> filterApples(List<Apple> inventory,
                                       ApplePredicate p) {
  List<Apple> result = new List<>();
  for (Apple apple : inventory) {
    if (p.test(apple)) {
      result.add(apple);
    }
  }
}
```

혹은 클래스를 선언하지 않고 익명 클래스를 사용해 해당 코드를 나타낼 수 도 있다.

```java
List<Apple> result = filterApples(inventory, new ApplePredicate() {
  public boolean test(Apple apple) {
    return RED.eqauls(apple.getColor());
  }
});
```

더 간단하게 람다로 표현한다면 아래와 같다.

```java
List<Apple> result = filterApples(inventory, (Apple apple) -> RED.eqauls(apple.getColor());
```



## Chap3. 람다 표현식

2장에서 단계적으로 변화하는 요구사항에 대한 구현을 해보았고 최종적으로 람다를 통해 깔끔한 코드 구현을 맛보았다. 람다 표현식은 익명 함수를 단순화 한것으로 파라미터 리스트, 바디, 반환 형식, 발생할 수 있는 예외 리스트를 포함한다. 자바 8에서는 이를 이용해 간결한 코드 구현을 제공한다.

그렇다면 이런 람다 형식이 있다는 것은 알겠는데 어디에 사용되는 것일까?

답은 함수형 인터페이스라는 문맥에서 람다 표현식을 이용할 수 있다.

### 함수형 인터페이스

그럼 함수형 인터페이스는 무엇인가? 바로 2장에서 배운 오직 하나의 추상메서드 가진 메서드를 말한다.

그 예로는 `Comparator` 혹은 `Runnable` 인터페이스가 있다.

책에서는 인터페이스에 여러 default 메서드가 선언되고 단 하나의 추상 메서드가 있다면 이것도 함수형 인터페이스라고 한다. 그렇다면 abstract class는 안될까? 물론 인터페이스가 아니지만 두 모양을 보고 비교하면 똑같은데 말이다. 

또한 함수형 인터페이스의 추상 메서드 시그니처는 람다 표현식의 시그니처를 가르킨다.

그렇기 때문에 하나의 추상 메서드가 있을 경우에만 람다 표현식을 사용할 수 있다. 만약 여러개라면 어떤 시그니처를 의미하는지 알수 없기 때문에 람다를 적용할 수 없을 것이다.

### 함수형 인터페이스의 사용

- Predicate

  ```java
  @FunctionalInterface
  public interface Prdicate<T> {
    boolean test(T t);
  }
  // 예제
  (List<String> list) -> list.isEmpty();
  ```

- Consumer

  ```java
  @FunctionalInterface
  public interface Consumer<T> {
    void accept(T t);
  }
  // 예제
  (Apple a) -> System.out.println(a.getWeight());
  ```

- Function

  ```java
  @FunctionalInterface
  public interface Function<T,R> {
    R apply(T t);
  }
  // 예제
  (String s) -> s.length();
  ```

- Supplier

  ```java
  @FunctionalInterface
  public interface Supplier<T> {
    T get();
  }
  // 예제
  () -> new Apple(10);
  ```

여러 함수형 인터페이스 예제를 살펴 보았다.  이외에도 여러 함수형 인터페이스가 있으며 직접 선언하여 사용할 수 있다. 또한 함수형 인터페이스는 **예외를 던지는 동작을 허용하지 않는다**. 그렇기에 함수 내부에 try ~ catch를 선언하거나 예외를 선언하여 던지는 함수형 인터페이스를 직접 정의해야 한다.



또한 여태 살펴본 람다의 형태는 자신의 바디 내부에서 인수를 사용하였다. 하지만 익명 클래스 처럼 자유변수를 활용할 수가 있는데 이 같은 동작을 `람다 캡처링(capturing lambda)` 이라고 한다.

이 자유 변수는 final 형태 혹은 final 처럼 사용되어져야 한다. 즉, 한번 선언되면 선언된 값이 변경되지 않고 그대로 유지되어야 한다는 뜻이다. 그 이유는 해당 변수는 지역 변수로 stack에 존재 하므로 자신을 정의한 쓰레드와 운명을 같이 한다. 그렇기 때문에 **쓰레드가 사라져 변수 할당이 해제 되었는데도 람다를 실행하는 쓰레드에서 해당 변수에 접근 할 수 있기 때문**에 원래 변수에 접근을 허용하는 것이 아니라 **자유 지역 변수의 복사본을 제공**하는 것이다.



### 메서드 참조

자바 8에서는 또 하나의 새로운 기능인 `메서드 참조` 라는 기능이 있다.

예제 코드를 통해 알아 보자

```java
inventory.sort((Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()));

// 메서드 참조 적용
inventory.sort(comparing(Apple::getWeight());
```

결과적으로 `(Apple a) -> a.getWeight()` 를 축약한 형태가 되었다. 이렇게 메서드 참조를 이용해 람다 표현식을 만들 수 있다. 

### 메서드 참조를 만드는 방법

- 정적 메서드 참조

  - Integer의 parseInt 메서드는 `Integer::parseInt` 로 표현한다.

- 다양한 형식의 인스턴스 메서드 참조

  - String의 length 메서드는 `String::length` 로 표현한다.

    ```java
    (String s) -> s.length();
    ```

- 기존 객체의 인스턴스 메서드 참조

  - Transaction의 인스턴스인 expensiveTransaction이 있고 해당 메서드인 geValue가 있다면,

    `expensiveTransaction::geValue` 로 표현한다.

    ```java
    () -> expensiveTransaction.getValue();
    ```

- Quiz

  ```java
  (String s) -> Integer.parseInt();
  => Integer::parseInt
  (list, element) -> list.contains(element);
  => List::contains
  (String string) -> this.startsWithNumber(string);
  => this::startsWithNumber
  ```

- 생성자 참조

  ```java
  //no arg construct
  Supplier<Apple> c1 = Apple::new;
  Apple a1 = c1.get();
  
  //arg construct
  Function<Integer, Apple> c2 = Apple::new;
  Apple a2 = c2.apply(100);
  ```

  







