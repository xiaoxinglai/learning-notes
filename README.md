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

>进入消费阻塞
生产：1562911018531:0
消费：1562911018531:0
生产：1562911019532:1
消费：1562911020535:1
生产：1562911020536:2
生产：1562911021541:3
消费：1562911022538:2
生产：1562911022542:4
生产：1562911023546:5
消费：1562911024540:3
生产：1562911024550:6
生产：1562911025553:7
消费：1562911026544:4
生产：1562911026556:8
生产：1562911027560:9
消费：1562911028548:5
生产：1562911028563:10
生产：1562911029567:11
消费：1562911030551:6
生产：1562911030568:12
生产：1562911031569:13
消费：1562911032554:7
生产：1562911032573:14
生产：1562911033574:15
消费：1562911034558:8
生产：1562911034577:16
生产：1562911035581:17
消费：1562911036563:9
生产：1562911036583:18
生产：1562911037586:19
消费：1562911038565:10
生产：1562911038589:20
进入生产阻塞
消费：1562911040567:11
生产：1562911040567:21