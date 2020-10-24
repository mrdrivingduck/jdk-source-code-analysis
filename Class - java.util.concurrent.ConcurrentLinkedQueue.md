# Class - java.util.concurrent.ConcurrentLinkedQueue

Created by : Mr Dk.

2020 / 10 / 24 👨‍💻 13:56

Nanjing, Jiangsu, China

---

## Definition

基于链结点实现的并发非阻塞 FIFO 队列。

```java
/**
 * An unbounded thread-safe {@linkplain Queue queue} based on linked nodes.
 * This queue orders elements FIFO (first-in-first-out).
 * The <em>head</em> of the queue is that element that has been on the
 * queue the longest time.
 * The <em>tail</em> of the queue is that element that has been on the
 * queue the shortest time. New elements
 * are inserted at the tail of the queue, and the queue retrieval
 * operations obtain elements at the head of the queue.
 * A {@code ConcurrentLinkedQueue} is an appropriate choice when
 * many threads will share access to a common collection.
 * Like most other concurrent collection implementations, this class
 * does not permit the use of {@code null} elements.
 *
 * <p>This implementation employs an efficient <em>non-blocking</em>
 * algorithm based on one described in <a
 * href="http://www.cs.rochester.edu/u/michael/PODC96.html"> Simple,
 * Fast, and Practical Non-Blocking and Blocking Concurrent Queue
 * Algorithms</a> by Maged M. Michael and Michael L. Scott.
 *
 * <p>Iterators are <i>weakly consistent</i>, returning elements
 * reflecting the state of the queue at some point at or since the
 * creation of the iterator.  They do <em>not</em> throw {@link
 * java.util.ConcurrentModificationException}, and may proceed concurrently
 * with other operations.  Elements contained in the queue since the creation
 * of the iterator will be returned exactly once.
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
 * <p>This class and its iterator implement all of the <em>optional</em>
 * methods of the {@link Queue} and {@link Iterator} interfaces.
 *
 * <p>Memory consistency effects: As with other concurrent
 * collections, actions in a thread prior to placing an object into a
 * {@code ConcurrentLinkedQueue}
 * <a href="package-summary.html#MemoryVisibility"><i>happen-before</i></a>
 * actions subsequent to the access or removal of that element from
 * the {@code ConcurrentLinkedQueue} in another thread.
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <E> the type of elements held in this collection
 */
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable {
    private static final long serialVersionUID = 196745693267521676L;

}
```

---

## Node Definition

链结点的定义包含两个部分：

* `item` 为结点的值
* `next` 为链表指针

其中，为 `item` 和 `next` 提供了 CAS 的赋值函数，为 `next` 提供了非线程安全的赋值函数 (用于一些不会发生竞争的场合)。

```java
private static class Node<E> {
    volatile E item;
    volatile Node<E> next;

    /**
     * Constructs a new node.  Uses relaxed write because item can
     * only be seen after publication via casNext.
     */
    Node(E item) {
        UNSAFE.putObject(this, itemOffset, item);
    }

    boolean casItem(E cmp, E val) {
        return UNSAFE.compareAndSwapObject(this, itemOffset, cmp, val);
    }

    void lazySetNext(Node<E> val) {
        UNSAFE.putOrderedObject(this, nextOffset, val);
    }

    boolean casNext(Node<E> cmp, Node<E> val) {
        return UNSAFE.compareAndSwapObject(this, nextOffset, cmp, val);
    }

    // Unsafe mechanics

    private static final sun.misc.Unsafe UNSAFE;
    private static final long itemOffset;
    private static final long nextOffset;

    static {
        try {
            UNSAFE = sun.misc.Unsafe.getUnsafe();
            Class<?> k = Node.class;
            itemOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("item"));
            nextOffset = UNSAFE.objectFieldOffset
                (k.getDeclaredField("next"));
        } catch (Exception e) {
            throw new Error(e);
        }
    }
}
```

## Fields

分别设置了头尾指针，由 `transient` 和 `volatile` 关键字修饰。

> `transient` 会使对象被 serialization 序列化时忽略这个字段，生命周期仅存在于内存。静态变量也不会被序列化。

