사용자가 page , size 를 get 으로 보낸다고 가정하자. 

CLIENT HTTP : GET /posts?page=0&size=10 > Controller 
controller 가 page = 1, size = 10 받고 Service 호출 

Service에서 
pagerequest(1,10) 생성 

repository 가 JPA 쿼리를 실행 
JPA 가 SQL로 바꿔서 DB에 보냄 

DB와 JPA는 JDBC 기반으로 통신 
내부적으로 Hibernate 같은 구현체가 SQL를 만듬 

Service call Repository.get() -> JPA가 내부적으로 SQL 구문을 만듦 -> DB가 해당 구문을 읽고 테이블 형태로 결과 반환 

JPA는 해당 테이블을 POST 객체로 바꿔줌 
Service 는 해당 엔티티를 그대로 내보내지않고 DTO 로 바꿔줌 
Controller가 이 객체를 가지고 request에 넣어서 보내줌 

JPA Page<T> 를 사용한다면, 내부적으로 count 쿼리를 한번 더 보낸다 

Page<Post> page = postRepository.findAll(pageable); 를 보낸다면 

실제 데이터 조회 쿼리와 전체 개수 조회 쿼리 두개를 보낸다는거다 

Page 객체안에는 
```java
page.getContent();        // 현재 페이지 글 목록
page.getNumber();         // 현재 페이지 번호
page.getSize();           // 한 페이지 크기
page.getTotalElements();  // 전체 데이터 수
page.getTotalPages();     // 전체 페이지 수
page.hasNext();           // 다음 페이지 있나
```

같이 부가정보가 있는데, 이걸 활용해서 thymeleaf에서 넘버링을 할때 쓰나보다. 
