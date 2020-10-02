# Class - java.util.concurrent.locks.ReentrantReadWriteLock

Created by : Mr Dk.

2020 / 01 / 09 22:04

Nanjing, Jiangsu, China

---

## Definition

```java
public class ReentrantReadWriteLock
        implements ReadWriteLock, java.io.Serializable {

}
```

读写锁的实现类。同时支持 `ReentrantLock` 的类似功能。在锁的获得顺序上，也有 **公平** 和 **非公平** 两个概念：一个长期被竞争的非公平锁将使读写线程无限延后，但会比公平锁有更好的吞吐率；公平锁保证了竞争锁的先后顺序，当锁被释放后：

* 等待时间最久的写者线程获得锁
* 比等待时间最久的写者线程还要久的一组读者线程获得锁

当写锁被持有，或有写线程正在等待时，试图获取公平读锁 (不可重入) 的线程将会被阻塞，直到最老的写线程获得并释放锁后才能得到读锁。除非正在等待中的写线程放弃等待。只有当读锁和写锁同时空闲时，一个写者线程才能获得公平写锁 (不可重入)。

### 重入性

所有的写锁都释放后，才允许读者使用读锁

## 锁降级

允许写锁降级为读锁 - 先获取写锁，再获取读锁，最后释放写锁，但是读锁不可能升级为写锁。

### Condition 支持

只针对写锁。

### Instrumentation

本类支持一些判断锁是否正被占有和竞争的函数。用于监控系统状态，而不是同步控制。示例用法:

```java
class CachedData {
    Object data;
    volatile boolean cacheValid;
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    void processCachedData() {
        rwl.readLock().lock();
        if (!cacheValid) {
            // Must release read lock before acquiring write lock
            rwl.readLock().unlock();
            rwl.writeLock().lock();
            try {
                // Recheck state because another thread might have
                // acquired write lock and changed state before we did.
                if (!cacheValid) {
                    data = ...
                    cacheValid = true;
                }
                // Downgrade by acquiring read lock before releasing write lock
                rwl.readLock().lock();
            } finally {
                rwl.writeLock().unlock(); // Unlock write, still hold read
            }
        }
        try {
            use(data);
        } finally {
            rwl.readLock().unlock();
        }
    }
}
```

本类可被用于提升一些集合的并发性，特别是集合很大，会被更多的读线程访问时。该锁最多支持 65535 次递归占用写锁和 65535 次递归占用读锁，超出这个数目会发生错误。

```java
/**
 * An implementation of {@link ReadWriteLock} supporting similar
 * semantics to {@link ReentrantLock}.
 * <p>This class has the following properties:
 *
 * <ul>
 * <li><b>Acquisition order</b>
 *
 * <p>This class does not impose a reader or writer preference
 * ordering for lock access.  However, it does support an optional
 * <em>fairness</em> policy.
 *
 * <dl>
 * <dt><b><i>Non-fair mode (default)</i></b>
 * <dd>When constructed as non-fair (the default), the order of entry
 * to the read and write lock is unspecified, subject to reentrancy
 * constraints.  A nonfair lock that is continuously contended may
 * indefinitely postpone one or more reader or writer threads, but
 * will normally have higher throughput than a fair lock.
 *
 * <dt><b><i>Fair mode</i></b>
 * <dd>When constructed as fair, threads contend for entry using an
 * approximately arrival-order policy. When the currently held lock
 * is released, either the longest-waiting single writer thread will
 * be assigned the write lock, or if there is a group of reader threads
 * waiting longer than all waiting writer threads, that group will be
 * assigned the read lock.
 *
 * <p>A thread that tries to acquire a fair read lock (non-reentrantly)
 * will block if either the write lock is held, or there is a waiting
 * writer thread. The thread will not acquire the read lock until
 * after the oldest currently waiting writer thread has acquired and
 * released the write lock. Of course, if a waiting writer abandons
 * its wait, leaving one or more reader threads as the longest waiters
 * in the queue with the write lock free, then those readers will be
 * assigned the read lock.
 *
 * <p>A thread that tries to acquire a fair write lock (non-reentrantly)
 * will block unless both the read lock and write lock are free (which
 * implies there are no waiting threads).  (Note that the non-blocking
 * {@link ReadLock#tryLock()} and {@link WriteLock#tryLock()} methods
 * do not honor this fair setting and will immediately acquire the lock
 * if it is possible, regardless of waiting threads.)
 * <p>
 * </dl>
 *
 * <li><b>Reentrancy</b>
 *
 * <p>This lock allows both readers and writers to reacquire read or
 * write locks in the style of a {@link ReentrantLock}. Non-reentrant
 * readers are not allowed until all write locks held by the writing
 * thread have been released.
 *
 * <p>Additionally, a writer can acquire the read lock, but not
 * vice-versa.  Among other applications, reentrancy can be useful
 * when write locks are held during calls or callbacks to methods that
 * perform reads under read locks.  If a reader tries to acquire the
 * write lock it will never succeed.
 *
 * <li><b>Lock downgrading</b>
 * <p>Reentrancy also allows downgrading from the write lock to a read lock,
 * by acquiring the write lock, then the read lock and then releasing the
 * write lock. However, upgrading from a read lock to the write lock is
 * <b>not</b> possible.
 *
 * <li><b>Interruption of lock acquisition</b>
 * <p>The read lock and write lock both support interruption during lock
 * acquisition.
 *
 * <li><b>{@link Condition} support</b>
 * <p>The write lock provides a {@link Condition} implementation that
 * behaves in the same way, with respect to the write lock, as the
 * {@link Condition} implementation provided by
 * {@link ReentrantLock#newCondition} does for {@link ReentrantLock}.
 * This {@link Condition} can, of course, only be used with the write lock.
 *
 * <p>The read lock does not support a {@link Condition} and
 * {@code readLock().newCondition()} throws
 * {@code UnsupportedOperationException}.
 *
 * <li><b>Instrumentation</b>
 * <p>This class supports methods to determine whether locks
 * are held or contended. These methods are designed for monitoring
 * system state, not for synchronization control.
 * </ul>
 *
 * <p>Serialization of this class behaves in the same way as built-in
 * locks: a deserialized lock is in the unlocked state, regardless of
 * its state when serialized.
 *
 * <p><b>Sample usages</b>. Here is a code sketch showing how to perform
 * lock downgrading after updating a cache (exception handling is
 * particularly tricky when handling multiple locks in a non-nested
 * fashion):
 *
 * <pre> {@code
 * class CachedData {
 *   Object data;
 *   volatile boolean cacheValid;
 *   final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
 *
 *   void processCachedData() {
 *     rwl.readLock().lock();
 *     if (!cacheValid) {
 *       // Must release read lock before acquiring write lock
 *       rwl.readLock().unlock();
 *       rwl.writeLock().lock();
 *       try {
 *         // Recheck state because another thread might have
 *         // acquired write lock and changed state before we did.
 *         if (!cacheValid) {
 *           data = ...
 *           cacheValid = true;
 *         }
 *         // Downgrade by acquiring read lock before releasing write lock
 *         rwl.readLock().lock();
 *       } finally {
 *         rwl.writeLock().unlock(); // Unlock write, still hold read
 *       }
 *     }
 *
 *     try {
 *       use(data);
 *     } finally {
 *       rwl.readLock().unlock();
 *     }
 *   }
 * }}</pre>
 *
 * ReentrantReadWriteLocks can be used to improve concurrency in some
 * uses of some kinds of Collections. This is typically worthwhile
 * only when the collections are expected to be large, accessed by
 * more reader threads than writer threads, and entail operations with
 * overhead that outweighs synchronization overhead. For example, here
 * is a class using a TreeMap that is expected to be large and
 * concurrently accessed.
 *
 *  <pre> {@code
 * class RWDictionary {
 *   private final Map<String, Data> m = new TreeMap<String, Data>();
 *   private final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
 *   private final Lock r = rwl.readLock();
 *   private final Lock w = rwl.writeLock();
 *
 *   public Data get(String key) {
 *     r.lock();
 *     try { return m.get(key); }
 *     finally { r.unlock(); }
 *   }
 *   public String[] allKeys() {
 *     r.lock();
 *     try { return m.keySet().toArray(); }
 *     finally { r.unlock(); }
 *   }
 *   public Data put(String key, Data value) {
 *     w.lock();
 *     try { return m.put(key, value); }
 *     finally { w.unlock(); }
 *   }
 *   public void clear() {
 *     w.lock();
 *     try { m.clear(); }
 *     finally { w.unlock(); }
 *   }
 * }}</pre>
 *
 * <h3>Implementation Notes</h3>
 *
 * <p>This lock supports a maximum of 65535 recursive write locks
 * and 65535 read locks. Attempts to exceed these limits result in
 * {@link Error} throws from locking methods.
 *
 * @since 1.5
 * @author Doug Lea
 */
```

