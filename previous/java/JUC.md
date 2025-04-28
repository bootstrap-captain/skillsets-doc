# 简介

## 1. 进程/线程

```bash
# 进程
- 由指令和数据组成，指令要运行，数据要读写，必须将指令加载到CPU，数据加载到内存。指令运行过程中还需要用到磁盘，网络等IO设备
- 进程用来加载指令，管理内存，管理IO
- 一个程序被运行，从磁盘加载这个程序的代码到内存，就是开启了一个进程
- 进程可以视为一个程序的实例
- 大部分程序可以运行多个实例进程，也有的程序只能启动一个实例进程

# 线程
- 一个进程内部包含1-n个线程
- 一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给CPU执行
- JVM中，进程作为资源分配的最小单元，线程作为最小调度单元

# 对比
- 进程基本上相互独立                                    线程存在于进程内，是进程的一个子集
- 进程拥有共享的资源，如内存空间，供其内部的线程共享
- 进程间通信比较复杂： 同一计算机的进程通信为IPC(Inter-process communication)， 
                    不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，如HTTP
- 线程通信比较简单，因为它们共享进程内的内存，如多个线程可以访问同一个共享变量
- 线程更轻量，线程上下文切换成本比进程低
```

## 2. 并行/并发

### 2.1 并发

```bash
- concurrent：cpu核心同一个时间应对多个线程的能力

- 单核cpu下，线程是串行
- 任务调度器： 操作系统组件，将cpu的时间片（windows下为15ms）分给不同的线程使用
- cpu在线程间的切换非常快，感觉就是各个线程是同时运行的
- 微观串行，宏观并行
```

### 2.2 并行

```bash
- parrel： 同一个时间内，cpu真正去执行多个线程的能力

- 多核cpu下，每个核都可以调度运行线程，这个时候线程就是并行的
```

- 很多时候，并发和并行是同时存在的

![image-20230428164109578](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230428164109578.png)

### 2.3 应用场景

```bash
# 异步调用
- 异步方法：  不需立刻返回结果
- 耗时业务：  不阻塞主线程的业务

# 提升效率
- 任务拆分：  耗时业务拆分，多任务间互不依赖
- 多核cpu：  任务拆分，多线程分别执行，这样就会分到更多的cpu
- 单核cpu：  没必要拆分，串行执行，并有上下文切换的损失。  但是能够在不同任务之间切换
```

## 3. 进程查看

### 3.1. Linux

```bash
# process status
ps -ef
ps -fe               # 进程查看

ps -ef|grep java
ps -fe|grep java     # 管道运算符， 查看java进程(包含java进程和查看的grep进程)

kill pid             # 杀死指定进程
top                  # 动态查看当前进程的占用cpu和mem情况。ctrl+c 退出

top -H -P (pid)      # 查看某个进程内部的所有线程的cpu使用
                     # -H : 表示查看线程
```

### 3.2 Java

```bash
# JPS
- 查看java进程

# JConsole
- Java内置，检测java线程的图形化界面工具
- 位于jdk/bin/目录下
- 可以用来检测死锁
```

## 4. 线程切换

- Thread Context Switch
- cpu不再执行当前线程，转而执行另一个线程代码
- 线程切换时，操作系统保存当前线程的状态，并恢复另一个线程的状态
- 线程切换频繁发生会影响性能

```bash
- 线程cpu时间片用完
- 垃圾回收              
- 有更高优先级的线程需要运行
- 线程自己调用了sleep，yield，wait，join，park，synchronized，lock等方法
```

# Park/UnPark

```java
// java.util.concurrent.locks.LockSupport 包的方法

// 暂停当前线程，线程一直生存，直到被打断才会继续往下执行
public static void park() {
    U.park(false, 0L);
}

// 恢复某个线程的运行
public static void unpark(Thread thread) {
    if (thread != null)
        U.unpark(thread);
}
```

## 1. 基本使用

