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


#####  守护线程与用户线程
java中的线程分为两类，分别为daemon线程和user线程，在jvm启动时候会调用main函数，main函数所在的线程就是一个用户线程，在JVM内部还启用了很多守护线程，比如说垃圾回收线程。



当最后一个非守护线程结束时候，JVM会正常退出，而不管当前是否有守护线程，也就是说守护线程是否结束并不影响JVM退出。

即只要有一个用户线程还没结束，正常情况下JVM就不会退出。


因此main线程结束之后，子线程如果是用户线程，那么不一定会结束，jvm也可能不结束


例子如下：
```

 public static void main(String[] args) {
        Thread t=new Thread(()->{
            for(;;){}
        });
        t.start();
        System.out.println("main thread is over");
    }

```

起了一个线程，里面是个死循环，那么main线程结束之后，子线程会结束吗？

答案是不会，运行jps 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190714132005104.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)

可以看到Daemon线程仍然在运行![在这里插入图片描述](https://img-blog.csdnimg.cn/20190714132031847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)

控制台的按钮也是红色，说明jvm没有退出
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019071413211916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)


如果设置该线程为守护线程呢？
```

 public static void main(String[] args) {
        Thread t=new Thread(()->{
            for(;;){}
        });
        //设置为守护线程
        t.setDaemon(true);
        t.start();
        System.out.println("main thread is over");
    }
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190714132223309.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190714132230474.png)

当main线程结束之后，子线程也随着结束了。

因这里，main线程是唯一的用户线程，用户线程结束，jvm则会退出。


总结:
如果你希望在主线程结束后JVM进程马上结束，那么在创建线程时可以将其设置为守护线程。
如果希望在主线程结束之后子线程继续工作，等子线程结束后再让JVM进程结束，那么就将子线程设置为用户线程



##### ThreadLocal介绍和原理解析

多线程访问一个共享变量时特别容易出现并发问题，特别是在多个线程需要对一个共享变量进行进入时。为了保证线程安全，一般使用着在访问共享变量时需要进行适当的同步

ThreadLocal可以做到，当创建一个变量之后，每个线程对其进行访问的时候，访问的是自己线程的变量。

ThreadLocal是jdk包提供的，它提供了线程本地变量，也就是如果你创建了ThreadLocal变量，那么访问这个变量的每个线程都会有这个变量的一个本地副本，从而避免了线程安全问题。创建一个ThreadLocal变量之后，每个线程都会复制一个变量到自己的本地内存

```
public class ThreadLocalTest {


    //(2)创建ThreadLocal本量
    static ThreadLocal<String> localVariable = new ThreadLocal<String>();


    /*
    * 本例：开启了两个线程，在每个线程内部都设置了本地变量的值，
    * 然后调用print函数打印当前本地变量。如果打印后调用了本地变量的remove方法，则会删除本地内存中的该变量
    * */

    //（1）print函数
    static void print(String str) {
        //1.1 打印当前线程本地内存中localVariable变量的值
        System.out.println(str + " : " + localVariable.get());
        localVariable.remove();
    }

    public static void main(String[] args) {
        new Thread(() -> {
            //设置线程One中本地变量localVariable
            localVariable.set("threadOne local variable");
            //调用打印函数
            print("threadOne");
            //打印本地变量值
            System.out.println("threadOne remove after" + " : " + localVariable.get());
        }).start();


        new Thread(() -> {
            //设置线程One中本地变量localVariable
            localVariable.set("threadTwo local variable");
            //调用打印函数
            print("threadTwo");
            //打印本地变量值
            System.out.println("threadTwo remove after" + " : " + localVariable.get());
        }).start();
    }
}

```

输出
```
threadOne : threadOne local variable
threadOne remove after : null
threadTwo : threadTwo local variable
threadTwo remove after : null
```


这里起了两个线程，每个线程  localVariable.set();  其实是将变量保存到了这个线程的threadLocals变量里面。 

ThreadLocal原理：
Thread类里面，有两个ThreadLocalMap类型的变量threadLocals和inheritableThreadLocals。 默认情况下这两个值都是Null，只有当前线程第一次调用ThreadLocal的set或者get方法时才会创建它们。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190716010643643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)

ThreadLocalMap是一个定制化的HashMap。

其实每个线程的本地变量并不是放在ThreadLocal实例里面，而是存放在调用线程的threadLocals变量里面。
也就是说，ThreadLocal类型的本地变量存放在具体的线程内存空间里，ThreadLocal就是一个工具壳，它通过set方法把value值放入调用线程的threadLocals里面存放起来，当调用线程调用它的get方法时，再从当前线程的threadLocals变量里面将其拿出来使用。

所以当不需要使用本地变量时可以通过调用ThreadLocal变量的remove方法，从当前线程的threadLocals里面删除该本地变量。

为什么ThreadLocal里面的threadLocals被设计成Map结构？因为每个线程都可以关联多个ThreadLocal变量


ThreadLocal的set、get和remove方法的实现逻辑
```
 /**
     * Sets the current thread's copy of this thread-local variable
     * to the specified value.  Most subclasses will have no need to
     * override this method, relying solely on the {@link #initialValue}
     * method to set the values of thread-locals.
     *
     * @param value the value to be stored in the current thread's copy of
     *        this thread-local.
     */
     /**
      *设置此线程局部变量的当前线程副本
      *到指定的值。 大多数子类都没有必要
      *覆盖此方法，完全依赖{@link #initialValue}
      *设置线程局部的值的方法。
     *
      * @param值存储在当前线程的副本中
      *这个线程本地。
      */
      
    public void set(T value) {
       //获取当前线程
        Thread t = Thread.currentThread();
        //将当前线程作为key, 去查找对应的线程变量threadLocals，找到则设置
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
        //第一次调用就创建当前线程对应的hashMap
            createMap(t, value);
    }
```


 ThreadLocalMap map = getMap(t);
可以看到，这个getMap就是获取Thread里面的变量threadLocals，这个是个map
```
**
     * Get the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param  t the current thread
     * @return the map
     */
    ThreadLocalMap getMap(Thread t) {
        return t.threadLocals;
    }
```


如果getMap(t)的返回值不为空，则把value的值设置到threadLocals中，也就是把当前的变量值放入到当前线程的内存变量threadLocals中。 
threadLocals是一个HashMap结构，其中key就是当前ThreadLocal的实例对象引用，value是通过set方法传递的值。

也就是说threadLocals的其实是形如Map<ThreadLocal, T value> 的结构
其类型为ThreadLocalMap 
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019071601173989.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)


如果getMap(t) 为空则说明是第一次调用set方法，这时创建当前线程的threadLocals变量。

```

 /**
     * Create the map associated with a ThreadLocal. Overridden in
     * InheritableThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the map
     */
    void createMap(Thread t, T firstValue) {
       //给线程实例的threadLocals初始化值
        t.threadLocals = new ThreadLocalMap(this, firstValue);
    }
