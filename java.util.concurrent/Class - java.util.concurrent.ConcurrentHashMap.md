# Class - java.util.concurrent.ConcurrentHashMap

Created by : Mr Dk.

2020 / 10 / 14 16:47

Nanjing, Jiangsu, China

---

## Definition

支持全量并发的读取操作和高并发量的更新操作。对于依赖 **线程安全** 特性而不是 **同步细节** 的程序来说，与 `java.util.Hashtable` 可以互相替换。后续需要对这两个线程安全的容器作一个比较。

```java
/**
 * A hash table supporting full concurrency of retrievals and
 * high expected concurrency for updates. This class obeys the
 * same functional specification as {@link java.util.Hashtable}, and
 * includes versions of methods corresponding to each method of
 * {@code Hashtable}. However, even though all operations are
 * thread-safe, retrieval operations do <em>not</em> entail locking,
 * and there is <em>not</em> any support for locking the entire table
 * in a way that prevents all access.  This class is fully
 * interoperable with {@code Hashtable} in programs that rely on its
 * thread safety but not on its synchronization details.
 *
 * <p>Retrieval operations (including {@code get}) generally do not
 * block, so may overlap with update operations (including {@code put}
 * and {@code remove}). Retrievals reflect the results of the most
 * recently <em>completed</em> update operations holding upon their
 * onset. (More formally, an update operation for a given key bears a
 * <em>happens-before</em> relation with any (non-null) retrieval for
 * that key reporting the updated value.)  For aggregate operations
 * such as {@code putAll} and {@code clear}, concurrent retrievals may
 * reflect insertion or removal of only some entries.  Similarly,
 * Iterators, Spliterators and Enumerations return elements reflecting the
 * state of the hash table at some point at or since the creation of the
 * iterator/enumeration.  They do <em>not</em> throw {@link
 * java.util.ConcurrentModificationException ConcurrentModificationException}.
 * However, iterators are designed to be used by only one thread at a time.
 * Bear in mind that the results of aggregate status methods including
 * {@code size}, {@code isEmpty}, and {@code containsValue} are typically
 * useful only when a map is not undergoing concurrent updates in other threads.
 * Otherwise the results of these methods reflect transient states
 * that may be adequate for monitoring or estimation purposes, but not
 * for program control.
 *
 * <p>The table is dynamically expanded when there are too many
 * collisions (i.e., keys that have distinct hash codes but fall into
 * the same slot modulo the table size), with the expected average
 * effect of maintaining roughly two bins per mapping (corresponding
 * to a 0.75 load factor threshold for resizing). There may be much
 * variance around this average as mappings are added and removed, but
 * overall, this maintains a commonly accepted time/space tradeoff for
 * hash tables.  However, resizing this or any other kind of hash
 * table may be a relatively slow operation. When possible, it is a
 * good idea to provide a size estimate as an optional {@code
 * initialCapacity} constructor argument. An additional optional
 * {@code loadFactor} constructor argument provides a further means of
 * customizing initial table capacity by specifying the table density
 * to be used in calculating the amount of space to allocate for the
 * given number of elements.  Also, for compatibility with previous
 * versions of this class, constructors may optionally specify an
 * expected {@code concurrencyLevel} as an additional hint for
 * internal sizing.  Note that using many keys with exactly the same
 * {@code hashCode()} is a sure way to slow down performance of any
 * hash table. To ameliorate impact, when keys are {@link Comparable},
 * this class may use comparison order among keys to help break ties.
 *
 * <p>A {@link Set} projection of a ConcurrentHashMap may be created
 * (using {@link #newKeySet()} or {@link #newKeySet(int)}), or viewed
 * (using {@link #keySet(Object)} when only keys are of interest, and the
 * mapped values are (perhaps transiently) not used or all take the
 * same mapping value.
 *
 * <p>A ConcurrentHashMap can be used as scalable frequency map (a
 * form of histogram or multiset) by using {@link
 * java.util.concurrent.atomic.LongAdder} values and initializing via
 * {@link #computeIfAbsent computeIfAbsent}. For example, to add a count
 * to a {@code ConcurrentHashMap<String,LongAdder> freqs}, you can use
 * {@code freqs.computeIfAbsent(k -> new LongAdder()).increment();}
 *
 * <p>This class and its views and iterators implement all of the
 * <em>optional</em> methods of the {@link Map} and {@link Iterator}
 * interfaces.
 *
 * <p>Like {@link Hashtable} but unlike {@link HashMap}, this class
 * does <em>not</em> allow {@code null} to be used as a key or value.
 *
 * <p>ConcurrentHashMaps support a set of sequential and parallel bulk
 * operations that, unlike most {@link Stream} methods, are designed
 * to be safely, and often sensibly, applied even with maps that are
 * being concurrently updated by other threads; for example, when
 * computing a snapshot summary of the values in a shared registry.
 * There are three kinds of operation, each with four forms, accepting
 * functions with Keys, Values, Entries, and (Key, Value) arguments
 * and/or return values. Because the elements of a ConcurrentHashMap
 * are not ordered in any particular way, and may be processed in
 * different orders in different parallel executions, the correctness
 * of supplied functions should not depend on any ordering, or on any
 * other objects or values that may transiently change while
 * computation is in progress; and except for forEach actions, should
 * ideally be side-effect-free. Bulk operations on {@link java.util.Map.Entry}
 * objects do not support method {@code setValue}.
 *
 * <ul>
 * <li> forEach: Perform a given action on each element.
 * A variant form applies a given transformation on each element
 * before performing the action.</li>
 *
 * <li> search: Return the first available non-null result of
 * applying a given function on each element; skipping further
 * search when a result is found.</li>
 *
 * <li> reduce: Accumulate each element.  The supplied reduction
 * function cannot rely on ordering (more formally, it should be
 * both associative and commutative).  There are five variants:
 *
 * <ul>
 *
 * <li> Plain reductions. (There is not a form of this method for
 * (key, value) function arguments since there is no corresponding
 * return type.)</li>
 *
 * <li> Mapped reductions that accumulate the results of a given
 * function applied to each element.</li>
 *
 * <li> Reductions to scalar doubles, longs, and ints, using a
 * given basis value.</li>
 *
 * </ul>
 * </li>
 * </ul>
 *
 * <p>These bulk operations accept a {@code parallelismThreshold}
 * argument. Methods proceed sequentially if the current map size is
 * estimated to be less than the given threshold. Using a value of
 * {@code Long.MAX_VALUE} suppresses all parallelism.  Using a value
 * of {@code 1} results in maximal parallelism by partitioning into
 * enough subtasks to fully utilize the {@link
 * ForkJoinPool#commonPool()} that is used for all parallel
 * computations. Normally, you would initially choose one of these
 * extreme values, and then measure performance of using in-between
 * values that trade off overhead versus throughput.
 *
 * <p>The concurrency properties of bulk operations follow
 * from those of ConcurrentHashMap: Any non-null result returned
 * from {@code get(key)} and related access methods bears a
 * happens-before relation with the associated insertion or
 * update.  The result of any bulk operation reflects the
 * composition of these per-element relations (but is not
 * necessarily atomic with respect to the map as a whole unless it
 * is somehow known to be quiescent).  Conversely, because keys
 * and values in the map are never null, null serves as a reliable
 * atomic indicator of the current lack of any result.  To
 * maintain this property, null serves as an implicit basis for
 * all non-scalar reduction operations. For the double, long, and
 * int versions, the basis should be one that, when combined with
 * any other value, returns that other value (more formally, it
 * should be the identity element for the reduction). Most common
 * reductions have these properties; for example, computing a sum
 * with basis 0 or a minimum with basis MAX_VALUE.
 *
 * <p>Search and transformation functions provided as arguments
 * should similarly return null to indicate the lack of any result
 * (in which case it is not used). In the case of mapped
 * reductions, this also enables transformations to serve as
 * filters, returning null (or, in the case of primitive
 * specializations, the identity basis) if the element should not
 * be combined. You can create compound transformations and
 * filterings by composing them yourself under this "null means
 * there is nothing there now" rule before using them in search or
 * reduce operations.
 *
 * <p>Methods accepting and/or returning Entry arguments maintain
 * key-value associations. They may be useful for example when
 * finding the key for the greatest value. Note that "plain" Entry
 * arguments can be supplied using {@code new
 * AbstractMap.SimpleEntry(k,v)}.
 *
 * <p>Bulk operations may complete abruptly, throwing an
 * exception encountered in the application of a supplied
 * function. Bear in mind when handling such exceptions that other
 * concurrently executing functions could also have thrown
 * exceptions, or would have done so if the first exception had
 * not occurred.
 *
 * <p>Speedups for parallel compared to sequential forms are common
 * but not guaranteed.  Parallel operations involving brief functions
 * on small maps may execute more slowly than sequential forms if the
 * underlying work to parallelize the computation is more expensive
 * than the computation itself.  Similarly, parallelization may not
 * lead to much actual parallelism if all processors are busy
 * performing unrelated tasks.
 *
 * <p>All arguments to all task methods must be non-null.
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @since 1.5
 * @author Doug Lea
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 */
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable {

}
```

