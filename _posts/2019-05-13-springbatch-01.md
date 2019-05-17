---
title: "[Spring Batch]Scaling and Parallel Processing"
date: 2019-04-19 15:05:28 -0400
categories: jekyll update
---

https://docs.spring.io/spring-batch/3.0.x/reference/html/scalability.html
* Spring Batch 공식 문서를 참고하여 병렬 처리에 대해 학습 및 번역한 내용을 정리한 포스트입니다.
* 코드는 Spring boot를 사용하여 작성했습니다.

병렬처리에는 2가지 모드가 있다.
- single process
- multi-threaded; and multi-process

이는 다음 카테고리들로 나뉜다.
- Multi-threaded Step (single process)
- Parallel Steps (single process)
- Remote Chunking of Step (multi process)
- Partitioning a Step (single or multi process)

1. Multi-threaded Step

병렬 처리를 시작하는 가장 간단한 벙법은 `TaskExecutor`를 Step configuration에 추가하는 것이다.
예를 들면 tasklet의 attribute로 다음과 같이 추가한다.
```java
@Bean
public Step step1() {
    return stepBuilderFactory.get("step1")
            .<SalesRecord, SalesRecord>chunk(100)
            .reader(reader(null))
            .writer(testJpaItemWriter())
            .taskExecutor(taskExecutor) // TaskExecutor 추가.
            .throttleLimit(10) // throttle limit 값 설정
            .build();
}
```
위 예제에서는 taskExecutor는 TaskExecutor 인터페이스를 구현한 다른 빈을 참조한 것이다.
TaskExecutor는 표준 Spring 인터페이스다. 따라서 자세한 사항은 Spring User Guide를 참조하라.

가장 간단한 멀티 스레드 `TaskExecutor`는 `SimpleAsyncTaskExecutor`이다.

위의 configuration의 결과로 Step은 분리된 스레드 실행(separate thread of execution)에서 아이템을 `청크 단위`로 읽고(reading), 처리하고(processing), 쓴다(reading).
이는 아이템들이 처리되는 순서는 고정되지 않았다는 것을 의미한다. 그리고 하나의 청크는 single-threaded case에 비교해서 연속적이지 않은 아이템을 포함할 수 있다.

TaskExecutor에 설정된 다른 제한과 함께 tasklet configuration에는 디폴트 값이 4인 throttle limit이 있다.
스레드 풀이 완전히 사용하도록 하기 위해서 throttle limit을 증가시킬 수 있다.

DataSource와 같이 Step에 사용된 모든 풀 리소스에 의해 동시성에 제한이 있을 수 있다는 점에 주의하라.
즉, step에서 바라는 동시 스레드의 수만큼 크게 가용한 풀이 있어야 한다.

흔한 Batch 유즈 케이스에서 multi-threaded Step들을 사용하는데에 실용적인 제한이 있다.
Step에서 사용되는 많은 reader와 writer는 stateful하다. 그리고 만약 이 상태(state)가 스레드에 의해 분리되지 않는다면 그런 컴포넌트들은 multi-threaded Step에서는 사용할 수 없다.
특히 대부분의 스프링 배치의 대부분의 reader와 writer는 multi-threaded 용도로 설계되지 않았다.