```



再看看如何获取线程本地变量
```
 /**
     * Returns the value in the current thread's copy of this
     * thread-local variable.  If the variable has no value for the
     * current thread, it is first initialized to the value returned
     * by an invocation of the {@link #initialValue} method.
     *
     * @return the current thread's value of this thread-local
     */
    public T get() {
    //获取当前线程
        Thread t = Thread.currentThread();
        //获取当前线程的threadlocals变量
        ThreadLocalMap map = getMap(t);
        //如果threadlocals的值不为空，则根据ThreadLocal实例的引用作为key，获取之前放入的值
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```


先获取当前线程实例，如果当前线程的threadLocals变量不为null,则直接返回当前线程绑定的本地变量，否则执行代码setInitialValue()进行初始化。

```
 /**
     * Variant of set() to establish initialValue. Used instead
     * of set() in case user has overridden the set() method.
     *
     * @return the initial value
     */
    private T setInitialValue() {
    //初始化为null
        T value = initialValue();
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        //如果当前线程的threadLocals变量不为空
        if (map != null)
            map.set(this, value);
        else
        //如果当前线程的threadlocals变量为空
            createMap(t, value);
        return value;
    }

  protected T initialValue() {
        return null;
    }

```


如果当前线程的threadLocal变量不为空，则设置当前线程的本地变量值为努力了，
否则调用createMap方法创建当前线程的createMap变量。


这里，也可以继承ThreadLocal类，重写initialValue方法，设置自己的默认值。
```
class   StringThreadLocal extends ThreadLocal{
      @Override
      protected Object initialValue() {
          return  "赖晓星";
      }
  }

```

这里第一次调用StringThreadLocal实例的get方法的时候，就会获取到这个值。
```
  public static void main(String[] args) {
  
        System.out.println(stringThreadLocal.get());
       
    }

    static StringThreadLocal stringThreadLocal = new StringThreadLocal();

    static class StringThreadLocal extends ThreadLocal {
        @Override
        protected Object initialValue() {
            return "赖晓星";
        }
    }

```
输出
```
赖晓星

```




接下来看remove()
```
 public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```

如果当前线程的threadLocals变量不为空，则删除当前线程中指定ThreadLocal实例的本地变量。

总结：
每个线程内部都有一个名为threadLocals的成员变量，该变量的类型为HashMao，其中key为我们定义的ThreadLocal变量的this引用，value则为我们使用set方法设置的值，每个线程的本地变量存放在线程自己的内存变量的threadLocals中，此使用完毕后记得调用ThreadLocal的remove方法删除对于线程的threadLocals中的本地变量。



##### ThreaLocal和InheritalblThreadLoal的用法，子线程获取父线程的本地变量
ThreaLocal中设置的变量，在子线程中无法获取

```
public class ThreadLocalExtendTest {


    //创建线程变量
    public  static ThreadLocal<String> threadLocal=new ThreadLocal<>();

    public static void main(String[] args) {
        threadLocal.set("hello World");

        new Thread(()->{
            System.out.println("thread:"+threadLocal.get());
        }).start();

        System.out.println("main"+threadLocal.get());
    }

```


输出
```
mainhello World
thread:null
```


同一个ThreadLocal变量在父线程中被设置值，然后在子线程中是获取不到的。因为在子线程thread里面调用get方法时当前线程为thread线程，而这里调用set方法设置线程变量的是main线程，两者是不同的线程，值是放在了两个不同的Thread实例的threadlocals变量里面。


为了解决这个问题，InheritableThreadLocal继承自ThreadLocal并提供了一个特性，就是让子线程可以访问父线程中设置的变量。

比如说
```
public class InheritableThreadLocalTest {


    //创建线程变量
    public  static InheritableThreadLocal<String> threadLocal=new InheritableThreadLocal<>();

    public static void main(String[] args) {
        threadLocal.set("hello World");

        new Thread(()->{
            System.out.println("thread:"+threadLocal.get());
        }).start();

        System.out.println("main"+threadLocal.get());

    }

```

输出
```
mainhello World
thread:hello World
```
在main线程里面设置了threadLocal的值为hello World，在子线程里面用这个threadLocal也照样获取到了。


其内部代码实现如下：
```
public class InheritableThreadLocal<T> extends ThreadLocal<T> {
 
    protected T childValue(T parentValue) {
        return parentValue;
    }

    /**
     * Get the map associated with a ThreadLocal.
     *
     * @param t the current thread
     */
    ThreadLocalMap getMap(Thread t) {
       return t.inheritableThreadLocals;
    }

    /**
     * Create the map associated with a ThreadLocal.
     *
     * @param t the current thread
     * @param firstValue value for the initial entry of the table.
     */
    void createMap(Thread t, T firstValue) {
        t.inheritableThreadLocals = new ThreadLocalMap(this, firstValue);
    }
}

```

由如上代码可知，InheritableThreadLocal继承了ThreadLocal，并重写了三个方法。 
重写了createMap方法方法，这样在第一次调用set方法时，创建的是当前线程的inheritableThreadLocals变量的实例而不是threadLocals。

重写了getMap方法，调用get的时候方法获取线程内部的map变量时，获取的是inHeritableThreadLocals而不再是threadLocals

因从，在InheritalblThreadLoal里面 变量inheritableThreadLocas替代了threadLocals。



那么inheritableThreadLocas是如何实现让子线程能访问到父线程的本地变量的呢？

要从线程创建的时候开始说起，Thread在实例化的时候，会调用初始化函数
```
  public Thread(Runnable target) {
        init(null, target, "Thread-" + nextThreadNum(), 0);
    }


//init方法里面
 private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize, AccessControlContext acc,
                      boolean inheritThreadLocals) {
    //..省略之前无关的
        Thread parent = currentThread();
    //..省略中间无关的
        if (inheritThreadLocals && parent.inheritableThreadLocals != null)
            this.inheritableThreadLocals =
                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
  //..
 
    }

```

当主线程在创建子线程的时候，会调用init方法，在这个方法里面，会获取当前线程，
并将当前线程的inheritableThreadLocals变量赋予子线程 。

createInheritedMap(parent.inheritableThreadLocals)
```
static ThreadLocalMap createInheritedMap(ThreadLocalMap parentMap) {
        return new ThreadLocalMap(parentMap);
    }

//其内部如下，对父线程的inheritableThreadLocals进行了一份拷贝
 private ThreadLocalMap(ThreadLocalMap parentMap) {
            Entry[] parentTable = parentMap.table;
            int len = parentTable.length;
            setThreshold(len);
            table = new Entry[len];

            for (int j = 0; j < len; j++) {
                Entry e = parentTable[j];
                if (e != null) {
                    @SuppressWarnings("unchecked")
                    ThreadLocal<Object> key = (ThreadLocal<Object>) e.get();
                    if (key != null) {
                        Object value = key.childValue(e.value);
                        Entry c = new Entry(key, value);
                        int h = key.threadLocalHashCode & (len - 1);
                        while (table[h] != null)
                            h = nextIndex(h, len);
                        table[h] = c;
                        size++;
                    }
                }
            }
        }
```

总结:
InheritableThreadLocal类继承自ThreadLocal类，并通过重写方法getMap(Thread t) 和重写方法createMap(Thread t,T fistValue) 使得让本地变量获取和保存到了该线程的inheritableLocals变量里面，因此线程在通过InheritableThreadLocal类实例的set或者get方法设置变量时，就会创建当前线程的inheritableThreadLocals变量，当父线程创建子线程时，构造函数会把父线程中inheritableThreadLocals变量里面的本地变量复制一份保存到子线程的inheritableThreadLocals变量里面。 

那么在什么情况下需要子线程可以获取父线程threadLocal变量呢？比如说子线程需要使用存放在ThreadLocal变量里面的用户登陆信息，再比如一些中间件需要把统一的id追踪到整个调用链路记录下来。其实子线程使用父线程的threadLocal方法有许多种方式，比如创建线程时传入父线程中的变量，并将其复制到子线程中，或者在父线程中构造一个map作为参数传递给子线程，但是最简单的方式就是使用InheritableThreadLocal。




思路总结：
每个Thread类里面都有两个属性threadLocals和InheritableThreadLocal，类型为ThreadLocalMap， key为ThreadLocal实例对象，value为T>。 
在设置线程本地变量的时候，ThreadLocal实例的set方法，实际上是将要保存的本地变量放到了当前线程对象的threadLocals属性里面，放的时候key是这个ThreadLocal实例，value是要放的值。
获取的时候根据这个ThreadLocal实例作为key去这个线程的ThreadLocals里面取。

InheritableThreadLocals则是在Thread对象创建初始化的时候，从创建这个Thread对象的线程里面取到当前这个线程的InheritableThreadLocals属性的值并复制给新的Thread对象的这个属性。这样子线程就可以读到和主线程的InheritableThreadLocals属性值一样的值了

InheritableThreadLocal继承了ThreadLocal类，重写了getMap和creatMap方法，以至于让get和set都是对这个Thread对象的InheritableThreadLocals属性进行操作。

#####  1.什么是多线程并发编程

首先要澄清并发和并行的概念，并发和指同一个时间段内多个任务同时都在执行，并且都没有执行结束。而并行是在单位时间内多个任务同时在执行。
并发任务强调在一个时间段内同时执行，而一个时间段由多个单位时间累积而成，所以说并发的多个任务在单位时间内不一定同时执行。
在单cpu的时代多个任务都是并发执行，因为单个cpu同时只能执行一个任务。
在单个cpu时代多任务是共享一个cpu的，当一个任务占用cpu运行时，其他任务就会被挂起，当占用cpu的任务时间片用完之后，会把cpu让给其他任务来使用。


##### 2.为什么要进行多线程并发编程

多核cpu时代的到来打破了单核cpu对于多线程效能的限制，多个cpu意味着每个线程可以使用自己的cpu运行，这减少了线程上下文切换的开销，但随着对应用系统性能和吞吐量要求的提高，出现了处理海量数据和请求的要求，这些都对高并发编程有着迫切的需求。


##### 3.java中线程安全问题
什么是共享资源？
该资源被多个线程所持有或者多个线程都可以去访问该资源
线程安全问题是指当多个线程同时读写一个共享资源并且没有任何同步措施时，导致出现脏数据或者其他不可预见的结果的问题。
线程安全问题和共享资源之间的关系？

并不是多个线程都去访问了共享资源就会出现线程安全问题，比如说所有线程都是去读。 

至少有一个线程去对共享资源进行了修改才会存在线程安全问题。比如说高并发下的计数器，如果不同步，就会统计不准确。

count++;
分为三步：
1.将count从内存读取到本线程的工作内存
2.在工作内存进行递增
3.写回到主内存

以上三步，如果同时有多个线程在进行，势必会让最终的count++的数据不准确。比如说线程A 执行了12然后切换到线程B 执行了123 然后切换回线程A执行了3， 此时的值其实是等于线程A递增的值为1次，实际上应该是递增了2次。

##### 4.java中共享内存可见性问题
java内存模型

将所有的变量都存放到主内存中，当线程使用的变量时，会将主内存里面的变量复制到自己的工作空间或者叫工作内存。
线程读写变量操作的是自己工作内存中的变量。

当一个线程操作共享变量时，首先从主内存复制共享变量到自己的工作内存，然后对工作内存里面的变量进行处理，处理完后将变量值更新到主内存。

对应的物理内存

假设有一个双核cpu系统。 每个核都有自己的控制器和运算器，其中控制器包含一组寄存器和操作控制器，运算器执行算术逻辑运算。

每个核都有自己的一级缓存，在有些架构里面还有一个所有cpu都共享的二级缓存，java内存模型里面的工作内存就对应这里的L1或者L2缓存或者cpu的寄存器。

因为cpu的核都各自的缓存，假设两个线程分别执行在不同的核上，将内存上要用的数据同步到使用各自的缓存，然后对各自缓存内数据更改，对其他核的缓存数据是没有影响的，也无法感知到其他核的缓存数据变更。

解决方式：使用一个值的时候，从主存读取最新的，对该值进行更改之后，立即写入到主存中。

java中的volatile可以达到这种效果

synchronized也可以达到这种效果，因为线程进入同步块的时候，会清空本地缓存并从主内读取最新的。然后在退出同步块的时候又会将工作内存里面的内容都写入到主存中。

##### 5.java中的synchronized关键字
5.1 synchronized关键字介绍
synchronized块是java提供的一种原子性内置锁，java中每个对象都可以把它当作一个同步锁使用，这些java内置的使用者看不到的锁叫做内部锁，也叫监视器锁。
线程的执行代码在进入synchronized代码块前会自动获取内部锁，这时候其他线程访问该同步代码块时就会阻塞挂起。拿到内部锁的线程会在正常退出同步代码块或者抛出异常后或者在同步块内调用了该内置锁资源的wait系列方法时释放该内置锁。内置锁是排他锁，也就是当一个线程获取这个锁后，其他线程必须等待该线程释放锁后才能获取该锁。
内部锁的标志在对象头上

java中的线程是与操作系统的原生线程一一对应的，所以当阻塞一个线程时，需要从用户态切换到内核态执行阻塞操作，这是很耗时的操作，而synchronized的使用会导致上下文切换，因为获取不到就要进行阻塞状态。 

5.2 synchronized内存语义
进入synchronized块的内存语义是把synchronized块内使用到的变量从线程的工作内存中清除，这样在synchronized块内存中使用到该变量时候，就会去主内存中获取，而不是使用本地工作内的值。
退出synchronized块的内存语义是把synchronized块内对共享变量的修改刷新到主内存。
获取锁后会清空锁块本地内存中将会被用到的共享变量，在使用这些共享变时从主内存加载，在释放锁的时候从本地内存修改的共享变量刷新到主内存。
synchronized除了可以解决内存可见性问题之外，还经常被用来实现原子操作，但是要注意，synchronized关键字会引起上下文切换并带来线程调度的开销

##### 6.java中volatitle关键字
除了使用锁的方式可以解决共享变量内存可见性问题，但是使用太笨重，因为它给予带来线程上下文切换开销。对于解决内存可见性问题，java还提供了一种弱形式的同步，也就是使用vlolatitle关键字。
该关键字可以确保对一个变量的更新对其他线程马上可见。
当一个变量被声明为volatitle时，线程在写入变量时不会把值缓存在寄存器或者其他地方，而是会把值刷新回主存。当其他线程读取该共享变量时，会从主内存重新获取最新值，而不是使用当前线程的工作内存中的值。

volatitle的内存语义和synchronized有相似之处，具体来说，就是当线程写入了volatitle变量时就等价于线程退出synchronized同步块（将写入工作内存的变量值同步到主内存),读取volatile变量值时就相当于进入同步块（先清空本地内存变量值，再从主存获取最新值）

volatile提供了内存可见性的保证，但并不保证操作的原子性，因此不能替代锁

什么时候使用volatile？
写入变量值不依赖变量当前值时。 因为如果依赖当前值，将是1.获取 2.计算，3.写入，三步操作，这三步操作不是原子性的，而volatile不保证原子性

也就是，即使有volatile 修饰int类型的count，在并发修改的时候，也可能会出现问题。
比如说线程1执行了12 切换到线程2执行了123，然后切换线程1执行了3.


读写变量值时没有加锁，因为加锁本身已经保证了内存可见性，这时候不需要把变量声明为volatitle

##### 7.java中的原子性操作
什么是原子操作？
所谓原子操作，是指执行一系列操作时，这些操作要么全部执行，要么全部不执行。不存在只只执行其中一部分的清空。
比如说在设计计数器时一般都先读取当前值，然后+1，再更新。这个过程是读-改-写的过程，如果不能保证这个过程是原子性的，那么就会出现线程安全问题。
比如说在设计计数器时一般都先读取当前值，然后+1，再更新。这个过程是读-改-写的过程，如果不能保证这个过程是原子性的，那么就会出现线程安全问题。
1.获取当前值，2,将常量1放到栈顶，3.将当前栈顶的两个值相加并放回到栈顶 4.将栈顶的结果赋予value变量

如何保证原子性
使用synchronzied关键字可以实现线程安全性，即内存可见性和原子性，但是synchronized是独占锁，没有获取内部锁的线程会被阻塞调。
public synchronized void inc(){
++value;
}

##### 8.java中的CAS操作
为什么要用CAS?
在java中，锁在并发处理中占据了一席之地，但是使用锁有一个不好的地方，就是当一个线程没有获取到锁时会被阻塞挂起，这回导致线程的上下文切换和重新调度开销。java提供了非阻塞的volatile关键字来解决共享内存的可见性问题，这在一定程度上弥补了锁带来的开销问题，但是volatitle只能保证共享变量的可见性，不能解决读-改-写的原子性问题。

CAS即compare and swap，是jdk提供的非阻塞原子性操作，它通过硬件保证了比较-更新的原子性。jdk里面的unsafe类提供了一系列的compareAndSwap方法。

以compareAndSwapLong方法为例
```
boolean compareAndSwapLong(Object obj,long valueOffset,long expect,long update)
```
其中compareAndSwap的意思是比较并交换。CAS有四个操作数，分别为：对象内存位置，对象中变量的偏移量，变量预期值和新的值。其操作含义是，如果对象obj中内存偏移量valueoffset的变量值为expect,则使用新的值update替换旧的值。 这是处理器提供的一个原子性指令

ABA问题

cas操作有个经典的ABA问题，具体来说：假如线程1使用CAS修改初始值为A的变量X，那么线程1会首先去获取当前变量X的值（为A），然后用CAS操作尝试修改X的值为B,如果CAS操作成功了，那么程序运行一定是正确的吗？其实未必，这是因为有可能在线程1获取到变量X的值A后，在执行CAS前，线程2使用CAS修改了变量X的值为B，然后又使用CAS修改了变量X的值为A。 所以虽然线程1执行CAS时候X的值为A但是这个A已经不是线程1获取时的A了，这就是ABA问题

ABA问题的产生是因为变量的状态值发生了环形转换，就是变量的值可以从A到B,然后从B到A，如果变量的值只能朝一个方向转换，比如A到B,B到C，不构成环，就不会存在问题，JDK中的AtomicStampedReference类给每个变量的状态值都配备了一个时间戳，从而避免了ABA问题的产生
```
public class AtomicStampedReference<V> {

    private static class Pair<T> {
        final T reference;
        final int stamp;
        private Pair(T reference, int stamp) {
            this.reference = reference;
            this.stamp = stamp;
        }
        static <T> Pair<T> of(T reference, int stamp) {
            return new Pair<T>(reference, stamp);
        }
    }

    private volatile Pair<V> pair;
```

假设变量X的初始值为A,时间戳为1，然后线程1通过cas获取到了该值，打算更新为B。 此时线程2也通过cas操作获取到了该值，然后更新为D，然后又更新回了A，此时变量X的值为A,但是时间戳已经变化了，为2。 然后此时线程1打算更新了，就会发现自己的A和预期的A不相同，因此更新失败，必须重新获取过A的值，进行更新。


##### 9.Unsafe类

Unsafe类中的重要方法
jdk的rt.jar包中的Unsafe类提供了硬件级别的原子性操作，Unsafe类中的方法都是native方法，它们用JNI的方式访问本地c++实现库。

```long objectFiedOffset(Field field)```方法
返回指定的变量在所属类中的内存偏移地址（即内存地址），该偏移地址仅仅在该Unsafe函数中访问指定字段时候使用。
例如:
```vavlueOffset=unsafe.objectFieldOffset(AtomicLong.class.getDeclaredFied("value"))```


```int arrayBaseOffset(Class arrayClass)```
获取数组第一个元素的地址

```int arrayIndexScale(Class arrayClass)```
获取数组中一个元素占用的字节

```boolean compareAndSwapLong(Object obj,long offset,long expec,long update)```
比较对象obj中偏移量为offset的变量的值是否与expect相等，相等则使用update值更新，然后返回true，否则返回false



```long getAndSetLong(Object obj,long offset,long update)```

获取对象obj中偏移量为offset的变量volatitle语义的当前值，并设置变量volatitle语义的值为update
```
public final long getAndSetLong(Object obj, long offset, long update){

long l;
do{
l=getLongvolatitle(obj,offset);
}while(!compareAndSwapLong(obj,offset,l,update));

return l;
}
```

内部是通过getLongvolatitle获取当前变量的值，然后使用cas原子操作设置新值。使用while循环是考虑到，多个线程同时调用的时候cas失败需要重试




long getAndAddLong(Object obj,long offset,long addValue) 

获取对象obj中偏移量为offset的变量volatitle语义的当前值，并设置变量值为原始值+addValue

类似getAndSetLong的实现，只是这里进行CAS操作时使用了原始值+传递的增量参数addValue的值


```
public final long getAndAddLong(Object obj,long offset,long addValue){
long l;
do{
l=getLongvolatitle(obj,offset);
} while(!compareAndSwapLong(obj,offset,l,l+addValue))
return l;
}
}
```





##### Unsafe如何使用

```
public class TestUnSafe {

    /**
     * 获取Unsafe的实例
     */
    static final Unsafe unsafe = Unsafe.getUnsafe();

    /**
     * 记录变量state在类TestUnSafe中的偏移量
     */
    static final long stateOffset;

    //变量
    private volatile long state = 0;


    static {

        try {
            //获取state变量在类TestUnSafe中的偏移值
            stateOffset = unsafe.objectFieldOffset(TestUnSafe.class.getDeclaredField("state"));
        } catch (Exception ex) {
            System.out.println(ex.getLocalizedMessage());
            throw new Error(ex);
        }


    }


    public static void main(String[] args) {
        //创建实例，并且设置state值为1
        TestUnSafe test=new TestUnSafe();
        Boolean sucees=unsafe.compareAndSwapInt(test,stateOffset,0,1);
        System.out.println(sucees);
    }


}


```


获取unsafe实例，先使用，获取到偏移量stateOffset
stateOffset = unsafe.objectFieldOffset(TestUnSafe.class.getDeclaredField("state"));
然后使用compareAndSwapInt  修改state的值
Boolean sucees=unsafe.compareAndSwapInt(test,stateOffset,0,1);


结果抛异常:
Exception in thread "main" java.lang.ExceptionInInitializerError
Caused by: java.lang.SecurityException: Unsafe
	at sun.misc.Unsafe.getUnsafe(Unsafe.java:90)
	at TestUnSafe.<clinit>(TestUnSafe.java:15)





看下这个Unsafe的报错信息里面提到的方法Unsafe.getUnsafe()

```

 @CallerSensitive
    public static Unsafe getUnsafe() {
    
       //获取调用getUnsafe这个方法的对象的Class对象，这里是TestUnSafe.class
        Class var0 = Reflection.getCallerClass();
        
        //判断是不是Boostrap类加载器的localClass，这里是看是不是Bootstrap加载器了TestUnSafe.class
        //很明显TestUnSafe.class是AppClassLocader加载的，所以这里直接抛出了异常 
        if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
            throw new SecurityException("Unsafe");
        } else {
            return theUnsafe;
        }
    }
```


因为Unsafe类是rt.jar包提供的，rt.jar 包里面的类是使用Bootstrap类加载的，而我们的启动main函数所在的类是使用AppClassLoader加载的，所以在main函数里面加载Unsafe类时，根据委托机制，给Bootstrap去加载Unsafe类， 然后这里限制了 类加载器必须是根加载器,不然就报错了。 

为什么呢？因为Unsafe类可以直接操作内存，这是不安全的，所以特意做了这个限制，而是在rt.jar包里面核心类使用Unsafe 功能。

那么只能反射来获取Unsafe实例方法。

先看getUnsafe方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190726094746451.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)
返回的是theUnsafe字段, 该字段是在static块里面进行实例化的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190726094838138.png)


因此 ，Usafe.getUnsafe()方法用不了的话，可以通过反射去获取Usafe里面的theUnsafe这个字段的值
```
 /**
             * 通过发射获取unsafe的实例
              */
            Field field=Unsafe.class.getDeclaredField("theUnsafe");
            field.setAccessible(true);

            unsafe=(Unsafe)field.get(null);
```

获取到了unsafe的实例 就可以使用unsafe的方法了。





##### java指令重排序
java内存模型允许编译器和处理器对指令重排序以提高性能，并且只会对不存在数据依赖性的指令重排序。在单线程下重排序可以保证最终执行的结果与程序顺序执行的结果一致，但是在多线程下就会存在问题。

例如：
int a=1;(1)
int b=2;(2)
int c=a+b;(3)

在如上代码中，变量c的值依赖变量a和b的值，所以重排序只会能够保证（3)的操作在（2）和（1)之后，但是1和2谁先执行就不一定了，这在单线程下不会存在问题，因为不会影响最终结果


通过volatitle修饰变量可以避免重排序和内存可见性问题：

写volatitle变量时，可以确保volatitle写之前的操作不会被编译器重排序到volatitle写之后，读volaititle变量时，可以确保volatitle读之后的操作不会被编译器重排序到volatitle读之前。


##### 伪共享
什么是伪共享

为了解决计算机系统中主内存与cpu之间的运行速度差问题，会在cpu与主内存之间添加一级或者多级告诉缓冲存储器Cache,这个cache一般被集成到cpu内部，也叫cpuCache ，有两级Cache结构和三级Cache结构等

在Cache内部是按行存储的，其中每一行称为一个Cache行。Cache行是Cache与主内存进行数据交换的单位，Cache行的大小一般数2的幂次数字节  

当cpu访问某个变量时，首先会去看cpu cache里面是否有该变量，如果有则直接从里面获取，否则就去主内存里面获取该变量，然后把该变量所在内存区域的一个Cache行大小的内存复制到Cache中，由于存放到cache行的是内存块而不是单个变量，所以可能会把多个变量存放到一个cache行中，当多个线程同时修改一个缓存行里面的多个变量时，由于只能有一个线程操作缓冲行，所以相比将每个变量放到一个缓存行，性能会有所下降，这就是伪共享。


比如说电脑有cpu1和cpu2, 变量x被放到了cpu1和cpu2的一级缓存行和二级缓存行里面，如果线程1使用cpu1对变量x进行更新，首先会修改一级缓存变量x所在的缓存行，这时在缓存一致性协议下，cpu2中变量对应的缓存行失效，那么线程2在写入变量x的时候只能去二级缓存中查找，而一级缓存要比二级缓存更快，因此性能下降了。 同时，也说明了多个线程不可能同时去修改字节所使用的cpu中相同缓存行里面的变量。更坏的情况是，cpu只有一级缓存，则会导致频繁的访问主内存


在多线程下并发修改一个缓存行中的多个变量时会竞争缓存行，从而导致性能降低


为何会出现伪共享

伪共享的产生是因为多个变量被放入了一个缓存行中，并且多个线程同时去写入缓存行中不同的变量，那么为何多个变量会被放入一个缓存行呢？因为缓存与内存交换数据的单位就是缓存行，当cpu要访问的变量没有在缓存中找到时候，根据程序运行的局部性原理，会把该变量在内存大小为缓存行的内存放入缓存行。

long a;
long b;
long c;
long d;

如上代码声明了四个long变量，假设缓存行中的大小为32字节，那么当cpu访问变量a时，发现该变量没有在缓存中，就会去主内存把变量a以及内存附近的bcd放入缓存行，也就是地址连续的多个变量才有可能被放入的一个缓存行中。当创建数组时，数组里面多个元素就会被放入同一个缓存行。那么在单线程下多个变量被放入同一个缓存行对代码执行是有利的，因为数据都在缓存中，代码执行会更快


```
public class TestForContent {

    static final int LINE_NUM=1024;
    static final int COLUM_NUM=1024;

    public static void main(String[] args) {
        long [][] array=new long[LINE_NUM][COLUM_NUM];

        long startTime=System.currentTimeMillis();
        for (int i = 0; i <LINE_NUM ; i++) {
            for (int j = 0; j < COLUM_NUM; j++) {
                array[j][i]=i*2+j;
            }
        }

        long endTime=System.currentTimeMillis();

        long cacheTime=endTime-startTime;
        System.out.println("cache time:"+cacheTime);

    }


}


```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190727102934745.png)


```
public class TestForContent {

    static final int LINE_NUM=1024;
    static final int COLUM_NUM=1024;

    public static void main(String[] args) {
        long [][] array=new long[LINE_NUM][COLUM_NUM];

        long startTime=System.currentTimeMillis();
        for (int i = 0; i <LINE_NUM ; i++) {
            for (int j = 0; j < COLUM_NUM; j++) {
                array[i][j]=i*2+j;
            }
        }

        long endTime=System.currentTimeMillis();

        long cacheTime=endTime-startTime;
        System.out.println("cache time:"+cacheTime);

    }


}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190727103004978.png)



