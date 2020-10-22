# Class - java.util.concurrent.LinkedTransferQueue

Created by : Mr Dk.

2020 / 10 / 22 16:08

Nanjing, Jiangsu, China

---

## Definition

注释多、代码多、复杂，没能详细读懂每一行代码，只能宏观分析内部的机制。

```java
/**
 * An unbounded {@link TransferQueue} based on linked nodes.
 * This queue orders elements FIFO (first-in-first-out) with respect
 * to any given producer.  The <em>head</em> of the queue is that
 * element that has been on the queue the longest time for some
 * producer.  The <em>tail</em> of the queue is that element that has
 * been on the queue the shortest time for some producer.
 *
 * <p>Beware that, unlike in most collections, the {@code size} method
 * is <em>NOT</em> a constant-time operation. Because of the
 * asynchronous nature of these queues, determining the current number
 * of elements requires a traversal of the elements, and so may report
 * inaccurate results if this collection is modified during traversal.
 * Additionally, the bulk operations {@code addAll},
 * {@code removeAll}, {@code retainAll}, {@code containsAll},
 * {@code equals}, and {@code toArray} are <em>not</em> guaranteed
 * to be performed atomically. For example, an iterator operating
 * concurrently with an {@code addAll} operation might view only some
 * of the added elements.
 *
 * <p>This class and its iterator implement all of the
 * <em>optional</em> methods of the {@link Collection} and {@link
 * Iterator} interfaces.
 *
 * <p>Memory consistency effects: As with other concurrent
 * collections, actions in a thread prior to placing an object into a
 * {@code LinkedTransferQueue}
 * <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
 * actions subsequent to the access or removal of that element from
 * the {@code LinkedTransferQueue} in another thread.
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.7
 * @author Doug Lea
 * @param <E> the type of elements held in this collection
 */
public class LinkedTransferQueue<E> extends AbstractQueue<E>
    implements TransferQueue<E>, java.io.Serializable {
    private static final long serialVersionUID = -3223113410248163686L;

}
```

这个类实现了 TransferQueue 接口，而 TransferQueue 接口又继承自 BlockingQueue。所以这个类将会实现：

* BlockingQueue 接口下的不同版本的 take / put 操作
* TransferQueue 接下口的不同版本的 transfer 操作

BlockingQueue 中 take / put 操作会在队列非空 / 非满时立刻成功，而在队列空 / 满时阻塞。对于底层为链表的实现方式，如果用户没有指定容量，put 操作将会一直成功，直到队列长度达到 `Integer.MAX_VALUE`。

而 TransferQueue 接口下的 transfer 操作对 put 操作作了限制，只有存在相应的 take 线程与其匹配上，操作才能算成功，而不进行入队操作。否则 transfer 失败，并提供了几种不同的失败方式：

* 非阻塞版本，如果失败则直接返回 `false`
* 定时版本，超时后失败并返回 `false`
* 阻塞版本，阻塞直到有 take 线程匹配

类内部维护了一个链表形式实现的队列，包含 head 和 tail。整个类内没有使用 `synchronized` / JDK 锁对象，enqueue / dequeue 操作全部由 CAS 操作实现。这个队列很有意思：一个队列，两种用途。当 take 线程较活跃时，这个队列成为 take 线程的等待队列；当 put 线程较活跃时，这个队列成为 put 线程的等待队列。队列在某个时刻只可能有一种用途 ()。当然，两种用途用于同一种定义的队列结点：

* `isData` 表明结点用途
    * `false` 说明结点不是数据结点，也就是用于 take 线程的等待
    * `true` 说明结点是数据结点，也就是用于 put 线程保存要放入的数据
* `item` 表明结点数据，只有当 `isData` 为 `true` 时才启用 (不为空)
* `next` 为链表指针
* `waiter` 指示了该结点上的等待线程

