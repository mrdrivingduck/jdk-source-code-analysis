# Interface - java.util.Map

Created by : Mr Dk.

2019 / 11 / 13 10:02

Nanjing, Jiangsu, China

---

## Definition

```java
public interface Map<K,V> {

}
```

建立从 key 到 value 的映射：map 中不能存在重复的 key，每个 key 只能最多对应一个值。

Map 接口提供了三个集合视角：

- key 的集合
- value 的集合
- key-value 映射的集合

Map 的顺序被定义为迭代器返回元素的顺序：

- TreeMap 保证特定的顺序
- HashMap 不保证顺序

需要注意的是可变对象作为 key 的情况：如果对象变化导致 `equals()` 和 `hashcode()` 的行为发生变化，map 的行为未知。

所有的 Map 实现都应当提供 **空构造函数** 和 **拷贝构造函数**。不同的集合实现对于 `null` key 等有各自的限制。

```java
/**
 * An object that maps keys to values.  A map cannot contain duplicate keys;
 * each key can map to at most one value.
 *
 * <p>This interface takes the place of the <tt>Dictionary</tt> class, which
 * was a totally abstract class rather than an interface.
 *
 * <p>The <tt>Map</tt> interface provides three <i>collection views</i>, which
 * allow a map's contents to be viewed as a set of keys, collection of values,
 * or set of key-value mappings.  The <i>order</i> of a map is defined as
 * the order in which the iterators on the map's collection views return their
 * elements.  Some map implementations, like the <tt>TreeMap</tt> class, make
 * specific guarantees as to their order; others, like the <tt>HashMap</tt>
 * class, do not.
 *
 * <p>Note: great care must be exercised if mutable objects are used as map
 * keys.  The behavior of a map is not specified if the value of an object is
 * changed in a manner that affects <tt>equals</tt> comparisons while the
 * object is a key in the map.  A special case of this prohibition is that it
 * is not permissible for a map to contain itself as a key.  While it is
 * permissible for a map to contain itself as a value, extreme caution is
 * advised: the <tt>equals</tt> and <tt>hashCode</tt> methods are no longer
 * well defined on such a map.
 *
 * <p>All general-purpose map implementation classes should provide two
 * "standard" constructors: a void (no arguments) constructor which creates an
 * empty map, and a constructor with a single argument of type <tt>Map</tt>,
 * which creates a new map with the same key-value mappings as its argument.
 * In effect, the latter constructor allows the user to copy any map,
 * producing an equivalent map of the desired class.  There is no way to
 * enforce this recommendation (as interfaces cannot contain constructors) but
 * all of the general-purpose map implementations in the JDK comply.
 *
 * <p>The "destructive" methods contained in this interface, that is, the
 * methods that modify the map on which they operate, are specified to throw
 * <tt>UnsupportedOperationException</tt> if this map does not support the
 * operation.  If this is the case, these methods may, but are not required
 * to, throw an <tt>UnsupportedOperationException</tt> if the invocation would
 * have no effect on the map.  For example, invoking the {@link #putAll(Map)}
 * method on an unmodifiable map may, but is not required to, throw the
 * exception if the map whose mappings are to be "superimposed" is empty.
 *
 * <p>Some map implementations have restrictions on the keys and values they
 * may contain.  For example, some implementations prohibit null keys and
 * values, and some have restrictions on the types of their keys.  Attempting
 * to insert an ineligible key or value throws an unchecked exception,
 * typically <tt>NullPointerException</tt> or <tt>ClassCastException</tt>.
 * Attempting to query the presence of an ineligible key or value may throw an
 * exception, or it may simply return false; some implementations will exhibit
 * the former behavior and some will exhibit the latter.  More generally,
 * attempting an operation on an ineligible key or value whose completion
 * would not result in the insertion of an ineligible element into the map may
 * throw an exception or it may succeed, at the option of the implementation.
 * Such exceptions are marked as "optional" in the specification for this
 * interface.
 *
 * <p>Many methods in Collections Framework interfaces are defined
 * in terms of the {@link Object#equals(Object) equals} method.  For
 * example, the specification for the {@link #containsKey(Object)
 * containsKey(Object key)} method says: "returns <tt>true</tt> if and
 * only if this map contains a mapping for a key <tt>k</tt> such that
 * <tt>(key==null ? k==null : key.equals(k))</tt>." This specification should
 * <i>not</i> be construed to imply that invoking <tt>Map.containsKey</tt>
 * with a non-null argument <tt>key</tt> will cause <tt>key.equals(k)</tt> to
 * be invoked for any key <tt>k</tt>.  Implementations are free to
 * implement optimizations whereby the <tt>equals</tt> invocation is avoided,
 * for example, by first comparing the hash codes of the two keys.  (The
 * {@link Object#hashCode()} specification guarantees that two objects with
 * unequal hash codes cannot be equal.)  More generally, implementations of
 * the various Collections Framework interfaces are free to take advantage of
 * the specified behavior of underlying {@link Object} methods wherever the
 * implementor deems it appropriate.
 *
 * <p>Some map operations which perform recursive traversal of the map may fail
 * with an exception for self-referential instances where the map directly or
 * indirectly contains itself. This includes the {@code clone()},
 * {@code equals()}, {@code hashCode()} and {@code toString()} methods.
 * Implementations may optionally handle the self-referential scenario, however
 * most current implementations do not do so.
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 *
 * @author  Josh Bloch
 * @see HashMap
 * @see TreeMap
 * @see Hashtable
 * @see SortedMap
 * @see Collection
 * @see Set
 * @since 1.2
 */
```