```java
/**
 * A node from which the first live (non-deleted) node (if any)
 * can be reached in O(1) time.
 * Invariants:
 * - all live nodes are reachable from head via succ()
 * - head != null
 * - (tmp = head).next != tmp || tmp != head
 * Non-invariants:
 * - head.item may or may not be null.
 * - it is permitted for tail to lag behind head, that is, for tail
 *   to not be reachable from head!
 */
private transient volatile Node<E> head;

/**
 * A node from which the last node on list (that is, the unique
 * node with node.next == null) can be reached in O(1) time.
 * Invariants:
 * - the last node is always reachable from tail via succ()
 * - tail != null
 * Non-invariants:
 * - tail.item may or may not be null.
 * - it is permitted for tail to lag behind head, that is, for tail
 *   to not be reachable from head!
 * - tail.next may or may not be self-pointing to tail.
 */
private transient volatile Node<E> tail;
```

## Constructor

构造函数负责初始化头、尾结点。可以选择在构造过程中是否传入一个初始集合。可以看到，由于初始化时不存在多线程竞争，所以对结点的 `next` 指针进行赋值时使用的是 `lazySetNext()`，而不是 `casNext()`。

```java
/**
 * Creates a {@code ConcurrentLinkedQueue} that is initially empty.
 */
public ConcurrentLinkedQueue() {
    head = tail = new Node<E>(null);
}

/**
 * Creates a {@code ConcurrentLinkedQueue}
 * initially containing the elements of the given collection,
 * added in traversal order of the collection's iterator.
 *
 * @param c the collection of elements to initially contain
 * @throws NullPointerException if the specified collection or any
 *         of its elements are null
 */
public ConcurrentLinkedQueue(Collection<? extends E> c) {
    Node<E> h = null, t = null;
    for (E e : c) {
        checkNotNull(e); // 确保元素不为 null
        Node<E> newNode = new Node<E>(e); // 新结点
        if (h == null)
            h = t = newNode; // 新结点成为队列中首个元素
        else {
            // t 为已入队列的最后一个结点
            t.lazySetNext(newNode); // t 链上新结点
            t = newNode; // 新结点成为队列中的最后一个结点
        }
    }
    if (h == null)
        h = t = new Node<E>(null); // 集合为空，初始化空的头尾结点
    head = h;
    tail = t;
}
```

## Input Operations

入队操作本质上要做的事情有两件：

1. 将新结点设置为当前队尾结点的 `next` 域中
2. 更新队列的 `tail` 指针

而由于类的实现中没有锁，这两个操作本身是原子的，但无法保证这两个操作的整体原子性。因此，`tail` 指针并不总是指向当前队列中的最后一个指针。首先，需要从 `tail` 开始定位到真正的队尾结点，并通过 CAS 操作将入队结点设置为队尾结点的 `next`。如果 CAS 失败，那么就需要重新定位队列真正的队尾结点。

如果要保证 `tail` 永远是队列的尾结点，需要使用 CAS + 循环的方式实现。这样逻辑比较清楚，但是会导致较多次数的 CAS 操作。目前的实现减少了 CAS 更新 `tail` 的次数，使得入队时可能需要多循环一次以定位真正的队尾结点，但仍然提高了效率 - 通过增加对 `volatile` 变量的读操作减少了对 `volatile` 变量的写操作。

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
    return offer(e);
}

/**
 * Inserts the specified element at the tail of this queue.
 * As the queue is unbounded, this method will never return {@code false}.
 *
 * @return {@code true} (as specified by {@link Queue#offer})
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);

    for (Node<E> t = tail, p = t;;) {
        Node<E> q = p.next;
        if (q == null) {
            // p is last node
            if (p.casNext(null, newNode)) {
                // Successful CAS is the linearization point
                // for e to become an element of this queue,
                // and for newNode to become "live".
                if (p != t) // hop two nodes at a time
                    casTail(t, newNode);  // Failure is OK.
                return true;
            }
            // Lost CAS race to another thread; re-read next
        }
        else if (p == q)
            // We have fallen off list.  If tail is unchanged, it
            // will also be off-list, in which case we need to
            // jump to head, from which all live nodes are always
            // reachable.  Else the new tail is a better bet.
            p = (t != (t = tail)) ? t : head;
        else
            // Check for tail updates after two hops.
            p = (p != t && t != (t = tail)) ? t : q;
    }
}
```

## Output Operations

出队操作本质上要做的事也有两件：

1. 将结点的 `item` 置为 `null` (清空结点对元素的引用)
2. 更新队列 `head` 指针

首先从 `head` 指针所指的结点开始搜素，查看结点的 `item`：

* 如果结点的 `item` 为空，那么另一个结点已经成功进行了出队操作
* 如果结点的 `item` 不为空，则试图 CAS 将其置为 `null`
    * 如果 CAS 成功，那么返回 `item`，并尝试更新 `head`
    * 如果 CAS 失败，那么说明另一个线程已经进行出队操作并更新了 `head`，所以要重新定位队头结点

```java
public E poll() {
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            E item = p.item;

            if (item != null && p.casItem(item, null)) {
                // Successful CAS is the linearization point
                // for item to be removed from this queue.
                if (p != h) // hop two nodes at a time
                    updateHead(h, ((q = p.next) != null) ? q : p);
                return item;
            }
            else if ((q = p.next) == null) {
                updateHead(h, p);
                return null;
            }
            else if (p == q)
                continue restartFromHead;
            else
                p = q;
        }
    }
}
```

其中更新 `head` 指针的 `updateHead()` 函数会将 `head` 通过 CAS 设置为新的结点 `p`，成功后让老结点的 `next` 指向自己，从而脱离链表。不让 `next` 等于 `null` 的原因在于，其它访问这个结点的线程会误以为这个结点是 `tail`。

```java
/**
 * Tries to CAS head to p. If successful, repoint old head to itself
 * as sentinel for succ(), below.
 */
