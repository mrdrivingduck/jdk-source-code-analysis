# Abstract Class - java.util.concurrent.locks.AbstractQueuedSynchronizer

Created by : Mr Dk.

2019 / 12 / 24 17:27

Nanjing, Jiangsu, China

---

## Definition

本类提供了一套框架，表示一个基于 FIFO 同步队列的抽象接口。基于该接口，子类可以实现阻塞锁、信号量等。在接口内部实现中，锁的状态用一个可被原子操作的 `int` 表示，继承自该类的子类需要：

* 提供函数来修改锁的状态
* 定义锁被持有和释放时对应的状态

除此之外，本类的其它部分用于实现排队和阻塞的机制，不实现具体的同步接口。本类支持两种模式：

* 互斥 (exclusive) 模式 - 其它线程的锁获取操作将不会成功
* 共享 (shared) 模式 - 多个线程共享获得锁可能会成功

本类只有在一个地方需要区分这两种情况：在一个线程获得锁后，下一个正在等待的锁是否也可以获得锁。在不同模式下等待的线程共享同一个 FIFO 队列，通常来说，子类实现时只需支持一种模式即可，另一种模式不需实现。

本类提供了对内部队列进行监控的函数。另外，本类的序列化只保存 `int` 锁变量的状态。因此，反序列化将得到空的线程队列。使用该类时，子类需要借助以下三个函数查看或修改同步状态 (即修改锁状态)：

* `getState()`
* `setState()`
* `compareAndSetState()`

这三个函数仅提供了修改锁状态的途径，但由于锁的具体定义不同 (锁状态的不同 bit 代表某种含义)，因此需要子类根据锁的使用方法来实现以下五个函数：

* `tryAcquire()`
* `tryRelease()`
* `tryAcquireShared()`
* `tryReleaseShared()`
* `isHeldExclusively()`

这些函数需要被实现为线程安全、简短、非阻塞。另外，从 `AbstractOwnableSynchronizer` 继承来的信息可以在互斥模式下追踪持有锁的线程。虽然内部实现是队列，但锁的获得不一定遵循 FIFO 的规则。

互斥访问的核心操作。获得锁：

```java
while (!tryAcquire(arg)) {
    // enqueue thread if it is not already queued
    // possibly block current thread
}
```

释放锁：

```java
if (tryRelease(arg))
    // unblock the first queued thread
```

可以看到，入队操作在 `tryAcquire()` 之后调用。在这之间，可能会被别的试图获得锁的线程插队，这是由 CPU 的调度决定的，即所谓的不公平锁。在子类实现 `tryAcquire()` 时，可以在内部调用一定的检查函数，以提供一个 **公平** 的 FIFO 锁获取顺序。默认的不公平获取顺序会有更高的吞吐率，因为刚释放锁的线程非常有可能立刻重新获得锁，不保证严格的入队顺序。