## Size

```java
/**
 * Returns the number of key-value mappings in this map.  If the
 * map contains more than <tt>Integer.MAX_VALUE</tt> elements, returns
 * <tt>Integer.MAX_VALUE</tt>.
 *
 * @return the number of key-value mappings in this map
 */
int size();

/**
 * Returns <tt>true</tt> if this map contains no key-value mappings.
 *
 * @return <tt>true</tt> if this map contains no key-value mappings
 */
boolean isEmpty();
```

## Contains

Map 中是否包含特定的 key 或 value？

- `key==null ? k==null : key.equals(k)`
- `value==null ? v==null : value.equals(v)`

```java
/**
 * Returns <tt>true</tt> if this map contains a mapping for the specified
 * key.  More formally, returns <tt>true</tt> if and only if
 * this map contains a mapping for a key <tt>k</tt> such that
 * <tt>(key==null ? k==null : key.equals(k))</tt>.  (There can be
 * at most one such mapping.)
 *
 * @param key key whose presence in this map is to be tested
 * @return <tt>true</tt> if this map contains a mapping for the specified
 *         key
 * @throws ClassCastException if the key is of an inappropriate type for
 *         this map
 * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if the specified key is null and this map
 *         does not permit null keys
 * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 */
boolean containsKey(Object key);
```

```java
/**
 * Returns <tt>true</tt> if this map maps one or more keys to the
 * specified value.  More formally, returns <tt>true</tt> if and only if
 * this map contains at least one mapping to a value <tt>v</tt> such that
 * <tt>(value==null ? v==null : value.equals(v))</tt>.  This operation
 * will probably require time linear in the map size for most
 * implementations of the <tt>Map</tt> interface.
 *
 * @param value value whose presence in this map is to be tested
 * @return <tt>true</tt> if this map maps one or more keys to the
 *         specified value
 * @throws ClassCastException if the value is of an inappropriate type for
 *         this map
 * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if the specified value is null and this
 *         map does not permit null values
 * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 */
boolean containsValue(Object value);
```

## Get

