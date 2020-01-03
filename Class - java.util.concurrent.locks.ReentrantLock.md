# Class - java.util.concurrent.locks.ReentrantLock

Created by : Mr Dk.

2020 / 01 / 03 15:22

Nanjing, Jiangsu, China

---

## Definition

```java
public class ReentrantLock implements Lock, java.io.Serializable {

}
```

可重入锁在基本行为上和 `synchronized` 关键字类似

但有更多的功能

可重入锁被上一次成功获得锁但还未释放的线程占有

当锁没有被其它线程持有时，一个线程调用 `lock()` 将会成功返回

如果锁被当前线程持有，该函数将会立刻返回

该类的构造函数带有一个 __是否公平__ 的可选参数 - `fairness`

* 当设定为 `true` 时，当锁被竞争时，等待最久的线程将获得锁 (FIFO)
* 否则不保证得到锁的顺序

使用公平锁的程序将具有相对低的吞吐率 (slower)，但能够防止饥饿

建议使用方法:

```java
class X {
    private final ReentrantLock lock = new ReentrantLock();
    // ...

    public void m() {
        lock.lock();  // block until condition holds
        try {
            // ... method body
        } finally {
            lock.unlock()
        }
    }
}
```

可重入锁最多支持同一个线程递归上锁 `2147483647` 次

```java
/**
 * A reentrant mutual exclusion {@link Lock} with the same basic
 * behavior and semantics as the implicit monitor lock accessed using
 * {@code synchronized} methods and statements, but with extended
 * capabilities.
 *
 * <p>A {@code ReentrantLock} is <em>owned</em> by the thread last
 * successfully locking, but not yet unlocking it. A thread invoking
 * {@code lock} will return, successfully acquiring the lock, when
 * the lock is not owned by another thread. The method will return
 * immediately if the current thread already owns the lock. This can
 * be checked using methods {@link #isHeldByCurrentThread}, and {@link
 * #getHoldCount}.
 *
 * <p>The constructor for this class accepts an optional
 * <em>fairness</em> parameter.  When set {@code true}, under
 * contention, locks favor granting access to the longest-waiting
 * thread.  Otherwise this lock does not guarantee any particular
 * access order.  Programs using fair locks accessed by many threads
 * may display lower overall throughput (i.e., are slower; often much
 * slower) than those using the default setting, but have smaller
 * variances in times to obtain locks and guarantee lack of
 * starvation. Note however, that fairness of locks does not guarantee
 * fairness of thread scheduling. Thus, one of many threads using a
 * fair lock may obtain it multiple times in succession while other
 * active threads are not progressing and not currently holding the
 * lock.
 * Also note that the untimed {@link #tryLock()} method does not
 * honor the fairness setting. It will succeed if the lock
 * is available even if other threads are waiting.
 *
 * <p>It is recommended practice to <em>always</em> immediately
 * follow a call to {@code lock} with a {@code try} block, most
 * typically in a before/after construction such as:
 *
 *  <pre> {@code
 * class X {
 *   private final ReentrantLock lock = new ReentrantLock();
 *   // ...
 *
 *   public void m() {
 *     lock.lock();  // block until condition holds
 *     try {
 *       // ... method body
 *     } finally {
 *       lock.unlock()
 *     }
 *   }
 * }}</pre>
 *
 * <p>In addition to implementing the {@link Lock} interface, this
 * class defines a number of {@code public} and {@code protected}
 * methods for inspecting the state of the lock.  Some of these
 * methods are only useful for instrumentation and monitoring.
 *
 * <p>Serialization of this class behaves in the same way as built-in
 * locks: a deserialized lock is in the unlocked state, regardless of
 * its state when serialized.
 *
 * <p>This lock supports a maximum of 2147483647 recursive locks by
 * the same thread. Attempts to exceed this limit result in
 * {@link Error} throws from locking methods.
 *
 * @since 1.5
 * @author Doug Lea
 */
```

---

首先，锁对象内部会维护一个 AQS (Abstract Queued Synchronizer)

作为锁的内部实现

在可重入锁中，AQS 中的原子状态用于表示持有锁的次数