---

## Put

由于之前已经分析过 `java.util.HashMap` 的源代码，对于并发版的容器来说，我决定从 `put()` 函数出发，梳理其中的并发特性。

```java
/**
 * Maps the specified key to the specified value in this table.
 * Neither the key nor the value can be null.
 *
 * <p>The value can be retrieved by calling the {@code get} method
 * with a key that is equal to the original key.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with {@code key}, or
 *         {@code null} if there was no mapping for {@code key}
 * @throws NullPointerException if the specified key or value is null
 */
public V put(K key, V value) {
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                            new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                    (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                            value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                        value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

首先，既然要 `put()` 一个新元素，首先需要计算它的 hashcode，确定把它放到哪一个 slot 中。计算 hashcode 的函数如下所示。可以看到这里做了一个与运算，抹去了 hashcode 的最高位，保证了 hashcode 肯定是一个正数。

```java
static final int HASH_BITS = 0x7fffffff; // usable bits of normal node hash

/**
 * Spreads (XORs) higher bits of hash to lower and also forces top
 * bit to 0. Because the table uses power-of-two masking, sets of
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
static final int spread(int h) {
    return (h ^ (h >>> 16)) & HASH_BITS;
}
```

接下来就是对整个 HashMap 进行 for 循环遍历了。遍历的目标是核心数据结构，也就是 HashMap 对应的 key-value 数组：

```java
/**
 * The array of bins. Lazily initialized upon first insertion.
 * Size is always a power of two. Accessed directly by iterators.
 */