取指定 key 对应的 value，如果 key 不存在，则返回 `null`。但有些集合实现允许 value 的值为空，所以返回 `null` 不一定意味着 key 不存在。

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
* <p>If this map permits null values, then a return value of
* {@code null} does not <i>necessarily</i> indicate that the map
* contains no mapping for the key; it's also possible that the map
* explicitly maps the key to {@code null}.  The {@link #containsKey
* containsKey} operation may be used to distinguish these two cases.
*
* @param key the key whose associated value is to be returned
* @return the value to which the specified key is mapped, or
*         {@code null} if this map contains no mapping for the key
* @throws ClassCastException if the key is of an inappropriate type for
*         this map
* (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
* @throws NullPointerException if the specified key is null and this map
*         does not permit null keys
* (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
*/
V get(Object key);
```

## Put

将一个 key 与 value 相关联，并放入 Map：

- 如果 map 之前已经存在该 key 的关联，则替代并返回旧的 value
- 如果没有，则返回 `null`

```java
/**
 * Associates the specified value with the specified key in this map
 * (optional operation).  If the map previously contained a mapping for
 * the key, the old value is replaced by the specified value.  (A map
 * <tt>m</tt> is said to contain a mapping for a key <tt>k</tt> if and only
 * if {@link #containsKey(Object) m.containsKey(k)} would return
 * <tt>true</tt>.)
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 *         (A <tt>null</tt> return can also indicate that the map
 *         previously associated <tt>null</tt> with <tt>key</tt>,
 *         if the implementation supports <tt>null</tt> values.)
 * @throws UnsupportedOperationException if the <tt>put</tt> operation
 *         is not supported by this map
 * @throws ClassCastException if the class of the specified key or value
 *         prevents it from being stored in this map
 * @throws NullPointerException if the specified key or value is null
 *         and this map does not permit null keys or values
 * @throws IllegalArgumentException if some property of the specified key
 *         or value prevents it from being stored in this map
 */
V put(K key, V value);
```

## Remove

移除指定的 key 对应的 value 并返回。如果没有 key 对应的 value，则返回 `null`。

```java
/**
 * Removes the mapping for a key from this map if it is present
 * (optional operation).   More formally, if this map contains a mapping
 * from key <tt>k</tt> to value <tt>v</tt> such that
 * <code>(key==null ?  k==null : key.equals(k))</code>, that mapping
 * is removed.  (The map can contain at most one such mapping.)
 *
 * <p>Returns the value to which this map previously associated the key,
 * or <tt>null</tt> if the map contained no mapping for the key.
 *
 * <p>If this map permits null values, then a return value of
 * <tt>null</tt> does not <i>necessarily</i> indicate that the map
 * contained no mapping for the key; it's also possible that the map
 * explicitly mapped the key to <tt>null</tt>.
 *
 * <p>The map will not contain a mapping for the specified key once the
 * call returns.
 *
 * @param key key whose mapping is to be removed from the map
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
 * @throws UnsupportedOperationException if the <tt>remove</tt> operation
 *         is not supported by this map
 * @throws ClassCastException if the key is of an inappropriate type for
 *         this map
 * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if the specified key is null and this
 *         map does not permit null keys
 * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 */
V remove(Object key);
```

## 集合视角

这个集合会在 Map 内部被维护，在迭代该集合的过程中对 Map 进行修改，行为未知。除非用迭代器自己的 `remove()` 函数。

```java
/**
 * Returns a {@link Set} view of the keys contained in this map.
 * The set is backed by the map, so changes to the map are
 * reflected in the set, and vice-versa.  If the map is modified
 * while an iteration over the set is in progress (except through
 * the iterator's own <tt>remove</tt> operation), the results of
 * the iteration are undefined.  The set supports element removal,
 * which removes the corresponding mapping from the map, via the
 * <tt>Iterator.remove</tt>, <tt>Set.remove</tt>,
 * <tt>removeAll</tt>, <tt>retainAll</tt>, and <tt>clear</tt>
 * operations.  It does not support the <tt>add</tt> or <tt>addAll</tt>
 * operations.
 *
 * @return a set view of the keys contained in this map
 */
Set<K> keySet();
```

同上，这个集合也会在 Map 中被维护。

```java
/**
 * Returns a {@link Collection} view of the values contained in this map.
 * The collection is backed by the map, so changes to the map are
 * reflected in the collection, and vice-versa.  If the map is
 * modified while an iteration over the collection is in progress
 * (except through the iterator's own <tt>remove</tt> operation),
 * the results of the iteration are undefined.  The collection
 * supports element removal, which removes the corresponding
 * mapping from the map, via the <tt>Iterator.remove</tt>,
 * <tt>Collection.remove</tt>, <tt>removeAll</tt>,
 * <tt>retainAll</tt> and <tt>clear</tt> operations.  It does not
 * support the <tt>add</tt> or <tt>addAll</tt> operations.
 *
 * @return a collection view of the values contained in this map
 */
Collection<V> values();
```

这个集合也会被内部维护？

```java
/**
 * Returns a {@link Set} view of the mappings contained in this map.
 * The set is backed by the map, so changes to the map are
 * reflected in the set, and vice-versa.  If the map is modified
 * while an iteration over the set is in progress (except through
 * the iterator's own <tt>remove</tt> operation, or through the
 * <tt>setValue</tt> operation on a map entry returned by the
 * iterator) the results of the iteration are undefined.  The set
 * supports element removal, which removes the corresponding
 * mapping from the map, via the <tt>Iterator.remove</tt>,
 * <tt>Set.remove</tt>, <tt>removeAll</tt>, <tt>retainAll</tt> and
 * <tt>clear</tt> operations.  It does not support the
 * <tt>add</tt> or <tt>addAll</tt> operations.
 *
 * @return a set view of the mappings contained in this map
 */
Set<Map.Entry<K, V>> entrySet();
```

## Entry

内部定义的子接口 `Entry<K,V>`。可以从 Entry 中提取 key 和 value，也支持设置某个 key 对应的 value。

```java
/**
 * A map entry (key-value pair).  The <tt>Map.entrySet</tt> method returns
 * a collection-view of the map, whose elements are of this class.  The
 * <i>only</i> way to obtain a reference to a map entry is from the
 * iterator of this collection-view.  These <tt>Map.Entry</tt> objects are
 * valid <i>only</i> for the duration of the iteration; more formally,
 * the behavior of a map entry is undefined if the backing map has been
 * modified after the entry was returned by the iterator, except through
 * the <tt>setValue</tt> operation on the map entry.
 *
 * @see Map#entrySet()
 * @since 1.2
 */
interface Entry<K,V> {
    /**
     * Returns the key corresponding to this entry.
     *
     * @return the key corresponding to this entry
     * @throws IllegalStateException implementations may, but are not
     *         required to, throw this exception if the entry has been
     *         removed from the backing map.
     */
    K getKey();

    /**
     * Returns the value corresponding to this entry.  If the mapping
     * has been removed from the backing map (by the iterator's
     * <tt>remove</tt> operation), the results of this call are undefined.
     *
     * @return the value corresponding to this entry
     * @throws IllegalStateException implementations may, but are not
     *         required to, throw this exception if the entry has been
     *         removed from the backing map.
     */
    V getValue();

    /**
     * Replaces the value corresponding to this entry with the specified
     * value (optional operation).  (Writes through to the map.)  The
     * behavior of this call is undefined if the mapping has already been
     * removed from the map (by the iterator's <tt>remove</tt> operation).
     *
     * @param value new value to be stored in this entry
     * @return old value corresponding to the entry
     * @throws UnsupportedOperationException if the <tt>put</tt> operation
     *         is not supported by the backing map
     * @throws ClassCastException if the class of the specified value
     *         prevents it from being stored in the backing map
     * @throws NullPointerException if the backing map does not permit
     *         null values, and the specified value is null
     * @throws IllegalArgumentException if some property of this value
     *         prevents it from being stored in the backing map
     * @throws IllegalStateException implementations may, but are not
     *         required to, throw this exception if the entry has been
     *         removed from the backing map.
     */
    V setValue(V value);

    /**
     * Compares the specified object with this entry for equality.
     * Returns <tt>true</tt> if the given object is also a map entry and
     * the two entries represent the same mapping.  More formally, two
     * entries <tt>e1</tt> and <tt>e2</tt> represent the same mapping
     * if<pre>
     *     (e1.getKey()==null ?
     *      e2.getKey()==null : e1.getKey().equals(e2.getKey()))  &amp;&amp;
     *     (e1.getValue()==null ?
     *      e2.getValue()==null : e1.getValue().equals(e2.getValue()))
     * </pre>
     * This ensures that the <tt>equals</tt> method works properly across
     * different implementations of the <tt>Map.Entry</tt> interface.
     *
     * @param o object to be compared for equality with this map entry
     * @return <tt>true</tt> if the specified object is equal to this map
     *         entry
     */
    boolean equals(Object o);

    /**
     * Returns the hash code value for this map entry.  The hash code
     * of a map entry <tt>e</tt> is defined to be: <pre>
     *     (e.getKey()==null   ? 0 : e.getKey().hashCode()) ^
     *     (e.getValue()==null ? 0 : e.getValue().hashCode())
     * </pre>
     * This ensures that <tt>e1.equals(e2)</tt> implies that
     * <tt>e1.hashCode()==e2.hashCode()</tt> for any two Entries
     * <tt>e1</tt> and <tt>e2</tt>, as required by the general
     * contract of <tt>Object.hashCode</tt>.
     *
     * @return the hash code value for this map entry
     * @see Object#hashCode()
     * @see Object#equals(Object)
     * @see #equals(Object)
     */
    int hashCode();

    /**
     * Returns a comparator that compares {@link Map.Entry} in natural order on key.
     *
     * <p>The returned comparator is serializable and throws {@link
     * NullPointerException} when comparing an entry with a null key.
     *
     * @param  <K> the {@link Comparable} type of then map keys
     * @param  <V> the type of the map values
     * @return a comparator that compares {@link Map.Entry} in natural order on key.
     * @see Comparable
     * @since 1.8
     */
    public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> c1.getKey().compareTo(c2.getKey());
    }

    /**
     * Returns a comparator that compares {@link Map.Entry} in natural order on value.
     *
     * <p>The returned comparator is serializable and throws {@link
     * NullPointerException} when comparing an entry with null values.
     *
     * @param <K> the type of the map keys
     * @param <V> the {@link Comparable} type of the map values
     * @return a comparator that compares {@link Map.Entry} in natural order on value.
     * @see Comparable
     * @since 1.8
     */
    public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> c1.getValue().compareTo(c2.getValue());
    }

    /**
     * Returns a comparator that compares {@link Map.Entry} by key using the given
     * {@link Comparator}.
     *
     * <p>The returned comparator is serializable if the specified comparator
     * is also serializable.
     *
     * @param  <K> the type of the map keys
     * @param  <V> the type of the map values
     * @param  cmp the key {@link Comparator}
     * @return a comparator that compares {@link Map.Entry} by the key.
     * @since 1.8
     */
    public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
        Objects.requireNonNull(cmp);
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
    }

    /**
     * Returns a comparator that compares {@link Map.Entry} by value using the given
     * {@link Comparator}.
     *
     * <p>The returned comparator is serializable if the specified comparator
     * is also serializable.
     *
     * @param  <K> the type of the map keys
     * @param  <V> the type of the map values
     * @param  cmp the value {@link Comparator}
     * @return a comparator that compares {@link Map.Entry} by the value.
     * @since 1.8
     */
    public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
        Objects.requireNonNull(cmp);
        return (Comparator<Map.Entry<K, V>> & Serializable)
            (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
    }
}
```

## Get or Default

返回 key 映射的 value。如果 key 的映射不存在，那么返回一个默认值。

```java
/**
 * Returns the value to which the specified key is mapped, or
 * {@code defaultValue} if this map contains no mapping for the key.
 *
 * @implSpec
 * The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties.
 *
 * @param key the key whose associated value is to be returned
 * @param defaultValue the default mapping of the key
 * @return the value to which the specified key is mapped, or
 * {@code defaultValue} if this map contains no mapping for the key
 * @throws ClassCastException if the key is of an inappropriate type for
 * this map
 * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if the specified key is null and this map
 * does not permit null keys
 * (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @since 1.8
 */
