### Redis
- in-memory 형태의 No-SQL로써 Key-Value 쌍의 해쉬 맵 형태의 데이터베이스

#### 장점

- in-memory: disk가 아닌 memory에 저장하기 때문에 disk I/O 작업이 발생하지 않아 속도가 빠르며, 휘발성이다.
- No-SQL: Not Only SQL를 뜻하며 RDBMS에 비해 속도가 빠름.

#### 특징
- String외에도 Set,Sorted Set, Hash, List 등 다양한 타입을 지원.
- 영속성: 특정 시점에 데이터를 디스크에 저장하여 보관할 수 있고, 장애 상황 시에 복구에 사용할 수 있다.
- 1개의 싱글 쓰레드로 수행되기 때문에, 인프라를 구축하기위해서는 Replication(Master-Slave 구조) 필수이다.


### 캐시 전략

#### Look aside 패턴
- 캐시를 한번 접근하여, 데이터가 없다면 DB 호출.

#### Write Back 패턴
- 쓰기 작업이 굉장히 많아서, INSERT 쿼리를 일일이 날리지 않고 한꺼번에 배치 처리를 하기 위해 사용
- insert 를 1개씩 500번 수행하는 것보다 500개를 한번에 삽입하는 동작이 훨씬 빠르기 때문에 write back 방식은 빠른 속도로 서비스가 가능
- DB에 데이터를 저장하기 전에 데이터 유실 문제.

#### Redis vs Memcached

#### 공통점
- 무료 오픈소스 데이터베이스
- Key-Value 형태로 저장되기 때문에 같은 Key-Value 로 저장되는 세션 데이터를 다루는데 적합

#### 비교
| .      | Redis                                       | Memcached            |
|--------|---------------------------------------------|----------------------|
| 코어     | 싱글                                          | 멀티                   |
| 자료구조   | Strings, Set, Sorted-Set, Hashes 등 다양한 자료구조 | String, Integer만 지원  |
| 데이터 저장 | Memory, Disk                                | Memory               |
| 영속성    | 영속성 데이터 사용                                  | 지원 X                 |


### 싱글스레드임에도 High Performance

- 싱글스레드 만으로 초당 10만건 이상의 데이터 요청을 처리할 수 있다.
- 레디스가 제공하는 클러스터 기능을 활용한다면 더 많은 요청을 처리할 수 있다.
- Redis 6.0부터 ThreadedIO가 추가되어 사용자 명령이 멀티쓰레드가 지원이 된다. 하지만 명령어를 실행하는 코어부분은 여전히 single thread로 동작하며, I/O Socket read/write를 할때 멀티쓰레드로 동작하여 전반적인 성능을 향상.

### Redis 데이터의 영속성
- in-memory DB임에도 불구하고 메모리 데이터를 disk에 저장할 수 있는 특징이 있다.
- 데이터를 영속성으로 저장하는 방법은 RDB (snapshotting) 방식과 AOF (Append only file) 두가지가 있다.

#### RDB
- 순간적으로 메모리에 있는 내용을 스냅샷을 떠서 DISK에 옮겨 담는 방식
- 스냅샷을 뜬다는 말은 특정 시점의 메모리에 있는 데이터를 바이너리 파일로 저장한다는 의미.
- 스냅샷을 추출하는데 오래걸리고, 도중에 서버가 꺼지면 데이터를 유실.

#### AOF
- redis의 모든 write/update 연산 자체를 모두 log 파일에 기록
- 조회를 제외한 입력/수정/삭제 명령이 실행될 때 마다 기록
- **서버가 다운되더라도 데이터가 사라지지 않는다.**
-  write/update 연산을 log파일에 남기기 때문에 log 데이터 양이 크고, 복구 시 저장된 모든 write/update연산을 다시 실행하기 때문에 재시작 속도가 느림.


ref https://inpa.tistory.com/m/entry/REDIS-%F0%9F%93%9A-%EA%B0%9C%EB%85%90-%EC%86%8C%EA%B0%9C-%EC%82%AC%EC%9A%A9%EC%B2%98-%EC%BA%90%EC%8B%9C-%EC%84%B8%EC%85%98-%ED%95%9C%EB%88%88%EC%97%90-%EC%8F%99-%EC%A0%95%EB%A6%AC </br>

ref https://velog.io/@mu1616/%EB%A0%88%EB%94%94%EC%8A%A4%EC%99%80-%EC%8B%B1%EA%B8%80%EC%8A%A4%EB%A0%88%EB%93%9C
