# Class - java.util.PriorityQueue

Created by : Mr Dk.

2019 / 11 / 12 21:26

Nanjing, Jiangsu, China

---

## Definition

```java
public class PriorityQueue<E> extends AbstractQueue<E>
    implements java.io.Serializable {

}
```

优先级队列的实现。基于 **优先级 heap** 来管理数据：优先级队列中的元素根据其自然顺序，或提供的 `Comparator` 排列。

- 不允许 `null` 元素
- 不允许插入无法比较大小的元素

优先队列的大小不受限制，但是有一个内部的容量，用于管理实际用于存储元素的数组。所以容量根据队列自动增长。

迭代器无法保证遍历元素的顺序，如果需要按顺序遍历，考虑使用 `Arrays.sort(pq.toArray())`。

当前实现是非同步的，而 `java.util.concurrent.PriorityBlockingQueue` 是线程安全的。其中通过一个 ReentrantLock 来锁定优先队列的修改操作，从而保证线程安全。

时间复杂度：

- O(log(n)) - `offer()` / `poll()` / `remove()` / `add()`
- O(n) - `remove(Object)` / `contains(Object)`
- O(1) - `peek()` / `element()` / `size()`

```java
/**
 * An unbounded priority {@linkplain Queue queue} based on a priority heap.
 * The elements of the priority queue are ordered according to their
 * {@linkplain Comparable natural ordering}, or by a {@link Comparator}
 * provided at queue construction time, depending on which constructor is
 * used.  A priority queue does not permit {@code null} elements.
 * A priority queue relying on natural ordering also does not permit
 * insertion of non-comparable objects (doing so may result in
 * {@code ClassCastException}).
 *
 * <p>The <em>head</em> of this queue is the <em>least</em> element
 * with respect to the specified ordering.  If multiple elements are
 * tied for least value, the head is one of those elements -- ties are
 * broken arbitrarily.  The queue retrieval operations {@code poll},
 * {@code remove}, {@code peek}, and {@code element} access the
 * element at the head of the queue.
 *
 * <p>A priority queue is unbounded, but has an internal
 * <i>capacity</i> governing the size of an array used to store the
 * elements on the queue.  It is always at least as large as the queue
 * size.  As elements are added to a priority queue, its capacity
 * grows automatically.  The details of the growth policy are not
 * specified.
 *
 * <p>This class and its iterator implement all of the
 * <em>optional</em> methods of the {@link Collection} and {@link
 * Iterator} interfaces.  The Iterator provided in method {@link
 * #iterator()} is <em>not</em> guaranteed to traverse the elements of
 * the priority queue in any particular order. If you need ordered
 * traversal, consider using {@code Arrays.sort(pq.toArray())}.
 *
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * Multiple threads should not access a {@code PriorityQueue}
 * instance concurrently if any of the threads modifies the queue.
 * Instead, use the thread-safe {@link
 * java.util.concurrent.PriorityBlockingQueue} class.
 *
 * <p>Implementation note: this implementation provides
 * O(log(n)) time for the enqueuing and dequeuing methods
 * ({@code offer}, {@code poll}, {@code remove()} and {@code add});
 * linear time for the {@code remove(Object)} and {@code contains(Object)}
 * methods; and constant time for the retrieval methods
 * ({@code peek}, {@code element}, and {@code size}).
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.5
 * @author Josh Bloch, Doug Lea
 * @param <E> the type of elements held in this collection
 */
```

---

```java
private static final long serialVersionUID = -7720805057305804111L;

private static final int DEFAULT_INITIAL_CAPACITY = 11;

/**
 * Priority queue represented as a balanced binary heap: the two
 * children of queue[n] are queue[2*n+1] and queue[2*(n+1)].  The
 * priority queue is ordered by comparator, or by the elements'
 * natural ordering, if comparator is null: For each node n in the
 * heap and each descendant d of n, n <= d.  The element with the
 * lowest value is in queue[0], assuming the queue is nonempty.
 */
transient Object[] queue; // non-private to simplify nested class access

/**
 * The number of elements in the priority queue.
 */
private int size = 0;

/**
 * The comparator, or null if priority queue uses elements'
 * natural ordering.
 */
private final Comparator<? super E> comparator;

/**
 * The number of times this priority queue has been
 * <i>structurally modified</i>.  See AbstractList for gory details.
 */
transient int modCount = 0; // non-private to simplify nested class access
```

初始默认容量为 `11`，管理为二叉堆结构。

- `queue[0]` 是最小的元素
- `queue[n]` 的子元素是 `queue[2n+1]` 和 `queue[2n+2]`

