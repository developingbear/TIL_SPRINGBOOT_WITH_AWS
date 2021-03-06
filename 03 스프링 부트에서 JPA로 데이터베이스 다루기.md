03 스프링 부트에서 JPA로 데이터베이스 다루기
=======================
웹 서비스를 개발하고 운영하다 보면 피할 수 없는 문제가 데이터베이스를 다루는 일이다.      
이전 데이터베이스 기술은 객체 모델링이 아닌 **테이블 모델링**에 집중되어있는 형태였고       
**객체를 테이블에 맞추어 데이터를 전달하는 형식**으로 객체지향 프로그래밍과 거리가 먼 형태였다     
그래서 이러한 문제의 해결책으로 JPA라는 자바 표준 ORM 기술을 사용해보자    

# 1. JPA 소개 
현대의 웹 애플리케이션에서 관계형 데이터베이스는 빠질 수 없는 요소이다.            
그러다 보니 객체를 관계형 데이터베이스에서 관리하는 것이 무엇보다 중요해졌다.         
이런 현상이 짙어지다 보니 모든 코드는 SQL 중심이 되기 시작했고          
현업 프로젝트는 애플리케이션 코드보다 SQL로 가득하게 되었다.       
     
개발자가 아무리 자바 클래스를 아름답게 설계해도 SQL을 통해야 데이터베이스를 사용할 수 있기에 피할 수 없다.         
하지만 **SQL 을 반복적으로 지속적으로 사용해야 하고 테이블이 수백개면 수백개의 SQL 코드를 작성해야한다.**      
그리고 **각각의 관계형 데이터베이스마다 쿼리문이 다르니 이는 기하 급수적으로 늘어나게 된다.**       

또 한가지 문제점이 있다. 바로 **패러다임 불일치**이다.    
* 관계형 데이터베이스 : **어떻게 데이터를 저장할지**
* 객체지향 프로그래밍 : **메시지를 기반으로 기능과 속성을 한 곳에서 관리하는 기술**

예를 들면 
```
User user = findUser();
Group group = user.getGroup();
```
위 코드와 같이 User와 Group 부모 자식 관계로 User를 통해서 Group을 얻을 수 있지만
여기에 데이터베이스 코드가 들어가게 된다면  
```
User user = userDao.findUser();
Group group = GroupDao.findGroup(user.getGroupId());
```
상속, 1:N 등 다양한 객체 모델링을 데이터베이스로는 구현을 할 수 없어             
각각에 DAO를 생성해주어 따로따로 조회를 해야하는 번거로움이 생긴다.          
이렇다 보니 웹 애플리케이션 개발은 점점 데이터베이스 모델링에만 집중하게 되었다.   
     
**JPA**는 서로 지향하는 바가 다른 2개의 영역을 **중간에서 패러다임 일치**를 시켜주기 위한 기술이다.      
개발자는 객체지향적으로 프로그래밍을 하고, JPA가 이를 관계형 데이터베이스에 맞게 SQL을 대신 생성해서 실행한다.    
개발자는 객체지향 프로그래밍만 신경쓰면 되는 것으로 **SQL에 종속적인 개발을 하지 않아도 된다.**   

## 1.1. SpringData JPA 
JPA는 인터페이스로 자바 표준 명세서이다.  
즉, 인터페이스인 JPA를 사용하기 위해서는 구현체(실체)인 Hibernate, Eclipse Link등이 있다.  
  
Spring에서는 이러한 구현체를 직접 다루지 않고 이 위에 SpringData JPA 모듈을 이용하여 JPA 기술을 다룬다.   
```
JPA <- Hibernate <- SpringData JPA
```  
그럼 이렇게 사용하는 이유는 무엇이 있을까? 매번 그렇듯 유지보수를 편하기 하기 위해서이다.   
    
* 구현체 교체의 용이성  
* 저장소 교체의 용이성   

**구현체 교체의 용이성**
```
Hibernate외에 다른 구현체로 쉽게 교체하기 위함
```  
SpringData JPA 내부에서 구현체 매핑을 지원해주기 때문에    
Hibernate가 언젠가 수명을 다해서 새로운 JPA 구현체가 대세로 떠오를 경우 손쉽게 교체하기 위해서이다.     
    