final void updateHead(Node<E> h, Node<E> p) {
    if (h != p && casHead(h, p))
        h.lazySetNext(h);
}
```

Peek 操作本质上只是想查看队头元素，而不进行出队操作，所以没有 CAS 操作，大致流程类似。首先从 `head` 开始搜素：

* 如果 `item` 不为空，那么直接返回 `item` 的引用，并更新 `head`
* 如果 `item` 为空，说明有线程已经成功进行了出队操作并修改了 `head`，所以需要重新定位队头结点

```java
public E peek() {
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            E item = p.item;
            if (item != null || (q = p.next) == null) {
                updateHead(h, p);
                return item;
            }
            else if (p == q)
                continue restartFromHead;
            else
                p = q;
        }
    }
}
```

## Size

在这里获取 size 实际上是不精确的，因为 size 的统计需要遍历所有的链结点，由于没有锁，可能存在并发的入队或出队操作：

```java
/**
 * Returns the number of elements in this queue.  If this queue
 * contains more than {@code Integer.MAX_VALUE} elements, returns
 * {@code Integer.MAX_VALUE}.
 *
 * <p>Beware that, unlike in most collections, this method is
 * <em>NOT</em> a constant-time operation. Because of the
 * asynchronous nature of these queues, determining the current
 * number of elements requires an O(n) traversal.
 * Additionally, if elements are added or removed during execution
 * of this method, the returned result may be inaccurate.  Thus,
 * this method is typically not very useful in concurrent
 * applications.
 *
 * @return the number of elements in this queue
 */
public int size() {
    int count = 0;
    for (Node<E> p = first(); p != null; p = succ(p))
        if (p.item != null)
            // Collection.size() spec says to max out
            if (++count == Integer.MAX_VALUE)
                break;
    return count;
}
```

由于遍历操作需要获取第一个结点和下一个结点，分别对这两个操作实现了子函数：

* `first()` 函数与 `peek()` 的原理一致，但是返回的是结点而不是结点中的 `item`
* `succ()` 返回下一个结点

```java
/**
 * Returns the first live (non-deleted) node on list, or null if none.
 * This is yet another variant of poll/peek; here returning the
 * first node, not element.  We could make peek() a wrapper around
 * first(), but that would cost an extra volatile read of item,
 * and the need to add a retry loop to deal with the possibility
 * of losing a race to a concurrent poll().
 */
Node<E> first() {
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            boolean hasItem = (p.item != null);
            if (hasItem || (q = p.next) == null) {
                updateHead(h, p);
                return hasItem ? p : null;
            }
            else if (p == q)
                continue restartFromHead;
            else
                p = q;
        }
    }
}
```

如果结点的 `next` 指向自身，说明当前结点是头结点；否则指向下一个结点。

```java
/**
 * Returns the successor of p, or the head node if p.next has been
 * linked to self, which will only be true if traversing with a
 * stale pointer that is now off the list.
 */
final Node<E> succ(Node<E> p) {
    Node<E> next = p.next;
    return (p == next) ? head : next;
}
```

---

## References

[知乎专栏 - Java 并发 ConcurrentLinkedQueue 源码学习与总结](https://zhuanlan.zhihu.com/p/266610706)

[非阻塞算法在并发容器中的实现](https://developer.ibm.com/zh/articles/j-lo-concurrent/)

---

