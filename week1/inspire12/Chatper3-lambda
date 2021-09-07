3장 람다 표현식

- 익명
- 함수
- 전달
- **간결성**

```jsx
(parameters) -> expression
(parameters) -> {statements;}
() -> {}
() -> "Raoul"
() -> {return "Mario";}
(Integer i) -> return "Alan" + i;
(String s) -> {"Iron Man";} 

1,2,3만 유효
```

함수형 인터페이스 

왜 함수형 인터페이스를 인수로 받는 메서드에만 람다 표현식을 사용할 수 있을까?

@FunctionalInterface 함수형 인터페이스를 가리키는 어노테이션

→ 추상 메서드가 한 개만 허용

<T>  

함수형 인터페이스의 추상메서드 시그니처 = 함수 디스크립터 

- Predicate

```jsx
@FunctaionalInterface
public interface Predicate<T> {
	boolean test(T t);
}
```

- Consumer

```jsx
@FunctaionalInterface
public interface Consumer<T> {
	void accept(T t);
}
```

- Function

```jsx
@FunctaionalInterface
public interface Function<T, R> {
	R apply(T t);
}
```

퀴즈 

1. T → R 
2. (int, int) → int 
3. T → void
4. () → T
5. (T, U) → R 

메서드 참조: 기존 정의를 재활용해서 람다처럼 전달 

람다 표현식 전체가 함수형 인터페이스의 인스턴스로 취급 

**람다 표현식을 조합**

Predicate

- negate
- and
- or

Function

- andThen
- compose

지역 변수 제약 

인스턴스 변수는 힙에 저장되는 반면 지역변수는 스택에 저장

람다에서 지역 변수에 바로 접근할 수 있다는 가정하에 람다가 스레드에서 실행된다면 변수를 할당한 스레드가 사라져서 변수 할당이 해제 되었는데도 람다를 실행하는 스레드에서는 해당 변수에 접근하려 할 수 있다 

따라서 자바 구현에서는 원래 변수에 접근을 허용하는 것이 아니라 자유 지역 변수의 복사본을 제공한다. 복사본의 값이 바뀌지 않아야 하므로 지역 변수에는 한번만 값을 할당해야한다 

→ 람다에 지역변수를 넘기면 값 첫 할당시 

function 조합 

compose → 파라미터로 넣어주는 함수를 먼저 실행 

andThen → 함수 구현체를 먼저 실행
