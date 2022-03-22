# Class - java.util.IdentityHashMap

Created by : Mr Dk.

2019 / 11 / 25 16:52

Nanjing, Jiangsu, China

---

## Definition

```java
public class IdentityHashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, java.io.Serializable, Cloneable
{

}
```

这个类也用 hash table 实现了 Map 接口，但在比较 key 时，使用了 **引用相等**，而不是 **对象相等**：

- **引用相等** 比较指向的内存地址是否相等
- **对象相等** 比较内存中存放的内容是否相等，使用 `equals()` 进行判断

区别在于：

- 在 `IdentityHashMap` 中，`k1` 和 `k2` 相等当且仅当 `k1 == k2`
- 在 `HashMap` 中，`k1` 和 `k2` 相等当且仅当 `(k1==null ? k2==null : k1.equals(k2))`

这个类不是一个通用实现，因为其颠覆了 Map 接口中的定义，所以只有在特殊场合下才会被使用。允许空的 key 或 value，不保证迭代顺序。

特殊参数：expected maximum size - 期望集合中容纳 entry 的数量。在内部被用于分配桶的数量，但两者没有确定的关系。如果集合中 entry 的实际数量超过了这个值较多，则会进行 rehashing。而且在 hash table 的具体实现中，不像 `HashMap` 中用 **链** 来解决 hash 冲突——使用了 **线性探测 (linear-probe)** 方法，在性能上可能会比 `HashMap` 更好。

```java
/**
 * This class implements the <tt>Map</tt> interface with a hash table, using
 * reference-equality in place of object-equality when comparing keys (and
 * values).  In other words, in an <tt>IdentityHashMap</tt>, two keys
 * <tt>k1</tt> and <tt>k2</tt> are considered equal if and only if
 * <tt>(k1==k2)</tt>.  (In normal <tt>Map</tt> implementations (like
 * <tt>HashMap</tt>) two keys <tt>k1</tt> and <tt>k2</tt> are considered equal
 * if and only if <tt>(k1==null ? k2==null : k1.equals(k2))</tt>.)
 *
 * <p><b>This class is <i>not</i> a general-purpose <tt>Map</tt>
 * implementation!  While this class implements the <tt>Map</tt> interface, it
 * intentionally violates <tt>Map's</tt> general contract, which mandates the
 * use of the <tt>equals</tt> method when comparing objects.  This class is
 * designed for use only in the rare cases wherein reference-equality
 * semantics are required.</b>
 *
 * <p>A typical use of this class is <i>topology-preserving object graph
 * transformations</i>, such as serialization or deep-copying.  To perform such
 * a transformation, a program must maintain a "node table" that keeps track
 * of all the object references that have already been processed.  The node
 * table must not equate distinct objects even if they happen to be equal.
 * Another typical use of this class is to maintain <i>proxy objects</i>.  For
 * example, a debugging facility might wish to maintain a proxy object for
 * each object in the program being debugged.
 *
 * <p>This class provides all of the optional map operations, and permits
 * <tt>null</tt> values and the <tt>null</tt> key.  This class makes no
 * guarantees as to the order of the map; in particular, it does not guarantee
 * that the order will remain constant over time.
 *
 * <p>This class provides constant-time performance for the basic
 * operations (<tt>get</tt> and <tt>put</tt>), assuming the system
 * identity hash function ({@link System#identityHashCode(Object)})
 * disperses elements properly among the buckets.
 *
 * <p>This class has one tuning parameter (which affects performance but not
 * semantics): <i>expected maximum size</i>.  This parameter is the maximum
 * number of key-value mappings that the map is expected to hold.  Internally,
 * this parameter is used to determine the number of buckets initially
 * comprising the hash table.  The precise relationship between the expected
 * maximum size and the number of buckets is unspecified.
 *
 * <p>If the size of the map (the number of key-value mappings) sufficiently
 * exceeds the expected maximum size, the number of buckets is increased.
 * Increasing the number of buckets ("rehashing") may be fairly expensive, so
 * it pays to create identity hash maps with a sufficiently large expected
 * maximum size.  On the other hand, iteration over collection views requires
 * time proportional to the number of buckets in the hash table, so it
 * pays not to set the expected maximum size too high if you are especially
 * concerned with iteration performance or memory usage.
 *
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access an identity hash map concurrently, and at
 * least one of the threads modifies the map structurally, it <i>must</i>
 * be synchronized externally.  (A structural modification is any operation
 * that adds or deletes one or more mappings; merely changing the value
 * associated with a key that an instance already contains is not a
 * structural modification.)  This is typically accomplished by
 * synchronizing on some object that naturally encapsulates the map.
 *
 * If no such object exists, the map should be "wrapped" using the
 * {@link Collections#synchronizedMap Collections.synchronizedMap}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the map:<pre>
 *   Map m = Collections.synchronizedMap(new IdentityHashMap(...));</pre>
 *
 * <p>The iterators returned by the <tt>iterator</tt> method of the
 * collections returned by all of this class's "collection view
 * methods" are <i>fail-fast</i>: if the map is structurally modified
 * at any time after the iterator is created, in any way except
 * through the iterator's own <tt>remove</tt> method, the iterator
 * will throw a {@link ConcurrentModificationException}.  Thus, in the
 * face of concurrent modification, the iterator fails quickly and
 * cleanly, rather than risking arbitrary, non-deterministic behavior
 * at an undetermined time in the future.
 *
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw <tt>ConcurrentModificationException</tt> on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness: <i>fail-fast iterators should be used only
 * to detect bugs.</i>
 *
 * <p>Implementation note: This is a simple <i>linear-probe</i> hash table,
 * as described for example in texts by Sedgewick and Knuth.  The array
 * alternates holding keys and values.  (This has better locality for large
 * tables than does using separate arrays.)  For many JRE implementations
 * and operation mixes, this class will yield better performance than
 * {@link HashMap} (which uses <i>chaining</i> rather than linear-probing).
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @see     System#identityHashCode(Object)
 * @see     Object#hashCode()
 * @see     Collection
 * @see     Map
 * @see     HashMap
 * @see     TreeMap
 * @author  Doug Lea and Josh Bloch
 * @since   1.4
 */
```