---

```java
private static final long serialVersionUID = -6992448646407690164L;
/** Inner class providing readlock */
private final ReentrantReadWriteLock.ReadLock readerLock;
/** Inner class providing writelock */
private final ReentrantReadWriteLock.WriteLock writerLock;
/** Performs all synchronization mechanics */
final Sync sync;
```

内部维护一个读锁一个写锁。有意思的是，读锁和写锁的数据类型居然是本身。

---

构造函数 (默认将被构造为非公平锁)。根据是否是公平锁，实例化内部的 AQS。

```java
/**
 * Creates a new {@code ReentrantReadWriteLock} with
 * default (nonfair) ordering properties.
 */
public ReentrantReadWriteLock() {
    this(false);
}

/**
 * Creates a new {@code ReentrantReadWriteLock} with
 * the given fairness policy.
 *
 * @param fair {@code true} if this lock should use a fair ordering policy
 */
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```

---

获取读锁写锁对象：

```java
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
```

---

AQS 的抽象类定义:

```java
/**
 * Synchronization implementation for ReentrantReadWriteLock.
 * Subclassed into fair and nonfair versions.
 */
abstract static class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 6317671515068378041L;

}
```

公平版本和非公平版本的 AQS 共有的函数在该类中实现。抽象只针对其中的两个函数：

* 公平锁与非公平锁使用相同的代码对锁进行释放
* 区别在于当队列非空时是否插队
* 下面两个函数指明，在公平或非公平策略下，读者线程或写者线程是否应该被阻塞

```java
/**
 * Returns true if the current thread, when trying to acquire
 * the read lock, and otherwise eligible to do so, should block
 * because of policy for overtaking other waiting threads.
 */
abstract boolean readerShouldBlock();

/**
 * Returns true if the current thread, when trying to acquire
 * the write lock, and otherwise eligible to do so, should block
 * because of policy for overtaking other waiting threads.
 */
abstract boolean writerShouldBlock();
```

这两个函数在之后分别被实现。非公平版本：

* 写者肯定可以插队
* 读者理论上也可以插队，但是需要避免写者陷入无限的饥饿中
* 如果线程临时出现在队列头部 (插队的)，则阻塞

```java
/**
 * Nonfair version of Sync
 */
static final class NonfairSync extends Sync {
    private static final long serialVersionUID = -8159625535654395037L;
    final boolean writerShouldBlock() {
        return false; // writers can always barge
    }
    final boolean readerShouldBlock() {
        /* As a heuristic to avoid indefinite writer starvation,
         * block if the thread that momentarily appears to be head
         * of queue, if one exists, is a waiting writer.  This is
         * only a probabilistic effect since a new reader will not
         * block if there is a waiting writer behind other enabled
         * readers that have not yet drained from the queue.
         */
        return apparentlyFirstQueuedIsExclusive();
    }
}
```

公平版本：策略很简单，只要等待队列中还有别的线程，你就给我阻塞然后乖乖排队。

