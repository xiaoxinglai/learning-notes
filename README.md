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





