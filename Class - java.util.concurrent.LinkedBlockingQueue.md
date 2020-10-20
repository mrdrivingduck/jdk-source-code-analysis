# Class - java.util.concurrent.LinkedBlockingQueue

Created by : Mr Dk.

2020 / 10 / 20 21:17

Nanjing, Jiangsu, China

---

## Definition

基于链结点实现的 BlockingQueue。队列中的元素遵循 FIFO，队头的元素在队列中的时间最长，队尾最短。新的元素在队尾被插入。

可以指定一个可选的容量边界参数，防止队列无限制扩大。如果未指定该参数，则默认容量为 `Integer.MAX_VALUE`。链接的结点会在插入过程中动态创建，除非插入操作会引发队列超过最大容量。

```java
/**
 * An optionally-bounded {@linkplain BlockingQueue blocking queue} based on
 * linked nodes.
 * This queue orders elements FIFO (first-in-first-out).
 * The <em>head</em> of the queue is that element that has been on the
 * queue the longest time.
 * The <em>tail</em> of the queue is that element that has been on the
 * queue the shortest time. New elements
 * are inserted at the tail of the queue, and the queue retrieval
 * operations obtain elements at the head of the queue.
 * Linked queues typically have higher throughput than array-based queues but
 * less predictable performance in most concurrent applications.
 *
 * <p>The optional capacity bound constructor argument serves as a
 * way to prevent excessive queue expansion. The capacity, if unspecified,
 * is equal to {@link Integer#MAX_VALUE}.  Linked nodes are
 * dynamically created upon each insertion unless this would bring the
 * queue above capacity.
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
 * @author Doug Lea
 * @param <E> the type of elements held in this collection
 */
public class LinkedBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {
    private static final long serialVersionUID = -6903933977591709194L;

}
```

双锁设计：

* Put Lock - 负责向队列中 put 的操作
* Take Lock - 负责从队列中 take 的操作

两者依赖于原子的 `count`，避免在大多数场合中需要同时获得两把锁。添加和删除操作并不是互斥的，可以并发进行。设计思想详见注释：

```java
/*
 * A variant of the "two lock queue" algorithm.  The putLock gates
 * entry to put (and offer), and has an associated condition for
 * waiting puts.  Similarly for the takeLock.  The "count" field
 * that they both rely on is maintained as an atomic to avoid
 * needing to get both locks in most cases. Also, to minimize need
 * for puts to get takeLock and vice-versa, cascading notifies are
 * used. When a put notices that it has enabled at least one take,
 * it signals taker. That taker in turn signals others if more
 * items have been entered since the signal. And symmetrically for
 * takes signalling puts. Operations such as remove(Object) and
 * iterators acquire both locks.
 *
 * Visibility between writers and readers is provided as follows:
 *
 * Whenever an element is enqueued, the putLock is acquired and
 * count updated.  A subsequent reader guarantees visibility to the
 * enqueued Node by either acquiring the putLock (via fullyLock)
 * or by acquiring the takeLock, and then reading n = count.get();
 * this gives visibility to the first n items.
 *
 * To implement weakly consistent iterators, it appears we need to
 * keep all Nodes GC-reachable from a predecessor dequeued Node.
 * That would cause two problems:
 * - allow a rogue Iterator to cause unbounded memory retention
 * - cause cross-generational linking of old Nodes to new Nodes if
 *   a Node was tenured while live, which generational GCs have a
 *   hard time dealing with, causing repeated major collections.
 * However, only non-deleted Nodes need to be reachable from
 * dequeued Nodes, and reachability does not necessarily have to
 * be of the kind understood by the GC.  We use the trick of
 * linking a Node that has just been dequeued to itself.  Such a
 * self-link implicitly means to advance to head.next.
 */
```

---

## Node Definition

很平常的泛型链表结点，包含一个 `item` 域保存值，一个 `next` 指针用于链接下一个元素：

```java
/**
 * Linked list node class
 */
static class Node<E> {
    E item;

    /**
     * One of:
     * - the real successor Node
     * - this Node, meaning the successor is head.next
     * - null, meaning there is no successor (this is the last node)
     */
    Node<E> next;

    Node(E x) { item = x; }
}
```

---

## Fields

包含头尾结点：

* 头结点的值为空
* 尾结点的 `next` 为空

包含两个可重入锁，及其对应的 `Condition` (非空条件 / 非满条件)。

> Condition 对象需要通过 Lock 对象创建。调用 `await()` 函数后，线程将进入睡眠，等待被信号唤醒。由 `signal()` 或 `signalAll()` 唤醒一个或多个等待的线程。

