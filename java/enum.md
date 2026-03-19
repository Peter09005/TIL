
```java
public enum Color{
	RED,GREEN,BLUE
}


// 컴파일 후 

public final class Enum extends java.lang.Enum<COLOR>{
	public static final Color RED = new COLOR("RED",0);
	public static final Color GREEN = new COLOR("GREEN",1);
	public static final Color BLUE = new COLOR("BLUE",2);
	
	private Color(String name, int ordinal) { super(name, ordinal); }
}
```


### 왜 == 를 쓰는게 좋을까? 

~~~
Color color = null;
if (myColor.equals(Color.RED)) -> nullpointerException 

.equals() 메소드는 Object o 를 파라미터로 받음
Color.RED.equals(Season.SPRING)) - > 컴파일 정상적으로 됨 

enum은 싱글톤 객체라 == 로 객체만 비교해도 괜찮음. 
~~~
