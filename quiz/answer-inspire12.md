1. Collector<T, A, R> 인터페이스 세계 파라미터의 역할은?
    1. T: 수집될 스트림 항목의 제네릭 형식(input)
    2. A: 누적자, 즉 수집 과정에서 **중간 결과를 누적**하는 객체의 형태
    3. R: 수집 **연산 결과** 객체의 형식 (대게 컬렉션 형식)
2. 스트림 종단 계산에서 partitioningBy(Prediction<>, groupingBy()) 함수의 역할은, (리턴값을 통해 표현)? 
    1. 분할, 그룹핑 ex) Map<Boolean, Map<A, List<B>>> 형태 
3. peek() 과 foreach() 의 차이는?
    1. peek은 stream 을 소비하지 않는 중간 단계 연산이며 foreach()는 stream을 소비하는 종단 연산다.  따라서 peek은 마지막에 종단연산이다.
    
4. 스트림을 한번만 소비하게 만든 이유?
    1. 스트림은 내부반복(반복을 알아서 처리하고 결과 스트림 값을 어딘가 저장) → 반복을 숨김으로서 컴퓨터에 맞김, 하드웨어 최적화, 병렬
    2. 만약 스트림이 I/O 채널일 경우 에러가 생김 
5. 함수형 프로그래밍에서 참조 투명성이란?
    1. **부작용을 감춰**야한다는 제약에서 참조 투명성 개념으로 귀결 
    2. 같은 인수로 함수 호출시 **항상 같은 결과**를 **반환**해야한다 
6. 함수형 프로그래밍에서 영속이란? 
    1. 결과 자료구조를 바꾸지 말라
    2. 불변성 (데이터베이스의 영속과는 다름)
7. 함수형 프로그래밍에서 두개 이상의 함수를 인수로 받아서 다른 함수를 반환하는 메서드나 함수
8. 함수형 프로그래밍에서 커링이란?
    1. 변환기, 로직을 재활용할 수 있다
9. 꼬리 재귀란 무엇이고 장점은?
    1. 재귀 호출을 함수 마지막에 이루어지도록 함
    2. 최신 JVM에선 여러 스택프레임이 아니라 하나의 스택프레임을 활용하도록 최적화 여지
10. Predict<A> predict 와 Function<A, Boolean> 의 차이
    1. Boolean은 박싱된 형태로 처리가 됨, 박싱된 변수는 성능에 영향을 주게 되는데 Predict는 boolean (primitive)로 사용 가능
11. Consumer / Runnable / Supplier  차이 
    1. Consumer는 리턴값만
    2. Suppiler는 인수를 받고
    3. Runnable은 둘다 가능한 형태
12. CompletableFuture 와 Future의 차이는? 
    1. 둘다 비동기 처리가 가능, CompletableFuture는 함수형으로 받아 처리해서 간결하게 표현 가능 
13. Flow API 의 네 개 인터페이스를 설명하라 
- Publisher  발행자

```java
@FunctionalInterface
public interface Publisher<T> {
	void subscribe(Subscriber<? super T> s;
}
```

- Subscriber  구독자

```java
public interface Subscriber<T> {
	void onSubscribe(Subscription s);
	void onNext(T t);
	void onError(Throwable t);
	void onComplete();
}
```

- Subscription  구독 (발행자, 구독자를 연결)

```java
public interface Subscription {
	void request(long n);
	void cancel();
}
```

- Processor 이벤트 변환 단계, 에러 회복등

```java
public interface Processor<T, R> extends Subscriber<T>, Publisher<T> {}
```
