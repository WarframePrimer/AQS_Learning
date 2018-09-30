
    CountDownLatch: A synchronization aid that allows one or more threads to wait until a set of operations being performed in other   threads completes.
    CyclicBarrier : A synchronization aid that allows a set of threads to all wait for each other to reach a common barrier point.

CountDownLatch : 一个线程(或者多个)， 等待另外N个线程完成某个事情之后才能执行。   
CyclicBarrier  : N个线程相互等待，任何一个线程完成之前，所有的线程都必须等待。


```java
public class CountDownLatchDemo {

    private static final int PLAYER_NUM = 5;

    public static void main(String[] args) {
        CountDownLatch start = new CountDownLatch(1);
        CountDownLatch end = new CountDownLatch(5);

        Player[] players = new Player[PLAYER_NUM];

        for (int i = 0; i < PLAYER_NUM; i++) {
            players[i] = new Player(start, end, i);
        }
        //指定线程个数的线程池
        ExecutorService executorService = Executors.newFixedThreadPool(PLAYER_NUM);
        for (Player player : players) {
            executorService.execute(player);
        }

        System.out.println("all thread start");

        start.countDown();

        try {
            end.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            System.out.println("all thread end");
            executorService.shutdown();
        }
    }

}

class Player implements Runnable {

    private CountDownLatch start;

    private CountDownLatch end;

    private int id;

    Random random = new Random();

    public Player(CountDownLatch start, CountDownLatch end, int id) {
        this.start = start;
        this.end = end;
        this.id = id;
    }

    @Override
    public void run() {
        try {
            //保证所有线程同时开始
            start.await();
            TimeUnit.SECONDS.sleep(random.nextInt(10));
            System.out.println("player-" + id + ":arrived");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            end.countDown();
            System.out.println(end.getCount());
        }
    }
}

```

```java
public class CyclicBarrierDemo {

    public static void main(String[] args) {
        int n = 4;
        CyclicBarrier barrier = new CyclicBarrier(4, () -> System.out.println("当前线程" + Thread.currentThread().getName()));

        for (int i = 0; i < n; i++) {
            new Writer(barrier).start();
        }

        try {
            TimeUnit.MILLISECONDS.sleep(25000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("CyclicBarrier重用");

        barrier.reset();
        for (int i = 0; i < n; i++) {
            new Writer(barrier).start();
        }
    }
}

class Writer extends Thread {

    private CyclicBarrier barrier;

    public Writer(CyclicBarrier barrier) {
        this.barrier = barrier;
    }

    @Override
    public void run() {
        System.out.println("Thread" + Thread.currentThread().getName() + "正在写入数据...");
        try {
            TimeUnit.MICROSECONDS.sleep(5000);
            System.out.println("Thread" + Thread.currentThread().getName() + "写入完成，等待其他线程写入完毕");
            barrier.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
        System.out.println("所有线程写入完毕，继续处理其他任务");
    }
}
```

