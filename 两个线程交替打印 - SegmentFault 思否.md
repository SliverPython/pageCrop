# 两个线程交替打印 - SegmentFault 思否
![](https://sponsor.segmentfault.com/lg.php?bannerid=0&campaignid=0&zoneid=5&loc=https%3A%2F%2Fsegmentfault.com%2Fq%2F1010000021259515&referer=https%3A%2F%2Fwww.baidu.com%2Flink%3Furl%3D3afQJiRoLO1ZIJFxZMynemh6i4DoffGHylhcuGw1jTAImNWCBYW5-zFzMuwgA99e1czI5qnoMN364kUoiQjMtq%26wd%3D%26eqid%3D8e64e52b0015ce8b000000046020d436&cb=f0d598b560)

![](https://sponsor.segmentfault.com/lg.php?bannerid=0&campaignid=0&zoneid=2&loc=https%3A%2F%2Fsegmentfault.com%2Fq%2F1010000021259515&referer=https%3A%2F%2Fwww.baidu.com%2Flink%3Furl%3D3afQJiRoLO1ZIJFxZMynemh6i4DoffGHylhcuGw1jTAImNWCBYW5-zFzMuwgA99e1czI5qnoMN364kUoiQjMtq%26wd%3D%26eqid%3D8e64e52b0015ce8b000000046020d436&cb=824f87592f)

1.  [首页](https://segmentfault.com/)
2.  [问答](https://segmentfault.com/questions)
3.  [java](https://segmentfault.com/t/java/questions)
4.  问答详情


     public class PrintABVolatile implements IPrintAB {
        private static final int MAX_PRINT_NUM = 100;
        private static volatile int count = 0;
    ​
        @Override
        public void printAB() {
            new Thread(() -> {
                while (count < MAX_PRINT_NUM) {
                    if (count % 2 == 0) {
                        log.info("num:" + count);
                        count++;
                    }
                }
            }).start();
    ​
            new Thread(() -> {
                while (count < MAX_PRINT_NUM) {
                    if (count % 2 == 1) {
                        log.info("num:" + count);
                        count++;
                    }
                }
            }).start();
        }
    }

请问这样能保证线程安全吗.. 感觉 volatile 不能保证啊 i++ 不是原子的

[![](https://sponsor-static.segmentfault.com/cafbf2f9192bd0d6314fbb98cb393e69.png)
](https://sponsor.segmentfault.com/ck.php?oaparams=2__bannerid=454__zoneid=3__cb=4d1354f477__oadest=https%3A%2F%2Fjinshuju.net%2Ff%2FgmdQ7p%3Fx_field_1%3D1)

![](https://sponsor.segmentfault.com/lg.php?bannerid=454&campaignid=114&zoneid=3&loc=https%3A%2F%2Fsegmentfault.com%2Fq%2F1010000021259515&referer=https%3A%2F%2Fwww.baidu.com%2Flink%3Furl%3D3afQJiRoLO1ZIJFxZMynemh6i4DoffGHylhcuGw1jTAImNWCBYW5-zFzMuwgA99e1czI5qnoMN364kUoiQjMtq%26wd%3D%26eqid%3D8e64e52b0015ce8b000000046020d436&cb=4d1354f477)

incrementAndGet() 底层是 CAS 操作，调用的是 Unsafe 类，线程安全

[**lunaticf**](https://segmentfault.com/u/lunaticf)：

谢谢... 是在一个地方看到的面试官给的线程交替打印的标准答案... 就有点困惑

### 我觉得是没有问题的

首先 volatile 是有可见性的，就是一个线程修改了 count，其他线程也可以获取 count 的最新值  
两个线程内部都是循环执行的，如果 count%2==0，那么线程一可以打印，那么线程二会一直循环，当 count++ 之后，线程二回获取到 count 值, 并执行打印。  
为什么会有 i++ 原子性问题呢？  
当然，我对线程也是一知半解，如果有错误欢迎指出错误

count++ 本身确实有原子性问题，但是 volatile 修饰后，可以认为两个线程同时看到的 count 的值是相同的，这样的话，关键点就是两个 while 循环里的 if 条件，是互斥的，也就是说，count 在同一时刻只能满足其中一个 if 的条件，而另外一个 if 不满足，就只能一直循环，等满足条件的 if 执行完修改了 count 的值之后再进入，这样就实现了交替

有问题，问题在于`count < MAX_PRINT_NUM / count % 2` 这两个判断不是连续原子的，可能导致`MAX_PRINT_NUM`依然被输出

##### 撰写回答

 [https://segmentfault.com/q/1010000021259515](https://segmentfault.com/q/1010000021259515)