包含容量值，以及原子计数器 `count`：

```java
/**
 * Head of linked list.
 * Invariant: head.item == null
 */
transient Node<E> head;

/**
 * Tail of linked list.
 * Invariant: last.next == null
 */
private transient Node<E> last;

/** Lock held by take, poll, etc */
private final ReentrantLock takeLock = new ReentrantLock();

/** Wait queue for waiting takes */
private final Condition notEmpty = takeLock.newCondition();

/** Lock held by put, offer, etc */
private final ReentrantLock putLock = new ReentrantLock();

/** Wait queue for waiting puts */
private final Condition notFull = putLock.newCondition();
```

---

## Constructor

初始化头尾结点，并设置内部容量，默认为 `Integer.MAX_VALUE`：

```java
/**
 * Creates a {@code LinkedBlockingQueue} with a capacity of
 * {@link Integer#MAX_VALUE}.
 */
public LinkedBlockingQueue() {
    this(Integer.MAX_VALUE);
}

/**
 * Creates a {@code LinkedBlockingQueue} with the given (fixed) capacity.
 *
 * @param capacity the capacity of this queue
 * @throws IllegalArgumentException if {@code capacity} is not greater
 *         than zero
 */
public LinkedBlockingQueue(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    this.capacity = capacity;
    last = head = new Node<E>(null);
}
```

以默认容量初始化队列内部信息，并用一个已有的集合初始化队列，加入队列的顺序为集合遍历顺序。在初始化过程中会对 `putLock` 上锁，防止其它线程向集合 put。`null` 元素禁止被放入队列中，如果添加过程中队列已满，则抛出异常。

```java
/**
 * Creates a {@code LinkedBlockingQueue} with a capacity of
 * {@link Integer#MAX_VALUE}, initially containing the elements of the
 * given collection,
 * added in traversal order of the collection's iterator.
 *
 * @param c the collection of elements to initially contain
 * @throws NullPointerException if the specified collection or any
 *         of its elements are null
 */
public LinkedBlockingQueue(Collection<? extends E> c) {
    this(Integer.MAX_VALUE);
    final ReentrantLock putLock = this.putLock;
    putLock.lock(); // Never contended, but necessary for visibility
    try {
        int n = 0;
        for (E e : c) {
            if (e == null)
                throw new NullPointerException();
            if (n == capacity)
                throw new IllegalStateException("Queue full");
            enqueue(new Node<E>(e));
            ++n;
        }
        count.set(n);
    } finally {
        putLock.unlock();
    }
}
```

其中用到的入队子函数：

```java
/**
 * Links node at end of queue.
 *
 * @param node the node
 */
private void enqueue(Node<E> node) {
    // assert putLock.isHeldByCurrentThread();
    // assert last.next == null;
    last = last.next = node;
}
```

## Capacity

返回队列中的元素个数。直接返回原子计数器 `count` 的值：

```java
// this doc comment is overridden to remove the reference to collections
// greater in size than Integer.MAX_VALUE
/**
 * Returns the number of elements in this queue.
 *
 * @return the number of elements in this queue
 */
public int size() {
    return count.get();
}
```

返回队列中的剩余容量：

```java
// this doc comment is a modified copy of the inherited doc comment,
// without the reference to unlimited queues.
/**
 * Returns the number of additional elements that this queue can ideally
 * (in the absence of memory or resource constraints) accept without
 * blocking. This is always equal to the initial capacity of this queue
 * less the current {@code size} of this queue.
 *
 * <p>Note that you <em>cannot</em> always tell if an attempt to insert
 * an element will succeed by inspecting {@code remainingCapacity}
 * because it may be the case that another thread is about to
 * insert or remove an element.
 */
public int remainingCapacity() {
    return capacity - count.get();
}
```

## Input Operations

### Put

向队尾插入元素，将一直等待直至队列中有空位。

* 首先上锁 `putLock`
* 如果队列长度已经达到容量上限，则调用 `await()` 进入睡眠，直到有空位
* 进行入队操作，并原子地增加 `count`
* 如果入队后队列中还有空位，那么释放一个 `signal()` 信号唤醒一个睡眠线程
* 解锁

如果插入之前队列是空的，那么唤醒一个正在睡眠等待中的 take 线程。为了保证只唤醒一个线程，需要获得 `takeLock`。

