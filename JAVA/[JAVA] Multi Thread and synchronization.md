
#### 공유자원

- 여러 Thread가 동시에 접근할 수 있는 자원

#### 임계영역
- 동시접근하려고 하는 자원에서 문제가 발생하지 않게 독점을 보장해줘야 하는 영역

##### 임계영역을 설정해주는 Synchronized
```java
public synchronized void method() {
	// 자원 경합 (race condition)이 일어나는 코드
}
```
- synchronized 함수를 통해 Blocking 동기화
- Blocking 상태로 인해 데드락, 성능문제

### 경쟁상태

- 둘 이상의 Thread가 공유자원을 병행적으로 읽거나 쓰는 동작을 할 때 타이밍이나 접근 순서에 따라 실행 결과가 달라지는 상황


### 연산의 원자성

#### Multi Thread상태의 연산 문제

1. Read - Modify - Write 패턴
- 메모리에서 값을 읽음(Read)
- 읽어온 값을 수정(modify)
- 수정한 값을 다시 메모리에 씀(write)
  ![](https://velog.velcdn.com/images/gon109/post/6b625b25-c669-4ea1-8719-ff4c97f3afb4/image.png)


2. Check - then - act 패턴
- 분기문 비교 (read)
- 로직(act)
  ![](https://velog.velcdn.com/images/gon109/post/3b40fd41-258a-4fbb-97e0-06c115a65894/image.png)



- 해결 방안: 공유 자원에 대한 작업 단위가 더이상 쪼갤 수 없는 하나의 연산인 것처럼 동작하도록 변경(원자성)

### 메모리 가시성

- 선언한 변수의 값은 Memory에만 존재하는 것이 아니라, CPU cahce라고 하는 영역에도 존재
- 문제는  CPU cache에 값이 Memory에 언제 옮겨갈지도 모른다는 것

![](https://velog.velcdn.com/images/gon109/post/08b57352-02d3-4f11-9046-2112c41f58b9/image.png)

- 해결 방안: volatile로 선언된 변수를 CPU에서 연산을 하면 바로 메모리에 쓴다.

### CAS(Compare and Swap)

- 변경하기 전에 기존에 가지고 있던 값이 내가 예상하던 값과 같을 경우에만 새로운 값으로 할당하는 방법
```java
 public boolean compareAndSwap(int oldVal, int newVal) {
        if(val == oldVal) {
            val = newVal;
            return true;
        } else {
            return false;
        }
    }
```

### AtomicInteger
- 멀티 쓰레드 환경에서 동기화 문제를 별도의 synchronized 키워드 없이 해결하기 위해서 고안된 방법

```
public class AtomicInteger extends Number implements java.io.Serializable 
{ 
	private volatile int value; 
    
	public final int incrementAndGet() { 
      int current; 
      int next; 
      do{ 
      	current = get(); 
        next = current + 1; 
      } while (!compareAndSet(current, next)); 
      return next; 
      } 
      public final boolean compareAndSet(int expect, int update) { 
      	return unsafe.compareAndSwapInt(this, valueOffset, expect, update); 
      } 
}
```


출처: https://www.baeldung.com/cs/race-conditions
출처: https://velog.io/@syleemk/Java-Concurrent-Programming-%EA%B0%80%EC%8B%9C%EC%84%B1%EA%B3%BC-%EC%9B%90%EC%9E%90%EC%84%B1