```java
/**
 * Fair version of Sync
 */
static final class FairSync extends Sync {
    private static final long serialVersionUID = -2274990926593161451L;
    final boolean writerShouldBlock() {
        return hasQueuedPredecessors();
    }
    final boolean readerShouldBlock() {
        return hasQueuedPredecessors();
    }
}
```

---

首先是用于锁计数的常量和函数。锁的状态被分为两个 `unsigned short` 变量：

* 低 16-bit 被用于互斥的写锁重入次数
* 高 16-bit 被用于共享的读锁持有次数

```java
/*
 * Read vs write count extraction constants and functions.
 * Lock state is logically divided into two unsigned shorts:
 * The lower one representing the exclusive (writer) lock hold count,
 * and the upper the shared (reader) hold count.
 */

static final int SHARED_SHIFT   = 16;
static final int SHARED_UNIT    = (1 << SHARED_SHIFT);
static final int MAX_COUNT      = (1 << SHARED_SHIFT) - 1;
static final int EXCLUSIVE_MASK = (1 << SHARED_SHIFT) - 1;

/** Returns the number of shared holds represented in count  */
static int sharedCount(int c)    { return c >>> SHARED_SHIFT; }
/** Returns the number of exclusive holds represented in count  */
static int exclusiveCount(int c) { return c & EXCLUSIVE_MASK; }
```

无符号右移 16-bit 后，相当于只保留了高 16-bit，而 `EXCLUSIVE_MASK` 为 1 左移 16-bit 再减 1，则低 16-bit 全为 1，从而能通过与运算选出低 16-bit。

---

```java
/**
 * A counter for per-thread read hold counts.
 * Maintained as a ThreadLocal; cached in cachedHoldCounter
 */
static final class HoldCounter {
    int count = 0;
    // Use id, not reference, to avoid garbage retention
    final long tid = getThreadId(Thread.currentThread());
}

/**
 * ThreadLocal subclass. Easiest to explicitly define for sake
 * of deserialization mechanics.
 */
static final class ThreadLocalHoldCounter
    extends ThreadLocal<HoldCounter> {
    public HoldCounter initialValue() {
        return new HoldCounter();
    }
}

/**
 * The number of reentrant read locks held by current thread.
 * Initialized only in constructor and readObject.
 * Removed whenever a thread's read hold count drops to 0.
 */
private transient ThreadLocalHoldCounter readHolds;

/**
 * The hold count of the last thread to successfully acquire
 * readLock. This saves ThreadLocal lookup in the common case
 * where the next thread to release is the last one to
 * acquire. This is non-volatile since it is just used
 * as a heuristic, and would be great for threads to cache.
 *
 * <p>Can outlive the Thread for which it is caching the read
 * hold count, but avoids garbage retention by not retaining a
 * reference to the Thread.
 *
 * <p>Accessed via a benign data race; relies on the memory
 * model's final field and out-of-thin-air guarantees.
 */
private transient HoldCounter cachedHoldCounter;

/**
 * firstReader is the first thread to have acquired the read lock.
 * firstReaderHoldCount is firstReader's hold count.
 *
 * <p>More precisely, firstReader is the unique thread that last
 * changed the shared count from 0 to 1, and has not released the
 * read lock since then; null if there is no such thread.
 *
 * <p>Cannot cause garbage retention unless the thread terminated
 * without relinquishing its read locks, since tryReleaseShared
 * sets it to null.
 *
 * <p>Accessed via a benign data race; relies on the memory
 * model's out-of-thin-air guarantees for references.
 *
 * <p>This allows tracking of read holds for uncontended read
 * locks to be very cheap.
 */
private transient Thread firstReader = null;
private transient int firstReaderHoldCount;

Sync() {
    readHolds = new ThreadLocalHoldCounter();
    setState(getState()); // ensures visibility of readHolds
}
```

> 没看懂... 😥

---

释放互斥写锁：

* 计算释放后的锁状态，并更新
* 如果释放后锁已空闲，则设置互斥写锁为空闲

```java
protected final boolean tryRelease(int releases) {
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    int nextc = getState() - releases;
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    setState(nextc);
    return free;
}
```

获得互斥写锁

1. 如果读锁非零 (正在被别的线程读取) 或写锁非零 (别的线程占有写锁)，则失败
2. 如果锁技术已经饱和，则失败
3. 否则，线程可以获得锁，更新锁的状态并设置锁的持有人

```java
protected final boolean tryAcquire(int acquires) {
    /*
     * Walkthrough:
     * 1. If read count nonzero or write count nonzero
     *    and owner is a different thread, fail.
     * 2. If count would saturate, fail. (This can only
     *    happen if count is already nonzero.)
     * 3. Otherwise, this thread is eligible for lock if
     *    it is either a reentrant acquire or
     *    queue policy allows it. If so, update state
     *    and set owner.
     */
    Thread current = Thread.currentThread();
    int c = getState();
    int w = exclusiveCount(c);
    if (c != 0) {
        // (Note: if c != 0 and w == 0 then shared count != 0)
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // Reentrant acquire
        setState(c + acquires);
        return true;
    }
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
```

释放共享读锁：

* *清理线程的读锁重入计数*
* 在死循环中通过 CAS 操作将读锁的计数 -1

```java
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        // assert firstReaderHoldCount > 0;
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            // Releasing the read lock has no effect on readers,
            // but it may allow waiting writers to proceed if
            // both read and write locks are now free.
            return nextc == 0;
    }
}
```

获得共享读锁：

1. 如果写锁正被其它线程占有，则失败 (但如果是当前线程持有写锁，则可以获取读锁)
2. 根据公平策略决定是否可以获取读锁，或判断读锁获取次数是否超过限制
3. 如果获取读锁失败，则进入循环尝试

