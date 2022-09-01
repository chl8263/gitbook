# Thread pool

## Thread pool 이란?

Thread 가 생성될 때 컴퓨터 내부적으로 운영체제가 메모리 공간을 확보하고 그 메모리를 Thread에게 할당해준다. 이때 Thread 생성/수거 비용이 무시할 수 없을만큼 상당하다.

이때문에 Thread 를 미리 생성하여 놓고 프로그램에서 Thread 를 생성/수거 하는 비용의 부담을 지게하지 않게 한다.

Thread pool 은 작업큐에 들어온 Task 일감들을 미리 생성해 놓은 Thread 들에게 위임하고 Thread 가 일을 마치면 다시 Application 에게 결과값을 반환한다.

![](<../../.gitbook/assets/image (1) (1) (2).png>)

Java 에서 ExecuterService 인터페이스, Executor 클래스를 제공하고 있고 Executors 의 다양한 정적 메소드를 이용하여 Thread pool 을 이용.

### Thread pool 장점&#x20;

* 매번 Thread 를 생성/수거 하는 부담을 줄일 수 있다.
* 서비스 측면으로 바라볼때 대다수 사용자의 요청을 수용하며 빠르게 처리할 수 있다.

### Thread pool 단점&#x20;

* 너무 많이 만들어 놨다가 일하지 않고 노는 Thread 가 발생할 수 있다. 예를들어 200개의 Thread 를 만들어 놨는데 50개의 Thread 만 일하는 상황.
* 위와 비슷하게 다른경우로 노는 Thread 가 생길 수 있다. A 작업을 하는 Thread 가 작업이 많아서 아직 작업이 끊나지 않을경우 B, C 가 노는 경우가 생길 수있다. -> Java 에서는 이를 방지하기 위해 **ForkJoinPool** 을 지원한다.

### Thread pool 의 종류&#x20;

#### **1. ThreadPoolExecutor**

* Executors.newFixedThreadPool(int nThread)

&#x20;고정된 Thread 갯수를 지정하여 사용하기 위해 사용.&#x20;

* Executors.newCachedThreadPool()

유동적으로 Thread 갯수를 증가시키고자할 때 사용 Thread 가 60초 동안 일을 하지않으면 Thread 를 종료시키고 pool에서 제거한다.

#### **2. ScheduledThreadPoolExecutor**

Thread pool 을 생성하고 그 Thread 들을 스케줄링하여 반복적으로 돌릴 수 있게 하거나 딜레이를 주어서 다양한 작업을 할 수 있다.

### Thread pool 의 종료

Thread pool은 메인 Thread 가 아니므로 프로그램이 종료되어도 Thread 가 남아있다. 그래서 프로그램이 종료될때 Thread pool 을 종료시켜주어야한다.

* &#x20;**excutorService.shutdown() :** 작업큐에 남아있는 작업까지 모두 마무리 후 종료
* &#x20;**excutorService.shoutdownNow() :** 작업큐 잔량과 상관없이 강제 종료
* **executorService.awitTermination()** : 모든 작업 처리를 timeout 시간에 처리하면 true리턴, 처리하지 못하면 작업스레드들을 interrupt() 시키고 flase리턴