```java
/**
 * Provides a framework for implementing blocking locks and related
 * synchronizers (semaphores, events, etc) that rely on
 * first-in-first-out (FIFO) wait queues.  This class is designed to
 * be a useful basis for most kinds of synchronizers that rely on a
 * single atomic {@code int} value to represent state. Subclasses
 * must define the protected methods that change this state, and which
 * define what that state means in terms of this object being acquired
 * or released.  Given these, the other methods in this class carry
 * out all queuing and blocking mechanics. Subclasses can maintain
 * other state fields, but only the atomically updated {@code int}
 * value manipulated using methods {@link #getState}, {@link
 * #setState} and {@link #compareAndSetState} is tracked with respect
 * to synchronization.
 *
 * <p>Subclasses should be defined as non-public internal helper
 * classes that are used to implement the synchronization properties
 * of their enclosing class.  Class
 * {@code AbstractQueuedSynchronizer} does not implement any
 * synchronization interface.  Instead it defines methods such as
 * {@link #acquireInterruptibly} that can be invoked as
 * appropriate by concrete locks and related synchronizers to
 * implement their public methods.
 *
 * <p>This class supports either or both a default <em>exclusive</em>
 * mode and a <em>shared</em> mode. When acquired in exclusive mode,
 * attempted acquires by other threads cannot succeed. Shared mode
 * acquires by multiple threads may (but need not) succeed. This class
 * does not &quot;understand&quot; these differences except in the
 * mechanical sense that when a shared mode acquire succeeds, the next
 * waiting thread (if one exists) must also determine whether it can
 * acquire as well. Threads waiting in the different modes share the
 * same FIFO queue. Usually, implementation subclasses support only
 * one of these modes, but both can come into play for example in a
 * {@link ReadWriteLock}. Subclasses that support only exclusive or
 * only shared modes need not define the methods supporting the unused mode.
 *
 * <p>This class defines a nested {@link ConditionObject} class that
 * can be used as a {@link Condition} implementation by subclasses
 * supporting exclusive mode for which method {@link
 * #isHeldExclusively} reports whether synchronization is exclusively
 * held with respect to the current thread, method {@link #release}
 * invoked with the current {@link #getState} value fully releases
 * this object, and {@link #acquire}, given this saved state value,
 * eventually restores this object to its previous acquired state.  No
 * {@code AbstractQueuedSynchronizer} method otherwise creates such a
 * condition, so if this constraint cannot be met, do not use it.  The
 * behavior of {@link ConditionObject} depends of course on the
 * semantics of its synchronizer implementation.
 *
 * <p>This class provides inspection, instrumentation, and monitoring
 * methods for the internal queue, as well as similar methods for
 * condition objects. These can be exported as desired into classes
 * using an {@code AbstractQueuedSynchronizer} for their
 * synchronization mechanics.
 *
 * <p>Serialization of this class stores only the underlying atomic
 * integer maintaining state, so deserialized objects have empty
 * thread queues. Typical subclasses requiring serializability will
 * define a {@code readObject} method that restores this to a known
 * initial state upon deserialization.
 *
 * <h3>Usage</h3>
 *
 * <p>To use this class as the basis of a synchronizer, redefine the
 * following methods, as applicable, by inspecting and/or modifying
 * the synchronization state using {@link #getState}, {@link
 * #setState} and/or {@link #compareAndSetState}:
 *
 * <ul>
 * <li> {@link #tryAcquire}
 * <li> {@link #tryRelease}
 * <li> {@link #tryAcquireShared}
 * <li> {@link #tryReleaseShared}
 * <li> {@link #isHeldExclusively}
 * </ul>
 *
 * Each of these methods by default throws {@link
 * UnsupportedOperationException}.  Implementations of these methods
 * must be internally thread-safe, and should in general be short and
 * not block. Defining these methods is the <em>only</em> supported
 * means of using this class. All other methods are declared
 * {@code final} because they cannot be independently varied.
 *
 * <p>You may also find the inherited methods from {@link
 * AbstractOwnableSynchronizer} useful to keep track of the thread
 * owning an exclusive synchronizer.  You are encouraged to use them
 * -- this enables monitoring and diagnostic tools to assist users in
 * determining which threads hold locks.
 *
 * <p>Even though this class is based on an internal FIFO queue, it
 * does not automatically enforce FIFO acquisition policies.  The core
 * of exclusive synchronization takes the form:
 *
 * <pre>
 * Acquire:
 *     while (!tryAcquire(arg)) {
 *        <em>enqueue thread if it is not already queued</em>;
 *        <em>possibly block current thread</em>;
 *     }
 *
 * Release:
 *     if (tryRelease(arg))
 *        <em>unblock the first queued thread</em>;
 * </pre>
 *
 * (Shared mode is similar but may involve cascading signals.)
 *
 * <p id="barging">Because checks in acquire are invoked before
 * enqueuing, a newly acquiring thread may <em>barge</em> ahead of
 * others that are blocked and queued.  However, you can, if desired,
 * define {@code tryAcquire} and/or {@code tryAcquireShared} to
 * disable barging by internally invoking one or more of the inspection
 * methods, thereby providing a <em>fair</em> FIFO acquisition order.
 * In particular, most fair synchronizers can define {@code tryAcquire}
 * to return {@code false} if {@link #hasQueuedPredecessors} (a method
 * specifically designed to be used by fair synchronizers) returns
 * {@code true}.  Other variations are possible.
 *
 * <p>Throughput and scalability are generally highest for the
 * default barging (also known as <em>greedy</em>,
 * <em>renouncement</em>, and <em>convoy-avoidance</em>) strategy.
 * While this is not guaranteed to be fair or starvation-free, earlier
 * queued threads are allowed to recontend before later queued
 * threads, and each recontention has an unbiased chance to succeed
 * against incoming threads.  Also, while acquires do not
 * &quot;spin&quot; in the usual sense, they may perform multiple
 * invocations of {@code tryAcquire} interspersed with other
 * computations before blocking.  This gives most of the benefits of
 * spins when exclusive synchronization is only briefly held, without
 * most of the liabilities when it isn't. If so desired, you can
 * augment this by preceding calls to acquire methods with
 * "fast-path" checks, possibly prechecking {@link #hasContended}
 * and/or {@link #hasQueuedThreads} to only do so if the synchronizer
 * is likely not to be contended.
 *
 * <p>This class provides an efficient and scalable basis for
 * synchronization in part by specializing its range of use to
 * synchronizers that can rely on {@code int} state, acquire, and
 * release parameters, and an internal FIFO wait queue. When this does
 * not suffice, you can build synchronizers from a lower level using
 * {@link java.util.concurrent.atomic atomic} classes, your own custom
 * {@link java.util.Queue} classes, and {@link LockSupport} blocking
 * support.
 *
 * <h3>Usage Examples</h3>
 *
 * <p>Here is a non-reentrant mutual exclusion lock class that uses
 * the value zero to represent the unlocked state, and one to
 * represent the locked state. While a non-reentrant lock
 * does not strictly require recording of the current owner
 * thread, this class does so anyway to make usage easier to monitor.
 * It also supports conditions and exposes
 * one of the instrumentation methods:
 *
 *  <pre> {@code
 * class Mutex implements Lock, java.io.Serializable {
 *
 *   // Our internal helper class
 *   private static class Sync extends AbstractQueuedSynchronizer {
 *     // Reports whether in locked state
 *     protected boolean isHeldExclusively() {
 *       return getState() == 1;
 *     }
 *
 *     // Acquires the lock if state is zero
 *     public boolean tryAcquire(int acquires) {
 *       assert acquires == 1; // Otherwise unused
 *       if (compareAndSetState(0, 1)) {
 *         setExclusiveOwnerThread(Thread.currentThread());
 *         return true;
 *       }
 *       return false;
 *     }
 *
 *     // Releases the lock by setting state to zero
 *     protected boolean tryRelease(int releases) {
 *       assert releases == 1; // Otherwise unused
 *       if (getState() == 0) throw new IllegalMonitorStateException();
 *       setExclusiveOwnerThread(null);
 *       setState(0);
 *       return true;
 *     }
 *
 *     // Provides a Condition
 *     Condition newCondition() { return new ConditionObject(); }
 *
 *     // Deserializes properly
 *     private void readObject(ObjectInputStream s)
 *         throws IOException, ClassNotFoundException {
 *       s.defaultReadObject();
 *       setState(0); // reset to unlocked state
 *     }
 *   }
 *
 *   // The sync object does all the hard work. We just forward to it.
 *   private final Sync sync = new Sync();
 *
 *   public void lock()                { sync.acquire(1); }
 *   public boolean tryLock()          { return sync.tryAcquire(1); }
 *   public void unlock()              { sync.release(1); }
 *   public Condition newCondition()   { return sync.newCondition(); }
 *   public boolean isLocked()         { return sync.isHeldExclusively(); }
 *   public boolean hasQueuedThreads() { return sync.hasQueuedThreads(); }
 *   public void lockInterruptibly() throws InterruptedException {
 *     sync.acquireInterruptibly(1);
 *   }
 *   public boolean tryLock(long timeout, TimeUnit unit)
 *       throws InterruptedException {
 *     return sync.tryAcquireNanos(1, unit.toNanos(timeout));
 *   }
 * }}</pre>
 *
 * <p>Here is a latch class that is like a
 * {@link java.util.concurrent.CountDownLatch CountDownLatch}
 * except that it only requires a single {@code signal} to
 * fire. Because a latch is non-exclusive, it uses the {@code shared}
 * acquire and release methods.
 *
 *  <pre> {@code
 * class BooleanLatch {
 *
 *   private static class Sync extends AbstractQueuedSynchronizer {
 *     boolean isSignalled() { return getState() != 0; }
 *
 *     protected int tryAcquireShared(int ignore) {
 *       return isSignalled() ? 1 : -1;
 *     }
 *
 *     protected boolean tryReleaseShared(int ignore) {
 *       setState(1);
 *       return true;
 *     }
 *   }
 *
 *   private final Sync sync = new Sync();
 *   public boolean isSignalled() { return sync.isSignalled(); }
 *   public void signal()         { sync.releaseShared(1); }
 *   public void await() throws InterruptedException {
 *     sync.acquireSharedInterruptibly(1);
 *   }
 * }}</pre>
 *
 * @since 1.5
 * @author Doug Lea
 */
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
    
}
```

