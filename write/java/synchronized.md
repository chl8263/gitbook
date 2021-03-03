# Synchronized 정리

## Synchronized 란?

Multi-thread로 인해 자원을 동기화 해야하는 경우가 발생한다. Multi-thread 환경에서 Thread간 동기화 문제는 반드시 해결해야하는 아주 중요한 문제. 

특정 자원의 Thread-safe를 위해 Java 에서 **Synchronized** 키워드를 사용한다. 이 키워드는 여러개의 Thread가 하나의 자원을 사용하고자 할 때 현재 데이터를 사용하고 있는 해당 쓰레드를 제외한 나머지 Thread는 그 자원에 접근하지 못하도록 막는것.

Thread Multi-thread 환경에서 프로그램적으로 좋은 성능을 기대하지만 **Synchronized** 키워드를 남발하거나 적시적소에 사용하지 못한다면 성능 저하를 불러온다. 왜냐면 block, unblock 하는 것도 전부 일이기 때문.

## Usage

Synchronized의 사용법은 2가지로 나뉜다.

### 1. 함수에 사용

```java
class Test{
    public synchronized void method(){
        // code
    }
}
```

대표적으로 위와 같이 함수에 사용한다.

```java
public class Main {
    public static void main(String[] args) {

        var test = new Test();

        new Thread(() -> {
            for(int i = 0; i < 10; i ++){
                try {
                    Thread.sleep(100);
                    test.addCount();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();

        new Thread(() -> {
            for(int i = 0; i < 10; i ++){
                try {
                    Thread.sleep(100);
                    test.addCount();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}

class Test{

    private Integer count = 0;

    public synchronized void addCount(){
        count++;
        System.out.println(Thread.currentThread() + " - " + count);
    }

    public synchronized void method2(){
        // code
    }
}
```

위의 코드에서 서로다른 Thread 두개가 한개의 Test 인스턴스 **addCount\(\)** 메서드를 호출한다.  **synchronized** 키워드가 없으면 ****Test 객체 안의 counter 변수 출력이 순서대로 증가하지않고 바뀌는 경우가 생긴다.

* synchronized 한경

```java
Thread[Thread-0,5,main] - 1
Thread[Thread-1,5,main] - 2
Thread[Thread-1,5,main] - 3
Thread[Thread-0,5,main] - 4
Thread[Thread-0,5,main] - 5
Thread[Thread-1,5,main] - 6
```

* synchronized 안한경

```java
Thread[Thread-0,5,main] - 1
Thread[Thread-1,5,main] - 1
Thread[Thread-0,5,main] - 2
Thread[Thread-1,5,main] - 2
Thread[Thread-1,5,main] - 3
Thread[Thread-0,5,main] - 3
```

**하지만 함수에 synchronized 키워드를 사용할 경우 함수만 Thread 에 block 되는것이 아닌 그 함수가 포함된 객체 전부 lock 걸리게 된다.** 

```java
class Test{

    private Integer count = 0;

    public synchronized void addCount(){
        count++;
        System.out.println(Thread.currentThread() + " - " + count);
    }

    public void method2(){
        // code
    }
}
```

만약 addCount\(\) 함수 처럼 공유자원을 사용하는 함수가 아닌 Method2함수를 사용해야하는 일이 발생한다면? 이미 addCount\(\)의 synchronized키워드 때문에 객체 전체가 lock이 걸려 원하는 성능을 기대할 수 없다.

### 2. **synchronized block 사용**

```java
class Test{
    private Object object;
    
    public Test(Object object){
        this.object = object;
    }

    public void method2(){
        synchronized (object){
            //code
        }
    }
}
```

synchronized 블록은 인자로 Object 를 받는다. \(primitive 타입 불가\)

위에말한 함수에 synchronized를 개선하자면 다음과 같다.

```java
class Test{

    private Integer count = 0;

    public void addCount(){
        synchronized (this){
            count++;
            System.out.println(Thread.currentThread() + " - " + count);
        }
    }
}
```

synchronized block 안의 this는 해당 객체안의 모든 synchronized block에 lock을 건다는 뜻이다. 즉, 만약 모든 함수안에 synchronized block이 있다면 위에 처럼 함수에 synchronized 키워드를 걸어 객체 전부 lock을 건것과 동일한 효과를 준다.

```java
class Test{

    private Integer count = 0;
    private Integer count2 = 0;

    public void addCount(){
        synchronized (this){
            count++;
            System.out.println(Thread.currentThread() + " - " + count);
        }
    }

    public void addCount2(){
        synchronized (this){
            count2++;
            System.out.println(Thread.currentThread() + " - " + count2);
        }
    }
}
```

위 코드와 같이 addCount\(\) 함수와 addCount2\(\) 함수가 서로 다른 자원에 대해 동기가 필요하지만 위처럼 synchronized block에 this 인자값을 주면 쓸데없는 짓이라는 것이다.

```java
class Test{

    private Integer count = 0;
    private Integer count2 = 0;

    public void addCount(){
        synchronized (count){
            count++;
            System.out.println(Thread.currentThread() + " - " + count);
        }
    }

    public void addCount2(){
        synchronized (count2){
            count2++;
            System.out.println(Thread.currentThread() + " - " + count2);
        }
    }
}
```

위처럼 synchronized block에 각기 다른 Object를 동기화하여 훨씬더 효율적인 코드가 된다.

### Static 의 경우

static method 에 **synchronized** 키워드가 붙는다면 객체의 인스턴스를 기준으로 lock을 잡는것이 아니라 **class 자체에 lock을 걸게 된다.**

```java
  public static synchronized void add(int value){
      count += value;
  }
```

### Static 과 non-static 을 혼용하여 사용할 경우

그럼 static 은 class를 기준으로 lock 을걸고 non-static 은 객체를 기준으로 lock을 걸기때문에 서로 lock을 거는것이 따로 놀게된다.

```java
class Test{

    private static Integer count = 0;

    public static synchronized void addCount(){
        count++;
        System.out.println(Thread.currentThread() + " - " + count);
    }

    public synchronized void addCount2(){
        count++;
        System.out.println(Thread.currentThread() + " - " + count);
    }
}
```

위와같이 static과 non-static 함수가 같은 count 변수를 공유한다면 동기화가 제대로 이루어지지 않아서 간헐적으로 동기화 이슈가 발생한다. 찾기도 힘들고 절대 이렇게 쓰지말것.

concurreny 는 왜 나왔고 Basic synchronized 는 뭐가 문제가 있나? volatile Atomic

