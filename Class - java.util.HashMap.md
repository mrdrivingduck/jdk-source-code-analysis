# Class - java.util.HashMap

Created by : Mr Dk.

2019 / 11 / 16 16:20

Nanjing, Jiangsu, China

---

## Definition

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {

}
```

基于 Hash 表的 Map 接口实现

提供了所有 Map 的可选操作

并允许 `null` 的 key 或 value

* 除了不同步和允许 `null` 以外，与 `Hashtable` 基本等价

这个实现类对于 Map 中的顺序不作任何保证

* 特别地，不保证顺序会随着时间而保持不变

关于时间复杂度：

* 如果哈希函数将元素均匀分布在桶中，`put()` 和 `get()` 将达到 `O(1)`
* 遍历集合，将与 `capacity` + `size` 的时间成比例

因此，如果迭代性能很重要

* 最好不要将初始的 `capacity` 设得过高
* 或者说，不要将 `load factor` 设置得过低

这两个因素影响着 HashMap 的性能

* `capacity` 指 hash table 中桶的个数
* `load factor` 指的是在 hash table 自动增长桶容量前，hash table 可以被装得多满

当 hash table 中的 `capacity` 和 `load factor` 超出，会进行 __rehashed__

* 内部数据结构会重建
* 桶个数提升两倍

在通常情况下，保持 `load factor` 为 __0.75__ 是时间开销和空间开销的一个折衷

* 如果有很多 entry 要被存储到 HashMap 中
    * 设置足够的初始 capacity 比自动的 rehash 更有效
    * 使用相同的 hashcode 作为 key 会降低性能

该实现类不同步

* 如果有超过 1 个线程对集合进行结构性修改 (增删 entry)，必须在外部被同步
    * 要么被封装集合的对象同步
    * 如果没有，则 `Map m = Collections.synchronizedMap(new HashMap(...));`

在迭代器迭代期间，除了使用迭代器自身的 `remove()` 外

任何对集合的结构修改都会导致 `ConcurrentModificationException`

* 但不对非同步的并发修改作出完全保证
* 因此不能利用这种异常来保证正确性
* 仅用于检测 bug

```java
/**
 * Hash table based implementation of the <tt>Map</tt> interface.  This
 * implementation provides all of the optional map operations, and permits
 * <tt>null</tt> values and the <tt>null</tt> key.  (The <tt>HashMap</tt>
 * class is roughly equivalent to <tt>Hashtable</tt>, except that it is
 * unsynchronized and permits nulls.)  This class makes no guarantees as to
 * the order of the map; in particular, it does not guarantee that the order
 * will remain constant over time.
 *
 * <p>This implementation provides constant-time performance for the basic
 * operations (<tt>get</tt> and <tt>put</tt>), assuming the hash function
 * disperses the elements properly among the buckets.  Iteration over
 * collection views requires time proportional to the "capacity" of the
 * <tt>HashMap</tt> instance (the number of buckets) plus its size (the number
 * of key-value mappings).  Thus, it's very important not to set the initial
 * capacity too high (or the load factor too low) if iteration performance is
 * important.
 *
 * <p>An instance of <tt>HashMap</tt> has two parameters that affect its
 * performance: <i>initial capacity</i> and <i>load factor</i>.  The
 * <i>capacity</i> is the number of buckets in the hash table, and the initial
 * capacity is simply the capacity at the time the hash table is created.  The
 * <i>load factor</i> is a measure of how full the hash table is allowed to
 * get before its capacity is automatically increased.  When the number of
 * entries in the hash table exceeds the product of the load factor and the
 * current capacity, the hash table is <i>rehashed</i> (that is, internal data
 * structures are rebuilt) so that the hash table has approximately twice the
 * number of buckets.
 *
 * <p>As a general rule, the default load factor (.75) offers a good
 * tradeoff between time and space costs.  Higher values decrease the
 * space overhead but increase the lookup cost (reflected in most of
 * the operations of the <tt>HashMap</tt> class, including
 * <tt>get</tt> and <tt>put</tt>).  The expected number of entries in
 * the map and its load factor should be taken into account when
 * setting its initial capacity, so as to minimize the number of
 * rehash operations.  If the initial capacity is greater than the
 * maximum number of entries divided by the load factor, no rehash
 * operations will ever occur.
 *
 * <p>If many mappings are to be stored in a <tt>HashMap</tt>
 * instance, creating it with a sufficiently large capacity will allow
 * the mappings to be stored more efficiently than letting it perform
 * automatic rehashing as needed to grow the table.  Note that using
 * many keys with the same {@code hashCode()} is a sure way to slow
 * down performance of any hash table. To ameliorate impact, when keys
 * are {@link Comparable}, this class may use comparison order among
 * keys to help break ties.
 *
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access a hash map concurrently, and at least one of
 * the threads modifies the map structurally, it <i>must</i> be
 * synchronized externally.  (A structural modification is any operation
 * that adds or deletes one or more mappings; merely changing the value
 * associated with a key that an instance already contains is not a
 * structural modification.)  This is typically accomplished by
 * synchronizing on some object that naturally encapsulates the map.
 *
 * If no such object exists, the map should be "wrapped" using the
 * {@link Collections#synchronizedMap Collections.synchronizedMap}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the map:<pre>
 *   Map m = Collections.synchronizedMap(new HashMap(...));</pre>
 *
 * <p>The iterators returned by all of this class's "collection view methods"
 * are <i>fail-fast</i>: if the map is structurally modified at any time after
 * the iterator is created, in any way except through the iterator's own
 * <tt>remove</tt> method, the iterator will throw a
 * {@link ConcurrentModificationException}.  Thus, in the face of concurrent
 * modification, the iterator fails quickly and cleanly, rather than risking
 * arbitrary, non-deterministic behavior at an undetermined time in the
 * future.
 *
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw <tt>ConcurrentModificationException</tt> on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness: <i>the fail-fast behavior of iterators
 * should be used only to detect bugs.</i>
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Doug Lea
 * @author  Josh Bloch
 * @author  Arthur van Hoff
 * @author  Neal Gafter
 * @see     Object#hashCode()
 * @see     Collection
 * @see     Map
 * @see     TreeMap
 * @see     Hashtable
 * @since   1.2
 */