```java
/** Synchronizer providing all implementation mechanics */
private final Sync sync;

/**
 * Base of synchronization control for this lock. Subclassed
 * into fair and nonfair versions below. Uses AQS state to
 * represent the number of holds on the lock.
 */
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = -5179523762034025860L;

    /**
     * Performs {@link Lock#lock}. The main reason for subclassing
     * is to allow fast path for nonfair version.
     */
    abstract void lock();

    /**
     * Performs non-fair tryLock.  tryAcquire is implemented in
     * subclasses, but both need nonfair try for trylock method.
     */
    final boolean nonfairTryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0) // overflow
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

    protected final boolean isHeldExclusively() {
        // While we must in general read state before owner,
        // we don't need to do so to check if current thread is owner
        return getExclusiveOwnerThread() == Thread.currentThread();
    }

    final ConditionObject newCondition() {
        return new ConditionObject();
    }

    // Methods relayed from outer class

    final Thread getOwner() {
        return getState() == 0 ? null : getExclusiveOwnerThread();
    }

    final int getHoldCount() {
        return isHeldExclusively() ? getState() : 0;
    }

    final boolean isLocked() {
        return getState() != 0;
    }

    /**
     * Reconstitutes the instance from a stream (that is, deserializes it).
     */
    private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        s.defaultReadObject();
        setState(0); // reset to unlocked state
    }
}
```

可以看到，实现了一个非公平的 `nonfairTryAcquire()`

* 如果锁没有被占用，则直接获得锁
* 如果锁被本线程占用，则将自身的状态变量 +1，还限制了最大持有次数
* 否则返回失败

此外，还实现了一个 `tryRelease()`

* 递减占用次数
* 如果占用次数为 0，就将锁置为空闲

> 纵观几个类来看，最基础的是 AbstractOwnableSynchronizer
>
> 其中维护了持有锁的 `Thread`
>
> AbstractQueuedSynchronizer 继承了 AOS
>
> 其中附加并维护了原子状态变量
>
> 最终，ReentrantLock 在类内部继承并维护了一个公平或非公平的 AQS
>
> 调用和维护层次是这样的

由于可重入锁支持 __公平锁__ 和 __非公平锁__

因此可重入锁内部支持的 AQS 也可以是公平或非公平的

非公平 AQS:

```java
/**
 * Sync object for non-fair locks
 */
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = 7316153563782823691L;

    /**
     * Performs lock.  Try immediate barge, backing up to normal
     * acquire on failure.
     */
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }

    protected final boolean tryAcquire(int acquires) {
        return nonfairTryAcquire(acquires);
    }
}
```

上来就直接 CAS，如果不成功，再调 `acquire` 进入等待队列

公平 AQS:

```java
/**
 * Sync object for fair locks
 */
static final class FairSync extends Sync {
    private static final long serialVersionUID = -3000897897090466540L;

    final void lock() {
        acquire(1);
    }

    /**
     * Fair version of tryAcquire.  Don't grant access unless
     * recursive call or no waiters or is first.
     */
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }
}
```

实现了公平版本的 `tryAcquire()`

* 严格按照等待队列的 FIFO 顺序进行 CAS
* 当前锁空闲时，当队列中没有前一个等待结点，且 CAS 成功时，获得锁
* 如果当前线程已经获得锁，则将状态变量 +1

---

构造函数 (默认为非公平版本)

```java
/**
 * Creates an instance of {@code ReentrantLock}.
 * This is equivalent to using {@code ReentrantLock(false)}.
 */
public ReentrantLock() {
    sync = new NonfairSync();
}

/**
 * Creates an instance of {@code ReentrantLock} with the
 * given fairness policy.
 *
 * @param fair {@code true} if this lock should use a fair ordering policy
 */
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

---

获得锁

* 如果当前线程第一次获得锁，则将锁的状态置为 1
* 如果当前线程已经获得锁，则将锁的状态 +1
* 如果锁被另一个线程持有，那么当前线程休眠，并被调度

```java
/**
 * Acquires the lock.
 *
 * <p>Acquires the lock if it is not held by another thread and returns
 * immediately, setting the lock hold count to one.
 *
 * <p>If the current thread already holds the lock then the hold
 * count is incremented by one and the method returns immediately.
 *
 * <p>If the lock is held by another thread then the
 * current thread becomes disabled for thread scheduling
 * purposes and lies dormant until the lock has been acquired,
 * at which time the lock hold count is set to one.
 */