default V getOrDefault(Object key, V defaultValue) {
    V v;
    return (((v = get(key)) != null) || containsKey(key))
        ? v
        : defaultValue;
}
```

## For Each

以迭代器迭代 `entrySet()` 的顺序对 entry 进行 `action` 操作，直到迭代完毕或抛出异常。不保证同步性。

```java
/**
 * Performs the given action for each entry in this map until all entries
 * have been processed or the action throws an exception.   Unless
 * otherwise specified by the implementing class, actions are performed in
 * the order of entry set iteration (if an iteration order is specified.)
 * Exceptions thrown by the action are relayed to the caller.
 *
 * @implSpec
 * The default implementation is equivalent to, for this {@code map}:
 * <pre> {@code
 * for (Map.Entry<K, V> entry : map.entrySet())
 *     action.accept(entry.getKey(), entry.getValue());
 * }</pre>
 *
 * The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties.
 *
 * @param action The action to be performed for each entry
 * @throws NullPointerException if the specified action is null
 * @throws ConcurrentModificationException if an entry is found to be
 * removed during iteration
 * @since 1.8
 */
default void forEach(BiConsumer<? super K, ? super V> action) {
    Objects.requireNonNull(action);
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }
        action.accept(k, v);
    }
}
```

## Replace All

根据 `entrySet()` 的遍历顺序，对每个 value 应用指定的函数后，将新的 value 代替旧的 value。

```java
/**
 * Replaces each entry's value with the result of invoking the given
 * function on that entry until all entries have been processed or the
 * function throws an exception.  Exceptions thrown by the function are
 * relayed to the caller.
 *
 * @implSpec
 * <p>The default implementation is equivalent to, for this {@code map}:
 * <pre> {@code
 * for (Map.Entry<K, V> entry : map.entrySet())
 *     entry.setValue(function.apply(entry.getKey(), entry.getValue()));
 * }</pre>
 *
 * <p>The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties.
 *
 * @param function the function to apply to each entry
 * @throws UnsupportedOperationException if the {@code set} operation
 * is not supported by this map's entry set iterator.
 * @throws ClassCastException if the class of a replacement value
 * prevents it from being stored in this map
 * @throws NullPointerException if the specified function is null, or the
 * specified replacement value is null, and this map does not permit null
 * values
 * @throws ClassCastException if a replacement value is of an inappropriate
 *         type for this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if function or a replacement value is null,
 *         and this map does not permit null keys or values
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws IllegalArgumentException if some property of a replacement value
 *         prevents it from being stored in this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws ConcurrentModificationException if an entry is found to be
 * removed during iteration
 * @since 1.8
 */