为什么这个代码执行的比上面的代码快？
因为数组元素的内存地址是连续的，当访问数组第一个元素时，会把第一个元素后面的若干个元素一起放到缓存行，这样顺序访问数组元素时候会在缓存直接命中，就不会去主内存读了。也就是一次内存访问可以让后续多个访问都命中缓存。

而代码2是跳跃性访问数组的，不是顺序的，因此下一次访问的并不是加载到缓存里的，而且缓存是有容量控制的，如果满了会根据一定的淘汰算法替换缓存行，这会导致从内存置换过来的缓存行元素还没等到被读取就被替换掉了。


如何避免伪共享

在jdk8之前一般都是通过字节填充的方式来避免该问题，也就是创建一个变量的时候，用填充字段填充该变量所在的缓存行，这样就避免了将多个变量存放在同一个缓存行中。

例如：
public final static class FilledLong{
public volatile long value=0L;
public long p1,p2,p3,p4,p5,p6;

}


假如缓存行为64个字节，那么在FiledLong类里面填充了6个long类型的变量，每个long类型的变量占用8字节，加上vlaue变量的8字节 总共56个字节，另外FiledLong是一个类对象，而类对象的字节码对象头是8个字节，这样一个FiledLong对象实际占用64个字节，这刚好可以放下一个缓存行