```java
protected final int tryAcquireShared(int unused) {
    /*
     * Walkthrough:
     * 1. If write lock held by another thread, fail.
     * 2. Otherwise, this thread is eligible for
     *    lock wrt state, so ask if it should block
     *    because of queue policy. If not, try
     *    to grant by CASing state and updating count.
     *    Note that step does not check for reentrant
     *    acquires, which is postponed to full version
     *    to avoid having to check hold count in
     *    the more typical non-reentrant case.
     * 3. If step 2 fails either because thread
     *    apparently not eligible or CAS fails or count
     *    saturated, chain to version with full retry loop.
     */
    Thread current = Thread.currentThread();
    int c = getState();
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    int r = sharedCount(c);
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        if (r == 0) {
            firstReader = current;
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            firstReaderHoldCount++;
        } else {
            HoldCounter rh = cachedHoldCounter;
            if (rh == null || rh.tid != getThreadId(current))
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                readHolds.set(rh);
            rh.count++;
        }
        return 1;
    }
    return fullTryAcquireShared(current);
}

/**
 * Full version of acquire for reads, that handles CAS misses
 * and reentrant reads not dealt with in tryAcquireShared.
 */
final int fullTryAcquireShared(Thread current) {
    /*
     * This code is in part redundant with that in
     * tryAcquireShared but is simpler overall by not
     * complicating tryAcquireShared with interactions between
     * retries and lazily reading hold counts.
     */
    HoldCounter rh = null;
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0) {
            if (getExclusiveOwnerThread() != current)
                return -1;
            // else we hold the exclusive lock; blocking here
            // would cause deadlock.
        } else if (readerShouldBlock()) {
            // Make sure we're not acquiring read lock reentrantly
            if (firstReader == current) {
                // assert firstReaderHoldCount > 0;
            } else {
                if (rh == null) {
                    rh = cachedHoldCounter;
                    if (rh == null || rh.tid != getThreadId(current)) {
                        rh = readHolds.get();
                        if (rh.count == 0)
                            readHolds.remove();
                    }
                }
                if (rh.count == 0)
                    return -1;
            }
        }
        if (sharedCount(c) == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (sharedCount(c) == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                if (rh == null)
                    rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
                cachedHoldCounter = rh; // cache for release
            }
            return 1;
        }
    }
}
```

---

以下两个函数用于绕开公平策略，试图通过插队来获得读锁和写锁。若写锁已被占有，或写锁重入次数超过限制，则返回失败；否则就通过一次 CAS 操作尝试获得写锁，如果成功，就设置锁的持有线程。对于读锁，则在一个死循环中不断尝试 CAS 直到成功或失败。

```java
/**
 * Performs tryLock for write, enabling barging in both modes.
 * This is identical in effect to tryAcquire except for lack
 * of calls to writerShouldBlock.
 */
final boolean tryWriteLock() {
    Thread current = Thread.currentThread();
    int c = getState();
    if (c != 0) {
        int w = exclusiveCount(c);
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        if (w == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
    }
    if (!compareAndSetState(c, c + 1))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}

/**
 * Performs tryLock for read, enabling barging in both modes.
 * This is identical in effect to tryAcquireShared except for
 * lack of calls to readerShouldBlock.
 */
final boolean tryReadLock() {
    Thread current = Thread.currentThread();
    for (;;) {
        int c = getState();
        if (exclusiveCount(c) != 0 &&
            getExclusiveOwnerThread() != current)
            return false;
        int r = sharedCount(c);
        if (r == MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        if (compareAndSetState(c, c + SHARED_UNIT)) {
            if (r == 0) {
                firstReader = current;
                firstReaderHoldCount = 1;
            } else if (firstReader == current) {
                firstReaderHoldCount++;
            } else {
                HoldCounter rh = cachedHoldCounter;
                if (rh == null || rh.tid != getThreadId(current))
                    cachedHoldCounter = rh = readHolds.get();
                else if (rh.count == 0)
                    readHolds.set(rh);
                rh.count++;
            }
            return true;
        }
    }
}
```

---

以下是查看锁状态的一些函数。主要是查询锁的持有者，读锁和写锁的计数：

```java
protected final boolean isHeldExclusively() {
    // While we must in general read state before owner,
    // we don't need to do so to check if current thread is owner
    return getExclusiveOwnerThread() == Thread.currentThread();
}

// Methods relayed to outer class

final ConditionObject newCondition() {
    return new ConditionObject();
}

final Thread getOwner() {
    // Must read state before owner to ensure memory consistency
    return ((exclusiveCount(getState()) == 0) ?
            null :
            getExclusiveOwnerThread());
}

final int getReadLockCount() {
    return sharedCount(getState());
}

final boolean isWriteLocked() {
    return exclusiveCount(getState()) != 0;
}

final int getWriteHoldCount() {
    return isHeldExclusively() ? exclusiveCount(getState()) : 0;
}

final int getReadHoldCount() {
    if (getReadLockCount() == 0)
        return 0;

    Thread current = Thread.currentThread();
    if (firstReader == current)
        return firstReaderHoldCount;

    HoldCounter rh = cachedHoldCounter;
    if (rh != null && rh.tid == getThreadId(current))
        return rh.count;

    int count = readHolds.get().count;
    if (count == 0) readHolds.remove();
    return count;
}

/**
 * Reconstitutes the instance from a stream (that is, deserializes it).
 */
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    s.defaultReadObject();
    readHolds = new ThreadLocalHoldCounter();
    setState(0); // reset to unlocked state
}

final int getCount() { return getState(); }
```

---

读锁的具体实现。获取和释放读锁时，调用 AQS 的共享锁策略。参数全部为 1 (获取 1 或释放 1)。

