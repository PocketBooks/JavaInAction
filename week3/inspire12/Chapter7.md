# 7장 병렬 데이터 처리와 성능

데이터 컬렉션을 병렬로 처리하기 

자바7 이전 병렬처리: 

1. 데이터를 서브 파트로 분할 
2. 분할된 서브파트를 각각의 스레드로 할당
3. 의도치 않은 레이스 컨디션이 발생하지 않도록 동기화를 추가
4. 결과 결합

→ 자바7 포크/조인 프레임워크 추가 로 간결

병렬 스트림 

순차 스트림 → 병렬 스트림

```jsx
public static long parallelSum (long n) {
        return Stream.iterate(1L, i -> i+1)
                .limit(n)
                .parallel()
                .reduce(10L, Long::sum);
    }
```

사실 parallel을 호출해도 스트림 자체에는 아무 변화가 없다. 단지 내부적으로 플레그가 설정 

마지막에 호출된 메서드가 전체 파이프라인에 영향을 미친다 

```java
// parallel 적용
public static long test1(int n){
        return Stream.iterate(1L, i -> i+1)
                .limit(n)
                .sequential()
                .parallel()
                .reduce(0L, (i, j) -> {
                    System.out.println(i + " " + j);
                    return i + j;
                });
    }
// sequential 적용
    public static long test2(int n){
        Long reduce = Stream.iterate(1L, i -> i + 1)
                .limit(n)
                .parallel()
                .sequential()
                .reduce(0L, (i, j) -> {
                    System.out.println(i + " " + j);
                    return i + j;
                });
        return reduce;
    }
```

병렬 스트림은 상태값을 바꾸면 정확한 값이 안나올 수 있다.

병렬 스트림 효과적으로 사용하기 

- 직접 측정하라: 언제나 병렬 스트림이 순차보다 빠른것은 아니다. 병렬 스트림 수행과정은 투명하지 않을 때가 많다
- 박싱을 주의하라: 자동 박싱과 언박싱은 성능을 크게 저하시킬 수 있는 요소다. 되도록 기본형 특화 스트림을 사용하는 것이 좋다
- 순차 스트림보다 병렬 스트림에서 성능이 떨어지는 연산이 있다. (limit, findFirst)
- 스트림에서 수행하는 전체 파이프라인 연산 비용을 고려하라: 처리할 요소 수 N, 요소당 처리비용 Q일 때 Q가 높아지면 병렬 스트림으로 성능 개선 여지가 있다
- 소량의 데이터에서는 병렬 스트림이 도움되지 않는다.
- 스트림을 구성하는 자료구조가 적절한지 확인
- 스트림 특성과 파이프라인의 중간 연산이 스트림의 특성을 어떻게 바꾸는지에 따라 분해 과정의 성능이 달라질 수 있다.
- 최종 연산의 병합 과정 비용을 살펴보라

스트림 소스와 분해성 

분해성이 좋음 

```jsx
ArrayList 
IntStream.range
```

중간 

```jsx
Hashset 
TreeSet
```

안좋음 

```jsx
LinkedList
Stream.iterate
```

## 포크/조인 프레임워크

스레드 풀을 이용하려면 RecursiveTask<R>의 서브클래스를 만들어야한다 

R은 병렬화된 테스크가 생성하는 결과 형식 또는 결과가 없을 때 RecursiveAction 형식이다.

RecursiveTask를 정의하려면 추상 메서드 compute를 구현

→ 분할 후 정복 알고리즘 병렬화 버전 

```java
if (태스크가 충분히 작거나 더 이상 분할할 수 없으면) {
	순차적으로 태스크 계산
} else {
	태스크를 두 서브태스크로 분할 
	태스크가 다시 서브태스크로 분할되도록 이 매서드를 재귀적으로 호출
	모든 서브태스크의 연산이 완료될 때까지 기다림
	각 서브태스크의 결과를 합침
}
```

작업 훔치기 

작업 훔치기 기법에서는 ForkJoinPool의 모든 스레드를 거의 공정하게 분할한다. 

각각의 스레드는 자신에게 할당된 태스크를 포함하는 이중 연결 리스트를 참조하면서 작업이 끝날 때마다 큐의 헤드에서 다른 태스크를 가져와서 작업 처리, 할일이 없어지면 스레드는 유휴 상태로 바뀌는 것이 아니라 **다른 스레드 큐의 꼬리에서 작업을 훔쳐온다.**

천만 개 배열에서 천 개 이상의 서브 태스크를 포크, 대부분의 기기에는 코어가 네 개뿐이므로 천 개 이상의 서브 태스크는 자원만 낭비하는 것처럼 보일 수 있으나 **실제로 코어 개수 관계없이 적절한 크기로 분할된 많은 태스크를 포킹하는 것이 바람직**

Spliterator 인터페이스: 분할할 수 있는 반복자 // 병렬 작업에 특화 

```java
public interface Spliterator<T> {
// 순차적으로 소비하면서 탐색해야 할 요소가 남아있으면 참을 반환
	boolean tryAdvance(Consumer<? super T> action; 
// 일부 요소를 분할해서 두번째 Spliterator를 생성하는 메서드
	Spliterator<T> trySplit(); 
//  메서드로 탐색해야 할 요소 수 정보를 제공
	long estimateSize();  

	int characteristics();  
}
```

 

```java
public class WordCounterSpliterator implements Spliterator<Character> {
    private final String string;
    private int currentChar = 0;

    public WordCounterSpliterator(String string) {
        this.string = string;
    }
    
    @Override
    public boolean tryAdvance(Consumer<? super Character> action) {
        action.accept(string.charAt(currentChar++)); // 현재 문자를 소비한다
        return currentChar < string.length(); // 소비할 문자가 남아있으면 true를 반환
    }

    @Override
    public Spliterator<Character> trySplit() {
        int currentSize = string.length() - currentChar;
        if (currentChar < 10) {
            return null;
        }
        for (int splitPos = currentSize / 2 + currentChar; splitPos < string.length(); splitPos++) {
            if (Character.isWhitespace(string.charAt(splitPos))) {
                Spliterator<Character> spliterator = new WordCounterSpliterator(string.substring(currentChar, splitPos));
                currentChar = splitPos;
                return spliterator;
            }
        }
        return null;
    }

    @Override
    public long estimateSize() {
        return string.length() - currentChar;
    }

    @Override
    public int characteristics() {
        return ORDERED + SIZED + SUBSIZED + NONNULL + IMMUTABLE;
    }
}
```
