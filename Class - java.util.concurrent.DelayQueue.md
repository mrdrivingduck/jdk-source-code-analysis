# Class - java.util.concurrent.DelayQueue

Created by : Mr Dk.

2020 / 10 / 25 15:24

Nanjing, Jiangsu, China

---

## Definition

延时阻塞队列的实现。所谓的延时是指，队列中的元素只有在指定的时间过去了以后才能被取出；所谓的阻塞是指，从队列中取出 / 放入元素时，如果队列为空 / 满，那么出队 / 入队操作将一直阻塞直到队列中有元素 / 有空位为止。

```java
/**
 * An unbounded {@linkplain BlockingQueue blocking queue} of
 * {@code Delayed} elements, in which an element can only be taken
 * when its delay has expired.  The <em>head</em> of the queue is that
 * {@code Delayed} element whose delay expired furthest in the
 * past.  If no delay has expired there is no head and {@code poll}
 * will return {@code null}. Expiration occurs when an element's
 * {@code getDelay(TimeUnit.NANOSECONDS)} method returns a value less
 * than or equal to zero.  Even though unexpired elements cannot be
 * removed using {@code take} or {@code poll}, they are otherwise
 * treated as normal elements. For example, the {@code size} method
 * returns the count of both expired and unexpired elements.
 * This queue does not permit null elements.
 *
 * <p>This class and its iterator implement all of the
 * <em>optional</em> methods of the {@link Collection} and {@link
 * Iterator} interfaces.  The Iterator provided in method {@link
 * #iterator()} is <em>not</em> guaranteed to traverse the elements of
 * the DelayQueue in any particular order.
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <E> the type of elements held in this collection
 */
public class DelayQueue<E extends Delayed> extends AbstractQueue<E>
    implements BlockingQueue<E> {

}
```

## Delayed

可以看到放入该队列的元素需要继承 `Delayed` 接口，而 `Delayed` 接口又继承自 `Comparable` 接口。所以能够放入该队列的元素必须要实现以下两个函数：

* `Delayed` 接口的 `getDelay()` 函数 - 负责获取超时的时间
* `Comparable` 接口的 `compareTo()` 函数负责按超时的时间进行优先级排序

```java
/**
 * Compares this object with the specified object for order.  Returns a
 * negative integer, zero, or a positive integer as this object is less
 * than, equal to, or greater than the specified object.
 *
 * <p>The implementor must ensure <tt>sgn(x.compareTo(y)) ==
 * -sgn(y.compareTo(x))</tt> for all <tt>x</tt> and <tt>y</tt>.  (This
 * implies that <tt>x.compareTo(y)</tt> must throw an exception iff
 * <tt>y.compareTo(x)</tt> throws an exception.)
 *
 * <p>The implementor must also ensure that the relation is transitive:
 * <tt>(x.compareTo(y)&gt;0 &amp;&amp; y.compareTo(z)&gt;0)</tt> implies
 * <tt>x.compareTo(z)&gt;0</tt>.
 *
 * <p>Finally, the implementor must ensure that <tt>x.compareTo(y)==0</tt>
 * implies that <tt>sgn(x.compareTo(z)) == sgn(y.compareTo(z))</tt>, for
 * all <tt>z</tt>.
 *
 * <p>It is strongly recommended, but <i>not</i> strictly required that
 * <tt>(x.compareTo(y)==0) == (x.equals(y))</tt>.  Generally speaking, any
 * class that implements the <tt>Comparable</tt> interface and violates
 * this condition should clearly indicate this fact.  The recommended
 * language is "Note: this class has a natural ordering that is
 * inconsistent with equals."
 *
 * <p>In the foregoing description, the notation
 * <tt>sgn(</tt><i>expression</i><tt>)</tt> designates the mathematical
 * <i>signum</i> function, which is defined to return one of <tt>-1</tt>,
 * <tt>0</tt>, or <tt>1</tt> according to whether the value of
 * <i>expression</i> is negative, zero or positive.
 *
 * @param   o the object to be compared.
 * @return  a negative integer, zero, or a positive integer as this object
 *          is less than, equal to, or greater than the specified object.
 *
 * @throws NullPointerException if the specified object is null
 * @throws ClassCastException if the specified object's type prevents it
 *         from being compared to this object.
 */
public int compareTo(T o);
```

```java
/**
 * Returns the remaining delay associated with this object, in the
 * given time unit.
 *
 * @param unit the time unit
 * @return the remaining delay; zero or negative values indicate
 * that the delay has already elapsed
 */
long getDelay(TimeUnit unit);
```

