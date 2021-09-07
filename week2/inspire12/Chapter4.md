# 4장 스트림 소개

 빅데이터! 패러다임의 변화

대용량 처리 → 질 

스트림 API의 특징 

- 선언형
- 조립 가능 → 유연성
- 병렬성

스트림 vs 컬렉션 

파이프라이닝 → 쇼트서킷, lazy

> 쇼트서킷: 전체를 처리하지 않아도 결과를 반환할 수 있으면 반환

내부 반복: 반복 알아서 처리하고 결과 스트림값을 어딘가에 저장

> 외부반복: 사용자가 직접 요소를 반복 (for -each)

중간 연산

```jsx
filter  T-> boolean
map     T -> R
limit 
sorted  (T, T) -> int 
distinct 
```

최종 연산

```jsx
forEach   스트림 각 요소를 소비하며 람다 적용
count     요소 갯수
collect   스트림을 리듀스해서 컬렉션을 만듬
```

스트림이 lazy인 이유  → 연산 순서 

[https://dororongju.tistory.com/137](https://dororongju.tistory.com/137)