如果 `Comparator` 为 `null`，就按照自然顺序排列。

## Constructor

可以指定初始的容量和 `Comparetor`：

- 如果未指定初始容量，则使用 `DEFAULT_INITIAL_CAPACITY`，即 11
- 如果未指定 `Comparator`，则为 null，使用自然顺序排列

```java
/**
 * Creates a {@code PriorityQueue} with the specified initial capacity
 * that orders its elements according to the specified comparator.
 *
 * @param  initialCapacity the initial capacity for this priority queue
 * @param  comparator the comparator that will be used to order this
 *         priority queue.  If {@code null}, the {@linkplain Comparable
 *         natural ordering} of the elements will be used.
 * @throws IllegalArgumentException if {@code initialCapacity} is
 *         less than 1
 */
public PriorityQueue(int initialCapacity,
                        Comparator<? super E> comparator) {
    // Note: This restriction of at least one is not actually needed,
    // but continues for 1.5 compatibility
    if (initialCapacity < 1)
        throw new IllegalArgumentException();
    this.queue = new Object[initialCapacity];
    this.comparator = comparator;
}
```

从一个集合创建优先队列，分为三种情况：

- 排序后的 Set 集合
- 另一个优先队列
- 普通集合

```java
/**
 * Creates a {@code PriorityQueue} containing the elements in the
 * specified collection.  If the specified collection is an instance of
 * a {@link SortedSet} or is another {@code PriorityQueue}, this
 * priority queue will be ordered according to the same ordering.
 * Otherwise, this priority queue will be ordered according to the
 * {@linkplain Comparable natural ordering} of its elements.
 *
 * @param  c the collection whose elements are to be placed
 *         into this priority queue
 * @throws ClassCastException if elements of the specified collection
 *         cannot be compared to one another according to the priority
 *         queue's ordering
 * @throws NullPointerException if the specified collection or any
 *         of its elements are null
 */
@SuppressWarnings("unchecked")
public PriorityQueue(Collection<? extends E> c) {
    if (c instanceof SortedSet<?>) {
        SortedSet<? extends E> ss = (SortedSet<? extends E>) c;
        this.comparator = (Comparator<? super E>) ss.comparator();
        initElementsFromCollection(ss);
    }
    else if (c instanceof PriorityQueue<?>) {
        PriorityQueue<? extends E> pq = (PriorityQueue<? extends E>) c;
        this.comparator = (Comparator<? super E>) pq.comparator();
        initFromPriorityQueue(pq);
    }
    else {
        this.comparator = null;
        initFromCollection(c);
    }
}
```

如果是有序的 Set，那么直接按 Set 的顺序拷贝为优先队列，并初始化所有 metadata：

```java
/**
 * Creates a {@code PriorityQueue} containing the elements in the
 * specified sorted set.   This priority queue will be ordered
 * according to the same ordering as the given sorted set.
 *
 * @param  c the sorted set whose elements are to be placed
 *         into this priority queue
 * @throws ClassCastException if elements of the specified sorted
 *         set cannot be compared to one another according to the
 *         sorted set's ordering
 * @throws NullPointerException if the specified sorted set or any
 *         of its elements are null
 */
@SuppressWarnings("unchecked")
public PriorityQueue(SortedSet<? extends E> c) {
    this.comparator = (Comparator<? super E>) c.comparator();
    initElementsFromCollection(c);
}

private void initElementsFromCollection(Collection<? extends E> c) {
    Object[] a = c.toArray();
    // If c.toArray incorrectly doesn't return Object[], copy it.
    if (a.getClass() != Object[].class)
        a = Arrays.copyOf(a, a.length, Object[].class);
    int len = a.length;
    if (len == 1 || this.comparator != null)
        for (int i = 0; i < len; i++)
            if (a[i] == null)
                throw new NullPointerException();
    this.queue = a;
    this.size = a.length;
}
```

如果利用另一个优先队列构造，那么直接将对应的元素和 metadata 拷贝。如果不是优先队列，则需要将元素拷贝后，对元素进行 **堆化**：因为原顺序不是堆序。