## Fields

类的内部使用可重入锁来实现并发控制，维护了一个优先队列。优先队列的实现之前已经看过了，基于一个 `Object[]` 数组实现了二叉堆，并随着元素的增多会扩展容量。所以延时阻塞队列的入队操作实际上永远不会阻塞。

```java
private final transient ReentrantLock lock = new ReentrantLock();
private final PriorityQueue<E> q = new PriorityQueue<E>();

/**
 * Thread designated to wait for the element at the head of
 * the queue.  This variant of the Leader-Follower pattern
 * (http://www.cs.wustl.edu/~schmidt/POSA/POSA2/) serves to
 * minimize unnecessary timed waiting.  When a thread becomes
 * the leader, it waits only for the next delay to elapse, but
 * other threads await indefinitely.  The leader thread must
 * signal some other thread before returning from take() or
 * poll(...), unless some other thread becomes leader in the
 * interim.  Whenever the head of the queue is replaced with
 * an element with an earlier expiration time, the leader
 * field is invalidated by being reset to null, and some
 * waiting thread, but not necessarily the current leader, is
 * signalled.  So waiting threads must be prepared to acquire
 * and lose leadership while waiting.
 */
private Thread leader = null;

/**
 * Condition signalled when a newer element becomes available
 * at the head of the queue or a new thread may need to
 * become leader.
 */
private final Condition available = lock.newCondition();
```

这里的线程指针 `leader` 遵循一种 *Leader-Follower* 模式，大致意思是，只有一个 leader 线程会等待下一个出队元素 (距离超时最近的元素)，其它线程作为 follower 将进入无限期的等待中。当 leader 线程从 `take()` 或 `poll()` 返回后，将从 follower 线程中选择一个线程作为新的 leader。

---

直接开始由构造函数开始一步一步分析。由于可重入锁和优先队列已经构造完毕，所以空构造函数不做任何事。带参数的构造函数会依次将一个集合中的元素初始化到优先队列中。

```java
/**
 * Creates a new {@code DelayQueue} that is initially empty.
 */
public DelayQueue() {}

/**
 * Creates a {@code DelayQueue} initially containing the elements of the
 * given collection of {@link Delayed} instances.
 *
 * @param c the collection of elements to initially contain
 * @throws NullPointerException if the specified collection or any
 *         of its elements are null
 */
public DelayQueue(Collection<? extends E> c) {
    this.addAll(c);
}
```

那么看看 `addAll()` 是怎么实现的。通过迭代参数集合，分别对每一个元素调用 `add()`。如果成功进行了至少一次 `add()` 操作，就会返回 `true` (即集合发生了修改)。

```java
/**
 * Adds all of the elements in the specified collection to this
 * queue.  Attempts to addAll of a queue to itself result in
 * <tt>IllegalArgumentException</tt>. Further, the behavior of
 * this operation is undefined if the specified collection is
 * modified while the operation is in progress.
 *
 * <p>This implementation iterates over the specified collection,
 * and adds each element returned by the iterator to this
 * queue, in turn.  A runtime exception encountered while
 * trying to add an element (including, in particular, a
 * <tt>null</tt> element) may result in only some of the elements
 * having been successfully added when the associated exception is
 * thrown.
 *
 * @param c collection containing elements to be added to this queue
 * @return <tt>true</tt> if this queue changed as a result of the call
 * @throws ClassCastException if the class of an element of the specified
 *         collection prevents it from being added to this queue
 * @throws NullPointerException if the specified collection contains a
 *         null element and this queue does not permit null elements,
 *         or if the specified collection is null
 * @throws IllegalArgumentException if some property of an element of the
 *         specified collection prevents it from being added to this
 *         queue, or if the specified collection is this queue
 * @throws IllegalStateException if not all the elements can be added at
 *         this time due to insertion restrictions
 * @see #add(Object)
 */
public boolean addAll(Collection<? extends E> c) {
    if (c == null)
        throw new NullPointerException();
    if (c == this)
        throw new IllegalArgumentException();
    boolean modified = false;
    for (E e : c)
        if (add(e))
            modified = true;
    return modified;
}
```

于是接下来跟进 `add()` 函数，其中又调用了 `offer()` 函数。这里指的一提的是，其它相关的入队操作最终都调用了 `offer()`，因为优先队列能够自行扩容，因此没有容量限制，任何入队操作都应当立刻成功返回。