---

## Constructor

```java
private static final long serialVersionUID = 7373984972572414691L;

/**
 * Creates a new {@code AbstractQueuedSynchronizer} instance
 * with initial synchronization state of zero.
 */
protected AbstractQueuedSynchronizer() { }
```

---

## Queue Nodes

内部同步队列中的结点定义，使用了 **CLH 锁队列** 的变种：

* CLH 通常被用于自旋锁
* 在这里也被实现为阻塞锁

内部的 `waitStatus` 维护了结点的线程等待状态：

* `CANCELLED` - 在同步队列中等待的线程等待超时或被中断，需要取消等待
* `SIGNAL` - 后继结点的线程处于等待，当前结点释放同步状态或被取消后，通知后续结点
* `CONDTITION` - 结点位于 **等待队列** 中，在接受到 `signal()` 后进入 **同步队列**
* `PROPAGATE` - 下一次共享同步装填将被无条件传播
* `INITIAL` - 初始状态

在 CLH 插入时，用一个原子的界限来表示一个线程是否已入队。`prev` 指针用于在线程被取消等待时重新链接链表。

```java
/**
 * Wait queue node class.
 *
 * <p>The wait queue is a variant of a "CLH" (Craig, Landin, and
 * Hagersten) lock queue. CLH locks are normally used for
 * spinlocks.  We instead use them for blocking synchronizers, but
 * use the same basic tactic of holding some of the control
 * information about a thread in the predecessor of its node.  A
 * "status" field in each node keeps track of whether a thread
 * should block.  A node is signalled when its predecessor
 * releases.  Each node of the queue otherwise serves as a
 * specific-notification-style monitor holding a single waiting
 * thread. The status field does NOT control whether threads are
 * granted locks etc though.  A thread may try to acquire if it is
 * first in the queue. But being first does not guarantee success;
 * it only gives the right to contend.  So the currently released
 * contender thread may need to rewait.
 *
 * <p>To enqueue into a CLH lock, you atomically splice it in as new
 * tail. To dequeue, you just set the head field.
 * <pre>
 *      +------+  prev +-----+       +-----+
 * head |      | <---- |     | <---- |     |  tail
 *      +------+       +-----+       +-----+
 * </pre>
 *
 * <p>Insertion into a CLH queue requires only a single atomic
 * operation on "tail", so there is a simple atomic point of
 * demarcation from unqueued to queued. Similarly, dequeuing
 * involves only updating the "head". However, it takes a bit
 * more work for nodes to determine who their successors are,
 * in part to deal with possible cancellation due to timeouts
 * and interrupts.
 *
 * <p>The "prev" links (not used in original CLH locks), are mainly
 * needed to handle cancellation. If a node is cancelled, its
 * successor is (normally) relinked to a non-cancelled
 * predecessor. For explanation of similar mechanics in the case
 * of spin locks, see the papers by Scott and Scherer at
 * http://www.cs.rochester.edu/u/scott/synchronization/
 *
 * <p>We also use "next" links to implement blocking mechanics.
 * The thread id for each node is kept in its own node, so a
 * predecessor signals the next node to wake up by traversing
 * next link to determine which thread it is.  Determination of
 * successor must avoid races with newly queued nodes to set
 * the "next" fields of their predecessors.  This is solved
 * when necessary by checking backwards from the atomically
 * updated "tail" when a node's successor appears to be null.
 * (Or, said differently, the next-links are an optimization
 * so that we don't usually need a backward scan.)
 *
 * <p>Cancellation introduces some conservatism to the basic
 * algorithms.  Since we must poll for cancellation of other
 * nodes, we can miss noticing whether a cancelled node is
 * ahead or behind us. This is dealt with by always unparking
 * successors upon cancellation, allowing them to stabilize on
 * a new predecessor, unless we can identify an uncancelled
 * predecessor who will carry this responsibility.
 *
 * <p>CLH queues need a dummy header node to get started. But
 * we don't create them on construction, because it would be wasted
 * effort if there is never contention. Instead, the node
 * is constructed and head and tail pointers are set upon first
 * contention.
 *
 * <p>Threads waiting on Conditions use the same nodes, but
 * use an additional link. Conditions only need to link nodes
 * in simple (non-concurrent) linked queues because they are
 * only accessed when exclusively held.  Upon await, a node is
 * inserted into a condition queue.  Upon signal, the node is
 * transferred to the main queue.  A special value of status
 * field is used to mark which queue a node is on.
 *
 * <p>Thanks go to Dave Dice, Mark Moir, Victor Luchangco, Bill
 * Scherer and Michael Scott, along with members of JSR-166
 * expert group, for helpful ideas, discussions, and critiques
 * on the design of this class.
 */
static final class Node {
    /** Marker to indicate a node is waiting in shared mode */
    static final Node SHARED = new Node();
    /** Marker to indicate a node is waiting in exclusive mode */
    static final Node EXCLUSIVE = null;

    /** waitStatus value to indicate thread has cancelled */
    static final int CANCELLED =  1;
    /** waitStatus value to indicate successor's thread needs unparking */
    static final int SIGNAL    = -1;
    /** waitStatus value to indicate thread is waiting on condition */
    static final int CONDITION = -2;
    /**
     * waitStatus value to indicate the next acquireShared should
     * unconditionally propagate
     */
    static final int PROPAGATE = -3;

    /**
     * Status field, taking on only the values:
     *   SIGNAL:     The successor of this node is (or will soon be)
     *               blocked (via park), so the current node must
     *               unpark its successor when it releases or
     *               cancels. To avoid races, acquire methods must
     *               first indicate they need a signal,
     *               then retry the atomic acquire, and then,
     *               on failure, block.
     *   CANCELLED:  This node is cancelled due to timeout or interrupt.
     *               Nodes never leave this state. In particular,
     *               a thread with cancelled node never again blocks.
     *   CONDITION:  This node is currently on a condition queue.
     *               It will not be used as a sync queue node
     *               until transferred, at which time the status
     *               will be set to 0. (Use of this value here has
     *               nothing to do with the other uses of the
     *               field, but simplifies mechanics.)
     *   PROPAGATE:  A releaseShared should be propagated to other
     *               nodes. This is set (for head node only) in
     *               doReleaseShared to ensure propagation
     *               continues, even if other operations have
     *               since intervened.
     *   0:          None of the above
     *
     * The values are arranged numerically to simplify use.
     * Non-negative values mean that a node doesn't need to
     * signal. So, most code doesn't need to check for particular
     * values, just for sign.
     *
     * The field is initialized to 0 for normal sync nodes, and
     * CONDITION for condition nodes.  It is modified using CAS
     * (or when possible, unconditional volatile writes).
     */
    volatile int waitStatus;

    /**
     * Link to predecessor node that current node/thread relies on
     * for checking waitStatus. Assigned during enqueuing, and nulled
     * out (for sake of GC) only upon dequeuing.  Also, upon
     * cancellation of a predecessor, we short-circuit while
     * finding a non-cancelled one, which will always exist
     * because the head node is never cancelled: A node becomes
     * head only as a result of successful acquire. A
     * cancelled thread never succeeds in acquiring, and a thread only
     * cancels itself, not any other node.
     */
    volatile Node prev;

    /**
     * Link to the successor node that the current node/thread
     * unparks upon release. Assigned during enqueuing, adjusted
     * when bypassing cancelled predecessors, and nulled out (for
     * sake of GC) when dequeued.  The enq operation does not
     * assign next field of a predecessor until after attachment,
     * so seeing a null next field does not necessarily mean that
     * node is at end of queue. However, if a next field appears
     * to be null, we can scan prev's from the tail to
     * double-check.  The next field of cancelled nodes is set to
     * point to the node itself instead of null, to make life
     * easier for isOnSyncQueue.
     */
    volatile Node next;

    /**
     * The thread that enqueued this node.  Initialized on
     * construction and nulled out after use.
     */
    volatile Thread thread;

    /**
     * Link to next node waiting on condition, or the special
     * value SHARED.  Because condition queues are accessed only
     * when holding in exclusive mode, we just need a simple
     * linked queue to hold nodes while they are waiting on
     * conditions. They are then transferred to the queue to
     * re-acquire. And because conditions can only be exclusive,
     * we save a field by using special value to indicate shared
     * mode.
     */
    Node nextWaiter;

    /**
     * Returns true if node is waiting in shared mode.
     */
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    /**
     * Returns previous node, or throws NullPointerException if null.
     * Use when predecessor cannot be null.  The null check could
     * be elided, but is present to help the VM.
     *
     * @return the predecessor of this node
     */
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null)
            throw new NullPointerException();
        else
            return p;
    }

    Node() {    // Used to establish initial head or SHARED marker
    }

    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) { // Used by Condition
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

---

## Queue

维护同步队列的头尾指针，以及同步状态 (锁状态) 变量。当线程获取同步状态失败时，会将当前线程以及等待状态封装在结点中，然后加入队列并阻塞线程。当同步状态释放后，队列中的头结点会被唤醒，使其再次尝试获取同步状态。

```java
/**
 * Head of the wait queue, lazily initialized.  Except for
 * initialization, it is modified only via method setHead.  Note:
 * If head exists, its waitStatus is guaranteed not to be
 * CANCELLED.
 */
