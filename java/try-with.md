
try-with-resource

클래스는 implements AutoClosable 필요 

@Override 
close() <--- 구현 필요 

try( ...) > 여기서 exception 나면 핵심예외에 포함시켜서 같이 밖으로 던져줌
Suppressed <----- 
 
try(a){
... -> 예외 발생 -> a close() -> catch 블록으로 이동 
}


