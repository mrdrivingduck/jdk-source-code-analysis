# Class - java.util.concurrent.SynchronousQueue

Created by : Mr Dk.

2020 / 11 / 01 13:53

Nanjing, Jiangsu, China

---

## Definition

同步队列是一个阻塞队列，每一次插入操作必须要等待一个相应线程的 remove 操作匹配，反之亦然。同步队列没有任何内部容量，从集合视角来看，队列表现为一个 **空集合**：在 remove 操作之前不能通过 `peek()` 查看内部元素，也无法对同步队列进行迭代 (你没东西迭代)。只有当另一个线程正试图进行 remove 操作时，线程的 insert 操作才会成功。队列的头结点为第一个试图对队列进行 insert 操作的线程。

这个类支持以公平 / 非公平方式进 / 出队的操作。根据构造函数提供的公平性选项，对于公平方式，内部队列将会被实例化为一个队列；对于非公平方式，内部队列将会被实例化为一个栈。

> 关于网上各类源代码解析中 **SynchronousQueue 不存放任何元素** 的说法，其实应该是特指 **不存放数据**。众所周知 SynchronousQueue 是一种阻塞队列，而阻塞队列适用于生产者 / 消费者模式：生产者将数据放入阻塞队列中，消费者将数据从阻塞队列中取走。而 SynchronousQueue 自身不会存放数据，而是存放 **阻塞的线程**：即生产者线程试图对同步队列进行 insert 操作，而此时没有消费者，那么生产者线程阻塞，维护在 SynchronousQueue 内部的队列中；当一个消费者线程试图对同步队列进行 remove 操作时，同步队列队头的消费者线程被唤醒，两者直接传递数据。这个过程中，数据始终没有和同步队列沾边，同步队列仅作为一个接头人、牵线人。因此，与集合视角相关的函数返回值都会表现为一个空集合。
>
> 而功能类似的 `LinkedTransferQueue`，生产者可以把数据放入队列中，内部队列相当于也可以维护数据。

```java
/**
 * A {@linkplain BlockingQueue blocking queue} in which each insert
 * operation must wait for a corresponding remove operation by another
 * thread, and vice versa.  A synchronous queue does not have any
 * internal capacity, not even a capacity of one.  You cannot
 * {@code peek} at a synchronous queue because an element is only
 * present when you try to remove it; you cannot insert an element
 * (using any method) unless another thread is trying to remove it;
 * you cannot iterate as there is nothing to iterate.  The
 * <em>head</em> of the queue is the element that the first queued
 * inserting thread is trying to add to the queue; if there is no such
 * queued thread then no element is available for removal and
 * {@code poll()} will return {@code null}.  For purposes of other
 * {@code Collection} methods (for example {@code contains}), a
 * {@code SynchronousQueue} acts as an empty collection.  This queue
 * does not permit {@code null} elements.
 *
 * <p>Synchronous queues are similar to rendezvous channels used in
 * CSP and Ada. They are well suited for handoff designs, in which an
 * object running in one thread must sync up with an object running
 * in another thread in order to hand it some information, event, or
 * task.
 *
 * <p>This class supports an optional fairness policy for ordering
 * waiting producer and consumer threads.  By default, this ordering
 * is not guaranteed. However, a queue constructed with fairness set
 * to {@code true} grants threads access in FIFO order.
 *
 * <p>This class and its iterator implement all of the
 * <em>optional</em> methods of the {@link Collection} and {@link
 * Iterator} interfaces.
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.5
 * @author Doug Lea and Bill Scherer and Michael Scott
 * @param <E> the type of elements held in this collection
 */
public class SynchronousQueue<E> extends AbstractQueue<E>
    implements BlockingQueue<E>, java.io.Serializable {
    
}
```

---

## Constructor

构造函数。根据公平性参数决定将内部队列构造为队列还是栈：

```java
/**
 * The transferer. Set only in constructor, but cannot be declared
 * as final without further complicating serialization.  Since
 * this is accessed only at most once per public method, there
 * isn't a noticeable performance penalty for using volatile
 * instead of final here.
 */
private transient volatile Transferer<E> transferer;

/**
 * Creates a {@code SynchronousQueue} with nonfair access policy.
 */
public SynchronousQueue() {
    this(false);
}

/**
 * Creates a {@code SynchronousQueue} with the specified fairness policy.
 *
 * @param fair if true, waiting threads contend in FIFO order for
 *        access; otherwise the order is unspecified.
 */
public SynchronousQueue(boolean fair) {
    transferer = fair ? new TransferQueue<E>() : new TransferStack<E>();
}
```

## Input Operations

阻塞版本。`put()` 函数的调用线程试图找到一个配对的消费者线程。如果没有，则进入队列中等待配对。如果等待过程中被中断，则 `transfer()` 返回 `null`，停止等待并抛出异常。

```java
/**
 * Adds the specified element to this queue, waiting if necessary for
 * another thread to receive it.
 *
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    if (transferer.transfer(e, false, 0) == null) {
        Thread.interrupted();
        throw new InterruptedException();
    }
}
```