jdk8提供了一个sun.misc.Comtended注解，用来解决伪共享的问题，将上面代码修改为

@sun.misc.Comtended
public final static class FilledLong{
public volatile long value=0L;

}

这个注解也可以用来修饰变量，比如说在Thread类里面就有：
@sun.misc.Comtended("tlr")
int threadLocalRandomSeed


默认情况下
@sum.misc.Comtend注解只用于java核心类，比如rt下的类，如果用户类路径下想用，需要加jvm参数: -XX:-RestrictContended

填充宽度默认为128
如果要自定义宽度，则设置参数 -XX:ComtendPaddingWidth




##### 锁的概述
1.乐观锁与悲观锁

悲观锁

悲观锁指对数据被外界修改持保守态度，认为数据很容易就会被其他线程修改，所以在数据被处理前先加锁，并在整个数据处理过程中，使数据处于锁定状态。悲观锁的实现往往要靠数据库提供的锁机制，即在数据库中，在对数据记录操作前给记录加排他锁，如果获取锁失败，则说明数据正在被其他线程修改，当前线程则等待或者抛出异常。如果获取成功，就对记录进行操作，然后提交事务后释放排他锁

```

悲观锁需要在查询的时候加上for update 锁定这一行记录

例如：
public int updateEntry(long id){


//for update 语句，悲观锁获取指定记录
EntryObject entry=query("select *from table1 where id=#{id} for update",id);

//修改记录内容
...

//update操作
int count=update("update table set name=#{name},age=#{age} where id=#{id}")

....

}
```