default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
    Objects.requireNonNull(function);
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }

        // ise thrown from function is not a cme.
        v = function.apply(k, v);

        try {
            entry.setValue(v);
        } catch(IllegalStateException ise) {
            // this usually means the entry is no longer in the map.
            throw new ConcurrentModificationException(ise);
        }
    }
}
```

## Put if Absent

如果指定的 key 在映射中不存在，或 key 存在但是对应的 value 为 `null`，就用新的 value 覆盖并返回 `null`；否则返回原有的非 `null` 值。

```java
/**
 * If the specified key is not already associated with a value (or is mapped
 * to {@code null}) associates it with the given value and returns
 * {@code null}, else returns the current value.
 *
 * @implSpec
 * The default implementation is equivalent to, for this {@code
 * map}:
 *
 * <pre> {@code
 * V v = map.get(key);
 * if (v == null)
 *     v = map.put(key, value);
 *
 * return v;
 * }</pre>
 *
 * <p>The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with the specified key, or
 *         {@code null} if there was no mapping for the key.
 *         (A {@code null} return can also indicate that the map
 *         previously associated {@code null} with the key,
 *         if the implementation supports null values.)
 * @throws UnsupportedOperationException if the {@code put} operation
 *         is not supported by this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws ClassCastException if the key or value is of an inappropriate
 *         type for this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if the specified key or value is null,
 *         and this map does not permit null keys or values
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws IllegalArgumentException if some property of the specified key
 *         or value prevents it from being stored in this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @since 1.8
 */