그러나 stateless하고 thread-safe한 reader와 writer로 작업하는 것이 가능하다.
데이터베이스 입력 테이블에서 처리된 아이템들을 추적하기 위해서 process indicator를 사용하는 것을 보여주는 [스프링 배치 예제](https://docs.spring.io/spring-batch/3.0.x/reference/html/readersAndWriters.html#process-indicator)가 있다.

    [6.12 Preventing State Persistence](https://docs.spring.io/spring-batch/3.0.x/reference/html/readersAndWriters.html#process-indicator)
    기본적으로, 모든 ItemReader와 ItemWriter 구현체는 커밋 전에 `ExecutionContext`에 그들의 현재 상태를 저장한다.
    그러나 이는 바람직한 행동이 아닐 수 있다. 예를 들면, 많은 개발자들은 process indicator를 사용하여 database reader를 'rerunnable'하게 만든다. 입력 데이터에 이미 처리됐는지 안됐는지를 가리키기 위해서 추가 컬럼이 붙는다. 특정 레코드를 읽는 중이거나 쓰여지고 있을 때 processed flag는 fals에서 true로 바뀐다. SQL 문장에는 where 절(where PROCESSED_IND = false)이 추가 된다. 이렇게 함으로써 처리되지 않은 레코드만이 재시작할 경우 반환될 것이다. 이런 시나리오에서는 어떠한 상태(현재 row 번호...)도 저장하지 않는 것이 낫다. 왜냐하면 재시작할 떄는 그 상태는 관련없는 것이기 때문이다. 이런 이유로 모든 reader와 writer는 `saveState` property를 포함한다.

스프링 배치는 `ItemWriter`와 `ItemReader`에 대한 구현제가 몇 개 있다. 보통 동시성 환경에서 문제를 피하기 위해 해야할 것들이 Javadocs에 나와있다.
Javadocs에 어떠한 정보도 없다면 상태가 존재 하는지 보기 위해서 구현체들을 확인할 수 있다. 
reader가 thread-safe하지 않다면, 자신의 동기화 delegator에서 사용하는 것이 효과적일 것이다.
당신은 read()에 대한 호출을 동기화 할 수 있고 processing과 writing이 청크에서 가장 비용이 많이 드는 부분이라면 
당신의 step은 여전히 single threaded configuration에서 보다 더 빠르게 완료될 수 있다.


2. Parallel Steps

병렬화할 필요가 있는 애플리케이션 로직이 별개의 책임고 나뉠 수 있어서 개별 step들에 할당될 수 있다면 single process에서 병렬화될 수 있다.
병렬 Step execution은 설정해서 사용하기 쉽다.
다음 예제에서는 2개의 flow가 있는데 각 flow는 병렬적으로 실행된다.
한 flow는 step1을 실행한 후 step2를 실행하고
다른 flow는 step3를 실행한다.

<job id="job1">
    <split id="split1" task-executor="taskExecutor" next="step4">
        <flow>
            <step id="step1" parent="s1" next="step2"/>
            <step id="step2" parent="s2"/>
        </flow>
        <flow>
            <step id="step3" parent="s3"/>
        </flow>
    </split>
    <step id="step4" parent="s4"/>
</job>

<beans:bean id="taskExecutor" class="org.spr...SimpleAsyncTaskExecutor"/>


3. Remote Chunking of Step

Remote Chunking에서는 Step processing이 여러 프로세스들로 분할되어 일부 미들웨어를 통해 서로 통신한다.
다음은 실제 패턴에 대한 그림이다.

<<<그림>>>

Master 컴포넌트는 단일 프로세스이고, Slave들은 여러개의 원격 프로세스들이다. 
이 패턴은 Master가 병목현상이 아닌 경우에 잘 작동하므로, 아이템을 읽는 것보다 처리하는 것이 더 비싸야 한다.(실무에는 이런 경우가 많다.)

Master는 스프링 배치 step의 구현일 뿐이며, 메시지로서 미들웨어에 아이템들의 청크를 보내는 방법을 알고 있는 일반 버전으로 대체된 ItemWriter를 갖는다.
Slave들은 어떤 미들웨어가 사용되든 간에 표준 listener 이다. (예를 들면, JMS와 함께 MessageListeners일 것이다.)
그리고 아이템들의 청크들을 처리하기 위해서 `ChunkProcessor` 인터페이스를 통해 표준 `ItemWriter` 혼자 또는 `ItemProcessor`를 함께 사용한다. 

이 패턴의 장점 중 하나는 reader, processor, writer 컴포넌트들이 off-the-shelf이라는 것이다.(step의 로컬 실행에 사용되는 것과 동일함)
아이템들은 동적으로 나뉘어져서 작업들은 미들웨어를 통해서 공유된다. listener들이 열렬한 소비자들이라면 로드밸런싱은 자동이다.

The middleware has to be durable, with guaranteed delivery and single consumer for each message. 
JMS is the obvious candidate, but other options exist in the grid computing and shared memory product space (e.g. Java Spaces).


4. Partitioning a Step

multi-threaded step 의 throttle-limit 속성과 유사한 grid-size 속성은 task executor가 단일 step으로 부터의 요청에 포화되지 않도록 방지한다.

    4.1 PartitionHandler
    4.2 Partitioner
    4.3 Binding Input Data to Steps



ThreadPoolTaskExecutor

    If the number of threads is less than the corePoolSize, create a new Thread to run a new task.
    If the number of threads is equal (or greater than) the corePoolSize, put the task into the queue.
    If the queue is full, and the number of threads is less than the maxPoolSize, create a new thread to run tasks in.
    If the queue is full, and the number of threads is greater than or equal to maxPoolSize, reject the task.


적절한 스레드 수를 어떻게 결정하는가?
    CPU-bound 인지 IO-bound 인지를 고려해야 한다.
    CPU-bound 라면 가용한 프로세서의 개수보다 살짝 많은 것이 좋다.
    If it's IO-bound (a single thread is spending most of its time waiting for external esources such as database connections, file systems, or other external sources of data) then you can assign (many) more threads than the number of available processors - of course how many depends also on how well the external resource scales though - local file systems, not that much probably.
    [https://stackoverflow.com/questions/42015714/how-to-get-ideal-number-of-threads-in-parallel-programs-in-java]
    