for update会锁定一行记录,
当整个方法执行完毕的提交事务时，query方法才会被提前，也就是记录的锁定会持续到整个方法结束。

当多个线程同时调用这个方法，且传入的是同一个id时，只有一个线程执行代码for update会成功，其他线程会被阻塞，因为同一时间只有一个线程可以获取到对应记录的锁，在获取锁的线程释放锁之前，其他线程必须等待，也就是同一时间只有一个线程可以对记录进行修改


这里是假设了updateEntry以及query和update都开启了事务且传播级别是required.
required级别下，如果上层方法有事务，那么被调用的方法会使用上层方法的事务。 因此，这里updateEntry，query,update 使用了updateEntry这个方法的事务，也就是要等到整个方法执行完毕才会提交事务，因此for update会一直锁到整个方法结束


补充：
若 ServiceB.methodB() 的传播行为定义为 PROPAGATION_REQUIRED , 那么在执行 ServiceA.methodA() 的时候，若 ServiceA.methodA() 已经开启了事务，这时调用 ServiceB.methodB()，ServiceB.methodB() 将会运行在 ServiceA.methodA() 的事务内部，而不再开启新的事务。而假如 ServiceA.methodA() 运行的时候发现自己没有在事务中，就会为它分配一个新事务。这样，在 ServiceA.methodA() 或者在 ServiceB.methodB() 内的任何地方出现异常，事务都会被回滚。即使 ServiceB.methodB() 的事务已经被
提交，但是 ServiceA.methodA() 在接下来的过程中 fail 要回滚，