## Members

默认的容量（必须是 2 的整数幂）：

```java
/**
 * The initial capacity used by the no-args constructor.
 * MUST be a power of two.  The value 32 corresponds to the
 * (specified) expected maximum size of 21, given a load factor
 * of 2/3.
 */
private static final int DEFAULT_CAPACITY = 32;
```

最小容量 / 最大容量：

```java
/**
 * The minimum capacity, used if a lower value is implicitly specified
 * by either of the constructors with arguments.  The value 4 corresponds
 * to an expected maximum size of 2, given a load factor of 2/3.
 * MUST be a power of two.
 */
private static final int MINIMUM_CAPACITY = 4;

/**
 * The maximum capacity, used if a higher value is implicitly specified
 * by either of the constructors with arguments.
 * MUST be a power of two <= 1<<29.
 *
 * In fact, the map can hold no more than MAXIMUM_CAPACITY-1 items
 * because it has to have at least one slot with the key == null
 * in order to avoid infinite loops in get(), put(), remove()
 */
private static final int MAXIMUM_CAPACITY = 1 << 29;
```

哈希表：

```java
/**
 * The table, resized as necessary. Length MUST always be a power of two.
 */
transient Object[] table; // non-private to simplify nested class access
```

用于表示 `null` key：

```java
/**
 * The number of key-value mappings contained in this identity hash map.
 *
 * @serial
 */
int size;

/**
 * The number of modifications, to support fast-fail iterators
 */
transient int modCount;
```

```java
/**
 * Value representing null keys inside tables.
 */
static final Object NULL_KEY = new Object();

/**
 * Use NULL_KEY for key if it is null.
 */
private static Object maskNull(Object key) {
    return (key == null ? NULL_KEY : key);
}

/**
 * Returns internal representation of null key back to caller as null.
 */
static final Object unmaskNull(Object key) {
    return (key == NULL_KEY ? null : key);
}
```

## Constructor

默认或指定 `expectedMaxSize` 来初始化哈希表：

