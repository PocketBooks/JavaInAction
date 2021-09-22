## Chap.15 CompletableFuture와 리액티브 프로그래밍 컨셉의 기초

- 병렬 실행과 자바의 발전
  - 하드웨어 : 멀티코어 프로세서의 발전
    - Future
    - fork/join 
    - 병렬 스트림
  - 소프트웨어 : 마이크로서비스 아키텍처, 메시업(mesh-up) 형태의 Application (의존성 증가)

- Future

  - 비동기 연산의 결과
  - `get(duration, Timeunit) `
  - `isCancelled()`
  - `isDone()`

- Executor, ThreadPool

  - Executor

    - `execute()`를 통해  runnable을 다른 쓰레드에서 실행할 수 있음

  - ExecutorService

    - Executor의 라이프 사이클 관리 및 Callable을 통해 결과값을 return 받을 수 있다.
    - `shutdown()`, `awaitTerminate(Timeout, TimeUint)`

  - ThreadPool

    - 작업을 처리하기 위한 쓰레드를 미리 생성

      - 쓰레드의 생성 및 종료에 따른 오버헤드 제거 
      - 무한히 생성되는 쓰레드의 생성을 막아 OutOfMemoryError 제거

    - `newFixedThreadPool()` : 고정된 크기의 쓰레드를 생성

    - `newChacedThreadPool()` : 작업 할당이 가능한 쓰레드가 없을 경우 새로운 쓰레드를 생성하여 작업할당, 지정시간동안 쓰레드에 작업할당이 되지 않으면 해당 쓰레드는 terminated됨

    - `newSingleThreadExecutor()` : 하나의 쓰레드만 생성

    - `newScheduledThreadPool()` : 주기적으로 실행하는 작업 쓰레드 생성, 지정시간 후에 작동이 되도록 설정도 가능

    - `workStealingThreadPool()` : 자바8에 추가되어 여러개의 Queue를 통해 작업을 할당하여 병렬 작업 수행

      ![사진](https://lh3.googleusercontent.com/proxy/BVPxH0UnJ7cJGyQGquiz9Y5Z2YT6Vf-gquwYOOfco_jlix_Sk72s7E_KfgoKRYmYhmeAdcBILdtw-wAfOKsHW9WAZNtAJ4h8BWhqeR0yVN95hdGre_9Fz8jmcFubt1z43y8)

- 스레드풀의 단점

  - 잠자기, I/O, 네트워크 연결등으로 block되는 task의 경우, thread의 비효율 사용
  - 메인 스레드가 종료되기 전 다른 스레드들이 종료되도록 관리 필요 (혹은 deamon 생성)
    - 종료되지 못한 스레드에 의한 애플리케이션 크래시
    - I/O작업 스레드가 제대로 종료되지 못해 데이터 일관성 파괴

- 동기 API와 비동기 API

  - 완전한 병렬 프로그래밍을 위한 비동기 API의 사용
    - 동기 API 사용 시 메인 스레드는 작업의 종료를 기다리게 된다.
  - tomcat vs netty

- 리액티브 형식 API

  - 리액티브?
    - 이벤트의 발생에 대한 반응 / 작업
    - callback을 이용해 결과가 준비되면 결과를 호출
  - Observer pattern
    - Publisher
    - Subscriber
    - Subscripton
    - Processor

- 비동기 API의 예외처리

  - 호출자와 다른 스레드에서의 예외 발생

  - 예외가 발생했을 때 수행할 콜백 인자 (`Subscribier<T>`)

    | Method       | 의미                                                         |
    | ------------ | ------------------------------------------------------------ |
    | onComplete() | 값을 다 소진했거나, 에러가 발생하여 더 이상 처리할 데이터가 없을 때 |
    | onError()    | 도중에 에러가 발생했을 때                                    |
    | onNext()     | 값이 있을때                                                  |

- CompletableFuture

  - Future의 구현체
  - 실행할 코드 없이 Future를 만듦
  - `compelete()` 메서드를 이용해 다른 스레드가 이를 완료할 수 있으며, `get()` 으로 값을 얻을 수 있음

- 역압력(back-pressure)

  - 생산만큼 배출이 원할하지 않아 해당 부하를 조절하는 기능



## Chap.16 CompletableFuture : 안정적 비동기 프로그래밍

- 비동기 애플리케이션 구현

  - 동기

    ```java
    public double getPrice(long id) {
      return ProductService.getProduct(id).getPrice();
    }
    ```

  - 비동기

    ```java
    public Future<Double> getPriceAsync(long id) {
     CompletableFuture<Double> futurePrice = new CompletableFuture<>;
      new Thread( () -> {
        double price = ProductService.getProduct(id).getPrice();
        futurePrice.complete(price);
      }).start;
     return futurePrice; 
    }
    ```

  - 구현

    ```java
    public static void main(String[] args) {
      Shop shop = new Shop("BestShop");
      long start System.nanoTime();	
      Future<Double> futurePrice = shop.getPriceAsync(1);
      long invocationTime = ((System.nanoTime - start) / 100_000_000);
      System.out.println("invocation returned after " + invocationTime + " msecs");
      
      doSomethingElse();
      
      try {
        double price = futurePrice.get();
        System.out.printf("price is %.2f%n", price);
      } catch(Exception e) {
        throw new RuntimeException();
      }
      long retrievalTime = ((System.nanoTime) - start) / 1_000_000;
      System.out.println("Price returned after " + retrievalTime + " msecs");
    }
    ```

- 비동기에서 에러 처리

  ```java
  public Future<Double> getPriceAsync(long id) {
   CompletableFuture<Double> futurePrice = new CompletableFuture<>;
    new Thread(() -> {
      try {
      double price = ProductService.getProduct(id).getPrice();
      futurePrice.complete(price);      
      } catch(Exception ex) {
        futurePrice.completeExceptionally(ex);
      }
    }).start;
   return futurePrice; 
  }
  ```

- CompletableFuture

  | 메서드                    | 의미                                                         |
  | ------------------------- | ------------------------------------------------------------ |
  | `run()`                   | Runnable 실행자를 실행                                       |
  | `supply()`                | ForkJoinPool의 Executor가 Supplier<T>를 실행                 |
  | `join()`                  | Future의 get과 같음<br />다만 아무 예외도 발생시키지 않음    |
  | `thenApply()`             | 이전 단계의 결과 값을 받아 이번 작업과 연관지어 작업, reduce와 비슷 |
  | `thenCompose()`           | 두 비동기 연산을 파이프라인으로 만듦, flatmap과 비슷<br />외부의 비동기 작업을 completableFuture형태로 진행할 때 사용 |
  | `thenCombine()`           | 두 CompletableFuture연산을 합침<br />BiFunction 인자를 이용해 결과를 어떻게 합칠지 정의 |
  | `orTimeout()`             | timeout을 통해 해당 시간내에 끝내지 못할 경우 TimeoutException 발생 |
  | `completeOnTimeout()`     | orTimeout과 같은 기능, CompletableFuture를 반환              |
  | `thenAccept()`            | Consumer를 인수로 받음                                       |
  | `completeExceptionally()` | exception이 발생했을 때                                      |
  |                           |                                                              |

- 각 메서드 + `Async()`형태로 구현 시 다른 쓰레드에서 동작 가능

  - 해당 기능을 이용해 다른 메서드 호출 시 기존의 `@Async` 를 메서드에 선언을 하지 않아도 동일 수행 가능
  
- stream 병렬화 vs CompletableFuture 병렬화

  - Custom Executor
    - N(thread) = N(CPU) * U(CPU) * (1+W/C)
      - N(CPU) : `Runtime.getRuntim().availableProcessors()` 가 반환하는 코어수
      - U(CPU) : 0과 1사이의 값을 갖는 CPU 활용 비율

  