public void lock() {
    sync.lock();
}
```

---

可被中断地获得锁

```java
/**
 * Acquires the lock unless the current thread is
 * {@linkplain Thread#interrupt interrupted}.
 *
 * <p>Acquires the lock if it is not held by another thread and returns
 * immediately, setting the lock hold count to one.
 *
 * <p>If the current thread already holds this lock then the hold count
 * is incremented by one and the method returns immediately.
 *
 * <p>If the lock is held by another thread then the
 * current thread becomes disabled for thread scheduling
 * purposes and lies dormant until one of two things happens:
 *
 * <ul>
 *
 * <li>The lock is acquired by the current thread; or
 *
 * <li>Some other thread {@linkplain Thread#interrupt interrupts} the
 * current thread.
 *
 * </ul>
 *
 * <p>If the lock is acquired by the current thread then the lock hold
 * count is set to one.
 *
 * <p>If the current thread:
 *
 * <ul>
 *
 * <li>has its interrupted status set on entry to this method; or
 *
 * <li>is {@linkplain Thread#interrupt interrupted} while acquiring
 * the lock,
 *
 * </ul>
 *
 * then {@link InterruptedException} is thrown and the current thread's
 * interrupted status is cleared.
 *
 * <p>In this implementation, as this method is an explicit
 * interruption point, preference is given to responding to the
 * interrupt over normal or reentrant acquisition of the lock.
 *
 * @throws InterruptedException if the current thread is interrupted
 */
public void lockInterruptibly() throws InterruptedException {
    sync.acquireInterruptibly(1);
}
```

---

在调用时，如果锁没有被占用，则立刻占用锁

这个一个破坏公平性的插队操作

```java
/**
 * Acquires the lock only if it is not held by another thread at the time
 * of invocation.
 *
 * <p>Acquires the lock if it is not held by another thread and
 * returns immediately with the value {@code true}, setting the
 * lock hold count to one. Even when this lock has been set to use a
 * fair ordering policy, a call to {@code tryLock()} <em>will</em>
 * immediately acquire the lock if it is available, whether or not
 * other threads are currently waiting for the lock.
 * This &quot;barging&quot; behavior can be useful in certain
 * circumstances, even though it breaks fairness. If you want to honor
 * the fairness setting for this lock, then use
 * {@link #tryLock(long, TimeUnit) tryLock(0, TimeUnit.SECONDS) }
 * which is almost equivalent (it also detects interruption).
 *
 * <p>If the current thread already holds this lock then the hold
 * count is incremented by one and the method returns {@code true}.
 *
 * <p>If the lock is held by another thread then this method will return
 * immediately with the value {@code false}.
 *
 * @return {@code true} if the lock was free and was acquired by the
 *         current thread, or the lock was already held by the current
 *         thread; and {@code false} otherwise
 */
public boolean tryLock() {
    return sync.nonfairTryAcquire(1);
}
```

可被中断或超时版本的占用锁:

```java
/**
 * Acquires the lock if it is not held by another thread within the given
 * waiting time and the current thread has not been
 * {@linkplain Thread#interrupt interrupted}.
 *
 * <p>Acquires the lock if it is not held by another thread and returns
 * immediately with the value {@code true}, setting the lock hold count
 * to one. If this lock has been set to use a fair ordering policy then
 * an available lock <em>will not</em> be acquired if any other threads
 * are waiting for the lock. This is in contrast to the {@link #tryLock()}
 * method. If you want a timed {@code tryLock} that does permit barging on
 * a fair lock then combine the timed and un-timed forms together:
 *
 *  <pre> {@code
 * if (lock.tryLock() ||
 *     lock.tryLock(timeout, unit)) {
 *   ...
 * }}</pre>
 *
 * <p>If the current thread
 * already holds this lock then the hold count is incremented by one and
 * the method returns {@code true}.
 *
 * <p>If the lock is held by another thread then the
 * current thread becomes disabled for thread scheduling
 * purposes and lies dormant until one of three things happens:
 *
 * <ul>
 *
 * <li>The lock is acquired by the current thread; or
 *
 * <li>Some other thread {@linkplain Thread#interrupt interrupts}
 * the current thread; or
 *
 * <li>The specified waiting time elapses
 *
 * </ul>
 *
 * <p>If the lock is acquired then the value {@code true} is returned and
 * the lock hold count is set to one.
 *
 * <p>If the current thread:
 *
 * <ul>
 *
 * <li>has its interrupted status set on entry to this method; or
 *
 * <li>is {@linkplain Thread#interrupt interrupted} while
 * acquiring the lock,
 *
 * </ul>
 * then {@link InterruptedException} is thrown and the current thread's
 * interrupted status is cleared.
 *
 * <p>If the specified waiting time elapses then the value {@code false}
 * is returned.  If the time is less than or equal to zero, the method
 * will not wait at all.
 *
 * <p>In this implementation, as this method is an explicit
 * interruption point, preference is given to responding to the
 * interrupt over normal or reentrant acquisition of the lock, and
 * over reporting the elapse of the waiting time.
 *
 * @param timeout the time to wait for the lock
 * @param unit the time unit of the timeout argument
 * @return {@code true} if the lock was free and was acquired by the
 *         current thread, or the lock was already held by the current
 *         thread; and {@code false} if the waiting time elapsed before
 *         the lock could be acquired
 * @throws InterruptedException if the current thread is interrupted
 * @throws NullPointerException if the time unit is null
 */