```java
/**
 * Constructs a new, empty identity hash map with a default expected
 * maximum size (21).
 */
public IdentityHashMap() {
    init(DEFAULT_CAPACITY);
}

/**
 * Constructs a new, empty map with the specified expected maximum size.
 * Putting more than the expected number of key-value mappings into
 * the map may cause the internal data structure to grow, which may be
 * somewhat time-consuming.
 *
 * @param expectedMaxSize the expected maximum size of the map
 * @throws IllegalArgumentException if <tt>expectedMaxSize</tt> is negative
 */
public IdentityHashMap(int expectedMaxSize) {
    if (expectedMaxSize < 0)
        throw new IllegalArgumentException("expectedMaxSize is negative: "
                                            + expectedMaxSize);
    init(capacity(expectedMaxSize));
}
```

该函数能够保证返回合法范围内的 2 整数次幂的容量：

```java
/**
 * Returns the appropriate capacity for the given expected maximum size.
 * Returns the smallest power of two between MINIMUM_CAPACITY and
 * MAXIMUM_CAPACITY, inclusive, that is greater than (3 *
 * expectedMaxSize)/2, if such a number exists.  Otherwise returns
 * MAXIMUM_CAPACITY.
 */
private static int capacity(int expectedMaxSize) {
    // assert expectedMaxSize >= 0;
    return
        (expectedMaxSize > MAXIMUM_CAPACITY / 3) ? MAXIMUM_CAPACITY :
        (expectedMaxSize <= 2 * MINIMUM_CAPACITY / 3) ? MINIMUM_CAPACITY :
        Integer.highestOneBit(expectedMaxSize + (expectedMaxSize << 1));
}
```

这个 `2` 是啥意思呢？没懂这个实现方式。一个 entry 占两个坑

- 一个放 key
- 一个放 value

所以分配空间一分就分配两倍。

```java
/**
 * Initializes object to be an empty map with the specified initial
 * capacity, which is assumed to be a power of two between
 * MINIMUM_CAPACITY and MAXIMUM_CAPACITY inclusive.
 */
private void init(int initCapacity) {
    // assert (initCapacity & -initCapacity) == initCapacity; // power of 2
    // assert initCapacity >= MINIMUM_CAPACITY;
    // assert initCapacity <= MAXIMUM_CAPACITY;

    table = new Object[2 * initCapacity];
}
```

拷贝构造函数，以 1.1 倍的扩充容量分配空间。

```java
/**
 * Constructs a new identity hash map containing the keys-value mappings
 * in the specified map.
 *
 * @param m the map whose mappings are to be placed into this map
 * @throws NullPointerException if the specified map is null
 */
public IdentityHashMap(Map<? extends K, ? extends V> m) {
    // Allow for a bit of growth
    this((int) ((1 + m.size()) * 1.1));
    putAll(m);
}
```

## Get

根据指定的 key (`tab[i]`) 取对应的 value (`tab[i + 1]`)

- 首先处理 key 为 `null` 的情况
- 然后计算 key 对应的 hashcode，在 hash table 的指定位置进行查找
- 如果发生碰撞，则使用 **线性探测** 寻找下一个位置

```java
/**
 * Returns the value to which the specified key is mapped,
 * or {@code null} if this map contains no mapping for the key.
 *
 * <p>More formally, if this map contains a mapping from a key
 * {@code k} to a value {@code v} such that {@code (key == k)},
 * then this method returns {@code v}; otherwise it returns
 * {@code null}.  (There can be at most one such mapping.)
 *
 * <p>A return value of {@code null} does not <i>necessarily</i>
 * indicate that the map contains no mapping for the key; it's also
 * possible that the map explicitly maps the key to {@code null}.
 * The {@link #containsKey containsKey} operation may be used to
 * distinguish these two cases.
 *
 * @see #put(Object, Object)
 */
@SuppressWarnings("unchecked")
public V get(Object key) {
    Object k = maskNull(key);
    Object[] tab = table;
    int len = tab.length;
    int i = hash(k, len);
    while (true) {
        Object item = tab[i];
        if (item == k)
            return (V) tab[i + 1];
        if (item == null)
            return null;
        i = nextKeyIndex(i, len);
    }
}
```

用对象的 hash code 与 hash table 的长度进行 `&` 运算，返回对象在数组中的 index：

```java
/**
 * Returns index for Object x.
 */
private static int hash(Object x, int length) {
    int h = System.identityHashCode(x);
    // Multiply by -127, and left-shift to use least bit as part of hash
    return ((h << 1) - (h << 8)) & (length - 1);
}
```