```java
/**
 * The lock returned by method {@link ReentrantReadWriteLock#readLock}.
 */
public static class ReadLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = -5992448646407690164L;
    private final Sync sync;

    /**
     * Constructor for use by subclasses
     *
     * @param lock the outer lock object
     * @throws NullPointerException if the lock is null
     */
    protected ReadLock(ReentrantReadWriteLock lock) {
        sync = lock.sync;
    }

    /**
     * Acquires the read lock.
     *
     * <p>Acquires the read lock if the write lock is not held by
     * another thread and returns immediately.
     *
     * <p>If the write lock is held by another thread then
     * the current thread becomes disabled for thread scheduling
     * purposes and lies dormant until the read lock has been acquired.
     */
    public void lock() {
        sync.acquireShared(1);
    }

    /**
     * Acquires the read lock unless the current thread is
     * {@linkplain Thread#interrupt interrupted}.
     *
     * <p>Acquires the read lock if the write lock is not held
     * by another thread and returns immediately.
     *
     * <p>If the write lock is held by another thread then the
     * current thread becomes disabled for thread scheduling
     * purposes and lies dormant until one of two things happens:
     *
     * <ul>
     *
     * <li>The read lock is acquired by the current thread; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread.
     *
     * </ul>
     *
     * <p>If the current thread:
     *
     * <ul>
     *
     * <li>has its interrupted status set on entry to this method; or
     *
     * <li>is {@linkplain Thread#interrupt interrupted} while
     * acquiring the read lock,
     *
     * </ul>
     *
     * then {@link InterruptedException} is thrown and the current
     * thread's interrupted status is cleared.
     *
     * <p>In this implementation, as this method is an explicit
     * interruption point, preference is given to responding to
     * the interrupt over normal or reentrant acquisition of the
     * lock.
     *
     * @throws InterruptedException if the current thread is interrupted
     */
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }

    /**
     * Acquires the read lock only if the write lock is not held by
     * another thread at the time of invocation.
     *
     * <p>Acquires the read lock if the write lock is not held by
     * another thread and returns immediately with the value
     * {@code true}. Even when this lock has been set to use a
     * fair ordering policy, a call to {@code tryLock()}
     * <em>will</em> immediately acquire the read lock if it is
     * available, whether or not other threads are currently
     * waiting for the read lock.  This &quot;barging&quot; behavior
     * can be useful in certain circumstances, even though it
     * breaks fairness. If you want to honor the fairness setting
     * for this lock, then use {@link #tryLock(long, TimeUnit)
     * tryLock(0, TimeUnit.SECONDS) } which is almost equivalent
     * (it also detects interruption).
     *
     * <p>If the write lock is held by another thread then
     * this method will return immediately with the value
     * {@code false}.
     *
     * @return {@code true} if the read lock was acquired
     */
    public boolean tryLock() {
        return sync.tryReadLock();
    }

    /**
     * Acquires the read lock if the write lock is not held by
     * another thread within the given waiting time and the
     * current thread has not been {@linkplain Thread#interrupt
     * interrupted}.
     *
     * <p>Acquires the read lock if the write lock is not held by
     * another thread and returns immediately with the value
     * {@code true}. If this lock has been set to use a fair
     * ordering policy then an available lock <em>will not</em> be
     * acquired if any other threads are waiting for the
     * lock. This is in contrast to the {@link #tryLock()}
     * method. If you want a timed {@code tryLock} that does
     * permit barging on a fair lock then combine the timed and
     * un-timed forms together:
     *
     *  <pre> {@code
     * if (lock.tryLock() ||
     *     lock.tryLock(timeout, unit)) {
     *   ...
     * }}</pre>
     *
     * <p>If the write lock is held by another thread then the
     * current thread becomes disabled for thread scheduling
     * purposes and lies dormant until one of three things happens:
     *
     * <ul>
     *
     * <li>The read lock is acquired by the current thread; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The specified waiting time elapses.
     *
     * </ul>
     *
     * <p>If the read lock is acquired then the value {@code true} is
     * returned.
     *
     * <p>If the current thread:
     *
     * <ul>
     *
     * <li>has its interrupted status set on entry to this method; or
     *
     * <li>is {@linkplain Thread#interrupt interrupted} while
     * acquiring the read lock,
     *
     * </ul> then {@link InterruptedException} is thrown and the
     * current thread's interrupted status is cleared.
     *
     * <p>If the specified waiting time elapses then the value
     * {@code false} is returned.  If the time is less than or
     * equal to zero, the method will not wait at all.
     *
     * <p>In this implementation, as this method is an explicit
     * interruption point, preference is given to responding to
     * the interrupt over normal or reentrant acquisition of the
     * lock, and over reporting the elapse of the waiting time.
     *
     * @param timeout the time to wait for the read lock
     * @param unit the time unit of the timeout argument
     * @return {@code true} if the read lock was acquired
     * @throws InterruptedException if the current thread is interrupted
     * @throws NullPointerException if the time unit is null
     */
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

    /**
     * Attempts to release this lock.
     *
     * <p>If the number of readers is now zero then the lock
     * is made available for write lock attempts.
     */
    public void unlock() {
        sync.releaseShared(1);
    }

    /**
     * Throws {@code UnsupportedOperationException} because
     * {@code ReadLocks} do not support conditions.
     *
     * @throws UnsupportedOperationException always
     */
    public Condition newCondition() {
        throw new UnsupportedOperationException();
    }

    /**
     * Returns a string identifying this lock, as well as its lock state.
     * The state, in brackets, includes the String {@code "Read locks ="}
     * followed by the number of held read locks.
     *
     * @return a string identifying this lock, as well as its lock state
     */
    public String toString() {
        int r = sync.getReadLockCount();
        return super.toString() +
            "[Read locks = " + r + "]";
    }
}
```

写锁的具体实现。获取和释放写锁时，调用 AQS 的互斥锁策略，参数也全部为 1。

