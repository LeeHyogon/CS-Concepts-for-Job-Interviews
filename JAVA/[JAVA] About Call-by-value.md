### Call-by-reference vs Call-by-value

- Call-by-value: **인자**의 값이 함수 전/후로 변경되지 않음.
- Call-by-reference: **인자**의 값이 함수 전/후로 변경.


### Java is Call-by-value

```java
    public static void main(String[] args) {

        Student some = new Student("gon");
        System.out.println("some : " + some);
        System.out.println("some.getName() = " + some.getName());
        foo(some);
        System.out.println("some : " + some);
        System.out.println("some.getName() = " + some.getName());
    }
    static void foo(Student some) {
        some.setName("Max");     // AAA
        some = new Student("Fifi");  // BBB
        some.setName("Rowlf");   // CCC
    }

```
![](https://velog.velcdn.com/images/gon109/post/17314133-fe5a-488c-85f0-1e6d5438fe01/image.png)

출력 결과만 봤을때는 student 객체가 동일하고, 수정한 후의 값이 변경되었으므로, Java는 Call-by-reference인 것처럼 보인다.

그러나 사실은 객체를 넘긴 것이 아니라, heap의 객체의 주소값을 가진 변수 a가 있을때, 변수 a의 값을 복사하여 주소값 변수 b를 넘긴 것이다.

그러므로 객체의 주소값을 some= new Student("Fifi")와 값이 변경해주어도,
원본의 주소값은 변하지 않고 Max라는 값을 가지게 된다.

![](https://velog.velcdn.com/images/gon109/post/dc063383-3051-4108-bf40-1f530a687516/image.png)

Java는 다음과 같은 구조로 주소값을 넘긴다.

### C is Call-by-value

- 대표적인 call-by-value 언어인 C를 통한 추가 예시


#### 예시 1

![](https://velog.velcdn.com/images/gon109/post/1ad6e4e1-c172-429f-9570-cbefe94ee2ea/image.png)

- 매개변수: *par
- 인자: args

- func라는 함수 내에서 arg라는 인자 값을 변경할 수 없으므로 Call-by-value

#### 예시 2


![](https://velog.velcdn.com/images/gon109/post/79e7288f-26a3-4ec9-b9b1-639f0b6d92e3/image.png)
- 매개변수: *par
- 인자: &num (num의 주소값)

- num의 주소값을 변경해도 변경이 반영되지 않으므로, Call-by-value


### C++ is Call-by-reference

![](https://velog.velcdn.com/images/gon109/post/da27eebc-6fa9-4aec-81b9-4d527be027cf/image.png)

- 매개변수: &a, &b
- 인자: a, b

- 인자의 값 변경가능하므로, Call-by-reference

ref: https://www.baeldung.com/java-pass-by-value-or-pass-by-reference

ref: https://m.blog.naver.com/han95173/220934411280