```java
/**
 * Inserts the specified element at the tail of this queue, waiting if
 * necessary for space to become available.
 *
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        /*
         * Note that count is used in wait guard even though it is
         * not protected by lock. This works because count can
         * only decrease at this point (all other puts are shut
         * out by lock), and we (or some other waiting put) are
         * signalled if it ever changes from capacity. Similarly
         * for all other uses of count in other wait guards.
         */
        while (count.get() == capacity) {
            notFull.await();
        }
        enqueue(node);
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
}
```

```java
/**
 * Signals a waiting take. Called only from put/offer (which do not
 * otherwise ordinarily lock takeLock.)
 */
private void signalNotEmpty() {
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
}
```

### Offer

将元素插入队尾，如果容量已满，只等待指定长的时间后返回成功 / 失败。

与 `put()` 的区别是，本函数将会根据 `timeout` 参数计算出应当等待的时间。如果队列中容量不够，同样会陷入等待，但是调用的是定时等待。只有当以下情况线程才会苏醒：

* 有线程对这个条件释放了 `signal()`，而当前线程恰好被选中接盘
* 有线程对这个条件释放了 `signalAll()`
* 超时

因此苏醒后，当前线程会判断等待的状态，如果超时，那么直接返回 `false`。在 `finally` 块中的会将之前获取的 putLock 释放。其余部分与上述函数实现一致：

* 如果入队成功后发现还有多余容量，对非空条件发送一个 signal
* 如果队列原先为空，则通知睡眠等待中的 take 线程

```java
/**
 * Inserts the specified element at the tail of this queue, waiting if
 * necessary up to the specified wait time for space to become available.
 *
 * @return {@code true} if successful, or {@code false} if
 *         the specified waiting time elapses before space is available
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {

    if (e == null) throw new NullPointerException();
    long nanos = unit.toNanos(timeout);
    int c = -1;
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly();
    try {
        while (count.get() == capacity) {
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);
        }
        enqueue(new Node<E>(e));
        c = count.getAndIncrement();
        if (c + 1 < capacity)
            notFull.signal();
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
    return true;
}
```

将元素插入队列的非定时阻塞等待版本。如果插入成功立刻返回 `true`，如果队列满插入失败则立刻返回 `false`。因此不需要使用到 Condition：

```java
/**
 * Inserts the specified element at the tail of this queue if it is
 * possible to do so immediately without exceeding the queue's capacity,
 * returning {@code true} upon success and {@code false} if this queue
 * is full.
 * When using a capacity-restricted queue, this method is generally
 * preferable to method {@link BlockingQueue#add add}, which can fail to
 * insert an element only by throwing an exception.
 *
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
    if (e == null) throw new NullPointerException();
    final AtomicInteger count = this.count;
    if (count.get() == capacity)
        return false;
    int c = -1;
    Node<E> node = new Node<E>(e);
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        if (count.get() < capacity) {
            enqueue(node);
            c = count.getAndIncrement();
            if (c + 1 < capacity)
                notFull.signal();
        }
    } finally {
        putLock.unlock();
    }
    if (c == 0)
        signalNotEmpty();
    return c >= 0;
}
```

## Output Operations

### Take

从队头获取元素。如果队列为空则进入无限时阻塞。

* 首先获取 `takeLock` 保证只有一个 take 线程对队列进行修改
* 如果队列为空，那么进入睡眠中等待被唤醒
* 出队操作
* 原子地减少 `count` 计数器 (与 put 线程可以进行并发修改)
* 解锁

如果操作之前队列是满的，那么就通知可能正在等待中的 put 线程：

* 对非满条件发一个 signal，唤醒一个 put 线程
* 为了保证只有一个 put 线程被唤醒，需要获取 `putLock`

```java
public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            notEmpty.await();
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

```java
/**
 * Removes a node from head of queue.
 *
 * @return the node
 */
private E dequeue() {
    // assert takeLock.isHeldByCurrentThread();
    // assert head.item == null;
    Node<E> h = head;
    Node<E> first = h.next;
    h.next = h; // help GC
    head = first;
    E x = first.item;
    first.item = null;
    return x;
}
```

```java
/**
 * Signals a waiting put. Called only from take/poll.
 */
private void signalNotFull() {
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        notFull.signal();
    } finally {
        putLock.unlock();
    }
}
```

### Poll

从队头获取元素，只阻塞等待指定的时间。每次被唤醒时，需要判断是否超时，如果超时，则直接返回 `null`；如果成功获取到元素，则与 `take()` 的行为一致。

```java
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    E x = null;
    int c = -1;
    long nanos = unit.toNanos(timeout);
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly();
    try {
        while (count.get() == 0) {
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos);
        }
        x = dequeue();
        c = count.getAndDecrement();
        if (c > 1)
            notEmpty.signal();
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