```java
package com.erick.multithread.d01;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.LockSupport;

@Slf4j
public class Demo01 {
    public static void main(String[] args) {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                log.info("start");
                LockSupport.park();
                log.info("end");
            }
        });

        thread.start();

        Sleep.sleep(2);
        log.info("main unPark");
        LockSupport.unpark(thread);
    }
}
```

## 2. 带锁实现

- park不会释放当前锁，类似sleep

```java
package com.erick.multithread.d01;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.LockSupport;

@Slf4j
public class Demo01 {
    private static final Object lock = new Object();

    public static void main(String[] args) {
        Thread firstThread = new Thread(() -> {
            synchronized (lock) {
                log.info("{} start", Thread.currentThread().getName());
                LockSupport.park();
                log.info("{} end", Thread.currentThread().getName());
            }
        });

        Thread secondThread = new Thread(() -> {
            synchronized (lock) {
                log.info("{} start", Thread.currentThread().getName());
                LockSupport.park();
                log.info("{} end", Thread.currentThread().getName());
            }
        });

        firstThread.start();
        secondThread.start();

        Sleep.sleep(2);

        LockSupport.unpark(firstThread);
    }
}
```

## 3. 先unPark后park

- unPark可以在park前调用，也可以在park后调用

```java
package com.erick.multithread.d01;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.LockSupport;

public class Demo01 {
    public static void main(String[] args) {
        ParkService service = new ParkService();
        Thread firstThread = new Thread(() -> service.work());
        /*后面的park就不会停下来了*/
        firstThread.start(); // 先启动要测试的线程
        /*先unPark*/
        new Thread(() -> service.unParkJob(firstThread)).start();
    }
}

@Slf4j
class ParkService {
    public synchronized void work() {
        log.info("{} start to run", Thread.currentThread().getName());

        Sleep.sleep(3);

        log.info("{} ready to park", Thread.currentThread().getName());
        LockSupport.park();
        log.info("{} resume running", Thread.currentThread().getName());
    }

    public void unParkJob(Thread thread) {
        log.info("{} unPark", Thread.currentThread().getName());
        LockSupport.unpark(thread);
    }
}
```

## 4. park/wait

|      | Park/Unpark                      | Wait/Notify                                   |
| ---- | -------------------------------- | --------------------------------------------- |
| 来源 | LockSupport的方法                | Object的方法                                  |
| 粒度 | 是以线程为单位来阻塞和唤醒的     | notify只能随机唤醒一个, notifyAll唤醒所有线程 |
| 顺序 | 可以先unPark                     | 不能先notify                                  |
| 用法 | 不用一定要锁，Park时候不会释放锁 | 必须先获取到锁，wait时候释放锁                |

## 5. Park原理

- 多次park

```java
package com.erick.multithread.d01;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.LockSupport;

@Slf4j
public class Demo02 {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            log.info("first park----");
            LockSupport.park();
            log.info("first resume---");

            log.info("second park----");
            LockSupport.park();
            log.info("second resume---");
        });

        thread.start();

        Sleep.sleep(1);
        log.info("first unPark");
        LockSupport.unpark(thread);

        Sleep.sleep(1);
        log.info("second unPark");
        LockSupport.unpark(thread);
    }
}
```

#### 

# AQS

- AbstractQueuedSynchronizer：抽象类，阻塞式锁和相关的同步器工具的框架
- 以下笔记，基于JDK-17

## 1. AbstractQueuedSynchronizer

### 1.1 state

- 表示资源的状态(独占模式， 共享模式)
- 子类需要定义如何维护这个状态，控制如何获取锁和释放锁
- 独占模式：只有一个线程能够访问资源
- 共享模式：允许多个线程访问资源，但对线程的最大数有限制

```java
private volatile int state;

protected final int getState() {
     return state;
}

protected final void setState(int newState) {
     state = newState;
}

// 乐观锁机制设置state状态，防止多个线程来修改state
protected final boolean compareAndSetState(int expect, int update) {
    return U.compareAndSetInt(this, STATE, expect, update);
}

// 调用Unsafe类来做
private static final Unsafe U = Unsafe.getUnsafe();
```