default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {
        v = put(key, value);
    }

    return v;
}
```

## Remove

当且仅当指定的 key 对应于指定的 value 时，删除 entry。

```java
/**
 * Removes the entry for the specified key only if it is currently
 * mapped to the specified value.
 *
 * @implSpec
 * The default implementation is equivalent to, for this {@code map}:
 *
 * <pre> {@code
 * if (map.containsKey(key) && Objects.equals(map.get(key), value)) {
 *     map.remove(key);
 *     return true;
 * } else
 *     return false;
 * }</pre>
 *
 * <p>The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties.
 *
 * @param key key with which the specified value is associated
 * @param value value expected to be associated with the specified key
 * @return {@code true} if the value was removed
 * @throws UnsupportedOperationException if the {@code remove} operation
 *         is not supported by this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws ClassCastException if the key or value is of an inappropriate
 *         type for this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if the specified key or value is null,
 *         and this map does not permit null keys or values
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @since 1.8
 */
default boolean remove(Object key, Object value) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, value) ||
        (curValue == null && !containsKey(key))) {
        return false;
    }
    remove(key);
    return true;
}
```

## Replace

当且仅当指定的 key 映射到指定的 value (`oldValue`) 时，将 `oldValue` 替换为 `newValue`。

```java
/**
 * Replaces the entry for the specified key only if currently
 * mapped to the specified value.
 *
 * @implSpec
 * The default implementation is equivalent to, for this {@code map}:
 *
 * <pre> {@code
 * if (map.containsKey(key) && Objects.equals(map.get(key), value)) {
 *     map.put(key, newValue);
 *     return true;
 * } else
 *     return false;
 * }</pre>
 *
 * The default implementation does not throw NullPointerException
 * for maps that do not support null values if oldValue is null unless
 * newValue is also null.
 *
 * <p>The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties.
 *
 * @param key key with which the specified value is associated
 * @param oldValue value expected to be associated with the specified key
 * @param newValue value to be associated with the specified key
 * @return {@code true} if the value was replaced
 * @throws UnsupportedOperationException if the {@code put} operation
 *         is not supported by this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws ClassCastException if the class of a specified key or value
 *         prevents it from being stored in this map
 * @throws NullPointerException if a specified key or newValue is null,
 *         and this map does not permit null keys or values
 * @throws NullPointerException if oldValue is null and this map does not
 *         permit null values
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws IllegalArgumentException if some property of a specified key
 *         or value prevents it from being stored in this map
 * @since 1.8
 */
default boolean replace(K key, V oldValue, V newValue) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, oldValue) ||
        (curValue == null && !containsKey(key))) {
        return false;
    }
    put(key, newValue);
    return true;
}
```

当且仅当指定的 key 存在映射时，用 `value` 替换旧的 value：

```java
/**
 * Replaces the entry for the specified key only if it is
 * currently mapped to some value.
 *
 * @implSpec
 * The default implementation is equivalent to, for this {@code map}:
 *
 * <pre> {@code
 * if (map.containsKey(key)) {
 *     return map.put(key, value);
 * } else
 *     return null;
 * }</pre>
 *
 * <p>The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties.
 *
 * @param key key with which the specified value is associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with the specified key, or
 *         {@code null} if there was no mapping for the key.
 *         (A {@code null} return can also indicate that the map
 *         previously associated {@code null} with the key,
 *         if the implementation supports null values.)
 * @throws UnsupportedOperationException if the {@code put} operation
 *         is not supported by this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws ClassCastException if the class of the specified key or value
 *         prevents it from being stored in this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if the specified key or value is null,
 *         and this map does not permit null keys or values
 * @throws IllegalArgumentException if some property of the specified key
 *         or value prevents it from being stored in this map
 * @since 1.8
 */
