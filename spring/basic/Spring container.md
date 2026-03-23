
ApplicationContext 를 스프링 컨테이너라고 한다 

컨테이너에 빈을 등록시키는 방법은 다양하지만 먼저 Config.class 를 만들어 직접 넣어주는 방법부터 알아보자


#### Config.class 

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


#### Test.Main 

```java
ApplicationContext ac;
ac = new AnnotationConfigApplicationContext(TestConfig.class);

Car carA = ac.getBean("carA",Car.class) // ok

Car carB = ac.getBean(Car.class) // not ok 

HashMap<String,Car> hashMap = ac.getBeansOfType(Car.class) // ok 
```

### 빈 조회

부모타입으로 조회하면, 주식 타입도 함께 조회한다. 

#### 싱글톤 

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

#### 싱글톤패턴의 문제

~~~java
private static final SingletonService instance = new SingletonService();

/*SingletonSerive는 DIP를 위반했다. 
*클라이언트가 구체클래스인 SingletonService에 의존하고 있어 OCP 원칙을 위반할수있다.
*생성자가 private이라 자식 클래스로 확장시키기도 어렵고, 내부속성을 변경하거나 초기화하기도 힘들다. 
*/ 
~~~

#### 싱글톤 컨테이너 

스프링 컨테이너에서 관리되는 스프링 빈의 기본 등록방식은 싱글톤으로 관리된다. 
스프링이 싱글톤패턴의 문제를 다 관리해주기 때문에, 코딩하기가 훨씬 편해졌다. 

#### 싱글톤 방식의 주의점

많은 사용자가 동시에 동일한 메모리에 있는 객체를 사용하다보니 , 공유자원이 존재하면 문제가 생긴다 
싱글톤 객체는 항상 무상태로 설계해야한다.


#### Configuration 과 싱글톤 

~~~java
@Configuration  
public class AppConfig {  
    @Bean  
    public MemberService memberService(){  
        return new MemberServiceImpl(memberRepository());  
    }  
    @Bean  
    public OrderService orderService(){  
        return new OrderServiceImpl(memberRepository(),discountPolicy());  
    }  
    @Bean  
    public DiscountPolicy discountPolicy(){  
        return new RateDiscountPolicy();  
    }  
    @Bean  
    public MemberRepository memberRepository(){  
        return new MemoryRepository();  
    }  
}
~~~

Configuration 빈을 보면 orderService, memberService 를 호출하게 되면 두 메소드는 memberRepository 메소드를 실행해 객체를 생성하는것처럼 보인다. 

직관적으로는 2개의 객체가 만들어져 싱글톤 패턴이 깨진것 처럼 보이지만, 
사실은 그렇지않다. 

스프링은 클래스의 바이트코드를 조작하는 라이브러리를 사용해 위 문제를 해결했다. 

```        
AppConfig ----> class AppConfig@CGLIB extends AppConfig {...}
```

1. CGLIB는 스프링 컨테이너가 만들어질때 AppConfig를 상속받는 프록시를 메모리에 동적으로 만들어냄 이 자식클래스는 AppConfig에 있는 모든 @Bean 메소드를 오버라이딩 함
  ↓
2.  외부에서 프록시 객체의 memberRepository()가 호출되면 프록시에 오버라이딩된 memberReposirtory( )가 실행됨 
  ↓
3. 오버라이딩 한 메소드 내부에는 intercept() 라는 메소드가 있는데, 외부에서 프록시 객체의 메소드를 호출하면 이 함수를 통해 제어권을 가져감
  ↓
4. intercept() 메소드에서 컨테이너에 빈이 있는지 확인후 있으면 기존 빈을 반환,
   컨테이너에 없다면 cglibMethodProxy.invokeSuper()로 진짜 memberRepository 가 실행됨 

#### 스프링 컨테이너와 ComponentScan 

```java
@Configuration
@ComponentScan 
public class AutoConfig{
	
}
```

@ComponentScan -> @Component로 설정되어있는 클래스를 전부 스캔해 컨테이너에 빈으로 등록해준다 

빈으로 설정할 클래스는 위에 @Component 애노테이션을 붙여줘야함.

등록될 빈의 이름은 보통 Class의 앞 대문자를 소문자로 바꾼형태로 저장되는게 기본이고, 
Component("name") 으로 이름을 변경할 수 있다. 

```java
@Component
public class A{
	private final B b;
	private final C c;

	@Autowired
	public A(B b,C c){
		this.b = b;
		this.c = b; 
	}
	
	public A(B b){
		this.b = b; 
	}
}
```

이런 경우 의존관계를 주입해줘야 하는 상황인데, 컴포넌트 스캔만으로는 의존관계까진 알수없다. 
의존관계를 주입받은 메소드에 @Autowired 애노테이션을 붙여주면 컨테이너에 저장되어있는 
객체를 가져와 주입시켜준다. 

#### 스캔의 범위

ComponentScan에 basePackages = "패키지 디렉토리" 를 설정해주면 해당 디렉토리부터 스캔한다. 

지정하지 않으면 @ComponentScan이 붙은 설정정보의 패키지부터 스캔한다.

#### 스캔 필터 

```java
@ComponentScan(
	includeFilters = @Filter(type = FilterType.ANNOTATION, 
	classes = MyIncludeComponent.class), 
	excludeFilters = @Filter(type = FilterType.ANNOTATION,
	classes = MyExcludeComponent.class)
	)
)
```

ComponentScan 에 필터 옵션을 넣어주면 스캔할때 필터 옵션에 맞춰 알아서 컨테이너에 등록해준다. 

#### lombok 



#### DI 과정에서 타입이 같은 빈이 여러개일떄

1. Autowired

2. Qualifier

3. Primary


#### Component Scan vs 수동 빈 등록 