### 1.2 阻塞队列

- 提供了基于FIFO的阻塞队列，类似于Monitor中的EntryList

### 1.3 等待队列

- 条件变量来实现等待，唤醒机制， 支持多个条件变量，类似于Monitor的WaitSet
- ConditionObject: 等待队列，是AbstractQueuedSynchronizer的内部类

![image-20240413112630288](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20240413112630288.png)

## 2. 自定义AQS锁

- 用AQS写一个不可重入锁

```bash
# 子类继承Lock接口
# 并实现下面的方法： 默认抛出UnsupportedOperationException
- tryAcquire
- tryRealease
- tryAcquiredShared
- tryReleaseShared
- isHeldExclusively
```

### 2.1 同步器类

- 同步器类来进行加锁解锁操作
- ErickSync继承AbstractQueuedSynchronizer，重写某些方法

#### AbstractQueuedSynchronizer待实现

```java
// 需要重写的方法：AbstractQueuedSynchronizer没有实现的

// 尝试获取锁
protected boolean tryAcquire(int arg) {
    throw new UnsupportedOperationException();
}


// 尝试释放锁
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}


// 是否持有当前锁
protected boolean isHeldExclusively() {
    throw new UnsupportedOperationException();
}
```

#### 同步器具体实现

```java
package com.citi.d01;

import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;

public class ErickSync extends AbstractQueuedSynchronizer {

    /*尝试加锁： 修改AbstractQueuedSynchronizer的state属性
     * 1. 修改state属性
     * 2. 设置当前线程为Owner中的，类似Monitor的Owner
     */
    @Override
    protected boolean tryAcquire(int arg) {
        /*修改AbstractQueuedSynchronizer的state属性
         * 0: 未加锁
         * 1: 加锁*/
        if (compareAndSetState(0, 1)) {
            // AbstractQueuedSynchronizer继承AbstractOwnableSynchronizer方法中的
            setExclusiveOwnerThread(Thread.currentThread()); // 设置当前线程为Owner的主人
            return true;
        }
        return false;
    }

    @Override
    protected boolean tryRelease(int arg) {
        setState(0); // 不用担心其他线程竞争
        setExclusiveOwnerThread(null); // 清空owner中的线程
        return true;
    }

    /*是否持有当前独占锁*/
    @Override
    protected boolean isHeldExclusively() {
        // AbstractOwnableSynchronizer中的方法
        return getExclusiveOwnerThread() == Thread.currentThread();
    }

    /*AbstractQueuedSynchronizer类中维护的属性*/
    public Condition newCondition() {
        return new ConditionObject();
    }
}
```

### 2.2 自定义锁

- 在锁中，组合上面同步器，调用上面同步器中的方法

```java
package com.citi.d01;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

/*实现Lock接口*/
@Slf4j
public class ErickLock implements Lock {

    /*其实就是调用同步器来进行加锁解锁*/
    private ErickSync erickSync = new ErickSync();

    /*加锁：不成功则会进入类似EntryList的阻塞队列*/

    /*   AbstractQueuedSynchronizer 中方法

          public final void acquire(int arg) {
            if (!tryAcquire(arg))
                acquire(null, arg, false, false, false, 0L);
          }

          先走自定义的tryAcquire，不成功则再次走AbstractQueuedSynchronizer的acquire
        */
    @Override
    public void lock() {
        erickSync.acquire(1); // 参数暂时没用
    }

    /*可打断的加锁： AbstractQueuedSynchronizer已经实现*/
    @Override
    public void lockInterruptibly() throws InterruptedException {
        erickSync.acquireInterruptibly(1);
    }

    /*尝试加锁： 只尝试一次*/
    @Override
    public boolean tryLock() {
        return erickSync.tryAcquire(1);
    }

    /*尝试加锁带超时：只尝试一次
    AbstractQueuedSynchronizer已经实现，会调用自定义的tryAcquire()
    * */
    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return erickSync.tryAcquireNanos(1, unit.toNanos(time));
    }

    /*释放锁：并唤醒阻塞的线程
     *  AbstractQueuedSynchronizer已经实现，tryRelease()*/
    @Override
    public void unlock() {
        erickSync.release(1);
    }

    /*创建条件变量*/
    @Override
    public Condition newCondition() {
        return erickSync.newCondition();
    }
}
```