default V replace(K key, V value) {
    V curValue;
    if (((curValue = get(key)) != null) || containsKey(key)) {
        curValue = put(key, value);
    }
    return curValue;
}
```

## Compute

如果 key 的映射不存在，或对应的 value 为 `null`，就利用 key 和 `mappingFunction` 计算一个非空的新 value，并加入映射。

```java
/**
 * If the specified key is not already associated with a value (or is mapped
 * to {@code null}), attempts to compute its value using the given mapping
 * function and enters it into this map unless {@code null}.
 *
 * <p>If the function returns {@code null} no mapping is recorded. If
 * the function itself throws an (unchecked) exception, the
 * exception is rethrown, and no mapping is recorded.  The most
 * common usage is to construct a new object serving as an initial
 * mapped value or memoized result, as in:
 *
 * <pre> {@code
 * map.computeIfAbsent(key, k -> new Value(f(k)));
 * }</pre>
 *
 * <p>Or to implement a multi-value map, {@code Map<K,Collection<V>>},
 * supporting multiple values per key:
 *
 * <pre> {@code
 * map.computeIfAbsent(key, k -> new HashSet<V>()).add(v);
 * }</pre>
 *
 *
 * @implSpec
 * The default implementation is equivalent to the following steps for this
 * {@code map}, then returning the current value or {@code null} if now
 * absent:
 *
 * <pre> {@code
 * if (map.get(key) == null) {
 *     V newValue = mappingFunction.apply(key);
 *     if (newValue != null)
 *         map.put(key, newValue);
 * }
 * }</pre>
 *
 * <p>The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties. In particular, all implementations of
 * subinterface {@link java.util.concurrent.ConcurrentMap} must document
 * whether the function is applied once atomically only if the value is not
 * present.
 *
 * @param key key with which the specified value is to be associated
 * @param mappingFunction the function to compute a value
 * @return the current (existing or computed) value associated with
 *         the specified key, or null if the computed value is null
 * @throws NullPointerException if the specified key is null and
 *         this map does not support null keys, or the mappingFunction
 *         is null
 * @throws UnsupportedOperationException if the {@code put} operation
 *         is not supported by this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws ClassCastException if the class of the specified key or value
 *         prevents it from being stored in this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @since 1.8
 */
default V computeIfAbsent(K key,
        Function<? super K, ? extends V> mappingFunction) {
    Objects.requireNonNull(mappingFunction);
    V v;
    if ((v = get(key)) == null) {
        V newValue;
        if ((newValue = mappingFunction.apply(key)) != null) {
            put(key, newValue);
            return newValue;
        }
    }

    return v;
}
```

针对指定的 key，Map 中已经存在非空映射，利用 `remappingFunction` 对 key 和 oldValue 进行计算。如果计算出的新 value 非空，则替代旧 value；如果为空，就将现有的映射删除。

```java
/**
 * If the value for the specified key is present and non-null, attempts to
 * compute a new mapping given the key and its current mapped value.
 *
 * <p>If the function returns {@code null}, the mapping is removed.  If the
 * function itself throws an (unchecked) exception, the exception is
 * rethrown, and the current mapping is left unchanged.
 *
 * @implSpec
 * The default implementation is equivalent to performing the following
 * steps for this {@code map}, then returning the current value or
 * {@code null} if now absent:
 *
 * <pre> {@code
 * if (map.get(key) != null) {
 *     V oldValue = map.get(key);
 *     V newValue = remappingFunction.apply(key, oldValue);
 *     if (newValue != null)
 *         map.put(key, newValue);
 *     else
 *         map.remove(key);
 * }
 * }</pre>
 *
 * <p>The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties. In particular, all implementations of
 * subinterface {@link java.util.concurrent.ConcurrentMap} must document
 * whether the function is applied once atomically only if the value is not
 * present.
 *
 * @param key key with which the specified value is to be associated
 * @param remappingFunction the function to compute a value
 * @return the new value associated with the specified key, or null if none
 * @throws NullPointerException if the specified key is null and
 *         this map does not support null keys, or the
 *         remappingFunction is null
 * @throws UnsupportedOperationException if the {@code put} operation
 *         is not supported by this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws ClassCastException if the class of the specified key or value
 *         prevents it from being stored in this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @since 1.8
 */
