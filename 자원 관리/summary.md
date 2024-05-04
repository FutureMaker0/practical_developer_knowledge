# 체계적인 자원 관리

## 자바 메모리 관리
  - C언어는 할당한 메모리를 직접 해제해줘야 한다.
  - 자바는 메모리 해제 함수가 따로 없기 때문에 개발자가 직접 메모리를 해제할 수는 없다.
  - JVM(Java Virtual Machine)이라고 하는 가상 머신이 이 역할을 수행한다.

## JVM(Java Virtual Machine)
  - 자바 애플리케이션이 사용하지 않는 메모리를 찾아내 주기적으로 정리하는데, 이 과정을 가비지 컬렉션(garbage collection) 이라고 한다.
  - 접근하는 프로세스가 있는 메모리는 사용 중으로 판단하고, 그렇지 않은 메모리는 사용하지 않는다고 판단한다.

## Garbage Collection 발생 시점
> Visual VM과 같은 자원 모니터링 도구를 활용하여 heap 메모리 크기를 관찰하여 메모리 누수 여부를 확인할 수 있다.
> 전반적으로 우상향 하는 그래프일 때, 가비지 정리가 제대로 되지 않아 메모리 누수가 발생하는 것으로 판단할 수 있다.
  - 예제 코드 1
    ```java
    String exData = new String("exData");
    System.out.println(exData);
    ```
    - new를 통해 문자열 객체를 생성하면 heap 영역에 메모리를 할당하고, 해당 위치의 주소를 반환하여 반환된 주소 값을 exData 변수에 할당한다. 이후 exData를 통해 메모리에
      접근하면, 이 메모리는 사용 중인 것으로 판단하여 가비지 컬렉션의 대상이 아니다.
  - 예제 코드 2
    ```java
    String exData = new String("exData");
    System.out.println(exData);
    exData = null;
    ```
    - exData 변수에 null을 주면 더 이상 메모리를 참조할 수 없게 된다. 메모리 주소를 참조하는 변수가 존재하지 않기 때문에 가비지 컬렉션의 대상이 된다.
    - 특정 변수에 null을 주면 모두 가비지 컬렉션의 대상이 되느냐? 그렇지 않다.
  - 예제 코드 3
    ```java
    String exData = new String("exData");
    String exData2 = null;
    
    exData2 = exData;

    exData = null;
    ```
    - exData를 마지막에 null로 주었기 때문에 가비지 컬렉션이 되상이 될 것 같지만 그렇지 않다.
    - exData 변수가 가지고 있는 주소 값을 exData2 변수에 복사하여 여전히 메모리 값을 참조하여 사용하기 있기 떄문이다.
    - 그렇기 때문에 exData = null 코드 라인은 작용하지 않아 exData는 가비지 컬렉션의 대상이 되지 않는다.
  - 예제 코드 4
    ```java
    String exData = new String("exData");
    String exData2 = null;
    
    exData2 = exData;

    exData = null;
    exData2 = null;
    ```
    - JVM이 메모리 해제하도록 하기 위해서는 exData, exData2 모두 명시적으로 null을 할당해주어야 한다. 이렇게 하면 메모리를 참조하는 변수가 없어져 더 이상 사용되지 않는
      메모리로 판단하여 가비지 컬렉션을 시작한다.