### 2.3 测试代码

```java
package com.erick.multithread.aqs;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

public class Demo01 {
    public static void main(String[] args) {
        LockTestService testService = new LockTestService();

        new Thread(() -> testService.work()).start();
        new Thread(() -> testService.work()).start();
    }
}

@Slf4j
class LockTestService {
    private ErickLock lock = new ErickLock();

    public void work() {
        try {
            lock.lock();
            log.info("{} will sleep", Thread.currentThread().getName());
            Sleep.sleep(2);
        } finally {
            log.info("{} unlock", Thread.currentThread().getName());
            lock.unlock();
        }
    }
}
```

# ReentryLock

- 是AQS的一种实现
- 典型的AQS锁，实现了Lock接口，内部组装了Sync同步器

```java
// 基本语法
ReentryLock lock = new Reentrylock();

lock.lock();
try{
   // 临界区代码
}finally{
  // 释放锁
  lock.unlock();
}
```

![ReentrantLock](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/ReentrantLock.png)

## 1. 可重入锁

- 可重入锁：一个线程已经获取锁，则该线程是锁Owner，跨方法获取该锁时，依然成功
- 不可重入：同一线程跨方法获取就会造成死锁，部分锁就是这种情况

```java
package com.erick.multithread.aqs;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.ReentrantLock;

public class Demo02 {
    public static void main(String[] args) {
        RetryTestService testService = new RetryTestService();

        /**
         * 111-加锁
         * 222-加锁
         * 222-解锁
         * 111-解锁
         */
        new Thread(() -> testService.first()).start();
    }
}

@Slf4j
class RetryTestService {
    private ReentrantLock lock = new ReentrantLock();

    public void first() {
        lock.lock();
        try {
            log.info("111-加锁");
            second();
        } finally {
            /*unlock必须放在finally中，保证锁一定可以释放*/
            lock.unlock();
            log.info("111-解锁");
        }
    }

    public void second() {
        lock.lock();
        try {
            log.info("222-加锁");
        } finally {
            lock.unlock();
            log.info("222-解锁");
        }
    }
}
```

## 2. 可打断锁

```bash
# 不可打断锁
- 如：尝试获取synchronized锁的线程，如果获取不到锁，则会在对应Monitor的EntryList中一直死等

# 可打断锁
- 避免一个线程一直等待当前要获取的锁
- 被动的方式避免死锁

#  lock.lockInterruptibly();
- 线程没有获取到锁，进入类似EntryList中等待
- 这种等待可以被打断
```

```java
// 可以主动打断
public void lockInterruptibly() throws InterruptedException {
    sync.lockInterruptibly();
}
```

```java
package com.erick.multithread.aqs;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.ReentrantLock;

public class Demo03 {
    public static void main(String[] args) {
        InterruptLockService service = new InterruptLockService();
        Thread firstThread = new Thread(() -> service.work());
        firstThread.start();

        Sleep.sleep(1);

        /*线程2在1s后打断自己获取锁的阻塞状态*/
        Thread secondThread = new Thread(() -> service.work());
        secondThread.start();
        secondThread.interrupt();
    }
}

@Slf4j
class InterruptLockService {
    private ReentrantLock lock = new ReentrantLock();

    public void work() {
        try {
            lock.lockInterruptibly();
            log.info("{} 加锁", Thread.currentThread().getName());
        } catch (InterruptedException e) {
            log.error("{} 锁打断", Thread.currentThread().getName());
            throw new RuntimeException(e);
        }

        try {
            executeBusiness();
        } finally {
            lock.unlock();
            log.info("{} 释放锁", Thread.currentThread().getName());
        }
    }

    private void executeBusiness() {
        log.info("doing business");
        Sleep.sleep(5);
    }
}
```

