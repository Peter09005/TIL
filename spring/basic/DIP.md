
```java
interface Car 


class CarA implements Car 


class CarB implements Car 


interface Drive{
	Car c = new CarA() // DIP, OCP 위반  
}
```

이렇게 구현한다면 Drive 인터페이스는 단순히 운전한다는 역할을 넘어 
Car 을 생성한다는 역할도 생긴다

이는 SRP 법칙에 위반될 뿐만 아니라 
사용자가 차를 바꿀때마다 Drive의 코드를 바꿔야해 
OCP , DIP 법칙을 따르지 않는 모습이다. 

역할을 분리하려면 

Drive에 Car을 생성한다는 역할을 구분하면된다. 

```java

```