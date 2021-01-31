# Class - java.util.LinkedHashMap

Created by : Mr Dk.

2019 / 11 / 18 20:56

Nanjing, Jiangsu, China

---

## Definition

```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
{

}
```

Map 的哈希表和链表的实现类

值得注意的是，继承自 `HashMap`

除了具有 `HashMap` 的特性以外

还多维护了一个双向链表结构

所以能够保证元素的迭代顺序

可以用于实现 LRU

特别地，`HashMap` 在迭代时的时间复杂度与 capacity 有关

而 `LinkedHashMap` 则与 size 有关

```java
/**
 * <p>Hash table and linked list implementation of the <tt>Map</tt> interface,
 * with predictable iteration order.  This implementation differs from
 * <tt>HashMap</tt> in that it maintains a doubly-linked list running through
 * all of its entries.  This linked list defines the iteration ordering,
 * which is normally the order in which keys were inserted into the map
 * (<i>insertion-order</i>).  Note that insertion order is not affected
 * if a key is <i>re-inserted</i> into the map.  (A key <tt>k</tt> is
 * reinserted into a map <tt>m</tt> if <tt>m.put(k, v)</tt> is invoked when
 * <tt>m.containsKey(k)</tt> would return <tt>true</tt> immediately prior to
 * the invocation.)
 *
 * <p>This implementation spares its clients from the unspecified, generally
 * chaotic ordering provided by {@link HashMap} (and {@link Hashtable}),
 * without incurring the increased cost associated with {@link TreeMap}.  It
 * can be used to produce a copy of a map that has the same order as the
 * original, regardless of the original map's implementation:
 * <pre>
 *     void foo(Map m) {
 *         Map copy = new LinkedHashMap(m);
 *         ...
 *     }
 * </pre>
 * This technique is particularly useful if a module takes a map on input,
 * copies it, and later returns results whose order is determined by that of
 * the copy.  (Clients generally appreciate having things returned in the same
 * order they were presented.)
 *
 * <p>A special {@link #LinkedHashMap(int,float,boolean) constructor} is
 * provided to create a linked hash map whose order of iteration is the order
 * in which its entries were last accessed, from least-recently accessed to
 * most-recently (<i>access-order</i>).  This kind of map is well-suited to
 * building LRU caches.  Invoking the {@code put}, {@code putIfAbsent},
 * {@code get}, {@code getOrDefault}, {@code compute}, {@code computeIfAbsent},
 * {@code computeIfPresent}, or {@code merge} methods results
 * in an access to the corresponding entry (assuming it exists after the
 * invocation completes). The {@code replace} methods only result in an access
 * of the entry if the value is replaced.  The {@code putAll} method generates one
 * entry access for each mapping in the specified map, in the order that
 * key-value mappings are provided by the specified map's entry set iterator.
 * <i>No other methods generate entry accesses.</i>  In particular, operations
 * on collection-views do <i>not</i> affect the order of iteration of the
 * backing map.
 *
 * <p>The {@link #removeEldestEntry(Map.Entry)} method may be overridden to
 * impose a policy for removing stale mappings automatically when new mappings
 * are added to the map.
 *
 * <p>This class provides all of the optional <tt>Map</tt> operations, and
 * permits null elements.  Like <tt>HashMap</tt>, it provides constant-time
 * performance for the basic operations (<tt>add</tt>, <tt>contains</tt> and
 * <tt>remove</tt>), assuming the hash function disperses elements
 * properly among the buckets.  Performance is likely to be just slightly
 * below that of <tt>HashMap</tt>, due to the added expense of maintaining the
 * linked list, with one exception: Iteration over the collection-views
 * of a <tt>LinkedHashMap</tt> requires time proportional to the <i>size</i>
 * of the map, regardless of its capacity.  Iteration over a <tt>HashMap</tt>
 * is likely to be more expensive, requiring time proportional to its
 * <i>capacity</i>.
 *
 * <p>A linked hash map has two parameters that affect its performance:
 * <i>initial capacity</i> and <i>load factor</i>.  They are defined precisely
 * as for <tt>HashMap</tt>.  Note, however, that the penalty for choosing an
 * excessively high value for initial capacity is less severe for this class
 * than for <tt>HashMap</tt>, as iteration times for this class are unaffected
 * by capacity.
 *
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access a linked hash map concurrently, and at least
 * one of the threads modifies the map structurally, it <em>must</em> be
 * synchronized externally.  This is typically accomplished by
 * synchronizing on some object that naturally encapsulates the map.
 *
 * If no such object exists, the map should be "wrapped" using the
 * {@link Collections#synchronizedMap Collections.synchronizedMap}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the map:<pre>
 *   Map m = Collections.synchronizedMap(new LinkedHashMap(...));</pre>
 *
 * A structural modification is any operation that adds or deletes one or more
 * mappings or, in the case of access-ordered linked hash maps, affects
 * iteration order.  In insertion-ordered linked hash maps, merely changing
 * the value associated with a key that is already contained in the map is not
 * a structural modification.  <strong>In access-ordered linked hash maps,
 * merely querying the map with <tt>get</tt> is a structural modification.
 * </strong>)
 *
 * <p>The iterators returned by the <tt>iterator</tt> method of the collections
 * returned by all of this class's collection view methods are
 * <em>fail-fast</em>: if the map is structurally modified at any time after
 * the iterator is created, in any way except through the iterator's own
 * <tt>remove</tt> method, the iterator will throw a {@link
 * ConcurrentModificationException}.  Thus, in the face of concurrent
 * modification, the iterator fails quickly and cleanly, rather than risking
 * arbitrary, non-deterministic behavior at an undetermined time in the future.
 *
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw <tt>ConcurrentModificationException</tt> on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness:   <i>the fail-fast behavior of iterators
 * should be used only to detect bugs.</i>
 *
 * <p>The spliterators returned by the spliterator method of the collections
 * returned by all of this class's collection view methods are
 * <em><a href="Spliterator.html#binding">late-binding</a></em>,
 * <em>fail-fast</em>, and additionally report {@link Spliterator#ORDERED}.
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @implNote
 * The spliterators returned by the spliterator method of the collections
 * returned by all of this class's collection view methods are created from
 * the iterators of the corresponding collections.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch
 * @see     Object#hashCode()
 * @see     Collection
 * @see     Map
 * @see     HashMap
 * @see     TreeMap
 * @see     Hashtable
 * @since   1.4
 */
```

