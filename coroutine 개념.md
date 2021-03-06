## 코틀린 코루틴(Coroutine)

#### 정의

**코루틴**은 비동기적으로 실행되는 코드를 간소화하기 위해 Android에서 사용할 수 있는 **동시 실행 설계 패턴**입니다.

Android에서 코루틴은 기본 스레드를 차단하여 앱이 응답하지 않게 만들 수도 있는 장기 실행 작업을 관리하는데 도움이 됩니다.



#### 기능

+ 경량 : 코루틴을 실행 중인 스레드를 차단하지 않는 정지를 지원하므로 단일 스레드에서 많은 코루틴을 실행할 수 있습니다. 정지는 많은 동시 작업을 지원하면서도 차단보다 메모리를 절약합니다.
+ 메모리 누수 감소 : 구조화된 동시 실행을 사용하여 범위 내에서 작업을 실행합니다.
+ 기본적으로 제공되는 취소 지원 : 실행 중인 코루틴 계층 구조를 통해 자동으로 취소가 전달됩니다.
+ JetPack 통합 : 많은 JetPack 라이브러리에 코루틴을 완전히 지원하는 확장 프로그램이 포함되어 있습니다. 일부 라이브러리는 구조화된 동시 실행에 사용할 수 있는 자체 코루틴 범위도 제공합니다.



#### 루틴과 서브루틴

루틴은 컴퓨터 프로그램에서 하나의 정리된 일 입니다. 프로그램은 보통 크고 작은 여러가지 루틴을 조합시킴으로써 성립합니다. 루틴은 메인 루틴(main routine) 과 서브루틴(subroutine)으로 나뉘게 됩니다. 메인 루틴은 프로그램 전체의 개괄적인 동작 절차를 표시하도록 만들어 집니다. 서브루틴은 반복되는 특정 기능을 모아 별도로 묶어 놓아 이름을 붙여 놓은 것이라고 할 수 있습니다. 



#### 서브루틴과 코루틴

코루틴도 루틴의 일정도이라고 할 수 있습니다. 

다만 3가지 정도의 차이가 있는데 

1. 코루틴에서는 메인-서브의 개념이 없습니다. 모든 루틴들이 서로를 호출할 수 있습니다.
2. 서브루틴의 경우에는 메인루틴에서 특정 서브루틴의 공간으로 이동한 후에 return 의해 호출자로 돌아와 다시 프로세스를 진행하는데 반해, 코루틴의 경우에는 루틴을 진행하는 중간에 멈추어서 특정 위치로 돌아갔다가 다시 원래 위치로 돌아와 나머지 루틴을 수행할 수 있습니다.
3. 서브루틴은 진입점과 반환점이 단 하나밖에 없어 메인루틴에 종속적이지만, 코루틴은 진입지점이 여러개이기 때문에 메인루틴에 종속적이지 않아 대등하게 데이터를 주고 받을 수 있다는 특징이 있습니다.



#### 선점형과 비선점형

하나의 Task가 Scheduler로 부터 CPU 사용권을 할당 받았을 때, Scheduler가 강제로 CPU 사용권을 뺐을 수 있으면 비선점형 멀티태스킹이며 뺏을 수 있으면 선점형 멀티태스킹 입니다.



코루틴은 **비선점 멀티태스킹** 이고, 쓰레드는 선점형 멀티태스킹 입니다.

즉 코루틴은 병행성을 제공하지만 병렬성은 제공하지 않는다는 의미입니다 .



#### 병행성과 병렬성 

**병행성(Concurrency)**

1. 동시에 실행되는 것처럼 보이는 것
2. Logical Level 사용
3. Single Core 사용
4. 물리적으로 병렬이 아닌 순차적으로 동작
5. 실제로는 Time-sharing으로 CPU를 나눠 사용함으로써 사용자가 Concurrency 체험

**병렬성(Parallelism)**