```

---

一些实现细节：

* 桶数组，每个桶可以存放 hash 相同的元素
* 当元素 hash 相同时 (hash 碰撞)
    * 将 hash 相同的元素构造为链表
    * 当链表长度超过阈值时，将链表转化为红黑树

---

```java
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16

/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the constructors with arguments.
 * MUST be a power of two <= 1<<30.
 */
static final int MAXIMUM_CAPACITY = 1 << 30;

/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
 * The bin count threshold for using a tree rather than list for a
 * bin.  Bins are converted to trees when adding an element to a
 * bin with at least this many nodes. The value must be greater
 * than 2 and should be at least 8 to mesh with assumptions in
 * tree removal about conversion back to plain bins upon
 * shrinkage.
 */
static final int TREEIFY_THRESHOLD = 8;

/**
 * The bin count threshold for untreeifying a (split) bin during a
 * resize operation. Should be less than TREEIFY_THRESHOLD, and at
 * most 6 to mesh with shrinkage detection under removal.
 */
static final int UNTREEIFY_THRESHOLD = 6;

/**
 * The smallest table capacity for which bins may be treeified.
 * (Otherwise the table is resized if too many nodes in a bin.)
 * Should be at least 4 * TREEIFY_THRESHOLD to avoid conflicts
 * between resizing and treeification thresholds.
 */
static final int MIN_TREEIFY_CAPACITY = 64;
```

可以看到，默认的初始化容量为 16

最大的容量为 1 << 30

当冲突的链表长度超过 8 时，将链表转化为树

当树的大小低于 6 时，将树转换为链表

当容量低于 64 时，hash 冲突不会被树化

---

```java
/**
 * Computes key.hashCode() and spreads (XORs) higher bits of hash
 * to lower.  Because the table uses power-of-two masking, sets of
 * hashes that vary only in bits above the current mask will
 * always collide. (Among known examples are sets of Float keys
 * holding consecutive whole numbers in small tables.)  So we
 * apply a transform that spreads the impact of higher bits
 * downward. There is a tradeoff between speed, utility, and
 * quality of bit-spreading. Because many common sets of hashes
 * are already reasonably distributed (so don't benefit from
 * spreading), and because we use trees to handle large sets of
 * collisions in bins, we just XOR some shifted bits in the
 * cheapest possible way to reduce systematic lossage, as well as
 * to incorporate impact of the highest bits that would otherwise
 * never be used in index calculations because of table bounds.
 */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

计算指定 key 对应的 hash 值

> 具体为什么要这么算没有看懂

---

```java
/**
 * The table, initialized on first use, and resized as
 * necessary. When allocated, length is always a power of two.
 * (We also tolerate length zero in some operations to allow
 * bootstrapping mechanics that are currently not needed.)
 */
transient Node<K,V>[] table;

/**
 * Holds cached entrySet(). Note that AbstractMap fields are used
 * for keySet() and values().
 */
transient Set<Map.Entry<K,V>> entrySet;

/**
 * The number of key-value mappings contained in this map.
 */
transient int size;

/**
 * The number of times this HashMap has been structurally modified
 * Structural modifications are those that change the number of mappings in
 * the HashMap or otherwise modify its internal structure (e.g.,
 * rehash).  This field is used to make iterators on Collection-views of
 * the HashMap fail-fast.  (See ConcurrentModificationException).
 */
transient int modCount;

/**
 * The next size value at which to resize (capacity * load factor).
 *
 * @serial
 */
// (The javadoc description is true upon serialization.
// Additionally, if the table array has not been allocated, this
// field holds the initial array capacity, or zero signifying
// DEFAULT_INITIAL_CAPACITY.)
int threshold;

/**
 * The load factor for the hash table.
 *
 * @serial
 */
final float loadFactor;
```