```java
/**
 * Creates a {@code PriorityQueue} containing the elements in the
 * specified priority queue.  This priority queue will be
 * ordered according to the same ordering as the given priority
 * queue.
 *
 * @param  c the priority queue whose elements are to be placed
 *         into this priority queue
 * @throws ClassCastException if elements of {@code c} cannot be
 *         compared to one another according to {@code c}'s
 *         ordering
 * @throws NullPointerException if the specified priority queue or any
 *         of its elements are null
 */
@SuppressWarnings("unchecked")
public PriorityQueue(PriorityQueue<? extends E> c) {
    this.comparator = (Comparator<? super E>) c.comparator();
    initFromPriorityQueue(c);
}

private void initFromPriorityQueue(PriorityQueue<? extends E> c) {
    if (c.getClass() == PriorityQueue.class) {
        this.queue = c.toArray();
        this.size = c.size();
    } else {
        initFromCollection(c);
    }
}

/**
 * Initializes queue array with elements from the given Collection.
 *
 * @param c the collection
 */
private void initFromCollection(Collection<? extends E> c) {
    initElementsFromCollection(c);
    heapify();
}
```

## Add

插入元素（两个函数等价）

- 将元素放在数组的最后位置
- 从该位置开始，将元素从堆的根部向上移动，直到满足堆的条件
- 如果是第一个元素，则直接放在 `queue[0]`
- 如果数组容量已经不够，那么将数组扩容

```java
/**
 * Inserts the specified element into this priority queue.
 *
 * @return {@code true} (as specified by {@link Collection#add})
 * @throws ClassCastException if the specified element cannot be
 *         compared with elements currently in this priority queue
 *         according to the priority queue's ordering
 * @throws NullPointerException if the specified element is null
 */
public boolean add(E e) {
    return offer(e);
}

/**
 * Inserts the specified element into this priority queue.
 *
 * @return {@code true} (as specified by {@link Queue#offer})
 * @throws ClassCastException if the specified element cannot be
 *         compared with elements currently in this priority queue
 *         according to the priority queue's ordering
 * @throws NullPointerException if the specified element is null
 */
public boolean offer(E e) {
    if (e == null)
        throw new NullPointerException();
    modCount++;
    int i = size;
    if (i >= queue.length)
        grow(i + 1);
    size = i + 1;
    if (i == 0)
        queue[0] = e;
    else
        siftUp(i, e);
    return true;
}
```

扩容的原则：

- 小于 64 个元素 - 新容量为 2n + 2
- 多于 64 个元素 - 扩容 50% (1.5 倍)
- 扩容要求的最小值超过 `MAX_ARRAY_SIZE`，就返回最大 `Integer`，但扩容最小值不能溢出

```java
/**
 * Increases the capacity of the array.
 *
 * @param minCapacity the desired minimum capacity
 */
private void grow(int minCapacity) {
    int oldCapacity = queue.length;
    // Double size if small; else grow by 50%
    int newCapacity = oldCapacity + ((oldCapacity < 64) ?
                                        (oldCapacity + 2) :
                                        (oldCapacity >> 1));
    // overflow-conscious code
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    queue = Arrays.copyOf(queue, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

- 查找堆顶元素：直接查看 `queue[0]` 即可
- 查找特定元素：需要在整个队列中搜索
- 是否包含特定元素：需要在整个队列中搜索

```java
@SuppressWarnings("unchecked")
public E peek() {
    return (size == 0) ? null : (E) queue[0];
}

private int indexOf(Object o) {
    if (o != null) {
        for (int i = 0; i < size; i++)
            if (o.equals(queue[i]))
                return i;
    }
    return -1;
}

/**
 * Returns {@code true} if this queue contains the specified element.
 * More formally, returns {@code true} if and only if this queue contains
 * at least one element {@code e} such that {@code o.equals(e)}.
 *
 * @param o object to be checked for containment in this queue
 * @return {@code true} if this queue contains the specified element
 */
public boolean contains(Object o) {
    return indexOf(o) != -1;
}
```

## Remove

删除特定的元素，同样也需要先遍历整个队列，找到对应元素所在的位置（下标），然后调用对应的堆操作将该元素移除。

```java
/**
 * Removes a single instance of the specified element from this queue,
 * if it is present.  More formally, removes an element {@code e} such
 * that {@code o.equals(e)}, if this queue contains one or more such
 * elements.  Returns {@code true} if and only if this queue contained
 * the specified element (or equivalently, if this queue changed as a
 * result of the call).
 *
 * @param o element to be removed from this queue, if present
 * @return {@code true} if this queue changed as a result of the call
 */
public boolean remove(Object o) {
    int i = indexOf(o);
    if (i == -1)
        return false;
    else {
        removeAt(i);
        return true;
    }
}

/**
 * Version of remove using reference equality, not equals.
 * Needed by iterator.remove.
 *
 * @param o element to be removed from this queue, if present
 * @return {@code true} if removed
 */