非阻塞版本。`offer()` 函数试图将调用线程加入队列中，如果没有消费者线程配对，则返回 `false`；否则返回 `true`：

```java
/**
 * Inserts the specified element into this queue, if another thread is
 * waiting to receive it.
 *
 * @param e the element to add
 * @return {@code true} if the element was added to this queue, else
 *         {@code false}
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    return transferer.transfer(e, true, 0) != null;
}
```

定时阻塞版本的 `offer()`，可以被中断打断：

```java
/**
 * Inserts the specified element into this queue, waiting if necessary
 * up to the specified wait time for another thread to receive it.
 *
 * @return {@code true} if successful, or {@code false} if the
 *         specified waiting time elapses before a consumer appears
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {
    if (e == null) throw new NullPointerException();
    if (transferer.transfer(e, true, unit.toNanos(timeout)) != null)
        return true;
    if (!Thread.interrupted())
        return false;
    throw new InterruptedException();
}
```

## Output Operations

阻塞版本。`take()` 函数试图获取一个配对的生产者线程，如果没有，则会一直阻塞；或被线程打断：

```java
/**
 * Retrieves and removes the head of this queue, waiting if necessary
 * for another thread to insert it.
 *
 * @return the head of this queue
 * @throws InterruptedException {@inheritDoc}
 */
public E take() throws InterruptedException {
    E e = transferer.transfer(null, false, 0);
    if (e != null)
        return e;
    Thread.interrupted();
    throw new InterruptedException();
}
```

非阻塞版本。`poll()` 函数试图获取一个配对的生产者线程，如果没有，则返回 `null`：

```java
/**
 * Retrieves and removes the head of this queue, if another thread
 * is currently making an element available.
 *
 * @return the head of this queue, or {@code null} if no
 *         element is available
 */
public E poll() {
    return transferer.transfer(null, true, 0);
}
```

定时阻塞版本：

```java
/**
 * Retrieves and removes the head of this queue, waiting
 * if necessary up to the specified wait time, for another thread
 * to insert it.
 *
 * @return the head of this queue, or {@code null} if the
 *         specified waiting time elapses before an element is present
 * @throws InterruptedException {@inheritDoc}
 */
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    E e = transferer.transfer(null, true, unit.toNanos(timeout));
    if (e != null || !Thread.interrupted())
        return e;
    throw new InterruptedException();
}
```

## Collection

集合视角全部表现出一个空集合的形式，以其中一个函数为例：

```java
/**
 * Always returns zero.
 * A {@code SynchronousQueue} has no internal capacity.
 *
 * @return zero
 */
public int size() {
    return 0;
}
```

---

## Transfer

可以看到，上述所有的入队 / 出队操作都由一个 `transfer()` 函数实现，其定义如下所示：

* `E` 表示队列元素，为空时代表队列需要返回一个生产者线程；不为空时代表要向队列插入消费者线程
* `timed` 表示操作是否需要超时，对于 `take()` / `put()` 来说，阻塞，不需要超时；对于 `offer()` / `poll()` 来说需要超时
* `nanos` 表示超时时间，`0` 显然代表立刻返回

```java
/**
 * Shared internal API for dual stacks and queues.
 */
abstract static class Transferer<E> {
    /**
     * Performs a put or take.
     *
     * @param e if non-null, the item to be handed to a consumer;
     *          if null, requests that transfer return an item
     *          offered by producer.
     * @param timed if this operation should timeout
     * @param nanos the timeout, in nanoseconds
     * @return if non-null, the item provided or received; if null,
     *         the operation failed due to timeout or interrupt --
     *         the caller can distinguish which of these occurred
     *         by checking Thread.interrupted.
     */
    abstract E transfer(E e, boolean timed, long nanos);
}
```

### Queue Implementation

来看一看公平模式下的同步队列内部实现：

* 如果队列为空，或者新结点与队列中结点的模式相同 (同为生产者或消费者)，则试图加入队列并等待
* 如果新结点与队列中结点的模式相反，则产生匹配并直接出队

