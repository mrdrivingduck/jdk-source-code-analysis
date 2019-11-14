# Class - java.util.TreeMap

Created by : Mr Dk.

2019 / 11 / 14 10:43

Nanjing, Jiangsu, China

---

## Definition

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
{

}
```

基于红黑树实现的 `NavigableMap`

* Map 基于自然顺序或构造时提供的 `Comparator` 进行排序

对于下列操作保证 `log(n)` 的时间复杂度：

* `containsKey()`
* `get()`
* `put()`
* `remove()`

对于 Map 接口来说，操作使用 `equals()` 函数

对于 SortedMap 接口来说，操作使用 `compareTo()` 函数

TreeMap 的实现是不同步的

* 要么封装该集合对象负责实现同步
* 要么就用 `Collections.synchronizedSortedMap(new TreeMap(...))` 来封装

在迭代器迭代过程中，如果 Map 发生结构修改，则会立刻抛出错误

* 除非使用迭代器自己的 `remove()`

所有函数中返回的 `Map.Entry<>` 都是映射的快照

* 不支持 `Entry.setValue()`
* 只能使用 `put()` 来改变映射

```java
/**
 * A Red-Black tree based {@link NavigableMap} implementation.
 * The map is sorted according to the {@linkplain Comparable natural
 * ordering} of its keys, or by a {@link Comparator} provided at map
 * creation time, depending on which constructor is used.
 *
 * <p>This implementation provides guaranteed log(n) time cost for the
 * {@code containsKey}, {@code get}, {@code put} and {@code remove}
 * operations.  Algorithms are adaptations of those in Cormen, Leiserson, and
 * Rivest's <em>Introduction to Algorithms</em>.
 *
 * <p>Note that the ordering maintained by a tree map, like any sorted map, and
 * whether or not an explicit comparator is provided, must be <em>consistent
 * with {@code equals}</em> if this sorted map is to correctly implement the
 * {@code Map} interface.  (See {@code Comparable} or {@code Comparator} for a
 * precise definition of <em>consistent with equals</em>.)  This is so because
 * the {@code Map} interface is defined in terms of the {@code equals}
 * operation, but a sorted map performs all key comparisons using its {@code
 * compareTo} (or {@code compare}) method, so two keys that are deemed equal by
 * this method are, from the standpoint of the sorted map, equal.  The behavior
 * of a sorted map <em>is</em> well-defined even if its ordering is
 * inconsistent with {@code equals}; it just fails to obey the general contract
 * of the {@code Map} interface.
 *
 * <p><strong>Note that this implementation is not synchronized.</strong>
 * If multiple threads access a map concurrently, and at least one of the
 * threads modifies the map structurally, it <em>must</em> be synchronized
 * externally.  (A structural modification is any operation that adds or
 * deletes one or more mappings; merely changing the value associated
 * with an existing key is not a structural modification.)  This is
 * typically accomplished by synchronizing on some object that naturally
 * encapsulates the map.
 * If no such object exists, the map should be "wrapped" using the
 * {@link Collections#synchronizedSortedMap Collections.synchronizedSortedMap}
 * method.  This is best done at creation time, to prevent accidental
 * unsynchronized access to the map: <pre>
 *   SortedMap m = Collections.synchronizedSortedMap(new TreeMap(...));</pre>
 *
 * <p>The iterators returned by the {@code iterator} method of the collections
 * returned by all of this class's "collection view methods" are
 * <em>fail-fast</em>: if the map is structurally modified at any time after
 * the iterator is created, in any way except through the iterator's own
 * {@code remove} method, the iterator will throw a {@link
 * ConcurrentModificationException}.  Thus, in the face of concurrent
 * modification, the iterator fails quickly and cleanly, rather than risking
 * arbitrary, non-deterministic behavior at an undetermined time in the future.
 *
 * <p>Note that the fail-fast behavior of an iterator cannot be guaranteed
 * as it is, generally speaking, impossible to make any hard guarantees in the
 * presence of unsynchronized concurrent modification.  Fail-fast iterators
 * throw {@code ConcurrentModificationException} on a best-effort basis.
 * Therefore, it would be wrong to write a program that depended on this
 * exception for its correctness:   <em>the fail-fast behavior of iterators
 * should be used only to detect bugs.</em>
 *
 * <p>All {@code Map.Entry} pairs returned by methods in this class
 * and its views represent snapshots of mappings at the time they were
 * produced. They do <strong>not</strong> support the {@code Entry.setValue}
 * method. (Note however that it is possible to change mappings in the
 * associated map using {@code put}.)
 *
 * <p>This class is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch and Doug Lea
 * @see Map
 * @see HashMap
 * @see Hashtable
 * @see Comparable
 * @see Comparator
 * @see Collection
 * @since 1.2
 */
```

---

```java
/**
 * The comparator used to maintain order in this tree map, or
 * null if it uses the natural ordering of its keys.
 *
 * @serial
 */
private final Comparator<? super K> comparator;

private transient Entry<K,V> root;

/**
 * The number of entries in the tree
 */
private transient int size = 0;

/**
 * The number of structural modifications to the tree.
 */
private transient int modCount = 0;
```

局部变量

主要维护 Map 的 size

以及 Map 的根结点

---

构造函数

```java
/**
 * Constructs a new, empty tree map, using the natural ordering of its
 * keys.  All keys inserted into the map must implement the {@link
 * Comparable} interface.  Furthermore, all such keys must be
 * <em>mutually comparable</em>: {@code k1.compareTo(k2)} must not throw
 * a {@code ClassCastException} for any keys {@code k1} and
 * {@code k2} in the map.  If the user attempts to put a key into the
 * map that violates this constraint (for example, the user attempts to
 * put a string key into a map whose keys are integers), the
 * {@code put(Object key, Object value)} call will throw a
 * {@code ClassCastException}.
 */
public TreeMap() {
    comparator = null;
}
```

创建一个新的空集合，并组织为 key 的自然排列顺序

```java
/**
 * Constructs a new, empty tree map, ordered according to the given
 * comparator.  All keys inserted into the map must be <em>mutually
 * comparable</em> by the given comparator: {@code comparator.compare(k1,
 * k2)} must not throw a {@code ClassCastException} for any keys
 * {@code k1} and {@code k2} in the map.  If the user attempts to put
 * a key into the map that violates this constraint, the {@code put(Object
 * key, Object value)} call will throw a
 * {@code ClassCastException}.
 *
 * @param comparator the comparator that will be used to order this map.
 *        If {@code null}, the {@linkplain Comparable natural
 *        ordering} of the keys will be used.
 */
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}
```

创建新的空集合，并用 `Comparator` 组织 key 的排列

```java
/**
 * Constructs a new tree map containing the same mappings as the given
 * map, ordered according to the <em>natural ordering</em> of its keys.
 * All keys inserted into the new map must implement the {@link
 * Comparable} interface.  Furthermore, all such keys must be
 * <em>mutually comparable</em>: {@code k1.compareTo(k2)} must not throw
 * a {@code ClassCastException} for any keys {@code k1} and
 * {@code k2} in the map.  This method runs in n*log(n) time.
 *
 * @param  m the map whose mappings are to be placed in this map
 * @throws ClassCastException if the keys in m are not {@link Comparable},
 *         or are not mutually comparable
 * @throws NullPointerException if the specified map is null
 */
public TreeMap(Map<? extends K, ? extends V> m) {
    comparator = null;
    putAll(m);
}
```

用一个已有的 Map 构造 TreeMap

按照自然顺序来组织 key，将 Map 中的每一个元素放入 TreeMap 中

```java
/**
 * Constructs a new tree map containing the same mappings and
 * using the same ordering as the specified sorted map.  This
 * method runs in linear time.
 *
 * @param  m the sorted map whose mappings are to be placed in this map,
 *         and whose comparator is to be used to sort this map
 * @throws NullPointerException if the specified map is null
 */