transient volatile Node<K,V>[] table;
```

For 循环中的几个分支分别对应了几种情况：

1. `table` 为空，则进行初始化 (懒初始化)，只允许一个线程初始化
2. hashcode 所在的槽为空，那么直接通过 CAS 操作把新结点放进去 (可能会有多个线程同时试图放进去，所以要 CAS)
3. 该位置正被扩容，则当前线程也加入该位置的辅助扩容中
4. 该位置所在槽不为空，且没有在扩容，那么通过 `synchronized` 锁住这个位置，然后开始单线程操作

```java
if (tab == null || (n = tab.length) == 0)
    tab = initTable();
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
    if (casTabAt(tab, i, null,
                    new Node<K,V>(hash, key, value, null)))
        break;                   // no lock when adding to empty bin
}
else if ((fh = f.hash) == MOVED)
    tab = helpTransfer(tab, f);
else {
    V oldVal = null;
    synchronized (f) {
        if (tabAt(tab, i) == f) {
            if (fh >= 0) {
                binCount = 1;
                for (Node<K,V> e = f;; ++binCount) {
                    K ek;
                    if (e.hash == hash &&
                        ((ek = e.key) == key ||
                            (ek != null && key.equals(ek)))) {
                        oldVal = e.val;
                        if (!onlyIfAbsent)
                            e.val = value;
                        break;
                    }
                    Node<K,V> pred = e;
                    if ((e = e.next) == null) {
                        pred.next = new Node<K,V>(hash, key,
                                                    value, null);
                        break;
                    }
                }
            }
            else if (f instanceof TreeBin) {
                Node<K,V> p;
                binCount = 2;
                if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                value)) != null) {
                    oldVal = p.val;
                    if (!onlyIfAbsent)
                        p.val = value;
                }
            }
        }
    }
    if (binCount != 0) {
        if (binCount >= TREEIFY_THRESHOLD)
            treeifyBin(tab, i);
        if (oldVal != null)
            return oldVal;
        break;
    }
}
```

第一步是初始化 key-value 数组 - 这件事由一个线程来完成就够了。在这个类中，由 `sizeCtl` 成员变量来确定由哪个线程负责初始化。一开始该变量的值为 `0`，多个线程通过 CAS 操作试图将其赋值为 `-1`，只有 CAS 成功了的那个线程可以负责进行初始化，其它线程全部自旋等待：

```java
/**
 * Table initialization and resizing control.  When negative, the
 * table is being initialized or resized: -1 for initialization,
 * else -(1 + the number of active resizing threads).  Otherwise,
 * when table is null, holds the initial table size to use upon
 * creation, or 0 for default. After initialization, holds the
 * next element count value upon which to resize the table.
 */