线性探测函数

- 每次探测当前 key 的下一个 key
- 当超出数组长度后，折返到第一个 key

```java
/**
 * Circularly traverses table of size len.
 */
private static int nextKeyIndex(int i, int len) {
    return (i + 2 < len ? i + 2 : 0);
}
```

## Contain

与上一个函数类似，但只返回 key 是否存在：

```java
/**
 * Tests whether the specified object reference is a key in this identity
 * hash map.
 *
 * @param   key   possible key
 * @return  <code>true</code> if the specified object reference is a key
 *          in this map
 * @see     #containsValue(Object)
 */
public boolean containsKey(Object key) {
    Object k = maskNull(key);
    Object[] tab = table;
    int len = tab.length;
    int i = hash(k, len);
    while (true) {
        Object item = tab[i];
        if (item == k)
            return true;
        if (item == null)
            return false;
        i = nextKeyIndex(i, len);
    }
}
```

检查指定的 value 是否在集合中。显然需要开始遍历整个哈希表数组了，由于是遍历 value，因此循环是从 `1` 下标开始，且循环增量为 `2`：

```java
/**
 * Tests whether the specified object reference is a value in this identity
 * hash map.
 *
 * @param value value whose presence in this map is to be tested
 * @return <tt>true</tt> if this map maps one or more keys to the
 *         specified object reference
 * @see     #containsKey(Object)
 */
public boolean containsValue(Object value) {
    Object[] tab = table;
    for (int i = 1; i < tab.length; i += 2)
        if (tab[i] == value && tab[i - 1] != null)
            return true;

    return false;
}
```

查看集合中是否存在某个 key 到 value 的映射。与上面的函数类似，首先找到特定的 key，返回该 key 对应的 value 与指定的 value 是否相同。

```java
/**
 * Tests if the specified key-value mapping is in the map.
 *
 * @param   key   possible key
 * @param   value possible value
 * @return  <code>true</code> if and only if the specified key-value
 *          mapping is in the map
 */
private boolean containsMapping(Object key, Object value) {
    Object k = maskNull(key);
    Object[] tab = table;
    int len = tab.length;
    int i = hash(k, len);
    while (true) {
        Object item = tab[i];
        if (item == k)
            return tab[i + 1] == value;
        if (item == null)
            return false;
        i = nextKeyIndex(i, len);
    }
}
```

## Put

第一次在 Java 中看见了标号 😯

该函数试图将 key 和 value 的映射加入到哈希表中如果映射已经存在，就用新的 value 替代老的 value

- 首先根据指定的 key 和哈希表的长度，计算映射到的 index
- 如果 index 有冲突，则使用线性探测，向后寻找空位，直到找到 old value 并返回

如果没有找到映射，着手将映射加入哈希表：

- 首先，检查一下加入这个 entry 之后，entry 总个数是否已经超过了哈希表数组的 1/3
- 如果是，那么就要进行 `resize()`，重新分配空间 (哈希表数组扩容两倍)
- 重新分配空间后，要跳转到标号处重新寻找 hash 位置 (因为数组的长度变了)

最终找到被插入的位置 `i`：

- key 存放在位置 `tab[i]`
- value 存放在位置 `tab[i + 1]`

```java
/**
 * Associates the specified value with the specified key in this identity
 * hash map.  If the map previously contained a mapping for the key, the
 * old value is replaced.
 *
 * @param key the key with which the specified value is to be associated
 * @param value the value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>.)
 * @see     Object#equals(Object)
 * @see     #get(Object)
 * @see     #containsKey(Object)
 */
public V put(K key, V value) {
    final Object k = maskNull(key);

    retryAfterResize: for (;;) {
        final Object[] tab = table;
        final int len = tab.length;
        int i = hash(k, len);

        for (Object item; (item = tab[i]) != null;
                i = nextKeyIndex(i, len)) {
            if (item == k) {
                @SuppressWarnings("unchecked")
                    V oldValue = (V) tab[i + 1];
                tab[i + 1] = value;
                return oldValue;
            }
        }

        final int s = size + 1;
        // Use optimized form of 3 * s.
        // Next capacity is len, 2 * current capacity.
        if (s + (s << 1) > len && resize(len))
            continue retryAfterResize;

        modCount++;
        tab[i] = k;
        tab[i + 1] = value;
        size = s;
        return null;
    }
}
```

