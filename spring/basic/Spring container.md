
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