```java
/**
 * Inserts the specified element into this delay queue.
 *
 * @param e the element to add
 * @return {@code true} (as specified by {@link Collection#add})
 * @throws NullPointerException if the specified element is null
 */
public boolean add(E e) {
    return offer(e);
}
```

接下来跟进 `offer()`。首先占据类内的可重入锁，然后将元素通过优先队列的 `offer()` 函数放进去。通过 `peek()` 查看刚放进去的结点是否是下一次马上就要出队的结点，如果是，那么需要重置 `leader`，让出队线程争抢 `leader` 的位置。

```java
/**
 * Inserts the specified element into this delay queue.
 *
 * @param e the element to add
 * @return {@code true}
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        q.offer(e);
        if (q.peek() == e) {
            leader = null;
            available.signal();
        }
        return true;
    } finally {
        lock.unlock();
    }
}
```

其中，`peek()` 函数用于获取 (但不移除) 队头元素。如果队列中没有过期元素，那么将会返回下一个即将过期的元素。

```java
/**
 * Retrieves, but does not remove, the head of this queue, or
 * returns {@code null} if this queue is empty.  Unlike
 * {@code poll}, if no expired elements are available in the queue,
 * this method returns the element that will expire next,
 * if one exists.
 *
 * @return the head of this queue, or {@code null} if this
 *         queue is empty
 */
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return q.peek();
    } finally {
        lock.unlock();
    }
}
```

接下来重点看一看出队操作，因为这里涉及到了比较重要的 **超时** 问题。

首先是非阻塞版本的 `poll()` 函数。该函数试图获取并移除队头元素并返回；如果没有元素到期，那么返回 `null`。在具体实现上，在占据可重入锁后，首先调用 `peek()` 获取到队头元素，然后调用 `getDeday()` 函数 (元素肯定实现了这个函数) 看看它有没有超时。如果已经超时，那么调用优先队列的 `poll()` 将元素取出；否则直接返回 `null`。

```java
/**
 * Retrieves and removes the head of this queue, or returns {@code null}
 * if this queue has no elements with an expired delay.
 *
 * @return the head of this queue, or {@code null} if this
 *         queue has no elements with an expired delay
 */
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        E first = q.peek();
        if (first == null || first.getDelay(NANOSECONDS) > 0)
            return null;
        else
            return q.poll();
    } finally {
        lock.unlock();
    }
}
```

阻塞版本的 `take()`，这里也是 Leader-Follower Pattern 的精髓所在：

* 只有 leader 线程使用 `awaitNanos()` 等待确切的超时时间
* 其它线程使用 `await()` 进入无限期等待，除非被确切地唤醒

同样，占据锁后，在死循环中不断 peek 队头元素。如果队头元素为空，那么直接进入无限期等待。如果队头不为空，那么获取队头的剩余超时时间：

* 如果队头已经到期，那么立刻返回 `poll()`
* 如果队头还未到期，而已经有 leader 线程在等待了，那么当前线程进入无限期等待
* 如果队头还未到期，而没有 leader 线程等待，那么就将自身设置为 `leader`，然后等待 **确切的时间** (队头过期)

接下来，只有 `leader` 线程才可能苏醒。苏醒后，它将自身从 `leader` 中解除引用，然后返回死循环开始处。而所有 follower 线程将不会被唤醒，以省去不必要的开销 (因为 leader 都还没开张呢，follower 就更别急了)。

```java
/**
 * Retrieves and removes the head of this queue, waiting if necessary
 * until an element with an expired delay is available on this queue.
 *
 * @return the head of this queue
 * @throws InterruptedException {@inheritDoc}
 */
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();
            if (first == null)
                available.await();
            else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    return q.poll();
                first = null; // don't retain ref while waiting
                if (leader != null)
                    available.await();
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        available.awaitNanos(delay);
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();
        lock.unlock();
    }
}
```

最终是定时阻塞版本的 `poll()` 函数。首先根据参数给定的超时时间计算出要等待的纳秒数。占据锁后，在一个死循环中不断尝试以下几种情况：

1. 队头为空 (优先队列中没有元素)
    * 如果等待时间已耗尽，直接返回 `null`
    * 如果等待时间未耗尽，那么阻塞等待至等待时间耗尽时
2. 队头不为空，那么取得队头元素的剩余等待时间
    * 如果队头已经过期，那么调用 `poll()` 出队并返回
    * 如果函数等待时间已耗尽，直接返回 `null`

剩余情况应该是队头不为空，但是队头还未超时，函数等待时间也还没耗尽的情况。如果：