从队头获取元素并立刻返回，不阻塞等待。

```java
public E poll() {
    final AtomicInteger count = this.count;
    if (count.get() == 0)
        return null;
    E x = null;
    int c = -1;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        if (count.get() > 0) {
            x = dequeue();
            c = count.getAndDecrement();
            if (c > 1)
                notEmpty.signal();
        }
    } finally {
        takeLock.unlock();
    }
    if (c == capacity)
        signalNotFull();
    return x;
}
```

### Peek

非阻塞地迅速 *瞥一眼* 队头。如果队头有元素，则立刻返回 (元素依然在队列中)；如果队头没有元素，则返回 `null`。

```java
public E peek() {
    if (count.get() == 0)
        return null;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        Node<E> first = head.next;
        if (first == null)
            return null;
        else
            return first.item;
    } finally {
        takeLock.unlock();
    }
}
```

## Remove

从队列中删除 **一个** 指定 (`equals()` 返回 `true`) 的元素。此时由于需要对队列进行遍历，因此需要同时获取 `takeLock` 和 `putLock` 两把锁。如果对队列进行了修改，返回 `true`；否则返回 `false`。

```java
/**
 * Removes a single instance of the specified element from this queue,
 * if it is present.  More formally, removes an element {@code e} such
 * that {@code o.equals(e)}, if this queue contains one or more such
 * elements.
 * Returns {@code true} if this queue contained the specified element
 * (or equivalently, if this queue changed as a result of the call).
 *
 * @param o element to be removed from this queue, if present
 * @return {@code true} if this queue changed as a result of the call
 */
public boolean remove(Object o) {
    if (o == null) return false;
    fullyLock();
    try {
        for (Node<E> trail = head, p = trail.next;
                p != null;
                trail = p, p = p.next) {
            if (o.equals(p.item)) {
                unlink(p, trail);
                return true;
            }
        }
        return false;
    } finally {
        fullyUnlock();
    }
}
```

同时获取 / 释放两把锁：

```java
/**
 * Locks to prevent both puts and takes.
 */
void fullyLock() {
    putLock.lock();
    takeLock.lock();
}

/**
 * Unlocks to allow both puts and takes.
 */
void fullyUnlock() {
    takeLock.unlock();
    putLock.unlock();
}
```

从队列中删除元素：

* 修改指针，将前后结点相连
* 原子地修改 `count` 计数器
* 如果原先队列是满的，那么可以 signal 一个 put 线程苏醒

```java
/**
 * Unlinks interior Node p with predecessor trail.
 */
void unlink(Node<E> p, Node<E> trail) {
    // assert isFullyLocked();
    // p.next is not changed, to allow iterators that are
    // traversing p to maintain their weak-consistency guarantee.
    p.item = null;
    trail.next = p.next;
    if (last == p)
        last = trail;
    if (count.getAndDecrement() == capacity)
        notFull.signal();
}
```

## Contain

队列中是否包含指定元素。由于需要遍历队列，同样需要同时上锁：

```java
/**
 * Returns {@code true} if this queue contains the specified element.
 * More formally, returns {@code true} if and only if this queue contains
 * at least one element {@code e} such that {@code o.equals(e)}.
 *
 * @param o object to be checked for containment in this queue
 * @return {@code true} if this queue contains the specified element
 */
public boolean contains(Object o) {
    if (o == null) return false;
    fullyLock();
    try {
        for (Node<E> p = head.next; p != null; p = p.next)
            if (o.equals(p.item))
                return true;
        return false;
    } finally {
        fullyUnlock();
    }
}
```

## To Array

分配一个新的 Object 数组，并将当前队列中的所有元素的引用复制到数组中。新数组和队列应当持有的是相同的对象引用 (对象本身没有被复制)，但是在新数组中操作对象引用对队列没有任何影响。需要遍历队列，因此同样需要上两把锁：

```java
/**
 * Returns an array containing all of the elements in this queue, in
 * proper sequence.
 *
 * <p>The returned array will be "safe" in that no references to it are
 * maintained by this queue.  (In other words, this method must allocate
 * a new array).  The caller is thus free to modify the returned array.
 *
 * <p>This method acts as bridge between array-based and collection-based
 * APIs.
 *
 * @return an array containing all of the elements in this queue
 */
public Object[] toArray() {
    fullyLock();
    try {
        int size = count.get();
        Object[] a = new Object[size];
        int k = 0;
        for (Node<E> p = head.next; p != null; p = p.next)
            a[k++] = p.item;
        return a;
    } finally {
        fullyUnlock();
    }
}
```

