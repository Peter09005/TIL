
```java
public class VolatileMain {  
    static void main(String[] args) {  
        MyTask myTask = new MyTask();  
        Thread thread = new Thread(myTask,"t1");  
  
        thread.start();  
        Thread.sleep(1000);
        log("runFlag false");
        myTask.runFlag = false;  
    }  
  
    static class MyTask implements Runnable{  
        boolean runFlag = true;  
        @Override  
        public void run() {  
            log("task 시작");  
            while(runFlag){  
            }  
            log("task 종료");  
        }  
    }  
}


/*
output:

02:09:41.483 [       t1] task 시작
02:09:42.473 [     main] runFlag false
*/
```

main 쓰레드에서 runFlag을 false로 바꿨는데 t1 쓰레드가 while문에서 벗어나지 못하고있음 


### Cache 메모리 

while (runflag) -> OS 입장에서 보면 이 runflag 메모리 주소에 정말 많이 접근할거라 예측한다. 

CPU <------> 메모리 에서 읽고 쓰는데에는 많은 비용이 필요해 
CPU는 주로 코어마다 캐쉬메모리를 가지고 있다. 

이 경우 

t1 쓰레드를 실행하는 cpu의 cache 메모리에 runFlag = true 로 저장되어있다. 
따라서 main 쓰레드가 실제 메모리의 값을 바꿔도 cpu는 cache 메모리의 값을 읽기떄문에
while 문을 빠져나올수없다. 

CPU 설계 방식에 따라 cache 메모리를 다시 read/ write 하는게 다 다르기에 예측 불가


### 메모리 가시성

한 쓰레드에서 변경된 값이 다른 쓰레드에 언제 보이는지에 대한 문제를 메모리 가시성이라고 한다. 
성능을 포기하는대신 read, write 할때 계속 메인메모리에 접근하라고 설정해주면됨 

```java
package thread.class4;  
  
import static util.MyLogger.log;  
import static util.ThradUtils.sleep;  
  
  
public class VolatileMain {  
    static void main(String[] args) {  
        MyTask myTask = new MyTask();  
        Thread thread = new Thread(myTask,"t1");  
  
        thread.start();  
        sleep(1000);  
        log("runFlag false");  
        myTask.runFlag = false;  
    }  
  
    static class MyTask implements Runnable{  
        volatile boolean runFlag = true;  
        @Override  
        public void run() {  
            log("task 시작");  
            while(runFlag){  
            }  
            log("task 종료");  
        }  
    }  
}

/*
02:26:50.323 [       t1] task 시작
02:26:51.316 [     main] runFlag false
02:26:51.316 [       t1] task 종료
*/
```

### JMM - happens-before 

happens before은 JMM에서 쓰레드 간 작업 순서를 정의하는 개념 

A happens before B 라면 다음을 보장한다

-  A 가 B보다 먼저 발생함을 보장한다
-  A와 B 간 메모리 가시성을 보장한다
-  A 가 한 작업을 B 가 참조할때 항상 최신 상태가 보장된다  즉 A작업에서 변경된 내용은 B 작업이 시작ㅎ


