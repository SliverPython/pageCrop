# java中DelayQueue的使用 - flydean - 博客园
java 中 DelayQueue 的使用

今天给大家介绍一下 DelayQueue,DelayQueue 是 BlockingQueue 的一种，所以它是线程安全的，DelayQueue 的特点就是插入 Queue 中的数据可以按照自定义的 delay 时间进行排序。只有 delay 时间小于 0 的元素才能够被取出。

先看一下 DelayQueue 的定义：

```null
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
    implements BlockingQueue<E>
```

从定义可以看到，DelayQueue 中存入的对象都必须是 Delayed 的子类。

Delayed 继承自 Comparable，并且需要实现一个 getDelay 的方法。

为什么这样设计呢？

因为 DelayQueue 的底层存储是一个 PriorityQueue，在之前的文章中我们讲过了，PriorityQueue 是一个可排序的 Queue，其中的元素必须实现 Comparable 方法。而 getDelay 方法则用来判断排序后的元素是否可以从 Queue 中取出。

DelayQueue 一般用于生产者消费者模式，我们下面举一个具体的例子。

首先要使用 DelayQueue，必须自定义一个 Delayed 对象：

```null
@Data
public class DelayedUser implements Delayed {
    private String name;
    private long avaibleTime;

    public DelayedUser(String name, long delayTime){
        this.name=name;
        
        this.avaibleTime=delayTime + System.currentTimeMillis();

    }

    @Override
    public long getDelay(TimeUnit unit) {
        
        long diffTime= avaibleTime- System.currentTimeMillis();
        return unit.convert(diffTime,TimeUnit.MILLISECONDS);
    }

    @Override
    public int compareTo(Delayed o) {
        
        return (int)(this.avaibleTime - ((DelayedUser) o).getAvaibleTime());
    }
}
```

上面的对象中，我们需要实现 getDelay 和 compareTo 方法。

接下来我们创建一个生产者：

```null
@Slf4j
@Data
@AllArgsConstructor
class DelayedQueueProducer implements Runnable {
    private DelayQueue<DelayedUser> delayQueue;

    private Integer messageCount;

    private long delayedTime;

    @Override
    public void run() {
        for (int i = 0; i < messageCount; i++) {
            try {
                DelayedUser delayedUser = new DelayedUser(
                        new Random().nextInt(1000)+"", delayedTime);
                log.info("put delayedUser {}",delayedUser);
                delayQueue.put(delayedUser);
                Thread.sleep(500);
            } catch (InterruptedException e) {
                log.error(e.getMessage(),e);
            }
        }
    }
}
```

在生产者中，我们每隔 0.5 秒创建一个新的 DelayedUser 对象，并入 Queue。

再创建一个消费者：

```null
@Slf4j
@Data
@AllArgsConstructor
public class DelayedQueueConsumer implements Runnable {

    private DelayQueue<DelayedUser> delayQueue;

    private int messageCount;

    @Override
    public void run() {
        for (int i = 0; i < messageCount; i++) {
            try {
                DelayedUser element = delayQueue.take();
                log.info("take {}",element );
            } catch (InterruptedException e) {
                log.error(e.getMessage(),e);
            }
        }
    }
}
```

在消费者中，我们循环从 queue 中获取对象。

最后看一个调用的例子：

```null
    @Test
    public void useDelayedQueue() throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(2);

        DelayQueue<DelayedUser> queue = new DelayQueue<>();
        int messageCount = 2;
        long delayTime = 500;
        DelayedQueueConsumer consumer = new DelayedQueueConsumer(
                queue, messageCount);
        DelayedQueueProducer producer = new DelayedQueueProducer(
                queue, messageCount, delayTime);

        
        executor.submit(producer);
        executor.submit(consumer);

        
        executor.awaitTermination(5, TimeUnit.SECONDS);
        executor.shutdown();

    }
```

上面的测试例子中，我们定义了两个线程的线程池，生产者产生两条消息，delayTime 设置为 0.5 秒，也就是说 0.5 秒之后，插入的对象能够被获取到。

线程池在 5 秒之后会被关闭。

运行看下结果：

```null
[pool-1-thread-1] INFO com.flydean.DelayedQueueProducer - put delayedUser DelayedUser(name=917, avaibleTime=1587623188389)
[pool-1-thread-2] INFO com.flydean.DelayedQueueConsumer - take DelayedUser(name=917, avaibleTime=1587623188389)
[pool-1-thread-1] INFO com.flydean.DelayedQueueProducer - put delayedUser DelayedUser(name=487, avaibleTime=1587623188899)
[pool-1-thread-2] INFO com.flydean.DelayedQueueConsumer - take DelayedUser(name=487, avaibleTime=1587623188899)
```

我们看到消息的 put 和 take 是交替进行的，符合我们的预期。

如果我们做下修改，将 delayTime 修改为 50000，那么在线程池关闭之前插入的元素是不会过期的，也就是说消费者是无法获取到结果的。

DelayQueue 是一种有奇怪特性的 BlockingQueue，可以在需要的时候使用。

本文的例子[https://github.com/ddean2009/learn-java-collections](https://github.com/ddean2009/learn-java-collections)

> 欢迎关注我的公众号: 程序那些事，更多精彩等着您！  
> 更多内容请访问 [www.flydean.com](http://www.flydean.com/java-delayqueue/) 
>  [https://www.cnblogs.com/flydean/p/java-delayqueue.html](https://www.cnblogs.com/flydean/p/java-delayqueue.html)
