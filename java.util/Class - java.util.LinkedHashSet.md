# Class - java.util.LinkedHashSet

Created by : Mr Dk.

2019 / 11 / 22 13:24

Nanjing, Jiangsu, China

---

## Definition

```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable {

}
```

与 HashSet 不同，在所有元素之间维护了一个双向链表

* 定义了迭代顺序
* 以元素的插入顺序为顺序
* 重新插入元素不会影响元素在链表中的顺序

该集合提供所有 Set 的基本操作

* 允许 `null` 元素

提供常数时间的增、删、查操作，但会比 HashSet 慢一些

* 因为要额外维护双向链表

但是遍历时间只和 `size` 有关，与 `capacity` 无关

与 HashSet 一样，`capacity` 和 `load factor` 是影响性能的两个参数

* 但是选择较大的 `capacity` 对性能带来的影响没有像对 HashSet 那样大

该集合的实现不是线程安全的

* 外部同步
* `Set s = Collections.synchronizedSet(new LinkedHashSet(...));`

```java
/**
 * <p>Hash table and linked list implementation of the <tt>Set</tt> interface,
 * with predictable iteration order.  This implementation differs from
 * <tt>HashSet</tt> in that it maintains a doubly-linked list running through
 * all of its entries.  This linked list defines the iteration ordering,
 * which is the order in which elements were inserted into the set
 * (<i>insertion-order</i>).  Note that insertion order is <i>not</i> affected
 * if an element is <i>re-inserted</i> into the set.  (An element <tt>e</tt>
 * is reinserted into a set <tt>s</tt> if <tt>s.add(e)</tt> is invoked when
 * <tt>s.contains(e)</tt> would return <tt>true</tt> immediately prior to
 * the invocation.)
 *
 * <p>This implementation spares its clients from the unspecified, generally
 * chaotic ordering provided by {@link HashSet}, without incurring the
 * increased cost associated with {@link TreeSet}.  It can be used to
 * produce a copy of a set that has the same order as the original, regardless
 * of the original set's implementation:
 * <pre>
 *     void foo(Set s) {
 *         Set copy = new LinkedHashSet(s);
 *         ...
 *     }
 * </pre>
 * This technique is particularly useful if a module takes a set on input,
 * copies it, and later returns results whose order is determined by that of
 * the copy.  (Clients generally appreciate having things returned in the same
 * order they were presented.)
 *
 * <p>This class provides all of the optional <tt>Set</tt> operations, and
 * permits null elements.  Like <tt>HashSet</tt>, it provides constant-time
 * performance for the basic operations (<tt>add</tt>, <tt>contains</tt> and
 * <tt>remove</tt>), assuming the hash function disperses elements
 * properly among the buckets.  Performance is likely to be just slightly
 * below that of <tt>HashSet</tt>, due to the added expense of maintaining the
 * linked list, with one exception: Iteration over a <tt>LinkedHashSet</tt>
 * requires time proportional to the <i>size</i> of the set, regardless of
 * its capacity.  Iteration over a <tt>HashSet</tt> is likely to be more
 * expensive, requiring time proportional to its <i>capacity</i>.
 *
 * <p>A linked hash set has two parameters that affect its performance:
 * <i>initial capacity</i> and <i>load factor</i>.  They are defined precisely
 * as for <tt>HashSet</tt>.  Note, however, that the penalty for choosing an
 * excessively high value for initial capacity is less severe for this class
 * than for <tt>HashSet</tt>, as iteration times for this class are unaffected
 * by capacity.
 *
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access a linked hash set concurrently, and at least
 * one of the threads modifies the set, it <em>must</em> be synchronized
 * externally.  This is typically accomplished by synchronizing on some
 * object that naturally encapsulates the set.
 *
 * If no such object exists, the set should be "wrapped" using the
 * {@link Collections#synchronizedSet Collections.synchronizedSet}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the set: <pre>
 *   Set s = Collections.synchronizedSet(new LinkedHashSet(...));</pre>
 *
 * <p>The iterators returned by this class's <tt>iterator</tt> method are
 * <em>fail-fast</em>: if the set is modified at any time after the iterator
 * is created, in any way except through the iterator's own <tt>remove</tt>
 * method, the iterator will throw a {@link ConcurrentModificationException}.
 * Thus, in the face of concurrent modification, the iterator fails quickly
 * and cleanly, rather than risking arbitrary, non-deterministic behavior at
 * an undetermined time in the future.
 *
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw <tt>ConcurrentModificationException</tt> on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness:   <i>the fail-fast behavior of iterators
 * should be used only to detect bugs.</i>
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <E> the type of elements maintained by this set
 *
 * @author  Josh Bloch
 * @see     Object#hashCode()
 * @see     Collection
 * @see     Set
 * @see     HashSet
 * @see     TreeSet
 * @see     Hashtable
 * @since   1.4
 */
```

---

```java
/**
 * Constructs a new, empty linked hash set with the specified initial
 * capacity and load factor.
 *
 * @param      initialCapacity the initial capacity of the linked hash set
 * @param      loadFactor      the load factor of the linked hash set
 * @throws     IllegalArgumentException  if the initial capacity is less
 *               than zero, or if the load factor is nonpositive
 */
public LinkedHashSet(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor, true);
}

/**
 * Constructs a new, empty linked hash set with the specified initial
 * capacity and the default load factor (0.75).
 *
 * @param   initialCapacity   the initial capacity of the LinkedHashSet
 * @throws  IllegalArgumentException if the initial capacity is less
 *              than zero
 */
public LinkedHashSet(int initialCapacity) {
    super(initialCapacity, .75f, true);
}

/**
 * Constructs a new, empty linked hash set with the default initial
 * capacity (16) and load factor (0.75).
 */
public LinkedHashSet() {
    super(16, .75f, true);
}

/**
 * Constructs a new linked hash set with the same elements as the
 * specified collection.  The linked hash set is created with an initial
 * capacity sufficient to hold the elements in the specified collection
 * and the default load factor (0.75).
 *
 * @param c  the collection whose elements are to be placed into
 *           this set
 * @throws NullPointerException if the specified collection is null
 */
public LinkedHashSet(Collection<? extends E> c) {
    super(Math.max(2*c.size(), 11), .75f, true);
    addAll(c);
}
```

由于 `LinkedHashSet` 继承自 `HashSet`

而 HashSet 内部已经维护了 HashMap

所以成员变量中不再需要维护一个 Map

默认的初始 capacity 为 `16`，load factory 为 `0.75f`

---

但是源码到这里就结束了？？？

逗我呢？双向链表呢？

虽然是一路继承下来了

可能 `LinkedHashSet` 借用了 `LinkedHashMap` 中的实现方式

但并没有发现 `LinkedHashSet` 和 `LinkedHashMap` 有什么依赖关系

？？？？？？ 😓🥱

---