public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
```

---

```java
/**
 * Attempts to release this lock.
 *
 * <p>If the current thread is the holder of this lock then the hold
 * count is decremented.  If the hold count is now zero then the lock
 * is released.  If the current thread is not the holder of this
 * lock then {@link IllegalMonitorStateException} is thrown.
 *
 * @throws IllegalMonitorStateException if the current thread does not
 *         hold this lock
 */
public void unlock() {
    sync.release(1);
}
```

如果当前线程占有该锁，那么锁的计数 -1

如果锁的计数已经为 0，那么锁被释放

如果当前线程没有持有锁，则会抛出异常

---

返回当前锁对象所使用的条件对象

```java
/**
 * Returns a {@link Condition} instance for use with this
 * {@link Lock} instance.
 *
 * <p>The returned {@link Condition} instance supports the same
 * usages as do the {@link Object} monitor methods ({@link
 * Object#wait() wait}, {@link Object#notify notify}, and {@link
 * Object#notifyAll notifyAll}) when used with the built-in
 * monitor lock.
 *
 * <ul>
 *
 * <li>If this lock is not held when any of the {@link Condition}
 * {@linkplain Condition#await() waiting} or {@linkplain
 * Condition#signal signalling} methods are called, then an {@link
 * IllegalMonitorStateException} is thrown.
 *
 * <li>When the condition {@linkplain Condition#await() waiting}
 * methods are called the lock is released and, before they
 * return, the lock is reacquired and the lock hold count restored
 * to what it was when the method was called.
 *
 * <li>If a thread is {@linkplain Thread#interrupt interrupted}
 * while waiting then the wait will terminate, an {@link
 * InterruptedException} will be thrown, and the thread's
 * interrupted status will be cleared.
 *
 * <li> Waiting threads are signalled in FIFO order.
 *
 * <li>The ordering of lock reacquisition for threads returning
 * from waiting methods is the same as for threads initially
 * acquiring the lock, which is in the default case not specified,
 * but for <em>fair</em> locks favors those threads that have been
 * waiting the longest.
 *
 * </ul>
 *
 * @return the Condition object
 */
public Condition newCondition() {
    return sync.newCondition();
}
```

---

获得当前锁被持有的次数

> 仅用于调试

```java
/**
 * Queries the number of holds on this lock by the current thread.
 *
 * <p>A thread has a hold on a lock for each lock action that is not
 * matched by an unlock action.
 *
 * <p>The hold count information is typically only used for testing and
 * debugging purposes. For example, if a certain section of code should
 * not be entered with the lock already held then we can assert that
 * fact:
 *
 *  <pre> {@code
 * class X {
 *   ReentrantLock lock = new ReentrantLock();
 *   // ...
 *   public void m() {
 *     assert lock.getHoldCount() == 0;
 *     lock.lock();
 *     try {
 *       // ... method body
 *     } finally {
 *       lock.unlock();
 *     }
 *   }
 * }}</pre>
 *
 * @return the number of holds on this lock by the current thread,
 *         or zero if this lock is not held by the current thread
 */
public int getHoldCount() {
    return sync.getHoldCount();
}
```

---

查询锁是否被当前线程持有

> 也是用于调试和测试

```java
/**
 * Queries if this lock is held by the current thread.
 *
 * <p>Analogous to the {@link Thread#holdsLock(Object)} method for
 * built-in monitor locks, this method is typically used for
 * debugging and testing. For example, a method that should only be
 * called while a lock is held can assert that this is the case:
 *
 *  <pre> {@code
 * class X {
 *   ReentrantLock lock = new ReentrantLock();
 *   // ...
 *
 *   public void m() {
 *       assert lock.isHeldByCurrentThread();
 *       // ... method body
 *   }
 * }}</pre>
 *
 * <p>It can also be used to ensure that a reentrant lock is used
 * in a non-reentrant manner, for example:
 *
 *  <pre> {@code
 * class X {
 *   ReentrantLock lock = new ReentrantLock();
 *   // ...
 *
 *   public void m() {
 *       assert !lock.isHeldByCurrentThread();
 *       lock.lock();
 *       try {
 *           // ... method body
 *       } finally {
 *           lock.unlock();
 *       }
 *   }
 * }}</pre>
 *
 * @return {@code true} if current thread holds this lock and
 *         {@code false} otherwise
 */