乐观锁

乐观锁是相对悲观锁来说的，它认为数据在一般情况下不会造成冲突，所以在访问记录前不会加排他锁，而是在数据更新的时候，才会对数据冲突与否进行检测，具体来说就是通过判断update返回的行数让用户决定如何去做

```
乐观锁需要在表中维护一个vesion字段，用于检测冲突。

public int updateEntry(long id){


//for update 语句，悲观锁获取指定记录
EntryObject entry=query("select *from table1 where id=#{id} ",id);

//修改记录内容
...

//update操作
int count=update("update table set name=#{name},age=#{age},vesion=#{vesion}+1  where id=#{id} and vesion=#{vesion}")

....

}
```

这里通过update语句返回的返回值，就可以知道是否已经更新成功，如果count为0，说明有冲突，数据已经被别人改掉了，那么可以执行重试，或者放弃这次操作。 乐观锁不会使用数据库提供的锁机制，一般在表中添加vesion字段或者使用业务状态来实现，乐观锁不会直到提交时才锁定，所以不会产生任何死锁


公平锁与非公平锁

根据线程获取锁的抢占机制，锁可以分为公平锁和非公平锁，公平锁表示线程获取锁的顺序是按照线程请求锁的时间早晚来决定的，也就是最早请求锁的线程将最早获取到锁。而非公平锁则是在运行时传入，也就是先来不一定先得。


公平锁：
ReentrantLock  lock=new ReentrantLock(true);
非公平锁
ReentrantLock  lock=new ReentrantLock(false);

假设线程A已经持有了锁，这时候线程B请求该锁其将会被挂起，当线程A释放锁后，假如当前有线程C也需要获取该锁，如果采用非公平锁方式，则根据线程调度策略，线程B和C两者都有可能获取锁，但是如果使用公平锁，则需要把c挂起，让b获取锁

没有公平性需求的情况下，尽量使用非公平锁，因为公平锁会带来性能开销

独占锁与共享锁

根据锁只能被单个线程持有还是能被多个线程共同持有，锁可以分为独占锁和共享锁。
独占锁保证任何时候都只有一个线程能得到锁，ReentrantLocak就是以独占方式实现的。共享锁则可以同时由多个线程持有，比如说ReadWriteLock读写锁，它允许一个资源可以被多线程同时进行读操作。

独占锁是一种悲观锁，由于每次访问资源都先加上互斥锁，这限制了并发性，因为读操作并不会影响数据的一致性，而独占锁只允许同一时间由一个线程读取数据，其他线程必须等待当前线程释放锁才能进行读取。

共享锁则是一种乐观锁，它放宽了加锁的条件，可以多个线程进行读操作


什么是可重入锁