记录了集合的当前状态

* 桶数组
* load factor
* 元素个数
* threshold - 下一次重新分配前的阈值
* ...

---

```java
/**
 * Returns a power of two size for the given target capacity.
 */
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

返回一个给定容量的 2 的整幂

保证数组的容量总是 2 的幂

无法解释原理

* 反正按照代码逻辑推理推理
* 最后一次将 16 bit 右移，充斥整个 32-bit integer 数据
* 保证 integer 前面的 bit 都是 0，后面的 bit 都是 1，即 `2^n - 1`

那么 `2^n - 1 + 1` 一定是 `2^n`

---

```java
/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and load factor.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                            initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                            loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

/**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and the default load factor (0.75).
 *
 * @param  initialCapacity the initial capacity.
 * @throws IllegalArgumentException if the initial capacity is negative.
 */
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

/**
 * Constructs an empty <tt>HashMap</tt> with the default initial capacity
 * (16) and the default load factor (0.75).
 */
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}
```

可以指定初始的 capacity 和 load factor 构造集合

* capacity 会被转化为对应的 2 的幂

```java
/**
 * Constructs a new <tt>HashMap</tt> with the same mappings as the
 * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
 * default load factor (0.75) and an initial capacity sufficient to
 * hold the mappings in the specified <tt>Map</tt>.
 *
 * @param   m the map whose mappings are to be placed in this map
 * @throws  NullPointerException if the specified map is null
 */
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

或者从另一个集合构造现集合

* 使用默认的 load factor `0.75f`
* 然后将 entry 加入集合

---

```java
/**
 * Basic hash bin node, used for most entries.  (See below for
 * TreeNode subclass, and in LinkedHashMap for its Entry subclass.)
 */
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

基本的桶结点的定义

---

```java
/**
 * Copies all of the mappings from the specified map to this map.
 * These mappings will replace any mappings that this map had for
 * any of the keys currently in the specified map.
 *
 * @param m mappings to be stored in this map
 * @throws NullPointerException if the specified map is null
 */
public void putAll(Map<? extends K, ? extends V> m) {
    putMapEntries(m, true);
}
```

将给定集合中的所有 entry 放入哈希表

```java
/**
 * Implements Map.putAll and Map constructor.
 *
 * @param m the map
 * @param evict false when initially constructing this map, else
 * true (relayed to method afterNodeInsertion).
 */
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        if (table == null) { // pre-size
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                        (int)ft : MAXIMUM_CAPACITY);
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        else if (s > threshold)
            resize();
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

用于实现 __拷贝构造函数__ 和 `putAll()`

* 首先判断 table 是否已分配
    * 如果还未分配，就计算分配的数组大小
    * `size / loadFactor` 并转化为 2 的幂
* 再判断加入的 size 是否会超出空间阈值
    * 如果超出阈值，就需要 `resize()`
* 最后将每一个元素放入集合

将元素放入集合使用的是接下来的 `putVal()` 函数

---

```java
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>.)
 */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

/**
 * Implements Map.put and related methods.
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

将 entry 加入哈希表

* 首先判断哈希表是否为空，为空则调 `resize()` 初始化
* 如果没有碰撞的函数 (没有桶)，则初始化新的桶结点
* 否则，桶中已发生碰撞 (hash 相同)
    * 如果 key 相同，记录原元素
    * 如果 key 不同，且已经是红黑树，那么直接调 `putTreeVal()` 插入红黑树
    * 否则就是链表
        * 循环到达链表尾部，并实例化新结点插入；如果超过树化阈值，则构建红黑树
        * 如果循环过程中发现 key 相同的元素，则记录原元素
    * 如果记录元素不为空 (说明发生 hash 碰撞且 key 也相同的元素)
        * 将旧的 value 记录下来
        * 用新的 value 替代旧的 value
        * 返回旧 value
* 添加结束后，如果超过了扩容阈值，则调 `resize()` 扩容

n 为哈希表的长度

* 哈希表的实际下标为 `[0, n - 1]`
* 所以将元素的 hash 值映射到哈希表上就是 `(n - 1) & hash`

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
 *
 * @see #put(Object, Object)
 */
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

/**
 * Returns <tt>true</tt> if this map contains a mapping for the
 * specified key.
 *
 * @param   key   The key whose presence in this map is to be tested
 * @return <tt>true</tt> if this map contains a mapping for the specified
 * key.
 */
public boolean containsKey(Object key) {
    return getNode(hash(key), key) != null;
}
```

这两个函数都要根据给定的 key 取得对应的 entry

调 `getNode()` 实现：

