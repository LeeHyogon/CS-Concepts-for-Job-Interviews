## 1. Flux vs Mono

Spring WebFlux는 Spring5에서 새롭게 추가된 모듈이다. WebFlux는 클라이언트, 서버에서 reactive 스타일의 애플리케이션 개발을 도와주는 모듈이며, reactive-stack web framework이며 non-blocking에 reactive stream을 지원한다.

Spring Webflux에서 사용하는 Reactive library가 Reactor이고 Reactor가 Reactive Streams의 구현체이다. 그래서 Webflux 문서에 Reactive Streams가 언급되는 것이고 그거와 같이 Reactor가 나오고 주요 객체인 Flux/Mono가 나오는 것이다. 결국 Webflux의 동작 구조를 이해하는 Flux 와 Mono를 알고 넘어가야 한다.


## 2. Flux

아래는 Flux가 아이템을 변환하는 방법을 도식화한 그림이다.

![flux](https://user-images.githubusercontent.com/30463982/234220555-16e3b211-5617-4ee6-8512-39bda1d2297b.png)

- 0개부터 N 개까지 다수의 응답을 만드는데 특화되어 있는 Publisher 구현체이다.

- onComplete나 OnError가 되기 전까지 무한 생성 가능한 Stream으로 생각해야 한다.

- 데이터가 전달될 때마다 onNext 이벤트를 발생시키고 모든 데이터의 전달 처리가 완료되면 onComplete, 데이터를 전달하는 과정에서 오류가 발생하면 onError로 종료된다.


Flux in action :
```java
Mono.fromCallable(System::currentTimeMillis)
    .flatMap(time -> Mono.first(serviceA.findRecent(time), serviceB.findRecent(time)))
    .timeout(Duration.ofSeconds(3), errorHandler::fallback)
    .doOnSuccess(r -> serviceM.incrementSuccess())
    .subscribe(System.out::println);
```

## 3. Mono

아래는 Mono가 아이템을 변환하는 방법을 도식화한 그림이다.

![mono](https://user-images.githubusercontent.com/30463982/234220546-076a10fa-8c37-412b-ae8d-b0a86d9aa3b5.png)

- 최대 1개의 응답을 만드는데 특화되어 있는 Publisher 구현체이다(0개 또는 1개로 명확하게 구분되는 것은 Mono를 사용해야 한다).


Mono in action :
```java
Flux.fromIterable(getSomeLongList())
    .mergeWith(Flux.interval(100))
    .doOnNext(serviceA::someObserver)
    .map(d -> d * 2)
    .take(3)
    .onErrorResume(errorHandler::fallback)
    .doAfterTerminate(serviceM::incrementTerminate)
    .subscribe(System.out::println);
```
Blocking Mono result :
```java
Tuple2<Long, Long> nowAndLater = 
        Mono.zip(
                Mono.just(System.currentTimeMillis()),
                Flux.just(1).delay(1).map(i -> System.currentTimeMillis()))
            .block();
```

## 4. 결론

- Mono 와 Flux는 모두 Publisher 인터페이스의 구현이다.

- Flux 와 Mono의 차이점은 발행하는 데이터 개수이다.

**Mono:**

1.  계산과 같은 작업을 수행하거나 데이터베이스 또는 외부 서비스에 요청하고 최대 하나의 결과를 예상할 때

2. 0 또는 1개의 값을 포함하므로 Java의 Optional 클래스 와 더 관련


**Flux:**

1. 계산, 데이터베이스 또는 외부 서비스 호출에서 여러 결과가 예상할 때

2. Flux는 N 개의 값을 가질 수 있으므로 List 와 더 관련



ref https://blog.naver.com/set_star/223084804736