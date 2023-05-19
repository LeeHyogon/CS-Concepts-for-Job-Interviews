> 이 개념을 여러 번 접하였는데 대학생 때는 의미도 모르고 외우기 급했고, 취업을 한 뒤에는 잘못된 지식을 사용하고 있었다. 이번 기회에 두루뭉술한 개념을 명확하게 짚고 넘어가려고 한다.

---

![68367180-75116900-0178-11ea-84dd-e9467b233eec](https://user-images.githubusercontent.com/30463982/236966665-739fe0ba-7d58-4d03-8656-5d39c7ac87bf.png)

## Blocking/NonBlocking

Blocking/NonBlocking은 호출되는 함수가 바로 리턴하느냐 마느냐가 관심사다.

- Blocking: **호출된 함수가 자신의 작업을 모두 마칠 때까지 호출한 함수에게 제어권을 넘겨주지 않고 대기.**

- NonBlocking: **호출된 함수가 자신이 할 일을 채 마치지 않았더라도 바로 제어권을 건네주어(Return) 호출한 함수가 다른 일을 진행**.


## Synchronous/Asynchronous

Synchronous/Asynchronous는 호출되는 함수의 작업 완료 여부를 누가 신경 쓰냐가 관심사다.

- Synchronous: 호출하는 함수가 호출되는 함수의 작업 완료 후 리턴을 기다리거나, **호출되는 함수로부터 바로 리턴 받더라도 작업 완료 여부를 호출하는 함수 스스로 계속 확인**하며 신경 씀.

- Asynchronous: 호출되는 함수에게 Callback을 전달해서, **호출되는 함수의 작업이 완료되면 호출되는 함수가 전달받은 Callback을 실행하고 호출하는 함수는 작업 완료 여부를 신경 쓰지 않음.**

---

## 1. Blocking & Synchronous Model

![image](https://user-images.githubusercontent.com/30463982/236966743-d05e0a6d-9514-4b67-af09-a1b26e87fb7e.png)

- 시스템 콜이 들어오면, **커널은 IO 작업이 완료되기 전에는 응답을 하지 않는다.** 

- 시스템 콜을 보낸 후에 유저 프로세스는 응답을 받기 전에는 **block이 되어 다른 작업을 하지 못한다.** 

- 즉, IO 작업이 완료되기 전에는 다른 작업을 수행하지 못한다.



## 2. Blocking & Async Model

![image](https://user-images.githubusercontent.com/30463982/236966864-e9dc50a2-07b4-42fd-80c8-b0b2c67356ad.png)

- IO는 NonBlocking이지만, **Notification는 blocking 방식**으로 하도록 되어있다.

- Select()는 유저 프로세스를 block 한다.

- Select()는 **데이터가 사용이 가능해지면 Notification를 받게 되고 block을 푼다.**



## 3. NonBlocking & Synchronous Model

![image](https://user-images.githubusercontent.com/30463982/236966914-6aab5901-bdc5-4be8-b009-8396b2f8782d.png)

- 시스템 콜이 들어오면 커널은 IO 작업의 완료 여부와는 무관하게 **즉시 응답**을 해준다. (완료되지 않았다면 에러코드를 응답함.)

- 이는 커널이 시스템 콜을 받자마자 제어권을 다시 유저 프로세스에게 넘겨준다는 것이기에, 유저 프로세스는 IO 가 완료되기 전에도 다른 작업을 할 수 있는 것이다.

- 유저 프로세스는 다른 작업들을 수행하다가 중간중간에 시스템 콜을 보내서 IO가 완료되었는지 **커널에게 물어보게 된다.**


## 4. NonBlocking & Async

![image](https://user-images.githubusercontent.com/30463982/236966921-6ac6754b-151e-45b7-a1e7-6d9774baee9b.png)

- 시스템 콜이 들어오면, 커널은 IO 작업의 완료 여부와는 무관하게 **즉시 응답**을 해준다.

- IO 처리는 **백그라운드에서 실행되다가, 완료되면 커널이 유저 프로세스에게 알려줌.**

- **NonBlocking Sync는 유저 프로세스가 IO 완료 여부를 커널에게 계속 물어봐야 했지만 NonBlocking Async는 IO 완료가 되면 그때 커널이 유저 프로세스에게 알려주는 방식**이다.

---

### +추가

Blocking Async의 대표적인 케이스가 Node.js와 MySQL의 조합이라고 한다. Node.js 쪽에서 Callback 지옥을 헤치면서 Async로 전진해와도, 결국 DB 작업 호출 시에는 MySQL에서 제공하는 드라이버를 호출하게 되는데, 이 드라이버가 Blocking 방식이라고 한다. 이건 사실 Node.js 뿐아니라 Java의 JDBC도 마찬가지다. 다만 Node.js가 싱글 스레드 루프 기반이라 멀티 스레드 기반인 Java의 Servlet 컨테이너보다 문제가 더 두드러져 보일 뿐, Blocking Async라는 근본 원인은 같다.

그래서 Blocking Async는 이렇게 정리해도 좋을 것 같다.
> **Blocking Async는 별다른 장점이 없어서 일부러 사용할 필요는 없지만, NonBlocking Async 방식을 쓰는데 그 과정 중에 하나라도 Blocking으로 동작하는 놈이 포함되어 있다면 의도하지 않게 Blocking Async로 동작**할 수 있다.

---

## Case Study: 대표님, 개발자 좀 더 뽑아주세요..

아래는 이해를 돕기 위한 예시이다.
읽다가 재밌어서 가져와봤다.

Blocking & Synchronous
>나 : 대표님, 개발자 좀 더 뽑아주세요..   
대표님 : 오케이, 잠깐만 거기 계세요!   
나 : …?!!   
대표님 : (채용 공고 등록.. 지원자 연락.. 면접 진행.. 연봉 협상..)   
나 : (과정 지켜봄.. 궁금함.. 어차피 내 일 하러는 못 가고 계속 서 있음)   


Blocking & Asynchronous
>나 : 대표님, 개발자 좀 더 뽑아주세요..   
대표님 : 오케이, 잠깐만 거기 계세요!   
나 : …?!!   
대표님 : (채용 공고 등록.. 지원자 연락.. 면접 진행.. 연봉 협상..)   
나 : (안 궁금함.. 지나가는 말로 여쭈었는데 붙잡혀버림.. 딴 생각.. 못 가고 계속 서 있음)


Non-blocking & Synchronous
>나 : 대표님, 개발자 좀 더 뽑아주세요..   
대표님 : 알겠습니다. 가서 볼 일 보세요.   
나 : 넵!   
대표님 : (채용 공고 등록.. 지원자 연락.. 면접 진행.. 연봉 협상..)   
나 : 채용하셨나요?   
대표님 : 아직요.   
나 : 채용하셨나요?   
대표님 : 아직요.   
나 : 채용하셨나요?   
대표님 : 아직요~!!!!!!   


Non-blocking & Asynchronous
>나 : 대표님, 개발자 좀 더 뽑아주세요..   
대표님 : 알겠습니다. 가서 볼 일 보세요.   
나 : 넵!   
대표님 : (채용 공고 등록.. 지원자 연락.. 면접 진행.. 연봉 협상..)   
나 : (열일중..)   
대표님 : 한 분 모시기로 했습니다~!   
나 : 😍   
내용을 입력하세요.   



ref https://blog.naver.com/set_star/223097053682