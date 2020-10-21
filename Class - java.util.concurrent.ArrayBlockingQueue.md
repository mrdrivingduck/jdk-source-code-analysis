# Class - java.util.concurrent.ArrayBlockingQueue

Created by : Mr Dk.

2020 / 10 / 21 15:21

Nanjing, Jiangsu, China

---

## Definition

底层基于数组实现的 BlockQueue (与链表实现的阻塞队列形成对照)。FIFO 的逻辑类似，但数组是有边界的，所以基于数组实现的阻塞队列是一个经典的 **有容量** 队列。当队列容量已满时，插入线程将会被阻塞；当队列容量为空时，读取线程也会被阻塞。另外，这个类还支持可选的 **公平性** 参数，以牺牲吞吐量为代价，保证加入队列的 FIFO 顺序一定被满足。

```java
/**
 * A bounded {@linkplain BlockingQueue blocking queue} backed by an
 * array.  This queue orders elements FIFO (first-in-first-out).  The
 * <em>head</em> of the queue is that element that has been on the
 * queue the longest time.  The <em>tail</em> of the queue is that
 * element that has been on the queue the shortest time. New elements
 * are inserted at the tail of the queue, and the queue retrieval
 * operations obtain elements at the head of the queue.
 *
 * <p>This is a classic &quot;bounded buffer&quot;, in which a
 * fixed-sized array holds elements inserted by producers and
 * extracted by consumers.  Once created, the capacity cannot be
 * changed.  Attempts to {@code put} an element into a full queue
 * will result in the operation blocking; attempts to {@code take} an
 * element from an empty queue will similarly block.
 *
 * <p>This class supports an optional fairness policy for ordering
 * waiting producer and consumer threads.  By default, this ordering
 * is not guaranteed. However, a queue constructed with fairness set
 * to {@code true} grants threads access in FIFO order. Fairness
 * generally decreases throughput but reduces variability and avoids
 * starvation.
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
public class ArrayBlockingQueue<E> extends AbstractQueue<E>
        implements BlockingQueue<E>, java.io.Serializable {

    /**
     * Serialization ID. This class relies on default serialization
     * even for the items array, which is default-serialized, even if
     * it is empty. Otherwise it could not be declared final, which is
     * necessary here.
     */
    private static final long serialVersionUID = -817911632652898426L;
}
```

---

## Fields

该类显然会使用一个对象数组作为队列内存。数组被修饰为 `final`，表示一经初始化后 **容量不可改变**。为了避免元素的复制，基于数组的队列实现一般都会循环利用数组空间，因此队列的头尾直接体现为两个数组下标。`count` 用于维护队列中的元素个数。

在并发控制方面，这个类使用了一把可重入锁来锁定所有的存取操作。

```java
/** The queued items */
final Object[] items;

/** items index for next take, poll, peek or remove */
int takeIndex;

/** items index for next put, offer, or add */
int putIndex;

/** Number of elements in the queue */
int count;

/*
    * Concurrency control uses the classic two-condition algorithm
    * found in any textbook.
    */

/** Main lock guarding all access */
final ReentrantLock lock;

/** Condition for waiting takes */
private final Condition notEmpty;

/** Condition for waiting puts */
private final Condition notFull;
```

---

## Constructor

对于构造函数来说，必备的参数是队列容量，用于初始化队列内部的数组。可选的参数是 *是否启用公平竞争机制*，默认是非公平锁，以及在初始化数组后预先放入一个集合内的所有元素：

* 根据 `capacity` 参数初始化内部数组
* 初始化可重入锁 (公平 / 非公平)，以及非空 / 非满的阻塞条件
* 如果需要加入初始元素
    * 首先对队列加锁
    * 依次放入集合元素
    * 初始元素溢出将会产生异常
    * 妥善设置队尾位置
    * 解锁

```java
/**
 * Creates an {@code ArrayBlockingQueue} with the given (fixed)
 * capacity and default access policy.
 *
 * @param capacity the capacity of this queue
 * @throws IllegalArgumentException if {@code capacity < 1}
 */
public ArrayBlockingQueue(int capacity) {
    this(capacity, false);
}

/**
 * Creates an {@code ArrayBlockingQueue} with the given (fixed)
 * capacity and the specified access policy.
 *
 * @param capacity the capacity of this queue
 * @param fair if {@code true} then queue accesses for threads blocked
 *        on insertion or removal, are processed in FIFO order;
 *        if {@code false} the access order is unspecified.
 * @throws IllegalArgumentException if {@code capacity < 1}
 */
public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}

/**
 * Creates an {@code ArrayBlockingQueue} with the given (fixed)
 * capacity, the specified access policy and initially containing the
 * elements of the given collection,
 * added in traversal order of the collection's iterator.
 *
 * @param capacity the capacity of this queue
 * @param fair if {@code true} then queue accesses for threads blocked
 *        on insertion or removal, are processed in FIFO order;
 *        if {@code false} the access order is unspecified.
 * @param c the collection of elements to initially contain
 * @throws IllegalArgumentException if {@code capacity} is less than
 *         {@code c.size()}, or less than 1.
 * @throws NullPointerException if the specified collection or any
 *         of its elements are null
 */
public ArrayBlockingQueue(int capacity, boolean fair,
                            Collection<? extends E> c) {
    this(capacity, fair);

    final ReentrantLock lock = this.lock;
    lock.lock(); // Lock only for visibility, not mutual exclusion
    try {
        int i = 0;
        try {
            for (E e : c) {
                checkNotNull(e);
                items[i++] = e;
            }
        } catch (ArrayIndexOutOfBoundsException ex) {
            throw new IllegalArgumentException();
        }
        count = i;
        putIndex = (i == capacity) ? 0 : i;
    } finally {
        lock.unlock();
    }
}
```