private transient volatile int sizeCtl;

/**
 * Initializes table, using the size recorded in sizeCtl.
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0) // 已经有一个线程将 sizeCtl 赋值为 -1 了，那么自旋等待
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) { // CAS 将 sizeCtl 赋值为 -1
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n]; // 初始化数组
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

第二步，数组中的 slot 为空。那没什么好说的，初始化一个新的 Node 然后放进去就可以了。需要注意的是可能有多个线程同时想往这个位置放新的 Node，因此放入的过程一定是 CAS 的：

```java
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
    if (casTabAt(tab, i, null,
                    new Node<K,V>(hash, key, value, null)))
        break;                   // no lock when adding to empty bin
}
```

```java
@SuppressWarnings("unchecked")
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}
```

第三步，如果发现当前要放入 Node 的位置 (表头结点) 正在进行扩容，则当前线程也加入辅助扩容。正在进行扩容时，`sizeCtl` 用于记录正在进行扩容的并发线程数 - 所以每个线程要通过 CAS 把 `sizeCtl` 自增后才会进入扩容；扩容结束后，再将 `sizeCtl` 减下来。每个线程会负责一定量的 slot 迁移，具体数量由 CPU 核心数与 `table` 数组长度决定，每个线程至少要负责 16 个 slot 的迁移。

在扩容过程中：

* 还没开始迁移的 slot 可以正常进行 `get()` / `put()`
* 正在进行迁移的 slot - `get()` 不被阻塞，`put()` 操作被阻塞
* 迁移已完毕的 slot - `get()` / `put()` 操作会被转发到扩容后的新数组上

```java
else if ((fh = f.hash) == MOVED)
    tab = helpTransfer(tab, f);
```

第四步，即要插入的位置已有元素 `f`，且未在迁移中，那么锁定该表头元素，然后当前线程独占这个位置，直到插入完成。插入分为链表和红黑树两种形式。在插入结束后，判断一次是否当前位置是否需要进行链表 -> 红黑树的转换。

```java
else {
    V oldVal = null;
    synchronized (f) {
        if (tabAt(tab, i) == f) {
            if (fh >= 0) {
                binCount = 1;
                for (Node<K,V> e = f;; ++binCount) {
                    K ek;
                    if (e.hash == hash &&
                        ((ek = e.key) == key ||
                            (ek != null && key.equals(ek)))) {
                        oldVal = e.val;
                        if (!onlyIfAbsent)
                            e.val = value;
                        break;
                    }
                    Node<K,V> pred = e;
                    if ((e = e.next) == null) {
                        pred.next = new Node<K,V>(hash, key,
                                                    value, null);
                        break;
                    }
                }
            }
            else if (f instanceof TreeBin) {
                Node<K,V> p;
                binCount = 2;
                if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                value)) != null) {
                    oldVal = p.val;
                    if (!onlyIfAbsent)
                        p.val = value;
                }
            }
        }
    }
    if (binCount != 0) {
        if (binCount >= TREEIFY_THRESHOLD)
            treeifyBin(tab, i);
        if (oldVal != null)
            return oldVal;
        break;
    }
}
```

---

## Get

关于 `get()` 操作，可以看到是全程无锁的。首先用 key 计算 hashcode，判断出结点所在的 slot。如果 key 和表头的 key 相等则直接返回，否则就需要对冲突链表进行遍历。

```java
/**
 * Returns the value to which the specified key is mapped,
 * or {@code null} if this map contains no mapping for the key.
 *
 * <p>More formally, if this map contains a mapping from a key
 * {@code k} to a value {@code v} such that {@code key.equals(k)},
 * then this method returns {@code v}; otherwise it returns
 * {@code null}.  (There can be at most one such mapping.)
 *
 * @throws NullPointerException if the specified key is null
 */
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