## 3. 超时锁

- 避免死锁： 主动方式来避免一个线程一直等待锁资源，带来的死锁问题

### 不带超时

```java
// 尝试获取锁，获取失败后, 自定义处理流程
public boolean tryLock() {
     return sync.tryLock();
}
```

```java
package com.erick.multithread.aqs;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.ReentrantLock;

public class Demo06 {
    public static void main(String[] args) {
        TimeOutTestService testService = new TimeOutTestService();
        new Thread(() -> testService.service()).start();
        new Thread(() -> testService.service()).start();
    }
}

@Slf4j
class TimeOutTestService {
    private ReentrantLock lock = new ReentrantLock();

    public void service() {
        boolean result = lock.tryLock();
        if (!result) {
            log.info("未获取到锁，放弃业务");
            return;
        }

        try {
            log.info("获取到锁，执行业务", Thread.currentThread().getName());
            execute();
        } finally {
            lock.unlock();
        }
    }

    private void execute() {
        Sleep.sleep(2);
        log.info("execute");
    }
}
```

### 带超时

- 等待过程中，如果获取到了锁，则执行正常的业务流程
- 等待一段时间后，如果没有获取到锁，则中断获取锁的竞争
- 等待过程中，依然可以被打断

```java
// 带超时的，在一定时间内获取锁，超时则进入类似EntryList的去等待
public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryLockNanos(unit.toNanos(timeout));
}
```

```java
package com.erick.multithread.aqs;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

public class Demo06 {
    public static void main(String[] args) {
        TimeOutTestService testService = new TimeOutTestService();
        new Thread(() -> testService.service()).start();
        new Thread(() -> testService.service()).start();
    }
}

@Slf4j
class TimeOutTestService {
    private ReentrantLock lock = new ReentrantLock();

    public void service() {
        boolean result;
        // 等待2s后，就不再等待锁
        try {
            result = lock.tryLock(2, TimeUnit.SECONDS);
        } catch (InterruptedException e) {
            // 可以被打断
            throw new RuntimeException(e);
        }
        if (!result) {
            log.info("未获取到锁，放弃业务");
            return;
        }

        try {
            log.info("获取到锁，执行业务", Thread.currentThread().getName());
            execute();
        } finally {
            lock.unlock();
        }
    }

    private void execute() {
        Sleep.sleep(3);
        log.info("execute");
    }
}
```

## 4. 公平锁

- 可以防止线程饥饿

```bash
# 1. 不公平锁
- synchronized锁
- 一个线程持有锁时，其他线程进入锁的 EntryList
- 获的锁的线程任务执行完后，Owner清空，EntryList中的线程随机挑选一个给锁，而不是按照进入的顺序先到先得

# 2. 公平锁： 通过ReentranLock实现
- public ReentrantLock(boolean fair) 
- 默认是非公平锁，传参为true，公平锁
```

```java
package com.erick.multithread.aqs;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.ReentrantLock;

public class Demo04 {
    public static void main(String[] args) {
        FairLockService fairLockService = new FairLockService();
        for (int i = 0; i < 10; i++) {
            String name = i + "号线程---";
            new Thread(() -> fairLockService.work(), name).start();
        }
    }
}

@Slf4j
class FairLockService {
    private ReentrantLock lock = new ReentrantLock(true);

    public void work() {
       lock.lock();
        try {
            log.info("{} 加锁", Thread.currentThread().getName());
            Sleep.sleep(2);
        } finally {
            log.info("{} 释放锁", Thread.currentThread().getName());
            lock.unlock();
        }
    }
}
```

## 5. 条件变量

- 类似WaitSet， 但是有可以有多个，可以和wait/notify更好的结合
- 将等待的不同WaitSet分类，然后指定WaitSet来进行唤醒