## Resize

将哈希表的空间开辟为原来的两倍，然后将旧的哈希表中的每一个 key 重新计算 hash。并将对应的 key 和 value 加入到新的哈希表中。

```java
/**
 * Resizes the table if necessary to hold given capacity.
 *
 * @param newCapacity the new capacity, must be a power of two.
 * @return whether a resize did in fact take place
 */
private boolean resize(int newCapacity) {
    // assert (newCapacity & -newCapacity) == newCapacity; // power of 2
    int newLength = newCapacity * 2;

    Object[] oldTable = table;
    int oldLength = oldTable.length;
    if (oldLength == 2 * MAXIMUM_CAPACITY) { // can't expand any further
        if (size == MAXIMUM_CAPACITY - 1)
            throw new IllegalStateException("Capacity exhausted.");
        return false;
    }
    if (oldLength >= newLength)
        return false;

    Object[] newTable = new Object[newLength];

    for (int j = 0; j < oldLength; j += 2) {
        Object key = oldTable[j];
        if (key != null) {
            Object value = oldTable[j+1];
            oldTable[j] = null;
            oldTable[j+1] = null;
            int i = hash(key, newLength);
            while (newTable[i] != null)
                i = nextKeyIndex(i, newLength);
            newTable[i] = key;
            newTable[i + 1] = value;
        }
    }
    table = newTable;
    return true;
}
```

## Put All

首先检查空间是否足够，不够的话调 `resize()` 进行扩容，然后迭代每一个 entry 并加入集合中。

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
    int n = m.size();
    if (n == 0)
        return;
    if (n > size)
        resize(capacity(n)); // conservatively pre-expand

    for (Entry<? extends K, ? extends V> e : m.entrySet())
        put(e.getKey(), e.getValue());
}
```

## Remove

从集合中移除 entry

- 首先根据指定的 key 计算 hash 值
- 获得在哈希表中的 index
- 从该 index 开始进行线性探测
- 直到找到 key 和指定的 key 相同的 entry
- 将 `tab[i]` 位置的 key 和 `tab[i + 1]` 位置的 value 移除

```java
/**
 * Removes the mapping for this key from this map if present.
 *
 * @param key key whose mapping is to be removed from the map
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>.)
 */
public V remove(Object key) {
    Object k = maskNull(key);
    Object[] tab = table;
    int len = tab.length;
    int i = hash(k, len);

    while (true) {
        Object item = tab[i];
        if (item == k) {
            modCount++;
            size--;
            @SuppressWarnings("unchecked")
                V oldValue = (V) tab[i + 1];
            tab[i + 1] = null;
            tab[i] = null;
            closeDeletion(i);
            return oldValue;
        }
        if (item == null)
            return null;
        i = nextKeyIndex(i, len);
    }
}
```

删除指定 key 和 value 的 entry

- 先根据 key 计算出 hash 值，找到哈希表中的 index
- 从 index 开始线性探测，直到找到指定 key 的 entry
- 比较该 entry 的 value 是否与指定的 value 相等
  - 如果相等，才返回成功
  - 否则返回失败

在上述两个操作中，移除 entry 后，就有了空位置。向后线性探测一下，看看需不需要把冲突元素搬运到空的位置上。

```java
/**
 * Removes the specified key-value mapping from the map if it is present.
 *
 * @param   key   possible key
 * @param   value possible value
 * @return  <code>true</code> if and only if the specified key-value
 *          mapping was in the map
 */
private boolean removeMapping(Object key, Object value) {
    Object k = maskNull(key);
    Object[] tab = table;
    int len = tab.length;
    int i = hash(k, len);

    while (true) {
        Object item = tab[i];
        if (item == k) {
            if (tab[i + 1] != value)
                return false;
            modCount++;
            size--;
            tab[i] = null;
            tab[i + 1] = null;
            closeDeletion(i);
            return true;
        }
        if (item == null)
            return false;
        i = nextKeyIndex(i, len);
    }
}
```

```java
/**
 * Rehash all possibly-colliding entries following a
 * deletion. This preserves the linear-probe
 * collision properties required by get, put, etc.
 *
 * @param d the index of a newly empty deleted slot
 */