## 스레드 풀링
  - 관련 용어 정리
    - 프로그램: 하드디스크에 저장된 바이너리 코드
    - 프로세스: 실행을 위해 메모리에 적재된 프로그램, CPU에 의해 실행중이거나 주기억장치에 적재된 프로그램과 데이터
    - 스레드: 프로세스 내부에서 실행되는 흐름의 단위
    - 멀티 스레드: 일반적으로 프로그램은 하나의 스레드를 갖지만, 프로그램 환경에 따라 둘 이상의 스레드를 동시에 실행하는 방식
    - 멀티 스레딩: 멀티스레드 방식을 사용하여 CPU가 하나 이상의 작업을 동시에 처리하는 것
  - ABOUT thread pool
    - 애플리케이션은 동시에 여러 작업을 처리하기 위해 여러 개의 스레드를 생성한다.
    - 최적의 작업 효율을 위해 스레드 풀이라는 것을 만들어놓고 활용한다.
    - 작업 처리를 위해 무한정 스레드를 생성하지 않고, 프로그램을 시작할 때 고정 갯수의 스레드를 미리 생성해놓고 풀 형태로 제공하는 것이다.
    - 작업을 처리할 때 가용한 스레드를 이용한다.
    - 스레드 풀을 활용하면, 스레드가 이미 생성되어있어 별도 생성 절차를 거치지 않아 작업을 빨리 할당하고, 컴퓨팅 자원의 양을 고려하여 스레드 생성 갯수를 제한할 수 있다.
  - WAS(Web Application Service)에서의 스레드 풀링
    - 동작 순서
      1. 클라이언트 요청
      2. 접수 스레드 - 접수 창구에서 접수
      3. 대기 큐 - 접수 후 대기
      4. 요청 처리 스레드 - 불러서 처리
    - 동작 원리
      - WAS가 시작될 때 일정 갯수만큼의 스레드 풀이 만들어진다.
      - 클라이언트 요청이 많아지면 동시 요청 처리를 위해 더 많은 스레드를 생성하게 되는데, 서버 자원의 고려 없이 무한 생산하기 때문에 어느 시점에서 서버 자원이 고갈되면 요청 처리를
        위한 스레드가 부족해질 수 있다. 이렇게 되면 웹 서버에서 일부 작업이 처리되지 못해 대기 큐에서 계속 대기하게 되고, 클라이언트로의 응답 시간이 지연된다.
    - 효율적 작업 처리를 위한 스레드 풀 설정 방법
      1. WAS 설정 파일 변경
         : <Executor name="tomcatThreadPool" namePrerix="catalina-exec-" maxThreads="200" minSpareThreads="5" />
      2. Business Logic 설계를 통한 스레드 풀 설계
         : 스레드 풀의 갯수, 대기 큐 생성, 스레드 풀 생성, 스레드 풀 종료, 서비스 로직 실행 등의 코드 구현을 통해 Java는 ThreadPoolExecutor를 구현하고 있다.
         ```java
         import java.util.concurrent.LinkedBlockingQueue;
         import java.util.concurrent.ThreadPoolExecutor;
         import java.util.concurrent.TimeUnit;

         public class ThreadPool {

           public static void main(String[] args) throws Exception {

             // THREAE POOL 갯수 설정
             int CORE_POOL_SIZE = 5;
             int MAX_POOL_SIZE = 10;
             int KEEP_ALIVE_TIME = 10;

             // 대기 queue 생성
             LinkedBlockQueue<Runnable> queue = new LinkedBlockQueue<>(10);

             // 스레드 풀 생성
             ThreadPoolExecutor threadPool = new ThreadPoolExecutor(CORE_POOL_SIZE, MAX_POOL_SIZE, KEEP_ALIVE_TIME,
         TimeUnit.SECONDS, queue);

             // 스레드 풀에 처리할 작업 할당
             for (int i=0; i<10; i++) {
               threadPool.execute(new Task());
             }
             threadPool.shutdown();
           }
  
           // 스레드가 처리할 작업 정의(클래스 내 내부정의)
           static class Task implements Runnable {
             @Override
             public void run() {
               System.out.println(Thread.currentThread().getName() + "번 스레드가 작업 처리를 완료하였습니다.");
             }
           }
         }
         ```
         - 다수의 작업 요청을 처리하는 애플리케이션을 개발하기 위해서는 대기 큐 사이즈, 최소/최대 작업 스레드 갯수를 적절하게 설정해주는 것이 중요하다.
         - 대기 큐의 크기는 증가시키고 최소 작업 스레드 갯수를 작게하면, 대기 큐가 가득 차지 않기 때문에 처리하기 위한 스레드 수가 증가하지 않는다. 이 경우 클라이언트 요청이
           대기 큐에서 오랜 시간 머물게 되어 서버의 응답 시간이 길어지는 문제로 서비스의 질적 하락을 초래할 수 있다.
         - 스레드 풀을 사용할 때는 예상되는 발생 부하량을 정밀히 예측학고, 예측값에 맞는 적절한 스레드 풀 설정 값을 찾아 셋팅해주는 것이 가장 중요하다.

      #### Thread Pool 관리를 위한 옵션
        - CORE_POOL_SIZE: 스레드 풀이 유지할 최소 스레드 갯수, 작업이 없어도 설정된 최소 스레드가 작업 수행을 위해 대기
        - MAX_POOL_SIZE: 최대 스레드 풀의 갯수, 작업량이 많을 때 가동할 수 있는 최대 스레드 수
        - KEEP_ALIVE_TIME: 초기 최소 스레드 갯수만큼 생성되고, 작업이 많아지면 최대 갯수까지 증가했다가, 다시 처리할 작업이 줄면 스레드 수가 감소한다. 이 때 설정된
          시간만큼 현황을 지켜보다가 작업 수 감소가 확인되면 스레드 수를 줄인다. 즉, 스레드 수 증감을 판단하기까지 일정 시간동안 지켜보는 기준 시간이다.

## DBMS에서의 스레드 풀링
  - 다수 서비스 또는 클라이언트가 DB에 접근하여 데이터를 다루고자 할 때, 그 수가 많아질 경우 접속 처리를 위한 스레드가 증가하여 오버헤드로 인한 성능저하가 발생하기도 한다.
  - 이 때 DBMS도 스레드 풀을 통한 처리로, 스레드 오버헤드로 인한 성능 저하를 어느정도 방지할 수 있다.
  - 다수 애플리케이션이 DBMS 접속을 위해 순서표를 발급받아 대기 큐에서 대기하는데, DB 카테고리에서 사용되는 스레드 풀을 커넥션 풀(Connection Pool)이라고 한다.
  --> Thread Pool은 콘서트(서버)장 입장을 통제하기 위해 존재하는 컨트롤 타워(스레드 풀) 같은 개념으로 이해하면 좋을 것 같다.