---

结点定义：

`LinkedHashMap.Entry<K,V>` 继承自 `HashMap.Node<K,V>`

```java
/**
 * HashMap.Node subclass for normal LinkedHashMap entries.
 */
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

除了 hash 值和 key、value 以外，多了 `before` 和 `after`

---

```java
/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;

/**
 * The iteration ordering method for this linked hash map: <tt>true</tt>
 * for access-order, <tt>false</tt> for insertion-order.
 *
 * @serial
 */
final boolean accessOrder;
```

多维护了头指针 `head` 和尾指针 `tail`

以及迭代顺序

* `true` - 按访问顺序
* `false` - 按插入顺序

---

```java
// link at the end of list
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
        head = p;
    else {
        p.before = last;
        last.after = p;
    }
}
```

将 entry 链到链表尾部

---

```java
// apply src's links to dst
private void transferLinks(LinkedHashMap.Entry<K,V> src,
                           LinkedHashMap.Entry<K,V> dst) {
    LinkedHashMap.Entry<K,V> b = dst.before = src.before;
    LinkedHashMap.Entry<K,V> a = dst.after = src.after;
    if (b == null)
        head = dst;
    else
        b.after = dst;
    if (a == null)
        tail = dst;
    else
        a.before = dst;
}
```

将链表中的 `src` 替换为 `dst` 结点，并维护所有指针的引用关系

---

```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
    LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
    LinkedHashMap.Entry<K,V> t =
        new LinkedHashMap.Entry<K,V>(q.hash, q.key, q.value, next);
    transferLinks(q, t);
    return t;
}

TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
    TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
    linkNodeLast(p);
    return p;
}

TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
    LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
    TreeNode<K,V> t = new TreeNode<K,V>(q.hash, q.key, q.value, next);
    transferLinks(q, t);
    return t;
}
```

新建普通结点或树结点

将普通结点替换为树结点，或将树结点替换为普通结点

维护双向链表的指针引用关系

---

```java
void afterNodeRemoval(Node<K,V> e) { // unlink
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}
```

在结点被移除后，维护该结点的前结点和后结点的引用关系

```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```

根据 `evict` 来决定，插入操作后，是否要删除最老的结点 (LRU)

```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    if (accessOrder && (last = tail) != e) {
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

如果 `accessOrder == true`，也就是按照访问顺序组织链表

就将被访问的结点从链表中拿出来，并塞回最后 (也是 LRU)

---

```java
/**
 * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
 * with the specified initial capacity and load factor.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}

/**
 * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
 * with the specified initial capacity and a default load factor (0.75).
 *
 * @param  initialCapacity the initial capacity
 * @throws IllegalArgumentException if the initial capacity is negative
 */
public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

/**
 * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
 * with the default initial capacity (16) and load factor (0.75).
 */
public LinkedHashMap() {
    super();
    accessOrder = false;
}

/**
 * Constructs an insertion-ordered <tt>LinkedHashMap</tt> instance with
 * the same mappings as the specified map.  The <tt>LinkedHashMap</tt>
 * instance is created with a default load factor (0.75) and an initial
 * capacity sufficient to hold the mappings in the specified map.
 *
 * @param  m the map whose mappings are to be placed in this map
 * @throws NullPointerException if the specified map is null
 */
public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}

/**
 * Constructs an empty <tt>LinkedHashMap</tt> instance with the
 * specified initial capacity, load factor and ordering mode.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @param  accessOrder     the ordering mode - <tt>true</tt> for
 *         access-order, <tt>false</tt> for insertion-order
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public LinkedHashMap(int initialCapacity,
                        float loadFactor,
                        boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

构造函数，内部直接调用 `HashMap` 的构造函数

* 大部分情况下，`accessOrder` 默认为 `false`，即按照插入顺序组织链表
* 但可以显式指定为 `true`，按照访问顺序组织链表

---

```java
/**
 * Returns <tt>true</tt> if this map maps one or more keys to the
 * specified value.
 *
 * @param value value whose presence in this map is to be tested
 * @return <tt>true</tt> if this map maps one or more keys to the
 *         specified value
 */
public boolean containsValue(Object value) {
    for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after) {
        V v = e.value;
        if (v == value || (value != null && value.equals(v)))
            return true;
    }
    return false;
}
```

由于 value 没有任何维护顺序，就遍历整个集合吧

由于实现中有了双向链表，可以直接使用双向链表遍历

时间复杂度上，比 `HashMap` 好一些

* `O(capacity)` 和 `O(size)` 的区别

---

```java
/**
 * Returns the value to which the specified key is mapped,
 * or {@code null} if this map contains no mapping for the key.
 *
 * <p>More formally, if this map contains a mapping from a key
 * {@code k} to a value {@code v} such that {@code (key==null ? k==null :
 * key.equals(k))}, then this method returns {@code v}; otherwise
 * it returns {@code null}.  (There can be at most one such mapping.)
 *
 * <p>A return value of {@code null} does not <i>necessarily</i>
 * indicate that the map contains no mapping for the key; it's also
 * possible that the map explicitly maps the key to {@code null}.
 * The {@link #containsKey containsKey} operation may be used to
 * distinguish these two cases.
 */
public V get(Object key) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return null;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}

/**
 * {@inheritDoc}
 */
public V getOrDefault(Object key, V defaultValue) {
    Node<K,V> e;
    if ((e = getNode(hash(key), key)) == null)
        return defaultValue;
    if (accessOrder)
        afterNodeAccess(e);
    return e.value;
}
```

调父类 `HashMap` 的 `get()` 来获得结点

如果双向链表按访问顺序来组织

那么调 `afterNodeAccess()` 把结点重新放到链表尾部

---

```java
/**
 * {@inheritDoc}
 */