**저장소 교체의 용이성**
```
관계형 데이터베이스 외에 다른 저장소로 쉽게 교체하기 위한 것이다.      
```
서비스 초기에는 관계형 데이터베이스로 모든 기능을 처리했지만,  
점점 트래픽이 많아져 관계형 데이터베이스로는 도저히 감당이 안될 때 non-sql로 교체를 할 수도 있다.(Mongo DB   
이때 개발자는 교체를 원한다면 SpringData JPA 에서 SpringData MongoDB로 의존성만 교체하면 된다.  
   
이는 SpringData의 하위 프로젝트들은 기본적으로 CRUD의 인터페이스가 같기 때문이다.  
그렇다보니 저장소가 교체되어도 기본적인 기능은 변경할 것이 없다.  
   
## 1.2. 실무에서 JPA 
실무에서 JPA를 사용하지 못하는 가장 큰 이유로 **높은 러닝 커브**를 이야기한다.   
JPA를 잘 쓰려면 객체지향 프로그래밍과 관계형 데이터베이스를 둘 다 이해해야 한다.  
   
하지만 JPA를 사용하게 되면 CRUD를 작성할 필요가 없어지고
부모-자식 관계, 1:N 관계 표현, 상태와 행위를 한 곳에서 관리하는 등 객체지향 프로그래밍을 쉽게 할 수 있다.
또한 속도 이슈도 없기에 많은 트래픽을 처리하는데도 사용해도 된다.  

## 1.3. JPA 사용하기
     
**게시판 기능**   
   
* 게시글 조회
* 게시글 등록
* 게시글 수정
* 게시글 삭제
___   
**회원 기능**   
    
* 구글 / 네이버 로그인    
* 로그인한 사용자 글 작성 권한    
* 본인 작성 글에 대한 권한 관리    
___

### 1.3.1. 라이브러리 의존성 주입 받기   
**build.gradle**   
```gradle
dependencies {
    compile('org.springframework.boot:spring-boot-starter-web')
    compile('org.projectlombok:lombok')
    compile('org.springframework.boot:spring-boot-starter-data-jpa')
    compile('com.h2database:h2')
    testCompile('org.springframework.boot:spring-boot-starter-test')
}
```
   
**소스 코드 해석**   
```gradle
spring-boot-starter-data-jpa
  * 스프링 부트용 Spring Data Jap 추상화 라이브러리입니다.  
  * 스프링 부트 버전에 맞춰 자동으로 JPA 관련 라이브러리들의 버전을 관리해줍니다.  
________________________________________________________________________________
h2
  * 인메모리 관계형 데이터베이스입니다.      
  * 별도의 설치가 필요 없이 프로젝트 의존성만으로 관리할 수 있습니다.      
  * 메모리에서 실행되기 때문에 애플리케이션을 재시작할 때마다 초기화된다는 점을 이용하여 테스트 용도로 많이 사용된다.       
  * 이 책에서는 JPA의 테스트, 로컬 환경에서의 구동에서 사용할 예정이다.     
  * 필자 주관으로 oracle의 sqlite 같은 격  
``` 

### 1.3.2. 폴더(패키지) 생성    
**java 폴더 -> 디폴트 패키지에 domain 폴더 생성** 
   
domain은 게시글, 댓글, 회원, 정산, 결제 등 소프트웨어에 대한 요구사항 혹은 문제 영역이라 생각하자  
기존 객체,DAO,xml 구조와 달리 객체클래스에서만 해결할 수 있다는 차이점에서 나온 용어이다.   

### 1.3.3. Posts 소스 코드 작성
domain 폴더 밑에 posts 폴더 생성후 클래스 파일 작성  

**Posts**    
```java
package com.jojoldu.book.springboot.domain.posts;

import com.jojoldu.book.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
@Entity
public class Posts extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long Id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column( columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(String title, String content, String author){
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public void update(String title, String content){
        this.title = title;
        this.content = content;
    }
}

```
Posts 클래스는 실제 DB의 테이블과 매칭될 클래스이며 보통 Entity 클래스라고 부른다.     
JPA를 사용한다면 DB 데이터에 작업할 경우 실제 쿼리를 날리기보다는, 이 Entity 클래스의 수정을 통해 작업한다.       
     
**꿀팁**   
```
웬만하면 Entity의 PK는 Long 타입의 Auto_increment를 추천한다.  
주민등록번호와 같이 비즈니스상 유니크 키나, 여러 키를 조합한 복합키로 PK를 선정할 경우 
   
1. FK 를 맺을때 다른 테이블에서 복합키 전부를 갖고 있거나, 중간 테이블을 하나 더 두는 상황이 생긴다.  
2. 인덱스에 좋은 영향을 끼치지 못한다.  
3. 유니크한 조건이 변경될 경우 PK 전체를 수정해야 하는 일이 발생한다.  
그렇기에 주민등록 번호, 복합키 등은 유니크 키로 별도로 추가를 해주자
```

**소스 코드 해석**
```java
@Entity

  * 테이블과 링크될 클래스임을 나타낸다.  
  * 기본값으로 클래스의 카멜케이스 이름을 언더스코어 네이밍으로 테이블 이름을 매칭한다.  
  * SalesManager.java -> sales_manager table
____________________________________________________________________________________
@Id

  * 해당 테이블의 PK 필드를 나타낸다.
____________________________________________________________________________________
@GerneratedValue

  * PK의 생성 규칙을 나타낸다.  
  * 스프링부트 2.0 에서는 GenerationType.IDENTITY 옵션을 추가해야만 auto_increment가 된다.   
  * 참고 사이트 : https://jojoldu.tistory.com/295에 정리
____________________________________________________________________________________
@Column

  * 테이블의 칼럼을 나타내며 굳이 선언하지 않더라도 해당 클래스의 필드는 모두 칼럼이 된다.  
  * 사용하는 이유는, 기본값 외에 추가로 변경이 필요한 옵션이 있으면 사용한다.  
  * 문자열의 경우 VARCHAR(255)가 기본값인데, 사이즈를 500으로 늘리고 싶거나, 
  타입을 TEXT로 변경하고 싶거나 등의 경우에 사용된다.  
____________________________________________________________________________________
@NoArgsConstructor

  * 기본 생성자 자동 추가 -> public Posts(){}
____________________________________________________________________________________
@Getter

  * 클래스 내 모든 필드의 Getter 메소드를 자동생성  
____________________________________________________________________________________
@Builder

  * 해당 클래의 빌더 패턴 클래스를 생성
  * 생성자 상단에 선언시 생성자에 포함된 빌드만 빌더에 포함  
```
이 Posts 클래스에는 한 가지 특징이 있는데 바로 Setter 메소드가 없다.     
자바빈 규약을 따지면 Getter/Setter 메소드를 정의해주는 것이 좋긴 하지만   
이렇게 되면 해당 클래스의 인스턴스 값들이      
언제 어디서 변해야 하는지 코드상으로 명확하게 구분할 수가 없어, 차후 기능 변경시 정말 복잡해진다.    
그래서 **Entity 클래스에서는 절대 Setter 메소드를 만들지 않는다.**      
대신 필드의 값 변경이 필요하면 명확히 그 목적과 의도를 나타낼 수 있는 메소드를 추가해야만 한다.     
   
```java
public class Order{
     public void setStatus(boolean status){
          this.status = status;
     }
     public void 주문서비스의_취소이벤트(){
          order.setStatus(false);
     }
}
```
   
위와 같이 Setter는 단순히 값을 세팅하는 것이기에 명확하게 목적과 의도를 나타내주지 못한다.     
   
```java
public class Order{
     public void cancleOrder(){
          this.status = false;
     }
     public void 주문서비스의_취소이벤트(){
          order.cancleOrder();
     }
}
```
    
위와 같이 메소드에 이름을 정확히 나타내주면 어떠한 목적과 의도로 값을 세팅하는지 파악이 가능해진다.     
   
**그러면**       
Setter 가 없는 이 상황에서 어떻게 값을 채워 DB에 삽입해야 할까?       
    
기본적인 구조는 생성자를 통해 최종값을 채운 후 DB에 삽입 하는 것이며,      
값 변경이 필요한 경우 해당 이벤트에 맞는 public 메소드를 호출하여 변경하는 것을 전제로한다.        
           
또한 생성자 대신에 @Builder를 통해 제공되는 빌더 클래스를 사용한다.            
생성자나 빌더나 생성 시점에 값을 채워주는 역할은 똑같다.           
다만, 생성자의 경우 지금 채워야 할 필드가 무엇인지 명확하게 지정을 할 수 없다.          
            
예를 들면 같은 자료형의 매개변수의 위치를 변경해도 에러가 일어나지 않아 문제를 잘 모른다.           
     
**생성자**
```java
String a = "woojae";
String b = "Charlie";

Example(b , a); -> 순서가 바

public Example(String pesrson_name, String dog_name){
     this.person_name = person_name;
     this.dog_name = dog_name;
}
```

**빌더**
```java
String a = "woojae";
String b = "Charlie";

Example.builder()
          .a(a) -> 어디에 넣는지 명확함
          .b(b) -> 어디에 넣는지 명확함
          .build();

@Builder
public Example(String pesrson_name, String dog_name){
     this.person_name = person_name;
     this.dog_name = dog_name;
}
```

### 1.3.4. DataBase 접근을 위한 JpaRepository 생성   
posts 폴더에 인터페이스 파일 생성 

**PostsRepository**
```java
package com.jojoldu.book.springboot.domain.posts;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;

import java.util.List;

public interface PostsRepository extends JpaRepository<Posts,Long> {
}
```   
DAO를 JPA에서는 Repository라고 부르며 인터페이스로 생성한다.        
단순히 인터페이스를 생성한 후, ```JpaRepository<Entity 클래스, PK 타입>```을 상속하면      
**기본적인 CRUD 메소드가 자동으로 생성된다.**      
     
기본 양식은 ```@Repository``` 어노테이션도 추가해야 되지만 굳이 기술할 필요는 없고         
다만, **Entity 클래스와 EntityRepository는 함께 위치하는 것을 권장한다.**          
이는 나중에 프로젝트 규모가 커졌을시에 함께 움직여야 하므로 도메인 패키지에서 함께 관리한다.   

### 1.3.5. Spring Data JPA 테스트 코드 작성하기     
test 디렉토리에 domain.posts 패키지를 생성하고,    
테스트 클래스는 PostsRepositoryTest란 이름으로 생성한다.   

**PostsRepositoryTest**
```java
package com.jojoldu.book.springboot.web.domain.posts;

import com.jojoldu.book.springboot.domain.posts.Posts;
import com.jojoldu.book.springboot.domain.posts.PostsRepository;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.time.LocalDateTime;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {

    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanup(){
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기(){
        // given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        postsRepository.save(Posts.builder()
                                    .title(title)
                                    .content(content)
                                    .author("jojoldu@gmail.com")
                                    .build());

        // when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }
}
```
    
**소스 코드 설명**
```java
@After

     * Junit에서 단위 테스트가 끝날 때마다 수행되는 메소드를 지정  
     * 보통은 배포 전 전체 테스트를 수행할 때 테스트간 데이터 침범을 막기 위해 사용한다.  
     * 여러 테스트가 동시에 수행되면 테스트용 데이터베이스인 H2에 데이터가 그대로 남아 있어 다음 테스트 실행 시 테스트가 실패할 수 있다.
__________________________________________________________________________________________________________________
postsRepository.save()

     * 테이블에 posts에 insert/update 쿼리를 실행한다.  
     * id값이 있다면, update가 없다면 insert 쿼리가 실행된다.  
__________________________________________________________________________________________________________________
postsRepository.findAll()

     * 테이블 posts에 있는 모든 데이터를 조회해오는 메소드이다.  
```   
별다른 설정 없이 ```@SpringTest```를 사용할 경우 H2 데이터베이스를 자동으로 실행해준다.      
(스프링에서는 H2 데이터베이스를 디폴트로 사용하기 때문이다.)         
   
여기서 추가적인 팁으로 ```application.properties```에 코드 한줄만 추가하면 실제 실행되는 쿼리문을 볼수 있다. 
```src/main/resources``` 디렉토리 아래에 ```application.properties``` 생성 후 아래와 같이 작성 해주자    

**application.properties**
```
spring.jpa.show_sql = true
```

위와 같이 했을 경우 테이블 생성 쿼리가 ```id bigint gernertated by default as identity```로 출력된다.  
이는 h2 데이터베이스 기준으로 쿼리가 출력된 것인데 이를 mysql 버전으로 변경하고자 하면 아래와 같이 작성해주자
    
**application.properties**
```
....
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
...
```
## 1.4. 등록/수정/조회 API 만들기     
### 1.4.1. Service 와 Domain    

API를 만들기 위해 총 3개의 클래스가 필요하다.    
    
1. Request 데이터를 받을 Dto        
2. API 요청을 받을 Controller     
3. 트랜잭션, 도메인 기능 간의 순서를 보장하는 Service    
  
여기서 많은 사람들이 **Service 에서 비즈니스 로직을 처리**해야 된다고 생각하고 있다.       
하지만 이는 과거이고 큰 오해이며 현재는 Service는 **트랜잭션, 도메인 간 순서 보장의 역할**만 한다.          
    
우선 이를 설명하기 전에 Spring 웹 계층에 대해서 알아보자    
[사진]    

* Web Layer
     * 흔히 사용하는 @Controller 와 JSP/Freemarker 등의 뷰 템플릿 영역입니다.  
     * 이외에도 필터, 인터셉터, 컨트롤러 어드바이스등 외부 요청과 응답에 대한 전반적인 영역을 이야기합니다.  
  
* Service Layer
     * @Service에 사용되는 서비스 영역입니다.  
     * 일반적으로 Controller와 DAO의 중간 영역에서 사용된다.  
     * @Transactional 이 사용되어야 하는 영역이기도 한다.  

* Repository Layer
     * Database와 같이 데이터 저장소에 접근하는 영역이다.  
     * 기존 개발 영역으로 본다면 DAO 영역이다.  

* Dtos
     * Dto 는 계층 간에 데이터 교환을 위한 객체를 이야기하며 Dtos는 이들의 영역을 얘기합니다.     
     * 뷰 템플릿 엔진에서 사용될 객체나 Repositroy Layer에서 결과로 넘겨준 객체 등이 이들을 이야기합니다.  

* Domain Model  
     * 도메인이라 불리는 개발 대상을    
     모든 사람이 동일한 관점에서 이해할 수 있고 공유할 수 있도록 단순화시킨 것을 도메인 모델이라 한다.    
     * 이를테면 택시 앱이라고 하면 배차, 탑승, 요금 등이 모두 도메인이 될 수 있다.  
     * @Entity를 사용하는 클래스도 도메인 모델이라 할 수 있다.  
     * 다만 무조건 데이터베이스의 테이블과 관계가 있어야만 하는 것은 아니다.
     * VO처럼 값 객체들도 이 영역에 해당하기 때문이다.    
         
이 5가지 에리어에서 비지니스 처리를 담당하는 곳은? 바로 Domain이다.         
이전에는 Service 방식에서 처리를 했고 이를 트랜잭션 스크립트라 불렀는데       
도메인이라는 개념이 생기고 나서 이러한 방식이 바뀌게 되었다.    
    
**트랜잭션 스크립트 - 과거 Service 방식**
```java
```
모든 로직이 Service 클래스 내부에서 처리된다.     
그러다 보니 서비스 계층이 무의미하며, 객체란 단순히 데이터 덩어리 역할만 하게 된다.   
   
반면, 도메인 모델(객체)에서 처리할 경우 다음과 같은 코드가 될 수 있다.     
     
**현재 Service 방식**    
```java     
    
```    
     
이해하기 쉽게 말하자면 Enitity 클래스에 메소드를 만들어서 처리하게끔 유도한 것이다.       
order, billing, delivery가 각자 본인의 취소 이벤트 처리를 하며,       
서비스 메소드는 **트랜잭션과 도메인 간의 순서만 보장해 준다.**      
       
조금 더 쉽게 얘기하고자 한다면 update 원할시 Entity에 update 메소드를 정의하고     
Service는 단순히 해당 객체와 메소드를 호출 및 순서만 보장해주면 된다.     
      
### 1.4.2. 등록/수정/삭제 기능 만들기  
PostsApiController를 web 패키지에,     
PostsService를 service 패키지에,
PostsSaveRequestDto를 web.dto 패키지에 생성한다.         

**PostsApiController**
```java   
package com.jojoldu.book.springboot.web;

import com.jojoldu.book.springboot.service.posts.PostsService;
import com.jojoldu.book.springboot.web.dto.PostsResponseDto;
import com.jojoldu.book.springboot.web.dto.PostsSaveRequestDto;
import com.jojoldu.book.springboot.web.dto.PostsUpdateRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;

@RequiredArgsConstructor
@RestController
public class PostsApiController {
    private final PostsService postsService; // 생성자로 주입

    @PostMapping("/api/v1/posts")
    public Long save(@RequestBody PostsSaveRequestDto requestDto){
        return postsService.save(requestDto);
    }

    @PutMapping("/api/v1/posts/{id}")
    public Long update(@PathVariable Long id, @RequestBody PostsUpdateRequestDto requestDto){
        return postsService.update(id, requestDto);
    }

    @GetMapping("/api/v1/posts/{id}")
    public PostsResponseDto findById (@PathVariable Long id) {
        return postsService.findById(id);
    }

    @DeleteMapping("/api/v1/posts/{id}")
    public Long delete(@PathVariable Long id){
        postsService.delete(id);
        return id;
    }

}
```

**PostsService**
```java
package com.jojoldu.book.springboot.service.posts;

import com.jojoldu.book.springboot.domain.posts.Posts;
import com.jojoldu.book.springboot.domain.posts.PostsRepository;
import com.jojoldu.book.springboot.web.dto.PostsListResponseDto;
import com.jojoldu.book.springboot.web.dto.PostsResponseDto;
import com.jojoldu.book.springboot.web.dto.PostsSaveRequestDto;
import com.jojoldu.book.springboot.web.dto.PostsUpdateRequestDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;


@RequiredArgsConstructor
@Service
public class PostsService {
    private final PostsRepository postsRepository;

    @Transactional
    public Long save(PostsSaveRequestDto requestDto){
        return postsRepository.save(requestDto.toEntity()).getId();
    }

    @Transactional
    public Long update(Long id, PostsUpdateRequestDto requestDto){
        Posts posts = postsRepository.findById(id).orElseThrow(() -> new
                IllegalArgumentException("해당 게시글이 없습니다. id="+ id));
        posts.update(requestDto.getTitle(), requestDto.getContent());

        return id;
    }

    @Transactional
    public PostsResponseDto findById(Long id){
        Posts entity = postsRepository.findById(id).orElseThrow(() -> new
                IllegalArgumentException("헤당 게시글이 없습니다. id="+id));
        return new PostsResponseDto(entity);
    }

    @Transactional
    public void delete (Long id){
        Posts posts = postsRepository.findById(id).orElseThrow(()->new
                IllegalArgumentException("해당 게시글이 없습니다. id="+ id));
        postsRepository.delete(posts);
    }

}
```    
   
스프링에서 Bean을 주입받는 방식은 3가지이다.
1. ```@Autowired``` (필드)
2. setter
3. 생성자
   
기존 스프링에서는 ```@Autowired (필드)```을 주로 사용하지만 이는 좋은 방법은 아니다.  
```@Autowired (필드)```로 생성할 경우 생기는 문제점은 아래와 같다.

1. 단일 책임의 원칙 위반 (생성자의 파라미터가 많아짐에 리팩토링을 할 가능성 증대된다.)   
2. 의존성이 숨는다 (숨은 의존성만 제공)
3. DI 컨테이너의 결합성과 테스트 용이성 (DI 컨테이너와 의존성을 가진 클래스는 곧바로 인스턴스화 할 수 없다.)
4. 불변성 (final 선언 불가로 객체가 변할 가능성이 있다.)

그렇기에 가장 권하는 방식은 **생성자로 주입** 받는 방식으로    
```@RequiredArgsConstructor```에서 final이 선언된 모든 필드를 인자값으로 하는 생성자를 만들어준다.   
      
**PostsSaveRequestDto**
```java
package com.jojoldu.book.springboot.web.dto;

import com.jojoldu.book.springboot.domain.posts.Posts;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class PostsSaveRequestDto {
    private String title;
    private String content;
    private String author;

    @Builder
    public PostsSaveRequestDto(String title, String content, String author){
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public Posts toEntity(){
        return Posts.builder()
                .title(title)
                .content(content)
                .author(author)
                .build();
    }
}
```

**PostsUpdateRequestDto**
```java
package com.jojoldu.book.springboot.web.dto;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class PostsUpdateRequestDto {
    private String title;
    private String content;

    @Builder
    public PostsUpdateRequestDto(String title, String content){
        this.title = title;
        this.content = content;
    }

}

```

**PostsResponseDto**
```java
package com.jojoldu.book.springboot.web.dto;

import com.jojoldu.book.springboot.domain.posts.Posts;
import lombok.Getter;

@Getter
public class PostsResponseDto {
    private Long id;
    private String title;
    private String content;
    private String author;

    public PostsResponseDto(Posts entity){
        this.id = entity.getId();
        this.title = entity.getTitle();
        this.content = entity.getContent();
        this.author = entity.getAuthor();
    }
}
```
      
Entity(도메인) 클래스와 유사한 Dto 클래스를 생성했다.         
그 이유는 Entity 클래스를 ```Request/Response``` 클래스로 사용하는 것은 매우 안좋기 때문이다.    
   
Entity 클래스는 데이터베이스와 맞닿은 핵심 클래스이다.     
Entity 클래스를 기준으로 테이블이 생성되고, 스키마가 변경된다.      
화면 변경은 아주 사소한 기능 변경인데, 
이를 위해 테이블과 연결된 Entity 클래스를 변경하는 것은 너무 큰 변경이다.    
Request/Response용 Dto는 View를 위한 클래스라 정말 자주 변경이 필요하다.  

View Layer와 DB Layer의 역할 분리를 철저하게 하고      
Controller에서 결괏값으로 여러 테이블을 조인해서 줘야할 경우가 빈번하므로   
Entity 클래스만으로 표현하기 어려운 경우가 많다.    
그렇기에 되도록 Entity 클래스와 Controller에서 쓸 Dto는 분리해서 사용하도록 하자  

**Posts**
```java
package com.jojoldu.book.springboot.domain.posts;

import com.jojoldu.book.springboot.domain.BaseTimeEntity;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

import javax.persistence.*;

@Getter
@NoArgsConstructor
@Entity
public class Posts extends BaseTimeEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long Id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column( columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Posts(String title, String content, String author){
        this.title = title;
        this.content = content;
        this.author = author;
    }

    public void update(String title, String content){
        this.title = title;
        this.content = content;
    }
}
```
여기서는 아주 신기한 것이 있다.   
PostsService 코드를 보면 update 기능에서는 업데이트 쿼리를 사용하는 부분이 없다.    
이는 바로 JPA의 **영속성 컨텍스트** 때문이다.  

**영속성 컨텍스트**       
엔티티를 영구 저장하는 환경     
일종의 논리적 개념이라 보면 되고, JPA의 핵심 내용은 엔티티가 영속성 컨텍스트에 포함되어 있냐 아니냐로 갈린다.      
JPA의 엔티티 매니저가 활성화된 상태로 트랜잭션 안에서 데이터베이스에 데이터를 가져오면     
이 데이터는 영속성 컨텍스트가 유지된 상태이다.      
이 상태에서 해당 데이터의 변경하면 트랜잭션이 끝나는 시점에 해당 테이블에 변경분을 반영한다.    
즉, Entity 객체의 값만 변경하면 별도로 update 쿼리를 사용하지 않아도 된다.      
이 개념을 **더티체킹**이라 한다.     

### 1.4.3. 등록/수정/삭제 테스트   
**PostsApiControllerTest**
```java   
package com.jojoldu.book.springboot.web;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.jojoldu.book.springboot.domain.posts.Posts;
import com.jojoldu.book.springboot.domain.posts.PostsRepository;
import com.jojoldu.book.springboot.web.dto.PostsSaveRequestDto;
import com.jojoldu.book.springboot.web.dto.PostsUpdateRequestDto;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.http.*;
import org.springframework.security.test.context.support.WithMockUser;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import org.springframework.web.context.WebApplicationContext;

import java.util.List;

import static org.springframework.security.test.web.servlet.setup.SecurityMockMvcConfigurers.springSecurity;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;

import static org.assertj.core.api.Assertions.assertThat;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class PostsApiControllerTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private PostsRepository postsRepository;

    @Autowired
    private WebApplicationContext context;

    @After
    public void tearDown() throws Exception{
        postsRepository.deleteAll();
    }

    @Test
    public void Posts_등록된다() throws Exception{
        // given
        String title = "title";
        String content = "content";
        PostsSaveRequestDto requestDto = PostsSaveRequestDto.builder()
                                                                .title(title)
                                                                .content(content)
                                                                .author("author")
                                                                .build();

        String url = "http://localhost:"+ port + "/api/v1/posts";

        //when 
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

        //then
        
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);
        
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(title);
        assertThat(all.get(0).getContent()).isEqualTo(content);
    }
    @Test
    public void Posts_수정된다() throws Exception{
        // given
        Posts savedPosts = postsRepository.save(Posts.builder()
                                                        .title("title")
                                                        .content("content")
                                                        .author("author")
                                                        .build());

        Long updateId = savedPosts.getId();
        String expectedTitle = "title2";
        String expectedContent = "content2";

        PostsUpdateRequestDto requestDto = PostsUpdateRequestDto.builder()
                                                                    .title(expectedTitle)
                                                                    .content(expectedContent)
                                                                    .build();

        String url = "http://localhost:"+ port + "/api/v1/posts/" + updateId;

        
        //when 
        ResponseEntity<Long> responseEntity = restTemplate.postForEntity(url, requestDto, Long.class);

        //then
        
        assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(responseEntity.getBody()).isGreaterThan(0L);
        
        List<Posts> all = postsRepository.findAll();
        assertThat(all.get(0).getTitle()).isEqualTo(expectedTitle);
        assertThat(all.get(0).getContent()).isEqualTo(expectedContent);
    }

}
```
이전과 달리 ```@WebMvcTest```를 사용하지 않았는데 이유는 JPA 기능이 작동하지 않기 때문이다.    
그렇기에 외부 연동 및 JPA 기능까지 한번에 테스트할 때는 ```@SpringBootTest```와 ```TestRestTemplate```을 사용하면 된다.    

### 1.4.4. H2 데이터베이스에 접근해보기  
로컬 환경에선 데이터베이스로 H2를 주로 사용한다.      
메모리에서 실행하기 때문에 **직접 접근하려면 웹 콘솔을 사용**해야만 한다.      
       
먼저 아래와 같은 방법으로 웹 콘솔 옵션을 활성화 하자     
   
**application.properties**    
```
spring.h2.console.enable=true
```
추가한 뒤 ```Application.class``` 의 main 메소드를 실행하고  
웹 브라우저에 ```http://localhost:8080/h2-console``` 로 접속하자   
그 후 JDBC URL이 ```jdbc:h2:mem:testdb```가 쓰여져 있는지 확인 후 connect 버튼을 눌러주자
이후 ```select * from posts```와 같은 간단한 쿼리를 입력해보면 쿼리가 정상적으로 실행된다.  
물론 아직 insert를 하지 않았지만 insert 후에 확인해보면 데이터가 정상 출력되는 것을 알 수 있다.  

## 1.5. JPA Auditing으로 생성시간/수정시간 자동화하기   
보통 엔티티에는 해당 데이터의 생성시간과 수정시간을 포함한다.               
언제 만들어졌는지, 언제 수정되었는지 등은 차후 유지보수에 있어 굉장히 중요한 정보이기 때문이다.           
그래서 DB에 삽입/갱신하기 전에 날짜 데이터를 등록해주는데            
이런 단순하고 반복적인 코드가 모든 테이블과 서비스 메소드에 포함되어있다면 이는 매우 귀찮고 지저분해진다.             
그래서 이러한 문제를 해결하기 위해서 JPA Auditing을 사용하자          

### 1.5.1. LocalDate 사용  
자바 8 부터는 Date 대신에 LocalDate를 사용한다.     
   
```
Date와 Calendar 클래스의 문제점 

1. 불변 객체가 아니다. (멀티스레드 환경에서 문제 발생 가능성 있음)    
2. Calendar는 월(Month)값 설계가 잘못되었다. (10월을 나타내는 Calendar.OCTOBER의 숫자값은 9이다)  
```
domain 패키지에 BaseTimeEntity 클래스를 생성한다.     
       
**BaseTimeEntity**
```java
package com.jojoldu.book.springboot.domain;

import lombok.Getter;
import org.springframework.data.annotation.CreatedDate;
import org.springframework.data.annotation.LastModifiedDate;
import org.springframework.data.jpa.domain.support.AuditingEntityListener;

import javax.persistence.EntityListeners;
import javax.persistence.MappedSuperclass;
import java.time.LocalDateTime;

@Getter
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public abstract class BaseTimeEntity {

    @CreatedDate
    private LocalDateTime createdDate;

    @LastModifiedDate
    private LocalDateTime modifiedDate;

}

```
BaseTimeEntity 클래스는 모든 Entity의 상위 클래스가 되어        
**Entity들의 createDate, modifiedDate를 자동으로 관리하는 역할이다.**                

**코드 설명**
```java
@MappedSuperclass

     * JPA Entity 클래스들이 BaseTimeEntity을 상속할 경우 필드들도 createdDate 와 modifiedDate '컬럼'으로 인식하도록합니다.  
______________________________________________________________________________________________
@EntityListeners(AuditingEntityListener.class)

     * BaseTimeEntity 클래스에 Auditing 기능을 포함시킨다.  

감사(Auditing)란?
  - 의심가는 데이터베이스의 작업을 모니터링 하고, 기록 정보를 수집 하는 기능 입니다.
  - 어느시간때에 어떤 작업들이 주로 발생하는지, 어떤 작업을 누가 하는지 추적 할 수 있습니다.
  - 감사 작업을 하면, 감사 로그를 기록해야 하므로 시스템의 속도는 더 느려질 수 밖에 없습니다.
______________________________________________________________________________________________
@CreateDate

     * Entity가 생성되어 저장될 때 시간이 자동 저장됩니다.  
______________________________________________________________________________________________
@LastModifiedDate

     * 조회한 Entity의 값을 변경할 때 시간이 자동 저장됩니다.  
______________________________________________________________________________________________
```
그리고 Posts 클래스가 BaseTimeEntity를 상속받도록 변경한다.     
   
**Posts**
```java
     ...
public class Posts extends BaseTimeEntity{
     ...
}
```
마지막으로 JPA Auditing 어노테이션들을 모두 활성화할 수 있도록     
Application 클래스에 활성화 어노테이션을 하나 추가하자     
       
**Application**
```java
@EnableJpaAuditing // 감사 -> 최소 1개 엔티티가 있어야함
@SpringBootApplication
public class Application{
     public static void main(String[] args){
          SpringApplication.run(Application.class, args);
     }
}
```
### 1.5.1. JPA Auditing 테스트 코드 작성하기  
기존에 작성했던 PostsRepositoryTest 클래스에 테스트 메소드를 하나 더 추가해주자  

**PostsRepositoryTest**
```java
package com.jojoldu.book.springboot.web.domain.posts;

import com.jojoldu.book.springboot.domain.posts.Posts;
import com.jojoldu.book.springboot.domain.posts.PostsRepository;
import org.junit.After;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.time.LocalDateTime;
import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PostsRepositoryTest {

    @Autowired
    PostsRepository postsRepository;

    @After
    public void cleanup(){
        postsRepository.deleteAll();
    }

    @Test
    public void 게시글저장_불러오기(){
        // given
        String title = "테스트 게시글";
        String content = "테스트 본문";

        postsRepository.save(Posts.builder()
                                    .title(title)
                                    .content(content)
                                    .author("jojoldu@gmail.com")
                                    .build());

        // when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);
        assertThat(posts.getTitle()).isEqualTo(title);
        assertThat(posts.getContent()).isEqualTo(content);
    }

    @Test
    public void BaseTimeEntity_등록(){
        // given
        LocalDateTime now = LocalDateTime.of(2019,6,4,0,0,0);
        postsRepository.save(Posts.builder()
                                    .title("title")
                                    .content("content")
                                    .author("author")
                                    .build());

        //when
        List<Posts> postsList = postsRepository.findAll();

        //then
        Posts posts = postsList.get(0);

        System.out.println(">>>>>>>>> createDate="+posts.getCreatedDate()+
                ", modifiedDate="+posts.getModifiedDate());

        assertThat(posts.getCreatedDate()).isAfter(now);
        assertThat(posts.getModifiedDate()).isAfter(now);
    }
}

```

앞으로 추가될 엔티티티들은 더이상 등록일/수정이로 고민할 필요가 없다.    
바로 BaseTimeEntity만 상속받으면 자동으로 해결되기 때문이다.  