```java
/**
 * Implements Map.get and related methods.
 *
 * @param hash hash for key
 * @param key the key
 * @return the node, or null if none
 */
final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

* 首先检查桶是否为空，为空则返回 `null`
* 若不为空，则检查第一个元素
    * 如果第一个元素的 key 与给定的 key 相等，则直接返回
    * 如果不相等
        * 如果是红黑树结构，则调 `getTreeNode()` 从树中找结点
        * 如果是链表结构，则遍历直至找到 key 相同的结点或到结尾

---

```java
/**
 * Initializes or doubles table size.  If null, allocates in
 * accord with initial capacity target held in field threshold.
 * Otherwise, because we are using power-of-two expansion, the
 * elements from each bin must either stay at same index, or move
 * with a power of two offset in the new table.
 *
 * @return the table
 */
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                    oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                    (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { // preserve order
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

根据现在的容量和扩容阈值，计算新的容量和扩容阈值

* 一般情况下，是扩容两倍

未初始化的哈希表也会调这个函数

* 将哈希表数组初始化为默认长度

用新计算出的扩容阈值替代 `threshold`

用新计算出的容量重新分配哈希表数组

如果原哈希表不为空，就把 entry 搬到新的哈希表中

* 如果数组中的某一项不为 `null`，则将该项置 `null` 便于 GC
    * 如果只有该结点一个，则直接复制
    * 如果是红黑树，调红黑树的 `split` 到新哈希表
    * 如果是链表，这里操作很骚：
        > xxxxx01001 & 0000011111
        > 
        > 在扩容两倍后，变成：
        >
        > xxxxX01001 & 0000111111
        > 
        > 之前的容量中，以 `01001` 结尾的元素都会被映射到同一个位置
        >
        > 扩容后，以 `01001` 结尾的元素会根据 `X` 位为 0 或 1 而被映射到两个位置
        >
        > 一个位置为 `01001` 对应的原位置，即下面的 `x`，另一个位置是 `x` + 原容量
        * 遍历原链表，将映射到 `x` 和 `x + 原容量` 的元素分别串成两个链表
        * 将这两个链表分别放入新哈希表 `x` 和 `x + 原容量` 的位置

---

```java
/**
 * Removes the mapping for the specified key from this map if present.
 *
 * @param  key key whose mapping is to be removed from the map
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>.)
 */
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}
```

调 `removeNode()` 删除指定 key 的 entry

```java
/**
 * Implements Map.remove and related methods.
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to match if matchValue, else ignored
 * @param matchValue if true only remove if value is equal
 * @param movable if false do not move other nodes while removing
 * @return the node, or null if none
 */
final Node<K,V> removeNode(int hash, Object key, Object value,
                            boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                            (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        if (node != null && (!matchValue || (v = node.value) == value ||
                                (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}
```

从哈希表中删除指定 key 的元素

* 另外还可以指定匹配特定的 value
* 如果不指定则忽略

显然，首先判断哈希表数组是否存在，不然删个鸡毛

* 如果指定 hash 位置上正好存在 key 相等的 entry，则直接记录该结点
* 如果指定 hash 位置上是红黑树或是链表
    * 如果是红黑树，则调红黑树的 `getTreeNode()` 找到指定 key 的结点
    * 如果是链表，则遍历链表直到对应 key 的结点被找到
    * 记录找到的结点为 `node`，前一个结点为 `p`

如果找到的结点不为空

* 如果是红黑树结点，调红黑树的 `removeTreeNode()` 删除该结点
* 如果是单独的结点，则直接将该结点置空
* 如果是链表，那么将结点从链表中删除

---

```java
/**
 * Removes all of the mappings from this map.
 * The map will be empty after this call returns.
 */
public void clear() {
    Node<K,V>[] tab;
    modCount++;
    if ((tab = table) != null && size > 0) {
        size = 0;
        for (int i = 0; i < tab.length; ++i)
            tab[i] = null;
    }
}
```

> 将整个集合清除，这没啥问题
>
> 不过，将哈希表数组中的每个元素置 `null` 就 ok 了吗？？？
>
> 那些链表啊红黑树结点啊不用被置 `null` 吗？
>
> 莫非会自动被 GC ？ 😶

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
    Node<K,V>[] tab; V v;
    if ((tab = table) != null && size > 0) {
        for (int i = 0; i < tab.length; ++i) {
            for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                if ((v = e.value) == value ||
                    (value != null && value.equals(v)))
                    return true;
            }
        }
    }
    return false;
}
```

判断某个 value 是否出现在集合中

* 第一层循环遍历哈希表数组中的每一个位置
* 对于每一个位置，第二层循环遍历每个位置上的链表

有些位置虽然已经被构造为红黑树

* 但是在树化的过程中，始终保持着结点的 `next` 域的引用关系不变
* 所以即使被树化，还是可以通过 `next` 域遍历所有结点
* 遍历没有必要和树结构扯上关系

---

```java
/**
 * Replaces all linked nodes in bin at index for given hash unless
 * table is too small, in which case resizes instead.
 */
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