1. 실제로 동시에 작업이 처리가 되는 것
2. Physical(Machine) Level에 속함
3. 오직 Multi Core에서만 가능



#### 코틀린 코루틴

``` kotlin
GlobalScope.launch{
  delay(1000L)
  println("world!")
}
println("Hello!")
Thread.sleep(2000L)
```



위의 예제에서 GlobalScope는 전체 어플리케이션의 라이프타임으로 scope로, launch는 해당 코드 블럭을 현재스레드를 blocking 하지 않고 코루틴으로 실행합니다.

위의 예제에서 delay 함수는 blocking 할 수 없는 suspending 함수이기 때문에 GlobalScope.launch 대신 Thread를 사용하면, 에러가 발생합니다.

그렇다면 blocking을 하기 위해서는 아래와 같은 조치를 취해야 합니다.



```Ko
GlobalScope.launch {
		delay(1000L)
		println("World!")
}
println("Hello")
runBlocking {
		delay(2000L)
}
```

coroutineScope는 runBlocking과는 다르게, 모든 자식들이 완료될 때 까지 현재 스레드를 Block 시키지 않습니다 . 

함수로 만들기 위해서는 suspend 키워드를 붙여서 만들어야 합니다.



#### 코루틴 취소 

```Ko
val job = launch {
		repeat(1_000) { i ->
				println("job: I'm sleeping $i ...")
				delay(500L)
		}
}
delay(1300L)
println("main: I'm tired of waiting!")
job.cancel()
job.join() 
println("main: Now I can quit.")
```

여기서 launch 블럭은 job Type을 리턴합니다.

job은 실행중인 코루틴을 취소할 수 있습니다.



#### withContext

스레드 간의 점프에 사용된다. withContext를 사용함으로써 사용하는 쓰레드 변경이 가능합니다.



#### Dispatchers

코루틴 실행에 사용하는 스레드를 결정한다. 모든 코루틴 빌더(ex, launch, async) 는 디스패처를 지정할 수 있습니다.

다음은 대표적인 Dispathers

1. Default: 오래 걸리는 작업을 할 때. 공유된 백그라운드 스레드 풀 사용합니다.
2. IO: 파일을 쓰거나 API 콜 같은 상황에서 사용합니다.
3. Main: 메인스레드 작업에 사용합니다.



#### 부모 코루틴의 책임

1. 코루틴이 다른 코루틴의 coroutineScope 내에서 실행되면 CoroutineScope.coroutineContext를 통해 컨텍스트를 상속 받고부모 코루틴job 의 자식이 됩니다.
2. 부모 코루틴이 취소되면, 자식 코루틴들은 재귀적으로 취소됩니다.
3. GlobalScope는 부모 코루틴의 영향을 받지 않습니다.
4. 부모 코루틴은 자식 코루틴이 완료될때까지 대기한다. join()이 필요가 없습니다.



#### 코루틴 예제

```Ko
GlobalScope.launch {
    launch {
        print("At Here!")
    }
    val value: Int = async {
        var total = 0
        for (i in 1..10) total += i
        total
    }.await()
    print("$value")
}
```

```Ko
GlobalScope.launch {
    val x = doSomething()
    print("done something, $x")
}
private suspend fun doSomething():Int {
    val value: Int = GlobalScope.async(Dispatchers.IO) {
        var total = 0
        for (i in 1..10) total += i
        print("do something in a suspend method: $total")
        total
    }.await()
    return value
}
```

```Ko
print("Start...")
GlobalScope.launch(Dispatchers.Main) {
    val job1 = async(Dispatchers.IO) {
        var total = 0
        for (i in 1..10) {
            total += i
            delay(100)
        }
        print("job1")
        total
    }
    val job2 = async(Dispatchers.Main) {
        var total = 0
        for (i in 1..10) {
            delay(100)
            total += i
        }
        print("job2")
        total
    }
    val result1 = job1.await()
    val result2 = job2.await()
    print("results are $result1 and $result2")
}
print("End.")
```

