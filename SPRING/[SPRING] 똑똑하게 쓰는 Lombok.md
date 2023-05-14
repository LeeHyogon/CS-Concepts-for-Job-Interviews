## 조사한 이유?

- Lombok Annotation이 정확히 어떤 코드를 대신 처리해주고 있는지 제대로 알지 못하는 경우가 많기 때문

## @Getter

@Getter는 단어 그대로 클래스 내의 변수를 외부에서 접근하기 위한 get 메서드들을 생성해주는 역할을 한다.  
하지만 값이 Optional일 경우 직접 Getter를 구현하는 것이 더 좋은 경우가 존재한다.

```java
@Getter
public class PagenationDto {
    private Integer pageSize;
    private Integer pageIndex;
    
    public PagenationDto() {}
    
    public Integer getPageSize() {
        return this.pageSize == null ? 10 : this.pageSize;
    }
    
    public Integer getPageIndex() {
        return this.pageIndex == null ? 1 : this.pageIndex;
    }
}
```
위의 코드처럼 Pagenation 기능을 구현 할 때, Client 측에서 Request로 PageSize와 PageIndex를
따로 서버에 넘겨주지 않았다면 기본값으로 각각 10과 1을 지정해달라는 요구사항이 있을 수 있다.
이러한 경우 DTO 단에서 Getter를 통해 바로 기본 값을 설정해줄 수 있다는 장점이 있다.

무턱대고 @Getter를 사용하기 보다, @Getter를 사용할 지 직접 Getter 메소드를 구현할 지 고민해 보는 습관이 필요하다.

## @Setter

데이터 중에는 변할 수 있는 데이터가 있고 변치 않아야 하는 데이터가 있다.  
실수로라도 변치 않아야 하는 데이터의 Setter를 구현하는 실수를 저지르고 있지 않은지 확인해야 한다.

<table>
  <tr>
    <td>Case 1</td> <td>Case 2</td>
  </tr>
  <tr>
    <td>
      <pre>
        <code class="language-java">
@Setter
public class Post {
    private long postId;
    private String title;
    private String content;
}
        </code>
      </pre>
    </td>
<td>
      <pre>
        <code class="language-java">
public class Post {
    private long postId; <br>
    @Setter
    private String title; <br>
    @Setter
    private String content;
}
        </code>
      </pre>
    </td>
  </tr>
</table>

Post 객체의 ```postId```는 변경될 일이 없는 데이터이다. Case 1의 코드처럼
@Setter를 class 전체에 적용하게 되면 ```postId```를 변경하는 일이 가능해진다.
반면, Case 2의 코드와 같이 변경이 필요한 데이터에 한하여 Setter를 적용한다면
Post 객체만으로도 변경할 수 있는 값 / 없는 값을 구분할 수 있고 불필요한 변경도
방지할 수 있다.

정확한 @Setter 명시를 통해 변경할 수 있는 값 / 없는 값을 구분해주는 친절함이 필요하다.

## @Builder

Builder 어노테이션에는 생각보다 많은 코드가 숨겨져 있다.  
Builder의 무분별한 사용으로 인해 Lombok이 컴파일 타임에 다량의 코드를 소스코드에 끼워 넣으며 빌드 시간을 증가시킬 수 있다.

```java
@Builder
public class Post {
    private long postId;
    private String title;
    private String content;
}
```
위의 코드는 아래의 코드를 생성한다.

```java
public interface PostBuilder {
    PostBuilder postId(long postId);
    PostBuilder title(String title);
    PostBuilder content(String content);
    Post build();
}

public class DefaultPostBuilder implements PostBuilder {
    private long postId;
    private String title;
    private String content;
    
    @Override
    public PostBuilder postId(long postId) {
        this.postId = postId;
        return this;
    }
    
    @Override
    public PostBuilder title(String title) {
        this.title = title;
        return this;
    }
    
    @Override
    public PostBuilder content(String content) {
        this.content = content;
        return this;
    }
    
    @Override
    public Post build() {
        return new Post(postId, title, content);
    }
}
```
3개의 변수만 가진 클래스인데도 코드의 양이 상당히 많다.
10개 이상의 변수를 가진 클래스에 class 단위로 @Builder를 적용하면 코드의 양은 더더욱 증가한다.

```java
public class Post {
    private long postId;
    private String title;
    private String content;
    
    @Builder
    public Post(long postId) {
        this.postId = postId;
    }
}
```
위의 코드와 같이 class 단위로 @Builder를 적용하는 것보다
객체의 생성에 필요한 변수만을 파라미터로 갖는 생성자에 @Builder를 적용하는 것을 추천한다.

## @Data

@Data 내부에는 @Getter, @Setter, @RequiredArgsConstructor, @ToString, @EqualsAndHashCode가 포함되어 있다.
어노테이션 하나로 코드를 생성해낼 수 있어 편리하지만 불필요한 코드를 양산해내고 있는 것은 아닌지 고민이 필요하다.

간혹 @Data와 @Getter를 함께 사용하고 있는 코드도 발견된다. 이는 코드의 중복이다.
Raw 코드를 작성해보고 @Data를 접하는 방식이 아니라 다른 개발자들이 편리하다기에 따라서
@Data를 사용하고 있지 않은지 돌아볼 필요가 있다.