boolean removeEq(Object o) {
    for (int i = 0; i < size; i++) {
        if (o == queue[i]) {
            removeAt(i);
            return true;
        }
    }
    return false;
}
```

出队操作：

- 将队头 (即堆顶) 的元素删除，`size` 减小了 1
- 将 `queue[0]` 元素用数组的最后一个元素代替
- 向下调整堆

```java
@SuppressWarnings("unchecked")
public E poll() {
    if (size == 0)
        return null;
    int s = --size;
    modCount++;
    E result = (E) queue[0];
    E x = (E) queue[s];
    queue[s] = null;
    if (s != 0)
        siftDown(0, x);
    return result;
}
```

删除任意指定数组下标的元素，将数组中的最后一个元素放到被删除的位置

- 试图向下调整堆（向下调整的概率较大，因为原最后一个元素位于堆底），并返回 `null`
- 如果不能再向下调整堆，则向上调整堆
  - 如果也无法向上调整堆，则说明该位置刚好适合该元素，返回 `null`
  - 否则返回该元素本身

这个操作用于防止迭代器调用 `remove()` 后漏掉遍历的元素：

- 如果返回 `null`，说明最后一个元素被放到了做删除操作的地方，没有上浮或下沉；该元素会被漏掉，迭代器的指针需要回退一下
- 如果返回不为 `null`，说明最后一个元素移到做删除操作的地方后已经上浮；该元素会被漏掉，而且靠迭代器指针回退一格已经无法遍历到它

```java
/**
 * Removes the ith element from queue.
 *
 * Normally this method leaves the elements at up to i-1,
 * inclusive, untouched.  Under these circumstances, it returns
 * null.  Occasionally, in order to maintain the heap invariant,
 * it must swap a later element of the list with one earlier than
 * i.  Under these circumstances, this method returns the element
 * that was previously at the end of the list and is now at some
 * position before i. This fact is used by iterator.remove so as to
 * avoid missing traversing elements.
 */
@SuppressWarnings("unchecked")
private E removeAt(int i) {
    // assert i >= 0 && i < size;
    modCount++;
    int s = --size;
    if (s == i) // removed last element
        queue[i] = null;
    else {
        E moved = (E) queue[s];
        queue[s] = null;
        siftDown(i, moved);
        if (queue[i] == moved) {
            siftUp(i, moved);
            if (queue[i] != moved)
                return moved;
        }
    }
    return null;
}
```

在这种情况下，可以看看迭代器的实现。迭代器中维护一个 `forgetMeNot` 队列，用于记录所有从未被访问的位置上浮到已被访问的位置后的元素。

- 如果被提上来的最后一个元素 **下沉**，或在原删除位置 **不上浮也不下沉**；这种情况下，只有原删除位置的元素会被忽略一次，只需要让 `cursor--`，重新迭代原删除位置，就能保证所有元素都被迭代
- 如果被提上来的元素上浮，被置换到原删除位置的元素肯定已经被访问过；这种情况下，浮上去的那个元素显然被忽略了，而且退指针也没用，就将其加入 `forgetMeNot` 队列中

在常规遍历结束后，还需要遍历 `forgetMeNot` 队列，才算完成一次完整的迭代。

```java
/**
 * Returns an iterator over the elements in this queue. The iterator
 * does not return the elements in any particular order.
 *
 * @return an iterator over the elements in this queue
 */
public Iterator<E> iterator() {
    return new Itr();
}

private final class Itr implements Iterator<E> {
    /**
     * Index (into queue array) of element to be returned by
     * subsequent call to next.
     */
    private int cursor = 0;

    /**
     * Index of element returned by most recent call to next,
     * unless that element came from the forgetMeNot list.
     * Set to -1 if element is deleted by a call to remove.
     */
    private int lastRet = -1;

    /**
     * A queue of elements that were moved from the unvisited portion of
     * the heap into the visited portion as a result of "unlucky" element
     * removals during the iteration.  (Unlucky element removals are those
     * that require a siftup instead of a siftdown.)  We must visit all of
     * the elements in this list to complete the iteration.  We do this
     * after we've completed the "normal" iteration.
     *
     * We expect that most iterations, even those involving removals,
     * will not need to store elements in this field.
     */
    private ArrayDeque<E> forgetMeNot = null;

    /**
     * Element returned by the most recent call to next iff that
     * element was drawn from the forgetMeNot list.
     */
    private E lastRetElt = null;