---

## Comparison with Collections.synchronizedMap()

```java
Map m = Collections.synchronizedMap(new HashMap());
```

`Collections` 类对普通 HashMap 的并发封装具体做了什么工作呢？实际上，该类只是将 HashMap 包了一层，包装为 SynchronizedMap。

```java
/**
 * Returns a synchronized (thread-safe) map backed by the specified
 * map.  In order to guarantee serial access, it is critical that
 * <strong>all</strong> access to the backing map is accomplished
 * through the returned map.<p>
 *
 * It is imperative that the user manually synchronize on the returned
 * map when iterating over any of its collection views:
 * <pre>
 *  Map m = Collections.synchronizedMap(new HashMap());
 *      ...
 *  Set s = m.keySet();  // Needn't be in synchronized block
 *      ...
 *  synchronized (m) {  // Synchronizing on m, not s!
 *      Iterator i = s.iterator(); // Must be in synchronized block
 *      while (i.hasNext())
 *          foo(i.next());
 *  }
 * </pre>
 * Failure to follow this advice may result in non-deterministic behavior.
 *
 * <p>The returned map will be serializable if the specified map is
 * serializable.
 *
 * @param <K> the class of the map keys
 * @param <V> the class of the map values
 * @param  m the map to be "wrapped" in a synchronized map.
 * @return a synchronized view of the specified map.
 */
public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
    return new SynchronizedMap<>(m);
}
```

而在这个包装出的 SynchronizedMap 中，额外维护了一个对象 `mutex` 作为锁。这个锁对象可以由用户通过构造函数指定，如不指定默认为当前的 SynchronizedMap 对象自身。

```java
/**
 * @serial include
 */
private static class SynchronizedMap<K,V>
    implements Map<K,V>, Serializable {
    private static final long serialVersionUID = 1978198479659022715L;

    private final Map<K,V> m;     // Backing Map
    final Object      mutex;        // Object on which to synchronize

    SynchronizedMap(Map<K,V> m) {
        this.m = Objects.requireNonNull(m);
        mutex = this;
    }

    SynchronizedMap(Map<K,V> m, Object mutex) {
        this.m = m;
        this.mutex = mutex;
    }

    // ...
}
```

有了这个锁对象后，对于所有的操作，首先要拿到这个锁，然后再进行相应的操作，从而实现了线程同步：