public TreeMap(SortedMap<K, ? extends V> m) {
    comparator = m.comparator();
    try {
        buildFromSorted(m.size(), m.entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }
}
```

用另一个 SortedMap 来构造 TreeMap

使用 SortedMap 的 `Comparator` 来组织 key 的排列

从而具有相同的 key 顺序

---

红黑树结点的定义：

```java
// Red-black mechanics

private static final boolean RED   = false;
private static final boolean BLACK = true;
```

```java
/**
 * Node in the Tree.  Doubles as a means to pass key-value pairs back to
 * user (see Map.Entry).
 */

static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;

    /**
     * Make a new cell with given key, value, and parent, and with
     * {@code null} child links, and BLACK color.
     */
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }

    /**
     * Returns the key.
     *
     * @return the key
     */
    public K getKey() {
        return key;
    }

    /**
     * Returns the value associated with the key.
     *
     * @return the value associated with the key
     */
    public V getValue() {
        return value;
    }

    /**
     * Replaces the value currently associated with the key with the given
     * value.
     *
     * @return the value associated with the key before this method was
     *         called
     */
    public V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;

        return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
    }

    public int hashCode() {
        int keyHash = (key==null ? 0 : key.hashCode());
        int valueHash = (value==null ? 0 : value.hashCode());
        return keyHash ^ valueHash;
    }

    public String toString() {
        return key + "=" + value;
    }
}
```

给定父结点、key、value 可以构造一个新结点

* 孩子指针为 `null`
* 结点默认颜色为黑色

---

```java
/**
 * Returns the successor of the specified Entry, or null if no such.
 */
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```

返回后一个 entry

* 首先寻找右子树的最左下结点
* 如果没有右子树，则需要回溯到 parent 结点
    * 如果当前结点为 parent 的左子树，下一个 entry 应当就是 parent 的 entry
    * 如果当前结点为 parent 的右子树，则需要一直向上回溯，直到该子树为某个结点的左子树为止
        * 该结点就是下一个结点

```java
/**
 * Returns the predecessor of the specified Entry, or null if no such.
 */
static <K,V> Entry<K,V> predecessor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.left != null) {
        Entry<K,V> p = t.left;
        while (p.right != null)
            p = p.right;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.left) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```

返回前一个 entry

* 首先寻找左子树的最右下结点
* 如果没有左子树，则向 parent 回溯
    * 如果当前结点为 parent 的右子树，那么 parent 的 entry 就是前一个 entry
    * 如果当前结点为 parent 的左子树，那么需要一直回溯，直到子树成为某个结点的右子树为止
        * 该结点就是前一个结点

---

```java
/**
 * Balancing operations.
 *
 * Implementations of rebalancings during insertion and deletion are
 * slightly different than the CLR version.  Rather than using dummy
 * nilnodes, we use a set of accessors that deal properly with null.  They
 * are used to avoid messiness surrounding nullness checks in the main
 * algorithms.
 */

private static <K,V> boolean colorOf(Entry<K,V> p) {
    return (p == null ? BLACK : p.color);
}

private static <K,V> Entry<K,V> parentOf(Entry<K,V> p) {
    return (p == null ? null: p.parent);
}

private static <K,V> void setColor(Entry<K,V> p, boolean c) {
    if (p != null)
        p.color = c;
}

private static <K,V> Entry<K,V> leftOf(Entry<K,V> p) {
    return (p == null) ? null: p.left;
}

private static <K,V> Entry<K,V> rightOf(Entry<K,V> p) {
    return (p == null) ? null: p.right;
}
```

红黑树操作的辅助函数

* 返回或设置颜色
* 返回左子树 / 右子树 / 父结点

```java
/** From CLR */
private void rotateLeft(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> r = p.right;
        p.right = r.left;
        if (r.left != null)
            r.left.parent = p;
        r.parent = p.parent;
        if (p.parent == null)
            root = r;
        else if (p.parent.left == p)
            p.parent.left = r;
        else
            p.parent.right = r;
        r.left = p;
        p.parent = r;
    }
}

/** From CLR */
private void rotateRight(Entry<K,V> p) {
    if (p != null) {
        Entry<K,V> l = p.left;
        p.left = l.right;
        if (l.right != null) l.right.parent = p;
        l.parent = p.parent;
        if (p.parent == null)
            root = l;
        else if (p.parent.right == p)
            p.parent.right = l;
        else p.parent.left = l;
        l.right = p;
        p.parent = l;
    }
}

/** From CLR */
private void fixAfterInsertion(Entry<K,V> x) {
    x.color = RED;

    while (x != null && x != root && x.parent.color == RED) {
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == rightOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateLeft(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateRight(parentOf(parentOf(x)));
            }
        } else {
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);
                setColor(y, BLACK);
                setColor(parentOf(parentOf(x)), RED);
                x = parentOf(parentOf(x));
            } else {
                if (x == leftOf(parentOf(x))) {
                    x = parentOf(x);
                    rotateRight(x);
                }
                setColor(parentOf(x), BLACK);
                setColor(parentOf(parentOf(x)), RED);
                rotateLeft(parentOf(parentOf(x)));
            }
        }
    }
    root.color = BLACK;
}

/**
    * Delete node p, and then rebalance the tree.
    */
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    // If strictly internal, copy successor's element to p and then make p
    // point to successor.
    if (p.left != null && p.right != null) {
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    } // p has 2 children

    // Start fixup at replacement node, if it exists.
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

    if (replacement != null) {
        // Link replacement to parent
        replacement.parent = p.parent;
        if (p.parent == null)
            root = replacement;
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        else
            p.parent.right = replacement;

        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;

        // Fix replacement
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) { // return if we are the only node.
        root = null;
    } else { //  No children. Use self as phantom replacement and unlink.
        if (p.color == BLACK)
            fixAfterDeletion(p);

        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}