private transient volatile Node head;

/**
 * Tail of the wait queue, lazily initialized.  Modified only via
 * method enq to add new wait node.
 */
private transient volatile Node tail;

/**
 * The synchronization state.
 */
private volatile int state;
```

以下三个函数可以修改锁状态。当可能有多个线程修改锁状态时，需要使用线程安全的 CAS 函数；如果确定只有一个线程修改锁状态，那么直接使用普通的 `get()` / `set()` 即可。

```java
/**
 * Returns the current value of synchronization state.
 * This operation has memory semantics of a {@code volatile} read.
 * @return current state value
 */
protected final int getState() {
    return state;
}

/**
 * Sets the value of synchronization state.
 * This operation has memory semantics of a {@code volatile} write.
 * @param newState the new state value
 */
protected final void setState(int newState) {
    state = newState;
}

/**
 * Atomically sets synchronization state to the given updated
 * value if the current state value equals the expected value.
 * This operation has memory semantics of a {@code volatile} read
 * and write.
 *
 * @param expect the expected value
 * @param update the new value
 * @return {@code true} if successful. False return indicates that the actual
 *         value was not equal to the expected value.
 */
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

加入队列的过程需要保证线程安全，因此使用 CAS 的方式设置头尾结点。

```java
/**
 * Inserts node into queue, initializing if necessary. See picture above.
 * @param node the node to insert
 * @return node's predecessor
 */
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}

/**
 * Creates and enqueues node for current thread and given mode.
 *
 * @param mode Node.EXCLUSIVE for exclusive, Node.SHARED for shared
 * @return the new node
 */
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}
```

