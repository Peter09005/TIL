
ApplicationContext 를 스프링 컨테이너라고 한다 

스프링 컨테이너 --->  @Configuration 이 적힌 config 파일에 있는 @Bean 을 호출해 스프링 컨테이너에 등록해놓는다. 

이렇게 스프링컨테이너에 등록된 객체를 스프링 빈이라고 한다. 

스프링빈에 저장되어있는 객체는 Config에 적힌 함수명을 스프링빈의 이름으로 한다. 

스프링빈은 applicationContext.getBean() 메소드로 찾을수있다. 