将某个 hash 值的链表组织为红黑树

* 如果哈希表数组低于树化阈值，就不整红黑树了，`resize()` 扩容一下拉倒
* 否则就将链表中的每个 `Node<K,V>` 转化为 `TreeNode<K,V>`
    * 然后调链表头 TreeNode 的 `treeify()` 建红黑树

可以看到这个函数中，甚至是 `TreeNode.treeify()` 中

* 只使用了 `next` 域，但没有做任何修改
* 因此链表虽然被组织为红黑树，但是 `next` 依旧发挥着链表的作用

---

关于树化的具体细节：

```java
// For treeifyBin
TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
    return new TreeNode<>(p.hash, p.key, p.value, next);
}
```

实际上是用当前 `Node` 结点实例化了一个新的 `TreeNode` 结点

具体的定义如下：

```java
/* ------------------------------------------------------------ */
// Tree bins

/**
 * Entry for Tree bins. Extends LinkedHashMap.Entry (which in turn
 * extends Node) so can be used as extension of either regular or
 * linked node.
 */
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }

    // ...
```

该结点继承自 `LinkedHashMap`

既是红黑树结点，也是个双向链表结点

---

以下是所有树节点的内部函数

```java
/**
 * Forms tree of the nodes linked from this node.
 */
final void treeify(Node<K,V>[] tab) {
    TreeNode<K,V> root = null;
    for (TreeNode<K,V> x = this, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        x.left = x.right = null;
        if (root == null) {
            x.parent = null;
            x.red = false;
            root = x;
        }
        else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            for (TreeNode<K,V> p = root;;) {
                int dir, ph;
                K pk = p.key;
                if ((ph = p.hash) > h)
                    dir = -1;
                else if (ph < h)
                    dir = 1;
                else if ((kc == null &&
                            (kc = comparableClassFor(k)) == null) ||
                            (dir = compareComparables(kc, k, pk)) == 0)
                    dir = tieBreakOrder(k, pk);

                TreeNode<K,V> xp = p;
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    root = balanceInsertion(root, x);
                    break;
                }
            }
        }
    }
    moveRootToFront(tab, root);
}
```

具体的红黑树操作不研究了

大体上，通过 `next` 遍历每一个元素

将所有结点构造为一颗以 root 为根的红黑树

* 但是 root 结点可能在链表的中间
* 所以要将 root 结点提到链表的最前面，放在哈希表数组一次就能引用到的地方
* 调下面的 `moveRootToFront()` 函数

```java
/**
 * Ensures that the given root is the first node of its bin.
 */
static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
    int n;
    if (root != null && tab != null && (n = tab.length) > 0) {
        int index = (n - 1) & root.hash;
        TreeNode<K,V> first = (TreeNode<K,V>)tab[index];
        if (root != first) {
            Node<K,V> rn;
            tab[index] = root;
            TreeNode<K,V> rp = root.prev;
            if ((rn = root.next) != null)
                ((TreeNode<K,V>)rn).prev = rp;
            if (rp != null)
                rp.next = rn;
            if (first != null)
                first.prev = root;
            root.next = first;
            root.prev = null;
        }
        assert checkInvariants(root);
    }
}
```

根据 hash 值，在哈希表数组中找到对应下标

将原第一个元素 `first` 记录

将 `root` 放到哈希表数组对应下标处

并将 `root` 从链表中间取出，将之前的链表和之后的链表串起来

最后将 `first` 接在 `root` 的后面

---

```java
/**
 * Returns a list of non-TreeNodes replacing those linked from
 * this node.
 */
final Node<K,V> untreeify(HashMap<K,V> map) {
    Node<K,V> hd = null, tl = null;
    for (Node<K,V> q = this; q != null; q = q.next) {
        Node<K,V> p = map.replacementNode(q, null);
        if (tl == null)
            hd = p;
        else
            tl.next = p;
        tl = p;
    }
    return hd;
}
```

返回非 TreeNode 版本的链表结点

---

```java
/**
 * Finds the node starting at root p with the given hash and key.
 * The kc argument caches comparableClassFor(key) upon first use
 * comparing keys.
 */
final TreeNode<K,V> find(int h, Object k, Class<?> kc) {
    TreeNode<K,V> p = this;
    do {
        int ph, dir; K pk;
        TreeNode<K,V> pl = p.left, pr = p.right, q;
        if ((ph = p.hash) > h)
            p = pl;
        else if (ph < h)
            p = pr;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if (pl == null)
            p = pr;
        else if (pr == null)
            p = pl;
        else if ((kc != null ||
                    (kc = comparableClassFor(k)) != null) &&
                    (dir = compareComparables(kc, k, pk)) != 0)
            p = (dir < 0) ? pl : pr;
        else if ((q = pr.find(h, k, kc)) != null)
            return q;
        else
            p = pl;
    } while (p != null);
    return null;
}

/**
 * Calls find for root node.
 */
final TreeNode<K,V> getTreeNode(int h, Object k) {
    return ((parent != null) ? root() : this).find(h, k, null);
}
```