同步队列遵循 FIFO，头结点是 **成功获取同步状态** 的结点，头结点在释放同步状态时，唤醒后继结点。以下函数获取了当前结点的 `next`，然后调用 `LockSupport` 的 `unpark()` 函数唤醒后继结点线程。

```java
/**
 * Wakes up node's successor, if one exists.
 *
 * @param node the node
 */
private void unparkSuccessor(Node node) {
    /*
     * If status is negative (i.e., possibly needing signal) try
     * to clear in anticipation of signalling.  It is OK if this
     * fails or if status is changed by waiting thread.
     */
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    /*
     * Thread to unpark is held in successor, which is normally
     * just the next node.  But if cancelled or apparently null,
     * traverse backwards from tail to find the actual
     * non-cancelled successor.
     */
    Node s = node.next;
    if (s == null || s.waitStatus > 0) {
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

## Exclusive Mode

### Acquire

以下函数首先试图以线程安全的方式尝试获取同步状态，如果获取失败，则构造 `EXCLUSIVE` 模式的结点并加入同步队列尾部，最终线程以死循环的方式获取同步状态。如果无法获取到，则阻塞结点中的线程。被阻塞线程只能通过以下两种方式唤醒：

* 前驱结点出队，唤醒当前结点
* 线程被中断

当线程已经从这个函数中返回时，代表线程已经获取到了同步状态。

```java
/**
 * Acquires in exclusive mode, ignoring interrupts.  Implemented
 * by invoking at least once {@link #tryAcquire},
 * returning on success.  Otherwise the thread is queued, possibly
 * repeatedly blocking and unblocking, invoking {@link
 * #tryAcquire} until success.  This method can be used
 * to implement method {@link Lock#lock}.
 *
 * @param arg the acquire argument.  This value is conveyed to
 *        {@link #tryAcquire} but is otherwise uninterpreted and
 *        can represent anything you like.
 */
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

