# Stream and Collection

- Collection은 생성부터 하지만 Stream은 lazy하게 생성한다
- Stream은 Index가 없으니 한번만 접근할 수 있음
- Collection은 외부반복 Stream은 내부반복

### Stream 연산

- Stream은 중간 연산(intermediate operation), 최종 연산(terminal operation) 둘로 나뉘는데
최종연산을 할때만 Stream이 처리됨, 즉 Lazy함
- primitive 타입용 Stream 따로있음 IntStream... LongStream...

### Stream 연산 고려 요소

- 쇼트 서킷
    - `takeWhile`, `drpoWhile`, `findFirst`, `findAny`, `allMatch`, `noneMatch`, `anyMatch`
- 순서와 정렬이 필요한지
- 연산의 상태/비상태

### Collector

어우.. 이부분은 한번 더봐야할듯..? 일단 요약하자면 결과만들고 Container들을 알잘딱 지지고볶는건 Collector

- collect : Stream API 최종 연산
- collection : 컬렉션
- collector : collect 함수 파라미터인 Collelctor Interface

## 병렬 데이터 처리와 성능

### Stream을 느리게 하는 것들

- 박싱과 언박싱

### 병렬을 느리게 하는 것들

- 코어간 스위칭 비용 → 하드웨어적인것 소프트웨어로 해결이 안됨
- 순서와 정렬
    - 순서와 정렬이 필요하다는 것은 상태가 필요하다는 의미이기도함. 즉 이전 결과에 대해 종속성을 가지게 되므로 느려짐
- 분할 (split)
    - 어떻게 분할해야하는지 사이즈라던가.. 정확히 알 수 없으면 매우 느려짐
- 병합 (collect, combine)

### 병렬이 유효한 경우

- 요소당 작업이 길 경우

```java
package me.labyu;
	
import java.util.List;
import java.util.Map;
import java.util.function.Predicate;

import static me.labyu.StyleFilterFactory.*;
import static java.util.Comparator.comparing;
import static java.util.stream.Collectors.groupingBy;

class Style {
    private final String name;
    private final int like_count;
    private final int comment_count;
    private final int visit_count;

    public Style(String name, int like_count, int comment_count, int visit_count) {
        this.name = name;
        this.like_count = like_count;
        this.comment_count = comment_count;
        this.visit_count = visit_count;
    }

    public String getName() {
        return this.name;
    }

    public int getStyleScore() {
        return like_count + comment_count + visit_count;
    }
}

interface StyleFilter extends Predicate<Style> {
}

class StyleFilterFactory {
    public static StyleFilter nameFilter() {
         return (style) -> !style.getName().equals("one");
    }
    public static StyleFilter scoreFilter() {
        return (style) -> style.getStyleScore() > 100;
    }
    public static StyleFilter someFilter() {
        return (style) -> true;
    }
}

class ScoredStyle {
    private final String name;
    private final int score;

    public ScoredStyle(Style style) {
        this.name = style.getName();
        this.score = style.getStyleScore();
    }

    public String getName() {
        return this.name;
    }

    public int getScore() {
        return this.score;
    }

    public String toString() {
        return this.name + " = " + this.score;
    }
}

public class StyleRankGenerator {
    public static void main(String [] args) {
        StyleRankGenerator generator = new StyleRankGenerator();
        generator.generateFeed();
    }

    public void generateFeed() {
        List<Style> styles = this.fetchStyles();

        Map<String, List<ScoredStyle>> styleMap = styles.stream()
                .filter(someFilter())
                .filter(nameFilter().and(scoreFilter()))
                .map(ScoredStyle::new)
                .sorted(comparing(ScoredStyle::getScore).reversed())
                .collect(groupingBy(ScoredStyle::getName));
    }

    private List<Style> fetchStyles() {
        return List.of(
                new Style("one", 10, 23, 124),
                new Style("two", 10, 43, 23),
                new Style("three", 23, 23, 54),
                new Style("four", 11, 3, 34),
                new Style("like", 3, 8, 12),
                new Style("six", 22, 11, 48)
        );
    }
}
```
