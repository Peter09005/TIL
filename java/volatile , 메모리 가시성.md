
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