当一个线程要获取一个被其他线程持有的独占锁时，该线程会被阻塞，那么当一个线程再次获取它自己已经获取的锁时，如果不阻塞，我们就可以说该锁是可重入的。

```

public  class hello{

public synchronized void helloA(){
System.out.println("hello A ");

}

public synchronized void helloB(){
System.out.println("hello B ");
helloA（）；
}


}
```

在进入helloB的时候，获取了该对象的监视器锁，如果该锁不能重入，那么在helloA（）； 线程就会因为监视器锁没释放而无法进入helloA，永远阻塞在这里。

事实上synchronized内部锁是可重入锁，可重入锁的原理是在锁内部维护一个线程标示，标示该锁目前被哪个线程占用，然后关联一个计算器，一开始计算器为0，标示没有任何线程占用，当一个线程获取了该锁，计数器值就会变成1。 这时当其他线程来获取该锁时，就会发现所有者不是自己就会被阻塞挂起。

但是当获取了该锁的线程再次获取锁时，发现锁拥有者是自己，就会把计数器值+1，当释放锁之后，计数器值-1，当计数器值为0时，锁里面的线程标示被重置为null，这时候被阻塞的线程就会被唤醒来竞争获取该锁。


自旋锁

由于java中的线程是与操作系统中的线程一一对应的，所以当一个线程在获取锁（如独占锁）失败之后，会被切换到内核状态而被挂起，当该线程获取到锁时又需要再次切换，然后从用户状态切换到内核状态的开销是比较大的，在一定程度上影响并发性能。自旋锁则是，当前线程在获取锁时，如果发现锁已经被其他线程占有，它不马上阻塞自己，在不放弃cpu的情况下，多次尝试，默认次数是10，可以使用-XX:PreBlockSpinsh参数设置该值， 很可能在后面的几次尝试中其他线程已经释放了锁，如果次数耗尽还没获取到锁才会被阻塞挂起。





##### java并发包中ThreadLocalRandom类原理剖析
Random类及其局限性

ThreadLocalRandom类是jdk7在juc包下新增的随机数生成器，它弥补了Random类在多线程下的缺陷。

jdk7之前包括现在，Random类都是使用比较广泛的随机数生成器，Math中随机数生成也是使用的Randmon实例

```
  public Random() {
        this(seedUniquifier() ^ System.nanoTime());
    }

    private static long seedUniquifier() {
        // L'Ecuyer, "Tables of Linear Congruential Generators of
        // Different Sizes and Good Lattice Structure", 1999
        for (;;) {
            long current = seedUniquifier.get();
            long next = current * 181783497276652981L;
            if (seedUniquifier.compareAndSet(current, next))
                return next;
        }
    }

    private static final AtomicLong seedUniquifier
        = new AtomicLong(8682522807148012L);

```

  //随机数生成需要一个默认的种子，是一个long类型的数字， 种子相同次每次随机生成的数组均相同
    //可以在构造函数中指定种子  Random random=new Random(1);
    //但是如果不指定，则使用默认的
    //默认种子一个原子常量和当前时间的纳秒值进行位运算 。
    //原子常量每使用一次都会更新 为静态常量，以确保全局不会出现重复的种子

```

        //创建一个默认种子的随机数生成器
        Random random=new Random();
        for (int i = 0; i <10 ; i++) {
            //输出10个在0-5之间的随机数 ，包含0 不包含5
            System.out.println(random.nextInt(5));
        }
```



```
 //有了默认种子之后 如何生成随机数
//    public int nextInt(int bound) {
//        if (bound <= 0)
//            throw new IllegalArgumentException(BadBound);
//
    //根据老的种子生成新的种子
//        int r = next(31);
//        int m = bound - 1;
    //根据新的种子计算新的随机数
//        if ((bound & m) == 0)  // i.e., bound is a power of 2
//            r = (int)((bound * (long)r) >> 31);
//        else {
//            for (int u = r;
//                 u - (r = u % bound) + m < 0;
//                 u = next(31))
//                ;
//        }
//        return r;
//    }

    //新的随机数生成需要两个步骤，首先根据老的种子生成新的种子，然后根据新的种子来计算新的随机数

    //在单线程下每次调用nextInt都是根据老的种子算出新的种子
//    protected int next(int bits) {
//        long oldseed, nextseed;
//        AtomicLong seed = this.seed;
//        do {
//            oldseed = seed.get();
//            nextseed = (oldseed * multiplier + addend) & mask;
//        } while (!seed.compareAndSet(oldseed, nextseed));
//        return (int)(nextseed >>> (48 - bits));
//    }


```


在多线程下，多个线程可能拿到同一个种子去计算新的种子，那么就会进入cas重试的阶段，因为要保证只有一个线程可以更新老的种子为新的，
   更新失败的线程需要获取到最新的种子再次计算，保证种子不重复才能保证随机数的随机性。

    每个Random实例里面都有一个原子性的种子变量用来记录当前的种子值，当要生成新的随机数时需要根据当前种子计算新的种子并更新回原子变量
   在多线程下使用单个Random实例生成随机数时，多个线程同时计算随机数来计算新的种子时，多个线程会竞争同一个原子变量的更新操作，由于原子变量的更新是cas操作，同时只有一个线程会成功，所以造成大量线程自旋重试，这会降低并发性能，于是ThreadLocalRandom应运而生。




ThreadLocalRandom


//创建一个默认种子的随机数生成器
        ThreadLocalRandom threadLocalRandom= ThreadLocalRandom.current();
        for (int i = 0; i <10 ; i++) {
            //输出10个在0-5之间的随机数 ，包含0 不包含5
            System.out.println(threadLocalRandom.nextInt(5));
        }


ThreadLocalRandom的实现原理，也是让每个线程都维护一个自己的种子变量，每个线程生成随机数时都根据自己老的种子计算新的种子，再根据新的种子计算随机数，不存在竞争问题。

Random的缺点就是多个线程使用一个random实例的时候，会使用同一个原子性种子变量，从而导致对原子变量更新的竞争



源码分析

ThreadLocalRandom继承自Random并重写了Random类的nextInt方法。
而且这个类并没有使用Random类的seed，在ThreadLocalRandom类中没有保存具体的种子，种子存放在Thread类的threadLocalRandomSeed变量里面，也就是当前线程的本地变量。
ThreadLocalRandom类似于ThreadLocal类，是个工具类，当调用ThreadLocalRandom.current方法时，ThreadLocalRandom负责初始化调用线程的threadLocalRandimseed变量，也就是初始化种子。

当调用ThreadLocalRandom的nextInt方法时，实际上是获取当前线程的threadLocalRandomSeed变量作为当前种子计算新的种子，并更新新的种子到当前线程的threadLocalRandomSeed变量里面，然后再根据新的种子计算新的随机数。
注意的是，这个种子是普通的long型，不是原子性的，因为作为线程本地变量，只有一个线程会使用，不会冲突。