## Input Operations

### Offer

非阻塞方法，向队尾插入元素。如果插入成功则返回 `true`，如果插入失败则返回 `false`。在操作前先要获取锁。

```java
/**
 * Inserts the specified element at the tail of this queue if it is
 * possible to do so immediately without exceeding the queue's capacity,
 * returning {@code true} upon success and {@code false} if this queue
 * is full.  This method is generally preferable to method {@link #add},
 * which can fail to insert an element only by throwing an exception.
 *
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count == items.length)
            return false;
        else {
            enqueue(e);
            return true;
        }
    } finally {
        lock.unlock();
    }
}
```

该函数还有定时阻塞版本：当队列已满时，等待指定长时间后如果队列还没有空，则返回 `false`：

```java
/**
 * Inserts the specified element at the tail of this queue, waiting
 * up to the specified wait time for space to become available if
 * the queue is full.
 *
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public boolean offer(E e, long timeout, TimeUnit unit)
    throws InterruptedException {

    checkNotNull(e);
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length) {
            if (nanos <= 0)
                return false;
            nanos = notFull.awaitNanos(nanos);
        }
        enqueue(e);
        return true;
    } finally {
        lock.unlock();
    }
}
```

以下是具体的入队函数。在队尾放入元素后，将队尾指针自增 - 如果队尾已经到达数组尾部，则回到数组开头。增加计数器。释放非空的信号给可能存在的 take 线程。

```java
/**
 * Inserts element at current put position, advances, and signals.
 * Call only when holding lock.
 */
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    notEmpty.signal();
}
```

### Add

默认使用了父类的 `add()`。在父类中可以看到，实际上使用 `offer()` 作为实现，进行非阻塞插入操作：

* 如果插入成功，则返回 `true`
* 如果因队列已满而插入失败，则抛出异常

```java
/**
 * Inserts the specified element at the tail of this queue if it is
 * possible to do so immediately without exceeding the queue's capacity,
 * returning {@code true} upon success and throwing an
 * {@code IllegalStateException} if this queue is full.
 *
 * @param e the element to add
 * @return {@code true} (as specified by {@link Collection#add})
 * @throws IllegalStateException if this queue is full
 * @throws NullPointerException if the specified element is null
 */
public boolean add(E e) {
    return super.add(e);
}
```

父类中的实现：

```java
/**
 * Inserts the specified element into this queue if it is possible to do so
 * immediately without violating capacity restrictions, returning
 * <tt>true</tt> upon success and throwing an <tt>IllegalStateException</tt>
 * if no space is currently available.
 *
 * <p>This implementation returns <tt>true</tt> if <tt>offer</tt> succeeds,
 * else throws an <tt>IllegalStateException</tt>.
 *
 * @param e the element to add
 * @return <tt>true</tt> (as specified by {@link Collection#add})
 * @throws IllegalStateException if the element cannot be added at this
 *         time due to capacity restrictions
 * @throws ClassCastException if the class of the specified element
 *         prevents it from being added to this queue
 * @throws NullPointerException if the specified element is null and
 *         this queue does not permit null elements
 * @throws IllegalArgumentException if some property of this element
 *         prevents it from being added to this queue
 */
public boolean add(E e) {
    if (offer(e))
        return true;
    else
        throw new IllegalStateException("Queue full");
}
```

### Put

向队列中插入元素，如果队列已满，则阻塞等待直到队列中有空位。

```java
/**
 * Inserts the specified element at the tail of this queue, waiting
 * for space to become available if the queue is full.
 *
 * @throws InterruptedException {@inheritDoc}
 * @throws NullPointerException {@inheritDoc}
 */
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
```

## Output Operations

### Poll

非阻塞函数，从队列中取出元素。如果队列为空，则立刻返回 `null`：

```java
public E poll() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return (count == 0) ? null : dequeue();
    } finally {
        lock.unlock();
    }
}
```