结点进入同步队列后，就开始自旋，不断尝试获取同步状态。只有前驱结点是头结点时，才能尝试获取同步状态：

1. 因为头结点线程释放同步状态后才会唤醒后继结点
2. 维护队列的 FIFO 原则

如果前驱结点是头结点，并且当前结点成功获取到了同步状态，那么就可以把头结点设为自己，并断开前驱结点的 `next` 引用。从而使前驱结点出队，当前结点成功获取到同步状态并成为队头。这里操作 `head` 指针不需要保证线程安全，因为只有当前线程会操作 `head` 指针。

```java
/**
 * Acquires in exclusive uninterruptible mode for thread already in
 * queue. Used by condition wait methods as well as acquire.
 *
 * @param node the node
 * @param arg the acquire argument
 * @return {@code true} if interrupted while waiting
 */
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}

/**
  * Sets head of queue to be node, thus dequeuing. Called only by
  * acquire methods.  Also nulls out unused fields for sake of GC
  * and to suppress unnecessary signals and traversals.
  *
  * @param node the node
  */
private void setHead(Node node) {
    head = node;
    node.thread = null;
    node.prev = null;
}
```

### Release

当线程释放同步状态后，会通过 `unparkSuccessor()` 唤醒后继结点。

```java
/**
 * Releases in exclusive mode.  Implemented by unblocking one or
 * more threads if {@link #tryRelease} returns true.
 * This method can be used to implement method {@link Lock#unlock}.
 *
 * @param arg the release argument.  This value is conveyed to
 *        {@link #tryRelease} but is otherwise uninterpreted and
 *        can represent anything you like.
 * @return the value returned from {@link #tryRelease}
 */
public final boolean release(int arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}

/**
 * Attempts to set the state to reflect a release in exclusive
 * mode.
 *
 * <p>This method is always invoked by the thread performing release.
 *
 * <p>The default implementation throws
 * {@link UnsupportedOperationException}.
 *
 * @param arg the release argument. This value is always the one
 *        passed to a release method, or the current state value upon
 *        entry to a condition wait.  The value is otherwise
 *        uninterpreted and can represent anything you like.
 * @return {@code true} if this object is now in a fully released
 *         state, so that any waiting threads may attempt to acquire;
 *         and {@code false} otherwise.
 * @throws IllegalMonitorStateException if releasing would place this
 *         synchronizer in an illegal state. This exception must be
 *         thrown in a consistent fashion for synchronization to work
 *         correctly.
 * @throws UnsupportedOperationException if exclusive mode is not supported
 */
protected boolean tryRelease(int arg) {
    throw new UnsupportedOperationException();
}
```