```bash
# 1. 创建一个等待的队列
- public Condition newCondition()

# 2. 将一个线程在某个队列中进行等待
Condition        public final void await() throws InterruptedException

# 3. 去某个队列中唤醒等待的线程
Condition        public final void signal()
                 public final void signalAll()
```

```java
package com.erick.multithread.aqs;

import com.erick.multithread.Sleep;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class Demo05 {
    public static void main(String[] args) {
        MultiWaitService service = new MultiWaitService();
        for (int i = 0; i < 3; i++) {
            new Thread(() -> service.boyWork(), "BOY").start();
            new Thread(() -> service.girlWork(), "GIRL").start();
        }

        Sleep.sleep(2);

        new Thread(() -> service.deliverCigarette()).start();
        Sleep.sleep(2);
        new Thread(() -> service.deliverDinner()).start();

    }
}

@Slf4j
class MultiWaitService {
    private ReentrantLock lock = new ReentrantLock();
    private Condition boyRoom = lock.newCondition();
    private Condition girlRoom = lock.newCondition();

    private boolean hasCigarette = false;
    private boolean hasDinner = false;

    public void boyWork() {
       lock.lock();
        try {
            while (!hasCigarette) {
                try {
                    log.info("{} 没烟，男人休息", Thread.currentThread().getName());
                    boyRoom.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.info("{} 烟来了，男人干活", Thread.currentThread().getName());
            }
        } finally {
            lock.unlock();
        }
    }

    public void girlWork() {
        try {
            lock.lock();
            while (!hasDinner) {
                try {
                    log.info("{} 没饭，女人休息", Thread.currentThread().getName());
                    girlRoom.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                log.info("{} 没饭，女人干活", Thread.currentThread().getName());
            }
        } finally {
            lock.unlock();
        }
    }

    public void deliverCigarette() {
         lock.lock();
        try {
            log.info("======================烟来了");
            hasCigarette = true;
            boyRoom.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public void deliverDinner() {
        lock.lock();
        try {
            log.info("======================饭来了");
            hasDinner = true;
            girlRoom.signalAll();
        } finally {
            lock.unlock();
        }
    }
}
```

## 6. ReentrantLock *vs* Synchronized

```bash
可重入性：   都支持
可打断性：   Synchronized锁不能被打断                 ReentrantLock可以被打断，防止死锁
超时性：     Synchronized锁获取时候会一直等待          ReentrantLock支持超时等待
公平性：     Synchronized的EntryList是不公平         ReentrantLock(true)可以是公平锁
条件变量：   Synchronized的WaitSet只有一个            ReentrantLock支持不同的WaitSet
```



# ReentrantReadWriteLock

- 读读不互斥，写写互斥提高并发能力
- 读写互斥：保证写的时候不能读

![ReentrantReadWriteLock](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/ReentrantReadWriteLock.png)

## 1. 基本使用

### 1.1读读

- 不互斥

```java
package com.citi.d01;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.ReentrantReadWriteLock;

public class Demo01 {
    public static void main(String[] args) {
        DataService service = new DataService();
        new Thread(() -> System.out.println(service.readData())).start();
        new Thread(() -> System.out.println(service.readData())).start();
    }
}

@Slf4j
class DataService {
    private Object data;

    private ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private ReentrantReadWriteLock.ReadLock readLock = readWriteLock.readLock();

    public Object readData() {
        readLock.lock();
        try {
            log.info("{}--获取读锁", Thread.currentThread().getName());
            Sleep.sleep(2);
        } finally {
            log.info("{}--释放读锁", Thread.currentThread().getName());
            readLock.unlock();
        }
        return data;
    }
}
```

### 1.2 写写

- 写写互斥