定时阻塞函数，从队列中取出元素。在等待指定长度的时间后，如果队列依旧为空，则返回 `null`：

```java
public E poll(long timeout, TimeUnit unit) throws InterruptedException {
    long nanos = unit.toNanos(timeout);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0) {
            if (nanos <= 0)
                return null;
            nanos = notEmpty.awaitNanos(nanos);
        }
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

其中用到的出队函数：

* 从 `takeIndex` (队头) 处取出元素，将队头对应的引用设置为 `null`
* 队头向后移；如果队头超过数组边界，则绕回数组的最开头
* `count` 计数器自减
* 向可能等待的 put 线程释放信号

```java
/**
 * Extracts element at current take position, advances, and signals.
 * Call only when holding lock.
 */
private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal();
    return x;
}
```

### Take

阻塞函数，从队列中取出元素。如果队列为空，则一直阻塞直至能够从队列中取出元素为止。

```java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}
```

### Peek

瞥一眼当前队头元素 (不进行出队操作)：

```java
public E peek() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return itemAt(takeIndex); // null when queue is empty
    } finally {
        lock.unlock();
    }
}
```

## Size

返回当前的队列长度。在对队列进行上锁后，直接返回 `count` 计数器的值：

```java
// this doc comment is overridden to remove the reference to collections
// greater in size than Integer.MAX_VALUE
/**
 * Returns the number of elements in this queue.
 *
 * @return the number of elements in this queue
 */
public int size() {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return count;
    } finally {
        lock.unlock();
    }
}
```

返回数组中剩余空间的长度。在上锁后，由数组容量减去当前队列长度就是剩余空间：

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
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        return items.length - count;
    } finally {
        lock.unlock();
    }
}
```

## Iteration

### Remove

从队头开始遍历队列，删除 `equals()` 返回 `true` 的一个元素，并返回 `true`；如果没有对队列进行修改，则返回 `false`：

```java
/**
 * Removes a single instance of the specified element from this queue,
 * if it is present.  More formally, removes an element {@code e} such
 * that {@code o.equals(e)}, if this queue contains one or more such
 * elements.
 * Returns {@code true} if this queue contained the specified element
 * (or equivalently, if this queue changed as a result of the call).
 *
 * <p>Removal of interior elements in circular array based queues
 * is an intrinsically slow and disruptive operation, so should
 * be undertaken only in exceptional circumstances, ideally
 * only when the queue is known not to be accessible by other
 * threads.
 *
 * @param o element to be removed from this queue, if present
 * @return {@code true} if this queue changed as a result of the call
 */
public boolean remove(Object o) {
    if (o == null) return false;
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count > 0) {
            final int putIndex = this.putIndex;
            int i = takeIndex;
            do {
                if (o.equals(items[i])) {
                    removeAt(i);
                    return true;
                }
                if (++i == items.length)
                    i = 0;
            } while (i != putIndex);
        }
        return false;
    } finally {
        lock.unlock();
    }
}
```

其中删除元素的具体逻辑：

* 如果被删除的元素恰好在队头，则直接出队
* 如果被删除的元素不在队头，需要把从该元素开始至队尾的每个元素向前移动
* 向可能存在的 put 线程释放信号

```java
/**
 * Deletes item at array index removeIndex.
 * Utility for remove(Object) and iterator.remove.
 * Call only when holding lock.
 */
void removeAt(final int removeIndex) {
    // assert lock.getHoldCount() == 1;
    // assert items[removeIndex] != null;
    // assert removeIndex >= 0 && removeIndex < items.length;
    final Object[] items = this.items;
    if (removeIndex == takeIndex) {
        // removing front item; just advance
        items[takeIndex] = null;
        if (++takeIndex == items.length)
            takeIndex = 0;
        count--;
        if (itrs != null)
            itrs.elementDequeued();
    } else {
        // an "interior" remove

        // slide over all others up through putIndex.
        final int putIndex = this.putIndex;
        for (int i = removeIndex;;) {
            int next = i + 1;
            if (next == items.length)
                next = 0;
            if (next != putIndex) {
                items[i] = items[next];
                i = next;
            } else {
                items[i] = null;
                this.putIndex = i;
                break;
            }
        }
        count--;
        if (itrs != null)
            itrs.removedAt(removeIndex);
    }
    notFull.signal();
}
```

### Contains

上锁并遍历数组，判断是否包含 `equals()` 返回 `true` 的元素。

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
    final Object[] items = this.items;
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        if (count > 0) {
            final int putIndex = this.putIndex;
            int i = takeIndex;
            do {
                if (o.equals(items[i]))
                    return true;
                if (++i == items.length)
                    i = 0;
            } while (i != putIndex);
        }
        return false;
    } finally {
        lock.unlock();
    }
}
```

## Iterator

> 以后再做补充，这里的迭代器实现还挺奇怪的。

---

