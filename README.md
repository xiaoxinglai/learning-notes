# learning-notes
学习笔记和总结的思维导图

#### 1.如何看源码
![avatar](https://github.com/xiaoxinglai/learning-notes/blob/master/png/%E7%9F%A5%E8%AF%86%E6%80%BB%E7%BB%93/%E5%A6%82%E4%BD%95%E7%9C%8B%E6%BA%90%E7%A0%81.png)

### java并发编程之美
#### 并发编程线程基础
##### 1.什么是线程？
![avatar](https://github.com/xiaoxinglai/learning-notes/blob/master/png/java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E4%B9%8B%E7%BE%8E/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80/%E4%BB%80%E4%B9%88%E6%98%AF%E7%BA%BF%E7%A8%8B.png)

##### 2.线程创建与运行
![avatar](https://github.com/xiaoxinglai/learning-notes/blob/master/png/java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E4%B9%8B%E7%BE%8E/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80/%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%88%9B%E5%BB%BA%E4%B8%8E%E8%BF%90%E8%A1%8C.png)

##### 3.wait
![avatar](https://github.com/xiaoxinglai/learning-notes/blob/master/png/java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E4%B9%8B%E7%BE%8E/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80/wait%E5%87%BD%E6%95%B0.png)


######  使用synchronized和wait以及notify实现生产者-消费者模式的阻塞队列
队列满了则生产者等待 
队列每放入一个元素就调用一次notifyAll唤醒消费线程

队列空了则消费者等待
队列每移除一个元素则调用notifyAll唤醒生产线程


```

class BlockQueue {

    private int max = 10;
    private List<String> queue = new ArrayList<>(max);


    public String get() throws InterruptedException {
        synchronized (queue) {
            //队列为空
            //if会有虚假唤醒的问题！！
            while (queue.size() == 0) {
                queue.wait();
                System.out.println("进入消费阻塞");
            }

            String s = queue.remove(0);
            queue.notifyAll();
            return s;
        }
    }

    public void put(String s) throws InterruptedException {
        synchronized (queue) {
            while (queue.size() == max) {
                System.out.println("进入生产阻塞");
                queue.wait();
            }
            queue.add(s);
            queue.notifyAll();
        }
    }


}
```

注意 不能用if 而是要用while去判断等待条件 就是为了防止虚假唤醒问题
https://blog.csdn.net/qq_20009015/article/details/95588574  


这里使用了queue作为共享变量，用synchronized获取的是queue的监视器锁

注意！！ 调用XXobject.wait()或者notify的时候，就必须是获取到了XXobject的监视器锁，其他人的监视器锁不算，否则会报IllegalMoniterState的异常


这里使用了ArrayList作为元素的容器，队列是先进先出。出的时候每次都remove头元素
remove(0),
```
 /**
     * Removes the element at the specified position in this list.
     * 移除指定位置的元素
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).
     * 将任何后续元素向左移位（从*索引中减去一个）
     *
     * @param index the index of the element to be removed
     * @return the element that was removed from the list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
```
remove第一个元素之后，后面的元素会自动往左移位





测试如下
```

  public static void main(String[] args) {

        BlockQueue blockQueue = new BlockQueue();


        new Thread(() -> {

            int i = 0;
            while (true) {
                try {
                    i++;
                    String s = blockQueue.get();
                    System.out.println("消费：" + System.currentTimeMillis() + ":" + s);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }


            }
        }).start();


        new Thread(() -> {
            int i = 0;
            while (true) {
                try {
                    blockQueue.put(String.valueOf(i));
                    System.out.println("生产：" + System.currentTimeMillis() + ":" + i);

                    Thread.sleep(1000);

                    i++;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }).start();


    }

```

>
进入消费阻塞 . 
生产：1562911018531:0 . 
消费：1562911018531:0. 
生产：1562911019532:1. 
消费：1562911020535:1. 
生产：1562911020536:2. 
生产：1562911021541:3. 
消费：1562911022538:2. 
生产：1562911022542:4. 
生产：1562911023546:5. 
消费：1562911024540:3. 
生产：1562911024550:6. 
生产：1562911025553:7. 
消费：1562911026544:4. 
生产：1562911026556:8. 
生产：1562911027560:9. 
消费：1562911028548:5. 
生产：1562911028563:10. 
生产：1562911029567:11. 
消费：1562911030551:6. 
生产：1562911030568:12. 
生产：1562911031569:13. 
消费：1562911032554:7. 
生产：1562911032573:14. 
生产：1562911033574:15. 
消费：1562911034558:8. 
生产：1562911034577:16. 
生产：1562911035581:17. 
消费：1562911036563:9. 
生产：1562911036583:18. 
生产：1562911037586:19. 
消费：1562911038565:10. 
生产：1562911038589:20. 
进入生产阻塞. 
消费：1562911040567:11. 
生产：1562911040567:21. 





######  wait的死锁例子
当前线程调用共享变量的wait()方法之后只会释放当前共享变量上的锁，比如说XX.wait(). 只会释放XX上的监视器锁，如果当前线程还持有其他共享变量的锁，这些锁是不会释放的。

因此可以模拟一下，长期获取不到锁的情况

情况1:线程A资源一直不释放，另线程B则一直请求资源 
解决方法就是打破一直持有的条件，比如说超时释放或者请求超时就放弃

```
package deadLock;

/**
 * @ClassName deadLock1
 * @Author laixiaoxing
 * @Date 2019/7/13 上午10:19
 * @Description 长时间等待资源的死锁
 * @Version 1.0
 */
public class DeadLock1 {


    private static volatile Object resourceA=new Object();
    private static volatile Object resourceB=new Object();

    public static void main(String[] args) {
        new Thread(()->{
            synchronized (resourceA){
                System.out.println("获取resourceA的锁");
                synchronized (resourceB){
                    System.out.println("获取resourceB的锁");
                    try {
                        //进入等待状态，此时会释放resource的监视器锁，但是不会释放resourceB的监视器锁
                        System.out.println("进入等待状态并释放resourceA的锁");
                        resourceA.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                }
            }

        }).start();


        new Thread(()->{
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            synchronized (resourceA){
                System.out.println("获取resourceA的锁");
                synchronized (resourceB){
                    System.out.println("获取resourceB的锁");
                    try {
                        resourceA.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }

                }
            }

        }).start();
    }

}


```


>获取resourceA的锁  
>获取resourceB的锁  
>进入等待状态并释放resourceA的锁  
>获取resourceA的锁


情况2:循环等待
线程A获取到资源a并请求资源b,线程B获取到资源b并请求资源a
解决的方式就是 一次性获取所有资源，或者获取不到资源的超时进行放弃。


```
package deadLock;

/**
 * @ClassName deadLock1
 * @Author laixiaoxing
 * @Date 2019/7/13 上午10:19
 * @Description 长时间等待资源的死锁
 * @Version 1.0
 */
public class DeadLock2 {


    private static volatile Object resourceA = new Object();
    private static volatile Object resourceB = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resourceA) {
                System.out.println("获取resourceA的锁");
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (resourceB) {
                    System.out.println("获取resourceB的锁");
                    //进入等待状态，此时会释放resource的监视器锁，但是不会释放resourceB的监视器锁
                    System.out.println("进行等待状态并释放resourceA的锁");
                }
            }

        }).start();


        new Thread(() -> {


            synchronized (resourceB) {
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("获取resourceB的锁");
                synchronized (resourceA) {
                    System.out.println("获取resourceA的锁");

                }
            }

        }).start();


    }


}
    

```

>获取resourceA的锁
获取resourceB的锁



##### wait状态下线程的中断

当一个线程调用共享对象的wait（）方法被阻塞挂起后，其他线程中断了该线程，则该线程会在wait所在的行抛出InterruptedException异常并返回, 不会继续往下执行

https://blog.csdn.net/qq_20009015/article/details/89449749  正常执行状态下的线程 去调用Interrupt ，线程本身不会有任何变化，这个响应逻辑需要自己写

```
/**
 * @ClassName WaitNotifyInterupt
 * @Author laixiaoxing
 * @Date 2019/7/13 下午12:02
 * @Description 线程中断的例子
 * @Version 1.0
 */
public class WaitNotifyInterupt {

    static Object obj = new Object();

    public static void main(String[] args) {
        Thread threadA = new Thread(() -> {
            synchronized (obj) {
                try {
                    System.out.println("begin");
                    obj.wait();
                    System.out.println("end");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });
        threadA.start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadA.interrupt();
        System.out.println("Thread end");


    }


}

```

输出结果:
```
begin  
java.lang.InterruptedException  
at java.lang.Object.wait(Native Method)  
at java.lang.Object.wait(Object.java:502)  
at WaitNotifyInterupt.lambda$main$0(WaitNotifyInterupt.java:17)  
at java.lang.Thread.run(Thread.java:748)  
Thread end
```




##### notify和notifyAll的用法
![avatar](https://github.com/xiaoxinglai/learning-notes/blob/master/png/java%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E4%B9%8B%E7%BE%8E/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E7%BA%BF%E7%A8%8B%E5%9F%BA%E7%A1%80/notify.png)

```
public class NotyfyDemo {


    private static volatile Object resourceA = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resourceA) {
                System.out.println("threadA get resourceA lock");
                try {
                    resourceA.wait();
                    System.out.println("threadA end wait");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }).start();

        new Thread(() -> {
            synchronized (resourceA) {
                System.out.println("threadB get resourceA lock");
                try {
                    resourceA.wait();
                    System.out.println("threadB end wait");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

            }
        }).start();


        new Thread(() -> {

            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (resourceA) {
                System.out.println("threadC get resourceA lock");
                System.out.println("threadC begin notify");
                resourceA.notify();
                  System.out.println("threadC end notify");

            }
        }).start();


    }
}

```

```
threadA get resourceA lock
threadB get resourceA lock
threadC get resourceA lock
threadC begin notify
threadC end notify
threadA end wait
```


ThreadC先sleep了1秒  为了让ThreadA和ThreadB都能获取锁并进入wait. 
然后调用notify 随机唤醒一个 ，这里是唤醒了ThreadA， 然后ThreadA继续执行到wait

这里需要注意的是：
1. **threadC begin notify ，ThreadC执行notify之后，并不会立即释放锁，仍然还是继续正常执行， 此时ThreadA被随机唤醒，也无法继续运行， 需要等待ThreadC的锁释放。**    所以运行结果才会是threadC begin notify -》threadC end notify-》threadA end wait
2.要想调用某对象的notify，前提是获取了该对象的监视器锁比如说
```
 synchronized (**resourceB**) {
                System.out.println("threadC get resourceA lock");
                System.out.println("threadC begin notify");
                **resourceA**.notify();
                System.out.println("threadC end notify");

            }
```
这里获取的是resourceB的监视器锁，却调用的resourceA的notify
运行时报错：
java.lang.IllegalMonitorStateException







**notifyAll**
 **resourceA**.notifyAll(); 此时会唤醒所有因为调用resourceA.wait而阻塞的线程
 

```
 synchronized (**resourceA**) {
                System.out.println("threadC get resourceA lock");
                System.out.println("threadC begin notify");
                **resourceA**.notifyAll();
                System.out.println("threadC end notify");

            }
```

```
threadA get resourceA lock
threadB get resourceA lock
threadC get resourceA lock
threadC begin notify
threadC end notify
threadB end wait
threadA end wait
```

##### join的用法
join方法是Thread类直接提供的，线程A调用线程B的join方法后，会被阻塞，等待B完成。
如果此时其他线程调用了线程A的interrupt()方法，线程A会抛出InterruptException异常



```
 public static void main(String[] args) throws InterruptedException {
        Thread threadOne=new Thread(()->{
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("child threadOne over!");
        });



        Thread threadtwo=new Thread(()->{
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            System.out.println("child threadTwo over!");
        });


        System.out.println("time:"+System.currentTimeMillis());
        threadOne.start();
        threadtwo.start();
        System.out.println("wait all child thread over!");
        threadOne.join();
        threadtwo.join();
        System.out.println("time:"+System.currentTimeMillis());
        
    }
```

输出
```
time:1563012587684
wait all child thread over!
child threadTwo over!
child threadOne over!
time:1563012588687
```

这里main线程调用了threadOne的join方法 ，就会进行阻塞 等待threadOne执行完毕之后返回，只有threadOne执行完毕之后，threadOne.join()方法才会返回。
  返回之后在main线程调用threadtwo.join()，等待threadTwo完成。
这里，threadOne和ThreadTwo都只执行1秒，所以最终的运行结果就是main线程阻塞了1秒。


修改ThreadOne和ThreadTwo的sleep时间为5秒，3秒，最终需要等待的时间是5秒
```
time:1563012942271
wait all child thread over!
child threadOne over!
child threadTwo over!
time:1563012947272
```

改成
```
   System.out.println("time:"+System.currentTimeMillis());
        threadOne.start();
        threadtwo.start();
        System.out.println("wait threadOne child thread over!");
        threadOne.join();
        System.out.println("wait threadtwo child thread over!");
        threadtwo.join();
        System.out.println("time:"+System.currentTimeMillis());

```

输出
```
time:1563013009495
wait threadOne child thread over!
child threadOne over!
wait threadtwo child thread over!
child threadTwo over!
time:1563013014500
```


如果在join的时候，有其他线程调用了该线程的interrupter方法，此时会在该线程调用join()所在的行抛出InterruptedException 并立即返回。


```
//获取主线程
        Thread mainThread=Thread.currentThread();

        new Thread(()->{
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            mainThread.interrupt();
            System.out.println("Interrupted main!");
        }).start();
```


输出如下
```
time:1563013331167
wait threadOne child thread over!
Interrupted main!
Exception in thread "main" java.lang.InterruptedException
	at java.lang.Object.wait(Native Method)
	at java.lang.Thread.join(Thread.java:1252)
	at java.lang.Thread.join(Thread.java:1326)
	at JoinDemo.main(JoinDemo.java:52)
child threadOne over!
child threadTwo over!

```


##### sleep方法的使用
Thread类中的静态方法sleep()，当一个执行中的线程调用了Thread的sleep()方法后，调用线程会暂时让出时间的执行权，这期间不参与cpu的调度，但是该线程持有的锁是不让出的。时间到了会正常返回，线程处于就绪状态，然后参与cpu调度，获取到cpu资源之后就可以运行。

如果在睡眠期间，其他线程调用了该线程的interrup()的方法中断了该线程，则该线程会调用sleep方法的地方抛出InterruptedException异常而返回

```
public class ThreadDemo {

    private static Object lock = new Object();

    public static void main(String[] args) {
        new Thread(()->{
            synchronized (lock){
                try {
                    System.out.println("A休眠10秒不放弃锁");
                    Thread.sleep(10000);
                    System.out.println("A休眠10秒醒来");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }

        }).start();



        new Thread(()->{

            synchronized (lock){
                System.out.println("B休眠10秒不放弃锁");
                try {
                    Thread.sleep(10000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("B休眠10秒醒来");

            }

        }).start();



    }
}
```


无论执行多少次，都是先A输出再B输出 或者先B输出再A输出，不会出现交叉输出的情况，
因为A获取到锁之后，即使是sleep也不会释放锁，因B获取不到锁，也就无法执行。

输出结果
```
A休眠10秒不放弃锁
A休眠10秒醒来
B休眠10秒不放弃锁
B休眠10秒醒来
```
或者
```
B休眠10秒不放弃锁
B休眠10秒醒来
A休眠10秒不放弃锁
A休眠10秒醒来
```


##### 让出cpu执行权的yield方法

Thread类中有一个静态的yield方法，当一个线程调用yield方法时，实际就是暗示线程调度器当前请求让出自己的cpu使用，但是线程调度器可以无条件忽略这个暗示。

操作系统为每一个线程分配一个时间片来占有cpu，正常情况下当一个线程把分配给自己的时间片用完之后，线程调度器才会进行下一轮的线程调度，而当一个线程调用了Thread类的静态方法yield时，是在告诉线程调度器自己占用的时间片还没用完，但是不想用了，让调度器现在就可以进行下一轮的线程调度

当一个线程调用yield方法时，当前线程会让出cpu使用权，然后处于就绪状态，线程调度器会从线程就绪队列里面获取一个优先级最高的线程，但是也有可能又会调度到刚刚让出cpu的哪个线程。

注意点：调用yield之后，线程是进入了就绪态，线程调度器下一次调度时还有可能调度到这个线程。 和调用sleep不同，sleep则是让线程阻塞挂起一段时间，期间线程不会被调度。 


目前能让线程阻塞方法有：join（）,wait（）,sleep（）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190713202251415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)


例子如下：
不加yield的情况下
```
public class yield {


    public static void main(String[] args) {
         new Thread(() -> {
            test();
        }).start();

        new Thread(() -> {
            test();
        }).start();


        new Thread(() -> {
            test();
        }).start();


    }

    private static void test() {
        for (int i = 0; i < 5; i++) {
            //当i=0时让出cpu执行权，放弃时间片，进行下一轮调度
            if ((i % 5) == 0) {
                System.out.println(Thread.currentThread() + "yield cpu ...");
            }

        }
        System.out.println(Thread.currentThread() + "is over");
    }
}
```

输出结果为
```
Thread[Thread-0,5,main]yield cpu ...
Thread[Thread-0,5,main]is over
Thread[Thread-1,5,main]yield cpu ...
Thread[Thread-1,5,main]is over
Thread[Thread-2,5,main]yield cpu ...
Thread[Thread-2,5,main]is over
```



加上yield之后
```
for (int i = 0; i < 5; i++) {
            //当i=0时让出cpu执行权，放弃时间片，进行下一轮调度
            if ((i % 5) == 0) {
                System.out.println(Thread.currentThread() + "yield cpu ...");
               **Thread.yield();**
            }

        }
        System.out.println(Thread.currentThread() + "is over");
```

结果如下
```
Thread[Thread-0,5,main]yield cpu ...
Thread[Thread-1,5,main]yield cpu ...
Thread[Thread-2,5,main]yield cpu ...
Thread[Thread-0,5,main]is over
Thread[Thread-1,5,main]is over
Thread[Thread-2,5,main]is over
```

线程在启动之后，调用了yield方法，让出了自己的cpu执行时间，导致这个线程两个输出没有跟在一起。 

但是以上的结果不是每次都是这样，只是大概率，因为线程调度器可以不理会yield请求



##### interrupt的用法
java中的线程中断是一种线程间的协作模式，通过设置线程的中断标志并不能直接终止该线程的执行，而是被中断的线程根据中断状态自行处理。


void interrupt()方法:
中断线程，例如当线程A运行时，线程B可以调用线程A的interrupt()方法来设置线程A的中断标志为true并立即返回。设置标志仅仅是设置标志，线程A实际上并没有被中断，它会继续往下执行。如果线程A因为调用了wait系列的函数，join方法或者sleep方法而被阻塞挂起，这时候若是线程B调用线程A的interrupt()方法，线程A会在调用这些方法的地方抛出InterruptedException异常而返回。


boolean  isInterrupted()方法:
检查当前线程是否被中断，如果是返回true，否则返回false

isInterrupted()的内部
```
public static boolean interrupted(){
return currentThread.isInterrupted(false);
}
```
false表示不清除中断状态


boolean interrupted()方法:
检测当前线程是否中断，如果是返回true,否则返回false。与isInterrupted不同的是，该方法如果发现当前线程被中断，则会清除中断标志，也就是如果第一次调用是true,再次调用返回的就是false，因为之前的中断状态被清除了。
并且该方法是static方法，可以通过Thread类直接调用。

目前已知的Thread类的静态方法有sleep(),yield(),interrupted()

interrupted的内部
```
public static boolean interrupted(){
return currentThread.isInterrupted(true);
}
```

true表示清除中断标志，这里是调用当前线程的isInterrupted方法获取当前线程的中断状态，而不是调用interrupted的实例对象的中断状态
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019071321272649.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)


使用中断的例子


```
 public static void main(String[] args) throws InterruptedException {

        Thread t = new Thread(() -> {

            while (!Thread.currentThread().isInterrupted()) {
                System.out.println(Thread.currentThread() + "  hello");
            }

        });
        t.start();
        
        //中断子线程
        System.out.println("main thread interrupt thread");
        t.interrupt();
        //等待子线程执行完毕
        t.join();

        System.out.println("main is over");

    }
```


输出如下
```
main thread interrupt thread
Thread[Thread-0,5,main]  hello
main is over
```


设计思路:
在线程里面使用了一个while循环来进行工作，如果不是中断状态就继续执行，如果是就跳出循环，跳出循环之后，这个线程就执行结束了，自然消亡。 

```
try {
          while (!Thread.currentThread().isInterrupted() && more work todo){
                    //do more work
                }
            } catch (InterruptedException e) {
                //thread was interrupted during sleep or wait
            } finally {
                //cleanup,if required
            }
```


另一种场景：
当线程为了等待一些特定的条件，会调用sleep或者wait或者join来阻塞挂起当前线程。 比如说有某个任务要3秒后才会到来，于是调用了sleep（3000）等待，那么3秒后线程才会激活，但是有可能3秒内那个需要处理的任务已经到了，这个时候继续阻塞下去就浪费了时间，可以用interrupter让这个线程从阻塞状态打断。


比如说：
```
public static void main(String[] args) throws InterruptedException {

        Thread t = new Thread(() -> {

            try {
                System.out.println("threadOne begin sleep for 2000 seconds");
                Thread.sleep(200000);
                System.out.println("threadOne awaking");
            } catch (InterruptedException e) {
                System.out.println("threadOne is interrupted while sleeping");
                return;
            }
        });


        t.start();

        Thread.sleep(1000);
        //中断子线程 让它从休眠中返回
        System.out.println("main thread interrupt thread");
        t.interrupt();
        //等待子线程执行完毕
        t.join();

        System.out.println("main is over");

    }

```

本了应该要休眠200秒，然后被中断了 ，捕获到异常之后，进行catch处的逻辑。


##### interrupted和isInterrupted的区别


boolean  isInterrupted()方法
1.isInterrupted是实例方法 
检查当前线程是否被中断，如果是返回true，否则返回false,不会清除中断状态

boolean interrupted()方法
1.interrupted是Thread的静态方法

检测当前线程是否中断，如果是返回true,否则返回false。与isInterrupted不同的是，该方法如果发现当前线程被中断，则会清除中断标志，也就是如果第一次调用是true,再次调用返回的就是false，因为之前的中断状态被清除了。
并且该方法是static方法，可以通过Thread类直接调用。


```
public class isInterrupted {
    public static void main(String[] args) {
        Thread t=new Thread(()->{
            for(;;){

            }
        });

        t.start();
        //设置中断标志
        t.interrupt();

        //获取中断标志 true
        System.out.println("t.isInterrupted"+t.isInterrupted());

        //获取中断标志并重置  false
        System.out.println("t.Interrupted"+t.interrupted());

        //获取中断标志并重置 false
        System.out.println("Thread.Interrupted"+Thread.interrupted());

        //获取中断标志 true
        System.out.println("t.isInterrupted"+t.isInterrupted());


    }
}

```

输出结果
```
t.isInterrupted: true
t.Interrupted: false
Thread.Interrupted: false
t.isInterrupted: true

```
为什么会是这个结果？

       //设置中断标志  设置之后线程实例t的状态为中断 是否中断为true
        t.interrupt(); 

       //获取中断标志    这里是获取了t的中断状态 且不重置，因此是true
        System.out.println("t.isInterrupted"+t.isInterrupted());

        //获取中断标志并重置  false    这里面 t.interrupted() 是获取并重置当前线程的状态， 当前线程是main线程,interrupted是Thread的静态方法，虽然可以通过实例去调
        //因为获取的是当前main线程的中断状态 因此返回的是false
      
        System.out.println("t.Interrupted"+t.interrupted());

        //获取中断标志并重置 false   同上
        System.out.println("Thread.Interrupted"+Thread.interrupted());

        //获取中断标志 true   获取的是t的状态
        System.out.println("t.isInterrupted"+t.isInterrupted());


##### 理解线程上下文切换

  在多线程编程中，线程个数一般都大于cpu个数，而每个cpu同一时刻只能被一个线程使用，为了让用户感觉多个线程是在同时执行的，cpu资源采用了时间片轮转的策略，也就是给每一个线程分配一个时间片，线程在时间片内占用cpu执行任务。当前线程使用时间片之后，就会处于就绪状态并让出cpu让其他线程占用，这就是上下文切换，从当前线程的上下文切换到了其他线程。

  那么就有一个问题，让出cpu的线程等下次轮到自己占有cpu的时，如何知道自己之前运行到了哪里，所以在切换上下文时需要保存当前线程的执行现场，当再次执行时根据保存的执行现场信息恢复执行现场。

线程上下文切换的时机：
当前线程的cpu时间片使用完处于就绪状态时，当前线程被其他线程中断时



##### 线程死锁

什么是线程死锁？
死锁是指两个或者两个以上的线程在执行过程中，因为争夺资源而造成的互相等待的现象。如果没有外力作用下，这些线程会一直互相等待而无法继续运行下去。


比如说线程A持有资源1，等待资源2
线程B持有资源2，等待资源1
且双方都不愿意放弃自己所持有的资源

死锁的四个条件：
1.互斥条件：资源只能同时被一个线程占用，如果此时有其他线程想要获取资源，则必须等待，直到占有资源的线程释放该资源
2.请求并持有条件：
指一个线程已经持有了至少一个资源，但是又提出了新的资源请求，而新资源已被其他线程占用，所以当前线程会被阻塞，但是又不释放自己的资源。
3.不可剥夺条件：
指线程获取到资源在自己使用完之前不被其他线程抢占，只有在自己使用完毕之后才由自己释放该资源。
4.环路等待条件：指在发生死锁时 必然存在一个线程-资源的环形链，比如说线程1等待线程2的资源，线程2等待线程1的资源 

```

ublic class DeadLock2 {


    private static volatile Object resourceA = new Object();
    private static volatile Object resourceB = new Object();

    public static void main(String[] args) {
        new Thread(() -> {
            synchronized (resourceA) {
                System.out.println("获取resourceA的锁");
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (resourceB) {
                    System.out.println("获取resourceB的锁");
                    //进入等待状态，此时会释放resource的监视器锁，但是不会释放resourceB的监视器锁
                    System.out.println("进行等待状态并释放resourceA的锁");
                }
            }

        }).start();


        new Thread(() -> {


            synchronized (resourceB) {
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("获取resourceB的锁");
                synchronized (resourceA) {
                    System.out.println("获取resourceA的锁");

                }
            }

        }).start();


    }


}
```


输出为：
```
获取resourceA的锁
获取resourceB的锁
```
然后就无法继续往下执行了，进入了死锁状态



如何避免死锁？
只需要破坏掉四个死锁条件中的一个构造死锁的必要条件即可。
但是只有请求并持有条件和环路等待条件是可以被破坏的。 
资源的互斥条件和不可剥夺条件无法被破坏。

**资源申请的有序性原则**

让多个线程都使用相同的顺序去获取资源。
比如说之前的线程1获取资源A再获取资源B,线程2获取资源B再获取资源A，这样有可能会发生死锁
但是改成，线程1先获取资源A,再获取B，线程2也先获取资源A再获取资源B. 
这样资源A只能被一个线程获取，只有获取到了资源A的线程才会继续去获取资源B。 从而避免了死锁

同时破坏了环路等待条件和请求并持有条件


**获取超时则放弃**

破坏了请求并持有条件