public void clear() {
    super.clear();
    head = tail = null;
}
```

调 `HashMap` 的 `clear()`

同时置空双向链表的头尾指针

---

```java
/**
 * Returns <tt>true</tt> if this map should remove its eldest entry.
 * This method is invoked by <tt>put</tt> and <tt>putAll</tt> after
 * inserting a new entry into the map.  It provides the implementor
 * with the opportunity to remove the eldest entry each time a new one
 * is added.  This is useful if the map represents a cache: it allows
 * the map to reduce memory consumption by deleting stale entries.
 *
 * <p>Sample use: this override will allow the map to grow up to 100
 * entries and then delete the eldest entry each time a new entry is
 * added, maintaining a steady state of 100 entries.
 * <pre>
 *     private static final int MAX_ENTRIES = 100;
 *
 *     protected boolean removeEldestEntry(Map.Entry eldest) {
 *        return size() &gt; MAX_ENTRIES;
 *     }
 * </pre>
 *
 * <p>This method typically does not modify the map in any way,
 * instead allowing the map to modify itself as directed by its
 * return value.  It <i>is</i> permitted for this method to modify
 * the map directly, but if it does so, it <i>must</i> return
 * <tt>false</tt> (indicating that the map should not attempt any
 * further modification).  The effects of returning <tt>true</tt>
 * after modifying the map from within this method are unspecified.
 *
 * <p>This implementation merely returns <tt>false</tt> (so that this
 * map acts like a normal map - the eldest element is never removed).
 *
 * @param    eldest The least recently inserted entry in the map, or if
 *           this is an access-ordered map, the least recently accessed
 *           entry.  This is the entry that will be removed it this
 *           method returns <tt>true</tt>.  If the map was empty prior
 *           to the <tt>put</tt> or <tt>putAll</tt> invocation resulting
 *           in this invocation, this will be the entry that was just
 *           inserted; in other words, if the map contains a single
 *           entry, the eldest entry is also the newest.
 * @return   <tt>true</tt> if the eldest entry should be removed
 *           from the map; <tt>false</tt> if it should be retained.
 */
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```

这个函数会在调用 `put()` 或 `putAll()` 后被调用

用于决定是否删除最老的元素

这个性质可以被用于实现一个固定大小的 LRU __cache__

在调 `afterNodeInsertion()` 时，对 head 结点调这个函数进行判断

* 如果返回 `true`，就把 head 结点给删掉

在 `LinkedHashMap` 的默认实现中，返回 `false`

* 即，不删除最老的结点

可以重写这个函数，实现固定大小的 LRU cache：

```java
private static final int MAX_ENTRIES = 100;

protected boolean removeEldestEntry(Map.Entry eldest) {
    return size() > MAX_ENTRIES;
}
```

---

```java
// Iterators

abstract class LinkedHashIterator {
    LinkedHashMap.Entry<K,V> next;
    LinkedHashMap.Entry<K,V> current;
    int expectedModCount;

    LinkedHashIterator() {
        next = head;
        expectedModCount = modCount;
        current = null;
    }

    public final boolean hasNext() {
        return next != null;
    }

    final LinkedHashMap.Entry<K,V> nextNode() {
        LinkedHashMap.Entry<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        current = e;
        next = e.after;
        return e;
    }

    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        current = null;
        K key = p.key;
        removeNode(hash(key), key, null, false, false);
        expectedModCount = modCount;
    }
}

final class LinkedKeyIterator extends LinkedHashIterator
    implements Iterator<K> {
    public final K next() { return nextNode().getKey(); }
}

final class LinkedValueIterator extends LinkedHashIterator
    implements Iterator<V> {
    public final V next() { return nextNode().value; }
}

final class LinkedEntryIterator extends LinkedHashIterator
    implements Iterator<Map.Entry<K,V>> {
    public final Map.Entry<K,V> next() { return nextNode(); }
}
```

迭代器维护 next 和 current 指针

分别指向下一个迭代结点和当前迭代结点

提供三个函数：

* `hasNext()` - 是否有下一个结点
* `nextNode()` - 迭代下一个结点
* `remove()` - 调 `HashMap` 的 `removeNode()` 函数
    * `HashMap` 的 `removeNode()` 函数中会调用 `afterNodeRemoval()` 处理双向链表指针

---

## Summary

所以主要还是看 HashMap 了

LinkedHashMap 也就是多维护了一个双向链表

---