/** From CLR */
private void fixAfterDeletion(Entry<K,V> x) {
    while (x != root && colorOf(x) == BLACK) {
        if (x == leftOf(parentOf(x))) {
            Entry<K,V> sib = rightOf(parentOf(x));

            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateLeft(parentOf(x));
                sib = rightOf(parentOf(x));
            }

            if (colorOf(leftOf(sib))  == BLACK &&
                colorOf(rightOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                if (colorOf(rightOf(sib)) == BLACK) {
                    setColor(leftOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateRight(sib);
                    sib = rightOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(rightOf(sib), BLACK);
                rotateLeft(parentOf(x));
                x = root;
            }
        } else { // symmetric
            Entry<K,V> sib = leftOf(parentOf(x));

            if (colorOf(sib) == RED) {
                setColor(sib, BLACK);
                setColor(parentOf(x), RED);
                rotateRight(parentOf(x));
                sib = leftOf(parentOf(x));
            }

            if (colorOf(rightOf(sib)) == BLACK &&
                colorOf(leftOf(sib)) == BLACK) {
                setColor(sib, RED);
                x = parentOf(x);
            } else {
                if (colorOf(leftOf(sib)) == BLACK) {
                    setColor(rightOf(sib), BLACK);
                    setColor(sib, RED);
                    rotateLeft(sib);
                    sib = leftOf(parentOf(x));
                }
                setColor(sib, colorOf(parentOf(x)));
                setColor(parentOf(x), BLACK);
                setColor(leftOf(sib), BLACK);
                rotateRight(parentOf(x));
                x = root;
            }
        }
    }

    setColor(x, BLACK);
}
```

具体的红黑树操作函数

* 左旋 / 右旋
* 插入和删除后的修复平衡性操作

不是说写得不好，~~只能说是没啥可读性了~~

---

```java
/**
 * Linear time tree building algorithm from sorted data.  Can accept keys
 * and/or values from iterator or stream. This leads to too many
 * parameters, but seems better than alternatives.  The four formats
 * that this method accepts are:
 *
 *    1) An iterator of Map.Entries.  (it != null, defaultVal == null).
 *    2) An iterator of keys.         (it != null, defaultVal != null).
 *    3) A stream of alternating serialized keys and values.
 *                                   (it == null, defaultVal == null).
 *    4) A stream of serialized keys. (it == null, defaultVal != null).
 *
 * It is assumed that the comparator of the TreeMap is already set prior
 * to calling this method.
 *
 * @param size the number of keys (or key-value pairs) to be read from
 *        the iterator or stream
 * @param it If non-null, new entries are created from entries
 *        or keys read from this iterator.
 * @param str If non-null, new entries are created from keys and
 *        possibly values read from this stream in serialized form.
 *        Exactly one of it and str should be non-null.
 * @param defaultVal if non-null, this default value is used for
 *        each value in the map.  If null, each value is read from
 *        iterator or stream, as described above.
 * @throws java.io.IOException propagated from stream reads. This cannot
 *         occur if str is null.
 * @throws ClassNotFoundException propagated from readObject.
 *         This cannot occur if str is null.
 */
private void buildFromSorted(int size, Iterator<?> it,
                                java.io.ObjectInputStream str,
                                V defaultVal)
    throws  java.io.IOException, ClassNotFoundException {
    this.size = size;
    root = buildFromSorted(0, 0, size-1, computeRedLevel(size),
                            it, str, defaultVal);
}

/**
 * Recursive "helper method" that does the real work of the
 * previous method.  Identically named parameters have
 * identical definitions.  Additional parameters are documented below.
 * It is assumed that the comparator and size fields of the TreeMap are
 * already set prior to calling this method.  (It ignores both fields.)
 *
 * @param level the current level of tree. Initial call should be 0.
 * @param lo the first element index of this subtree. Initial should be 0.
 * @param hi the last element index of this subtree.  Initial should be
 *        size-1.
 * @param redLevel the level at which nodes should be red.
 *        Must be equal to computeRedLevel for tree of this size.
 */
@SuppressWarnings("unchecked")
private final Entry<K,V> buildFromSorted(int level, int lo, int hi,
                                            int redLevel,
                                            Iterator<?> it,
                                            java.io.ObjectInputStream str,
                                            V defaultVal)
    throws  java.io.IOException, ClassNotFoundException {
    /*
     * Strategy: The root is the middlemost element. To get to it, we
     * have to first recursively construct the entire left subtree,
     * so as to grab all of its elements. We can then proceed with right
     * subtree.
     *
     * The lo and hi arguments are the minimum and maximum
     * indices to pull out of the iterator or stream for current subtree.
     * They are not actually indexed, we just proceed sequentially,
     * ensuring that items are extracted in corresponding order.
     */

    if (hi < lo) return null;

    int mid = (lo + hi) >>> 1;

    Entry<K,V> left  = null;
    if (lo < mid)
        left = buildFromSorted(level+1, lo, mid - 1, redLevel,
                                it, str, defaultVal);

    // extract key and/or value from iterator or stream
    K key;
    V value;
    if (it != null) {
        if (defaultVal==null) {
            Map.Entry<?,?> entry = (Map.Entry<?,?>)it.next();
            key = (K)entry.getKey();
            value = (V)entry.getValue();
        } else {
            key = (K)it.next();
            value = defaultVal;
        }
    } else { // use stream
        key = (K) str.readObject();
        value = (defaultVal != null ? defaultVal : (V) str.readObject());
    }

    Entry<K,V> middle =  new Entry<>(key, value, null);

    // color nodes in non-full bottommost level red
    if (level == redLevel)
        middle.color = RED;

    if (left != null) {
        middle.left = left;
        left.parent = middle;
    }

    if (mid < hi) {
        Entry<K,V> right = buildFromSorted(level+1, mid+1, hi, redLevel,
                                            it, str, defaultVal);
        middle.right = right;
        right.parent = middle;
    }

    return middle;
}
```

这是一个递归函数

* `lo` 和 `hi` 分别表示该树的第一个元素和最后一个元素

分别构造左右子树，并设置子树与父结点之间的引用关系，返回中间结点

递归结束后，函数最终返回 root 结点

> 关于红黑树的具体细节不想看了 😣

---

```java
/**
 * Returns {@code true} if this map contains a mapping for the specified
 * key.
 *
 * @param key key whose presence in this map is to be tested
 * @return {@code true} if this map contains a mapping for the
 *         specified key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
public boolean containsKey(Object key) {
    return getEntry(key) != null;
}
```

```java
/**
 * Returns the value to which the specified key is mapped,
 * or {@code null} if this map contains no mapping for the key.
 *
 * <p>More formally, if this map contains a mapping from a key
 * {@code k} to a value {@code v} such that {@code key} compares
 * equal to {@code k} according to the map's ordering, then this
 * method returns {@code v}; otherwise it returns {@code null}.
 * (There can be at most one such mapping.)
 *
 * <p>A return value of {@code null} does not <em>necessarily</em>
 * indicate that the map contains no mapping for the key; it's also
 * possible that the map explicitly maps the key to {@code null}.
 * The {@link #containsKey containsKey} operation may be used to
 * distinguish these two cases.
 *
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}
```

查找 Map 中是否存在某个 key

以及，取得某个 key 对应的 value

这两个操作都需要从 root 结点出发进行搜索

```java
/**
 * Returns this map's entry for the given key, or {@code null} if the map
 * does not contain an entry for the key.
 *
 * @return this map's entry for the given key, or {@code null} if the map
 *         does not contain an entry for the key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
final Entry<K,V> getEntry(Object key) {
    // Offload comparator-based version for sake of performance
    if (comparator != null)
        return getEntryUsingComparator(key);
    if (key == null)
        throw new NullPointerException();
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}

/**
 * Version of getEntry using comparator. Split off from getEntry
 * for performance. (This is not worth doing for most methods,
 * that are less dependent on comparator performance, but is
 * worthwhile here.)
 */
final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
        K k = (K) key;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
    }
    return null;
}
```

* 根据 `Comparator` 是否为 `null`，选择调用两个版本的查找
    * 自然顺序版本 - 使用 key 的 `compareTo()` 进行比较
    * 使用 `Comparator` 比较的版本 - 使用 `Comparator` 的 `compare()` 进行比较

从 root 结点开始比较

* 如果 key 比当前结点小，则进入左子树 (或 `null`)
* 如果 key 比当前结点大，则进入右子树 (或 `null`)
* 如果 key 相同，则找到
* 否则就是 `null` (不存在)

---

```java
/**
 * Returns {@code true} if this map maps one or more keys to the
 * specified value.  More formally, returns {@code true} if and only if
 * this map contains at least one mapping to a value {@code v} such
 * that {@code (value==null ? v==null : value.equals(v))}.  This
 * operation will probably require time linear in the map size for
 * most implementations.
 *
 * @param value value whose presence in this map is to be tested
 * @return {@code true} if a mapping to {@code value} exists;
 *         {@code false} otherwise
 * @since 1.2
 */
public boolean containsValue(Object value) {
    for (Entry<K,V> e = getFirstEntry(); e != null; e = successor(e))
        if (valEquals(value, e.value))
            return true;
    return false;
}
```

查找 Map 中是否存在某个 value

由于 Map 按 key 排序，与 value 的值毛关系都没有

没办法，你就线性搜索吧

---

```java
/**
 * @throws NoSuchElementException {@inheritDoc}
 */
public K firstKey() {
    return key(getFirstEntry());
}

/**
 * @throws NoSuchElementException {@inheritDoc}
 */
public K lastKey() {
    return key(getLastEntry());
}

/**
 * Returns the key corresponding to the specified Entry.
 * @throws NoSuchElementException if the Entry is null
 */
static <K> K key(Entry<K,?> e) {
    if (e==null)
        throw new NoSuchElementException();
    return e.key;
}
```

```java
/**
 * Returns the first Entry in the TreeMap (according to the TreeMap's
 * key-sort function).  Returns null if the TreeMap is empty.
 */