```java
/**
 * The lock returned by method {@link ReentrantReadWriteLock#writeLock}.
 */
public static class WriteLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = -4992448646407690164L;
    private final Sync sync;

    /**
     * Constructor for use by subclasses
     *
     * @param lock the outer lock object
     * @throws NullPointerException if the lock is null
     */
    protected WriteLock(ReentrantReadWriteLock lock) {
        sync = lock.sync;
    }

    /**
     * Acquires the write lock.
     *
     * <p>Acquires the write lock if neither the read nor write lock
     * are held by another thread
     * and returns immediately, setting the write lock hold count to
     * one.
     *
     * <p>If the current thread already holds the write lock then the
     * hold count is incremented by one and the method returns
     * immediately.
     *
     * <p>If the lock is held by another thread then the current
     * thread becomes disabled for thread scheduling purposes and
     * lies dormant until the write lock has been acquired, at which
     * time the write lock hold count is set to one.
     */
    public void lock() {
        sync.acquire(1);
    }

    /**
     * Acquires the write lock unless the current thread is
     * {@linkplain Thread#interrupt interrupted}.
     *
     * <p>Acquires the write lock if neither the read nor write lock
     * are held by another thread
     * and returns immediately, setting the write lock hold count to
     * one.
     *
     * <p>If the current thread already holds this lock then the
     * hold count is incremented by one and the method returns
     * immediately.
     *
     * <p>If the lock is held by another thread then the current
     * thread becomes disabled for thread scheduling purposes and
     * lies dormant until one of two things happens:
     *
     * <ul>
     *
     * <li>The write lock is acquired by the current thread; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread.
     *
     * </ul>
     *
     * <p>If the write lock is acquired by the current thread then the
     * lock hold count is set to one.
     *
     * <p>If the current thread:
     *
     * <ul>
     *
     * <li>has its interrupted status set on entry to this method;
     * or
     *
     * <li>is {@linkplain Thread#interrupt interrupted} while
     * acquiring the write lock,
     *
     * </ul>
     *
     * then {@link InterruptedException} is thrown and the current
     * thread's interrupted status is cleared.
     *
     * <p>In this implementation, as this method is an explicit
     * interruption point, preference is given to responding to
     * the interrupt over normal or reentrant acquisition of the
     * lock.
     *
     * @throws InterruptedException if the current thread is interrupted
     */
    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    /**
     * Acquires the write lock only if it is not held by another thread
     * at the time of invocation.
     *
     * <p>Acquires the write lock if neither the read nor write lock
     * are held by another thread
     * and returns immediately with the value {@code true},
     * setting the write lock hold count to one. Even when this lock has
     * been set to use a fair ordering policy, a call to
     * {@code tryLock()} <em>will</em> immediately acquire the
     * lock if it is available, whether or not other threads are
     * currently waiting for the write lock.  This &quot;barging&quot;
     * behavior can be useful in certain circumstances, even
     * though it breaks fairness. If you want to honor the
     * fairness setting for this lock, then use {@link
     * #tryLock(long, TimeUnit) tryLock(0, TimeUnit.SECONDS) }
     * which is almost equivalent (it also detects interruption).
     *
     * <p>If the current thread already holds this lock then the
     * hold count is incremented by one and the method returns
     * {@code true}.
     *
     * <p>If the lock is held by another thread then this method
     * will return immediately with the value {@code false}.
     *
     * @return {@code true} if the lock was free and was acquired
     * by the current thread, or the write lock was already held
     * by the current thread; and {@code false} otherwise.
     */
    public boolean tryLock( ) {
        return sync.tryWriteLock();
    }

    /**
     * Acquires the write lock if it is not held by another thread
     * within the given waiting time and the current thread has
     * not been {@linkplain Thread#interrupt interrupted}.
     *
     * <p>Acquires the write lock if neither the read nor write lock
     * are held by another thread
     * and returns immediately with the value {@code true},
     * setting the write lock hold count to one. If this lock has been
     * set to use a fair ordering policy then an available lock
     * <em>will not</em> be acquired if any other threads are
     * waiting for the write lock. This is in contrast to the {@link
     * #tryLock()} method. If you want a timed {@code tryLock}
     * that does permit barging on a fair lock then combine the
     * timed and un-timed forms together:
     *
     *  <pre> {@code
     * if (lock.tryLock() ||
     *     lock.tryLock(timeout, unit)) {
     *   ...
     * }}</pre>
     *
     * <p>If the current thread already holds this lock then the
     * hold count is incremented by one and the method returns
     * {@code true}.
     *
     * <p>If the lock is held by another thread then the current
     * thread becomes disabled for thread scheduling purposes and
     * lies dormant until one of three things happens:
     *
     * <ul>
     *
     * <li>The write lock is acquired by the current thread; or
     *
     * <li>Some other thread {@linkplain Thread#interrupt interrupts}
     * the current thread; or
     *
     * <li>The specified waiting time elapses
     *
     * </ul>
     *
     * <p>If the write lock is acquired then the value {@code true} is
     * returned and the write lock hold count is set to one.
     *
     * <p>If the current thread:
     *
     * <ul>
     *
     * <li>has its interrupted status set on entry to this method;
     * or
     *
     * <li>is {@linkplain Thread#interrupt interrupted} while
     * acquiring the write lock,
     *
     * </ul>
     *
     * then {@link InterruptedException} is thrown and the current
     * thread's interrupted status is cleared.
     *
     * <p>If the specified waiting time elapses then the value
     * {@code false} is returned.  If the time is less than or
     * equal to zero, the method will not wait at all.
     *
     * <p>In this implementation, as this method is an explicit
     * interruption point, preference is given to responding to
     * the interrupt over normal or reentrant acquisition of the
     * lock, and over reporting the elapse of the waiting time.
     *
     * @param timeout the time to wait for the write lock
     * @param unit the time unit of the timeout argument
     *
     * @return {@code true} if the lock was free and was acquired
     * by the current thread, or the write lock was already held by the
     * current thread; and {@code false} if the waiting time
     * elapsed before the lock could be acquired.
     *
     * @throws InterruptedException if the current thread is interrupted
     * @throws NullPointerException if the time unit is null
     */
    public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }

    /**
     * Attempts to release this lock.
     *
     * <p>If the current thread is the holder of this lock then
     * the hold count is decremented. If the hold count is now
     * zero then the lock is released.  If the current thread is
     * not the holder of this lock then {@link
     * IllegalMonitorStateException} is thrown.
     *
     * @throws IllegalMonitorStateException if the current thread does not
     * hold this lock
     */
    public void unlock() {
        sync.release(1);
    }

    /**
     * Returns a {@link Condition} instance for use with this
     * {@link Lock} instance.
     * <p>The returned {@link Condition} instance supports the same
     * usages as do the {@link Object} monitor methods ({@link
     * Object#wait() wait}, {@link Object#notify notify}, and {@link
     * Object#notifyAll notifyAll}) when used with the built-in
     * monitor lock.
     *
     * <ul>
     *
     * <li>If this write lock is not held when any {@link
     * Condition} method is called then an {@link
     * IllegalMonitorStateException} is thrown.  (Read locks are
     * held independently of write locks, so are not checked or
     * affected. However it is essentially always an error to
     * invoke a condition waiting method when the current thread
     * has also acquired read locks, since other threads that
     * could unblock it will not be able to acquire the write
     * lock.)
     *
     * <li>When the condition {@linkplain Condition#await() waiting}
     * methods are called the write lock is released and, before
     * they return, the write lock is reacquired and the lock hold
     * count restored to what it was when the method was called.
     *
     * <li>If a thread is {@linkplain Thread#interrupt interrupted} while
     * waiting then the wait will terminate, an {@link
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

    /**
     * Returns a string identifying this lock, as well as its lock
     * state.  The state, in brackets includes either the String
     * {@code "Unlocked"} or the String {@code "Locked by"}
     * followed by the {@linkplain Thread#getName name} of the owning thread.
     *
     * @return a string identifying this lock, as well as its lock state
     */
    public String toString() {
        Thread o = sync.getOwner();
        return super.toString() + ((o == null) ?
                                    "[Unlocked]" :
                                    "[Locked by thread " + o.getName() + "]");
    }

    /**
     * Queries if this write lock is held by the current thread.
     * Identical in effect to {@link
     * ReentrantReadWriteLock#isWriteLockedByCurrentThread}.
     *
     * @return {@code true} if the current thread holds this lock and
     *         {@code false} otherwise
     * @since 1.6
     */
    public boolean isHeldByCurrentThread() {
        return sync.isHeldExclusively();
    }

    /**
     * Queries the number of holds on this write lock by the current
     * thread.  A thread has a hold on a lock for each lock action
     * that is not matched by an unlock action.  Identical in effect
     * to {@link ReentrantReadWriteLock#getWriteHoldCount}.
     *
     * @return the number of holds on this lock by the current thread,
     *         or zero if this lock is not held by the current thread
     * @since 1.6
     */
    public int getHoldCount() {
        return sync.getWriteHoldCount();
    }
}
```