    /**
     * The modCount value that the iterator believes that the backing
     * Queue should have.  If this expectation is violated, the iterator
     * has detected concurrent modification.
     */
    private int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor < size ||
            (forgetMeNot != null && !forgetMeNot.isEmpty());
    }

    @SuppressWarnings("unchecked")
    public E next() {
        if (expectedModCount != modCount)
            throw new ConcurrentModificationException();
        if (cursor < size)
            return (E) queue[lastRet = cursor++];
        if (forgetMeNot != null) {
            lastRet = -1;
            lastRetElt = forgetMeNot.poll();
            if (lastRetElt != null)
                return lastRetElt;
        }
        throw new NoSuchElementException();
    }

    public void remove() {
        if (expectedModCount != modCount)
            throw new ConcurrentModificationException();
        if (lastRet != -1) {
            E moved = PriorityQueue.this.removeAt(lastRet);
            lastRet = -1;
            if (moved == null)
                cursor--;
            else {
                if (forgetMeNot == null)
                    forgetMeNot = new ArrayDeque<>();
                forgetMeNot.add(moved);
            }
        } else if (lastRetElt != null) {
            PriorityQueue.this.removeEq(lastRetElt);
            lastRetElt = null;
        } else {
            throw new IllegalStateException();
        }
        expectedModCount = modCount;
    }
}
```

在逻辑上将 `x` 元素插入到 `k` 位置 (实际上暂时不用)。因为要保证其堆的合法性而将 `x` 上浮到堆的某个位置，直到大于等于它的 parent，或成为 root。

```java
/**
 * Inserts item x at position k, maintaining heap invariant by
 * promoting x up the tree until it is greater than or equal to
 * its parent, or is the root.
 *
 * To simplify and speed up coercions and comparisons. the
 * Comparable and Comparator versions are separated into different
 * methods that are otherwise identical. (Similarly for siftDown.)
 *
 * @param k the position to fill
 * @param x the item to insert
 */
private void siftUp(int k, E x) {
    if (comparator != null)
        siftUpUsingComparator(k, x);
    else
        siftUpComparable(k, x);
}

@SuppressWarnings("unchecked")
private void siftUpComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>) x;
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (key.compareTo((E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = key;
}

@SuppressWarnings("unchecked")
private void siftUpUsingComparator(int k, E x) {
    while (k > 0) {
        int parent = (k - 1) >>> 1;
        Object e = queue[parent];
        if (comparator.compare(x, (E) e) >= 0)
            break;
        queue[k] = e;
        k = parent;
    }
    queue[k] = x;
}
```

将 `x` 插入到 `k` 位置，为维护堆合法性，将其向堆下移动，直到其小于等于其 children，或成为叶子结点。`half` 为第一个非叶子结点，如果 `k` 已经是叶子结点，则不用做操作了。

```java
/**
 * Inserts item x at position k, maintaining heap invariant by
 * demoting x down the tree repeatedly until it is less than or
 * equal to its children or is a leaf.
 *
 * @param k the position to fill
 * @param x the item to insert
 */
private void siftDown(int k, E x) {
    if (comparator != null)
        siftDownUsingComparator(k, x);
    else
        siftDownComparable(k, x);
}

@SuppressWarnings("unchecked")
private void siftDownComparable(int k, E x) {
    Comparable<? super E> key = (Comparable<? super E>)x;
    int half = size >>> 1;        // loop while a non-leaf
    while (k < half) {
        int child = (k << 1) + 1; // assume left child is least
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            ((Comparable<? super E>) c).compareTo((E) queue[right]) > 0)
            c = queue[child = right];
        if (key.compareTo((E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = key;
}

@SuppressWarnings("unchecked")
private void siftDownUsingComparator(int k, E x) {
    int half = size >>> 1;
    while (k < half) {
        int child = (k << 1) + 1;
        Object c = queue[child];
        int right = child + 1;
        if (right < size &&
            comparator.compare((E) c, (E) queue[right]) > 0)
            c = queue[child = right];
        if (comparator.compare(x, (E) c) <= 0)
            break;
        queue[k] = c;
        k = child;
    }
    queue[k] = x;
}
```

从第一个非叶子结点开始建立堆序，将每一个非叶子结点做一次下沉操作。

```java
/**
 * Establishes the heap invariant (described above) in the entire tree,
 * assuming nothing about the order of the elements prior to the call.
 */
@SuppressWarnings("unchecked")
private void heapify() {
    for (int i = (size >>> 1) - 1; i >= 0; i--)
        siftDown(i, (E) queue[i]);
}
```

## Summary

根据代码中的比较逻辑，Java 维护的堆是一个小顶堆，堆顶是最小元素。如果想要颠覆这一比较方法，那就自己实现 `Comparator` 颠倒过来吧。顺带复习了一下堆 - 发现自己已经快忘了 😥

参考：[数据结构与算法 (4) —— 优先队列和堆](https://www.cnblogs.com/wmyskxz/p/9301021.html)