final Entry<K,V> getFirstEntry() {
    Entry<K,V> p = root;
    if (p != null)
        while (p.left != null)
            p = p.left;
    return p;
}

/**
 * Returns the last Entry in the TreeMap (according to the TreeMap's
 * key-sort function).  Returns null if the TreeMap is empty.
 */
final Entry<K,V> getLastEntry() {
    Entry<K,V> p = root;
    if (p != null)
        while (p.right != null)
            p = p.right;
    return p;
}
```

遍历至最左下和最右下结点

分别取得第一个和最后一个 entry 或 key

---

```java
/**
 * Copies all of the mappings from the specified map to this map.
 * These mappings replace any mappings that this map had for any
 * of the keys currently in the specified map.
 *
 * @param  map mappings to be stored in this map
 * @throws ClassCastException if the class of a key or value in
 *         the specified map prevents it from being stored in this map
 * @throws NullPointerException if the specified map is null or
 *         the specified map contains a null key and this map does not
 *         permit null keys
 */
public void putAll(Map<? extends K, ? extends V> map) {
    int mapSize = map.size();
    if (size==0 && mapSize!=0 && map instanceof SortedMap) {
        Comparator<?> c = ((SortedMap<?,?>)map).comparator();
        if (c == comparator || (c != null && c.equals(comparator))) {
            ++modCount;
            try {
                buildFromSorted(mapSize, map.entrySet().iterator(),
                                null, null);
            } catch (java.io.IOException cannotHappen) {
            } catch (ClassNotFoundException cannotHappen) {
            }
            return;
        }
    }
    super.putAll(map);
}
```

将指定的 Map 中的所有元素拷贝到现在的 Map 中

并替换 Map 中已经出现的 entry

其中，对 `Comparator` 相同的 SortedMap 进行了特殊处理

> ？为啥呢

---

```java
/**
 * Associates the specified value with the specified key in this map.
 * If the map previously contained a mapping for the key, the old
 * value is replaced.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 *
 * @return the previous value associated with {@code key}, or
 *         {@code null} if there was no mapping for {@code key}.
 *         (A {@code null} return can also indicate that the map
 *         previously associated {@code null} with {@code key}.)
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        compare(key, key); // type (and possibly null) check

        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

插入操作

* 首先定位到 root 结点
    * 如果 root 结点为空，那么直接建立新结点作为 root 并返回 `null`
* 根据 `Comparator` 是否为 `null`，进行两种不同版本的比较
    * 用各自的比较函数进行比较，进入左右子树
    * 如果已经找到对应的 key，则用 `setValue()` 将原值替换
    * 如果因为 `null` 而停止
        * 建立新的结点
        * 并根据停止条件，插入为 parent 结点的左子树或右子树
        * 调 `fixAfterInsertion()` 红黑树合法性调整

---

```java
/**
 * Removes the mapping for this key from this TreeMap if present.
 *
 * @param  key key for which mapping should be removed
 * @return the previous value associated with {@code key}, or
 *         {@code null} if there was no mapping for {@code key}.
 *         (A {@code null} return can also indicate that the map
 *         previously associated {@code null} with {@code key}.)
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 */
public V remove(Object key) {
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    deleteEntry(p);
    return oldValue;
}
```

删除操作

首先根据 key 找到对应的 entry

* 如果不存在该 entry，直接返回皆大欢喜
* 如果存在，那么用一个变量暂存 value 用于返回，然后调 `deleteEntry()` 删除 entry
    * 在 `deleteEntry()` 中会调用 `fixAfterDeletion()` 调整红黑树合法性
* 返回被删除的 entry 中的 value

---

```java
/**
 * Removes all of the mappings from this map.
 * The map will be empty after this call returns.
 */
public void clear() {
    modCount++;
    size = 0;
    root = null;
}
```

清空映射

---

```java
/**
 * Returns a shallow copy of this {@code TreeMap} instance. (The keys and
 * values themselves are not cloned.)
 *
 * @return a shallow copy of this map
 */
public Object clone() {
    TreeMap<?,?> clone;
    try {
        clone = (TreeMap<?,?>) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }

    // Put clone into "virgin" state (except for comparator)
    clone.root = null;
    clone.size = 0;
    clone.modCount = 0;
    clone.entrySet = null;
    clone.navigableKeySet = null;
    clone.descendingMap = null;

    // Initialize clone with our mappings
    try {
        clone.buildFromSorted(size, entrySet().iterator(), null, null);
    } catch (java.io.IOException cannotHappen) {
    } catch (ClassNotFoundException cannotHappen) {
    }

    return clone;
}
```

浅拷贝一份 TreeMap

* 实例化一个处女状态的 TreeMap
* 用现有的 entry 重新构建一个 TreeMap 并返回

---

```java
/**
 * @since 1.6
 */
public Map.Entry<K,V> pollFirstEntry() {
    Entry<K,V> p = getFirstEntry();
    Map.Entry<K,V> result = exportEntry(p);
    if (p != null)
        deleteEntry(p);
    return result;
}

/**
 * @since 1.6
 */
public Map.Entry<K,V> pollLastEntry() {
    Entry<K,V> p = getLastEntry();
    Map.Entry<K,V> result = exportEntry(p);
    if (p != null)
        deleteEntry(p);
    return result;
}
```

删除第一个或最后一个 entry

* 首先找到第一个或最后一个 entry
* 调 `deleteEntry()` 删除，并自动调整红黑树

---

```java
/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @since 1.6
 */
public Map.Entry<K,V> lowerEntry(K key) {
    return exportEntry(getLowerEntry(key));
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @since 1.6
 */
public K lowerKey(K key) {
    return keyOrNull(getLowerEntry(key));
}
```

```java
/**
 * Returns the entry for the greatest key less than the specified key; if
 * no such entry exists (i.e., the least key in the Tree is greater than
 * the specified key), returns {@code null}.
 */
final Entry<K,V> getLowerEntry(K key) {
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = compare(key, p.key);
        if (cmp > 0) {
            if (p.right != null)
                p = p.right;
            else
                return p;
        } else {
            if (p.left != null) {
                p = p.left;
            } else {
                Entry<K,V> parent = p.parent;
                Entry<K,V> ch = p;
                while (parent != null && ch == parent.left) {
                    ch = parent;
                    parent = parent.parent;
                }
                return parent;
            }
        }
    }
    return null;
}
```

返回小于给定 key 的最大 entry 及其 key

从 root 结点开始搜索

* 如果 key 已经大于当前结点
    * 需要往当前结点的右子树寻找 (尽可能接近 key 的结点)
    * 如果没有右子树
        * 该结点就是小于 key 的最大结点，返回
* 如果 key 小于当前结点
    * 需要往当前结点的左子树寻找 (尽可能接近 key 的结点)
    * 如果没有左子树
        * 回溯，直到找到一个有左子树的祖先 (祖先肯定比 key 小，不然不会进行到当前层)
        * 该祖先就是小于 key 且最大的结点，返回
* 不然就返回 `null`

---

```java
/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @since 1.6
 */
public Map.Entry<K,V> floorEntry(K key) {
    return exportEntry(getFloorEntry(key));
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @since 1.6
 */
public K floorKey(K key) {
    return keyOrNull(getFloorEntry(key));
}
```

```java
/**
 * Gets the entry corresponding to the specified key; if no such entry
 * exists, returns the entry for the greatest key less than the specified
 * key; if no such entry exists, returns {@code null}.
 */
final Entry<K,V> getFloorEntry(K key) {
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = compare(key, p.key);
        if (cmp > 0) {
            if (p.right != null)
                p = p.right;
            else
                return p;
        } else if (cmp < 0) {
            if (p.left != null) {
                p = p.left;
            } else {
                Entry<K,V> parent = p.parent;
                Entry<K,V> ch = p;
                while (parent != null && ch == parent.left) {
                    ch = parent;
                    parent = parent.parent;
                }
                return parent;
            }
        } else
            return p;

    }
    return null;
}
```

返回指定 key 的 entry

如果不存在，就返回小于 key 的最大 entry

与前一个函数的差别在于在进入左右子树时，如果 key 相等就直接返回

否则就还是寻找比 key 小的最大 entry

---

```java
/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @since 1.6
 */