```java
package com.citi.d01;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.ReentrantReadWriteLock;

public class Demo01 {
    public static void main(String[] args) {
        DataService service = new DataService();
        new Thread(() -> System.out.println(service.writeData())).start();
        new Thread(() -> System.out.println(service.writeData())).start();
    }
}

@Slf4j
class DataService {
    private Object data;

    private ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private ReentrantReadWriteLock.WriteLock writeLock = readWriteLock.writeLock();

    public Object writeData() {
        writeLock.lock();
        try {
            log.info("{}--获取写锁", Thread.currentThread().getName());
            data = new Object();
            Sleep.sleep(2);
        } finally {
            log.info("{}--释放写锁", Thread.currentThread().getName());
            writeLock.unlock();
        }
        return data;
    }
}
```

### 1.3 读-写

- 读写互斥

```java
package com.citi.d01;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.ReentrantReadWriteLock;

public class Demo01 {
    public static void main(String[] args) {
        DataService service = new DataService();
        new Thread(() -> System.out.println(service.readData())).start();
        new Thread(() -> System.out.println(service.writeData())).start();
    }
}

@Slf4j
class DataService {
    private Object data;

    private ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();

    private ReentrantReadWriteLock.ReadLock readLock = readWriteLock.readLock();

    private ReentrantReadWriteLock.WriteLock writeLock = readWriteLock.writeLock();

    public Object readData() {
        readLock.lock();
        try {
            log.info("{}--获取读锁", Thread.currentThread().getName());
            data = new Object();
            Sleep.sleep(2);
        } finally {
            log.info("{}--释放读锁", Thread.currentThread().getName());
            readLock.unlock();
        }
        return data;
    }

    public Object writeData() {
        writeLock.lock();
        try {
            log.info("{}--获取写锁", Thread.currentThread().getName());
            data = new Object();
            Sleep.sleep(2);
        } finally {
            log.info("{}--释放写锁", Thread.currentThread().getName());
            writeLock.unlock();
        }
        return data;
    }
}
```



# 固定顺序输出

## 1. 先2后1

- 线程2运行完毕后，线程1再开始运行

### 1.1  synchronized

- synchronized + wait + notify

```java
package com.erick.multithread.d02;

import com.erick.multithread.Sleep;

public class Demo01 {

    private static final Object lock = new Object();

    public static void main(String[] args) {

        Thread first = new Thread(() -> {
            synchronized (lock) {
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                System.out.println(Thread.currentThread().getName() + " running");
            }
        }, "t-1");
        first.start();

        Sleep.sleep(2);

        Thread second = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (lock) {
                    System.out.println(Thread.currentThread().getName() + " running");
                    lock.notify();
                }
            }
        }, "t-2");
        second.start();
    }
}
```

### 1.2 reentrylock

- reentrylock + await + signal

```java
package com.erick.multithread.d02;

import com.erick.multithread.Sleep;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class Demo02 {
    private static ReentrantLock lock = new ReentrantLock();

    private static Condition room = lock.newCondition();

    public static void main(String[] args) {
        Thread first = new Thread(() -> {
            lock.lock();
            try {
                room.await();
                System.out.println(Thread.currentThread().getName() + " running");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            } finally {
                lock.unlock();
            }
        }, "t-1");
        first.start();

        Sleep.sleep(2);

        Thread second = new Thread(new Runnable() {
            @Override
            public void run() {
                lock.lock();
                try {
                    System.out.println(Thread.currentThread().getName() + " running");
                    room.signal();
                } finally {
                    lock.unlock();
                }
            }
        }, "t-2");
        second.start();
    }
}
```

### 1.3 park + unpark

```java
package com.erick.multithread.d02;

import com.erick.multithread.Sleep;

import java.util.concurrent.locks.LockSupport;

public class Demo03 {
    public static void main(String[] args) {
        Thread first = new Thread(() -> {
            LockSupport.park();
            System.out.println(Thread.currentThread().getName() + " running");
        }, "t-1");

        first.start();

        Sleep.sleep(2);

        Thread second = new Thread(() -> {
            System.out.println(Thread.currentThread().getName() + " running");
            LockSupport.unpark(first);
        }, "t-2");

        second.start();
    }
}
```

## 2. 交替输出字符