```java
/**
 * Puts or takes an item.
 */
@SuppressWarnings("unchecked")
E transfer(E e, boolean timed, long nanos) {
    /* Basic algorithm is to loop trying to take either of
     * two actions:
     *
     * 1. If queue apparently empty or holding same-mode nodes,
     *    try to add node to queue of waiters, wait to be
     *    fulfilled (or cancelled) and return matching item.
     *
     * 2. If queue apparently contains waiting items, and this
     *    call is of complementary mode, try to fulfill by CAS'ing
     *    item field of waiting node and dequeuing it, and then
     *    returning matching item.
     *
     * In each case, along the way, check for and try to help
     * advance head and tail on behalf of other stalled/slow
     * threads.
     *
     * The loop starts off with a null check guarding against
     * seeing uninitialized head or tail values. This never
     * happens in current SynchronousQueue, but could if
     * callers held non-volatile/final ref to the
     * transferer. The check is here anyway because it places
     * null checks at top of loop, which is usually faster
     * than having them implicitly interspersed.
     */

    QNode s = null; // constructed/reused as needed
    boolean isData = (e != null);

    for (;;) {
        QNode t = tail;
        QNode h = head;
        if (t == null || h == null)         // saw uninitialized value
            continue;                       // spin

        if (h == t || t.isData == isData) { // empty or same-mode
            QNode tn = t.next;
            if (t != tail)                  // inconsistent read
                continue;
            if (tn != null) {               // lagging tail
                advanceTail(t, tn);
                continue;
            }
            if (timed && nanos <= 0)        // can't wait
                return null;
            if (s == null)
                s = new QNode(e, isData);
            if (!t.casNext(null, s))        // failed to link in
                continue;

            advanceTail(t, s);              // swing tail and wait
            Object x = awaitFulfill(s, e, timed, nanos);
            if (x == s) {                   // wait was cancelled
                clean(t, s);
                return null;
            }

            if (!s.isOffList()) {           // not already unlinked
                advanceHead(t, s);          // unlink if head
                if (x != null)              // and forget fields
                    s.item = s;
                s.waiter = null;
            }
            return (x != null) ? (E)x : e;

        } else {                            // complementary-mode
            QNode m = h.next;               // node to fulfill
            if (t != tail || m == null || h != head)
                continue;                   // inconsistent read

            Object x = m.item;
            if (isData == (x != null) ||    // m already fulfilled
                x == m ||                   // m cancelled
                !m.casItem(x, e)) {         // lost CAS
                advanceHead(h, m);          // dequeue and retry
                continue;
            }

            advanceHead(h, m);              // successfully fulfilled
            LockSupport.unpark(m.waiter);
            return (x != null) ? (E)x : e;
        }
    }
}
```

### Stack Implementation

非公平的同步队列使用栈结构来实现配对，具体过程大同小异：

* 如果栈为空，或者新结点与栈中结点的模式相同 (同为生产者或消费者)，则试图 push 到栈中并等待匹配
* 如果新结点与栈中结点模式相反，则先将结点 push 到栈中，匹配后双双 pop 出栈

```java
/**
 * Puts or takes an item.
 */
@SuppressWarnings("unchecked")
E transfer(E e, boolean timed, long nanos) {
    /*
     * Basic algorithm is to loop trying one of three actions:
     *
     * 1. If apparently empty or already containing nodes of same
     *    mode, try to push node on stack and wait for a match,
     *    returning it, or null if cancelled.
     *
     * 2. If apparently containing node of complementary mode,
     *    try to push a fulfilling node on to stack, match
     *    with corresponding waiting node, pop both from
     *    stack, and return matched item. The matching or
     *    unlinking might not actually be necessary because of
     *    other threads performing action 3:
     *
     * 3. If top of stack already holds another fulfilling node,
     *    help it out by doing its match and/or pop
     *    operations, and then continue. The code for helping
     *    is essentially the same as for fulfilling, except
     *    that it doesn't return the item.
     */

    SNode s = null; // constructed/reused as needed
    int mode = (e == null) ? REQUEST : DATA;

    for (;;) {
        SNode h = head;
        if (h == null || h.mode == mode) {  // empty or same-mode
            if (timed && nanos <= 0) {      // can't wait
                if (h != null && h.isCancelled())
                    casHead(h, h.next);     // pop cancelled node
                else
                    return null;
            } else if (casHead(h, s = snode(s, e, h, mode))) {
                SNode m = awaitFulfill(s, timed, nanos);
                if (m == s) {               // wait was cancelled
                    clean(s);
                    return null;
                }
                if ((h = head) != null && h.next == s)
                    casHead(h, s.next);     // help s's fulfiller
                return (E) ((mode == REQUEST) ? m.item : s.item);
            }
        } else if (!isFulfilling(h.mode)) { // try to fulfill
            if (h.isCancelled())            // already cancelled
                casHead(h, h.next);         // pop and retry
            else if (casHead(h, s=snode(s, e, h, FULFILLING|mode))) {
                for (;;) { // loop until matched or waiters disappear
                    SNode m = s.next;       // m is s's match
                    if (m == null) {        // all waiters are gone
                        casHead(s, null);   // pop fulfill node
                        s = null;           // use new node next time
                        break;              // restart main loop
                    }
                    SNode mn = m.next;
                    if (m.tryMatch(s)) {
                        casHead(s, mn);     // pop both s and m
                        return (E) ((mode == REQUEST) ? m.item : s.item);
                    } else                  // lost match
                        s.casNext(m, mn);   // help unlink
                }
            }
        } else {                            // help a fulfiller
            SNode m = h.next;               // m is h's match
            if (m == null)                  // waiter is gone
                casHead(h, null);           // pop fulfilling node
            else {
                SNode mn = m.next;
                if (m.tryMatch(h))          // help match
                    casHead(h, mn);         // pop both h and m
                else                        // lost match
                    h.casNext(m, mn);       // help unlink
            }
        }
    }
}
```

---