public Map.Entry<K,V> higherEntry(K key) {
    return exportEntry(getHigherEntry(key));
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @since 1.6
 */
public K higherKey(K key) {
    return keyOrNull(getHigherEntry(key));
}
```


```java
/**
 * Gets the entry for the least key greater than the specified
 * key; if no such entry exists, returns the entry for the least
 * key greater than the specified key; if no such entry exists
 * returns {@code null}.
 */
final Entry<K,V> getHigherEntry(K key) {
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = compare(key, p.key);
        if (cmp < 0) {
            if (p.left != null)
                p = p.left;
            else
                return p;
        } else {
            if (p.right != null) {
                p = p.right;
            } else {
                Entry<K,V> parent = p.parent;
                Entry<K,V> ch = p;
                while (parent != null && ch == parent.right) {
                    ch = parent;
                    parent = parent.parent;
                }
                return parent;
            }
        }
    }
    return null;
}
```

从 root 结点开始搜索

* 若当前结点已经大于 key
    * 那么就找当前结点的左子树，以便尽可能接近 key
    * 如果当前结点没有左子树，那么当前结点就是大于 key 的最小结点，返回
* 若当前元素还小于 key
    * 就往当前结点的右子树找，以便尽可能接近 key
    * 若当前结点没有右子树了，而元素还小于 key
        * 那么就要一直回溯，直到结点有右子树为止 (这个结点肯定比 key 大，因为之前 key 进入其左子树)
        * 该结点就是大于 key 的最小结点，返回

---

```java
/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @since 1.6
 */
public Map.Entry<K,V> ceilingEntry(K key) {
    return exportEntry(getCeilingEntry(key));
}

/**
 * @throws ClassCastException {@inheritDoc}
 * @throws NullPointerException if the specified key is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @since 1.6
 */
public K ceilingKey(K key) {
    return keyOrNull(getCeilingEntry(key));
}
```

```java
/**
 * Gets the entry corresponding to the specified key; if no such entry
 * exists, returns the entry for the least key greater than the specified
 * key; if no such entry exists (i.e., the greatest key in the Tree is less
 * than the specified key), returns {@code null}.
 */
final Entry<K,V> getCeilingEntry(K key) {
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = compare(key, p.key);
        if (cmp < 0) {
            if (p.left != null)
                p = p.left;
            else
                return p;
        } else if (cmp > 0) {
            if (p.right != null) {
                p = p.right;
            } else {
                Entry<K,V> parent = p.parent;
                Entry<K,V> ch = p;
                while (parent != null && ch == parent.right) {
                    ch = parent;
                    parent = parent.parent;
                }
                return parent;
            }
        } else
            return p;
    }
    return null;
}
```

返回指定的 key 对应的 entry

如果不存在，则返回大于 key 的最小 entry

与前一个函数基本相同，就是多了个 key 与当前结点相同时直接返回

---

```java
/**
 * Fields initialized to contain an instance of the entry set view
 * the first time this view is requested.  Views are stateless, so
 * there's no reason to create more than one.
 */
private transient EntrySet entrySet;
private transient KeySet<K> navigableKeySet;
private transient NavigableMap<K,V> descendingMap;
```

集合视角

* 在内部维护这三个变量
* 懒加载，这三个变量在被请求时才会被初始化
* 这三个变量又分别在内部维护了整个集合，好像有种传说中的自引用关系... 😥

以 keySet 的维护为例：

```java
/**
 * Returns a {@link Set} view of the keys contained in this map.
 *
 * <p>The set's iterator returns the keys in ascending order.
 * The set's spliterator is
 * <em><a href="Spliterator.html#binding">late-binding</a></em>,
 * <em>fail-fast</em>, and additionally reports {@link Spliterator#SORTED}
 * and {@link Spliterator#ORDERED} with an encounter order that is ascending
 * key order.  The spliterator's comparator (see
 * {@link java.util.Spliterator#getComparator()}) is {@code null} if
 * the tree map's comparator (see {@link #comparator()}) is {@code null}.
 * Otherwise, the spliterator's comparator is the same as or imposes the
 * same total ordering as the tree map's comparator.
 *
 * <p>The set is backed by the map, so changes to the map are
 * reflected in the set, and vice-versa.  If the map is modified
 * while an iteration over the set is in progress (except through
 * the iterator's own {@code remove} operation), the results of
 * the iteration are undefined.  The set supports element removal,
 * which removes the corresponding mapping from the map, via the
 * {@code Iterator.remove}, {@code Set.remove},
 * {@code removeAll}, {@code retainAll}, and {@code clear}
 * operations.  It does not support the {@code add} or {@code addAll}
 * operations.
 */
public Set<K> keySet() {
    return navigableKeySet();
}

/**
 * @since 1.6
 */
public NavigableSet<K> navigableKeySet() {
    KeySet<K> nks = navigableKeySet;
    return (nks != null) ? nks : (navigableKeySet = new KeySet<>(this));
}

static final class KeySet<E> extends AbstractSet<E> implements NavigableSet<E> {
    private final NavigableMap<E, ?> m;
    KeySet(NavigableMap<E,?> map) { m = map; }

    public Iterator<E> iterator() {
        if (m instanceof TreeMap)
            return ((TreeMap<E,?>)m).keyIterator();
        else
            return ((TreeMap.NavigableSubMap<E,?>)m).keyIterator();
    }

    public Iterator<E> descendingIterator() {
        if (m instanceof TreeMap)
            return ((TreeMap<E,?>)m).descendingKeyIterator();
        else
            return ((TreeMap.NavigableSubMap<E,?>)m).descendingKeyIterator();
    }

    public int size() { return m.size(); }
    public boolean isEmpty() { return m.isEmpty(); }
    public boolean contains(Object o) { return m.containsKey(o); }
    public void clear() { m.clear(); }
    public E lower(E e) { return m.lowerKey(e); }
    public E floor(E e) { return m.floorKey(e); }
    public E ceiling(E e) { return m.ceilingKey(e); }
    public E higher(E e) { return m.higherKey(e); }
    public E first() { return m.firstKey(); }
    public E last() { return m.lastKey(); }
    public Comparator<? super E> comparator() { return m.comparator(); }
    public E pollFirst() {
        Map.Entry<E,?> e = m.pollFirstEntry();
        return (e == null) ? null : e.getKey();
    }
    public E pollLast() {
        Map.Entry<E,?> e = m.pollLastEntry();
        return (e == null) ? null : e.getKey();
    }
    public boolean remove(Object o) {
        int oldSize = size();
        m.remove(o);
        return size() != oldSize;
    }
    public NavigableSet<E> subSet(E fromElement, boolean fromInclusive,
                                    E toElement,   boolean toInclusive) {
        return new KeySet<>(m.subMap(fromElement, fromInclusive,
                                        toElement,   toInclusive));
    }
    public NavigableSet<E> headSet(E toElement, boolean inclusive) {
        return new KeySet<>(m.headMap(toElement, inclusive));
    }
    public NavigableSet<E> tailSet(E fromElement, boolean inclusive) {
        return new KeySet<>(m.tailMap(fromElement, inclusive));
    }
    public SortedSet<E> subSet(E fromElement, E toElement) {
        return subSet(fromElement, true, toElement, false);
    }
    public SortedSet<E> headSet(E toElement) {
        return headSet(toElement, false);
    }
    public SortedSet<E> tailSet(E fromElement) {
        return tailSet(fromElement, true);
    }
    public NavigableSet<E> descendingSet() {
        return new KeySet<>(m.descendingMap());
    }