---

剩下的是一些和锁的状态有关的函数。比如，锁是否公平，锁的持有者，持有次数，以及 AQS 中队列的 metadata，如队列等待线程数，以及这些相关信息的组合条件。

```java
// Instrumentation and status

/**
 * Returns {@code true} if this lock has fairness set true.
 *
 * @return {@code true} if this lock has fairness set true
 */
public final boolean isFair() {
    return sync instanceof FairSync;
}

/**
 * Returns the thread that currently owns the write lock, or
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

/**
 * Queries the number of read locks held for this lock. This
 * method is designed for use in monitoring system state, not for
 * synchronization control.
 * @return the number of read locks held
 */
public int getReadLockCount() {
    return sync.getReadLockCount();
}

/**
 * Queries if the write lock is held by any thread. This method is
 * designed for use in monitoring system state, not for
 * synchronization control.
 *
 * @return {@code true} if any thread holds the write lock and
 *         {@code false} otherwise
 */
public boolean isWriteLocked() {
    return sync.isWriteLocked();
}

/**
 * Queries if the write lock is held by the current thread.
 *
 * @return {@code true} if the current thread holds the write lock and
 *         {@code false} otherwise
 */
public boolean isWriteLockedByCurrentThread() {
    return sync.isHeldExclusively();
}

/**
 * Queries the number of reentrant write holds on this lock by the
 * current thread.  A writer thread has a hold on a lock for
 * each lock action that is not matched by an unlock action.
 *
 * @return the number of holds on the write lock by the current thread,
 *         or zero if the write lock is not held by the current thread
 */
public int getWriteHoldCount() {
    return sync.getWriteHoldCount();
}

/**
 * Queries the number of reentrant read holds on this lock by the
 * current thread.  A reader thread has a hold on a lock for
 * each lock action that is not matched by an unlock action.
 *
 * @return the number of holds on the read lock by the current thread,
 *         or zero if the read lock is not held by the current thread
 * @since 1.6
 */
public int getReadHoldCount() {
    return sync.getReadHoldCount();
}

/**
 * Returns a collection containing threads that may be waiting to
 * acquire the write lock.  Because the actual set of threads may
 * change dynamically while constructing this result, the returned
 * collection is only a best-effort estimate.  The elements of the
 * returned collection are in no particular order.  This method is
 * designed to facilitate construction of subclasses that provide
 * more extensive lock monitoring facilities.
 *
 * @return the collection of threads
 */
protected Collection<Thread> getQueuedWriterThreads() {
    return sync.getExclusiveQueuedThreads();
}

/**
 * Returns a collection containing threads that may be waiting to
 * acquire the read lock.  Because the actual set of threads may
 * change dynamically while constructing this result, the returned
 * collection is only a best-effort estimate.  The elements of the
 * returned collection are in no particular order.  This method is
 * designed to facilitate construction of subclasses that provide
 * more extensive lock monitoring facilities.
 *
 * @return the collection of threads
 */
protected Collection<Thread> getQueuedReaderThreads() {
    return sync.getSharedQueuedThreads();
}

/**
 * Queries whether any threads are waiting to acquire the read or
 * write lock. Note that because cancellations may occur at any
 * time, a {@code true} return does not guarantee that any other
 * thread will ever acquire a lock.  This method is designed
 * primarily for use in monitoring of the system state.
 *
 * @return {@code true} if there may be other threads waiting to
 *         acquire the lock
 */
public final boolean hasQueuedThreads() {
    return sync.hasQueuedThreads();
}

/**
 * Queries whether the given thread is waiting to acquire either
 * the read or write lock. Note that because cancellations may
 * occur at any time, a {@code true} return does not guarantee
 * that this thread will ever acquire a lock.  This method is
 * designed primarily for use in monitoring of the system state.
 *
 * @param thread the thread
 * @return {@code true} if the given thread is queued waiting for this lock
 * @throws NullPointerException if the thread is null
 */
public final boolean hasQueuedThread(Thread thread) {
    return sync.isQueued(thread);
}

/**
 * Returns an estimate of the number of threads waiting to acquire
 * either the read or write lock.  The value is only an estimate
 * because the number of threads may change dynamically while this
 * method traverses internal data structures.  This method is
 * designed for use in monitoring of the system state, not for
 * synchronization control.
 *
 * @return the estimated number of threads waiting for this lock
 */
public final int getQueueLength() {
    return sync.getQueueLength();
}

/**
 * Returns a collection containing threads that may be waiting to
 * acquire either the read or write lock.  Because the actual set
 * of threads may change dynamically while constructing this
 * result, the returned collection is only a best-effort estimate.
 * The elements of the returned collection are in no particular
 * order.  This method is designed to facilitate construction of
 * subclasses that provide more extensive monitoring facilities.
 *
 * @return the collection of threads
 */
protected Collection<Thread> getQueuedThreads() {
    return sync.getQueuedThreads();
}

/**
 * Queries whether any threads are waiting on the given condition
 * associated with the write lock. Note that because timeouts and
 * interrupts may occur at any time, a {@code true} return does
 * not guarantee that a future {@code signal} will awaken any
 * threads.  This method is designed primarily for use in
 * monitoring of the system state.
 *
 * @param condition the condition
 * @return {@code true} if there are any waiting threads
 * @throws IllegalMonitorStateException if this lock is not held
 * @throws IllegalArgumentException if the given condition is
 *         not associated with this lock
 * @throws NullPointerException if the condition is null
 */
public boolean hasWaiters(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.hasWaiters((AbstractQueuedSynchronizer.ConditionObject)condition);
}

/**
 * Returns an estimate of the number of threads waiting on the
 * given condition associated with the write lock. Note that because
 * timeouts and interrupts may occur at any time, the estimate
 * serves only as an upper bound on the actual number of waiters.
 * This method is designed for use in monitoring of the system
 * state, not for synchronization control.
 *
 * @param condition the condition
 * @return the estimated number of waiting threads
 * @throws IllegalMonitorStateException if this lock is not held
 * @throws IllegalArgumentException if the given condition is
 *         not associated with this lock
 * @throws NullPointerException if the condition is null
 */
public int getWaitQueueLength(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.getWaitQueueLength((AbstractQueuedSynchronizer.ConditionObject)condition);
}

/**
 * Returns a collection containing those threads that may be
 * waiting on the given condition associated with the write lock.
 * Because the actual set of threads may change dynamically while
 * constructing this result, the returned collection is only a
 * best-effort estimate. The elements of the returned collection
 * are in no particular order.  This method is designed to
 * facilitate construction of subclasses that provide more
 * extensive condition monitoring facilities.
 *
 * @param condition the condition
 * @return the collection of threads
 * @throws IllegalMonitorStateException if this lock is not held
 * @throws IllegalArgumentException if the given condition is
 *         not associated with this lock
 * @throws NullPointerException if the condition is null
 */
protected Collection<Thread> getWaitingThreads(Condition condition) {
    if (condition == null)
        throw new NullPointerException();
    if (!(condition instanceof AbstractQueuedSynchronizer.ConditionObject))
        throw new IllegalArgumentException("not owner");
    return sync.getWaitingThreads((AbstractQueuedSynchronizer.ConditionObject)condition);
}
```

