## 1. Applicability

스프링 MVC냐, 웹 플럭스냐?

![mvcVsFlux](https://user-images.githubusercontent.com/30463982/232762342-0670e9f5-f9b8-4b9a-b958-28932984a48e.png)

1. **제대로 작동하는 Spring MVC 애플리케이션이 있다면 변경할 필요가 없다.** Spring MVC는 명령형 프로그래밍으로 코드 작성, 이해 및 디버그 하는 가장 쉬운 방법이다. 

2. 이미 Non-blocking 웹 스택을 찾고 있다면 Spring WebFlux는 이 분야의 다른 제품과 동일한 실행 모델이라는 이점과 함께 서버(Netty, Tomcat, Jetty, Undertow 및 Servlet 컨테이너), 프로그래밍 모델(annotated controllers and functional web endpoints) 및 반응성 라이브러리(Reactor, RxJava 또는 기타)를 제공한다.

3. Java 8 람다 또는 Kotlin과 함께 사용할 수 있는 가볍고 Functional 웹 프레임워크에 관심이 있다면 Spring WebFlux 함수형 웹 엔드 포인트를 사용할 수 있다. 또한 소규모 애플리케이션 또는 **마이크로 서비스에 적합**한 선택일 수 있다.

4. 마이크로 서비스 아키텍처에서는 Spring MVC와 Spring WebFlux 컨트롤러, Spring WebFlux 기능 엔드 포인트와 함께 애플리케이션을 혼합할 수 있다. **이 두 프레임워크 모두 동일한 어노테이션 기반 프로그래밍 모델을 지원한다는 점은 올바른 도구를 적재적소에 사용하는 동시에 기존 지식을 재사용하기 쉽게 만들어준다.**

5. 애플리케이션을 평가하는 간단한 방법은 의존성을 확인하는 것이다. 사용할 **blocking JPA, JDBC 또는 네트워킹 API가 있는 경우 Spring MVC는 최소한 공통 아키텍처에 대한 최상의 선택**이다. 별도의 스레드에서 blocking 호출을 수행하는 것은 Reactor와 RxJava에서 모두 기술적으로 가능하지만 non-blocking 웹 스택을 최대한 활용하지는 못할 것이다.

6. 원격 서비스에 대한 호출이 있는 Spring MVC 응용 프로그램이 있는 경우 Reactive WebClient를 사용해 보자. Spring MVC 컨트롤러 메서드에서 직접 반응 유형(Reactor, RxJava 또는 기타)을 반환할 수 있다. 호출 당, 혹은 호출 간 상호 작용의 지연이 클수록, 더욱 드라마틱한 이점을 얻을 수 있다. 스프링 MVC 컨트롤러는 다른 리액티브 컴포넌트를 똑같이 호출할 수 있다.

7. 대규모 팀이 있는 경우 non-blocking 함수 및 선언적 프로그래밍으로 전환하는 과정에서 가파른 학습 곡선을 염두에 두어야 한다.

​
## 2. Performance

**리액티브와 논 블로킹을 사용할 때 중요한 이점은 적고 고정된 수의 스레드와 보다 적은 메모리를 사용하도록 조정할 수 있는 능력에 있다.** 이는 애플리케이션이 부하에 대해 더 탄력적으로 동작할 수 있도록 한다. 왜냐하면 보다 예측할 수 있는 방법으로 조정되기 때문이다. 하지만 이 스케일링을 관측하기 위해서는 약간의 지연을 필요로 한다(느리고 예측 불가능한 네트워크 I/O의 혼재로 인해). 여기서 리액티브 스택의 장점을 볼 수 있으며, 그 차이가 드라마틱 하게 드러나는 지점이다.

​
## 3. Concurrency Model

- Spring MVC와 Spring WebFlux 모두 어노테이션 컨트롤러를 지원하지만 동시성 모델과 차단 및 스레드에 대한 기본 가정에는 중요한 차이점이 있다.

- 스프링 **MVC(그리고 일반적인 서블릿 애플리케이션)에서는 애플리케이션은 현재 스레드가 Blocking 될 것을 상정**한다(예로, 원격 호출에 대하여). 그리고 이로 인하여 서블릿 컨테이너는 요청을 핸들링하는 동안 발생할 수 있는 잠재적인 Blocking에 대비하기 위해 큰 수의 스레드 풀을 사용하게 된다.

- 스프링 웹플럭스(그리고 일반적인 Non-blocking 서버에서)에서는 **애플리케이션은 스레드를 Blocking 하지 않을 것을 상정**한다. 따라서 Non-Blocking 서버는 적고 고정된 크기의 스레드 풀을 사용하여 요청을 처리한다(이벤트 루프 워커).

- "스케일링"과 "적은 수의 스레드"는 모순으로 보일 수 있지만, **현재 스레드가 절대 Blocking 되지 않는다는 것은 Blocking 호출을 받아들일 추가 스레드가 필요하지 않다는 의미**가 된다.

​

## 4. MVC vs WebFlux 결론

1. Spring MVC와 WebFlux의 공통점은 @Controller, Reactive 클라이언트이다. 둘 다 Tomcat, Jetty, Undertow와 같은 서버에서 실행할 수 있다.

2. Spring MVC에서는 명령형 로직, JDBC, JPA를 가질 수 있다. Spring WebFlux에서는 기능적 끝점, 이벤트 루프, 동시성 모델을 가질 수 있다. Spring WebFlux는 Netty 서버에서 실행할 수 있다.
​

### 동작 방식

- Spring MVC:

    단일 스레드로 Blocking을 하며 하나의 프로세스 처리 => 많은 요청 발생 시 => 동시성 달성을 위한 많은 스레드 풀 사용(스레드가 여러 개)

    많은 요청 예: DB 사용, 외부 API 호출

​
- Spring Webflux:

    Non-Blocking으로 Reactive Stack(Push 기반, 단일 디스패처 스레드)을 사용 => 스레드 풀을 사용하지만 적은 수의 스레드를 사용(=이벤트 루프).


**같은 스레드 풀을 사용하지만 MVC와 Webflux는 다른 동시성 모델임.**


ref https://blog.naver.com/set_star/223078549395