```java
public int size() {
    synchronized (mutex) {return m.size();}
}
public boolean isEmpty() {
    synchronized (mutex) {return m.isEmpty();}
}
public boolean containsKey(Object key) {
    synchronized (mutex) {return m.containsKey(key);}
}
public boolean containsValue(Object value) {
    synchronized (mutex) {return m.containsValue(value);}
}
public V get(Object key) {
    synchronized (mutex) {return m.get(key);}
}

public V put(K key, V value) {
    synchronized (mutex) {return m.put(key, value);}
}
public V remove(Object key) {
    synchronized (mutex) {return m.remove(key);}
}
public void putAll(Map<? extends K, ? extends V> map) {
    synchronized (mutex) {m.putAll(map);}
}
public void clear() {
    synchronized (mutex) {m.clear();}
}

private transient Set<K> keySet;
private transient Set<Map.Entry<K,V>> entrySet;
private transient Collection<V> values;

public Set<K> keySet() {
    synchronized (mutex) {
        if (keySet==null)
            keySet = new SynchronizedSet<>(m.keySet(), mutex);
        return keySet;
    }
}

public Set<Map.Entry<K,V>> entrySet() {
    synchronized (mutex) {
        if (entrySet==null)
            entrySet = new SynchronizedSet<>(m.entrySet(), mutex);
        return entrySet;
    }
}

public Collection<V> values() {
    synchronized (mutex) {
        if (values==null)
            values = new SynchronizedCollection<>(m.values(), mutex);
        return values;
    }
}

public boolean equals(Object o) {
    if (this == o)
        return true;
    synchronized (mutex) {return m.equals(o);}
}
public int hashCode() {
    synchronized (mutex) {return m.hashCode();}
}
public String toString() {
    synchronized (mutex) {return m.toString();}
}

// Override default methods in Map
@Override
public V getOrDefault(Object k, V defaultValue) {
    synchronized (mutex) {return m.getOrDefault(k, defaultValue);}
}
@Override
public void forEach(BiConsumer<? super K, ? super V> action) {
    synchronized (mutex) {m.forEach(action);}
}
@Override
public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
    synchronized (mutex) {m.replaceAll(function);}
}
@Override
public V putIfAbsent(K key, V value) {
    synchronized (mutex) {return m.putIfAbsent(key, value);}
}
@Override
public boolean remove(Object key, Object value) {
    synchronized (mutex) {return m.remove(key, value);}
}
@Override
public boolean replace(K key, V oldValue, V newValue) {
    synchronized (mutex) {return m.replace(key, oldValue, newValue);}
}
@Override
public V replace(K key, V value) {
    synchronized (mutex) {return m.replace(key, value);}
}
@Override
public V computeIfAbsent(K key,
        Function<? super K, ? extends V> mappingFunction) {
    synchronized (mutex) {return m.computeIfAbsent(key, mappingFunction);}
}
@Override
public V computeIfPresent(K key,
        BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    synchronized (mutex) {return m.computeIfPresent(key, remappingFunction);}
}
@Override
public V compute(K key,
        BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    synchronized (mutex) {return m.compute(key, remappingFunction);}
}
@Override
public V merge(K key, V value,
        BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    synchronized (mutex) {return m.merge(key, value, remappingFunction);}
}

private void writeObject(ObjectOutputStream s) throws IOException {
    synchronized (mutex) {s.defaultWriteObject();}
}
```

## Compared with Hashtable

Hashtable 中基本所有的函数都被 `synchronized` 关键字修饰 (效率慢？)。

> 从 Java 8 开始，`synchronized` 关键字实际上会经历锁升级过程逐步重量化。所以到底是无锁的 CAS 更快还是 `synchronized` 更快好像也很难下定论。

---

## References

[知乎专栏 - 《进大厂系列》系列-ConcurrentHashMap & HashTable](https://zhuanlan.zhihu.com/p/97902016)

[知乎专栏 - ConcurrentHashMap 源码分析 (基于 JDK 1.8)](https://zhuanlan.zhihu.com/p/134569320)

---