## Clear

删除队列中的所有元素。由于需要保证操作后的队列为空，因此需要上两把锁。操作成功后，如果队列原先是满的，还要对可能正在等待的 put 线程发送 signal。

```java
/**
 * Atomically removes all of the elements from this queue.
 * The queue will be empty after this call returns.
 */
public void clear() {
    fullyLock();
    try {
        for (Node<E> p, h = head; (p = h.next) != null; h = p) {
            h.next = h;
            p.item = null;
        }
        head = last;
        // assert head.item == null && head.next == null;
        if (count.getAndSet(0) == capacity)
            notFull.signal();
    } finally {
        fullyUnlock();
    }
}
```

## Drain To

将队列中所有 (或最多前 `maxElements`) 个元素取出并放入一个集合中。

* 集合不能为 `null`
* 集合不能是自身
* `maxElements` 必须大于等于 0 (否则无实际效果)

由于函数只是从队列中取元素，因此只需要占用 `takeLock` 即可。函数不会阻塞，是一次性取出队列中的所有元素，不管队列中目前有没有元素。

* 如果队列中有元素，那么在操作完毕后更新 `count` 计数器
* 如果队列原先是满的，那么对可能存在的 put 线程发送 signal

```java
/**
 * @throws UnsupportedOperationException {@inheritDoc}
 * @throws ClassCastException            {@inheritDoc}
 * @throws NullPointerException          {@inheritDoc}
 * @throws IllegalArgumentException      {@inheritDoc}
 */
public int drainTo(Collection<? super E> c) {
    return drainTo(c, Integer.MAX_VALUE);
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
    boolean signalNotFull = false;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lock();
    try {
        int n = Math.min(maxElements, count.get());
        // count.get provides visibility to first n Nodes
        Node<E> h = head;
        int i = 0;
        try {
            while (i < n) {
                Node<E> p = h.next;
                c.add(p.item);
                p.item = null;
                h.next = h;
                h = p;
                ++i;
            }
            return n;
        } finally {
            // Restore invariants even if c.add() threw
            if (i > 0) {
                // assert h.item == null;
                head = h;
                signalNotFull = (count.getAndAdd(-i) == capacity);
            }
        }
    } finally {
        takeLock.unlock();
        if (signalNotFull)
            signalNotFull();
    }
}
```

## Iterator

迭代器在迭代的过程中都会同时占有两把锁：

```java
/**
 * Returns an iterator over the elements in this queue in proper sequence.
 * The elements will be returned in order from first (head) to last (tail).
 *
 * <p>The returned iterator is
 * <a href="package-summary.html#Weakly"><i>weakly consistent</i></a>.
 *
 * @return an iterator over the elements in this queue in proper sequence
 */
public Iterator<E> iterator() {
    return new Itr();
}

private class Itr implements Iterator<E> {
    /*
     * Basic weakly-consistent iterator.  At all times hold the next
     * item to hand out so that if hasNext() reports true, we will
     * still have it to return even if lost race with a take etc.
     */

    private Node<E> current;
    private Node<E> lastRet;
    private E currentElement;

    Itr() {
        fullyLock();
        try {
            current = head.next;
            if (current != null)
                currentElement = current.item;
        } finally {
            fullyUnlock();
        }
    }

    public boolean hasNext() {
        return current != null;
    }

    /**
     * Returns the next live successor of p, or null if no such.
     *
     * Unlike other traversal methods, iterators need to handle both:
     * - dequeued nodes (p.next == p)
     * - (possibly multiple) interior removed nodes (p.item == null)
     */
    private Node<E> nextNode(Node<E> p) {
        for (;;) {
            Node<E> s = p.next;
            if (s == p)
                return head.next;
            if (s == null || s.item != null)
                return s;
            p = s;
        }
    }

    public E next() {
        fullyLock();
        try {
            if (current == null)
                throw new NoSuchElementException();
            E x = currentElement;
            lastRet = current;
            current = nextNode(current);
            currentElement = (current == null) ? null : current.item;
            return x;
        } finally {
            fullyUnlock();
        }
    }

    public void remove() {
        if (lastRet == null)
            throw new IllegalStateException();
        fullyLock();
        try {
            Node<E> node = lastRet;
            lastRet = null;
            for (Node<E> trail = head, p = trail.next;
                    p != null;
                    trail = p, p = p.next) {
                if (p == node) {
                    unlink(p, trail);
                    break;
                }
            }
        } finally {
            fullyUnlock();
        }
    }
}
```

---