private void closeDeletion(int d) {
    // Adapted from Knuth Section 6.4 Algorithm R
    Object[] tab = table;
    int len = tab.length;

    // Look for items to swap into newly vacated slot
    // starting at index immediately following deletion,
    // and continuing until a null slot is seen, indicating
    // the end of a run of possibly-colliding keys.
    Object item;
    for (int i = nextKeyIndex(d, len); (item = tab[i]) != null;
            i = nextKeyIndex(i, len) ) {
        // The following test triggers if the item at slot i (which
        // hashes to be at slot r) should take the spot vacated by d.
        // If so, we swap it in, and then continue with d now at the
        // newly vacated i.  This process will terminate when we hit
        // the null slot at the end of this run.
        // The test is messy because we are using a circular table.
        int r = hash(item, len);
        if ((i < r && (r <= d || d <= i)) || (r <= d && d <= i)) {
            tab[d] = item;
            tab[d + 1] = tab[i + 1];
            tab[i] = null;
            tab[i + 1] = null;
            d = i;
        }
    }
}
```

## Clear

将所有 entry 置空。

```java
/**
 * Removes all of the mappings from this map.
 * The map will be empty after this call returns.
 */
public void clear() {
    modCount++;
    Object[] tab = table;
    for (int i = 0; i < tab.length; i++)
        tab[i] = null;
    size = 0;
}
```

## Equals

要验证每一个 entry 都相同。

```java
/**
 * Compares the specified object with this map for equality.  Returns
 * <tt>true</tt> if the given object is also a map and the two maps
 * represent identical object-reference mappings.  More formally, this
 * map is equal to another map <tt>m</tt> if and only if
 * <tt>this.entrySet().equals(m.entrySet())</tt>.
 *
 * <p><b>Owing to the reference-equality-based semantics of this map it is
 * possible that the symmetry and transitivity requirements of the
 * <tt>Object.equals</tt> contract may be violated if this map is compared
 * to a normal map.  However, the <tt>Object.equals</tt> contract is
 * guaranteed to hold among <tt>IdentityHashMap</tt> instances.</b>
 *
 * @param  o object to be compared for equality with this map
 * @return <tt>true</tt> if the specified object is equal to this map
 * @see Object#equals(Object)
 */
public boolean equals(Object o) {
    if (o == this) {
        return true;
    } else if (o instanceof IdentityHashMap) {
        IdentityHashMap<?,?> m = (IdentityHashMap<?,?>) o;
        if (m.size() != size)
            return false;

        Object[] tab = m.table;
        for (int i = 0; i < tab.length; i+=2) {
            Object k = tab[i];
            if (k != null && !containsMapping(k, tab[i + 1]))
                return false;
        }
        return true;
    } else if (o instanceof Map) {
        Map<?,?> m = (Map<?,?>)o;
        return entrySet().equals(m.entrySet());
    } else {
        return false;  // o is not a Map
    }
}
```

## Hash Code

每个 entry 的 key 与 value 异或后的和。

```java
/**
 * Returns the hash code value for this map.  The hash code of a map is
 * defined to be the sum of the hash codes of each entry in the map's
 * <tt>entrySet()</tt> view.  This ensures that <tt>m1.equals(m2)</tt>
 * implies that <tt>m1.hashCode()==m2.hashCode()</tt> for any two
 * <tt>IdentityHashMap</tt> instances <tt>m1</tt> and <tt>m2</tt>, as
 * required by the general contract of {@link Object#hashCode}.
 *
 * <p><b>Owing to the reference-equality-based semantics of the
 * <tt>Map.Entry</tt> instances in the set returned by this map's
 * <tt>entrySet</tt> method, it is possible that the contractual
 * requirement of <tt>Object.hashCode</tt> mentioned in the previous
 * paragraph will be violated if one of the two objects being compared is
 * an <tt>IdentityHashMap</tt> instance and the other is a normal map.</b>
 *
 * @return the hash code value for this map
 * @see Object#equals(Object)
 * @see #equals(Object)
 */