给定 hash 和 key，从 root 寻找结点

* 比较 hashcode，进入左子树或右子树

---

```java
/**
 * Tree version of putVal.
 */
final TreeNode<K,V> putTreeVal(HashMap<K,V> map, Node<K,V>[] tab,
                                int h, K k, V v) {
    Class<?> kc = null;
    boolean searched = false;
    TreeNode<K,V> root = (parent != null) ? root() : this;
    for (TreeNode<K,V> p = root;;) {
        int dir, ph; K pk;
        if ((ph = p.hash) > h)
            dir = -1;
        else if (ph < h)
            dir = 1;
        else if ((pk = p.key) == k || (k != null && k.equals(pk)))
            return p;
        else if ((kc == null &&
                    (kc = comparableClassFor(k)) == null) ||
                    (dir = compareComparables(kc, k, pk)) == 0) {
            if (!searched) {
                TreeNode<K,V> q, ch;
                searched = true;
                if (((ch = p.left) != null &&
                        (q = ch.find(h, k, kc)) != null) ||
                    ((ch = p.right) != null &&
                        (q = ch.find(h, k, kc)) != null))
                    return q;
            }
            dir = tieBreakOrder(k, pk);
        }

        TreeNode<K,V> xp = p;
        if ((p = (dir <= 0) ? p.left : p.right) == null) {
            Node<K,V> xpn = xp.next;
            TreeNode<K,V> x = map.newTreeNode(h, k, v, xpn);
            if (dir <= 0)
                xp.left = x;
            else
                xp.right = x;
            xp.next = x;
            x.parent = x.prev = xp;
            if (xpn != null)
                ((TreeNode<K,V>)xpn).prev = x;
            moveRootToFront(tab, balanceInsertion(root, x));
            return null;
        }
    }
}
```

将 entry 放入树中

* 首先寻找插入位置，如果结点已经存在，则直接返回结点位置
    * 由调用该函数的 `putVal()` 来记录 old value 并返回
* 如果没有找到结点，则新建结点
    * 加入树中
    * 加入链表中
    * 调 `moveRootToFront()` 将 root 结点调整到哈希表数组中

---

```java
/**
 * Removes the given node, that must be present before this call.
 * This is messier than typical red-black deletion code because we
 * cannot swap the contents of an interior node with a leaf
 * successor that is pinned by "next" pointers that are accessible
 * independently during traversal. So instead we swap the tree
 * linkages. If the current tree appears to have too few nodes,
 * the bin is converted back to a plain bin. (The test triggers
 * somewhere between 2 and 6 nodes, depending on tree structure).
 */
final void removeTreeNode(HashMap<K,V> map, Node<K,V>[] tab,
                            boolean movable) {
    int n;
    if (tab == null || (n = tab.length) == 0)
        return;
    int index = (n - 1) & hash;
    TreeNode<K,V> first = (TreeNode<K,V>)tab[index], root = first, rl;
    TreeNode<K,V> succ = (TreeNode<K,V>)next, pred = prev;
    if (pred == null)
        tab[index] = first = succ;
    else
        pred.next = succ;
    if (succ != null)
        succ.prev = pred;
    if (first == null)
        return;
    if (root.parent != null)
        root = root.root();
    if (root == null
        || (movable
            && (root.right == null
                || (rl = root.left) == null
                || rl.left == null))) {
        tab[index] = first.untreeify(map);  // too small
        return;
    }
    TreeNode<K,V> p = this, pl = left, pr = right, replacement;
    if (pl != null && pr != null) {
        TreeNode<K,V> s = pr, sl;
        while ((sl = s.left) != null) // find successor
            s = sl;
        boolean c = s.red; s.red = p.red; p.red = c; // swap colors
        TreeNode<K,V> sr = s.right;
        TreeNode<K,V> pp = p.parent;
        if (s == pr) { // p was s's direct parent
            p.parent = s;
            s.right = p;
        }
        else {
            TreeNode<K,V> sp = s.parent;
            if ((p.parent = sp) != null) {
                if (s == sp.left)
                    sp.left = p;
                else
                    sp.right = p;
            }
            if ((s.right = pr) != null)
                pr.parent = s;
        }
        p.left = null;
        if ((p.right = sr) != null)
            sr.parent = p;
        if ((s.left = pl) != null)
            pl.parent = s;
        if ((s.parent = pp) == null)
            root = s;
        else if (p == pp.left)
            pp.left = s;
        else
            pp.right = s;
        if (sr != null)
            replacement = sr;
        else
            replacement = p;
    }
    else if (pl != null)
        replacement = pl;
    else if (pr != null)
        replacement = pr;
    else
        replacement = p;
    if (replacement != p) {
        TreeNode<K,V> pp = replacement.parent = p.parent;
        if (pp == null)
            root = replacement;
        else if (p == pp.left)
            pp.left = replacement;
        else
            pp.right = replacement;
        p.left = p.right = p.parent = null;
    }

    TreeNode<K,V> r = p.red ? root : balanceDeletion(root, replacement);

    if (replacement == p) {  // detach
        TreeNode<K,V> pp = p.parent;
        p.parent = null;
        if (pp != null) {
            if (p == pp.left)
                pp.left = null;
            else if (p == pp.right)
                pp.right = null;
        }
    }
    if (movable)
        moveRootToFront(tab, r);
}
```

