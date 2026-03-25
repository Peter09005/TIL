```java 

FileOnputStream fos = new FileOutputStream("temp/data.dat")
fos.write(65)

/*
* data.dat -> A
*/
```

#### 스트림 

바이트 단위로 자료가 이동함 
이렇게 데이터를 주고받는걸 I/O 라고 함 

I/O 

하드디스크에 지속적으로 접속하는건 너무 시간이 오래 걸림 
Buffer 를 만들어서 어느정도 채워졌을때 한번에 읽고 쓰게 만들면 
속도가 개선됨 

```java
BufferedInputStream - read from HDD
BufferedOutputStream - write to HDD 

전부 I/O 작업이다. -> 속도가 느릴수밖에없음 
따라서 Buffered는 생성할때 

new BufferedInputStream("target stream instance", buffered_size) 
로 초기화해준다. 

buffered 사이즈까지 채워지면 그때 I/O 작업을 실행한다. 
```


직접 구현한 bufferedStream 보다 자바에서 제공해주는 BufferedInputStream이 일반적으로 좀 성능이 
떨어진다. 

그 이유는 동기화 문제에 있다. 

멀티쓰레드 환경에서 각 BufferedInputStream , OutputStream은 모든 쓰레드에서 접근 가능하다. 
Buffer에 읽고 쓰는건 모든 쓰레드에서 공유되는 자원이기 때문에 lock , synchronized를 사용해 동시성 문제를 
해결해야함. 

-> bufferedStream은 이 점을 고려하고 설계가 되었기 때문에 lock , synchronized 부분을 코드에 넣어두었다. lock , synchronized 도 일반 코드에 비해 비용이 비싼 편이라 직접 구현한 코드보다는 느린 편에 속한다. 

멀티쓰레드를 지원한다고 해서 모든걸 지원하는건 아니다. 다음을 생각해보자. 

Thread A  HELLO WORLD 

Thread B  GOOD BYE

BufferedOutputStream bos (size = 5)

ThreadA - > ThreadB -> ThreadA -> 이런식으로 스케쥴링 큐에 잡힌다고 하면 

HELLO GOOD WORLD .... 

이런식으로 두 정보가 아예 섞여버리는 경우도 발생한다. 

이런 경우는 전담 쓰레드를 두던가, buffered 를 늘리던가, 파일락을 걸던가 해야한다. 

#### 객체 스트림

```java
class Member{
	int age; 
	String name; 
}

member.dat 에 age , name 저장되어있다 가정

FileWriter() -> 

"ABC" -> FileWriter -> 주어진 인코딩 방식으로 byte로 변환 -> FileOutputStream이 파일에 작성 

age = 10 , name = "hwang" 을 전달해도 결국 FileWriter를 거치면 String 으로 encoding된다. 결국 읽을때나 Member 객체를 초기화 해줄떄 decoding을 귀찮게 거쳐야한다. 

DataOutputStream으로 개선해줄순 있지만 결국 필드 하나하나 조회하면서 타입에 따라 다른 메소드를 호출해서 넣어주는 작업을 거쳐야하므로 귀찮다. 
```

객체를 애초에 저장할때부터 메모리에 직렬로 보관하면 편하지 않을까? 

애초에 물리RAM에서는 객체를 연속해서 저장해두지 않는다. 
객체 단위로 I/O를 하려면 Outpu