unsafe机制

  static {
        try {

//获取unsafe实例
            UNSAFE = sun.misc.Unsafe.getUnsafe();
//获取Thread类里面的threadLocalRandomSeed变量在Thread实例里面的偏移量
            Class<?> tk = Thread.class;
            SEED = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSeed"));
//获取Thread类里面threadLocalRandomProbe变量在Thread实例里面的偏移量
            PROBE = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomProbe"));
//获取Thread类里面threadLocalRandomSecondarySeed变量在Thread实例里面的偏移量
            SECONDARY = UNSAFE.objectFieldOffset
                (tk.getDeclaredField("threadLocalRandomSecondarySeed"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }


ThreadLocalRandom current()方法

该方法获取ThreadLocalRandom实例，并初始化调用线程中的threadLocalRandomSeed和threadLocalRandomProbe变量


```
   /** The common ThreadLocalRandom */
    static final ThreadLocalRandom instance = new ThreadLocalRandom();


public static ThreadLocalRandom current() {
        if (UNSAFE.getInt(Thread.currentThread(), PROBE) == 0)
            localInit();
        return instance; 
}




 static final void localInit() {
        int p = probeGenerator.addAndGet(PROBE_INCREMENT);
        int probe = (p == 0) ? 1 : p; // skip 0
        long seed = mix64(seeder.getAndAdd(SEEDER_INCREMENT));
        Thread t = Thread.currentThread();
        UNSAFE.putLong(t, SEED, seed);
        UNSAFE.putInt(t, PROBE, probe);
    }
    }



```

ThreadLocalRandom使用ThreadLocal的原理，让每个线程都持有一个本地的种子变量，该种子变量只有在使用随机数时才会被初始化，在多线程下计算新种子时是根据自己线程内维护种子变量进行更新，从而避免了竞争。 



###### LockSupport工具类学习
jdk的rt.jar包里面的LockSupport是个工具类，主要作用了挂起和唤醒线程，该工具类是创建锁和其他同步类的几次。

调用park()系列的方法可以使线程进入阻塞状态，调用unpark方法可以唤醒。
功能类似wait和notify，但是不需要像wait和notify那样必须在synchronized的范围内使用
且如果先调用了unpark再调用park，此线程是不会进入阻塞状态的,原因是调用unpark(Thread)的时候，已经给了该线程一个许可证，park的原理是没有许可证的进入阻塞状态。
但是如果先调用notify 再调用wait 线程依然会进入阻塞。


内部是使用了unsafe类实现。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810163854285.png)

unsafe的park是个native方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810163918672.png)


使用例子
```

public class LockSupportDemo {

    public  void testPark(){
        LockSupport.park();
    }


    public static void main(String[] args) {
        LockSupportDemo lockSupportDemo=new LockSupportDemo();
        lockSupportDemo.testPark();

    }
}

```

随机发现线程挂起了

使用jps查看pid
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810164030440.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)

我的程序名是LockSupportDemo

用jstack查看此pid的状态

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810164245614.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)

可以看到此时main的线程状态为waiting


park方法还支持带有blocker参数的方法 void park(Object blocker) 
当线程调用park方法且进入到了阻塞状态时，这个blocker对象会被记录到该线程内部
此时使用诊断工具可以看到线程被阻塞的原因，是通过调用getBlocker(Thread)方法来获取blocker对象的。
因此推荐使用带有blocker参数的park方法，并且blocker被设置为this,这样当在打印线程堆栈排查问题时就能知道是哪个类被阻塞了。


例如：
```

public class LockSupportDemo {

    public  void testPark(){
        LockSupport.park(this);
    }


    public static void main(String[] args) {
        LockSupportDemo lockSupportDemo=new LockSupportDemo();
        lockSupportDemo.testPark();

    }
   }

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810164940121.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)
跟上图相比，多了一行信息，parking to wait for  XXXX。


此方法的源码
setBlocker将blocker对象保存到了Thread实例对象里面
```
public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker);
        UNSAFE.park(false, 0L);
        setBlocker(t, null);
    }
```

保存是通过UNSAFE工具类直接设定的
```
private static void setBlocker(Thread t, Object arg) {
        // Even though volatile, hotspot doesn't need a write barrier here.
        UNSAFE.putObject(t, parkBlockerOffset, arg);
    }

```
具体保存的位置是Thread实例的volatile Object parkBlocker; 这个成员变量里面
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019081016582680.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)
也就是通过unsafe工具类直接操作内存将这个参数保存进了这个线程实例的parkBlocker成员变量里面。



其他带有时间的参数类型wait带有时间的参数方法，都是阻塞一段时间然后自动唤醒。
比如说void parkNanos (long nanos); 纳秒


###### 使用LocalSupport和队列设计一个先入先出的锁
使用LocalSupport和队列设计一个先入先出的锁

```
public class FIFOMutex {

/**
* 等待队列，只有队首的线程有资格获得锁
*/
    private final Queue<Thread> waiters = new ConcurrentLinkedQueue<Thread>();
    /**
     * 0 无锁  1 有锁
     */
    private volatile AtomicInteger lock = new AtomicInteger(0);

    public void lock() {
        //获取当前调用该方法的线程对象
        Thread current = Thread.currentThread();
        //将该线程加入到等待队列中
        waiters.add(current);

        //当获取到的队首线程不是当前线程  且将锁状态从无锁改到有锁失败 
        while (waiters.peek() != current || !lock.compareAndSet(0, 1)) {
            //将当前线程挂起
            LockSupport.park(this);
        }
    }


    public void unlock() {
        //获取当前调用该方法的线程对象
        Thread current = Thread.currentThread();
        //当前线程等于队列首线程
        if (waiters.peek()==current){
            //释放锁标志 将锁从有锁改为无锁
           lock.compareAndSet(1, 0);
              ///将队首线程从等待队列移除 
               waiters.remove();  
               //唤醒队首线程 
               LockSupport.unpark(waiters.peek());
        }
    }


    public static void main(String[] args) {

        FIFOMutex fifoMutex=new FIFOMutex();

        new Thread(() -> {
            //获取锁
            fifoMutex.lock();
            System.out.println("线程1获取到了锁");

            try {
                Thread.sleep(10000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            fifoMutex.unlock();
        }).start();


         new Thread(() -> {

             try {
                 Thread.sleep(1000L);
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }

             System.out.println("线程2尝试获取锁");
            fifoMutex.lock();
            System.out.println("线程2尝试获取锁成功");
            fifoMutex.unlock();
        }).start();



         new Thread(() -> {
            try {
                Thread.sleep(3000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程3直接unlock");
            fifoMutex.unlock();
            System.out.println("线程3直接unlock成功");
        }).start();

    }


}

```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810214832818.png)


测试里面有三个线程， 第一个线程调用lock方法， 判断是否是队首线程且修改锁状态成功，使用cas方法去修改锁状态，只有一个能修改成功，如果成功则代表成功获取到了锁 ，如果失败则调用park方法进入阻塞状态。 
然后休眠10秒， 期间第二个线程去尝试获取锁，进入阻塞，然后第三线程直接unlock但是由于它不是队首线程， 因此不能解锁。  然后第一个线程10秒后时间到了 ，调用unlock方法，先判断这个线程是不是队首线程，如果是，则修改锁状态无锁且将该线程从队首移出，然后唤醒新的队首线程。




如果将我自定义的锁换成ReentrantLock ，那么直接unlock会抛非法的监视器锁状态异常
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810221405390.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190810221358794.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzIwMDA5MDE1,size_16,color_FFFFFF,t_70)













