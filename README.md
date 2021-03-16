# Channels

Deferred 값은 Coroutine 간 단일 값을 전송하는 방법을 제공한다면, Channel은 stream한 value를 전송하는 방법을 제공한다.

# Channel Basics

Channel은 개념적으로 BlockingQueue와 매우 유사하다. 차이점으로는 put 대신 suspend한 send가 있고, take 대신 suspend한 receive 가 있다.

# Closing and iteration over channels

Queue와 다르게 채널은 close를 함으로써 더 이상 element가 오지 않음을 표현이 가능하다. receiver 에서는 for-loop 을 사용하여 편리하게 element를 받을 수 있다.

개념적으로 close는 channel에 close 이벤트를 보내는 것과 같다. close 이벤트가 수신되면 즉시 반복이 중지되므로 close 되기 전에 이전에 보낸 모든 element가 수신된다는 것을 보장합니다.

```kotlin
fun main() = runBlocking {
    val channel = Channel<Int>()

    launch {
        for (x in 1..5) channel.send(x * x)
        channel.close() // 전송 중지
    }

    // 여기서 수신된 값은 for 루프를 사용하여 출력됩니다.(채널이 닫힐 때까지)
    for (y in channel) println(y)

    println("Done!")
}
```

```
출력 결과
1
4
9
16
25
Done!
```

이 코드를 작성해보고 실행해 보면서 재미있는 점을 찾았다. 
아래와 같이 receive 하는 곳에  delay가 발생하게 된다면 receive의 처리가 끝날때까지 기다려 같이 delay가 되는 증상을 확인했다.
RxJava에서는 emit 되는 속도가 데이터를 처리하는 속도보다 빠르면 배압 이슈가 발생하는데 RxJava의 Flowable과 동일하게 동작하는것 같다.

```kotlin
for (y in channel) {
    delay(1000)
    println(y)
}
```

# Building channel producers

coroutine에서 sequence한 element를 생성하는 패턴은 흔하다. 이것은 concurrent 코드에서 흔히 발견할 수 있는 producer-consumer 패턴의 한 부분이다.

channel에서는 생산자 측에서 바로 produce 할 수 있도록 하는 편리한 coroutine 빌더와 producer 측에서 for-loop을 대체하여 사용할 수 있는 consumeEach 라는 extention function이 있다.

```kotlin
import kotlinx.coroutines.*
import kotlinx.coroutines.channels.*

fun CoroutineScope.produceSquares(): ReceiveChannel<Int> = produce {
    for (x in 1..5) send(x * x)
}

fun main() = runBlocking {
    val squares = produceSquares()
    squares.consumeEach { println(it) }
    println("Done!")
}
```

```
출력 결과
1
4
9
16
25
Done!
```

# Pipelines

To be Continue

참고링크
[https://kotlinlang.org/docs/reference/coroutines/channels.html](https://kotlinlang.org/docs/reference/coroutines/channels.html)