将结点从 `next` 和 `pred` 组织的链表中移出来

如果移除后长度过小，则将红黑树复原为链表并返回

否则就进行红黑树的删除结点操作

并将新的 root 移动到哈希表数组中

---

```java
/**
 * Splits nodes in a tree bin into lower and upper tree bins,
 * or untreeifies if now too small. Called only from resize;
 * see above discussion about split bits and indices.
 *
 * @param map the map
 * @param tab the table for recording bin heads
 * @param index the index of the table being split
 * @param bit the bit of hash to split on
 */
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    TreeNode<K,V> b = this;
    // Relink into lo and hi lists, preserving order
    TreeNode<K,V> loHead = null, loTail = null;
    TreeNode<K,V> hiHead = null, hiTail = null;
    int lc = 0, hc = 0;
    for (TreeNode<K,V> e = b, next; e != null; e = next) {
        next = (TreeNode<K,V>)e.next;
        e.next = null;
        if ((e.hash & bit) == 0) {
            if ((e.prev = loTail) == null)
                loHead = e;
            else
                loTail.next = e;
            loTail = e;
            ++lc;
        }
        else {
            if ((e.prev = hiTail) == null)
                hiHead = e;
            else
                hiTail.next = e;
            hiTail = e;
            ++hc;
        }
    }

    if (loHead != null) {
        if (lc <= UNTREEIFY_THRESHOLD)
            tab[index] = loHead.untreeify(map);
        else {
            tab[index] = loHead;
            if (hiHead != null) // (else is already treeified)
                loHead.treeify(tab);
        }
    }
    if (hiHead != null) {
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        else {
            tab[index + bit] = hiHead;
            if (loHead != null)
                hiHead.treeify(tab);
        }
    }
}
```

这个函数只会被 `resize()` 调用

将当前的红黑树拆分到新的哈希表数组中

如果切分后树的大小低于阈值，则复原为链表

同样，扩容后，原先映射到该桶 (红黑树) 的所有结点将可能映射到两个桶中

* 一个是原位置的桶
* 一个是原位置 + 原容量的桶

利用 `next` 域对红黑树进行遍历

根据计算出的 hash 值，将映射到两个桶的结点串成两个链表 `lo` 和 `hi`

* 若链表不为空且低于阈值，则转化为链表并放入对应桶中
* 若链表不为空且高于阈值，则将链表放入对应桶中，并对链表头调用 `treeify()` 进行树化

---

关于红黑树的操作函数就略了...

```java
static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root, TreeNode<K,V> p) {}

static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root, TreeNode<K,V> p) {}

static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root, TreeNode<K,V> x) {}

static <K,V> TreeNode<K,V> balanceDeletion(TreeNode<K,V> root, TreeNode<K,V> x) {}

static <K,V> boolean checkInvariants(TreeNode<K,V> t) {}
```

---

注释中的相关实现细节：