## Shared Mode

共享模式与独占模式的区别是，同一时刻是否能有多个线程同时获取到同步状态。

### Acquire

首先调用用户自行实现的 `tryAcquireShared()` 函数试图获取同步状态，返回值 `≥ 0` 意味着成功获取到了同步状态，否则说明获取同步状态失败，进入自旋等待。等待的过程依旧是，当前驱结点是头结点时，试图获取同步状态，如果成功，则从自旋状态下退出。

```java
/**
 * Acquires in shared mode, ignoring interrupts.  Implemented by
 * first invoking at least once {@link #tryAcquireShared},
 * returning on success.  Otherwise the thread is queued, possibly
 * repeatedly blocking and unblocking, invoking {@link
 * #tryAcquireShared} until success.
 *
 * @param arg the acquire argument.  This value is conveyed to
 *        {@link #tryAcquireShared} but is otherwise uninterpreted
 *        and can represent anything you like.
 */
public final void acquireShared(int arg) {
    if (tryAcquireShared(arg) < 0)
        doAcquireShared(arg);
}

/**
 * Acquires in shared uninterruptible mode.
 * @param arg the acquire argument
 */
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head) {
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

### Release

共享模式的锁释放操作：

* 通知后继结点
* 保证锁释放的状态要传递下去

```java
/**
 * Releases in shared mode.  Implemented by unblocking one or more
 * threads if {@link #tryReleaseShared} returns true.
 *
 * @param arg the release argument.  This value is conveyed to
 *        {@link #tryReleaseShared} but is otherwise uninterpreted
 *        and can represent anything you like.
 * @return the value returned from {@link #tryReleaseShared}
 */
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}

