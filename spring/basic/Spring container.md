
ApplicationContext 를 스프링 컨테이너라고 한다 

컨테이너에 빈을 등록시키는 방법은 다양하지만 먼저 Config.class 를 만들어 직접 넣어주는 방법부터 알아보자


### Config.class 

```java
@Configuration
public class Config{
	
	@Bean
	public Car carA{
		return new CarA()
	}
	@Bean 
	public Car carB{
		return new CarB()
	}
}
```


### Test.Main 

```java
ApplicationContext ac;
ac = new AnnotationConfigApplicationContext(TestConfig.class);

Car carA = ac.getBean("carA",Car.class) // ok

Car carB = ac.getBean(Car.class) // not ok 

HashMap<String,Car> hashMap = ac.getBeansOfType(Car.class) // ok 
```

### 빈 조회

부모타입으로 조회하면, 주식 타입도 함께 조회한다. 

### 싱글톤 

~~~java
void pureContainer(){  
    AppConfig appConfig = new AppConfig();  
  
    MemberService ms1 = appConfig.memberService();  
    MemberService ms2 = appConfig.memberService();  
  
    System.out.println("ms1 = " + ms1);  
    System.out.println("ms2 = " + ms2);  
  
    Assertions.assertThat(ms1).isNotEqualTo(ms2);  
}
~~~

사용자가 서비스를 이용하려고 할떄마다 새로운 객체를 반환하는건 메모리낭비가 너무 심하다. 
서비스 객체를 딱 하나 만들어넣고 그 객체를 사용자들이 공유하게 설계하면된다. 
이를 우린 **싱글톤 패턴** 이라고 한다. 


~~~java
public class SingletonService{  
	private static final SingletonService instance = new SingletonService();  
  
    public static SingletonService getInstance(){  
        return instance;  
    }  
  
    private SingletonService(){ // 밖에서 SingletonService 객체 생성을 막아줌  
    }  
  
    public void login(){  
        System.out.println("싱글톤 객체 로직 호출");  
    }  
}
~~~

#### 싱글톤패턴

~~~java
private static final SingletonService instance = new SingletonService();

/*SingletonSerive는 DIP를 위반했다. 
클라이언트가 구체클래스인 SingletonService에 의존하고 있어 OCP 원칙을 위반할수있다.*/ 
~~~