public int hashCode() {
    int result = 0;
    Object[] tab = table;
    for (int i = 0; i < tab.length; i +=2) {
        Object key = tab[i];
        if (key != null) {
            Object k = unmaskNull(key);
            result += System.identityHashCode(k) ^
                      System.identityHashCode(tab[i + 1]);
        }
    }
    return result;
}
```

## Clone

浅拷贝。

```java
/**
 * Returns a shallow copy of this identity hash map: the keys and values
 * themselves are not cloned.
 *
 * @return a shallow copy of this map
 */
public Object clone() {
    try {
        IdentityHashMap<?,?> m = (IdentityHashMap<?,?>) super.clone();
        m.entrySet = null;
        m.table = table.clone();
        return m;
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }
}
```

## Iterator

关于迭代器的基本实现：

```java
private abstract class IdentityHashMapIterator<T> implements Iterator<T> {}
```

内部成员变量：

```java
int index = (size != 0 ? 0 : table.length); // current slot.
int expectedModCount = modCount; // to support fast-fail
int lastReturnedIndex = -1;      // to allow remove()
boolean indexValid; // To avoid unnecessary next computation
Object[] traversalTable = table; // reference to main table or copy
```

由于哈希表的内部实现是数组，`index` 显然用于记录数组索引下标。

查看是否还有下一个迭代的元素：

```java
public boolean hasNext() {
    Object[] tab = traversalTable;
    for (int i = index; i < tab.length; i+=2) {
        Object key = tab[i];
        if (key != null) {
            index = i;
            return indexValid = true;
        }
    }
    index = tab.length;
    return false;
}
```

从迭代器的当前位置开始开始向后搜索。如果不为 `null`，则返回 `true`；如果为 `null`，则继续搜索，直到哈希表的最后。

以下两个函数没吃透，只知道大概的用途，以后再看吧：

```java
protected int nextIndex() {
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();
    if (!indexValid && !hasNext())
        throw new NoSuchElementException();

    indexValid = false;
    lastReturnedIndex = index;
    index += 2;
    return lastReturnedIndex;
}
```

```java
public void remove() {
    if (lastReturnedIndex == -1)
        throw new IllegalStateException();
    if (modCount != expectedModCount)
        throw new ConcurrentModificationException();

    expectedModCount = ++modCount;
    int deletedSlot = lastReturnedIndex;
    lastReturnedIndex = -1;
    // back up index to revisit new contents after deletion
    index = deletedSlot;
    indexValid = false;

    // Removal code proceeds as in closeDeletion except that
    // it must catch the rare case where an element already
    // seen is swapped into a vacant slot that will be later
    // traversed by this iterator. We cannot allow future
    // next() calls to return it again.  The likelihood of
    // this occurring under 2/3 load factor is very slim, but
    // when it does happen, we must make a copy of the rest of
    // the table to use for the rest of the traversal. Since
    // this can only happen when we are near the end of the table,
    // even in these rare cases, this is not very expensive in
    // time or space.

    Object[] tab = traversalTable;
    int len = tab.length;

    int d = deletedSlot;
    Object key = tab[d];
    tab[d] = null;        // vacate the slot
    tab[d + 1] = null;

    // If traversing a copy, remove in real table.
    // We can skip gap-closure on copy.
    if (tab != IdentityHashMap.this.table) {
        IdentityHashMap.this.remove(key);
        expectedModCount = modCount;
        return;
    }

    size--;

    Object item;
    for (int i = nextKeyIndex(d, len); (item = tab[i]) != null;
            i = nextKeyIndex(i, len)) {
        int r = hash(item, len);
        // See closeDeletion for explanation of this conditional
        if ((i < r && (r <= d || d <= i)) ||
            (r <= d && d <= i)) {

            // If we are about to swap an already-seen element
            // into a slot that may later be returned by next(),
            // then clone the rest of table for use in future
            // next() calls. It is OK that our copy will have
            // a gap in the "wrong" place, since it will never
            // be used for searching anyway.

            if (i < deletedSlot && d >= deletedSlot &&
                traversalTable == IdentityHashMap.this.table) {
                int remaining = len - deletedSlot;
                Object[] newTable = new Object[remaining];
                System.arraycopy(tab, deletedSlot,
                                    newTable, 0, remaining);
                traversalTable = newTable;
                index = 0;
            }

            tab[d] = item;
            tab[d + 1] = tab[i + 1];
            tab[i] = null;
            tab[i + 1] = null;
            d = i;
        }
    }
}
```
