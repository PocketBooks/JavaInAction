# 5장 스트림 활용

takeWhile 
dropWhile 

둘 다 이미 정렬되어있을 경우 사용 

쇼트서킷이 들어간 stream 함수

```jsx
allMatch
noneMatch
findFirst
findAny
```

map vs  flatmap

map → Stream<Stream<Integer[]>>

flatmap → Stream<Integer[]>

reduce 메서드의 장점과 병렬화 

내부 반복이 추상화되어 내부 구현에서 병렬로 reduce를 실행 가능

 - 외부 반복에서는 sum 변수를 공유해야 해서 쉽게 병렬화하기 어렵다 
 → 강제로 별렬화 해도 스레드간 외부변수 접근으로 인한 소모적인 경쟁이 이점을 상쇄한다

스트림 연산

상태 없음: filter, map

상태 있음: reduce, sum, max 연산 결과를 누적할 내부 상태가 필요, sorted, distinct 과거 상태를 알아야함

기본형 특화 스트림 

IntStream, DoubleStream 등

mapToInt, mapToDouble 등 

map과 같은 기능을 수행하지만 특화된 스트림(IntStream, DoubleStream)을 반환 

요약

Stream으로 복잡한 데이터 처리를 질의로 처리할 수 있다

소스가 정렬되어 있으면 takeWhile, dropWhile 메소드가 효과적

map, flatMap 메서드로 스트림의 요소를 추출하거나 변환 

findFirst, findAny 메서드로 스트림의 요소 검색 가능, allMatch, noneMatch, anyMatch 메서드를 이용해서 주어진 프레디케이트와 일치하는 요소를 스트림에서 검색 

퀴즈

- findFist와 findAny는 어디서 쓰는게 좋은가?
    - 병렬일 경우 첫번째를 찾는 findFirst는 비용이 올라감, 따라서 병렬일 때 굳이 첫 요소가 아니면 findAny 권장
