# (10条消息) java定时器之Timer使用与原理分析_路漫漫，水迢迢-CSDN博客_java定时器原理
**Timer 和 TimerTask**

Timer 是 jdk 中提供的一个定时器工具，使用的时候会在主线程之外起一个单独的线程执行指定的计划任务，可以指定执行一次或者反复执行多次。

TimerTask 是一个实现了 Runnable 接口的抽象类，代表一个可以被 Timer 执行的任务。

**【使用举例】** 

【schedule(TimerTask task, long delay) 延迟 delay 毫秒 执行】

````null
public static void main(String[] args) {for (int i = 0; i < 10; ++i) {new Timer("timer - " + i).schedule(new TimerTask() {                    println(Thread.currentThread().getName() + " run ");```

【schedule(TimerTask task, Date time) 特定时间执行】

```null
public static void main(String[] args) {for (int i = 0; i < 10; ++i) {new Timer("timer - " + i).schedule(new TimerTask() {                    println(Thread.currentThread().getName() + " run ");            }, new Date(System.currentTimeMillis() + 2000));```

【schedule(TimerTask task, long delay, long period) 延迟 delay 执行并每隔period 执行一次】

```null
public static void main(String[] args) {for (int i = 0; i < 10; ++i) {new Timer("timer - " + i).schedule(new TimerTask() {                    println(Thread.currentThread().getName() + " run ");```

**【原理分析】** 

timer底层是把一个个任务放在一个TaskQueue中，TaskQueue是以平衡二进制堆表示的优先级队列，他是通过nextExecutionTime进行优先级排序的，距离下次执行时间越短优先级越高，通过getMin()获得queue\[1\]

并且出队的时候通过synchronized保证线程安全，延迟执行和特定时间执行的底层实现类似，源码如下：

```null
private void sched(TimerTask task, long time, long period) {throw new IllegalArgumentException("Illegal execution time.");if (Math.abs(period) > (Long.MAX_VALUE >> 1))if (!thread.newTasksMayBeScheduled)throw new IllegalStateException("Timer already cancelled.");synchronized(task.lock) {if (task.state != TimerTask.VIRGIN)throw new IllegalStateException("Task already scheduled or cancelled");                task.nextExecutionTime = time;                task.state = TimerTask.SCHEDULED;if (queue.getMin() == task) ```

我们主要来看下周期性调度通过什么方式实现的，我们直接来分析源码如下：

```null
private void mainLoop() {while (queue.isEmpty() && newTasksMayBeScheduled)long currentTime, executionTime;synchronized(task.lock) {if (task.state == TimerTask.CANCELLED) {                        currentTime = System.currentTimeMillis();                        executionTime = task.nextExecutionTime;if (taskFired = (executionTime<=currentTime)) {                                task.state = TimerTask.EXECUTED;                                  task.period<0 ? currentTime - task.period                                                : executionTime + task.period);                        queue.wait(executionTime - currentTime);            } catch(InterruptedException e) {```

这种定时器适合单点或者多台同时执行互不影响的场景

**【缺点】** 

1、首先Timer对调度的支持是基于绝对时间的，而不是相对时间，所以它对系统时间的改变非常敏感。

2、其次Timer线程是不会捕获异常的，如果TimerTask抛出的了未检查异常则会导致Timer线程终止，同时Timer也不会重新恢复线程的执行，他会错误的认为整个Timer线程都会取消。同时，已经被安排单尚未执行的TimerTask也不会再执行了，新的任务也不能被调度。故如果TimerTask抛出未检查的异常，Timer将会产生无法预料的行为

3、Timer在执行定时任务时只会创建一个线程任务，如果存在多个线程，若其中某个线程因为某种原因而导致线程任务执行时间过长，超过了两个任务的间隔时间，会导致下一个任务执行时间滞后

这些缺点可以通过ScheduledExecutorService来代替 
 [https://blog.csdn.net/fuyuwei2015/article/details/83825851](https://blog.csdn.net/fuyuwei2015/article/details/83825851)
````