* 函数的剩余等待时间 < 队头过期剩余时间
* 已有 `leader` 等待

两者存在一种，那么使当前线程阻塞等待直至函数等待时间耗尽。这种线程虽然最终肯定会拿到 `null`，但是还是要让其阻塞完才能返回 (对应于阻塞版本的无限期阻塞)。

如果队头过期剩余时间 < 函数的剩余等待时间，并且 `leader` 为空，那么将当前线程设置为 `leader` 后，阻塞直至队头过期。`leader` 苏醒后，更新函数自身的剩余等待时间。如果线程自身是 `leader`，则解除自身为 `leader`，并重新开始新一轮的循环 (在新一轮循环中应该可以取得过期的队头元素)。

当函数最终返回时，如果队头不为空且 `leader` 空缺，那么释放一个 `available` 的信号唤醒阻塞中的线程，使其中之一成为新的 `leader`。

> 疑惑：外层已经加了锁，那还可能有多个线程进入这一段死循环代码中吗？

```java
/**
 * Retrieves and removes the head of this queue, waiting if necessary
 * until an element with an expired delay is available on this queue,
 * or the specified wait time expires.
 *
 * @return the head of this queue, or {@code null} if the
 *         specified waiting time elapses before an element with
 *         an expired delay becomes available
 * @throws InterruptedException {@inheritDoc}
 */
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        for (;;) {
            E first = q.peek();
            if (first == null) {
                if (nanos <= 0)
                    return null;
                else
                    nanos = available.awaitNanos(nanos);
            } else {
                long delay = first.getDelay(NANOSECONDS);
                if (delay <= 0)
                    return q.poll();
                if (nanos <= 0)
                    return null;
                first = null; // don't retain ref while waiting
                if (nanos < delay || leader != null)
                    nanos = available.awaitNanos(nanos);
                else {
                    Thread thisThread = Thread.currentThread();
                    leader = thisThread;
                    try {
                        long timeLeft = available.awaitNanos(delay);
                        nanos -= delay - timeLeft;
                    } finally {
                        if (leader == thisThread)
                            leader = null;
                    }
                }
            }
        }
    } finally {
        if (leader == null && q.peek() != null)
            available.signal();
        lock.unlock();
    }
}
```

下面的 `drainTo()` 函数负责返回所有 / 最多 `maxElements` 个 **已过期元素**。判断元素是否过期使用了 `peekExpired()` 函数 - 当且仅当队头元素不为空且已经过期时返回该元素，否则返回 `null`：

```java
/**
 * Returns first element only if it is expired.
 * Used only by drainTo.  Call only when holding lock.
 */
private E peekExpired() {
    // assert lock.isHeldByCurrentThread();
    E first = q.peek();
    return (first == null || first.getDelay(NANOSECONDS) > 0) ?
        null : first;
}
```

而 `drainTo()` 函数不断出队元素并放入返回集合中，直到 `peekExpired()` 返回 `null` 或返回集合规模达到 `maxElements` 时停止：

```java
/**
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 * @throws IllegalArgumentException      {@inheritDoc}
 */
public int drainTo(Collection<? super E> c) {
    if (c == null)
        throw new NullPointerException();
    if (c == this)
        throw new IllegalArgumentException();
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        int n = 0;
        for (E e; (e = peekExpired()) != null;) {
            c.add(e);       // In this order, in case add() throws.
            q.poll();
            ++n;
        }
        return n;
    } finally {
        lock.unlock();
    }
}

/**
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 * @throws IllegalArgumentException      {@inheritDoc}
 */
public int drainTo(Collection<? super E> c, int maxElements) {
    if (c == null)
        throw new NullPointerException();
    if (c == this)
        throw new IllegalArgumentException();
    if (maxElements <= 0)
        return 0;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        int n = 0;
        for (E e; n < maxElements && (e = peekExpired()) != null;) {
            c.add(e);       // In this order, in case add() throws.
            q.poll();
            ++n;
        }
        return n;
    } finally {
        lock.unlock();
    }
}
```

---

## Others

剩余函数的基本思路都是：

1. 先占据可重入锁
2. 调用优先队列的同名相应函数

---

## References

[Stackoverflow - What exactly is the leader used for in DelayQueue?](https://stackoverflow.com/questions/48493830/what-exactly-is-the-leader-used-for-in-delayqueue)

[知乎专栏 - 死磕 Java 集合之 DelayQueue 源码分析](https://zhuanlan.zhihu.com/p/64152280)

---