/**
 * Release action for shared mode -- signals successor and ensures
 * propagation. (Note: For exclusive mode, release just amounts
 * to calling unparkSuccessor of head if it needs signal.)
 */
private void doReleaseShared() {
    /*
     * Ensure that a release propagates, even if there are other
     * in-progress acquires/releases.  This proceeds in the usual
     * way of trying to unparkSuccessor of head if it needs
     * signal. But if it does not, status is set to PROPAGATE to
     * ensure that upon release, propagation continues.
     * Additionally, we must loop in case a new node is added
     * while we are doing this. Also, unlike other uses of
     * unparkSuccessor, we need to know if CAS to reset status
     * fails, if so rechecking.
     */
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                        !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        if (h == head)                   // loop if head changed
            break;
    }
}
```

## Exclusive Timeout Mode

### Acquire

如果能在指定时间段内获得同步状态则返回 `true`，否则返回 `false`。其本质在于计算剩余需要睡眠的时间间隔 `nanosTimeout`。每当线程被中断意外唤醒时，都需要重新计算剩余的睡眠间隔：`nanosTimeout -= now - lastTime`。如果睡眠间隔大于 `0` 表示还未超时，否则说明已经超时。

在一个死循环中，每次试图获取同步状态。如果没能获取成功，就统计一下剩余睡眠时间并判断是否超时。当剩余睡眠时间 `≤ spinForTImeoutTHreshold` (1000ns) 时，线程将不会重新等待，而是直接自旋等待。因为剩下的计时无法做到精确。

```java
/**
 * Acquires in exclusive timed mode.
 *
 * @param arg the acquire argument
 * @param nanosTimeout max wait time
 * @return {@code true} if acquired
 */
private boolean doAcquireNanos(int arg, long nanosTimeout)
    throws InterruptedException {
    if (nanosTimeout <= 0L)
        return false;
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return true;
            }
            nanosTimeout = deadline - System.nanoTime();
            if (nanosTimeout <= 0L)
                return false;
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

---

## Summary

大致看懂了层次，但是对于队列的一些具体操作没看懂，还是直接看具体的锁的实现类吧。

该类的主要作用在于维护了内部等待队列，实现了所有的队列操作，并暴露 5 个函数需要被子类重写。这 5 个函数定义了修改锁状态的行为：

* `tryAcquire()`
* `tryRelease()`
* `tryAcquireShared()`
* `tryReleaseShared()`
* `isHeldExclusively()`

因为对于这个类来说，它只抽象地知道锁是一个 `int` 类型的原子变量，并不知道其中每一位具体的含义是什么。这是由实现了具体锁的子类定义的。比如，`int` 中的某一位代表该锁是否被持有...

在子类实现的函数修改锁状态后，该类中的函数再对内部等待队列进行相应的操作。相当于这个类已经实现了同步队列操作的大致框架，但是有几个核心部分的函数需要由用户来自行实现。用户用不同方式实现后，就能使用各种不同功能的锁。

---