---

用于打印锁的状态：

* 写锁的重入次数
* 读锁的共享次数

```java
/**
 * Returns a string identifying this lock, as well as its lock state.
 * The state, in brackets, includes the String {@code "Write locks ="}
 * followed by the number of reentrantly held write locks, and the
 * String {@code "Read locks ="} followed by the number of held
 * read locks.
 *
 * @return a string identifying this lock, as well as its lock state
 */
public String toString() {
    int c = sync.getCount();
    int w = Sync.exclusiveCount(c);
    int r = Sync.sharedCount(c);

    return super.toString() +
        "[Write locks = " + w + ", Read locks = " + r + "]";
}
```

---

JVM 的内部方法，用于获取给定线程的线程 id。

```java
/**
 * Returns the thread id for the given thread.  We must access
 * this directly rather than via method Thread.getId() because
 * getId() is not final, and has been known to be overridden in
 * ways that do not preserve unique mappings.
 */
static final long getThreadId(Thread thread) {
    return UNSAFE.getLongVolatile(thread, TID_OFFSET);
}

// Unsafe mechanics
private static final sun.misc.Unsafe UNSAFE;
private static final long TID_OFFSET;
static {
    try {
        UNSAFE = sun.misc.Unsafe.getUnsafe();
        Class<?> tk = Thread.class;
        TID_OFFSET = UNSAFE.objectFieldOffset
            (tk.getDeclaredField("tid"));
    } catch (Exception e) {
        throw new Error(e);
    }
}
```

---

## Summary

有关于 ReentrantReadWriteLock 的特性：锁对象在内部维护一个公平或非公平的 AQS。

AQS 的原子状态变量被切分成高 16-bit 和低 16-bit，分别表示读锁状态和写锁状态。读锁以共享的策略访问，写锁以互斥的策略访问。获得锁之前，要对原子状态变量中读写锁的状态进行判断：

* 写锁状态维护了某个线程持有写锁后，重入该锁的次数
* 读锁状态维护了所有线程获取读锁次数的总和
* 每个线程获得的读锁次数保存在 `ThreadLocal` 中，由线程自身维护

---

