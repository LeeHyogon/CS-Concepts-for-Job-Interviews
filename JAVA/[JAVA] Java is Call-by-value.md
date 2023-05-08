```java
    public static void main(String[] args) {
        Student std = new Student("hglee");

        System.out.println("std : " + std);
        System.out.println("std.name : " + std.name);

        changeValue(std);
        System.out.println("std : " + std);
        System.out.println("std.name : " + std.name);
    }
    static void changeValue(Student student) {
        student.name="gon";
        System.out.println("student : " + student);
        System.out.println("student.name : " + student.name);
    }

```
![](https://velog.velcdn.com/images/gon109/post/d62c6051-3a49-44db-97c5-541af4413168/image.png)


출력 결과만 봤을때는 student 객체가 동일하고, 수정한 후의 값이 변경되었으므로, Java는 Call-by-reference인 것처럼 보인다.

그러나 사실은 객체를 넘긴 것이 아니라, heap의 객체의 주소값을 가진 변수 a가 있을때, 변수 a의 값을 복사하여 주소값 변수 b를 넘긴 것이다.

그러므로 Java는 Call-by-value이다.

![](https://velog.velcdn.com/images/gon109/post/dc063383-3051-4108-bf40-1f530a687516/image.png)

다음과 같은 구조로 값을 넘긴다.

ref: https://www.baeldung.com/java-pass-by-value-or-pass-by-reference