```java
/*
 * Implementation notes.
 *
 * This map usually acts as a binned (bucketed) hash table, but
 * when bins get too large, they are transformed into bins of
 * TreeNodes, each structured similarly to those in
 * java.util.TreeMap. Most methods try to use normal bins, but
 * relay to TreeNode methods when applicable (simply by checking
 * instanceof a node).  Bins of TreeNodes may be traversed and
 * used like any others, but additionally support faster lookup
 * when overpopulated. However, since the vast majority of bins in
 * normal use are not overpopulated, checking for existence of
 * tree bins may be delayed in the course of table methods.
 *
 * Tree bins (i.e., bins whose elements are all TreeNodes) are
 * ordered primarily by hashCode, but in the case of ties, if two
 * elements are of the same "class C implements Comparable<C>",
 * type then their compareTo method is used for ordering. (We
 * conservatively check generic types via reflection to validate
 * this -- see method comparableClassFor).  The added complexity
 * of tree bins is worthwhile in providing worst-case O(log n)
 * operations when keys either have distinct hashes or are
 * orderable, Thus, performance degrades gracefully under
 * accidental or malicious usages in which hashCode() methods
 * return values that are poorly distributed, as well as those in
 * which many keys share a hashCode, so long as they are also
 * Comparable. (If neither of these apply, we may waste about a
 * factor of two in time and space compared to taking no
 * precautions. But the only known cases stem from poor user
 * programming practices that are already so slow that this makes
 * little difference.)
 *
 * Because TreeNodes are about twice the size of regular nodes, we
 * use them only when bins contain enough nodes to warrant use
 * (see TREEIFY_THRESHOLD). And when they become too small (due to
 * removal or resizing) they are converted back to plain bins.  In
 * usages with well-distributed user hashCodes, tree bins are
 * rarely used.  Ideally, under random hashCodes, the frequency of
 * nodes in bins follows a Poisson distribution
 * (http://en.wikipedia.org/wiki/Poisson_distribution) with a
 * parameter of about 0.5 on average for the default resizing
 * threshold of 0.75, although with a large variance because of
 * resizing granularity. Ignoring variance, the expected
 * occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
 * factorial(k)). The first values are:
 *
 * 0:    0.60653066
 * 1:    0.30326533
 * 2:    0.07581633
 * 3:    0.01263606
 * 4:    0.00157952
 * 5:    0.00015795
 * 6:    0.00001316
 * 7:    0.00000094
 * 8:    0.00000006
 * more: less than 1 in ten million
 *
 * The root of a tree bin is normally its first node.  However,
 * sometimes (currently only upon Iterator.remove), the root might
 * be elsewhere, but can be recovered following parent links
 * (method TreeNode.root()).
 *
 * All applicable internal methods accept a hash code as an
 * argument (as normally supplied from a public method), allowing
 * them to call each other without recomputing user hashCodes.
 * Most internal methods also accept a "tab" argument, that is
 * normally the current table, but may be a new or old one when
 * resizing or converting.
 *
 * When bin lists are treeified, split, or untreeified, we keep
 * them in the same relative access/traversal order (i.e., field
 * Node.next) to better preserve locality, and to slightly
 * simplify handling of splits and traversals that invoke
 * iterator.remove. When using comparators on insertion, to keep a
 * total ordering (or as close as is required here) across
 * rebalancings, we compare classes and identityHashCodes as
 * tie-breakers.
 *
 * The use and transitions among plain vs tree modes is
 * complicated by the existence of subclass LinkedHashMap. See
 * below for hook methods defined to be invoked upon insertion,
 * removal and access that allow LinkedHashMap internals to
 * otherwise remain independent of these mechanics. (This also
 * requires that a map instance be passed to some utility methods
 * that may create new nodes.)
 *
 * The concurrent-programming-like SSA-based coding style helps
 * avoid aliasing errors amid all of the twisty pointer operations.
 */
```

引入红黑树主要是为了 hashcode 分布特别不均匀时防止性能下降 (链表过长)

* 比如多个 key 共用同一个 hashcode 的情况

在一个分布较为均匀的 hashcode 下，基本上用不到树结点

理想情况下，使用随机 hashcode，每个桶中的结点数量满足泊松分布，概率为：

* 0: 0.60653066
* 1: 0.30326533
* 2: 0.07581633
* 3: 0.01263606
* 4: 0.00157952
* 5: 0.00015795
* 6: 0.00001316
* 7: 0.00000094
* 8: 0.00000006
* more: less than 1 in ten million

可以看到，超过树化阈值 (8) 的概率已经相当低了

---

## Summary

把握 HashMap 具体实现的核心数据结构：

* 哈希表数组 + 冲突链表 + 链表过长后转为红黑树

还要把握所有操作遵循的共同思想

需要遍历所有元素的函数：

* 遍历哈希表数组的每一个位置
* 对于某一个位置，如果是链表 (或红黑树)，则直接用 `next` 域遍历到底

插入元素的函数：

* 计算 hash，找到对应的位置
* 如果没有冲突，直接放入集合
* 如果发生冲突，且数据结构被组织为链表，则插入链表
* 如果发生冲突，且数据结构被组织为红黑树，则插入红黑树并重整 root 到链表头
* 如果插入完成后，元素个数超过了容量阈值，则进行 resize
    * 容量扩容了两倍
    * 对于同一个碰撞位置上的每一个元素 (链表或红黑树)，一定有了一个新的可能的坑位可以去 😅
    * 重新计算 hash，将映射到老位置和新位置的元素分别串成两个链表，并放到对应位置
    * 如果两个新链表超过了树化阈值，则进行树化

删除元素的函数：

* 计算 hash 值
* 如果元素不冲突，则直接在哈希表数组中删除
* 如果是链表，那么在链表中寻找对应元素删除
* 如果是红黑树，那么用红黑树的函数删掉元素
    * 如果删除后树中元素过少，则将红黑树重新组织为链表

总体思想都是类似的，即：

* 计算 hash
* 分别用不同数据结构对应的操作实现功能
    * `O(1)` 的数组随机访问
    * `O(n)` 的链表
    * `O(log(n))` 的红黑树
* 维护这几种数据结构之间的转化
* 维护总体的容量和阈值

HashMap 算是所有的容器中最复杂的一个了

专家不愧是专家，实现得很漂亮 👨‍💻

---