public boolean isHeldByCurrentThread() {
    return sync.isHeldExclusively();
}
```

---

查询当前锁是否被线程占有

```java
/**
 * Queries if this lock is held by any thread. This method is
 * designed for use in monitoring of the system state,
 * not for synchronization control.
 *
 * @return {@code true} if any thread holds this lock and
 *         {@code false} otherwise
 */
public boolean isLocked() {
    return sync.isLocked();
}
```

---

查看当前锁是否是公平锁

```java
/**
 * Returns {@code true} if this lock has fairness set true.
 *
 * @return {@code true} if this lock has fairness set true
 */
public final boolean isFair() {
    return sync instanceof FairSync;
}
```

---

取得持有锁的线程的引用

```java
/**
 * Returns the thread that currently owns this lock, or
 * {@code null} if not owned. When this method is called by a
 * thread that is not the owner, the return value reflects a
 * best-effort approximation of current lock status. For example,
 * the owner may be momentarily {@code null} even if there are
 * threads trying to acquire the lock but have not yet done so.
 * This method is designed to facilitate construction of
 * subclasses that provide more extensive lock monitoring
 * facilities.
 *
 * @return the owner, or {@code null} if not owned
 */
protected Thread getOwner() {
    return sync.getOwner();
}
```

---

查询是否有线程正在等待该锁

```java
/**
 * Queries whether any threads are waiting to acquire this lock. Note that
 * because cancellations may occur at any time, a {@code true}
 * return does not guarantee that any other thread will ever
 * acquire this lock.  This method is designed primarily for use in
 * monitoring of the system state.
 *
 * @return {@code true} if there may be other threads waiting to
 *         acquire the lock
 */
public final boolean hasQueuedThreads() {
    return sync.hasQueuedThreads();
}
```

---

查询给定的线程是否正在等待锁

```java
/**
 * Queries whether the given thread is waiting to acquire this
 * lock. Note that because cancellations may occur at any time, a
 * {@code true} return does not guarantee that this thread
 * will ever acquire this lock.  This method is designed primarily for use
 * in monitoring of the system state.
 *
 * @param thread the thread
 * @return {@code true} if the given thread is queued waiting for this lock
 * @throws NullPointerException if the thread is null
 */
public final boolean hasQueuedThread(Thread thread) {
    return sync.isQueued(thread);
}
```

---

获得等待队列的长度

```java
/**
 * Returns an estimate of the number of threads waiting to
 * acquire this lock.  The value is only an estimate because the number of
 * threads may change dynamically while this method traverses
 * internal data structures.  This method is designed for use in
 * monitoring of the system state, not for synchronization
 * control.
 *
 * @return the estimated number of threads waiting for this lock
 */
public final int getQueueLength() {
    return sync.getQueueLength();
}
```

---

获得所有正在等待的线程

```java
/**
 * Returns a collection containing threads that may be waiting to
 * acquire this lock.  Because the actual set of threads may change
 * dynamically while constructing this result, the returned
 * collection is only a best-effort estimate.  The elements of the
 * returned collection are in no particular order.  This method is
 * designed to facilitate construction of subclasses that provide
 * more extensive monitoring facilities.
 *
 * @return the collection of threads
 */
protected Collection<Thread> getQueuedThreads() {
    return sync.getQueuedThreads();
}
```

---

## Summary

_Reentrant_ 是可重入性的意思

体现为同一个线程可以多次持有该锁

当锁计数器为 0 时，线程才可以释放锁

可以看到 ReentrantLock 的占用过程中

先进行 CAS 试图占有锁，如果失败，则加入 AQS 的等待队列

所以 ReentrantLock 是一种悲观锁

因为它是互斥的，所以占用不到锁的线程就只能进入休眠

CAS 仅用于解决 __锁有没有被抓住__ 这一关键问题

### 与 `synchronized` 关键字的区别

* 一个实现于 JVM 中，是关键字；一个实现于 JDK 中，是类
* 线程在等待 ReentrantLock 时可以被中断，而等待 `synchronized` 不行
* ReentrantLock 可以被指定为公平锁
* ReentrantLock 可以绑定多个 Condition 对象
* 在性能上，`synchronized` 关键字在 JVM 中被优化

---