```java
/**
 * Queue nodes. Uses Object, not E, for items to allow forgetting
 * them after use.  Relies heavily on Unsafe mechanics to minimize
 * unnecessary ordering constraints: Writes that are intrinsically
 * ordered wrt other accesses or CASes use simple relaxed forms.
 */
static final class Node {
    final boolean isData;   // false if this is a request node
    volatile Object item;   // initially non-null if isData; CASed to match
    volatile Node next;
    volatile Thread waiter; // null until waiting

    // ...
}
```

在向队列中存取数据时，首先使用队列头结点来判断队列的用途：

* 如果两者用途不同，则意味着成功进行了一次匹配 (take & put)
* 如果两者用途相同，则进入队列进行等待 (take & take / put & put)

类中为所有的存取操作定义了四种访问模式，并通过一个公用的函数实现了所有的队列存取操作：

```java
/*
 * Possible values for "how" argument in xfer method.
 */
private static final int NOW   = 0; // for untimed poll, tryTransfer
private static final int ASYNC = 1; // for offer, put, add
private static final int SYNC  = 2; // for transfer, take
private static final int TIMED = 3; // for timed poll, tryTransfer

/**
 * Implements all queuing methods. See above for explanation.
 *
 * @param e the item or null for take
 * @param haveData true if this is a put, else a take
 * @param how NOW, ASYNC, SYNC, or TIMED
 * @param nanos timeout in nanosecs, used only if mode is TIMED
 * @return an item if matched, else e
 * @throws NullPointerException if haveData mode but e is null
 */
private E xfer(E e, boolean haveData, int how, long nanos) {
    // ...
}
```

函数参数中：

* `e` 表示元素，take 线程该参数为 `null`
* `havaData` 表示操作是否添加数据，put 线程才会为 `true`
* `how` 指定了存取方式
* `nanos` 指定了操作超时的时间

以下所有的队列操作都基于这些的参数的排列组合实现。

## Producer

### Put

向队列尾部插入元素。由于队列没有容量限制，这个函数不会阻塞。

```java
/**
 * Inserts the specified element at the tail of this queue.
 * As the queue is unbounded, this method will never block.
 *
 * @throws NullPointerException if the specified element is null
 */
public void put(E e) {
    xfer(e, true, ASYNC, 0);
}
```

### Offer

向队列尾部插入元素。由于队列没有容量限制，函数不会阻塞，并且保证成功，所以超时参数没有实际作用。

```java
/**
 * Inserts the specified element at the tail of this queue.
 * As the queue is unbounded, this method will never block or
 * return {@code false}.
 *
 * @return {@code true} (as specified by
 *  {@link java.util.concurrent.BlockingQueue#offer(Object,long,TimeUnit)
 *  BlockingQueue.offer})
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e, long timeout, TimeUnit unit) {
    xfer(e, true, ASYNC, 0);
    return true;
}

/**
 * Inserts the specified element at the tail of this queue.
 * As the queue is unbounded, this method will never return {@code false}.
 *
 * @return {@code true} (as specified by {@link Queue#offer})
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
    xfer(e, true, ASYNC, 0);
    return true;
}
```

### Add

同上。

```java
/**
 * Inserts the specified element at the tail of this queue.
 * As the queue is unbounded, this method will never throw
 * {@link IllegalStateException} or return {@code false}.
 *
 * @return {@code true} (as specified by {@link Collection#add})
 * @throws NullPointerException if the specified element is null
 */
public boolean add(E e) {
    xfer(e, true, ASYNC, 0);
    return true;
}
```

### Transfer

该接口下实现的函数将会对生产者的活跃度进行一定的限制 (操作有一定可能失败)。

非阻塞版本，只有当队列被用作 take 线程的等待队列时才可能成功 (即存在等待中的消费者)。当与队列中的 take 线程匹配上就立刻返回；否则返回 false 而不入队：