- 两个线程交替输出a,b

### 2.1  synchronized

- synchronized + wait + notify

```java
package com.erick.multithread.d02;

public class Demo04 {
    public static void main(String[] args) {
        PrintService service = new PrintService();
        new Thread(() -> service.print("a")).start();
        new Thread(() -> service.print("b")).start();
    }
}

class PrintService {
    public void print(String number) {
        while (true) {
            synchronized (this) {
                System.out.println(number);
                this.notify();
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}
```

### 2.2 reentrylock

- reentrylock + await + signal

```java
package com.erick.multithread.d02;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class Demo04 {
    public static void main(String[] args) {
        PrintService service = new PrintService();
        new Thread(() -> service.print("a")).start();
        new Thread(() -> service.print("b")).start();
    }
}

class PrintService {
    private ReentrantLock lock = new ReentrantLock();
    private Condition room = lock.newCondition();

    public void print(String number) {
        while (true) {
            lock.lock();
            try {
                System.out.println(number);
                room.signal();
                
                try {
                    room.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            } finally {
                lock.unlock();
            }
        }
    }
}
```

### 2.3 park + unpark

```java
package com.erick.multithread.d02;

import java.util.concurrent.locks.LockSupport;

public class Demo04 {

    private static Thread firstThread;
    private static Thread secondThread;

    public static void main(String[] args) {
        PrintService service = new PrintService();

        firstThread = new Thread(() -> service.print("a", secondThread));
        secondThread = new Thread(() -> service.print("b", firstThread));

        firstThread.start();
        secondThread.start();
    }
}

class PrintService {
    public void print(String number, Thread thread) {
        while (true) {
            System.out.println(number);
            LockSupport.unpark(thread);
            LockSupport.park();
        }
    }
}
```

## 3. 交替输出奇偶数

- 两个线程，交替输出奇偶数

### 3.1  synchronized

- synchronized + wait + notify

```java
package com.erick.multithread.d02;

import java.util.concurrent.TimeUnit;

public class Demo05 {
    public static void main(String[] args) {
        PrintNumberService service = new PrintNumberService();

        new Thread(() -> service.print()).start();
        new Thread(() -> service.print()).start();
    }
}

class PrintNumberService {
    private int number = 1;

    public void print() {
        while (true) {
            synchronized (this) {
                System.out.println(Thread.currentThread().getName() + "-----" + number);
                number++;
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
                this.notify();
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}
```

### 3.2 reentrylock

- reentrylock + await + signal

```java
package com.erick.multithread.d02;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

public class Demo05 {
    public static void main(String[] args) {
        PrintNumberService service = new PrintNumberService();

        new Thread(() -> service.print()).start();
        new Thread(() -> service.print()).start();
    }
}

class PrintNumberService {
    private int number = 1;

    private ReentrantLock lock = new ReentrantLock();
    private Condition room = lock.newCondition();

    public void print() {
        while (true) {
            lock.lock();
            try {
                System.out.println(Thread.currentThread().getName() + "-----" + number);
                number++;
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

                room.signal();

                try {
                    room.await();
                } catch (InterruptedException e) {
                    throw new RuntimeException(e);
                }

            } finally {
                lock.unlock();
            }

        }
    }
}
```

### 3.3 park + unpark

```java
package com.erick.multithread.d02;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.LockSupport;

public class Demo05 {
    private static Thread firstThread;
    private static Thread secondThread;

    public static void main(String[] args) {
        PrintNumberService service = new PrintNumberService();
        firstThread = new Thread(() -> service.print(secondThread));
        secondThread = new Thread(() -> service.print(firstThread));

        firstThread.start();
        secondThread.start();

        // 触发
        LockSupport.unpark(firstThread);
    }
}

class PrintNumberService {
    private int number = 1;

    public void print(Thread thread) {
        while (true) {
            LockSupport.park();

            System.out.println(Thread.currentThread().getName() + "-----" + number);
            number++;
            LockSupport.unpark(thread);

            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