    public Spliterator<E> spliterator() {
        return keySpliteratorFor(m);
    }
}
```

所以，对集合的操作也会体现在 keySet 上

对 keySet 的操作也会体现在集合上

* 在 keySet 上的操作都使用集合的函数实现

对 value 集合的维护：

```java
/**
 * Returns a {@link Collection} view of the values contained in this map.
 *
 * <p>The collection's iterator returns the values in ascending order
 * of the corresponding keys. The collection's spliterator is
 * <em><a href="Spliterator.html#binding">late-binding</a></em>,
 * <em>fail-fast</em>, and additionally reports {@link Spliterator#ORDERED}
 * with an encounter order that is ascending order of the corresponding
 * keys.
 *
 * <p>The collection is backed by the map, so changes to the map are
 * reflected in the collection, and vice-versa.  If the map is
 * modified while an iteration over the collection is in progress
 * (except through the iterator's own {@code remove} operation),
 * the results of the iteration are undefined.  The collection
 * supports element removal, which removes the corresponding
 * mapping from the map, via the {@code Iterator.remove},
 * {@code Collection.remove}, {@code removeAll},
 * {@code retainAll} and {@code clear} operations.  It does not
 * support the {@code add} or {@code addAll} operations.
 */
public Collection<V> values() {
    Collection<V> vs = values;
    if (vs == null) {
        vs = new Values();
        values = vs;
    }
    return vs;
}

class Values extends AbstractCollection<V> {
    public Iterator<V> iterator() {
        return new ValueIterator(getFirstEntry());
    }

    public int size() {
        return TreeMap.this.size();
    }

    public boolean contains(Object o) {
        return TreeMap.this.containsValue(o);
    }

    public boolean remove(Object o) {
        for (Entry<K,V> e = getFirstEntry(); e != null; e = successor(e)) {
            if (valEquals(e.getValue(), o)) {
                deleteEntry(e);
                return true;
            }
        }
        return false;
    }

    public void clear() {
        TreeMap.this.clear();
    }

    public Spliterator<V> spliterator() {
        return new ValueSpliterator<K,V>(TreeMap.this, null, null, 0, -1, 0);
    }
}
```

对于 entrySet 的维护：

```java
/**
 * Returns a {@link Set} view of the mappings contained in this map.
 *
 * <p>The set's iterator returns the entries in ascending key order. The
 * sets's spliterator is
 * <em><a href="Spliterator.html#binding">late-binding</a></em>,
 * <em>fail-fast</em>, and additionally reports {@link Spliterator#SORTED} and
 * {@link Spliterator#ORDERED} with an encounter order that is ascending key
 * order.
 *
 * <p>The set is backed by the map, so changes to the map are
 * reflected in the set, and vice-versa.  If the map is modified
 * while an iteration over the set is in progress (except through
 * the iterator's own {@code remove} operation, or through the
 * {@code setValue} operation on a map entry returned by the
 * iterator) the results of the iteration are undefined.  The set
 * supports element removal, which removes the corresponding
 * mapping from the map, via the {@code Iterator.remove},
 * {@code Set.remove}, {@code removeAll}, {@code retainAll} and
 * {@code clear} operations.  It does not support the
 * {@code add} or {@code addAll} operations.
 */
public Set<Map.Entry<K,V>> entrySet() {
    EntrySet es = entrySet;
    return (es != null) ? es : (entrySet = new EntrySet());
}

class EntrySet extends AbstractSet<Map.Entry<K,V>> {
    public Iterator<Map.Entry<K,V>> iterator() {
        return new EntryIterator(getFirstEntry());
    }

    public boolean contains(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> entry = (Map.Entry<?,?>) o;
        Object value = entry.getValue();
        Entry<K,V> p = getEntry(entry.getKey());
        return p != null && valEquals(p.getValue(), value);
    }

    public boolean remove(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> entry = (Map.Entry<?,?>) o;
        Object value = entry.getValue();
        Entry<K,V> p = getEntry(entry.getKey());
        if (p != null && valEquals(p.getValue(), value)) {
            deleteEntry(p);
            return true;
        }
        return false;
    }

    public int size() {
        return TreeMap.this.size();
    }

    public void clear() {
        TreeMap.this.clear();
    }

    public Spliterator<Map.Entry<K,V>> spliterator() {
        return new EntrySpliterator<K,V>(TreeMap.this, null, null, 0, -1, 0);
    }
}
```

> 不知道理解得到不到位
>
> 这些类的存在，只是为了给用户一种视角
>
> 有只关心 key 的视角，有只关心 value 的视角，也有只关心 entry 的视角
>
> 而针对视角所做的操作，都由 Map 提供的函数在内部加以实现，这部分对用户透明
>
> 不过自己引用自己确实要给绕晕了...... 😫

---

```java
@Override
public boolean replace(K key, V oldValue, V newValue) {
    Entry<K,V> p = getEntry(key);
    if (p!=null && Objects.equals(oldValue, p.value)) {
        p.value = newValue;
        return true;
    }
    return false;
}

@Override
public V replace(K key, V value) {
    Entry<K,V> p = getEntry(key);
    if (p!=null) {
        V oldValue = p.value;
        p.value = value;
        return oldValue;
    }
    return null;
}
```

替换操作，底层由 `getEntry()` 支持

先找到对应的 entry，然后判断条件并替换

---

```java
@Override
public void forEach(BiConsumer<? super K, ? super V> action) {
    Objects.requireNonNull(action);
    int expectedModCount = modCount;
    for (Entry<K, V> e = getFirstEntry(); e != null; e = successor(e)) {
        action.accept(e.key, e.value);

        if (expectedModCount != modCount) {
            throw new ConcurrentModificationException();
        }
    }
}

@Override
public void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
    Objects.requireNonNull(function);
    int expectedModCount = modCount;

    for (Entry<K, V> e = getFirstEntry(); e != null; e = successor(e)) {
        e.value = function.apply(e.key, e.value);

        if (expectedModCount != modCount) {
            throw new ConcurrentModificationException();
        }
    }
}
```

遍历每一个 entry，并应用对应的操作

---

```java
/**
 * @throws ClassCastException       {@inheritDoc}
 * @throws NullPointerException if {@code fromKey} or {@code toKey} is
 *         null and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @throws IllegalArgumentException {@inheritDoc}
 * @since 1.6
 */
public NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                                K toKey,   boolean toInclusive) {
    return new AscendingSubMap<>(this,
                                    false, fromKey, fromInclusive,
                                    false, toKey,   toInclusive);
}

/**
 * @throws ClassCastException       {@inheritDoc}
 * @throws NullPointerException if {@code toKey} is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @throws IllegalArgumentException {@inheritDoc}
 * @since 1.6
 */
public NavigableMap<K,V> headMap(K toKey, boolean inclusive) {
    return new AscendingSubMap<>(this,
                                    true,  null,  true,
                                    false, toKey, inclusive);
}

/**
 * @throws ClassCastException       {@inheritDoc}
 * @throws NullPointerException if {@code fromKey} is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @throws IllegalArgumentException {@inheritDoc}
 * @since 1.6
 */
public NavigableMap<K,V> tailMap(K fromKey, boolean inclusive) {
    return new AscendingSubMap<>(this,
                                    false, fromKey, inclusive,
                                    true,  null,    true);
}

/**
 * @throws ClassCastException       {@inheritDoc}
 * @throws NullPointerException if {@code fromKey} or {@code toKey} is
 *         null and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @throws IllegalArgumentException {@inheritDoc}
 */
public SortedMap<K,V> subMap(K fromKey, K toKey) {
    return subMap(fromKey, true, toKey, false);
}

/**
 * @throws ClassCastException       {@inheritDoc}
 * @throws NullPointerException if {@code toKey} is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @throws IllegalArgumentException {@inheritDoc}
 */