default V computeIfPresent(K key,
        BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue;
    if ((oldValue = get(key)) != null) {
        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue != null) {
            put(key, newValue);
            return newValue;
        } else {
            remove(key);
            return null;
        }
    } else {
        return null;
    }
}
```

对每一个指定的 key 及其旧的 value（或不存在 key 的映射而为 `null`），利用 `remappingFunction` 计算新 value。如果新 value 不为 `null`，就将其替代旧 value 或 `null`；否则就删除原有 key 的映射。

```java
/**
 * Attempts to compute a mapping for the specified key and its current
 * mapped value (or {@code null} if there is no current mapping). For
 * example, to either create or append a {@code String} msg to a value
 * mapping:
 *
 * <pre> {@code
 * map.compute(key, (k, v) -> (v == null) ? msg : v.concat(msg))}</pre>
 * (Method {@link #merge merge()} is often simpler to use for such purposes.)
 *
 * <p>If the function returns {@code null}, the mapping is removed (or
 * remains absent if initially absent).  If the function itself throws an
 * (unchecked) exception, the exception is rethrown, and the current mapping
 * is left unchanged.
 *
 * @implSpec
 * The default implementation is equivalent to performing the following
 * steps for this {@code map}, then returning the current value or
 * {@code null} if absent:
 *
 * <pre> {@code
 * V oldValue = map.get(key);
 * V newValue = remappingFunction.apply(key, oldValue);
 * if (oldValue != null ) {
 *    if (newValue != null)
 *       map.put(key, newValue);
 *    else
 *       map.remove(key);
 * } else {
 *    if (newValue != null)
 *       map.put(key, newValue);
 *    else
 *       return null;
 * }
 * }</pre>
 *
 * <p>The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties. In particular, all implementations of
 * subinterface {@link java.util.concurrent.ConcurrentMap} must document
 * whether the function is applied once atomically only if the value is not
 * present.
 *
 * @param key key with which the specified value is to be associated
 * @param remappingFunction the function to compute a value
 * @return the new value associated with the specified key, or null if none
 * @throws NullPointerException if the specified key is null and
 *         this map does not support null keys, or the
 *         remappingFunction is null
 * @throws UnsupportedOperationException if the {@code put} operation
 *         is not supported by this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws ClassCastException if the class of the specified key or value
 *         prevents it from being stored in this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @since 1.8
 */
default V compute(K key,
        BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue = get(key);

    V newValue = remappingFunction.apply(key, oldValue);
    if (newValue == null) {
        // delete mapping
        if (oldValue != null || containsKey(key)) {
            // something to remove
            remove(key);
            return null;
        } else {
            // nothing to do. Leave things as they were.
            return null;
        }
    } else {
        // add or replace old mapping
        put(key, newValue);
        return newValue;
    }
}
```

## Merge

对于每一个指定的 key，如果映射不存在，或映射到的 value 为 `null`，就映射一个非空的默认值；对于映射 value 非空的 key，利用 `remappingFunction` 计算新 value。

- 如果新 value 为空，就删除映射
- 如果新 value 不为空，就替代旧 value

```java
/**
 * If the specified key is not already associated with a value or is
 * associated with null, associates it with the given non-null value.
 * Otherwise, replaces the associated value with the results of the given
 * remapping function, or removes if the result is {@code null}. This
 * method may be of use when combining multiple mapped values for a key.
 * For example, to either create or append a {@code String msg} to a
 * value mapping:
 *
 * <pre> {@code
 * map.merge(key, msg, String::concat)
 * }</pre>
 *
 * <p>If the function returns {@code null} the mapping is removed.  If the
 * function itself throws an (unchecked) exception, the exception is
 * rethrown, and the current mapping is left unchanged.
 *
 * @implSpec
 * The default implementation is equivalent to performing the following
 * steps for this {@code map}, then returning the current value or
 * {@code null} if absent:
 *
 * <pre> {@code
 * V oldValue = map.get(key);
 * V newValue = (oldValue == null) ? value :
 *              remappingFunction.apply(oldValue, value);
 * if (newValue == null)
 *     map.remove(key);
 * else
 *     map.put(key, newValue);
 * }</pre>
 *
 * <p>The default implementation makes no guarantees about synchronization
 * or atomicity properties of this method. Any implementation providing
 * atomicity guarantees must override this method and document its
 * concurrency properties. In particular, all implementations of
 * subinterface {@link java.util.concurrent.ConcurrentMap} must document
 * whether the function is applied once atomically only if the value is not
 * present.
 *
 * @param key key with which the resulting value is to be associated
 * @param value the non-null value to be merged with the existing value
 *        associated with the key or, if no existing value or a null value
 *        is associated with the key, to be associated with the key
 * @param remappingFunction the function to recompute a value if present
 * @return the new value associated with the specified key, or null if no
 *         value is associated with the key
 * @throws UnsupportedOperationException if the {@code put} operation
 *         is not supported by this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws ClassCastException if the class of the specified key or value
 *         prevents it from being stored in this map
 *         (<a href="{@docRoot}/java/util/Collection.html#optional-restrictions">optional</a>)
 * @throws NullPointerException if the specified key is null and this map
 *         does not support null keys or the value or remappingFunction is
 *         null
 * @since 1.8
 */
default V merge(K key, V value,
        BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
                remappingFunction.apply(oldValue, value);
    if(newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    }
    return newValue;
}
```