```java
/**
 * Transfers the element to a waiting consumer immediately, if possible.
 *
 * <p>More precisely, transfers the specified element immediately
 * if there exists a consumer already waiting to receive it (in
 * {@link #take} or timed {@link #poll(long,TimeUnit) poll}),
 * otherwise returning {@code false} without enqueuing the element.
 *
 * @throws NullPointerException if the specified element is null
 */
public boolean tryTransfer(E e) {
    return xfer(e, true, NOW, 0) == null;
}
```

阻塞版本：

* 当队列被用作 take 线程的等待队列时，立刻匹配并返回
* 当队列被用作 put 线程的等待队列时，加入队列尾部并阻塞等待，直到被匹配上

```java
/**
 * Transfers the element to a consumer, waiting if necessary to do so.
 *
 * <p>More precisely, transfers the specified element immediately
 * if there exists a consumer already waiting to receive it (in
 * {@link #take} or timed {@link #poll(long,TimeUnit) poll}),
 * else inserts the specified element at the tail of this queue
 * and waits until the element is received by a consumer.
 *
 * @throws NullPointerException if the specified element is null
 */
public void transfer(E e) throws InterruptedException {
    if (xfer(e, true, SYNC, 0) != null) {
        Thread.interrupted(); // failure possible only due to interrupt
        throw new InterruptedException();
    }
}
```

定时版本：

> 如果成功与消费者匹配，则立刻成功返回；否则加入队列等待。如果超出等待时间还没有被成功匹配，则返回 `false`。这里有个问题：结点在队列中等待，如果超时，那么需要把这个结点从队列中移出吗？

```java
/**
 * Transfers the element to a consumer if it is possible to do so
 * before the timeout elapses.
 *
 * <p>More precisely, transfers the specified element immediately
 * if there exists a consumer already waiting to receive it (in
 * {@link #take} or timed {@link #poll(long,TimeUnit) poll}),
 * else inserts the specified element at the tail of this queue
 * and waits until the element is received by a consumer,
 * returning {@code false} if the specified wait time elapses
 * before the element can be transferred.
 *
 * @throws NullPointerException if the specified element is null
 */
public boolean tryTransfer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {
    if (xfer(e, true, TIMED, unit.toNanos(timeout)) == null)
        return true;
    if (!Thread.interrupted())
        return false;
    throw new InterruptedException();
}
```

## Consumer

没啥特别的。

```java
public E take() throws InterruptedException {
    E e = xfer(null, false, SYNC, 0);
    if (e != null)
        return e;
    Thread.interrupted();
    throw new InterruptedException();
}

public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    E e = xfer(null, false, TIMED, unit.toNanos(timeout));
    if (e != null || !Thread.interrupted())
        return e;
    throw new InterruptedException();
}

public E poll() {
    return xfer(null, false, NOW, 0);
}

/**
 * @throws NullPointerException     {@inheritDoc}
 * @throws IllegalArgumentException {@inheritDoc}
 */
public int drainTo(Collection<? super E> c) {
    if (c == null)
        throw new NullPointerException();
    if (c == this)
        throw new IllegalArgumentException();
    int n = 0;
    for (E e; (e = poll()) != null;) {
        c.add(e);
        ++n;
    }
    return n;
}

/**
 * @throws NullPointerException     {@inheritDoc}
 * @throws IllegalArgumentException {@inheritDoc}
 */
public int drainTo(Collection<? super E> c, int maxElements) {
    if (c == null)
        throw new NullPointerException();
    if (c == this)
        throw new IllegalArgumentException();
    int n = 0;
    for (E e; n < maxElements && (e = poll()) != null;) {
        c.add(e);
        ++n;
    }
    return n;
}
```

## More

关于锁的操作没有看懂：自旋 -> LockSupport。

---

## References

[知乎专栏 - 死磕 Java 集合之 LinkedTransferQueue 源码分析](https://zhuanlan.zhihu.com/p/64002492)

[知乎专栏 - 阻塞队列的综合体 LinkedTransferQueue](https://zhuanlan.zhihu.com/p/258118884)

---