public SortedMap<K,V> headMap(K toKey) {
    return headMap(toKey, false);
}

/**
 * @throws ClassCastException       {@inheritDoc}
 * @throws NullPointerException if {@code fromKey} is null
 *         and this map uses natural ordering, or its comparator
 *         does not permit null keys
 * @throws IllegalArgumentException {@inheritDoc}
 */
public SortedMap<K,V> tailMap(K fromKey) {
    return tailMap(fromKey, true);
}
```

取子集的函数

* 由于集合有序，只需要指明 __边界__ 和 __是否包含边界__
* 子集内部维护集合本身，也是自引用，但需要额外维护 __子集范围__

```java
/**
 * @serial include
 */
abstract static class NavigableSubMap<K,V> extends AbstractMap<K,V>
    implements NavigableMap<K,V>, java.io.Serializable {
    private static final long serialVersionUID = -2102997345730753016L;
    /**
     * The backing map.
     */
    final TreeMap<K,V> m;

    /**
     * Endpoints are represented as triples (fromStart, lo,
     * loInclusive) and (toEnd, hi, hiInclusive). If fromStart is
     * true, then the low (absolute) bound is the start of the
     * backing map, and the other values are ignored. Otherwise,
     * if loInclusive is true, lo is the inclusive bound, else lo
     * is the exclusive bound. Similarly for the upper bound.
     */
    final K lo, hi;
    final boolean fromStart, toEnd;
    final boolean loInclusive, hiInclusive;

    NavigableSubMap(TreeMap<K,V> m,
                    boolean fromStart, K lo, boolean loInclusive,
                    boolean toEnd,     K hi, boolean hiInclusive) {
        if (!fromStart && !toEnd) {
            if (m.compare(lo, hi) > 0)
                throw new IllegalArgumentException("fromKey > toKey");
        } else {
            if (!fromStart) // type check
                m.compare(lo, lo);
            if (!toEnd)
                m.compare(hi, hi);
        }

        this.m = m;
        this.fromStart = fromStart;
        this.lo = lo;
        this.loInclusive = loInclusive;
        this.toEnd = toEnd;
        this.hi = hi;
        this.hiInclusive = hiInclusive;
    }

    // internal utilities

    final boolean tooLow(Object key) {
        if (!fromStart) {
            int c = m.compare(key, lo);
            if (c < 0 || (c == 0 && !loInclusive))
                return true;
        }
        return false;
    }

    final boolean tooHigh(Object key) {
        if (!toEnd) {
            int c = m.compare(key, hi);
            if (c > 0 || (c == 0 && !hiInclusive))
                return true;
        }
        return false;
    }

    final boolean inRange(Object key) {
        return !tooLow(key) && !tooHigh(key);
    }

    final boolean inClosedRange(Object key) {
        return (fromStart || m.compare(key, lo) >= 0)
            && (toEnd || m.compare(hi, key) >= 0);
    }

    final boolean inRange(Object key, boolean inclusive) {
        return inclusive ? inRange(key) : inClosedRange(key);
    }

    /*
     * Absolute versions of relation operations.
     * Subclasses map to these using like-named "sub"
     * versions that invert senses for descending maps
     */

    final TreeMap.Entry<K,V> absLowest() {
        TreeMap.Entry<K,V> e =
            (fromStart ?  m.getFirstEntry() :
                (loInclusive ? m.getCeilingEntry(lo) :
                            m.getHigherEntry(lo)));
        return (e == null || tooHigh(e.key)) ? null : e;
    }

    final TreeMap.Entry<K,V> absHighest() {
        TreeMap.Entry<K,V> e =
            (toEnd ?  m.getLastEntry() :
                (hiInclusive ?  m.getFloorEntry(hi) :
                                m.getLowerEntry(hi)));
        return (e == null || tooLow(e.key)) ? null : e;
    }

    final TreeMap.Entry<K,V> absCeiling(K key) {
        if (tooLow(key))
            return absLowest();
        TreeMap.Entry<K,V> e = m.getCeilingEntry(key);
        return (e == null || tooHigh(e.key)) ? null : e;
    }

    final TreeMap.Entry<K,V> absHigher(K key) {
        if (tooLow(key))
            return absLowest();
        TreeMap.Entry<K,V> e = m.getHigherEntry(key);
        return (e == null || tooHigh(e.key)) ? null : e;
    }

    final TreeMap.Entry<K,V> absFloor(K key) {
        if (tooHigh(key))
            return absHighest();
        TreeMap.Entry<K,V> e = m.getFloorEntry(key);
        return (e == null || tooLow(e.key)) ? null : e;
    }

    final TreeMap.Entry<K,V> absLower(K key) {
        if (tooHigh(key))
            return absHighest();
        TreeMap.Entry<K,V> e = m.getLowerEntry(key);
        return (e == null || tooLow(e.key)) ? null : e;
    }

    /** Returns the absolute high fence for ascending traversal */
    final TreeMap.Entry<K,V> absHighFence() {
        return (toEnd ? null : (hiInclusive ?
                                m.getHigherEntry(hi) :
                                m.getCeilingEntry(hi)));
    }

    /** Return the absolute low fence for descending traversal  */
    final TreeMap.Entry<K,V> absLowFence() {
        return (fromStart ? null : (loInclusive ?
                                    m.getLowerEntry(lo) :
                                    m.getFloorEntry(lo)));
    }

    // Abstract methods defined in ascending vs descending classes
    // These relay to the appropriate absolute versions

    abstract TreeMap.Entry<K,V> subLowest();
    abstract TreeMap.Entry<K,V> subHighest();
    abstract TreeMap.Entry<K,V> subCeiling(K key);
    abstract TreeMap.Entry<K,V> subHigher(K key);
    abstract TreeMap.Entry<K,V> subFloor(K key);
    abstract TreeMap.Entry<K,V> subLower(K key);

    /** Returns ascending iterator from the perspective of this submap */
    abstract Iterator<K> keyIterator();

    abstract Spliterator<K> keySpliterator();

    /** Returns descending iterator from the perspective of this submap */
    abstract Iterator<K> descendingKeyIterator();

    // public methods

    public boolean isEmpty() {
        return (fromStart && toEnd) ? m.isEmpty() : entrySet().isEmpty();
    }

    public int size() {
        return (fromStart && toEnd) ? m.size() : entrySet().size();
    }

    public final boolean containsKey(Object key) {
        return inRange(key) && m.containsKey(key);
    }

    public final V put(K key, V value) {
        if (!inRange(key))
            throw new IllegalArgumentException("key out of range");
        return m.put(key, value);
    }

    public final V get(Object key) {
        return !inRange(key) ? null :  m.get(key);
    }

    public final V remove(Object key) {
        return !inRange(key) ? null : m.remove(key);
    }

    public final Map.Entry<K,V> ceilingEntry(K key) {
        return exportEntry(subCeiling(key));
    }

    public final K ceilingKey(K key) {
        return keyOrNull(subCeiling(key));
    }

    public final Map.Entry<K,V> higherEntry(K key) {
        return exportEntry(subHigher(key));
    }

    public final K higherKey(K key) {
        return keyOrNull(subHigher(key));
    }

    public final Map.Entry<K,V> floorEntry(K key) {
        return exportEntry(subFloor(key));
    }

    public final K floorKey(K key) {
        return keyOrNull(subFloor(key));
    }

    public final Map.Entry<K,V> lowerEntry(K key) {
        return exportEntry(subLower(key));
    }

    public final K lowerKey(K key) {
        return keyOrNull(subLower(key));
    }

    public final K firstKey() {
        return key(subLowest());
    }

    public final K lastKey() {
        return key(subHighest());
    }

    public final Map.Entry<K,V> firstEntry() {
        return exportEntry(subLowest());
    }

    public final Map.Entry<K,V> lastEntry() {
        return exportEntry(subHighest());
    }

    public final Map.Entry<K,V> pollFirstEntry() {
        TreeMap.Entry<K,V> e = subLowest();
        Map.Entry<K,V> result = exportEntry(e);
        if (e != null)
            m.deleteEntry(e);
        return result;
    }

    public final Map.Entry<K,V> pollLastEntry() {
        TreeMap.Entry<K,V> e = subHighest();
        Map.Entry<K,V> result = exportEntry(e);
        if (e != null)
            m.deleteEntry(e);
        return result;
    }

    // Views
    transient NavigableMap<K,V> descendingMapView;
    transient EntrySetView entrySetView;
    transient KeySet<K> navigableKeySetView;

    public final NavigableSet<K> navigableKeySet() {
        KeySet<K> nksv = navigableKeySetView;
        return (nksv != null) ? nksv :
            (navigableKeySetView = new TreeMap.KeySet<>(this));
    }

    public final Set<K> keySet() {
        return navigableKeySet();
    }

    public NavigableSet<K> descendingKeySet() {
        return descendingMap().navigableKeySet();
    }

    public final SortedMap<K,V> subMap(K fromKey, K toKey) {
        return subMap(fromKey, true, toKey, false);
    }

    public final SortedMap<K,V> headMap(K toKey) {
        return headMap(toKey, false);
    }

    public final SortedMap<K,V> tailMap(K fromKey) {
        return tailMap(fromKey, true);
    }

    // View classes

    abstract class EntrySetView extends AbstractSet<Map.Entry<K,V>> {
        private transient int size = -1, sizeModCount;

        public int size() {
            if (fromStart && toEnd)
                return m.size();
            if (size == -1 || sizeModCount != m.modCount) {
                sizeModCount = m.modCount;
                size = 0;
                Iterator<?> i = iterator();
                while (i.hasNext()) {
                    size++;
                    i.next();
                }
            }
            return size;
        }

        public boolean isEmpty() {
            TreeMap.Entry<K,V> n = absLowest();
            return n == null || tooHigh(n.key);
        }

        public boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> entry = (Map.Entry<?,?>) o;
            Object key = entry.getKey();
            if (!inRange(key))
                return false;
            TreeMap.Entry<?,?> node = m.getEntry(key);
            return node != null &&
                valEquals(node.getValue(), entry.getValue());
        }

        public boolean remove(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> entry = (Map.Entry<?,?>) o;
            Object key = entry.getKey();
            if (!inRange(key))
                return false;
            TreeMap.Entry<K,V> node = m.getEntry(key);
            if (node!=null && valEquals(node.getValue(),
                                        entry.getValue())) {
                m.deleteEntry(node);
                return true;
            }
            return false;
        }
    }

    /**
     * Iterators for SubMaps
     */
    abstract class SubMapIterator<T> implements Iterator<T> {
        TreeMap.Entry<K,V> lastReturned;
        TreeMap.Entry<K,V> next;
        final Object fenceKey;
        int expectedModCount;

        SubMapIterator(TreeMap.Entry<K,V> first,
                        TreeMap.Entry<K,V> fence) {
            expectedModCount = m.modCount;
            lastReturned = null;
            next = first;
            fenceKey = fence == null ? UNBOUNDED : fence.key;
        }

        public final boolean hasNext() {
            return next != null && next.key != fenceKey;
        }

        final TreeMap.Entry<K,V> nextEntry() {
            TreeMap.Entry<K,V> e = next;
            if (e == null || e.key == fenceKey)
                throw new NoSuchElementException();
            if (m.modCount != expectedModCount)
                throw new ConcurrentModificationException();
            next = successor(e);
            lastReturned = e;
            return e;
        }

        final TreeMap.Entry<K,V> prevEntry() {
            TreeMap.Entry<K,V> e = next;
            if (e == null || e.key == fenceKey)
                throw new NoSuchElementException();
            if (m.modCount != expectedModCount)
                throw new ConcurrentModificationException();
            next = predecessor(e);
            lastReturned = e;
            return e;
        }

        final void removeAscending() {
            if (lastReturned == null)
                throw new IllegalStateException();
            if (m.modCount != expectedModCount)
                throw new ConcurrentModificationException();
            // deleted entries are replaced by their successors
            if (lastReturned.left != null && lastReturned.right != null)
                next = lastReturned;
            m.deleteEntry(lastReturned);
            lastReturned = null;
            expectedModCount = m.modCount;
        }

        final void removeDescending() {
            if (lastReturned == null)
                throw new IllegalStateException();
            if (m.modCount != expectedModCount)
                throw new ConcurrentModificationException();
            m.deleteEntry(lastReturned);
            lastReturned = null;
            expectedModCount = m.modCount;
        }

    }

    final class SubMapEntryIterator extends SubMapIterator<Map.Entry<K,V>> {
        SubMapEntryIterator(TreeMap.Entry<K,V> first,
                            TreeMap.Entry<K,V> fence) {
            super(first, fence);
        }
        public Map.Entry<K,V> next() {
            return nextEntry();
        }
        public void remove() {
            removeAscending();
        }
    }

    final class DescendingSubMapEntryIterator extends SubMapIterator<Map.Entry<K,V>> {
        DescendingSubMapEntryIterator(TreeMap.Entry<K,V> last,
                                        TreeMap.Entry<K,V> fence) {
            super(last, fence);
        }

        public Map.Entry<K,V> next() {
            return prevEntry();
        }
        public void remove() {
            removeDescending();
        }
    }

    // Implement minimal Spliterator as KeySpliterator backup
    final class SubMapKeyIterator extends SubMapIterator<K>
        implements Spliterator<K> {
        SubMapKeyIterator(TreeMap.Entry<K,V> first,
                            TreeMap.Entry<K,V> fence) {
            super(first, fence);
        }
        public K next() {
            return nextEntry().key;
        }
        public void remove() {
            removeAscending();
        }
        public Spliterator<K> trySplit() {
            return null;
        }
        public void forEachRemaining(Consumer<? super K> action) {
            while (hasNext())
                action.accept(next());
        }
        public boolean tryAdvance(Consumer<? super K> action) {
            if (hasNext()) {
                action.accept(next());
                return true;
            }
            return false;
        }
        public long estimateSize() {
            return Long.MAX_VALUE;
        }
        public int characteristics() {
            return Spliterator.DISTINCT | Spliterator.ORDERED |
                Spliterator.SORTED;
        }
        public final Comparator<? super K>  getComparator() {
            return NavigableSubMap.this.comparator();
        }
    }

    final class DescendingSubMapKeyIterator extends SubMapIterator<K>
        implements Spliterator<K> {
        DescendingSubMapKeyIterator(TreeMap.Entry<K,V> last,
                                    TreeMap.Entry<K,V> fence) {
            super(last, fence);
        }
        public K next() {
            return prevEntry().key;
        }
        public void remove() {
            removeDescending();
        }
        public Spliterator<K> trySplit() {
            return null;
        }
        public void forEachRemaining(Consumer<? super K> action) {
            while (hasNext())
                action.accept(next());
        }
        public boolean tryAdvance(Consumer<? super K> action) {
            if (hasNext()) {
                action.accept(next());
                return true;
            }
            return false;
        }
        public long estimateSize() {
            return Long.MAX_VALUE;
        }
        public int characteristics() {
            return Spliterator.DISTINCT | Spliterator.ORDERED;
        }
    }
}
```

从该类派生出两个实现类，分别实现正序子集和反序子集：

* `static final class AscendingSubMap<K,V> extends NavigableSubMap<K,V> {}`
* `static final class DescendingSubMap<K,V> extends NavigableSubMap<K,V> {}`

---

## Summary

em...够复杂的人要晕了

关于集合视角也好、正反视角也好，其实都是封装

最主要的还是要把握红黑树这一核心实现思想

只需要知道内部有一颗增删改查性能都为 `O(log(n))` 的 BST 即可

在这一前提下，实现所有的基本操作也就是对 BST 进行增删改查、遍历咯

---

