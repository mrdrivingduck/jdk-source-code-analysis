# Interface - java.util.NavigableMap

Created by : Mr Dk.

2019 / 11 / 13 14:29

Nanjing, Jiangsu, China

---

## Definition

```java
public interface NavigableMap<K,V> extends SortedMap<K,V> {

}
```

继承自 `SortedMap`，能够快速返回与给定目标最近的匹配。

* `lowEntry()` - 返回小于给定的 key 的 entry
* `floorEntry()` - 返回小于等于给定 key 的 entry
* `ceilingEntry()` - 返回大于等于给定 key 的 entry
* `higherEntry()` - 返回大于给定 key 的 entry

如果没有匹配的 key，就返回 `null`。四个函数还分别有相对应的只返回 key 的版本：这些操作都用于定位，而不是遍历。

`descendingMap()` 返回一个视角相反的 Map，性能上比正视角的 Map 要慢。

`subMap()`, `headMap()`, `tailMap()` 与 `SortedMap` 中的函数不同：指定了额外的参数，来表示包含或是不包含边界。此外，还提供了从两端删除最大或最小的 entry 的函数。

```java
/**
 * A {@link SortedMap} extended with navigation methods returning the
 * closest matches for given search targets. Methods
 * {@code lowerEntry}, {@code floorEntry}, {@code ceilingEntry},
 * and {@code higherEntry} return {@code Map.Entry} objects
 * associated with keys respectively less than, less than or equal,
 * greater than or equal, and greater than a given key, returning
 * {@code null} if there is no such key.  Similarly, methods
 * {@code lowerKey}, {@code floorKey}, {@code ceilingKey}, and
 * {@code higherKey} return only the associated keys. All of these
 * methods are designed for locating, not traversing entries.
 *
 * <p>A {@code NavigableMap} may be accessed and traversed in either
 * ascending or descending key order.  The {@code descendingMap}
 * method returns a view of the map with the senses of all relational
 * and directional methods inverted. The performance of ascending
 * operations and views is likely to be faster than that of descending
 * ones.  Methods {@code subMap}, {@code headMap},
 * and {@code tailMap} differ from the like-named {@code
 * SortedMap} methods in accepting additional arguments describing
 * whether lower and upper bounds are inclusive versus exclusive.
 * Submaps of any {@code NavigableMap} must implement the {@code
 * NavigableMap} interface.
 *
 * <p>This interface additionally defines methods {@code firstEntry},
 * {@code pollFirstEntry}, {@code lastEntry}, and
 * {@code pollLastEntry} that return and/or remove the least and
 * greatest mappings, if any exist, else returning {@code null}.
 *
 * <p>Implementations of entry-returning methods are expected to
 * return {@code Map.Entry} pairs representing snapshots of mappings
 * at the time they were produced, and thus generally do <em>not</em>
 * support the optional {@code Entry.setValue} method. Note however
 * that it is possible to change mappings in the associated map using
 * method {@code put}.
 *
 * <p>Methods
 * {@link #subMap(Object, Object) subMap(K, K)},
 * {@link #headMap(Object) headMap(K)}, and
 * {@link #tailMap(Object) tailMap(K)}
 * are specified to return {@code SortedMap} to allow existing
 * implementations of {@code SortedMap} to be compatibly retrofitted to
 * implement {@code NavigableMap}, but extensions and implementations
 * of this interface are encouraged to override these methods to return
 * {@code NavigableMap}.  Similarly,
 * {@link #keySet()} can be overriden to return {@code NavigableSet}.
 *
 * <p>This interface is a member of the
 * <a href="{@docRoot}/../technotes/guides/collections/index.html">
 * Java Collections Framework</a>.
 *
 * @author Doug Lea
 * @author Josh Bloch
 * @param <K> the type of keys maintained by this map
 * @param <V> the type of mapped values
 * @since 1.6
 */
```

---

```java
/**
 * Returns a key-value mapping associated with the greatest key
 * strictly less than the given key, or {@code null} if there is
 * no such key.
 *
 * @param key the key
 * @return an entry with the greatest key less than {@code key},
 *         or {@code null} if there is no such key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map does not permit null keys
 */
Map.Entry<K,V> lowerEntry(K key);

/**
 * Returns the greatest key strictly less than the given key, or
 * {@code null} if there is no such key.
 *
 * @param key the key
 * @return the greatest key less than {@code key},
 *         or {@code null} if there is no such key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map does not permit null keys
 */
K lowerKey(K key);
```

返回小于指定 key 的最大 entry 或 key。

---

```java
/**
 * Returns a key-value mapping associated with the greatest key
 * less than or equal to the given key, or {@code null} if there
 * is no such key.
 *
 * @param key the key
 * @return an entry with the greatest key less than or equal to
 *         {@code key}, or {@code null} if there is no such key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map does not permit null keys
 */
Map.Entry<K,V> floorEntry(K key);

/**
 * Returns the greatest key less than or equal to the given key,
 * or {@code null} if there is no such key.
 *
 * @param key the key
 * @return the greatest key less than or equal to {@code key},
 *         or {@code null} if there is no such key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map does not permit null keys
 */
K floorKey(K key);
```

返回小于等于指定 key 的最大 entry 或 key。

---

```java
/**
 * Returns a key-value mapping associated with the least key
 * greater than or equal to the given key, or {@code null} if
 * there is no such key.
 *
 * @param key the key
 * @return an entry with the least key greater than or equal to
 *         {@code key}, or {@code null} if there is no such key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map does not permit null keys
 */
Map.Entry<K,V> ceilingEntry(K key);

/**
 * Returns the least key greater than or equal to the given key,
 * or {@code null} if there is no such key.
 *
 * @param key the key
 * @return the least key greater than or equal to {@code key},
 *         or {@code null} if there is no such key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map does not permit null keys
 */
K ceilingKey(K key);
```

返回大于等于指定 key 的最小 entry 或 key。

---

```java
/**
 * Returns a key-value mapping associated with the least key
 * strictly greater than the given key, or {@code null} if there
 * is no such key.
 *
 * @param key the key
 * @return an entry with the least key greater than {@code key},
 *         or {@code null} if there is no such key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map does not permit null keys
 */
Map.Entry<K,V> higherEntry(K key);

/**
 * Returns the least key strictly greater than the given key, or
 * {@code null} if there is no such key.
 *
 * @param key the key
 * @return the least key greater than {@code key},
 *         or {@code null} if there is no such key
 * @throws ClassCastException if the specified key cannot be compared
 *         with the keys currently in the map
 * @throws NullPointerException if the specified key is null
 *         and this map does not permit null keys
 */
K higherKey(K key);
```

返回大于指定 key 的最小 entry 或 key。

---

```java
/**
 * Returns a key-value mapping associated with the least
 * key in this map, or {@code null} if the map is empty.
 *
 * @return an entry with the least key,
 *         or {@code null} if this map is empty
 */
Map.Entry<K,V> firstEntry();

/**
 * Returns a key-value mapping associated with the greatest
 * key in this map, or {@code null} if the map is empty.
 *
 * @return an entry with the greatest key,
 *         or {@code null} if this map is empty
 */
Map.Entry<K,V> lastEntry();
```

返回 key 最小 / 最大的 entry。

---

```java
/**
 * Removes and returns a key-value mapping associated with
 * the least key in this map, or {@code null} if the map is empty.
 *
 * @return the removed first entry of this map,
 *         or {@code null} if this map is empty
 */
Map.Entry<K,V> pollFirstEntry();

/**
 * Removes and returns a key-value mapping associated with
 * the greatest key in this map, or {@code null} if the map is empty.
 *
 * @return the removed last entry of this map,
 *         or {@code null} if this map is empty
 */
Map.Entry<K,V> pollLastEntry();
```

移除 key 最大 / 最小的 entry。

---

```java
/**
 * Returns a reverse order view of the mappings contained in this map.
 * The descending map is backed by this map, so changes to the map are
 * reflected in the descending map, and vice-versa.  If either map is
 * modified while an iteration over a collection view of either map
 * is in progress (except through the iterator's own {@code remove}
 * operation), the results of the iteration are undefined.
 *
 * <p>The returned map has an ordering equivalent to
 * <tt>{@link Collections#reverseOrder(Comparator) Collections.reverseOrder}(comparator())</tt>.
 * The expression {@code m.descendingMap().descendingMap()} returns a
 * view of {@code m} essentially equivalent to {@code m}.
 *
 * @return a reverse order view of this map
 */
NavigableMap<K,V> descendingMap();
```

返回一个视角相反的 `NavigableMap`

* 但是这个返回的集合与原集合依旧互相作用
* 底层的副本应当是同一份
* 只是 `Comparator` 的逻辑取反了

---

```java
/**
 * Returns a {@link NavigableSet} view of the keys contained in this map.
 * The set's iterator returns the keys in ascending order.
 * The set is backed by the map, so changes to the map are reflected in
 * the set, and vice-versa.  If the map is modified while an iteration
 * over the set is in progress (except through the iterator's own {@code
 * remove} operation), the results of the iteration are undefined.  The
 * set supports element removal, which removes the corresponding mapping
 * from the map, via the {@code Iterator.remove}, {@code Set.remove},
 * {@code removeAll}, {@code retainAll}, and {@code clear} operations.
 * It does not support the {@code add} or {@code addAll} operations.
 *
 * @return a navigable set view of the keys in this map
 */
NavigableSet<K> navigableKeySet();
```

返回 Map 中包含的所有 key，迭代器以升序访问。这个集合被 Map 维护，所以对这个集合的操作将会在 Map 中体现，反之亦然。

---

```java
/**
 * Returns a reverse order {@link NavigableSet} view of the keys contained in this map.
 * The set's iterator returns the keys in descending order.
 * The set is backed by the map, so changes to the map are reflected in
 * the set, and vice-versa.  If the map is modified while an iteration
 * over the set is in progress (except through the iterator's own {@code
 * remove} operation), the results of the iteration are undefined.  The
 * set supports element removal, which removes the corresponding mapping
 * from the map, via the {@code Iterator.remove}, {@code Set.remove},
 * {@code removeAll}, {@code retainAll}, and {@code clear} operations.
 * It does not support the {@code add} or {@code addAll} operations.
 *
 * @return a reverse order navigable set view of the keys in this map
 */
NavigableSet<K> descendingKeySet();
```

返回一个视角相反的有序 key Set，性质与正序的 key Set 相同

---

```java
/**
 * Returns a view of the portion of this map whose keys range from
 * {@code fromKey} to {@code toKey}.  If {@code fromKey} and
 * {@code toKey} are equal, the returned map is empty unless
 * {@code fromInclusive} and {@code toInclusive} are both true.  The
 * returned map is backed by this map, so changes in the returned map are
 * reflected in this map, and vice-versa.  The returned map supports all
 * optional map operations that this map supports.
 *
 * <p>The returned map will throw an {@code IllegalArgumentException}
 * on an attempt to insert a key outside of its range, or to construct a
 * submap either of whose endpoints lie outside its range.
 *
 * @param fromKey low endpoint of the keys in the returned map
 * @param fromInclusive {@code true} if the low endpoint
 *        is to be included in the returned view
 * @param toKey high endpoint of the keys in the returned map
 * @param toInclusive {@code true} if the high endpoint
 *        is to be included in the returned view
 * @return a view of the portion of this map whose keys range from
 *         {@code fromKey} to {@code toKey}
 * @throws ClassCastException if {@code fromKey} and {@code toKey}
 *         cannot be compared to one another using this map's comparator
 *         (or, if the map has no comparator, using natural ordering).
 *         Implementations may, but are not required to, throw this
 *         exception if {@code fromKey} or {@code toKey}
 *         cannot be compared to keys currently in the map.
 * @throws NullPointerException if {@code fromKey} or {@code toKey}
 *         is null and this map does not permit null keys
 * @throws IllegalArgumentException if {@code fromKey} is greater than
 *         {@code toKey}; or if this map itself has a restricted
 *         range, and {@code fromKey} or {@code toKey} lies
 *         outside the bounds of the range
 */
NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                            K toKey,   boolean toInclusive);

/**
 * {@inheritDoc}
 *
 * <p>Equivalent to {@code subMap(fromKey, true, toKey, false)}.
 *
 * @throws ClassCastException       {@inheritDoc}
 * @throws NullPointerException     {@inheritDoc}
 * @throws IllegalArgumentException {@inheritDoc}
 */
SortedMap<K,V> subMap(K fromKey, K toKey);
```

取指定 key 范围内的子集，但可以根据参数确定是否包含边界。

---

```java
/**
 * Returns a view of the portion of this map whose keys are less than (or
 * equal to, if {@code inclusive} is true) {@code toKey}.  The returned
 * map is backed by this map, so changes in the returned map are reflected
 * in this map, and vice-versa.  The returned map supports all optional
 * map operations that this map supports.
 *
 * <p>The returned map will throw an {@code IllegalArgumentException}
 * on an attempt to insert a key outside its range.
 *
 * @param toKey high endpoint of the keys in the returned map
 * @param inclusive {@code true} if the high endpoint
 *        is to be included in the returned view
 * @return a view of the portion of this map whose keys are less than
 *         (or equal to, if {@code inclusive} is true) {@code toKey}
 * @throws ClassCastException if {@code toKey} is not compatible
 *         with this map's comparator (or, if the map has no comparator,
 *         if {@code toKey} does not implement {@link Comparable}).
 *         Implementations may, but are not required to, throw this
 *         exception if {@code toKey} cannot be compared to keys
 *         currently in the map.
 * @throws NullPointerException if {@code toKey} is null
 *         and this map does not permit null keys
 * @throws IllegalArgumentException if this map itself has a
 *         restricted range, and {@code toKey} lies outside the
 *         bounds of the range
 */
NavigableMap<K,V> headMap(K toKey, boolean inclusive);

/**
 * {@inheritDoc}
 *
 * <p>Equivalent to {@code headMap(toKey, false)}.
 *
 * @throws ClassCastException       {@inheritDoc}
 * @throws NullPointerException     {@inheritDoc}
 * @throws IllegalArgumentException {@inheritDoc}
 */
SortedMap<K,V> headMap(K toKey);
```

取比指定的 key 更小的子集，可以根据参数确定是否包含边界。

---

```java
/**
 * Returns a view of the portion of this map whose keys are greater than (or
 * equal to, if {@code inclusive} is true) {@code fromKey}.  The returned
 * map is backed by this map, so changes in the returned map are reflected
 * in this map, and vice-versa.  The returned map supports all optional
 * map operations that this map supports.
 *
 * <p>The returned map will throw an {@code IllegalArgumentException}
 * on an attempt to insert a key outside its range.
 *
 * @param fromKey low endpoint of the keys in the returned map
 * @param inclusive {@code true} if the low endpoint
 *        is to be included in the returned view
 * @return a view of the portion of this map whose keys are greater than
 *         (or equal to, if {@code inclusive} is true) {@code fromKey}
 * @throws ClassCastException if {@code fromKey} is not compatible
 *         with this map's comparator (or, if the map has no comparator,
 *         if {@code fromKey} does not implement {@link Comparable}).
 *         Implementations may, but are not required to, throw this
 *         exception if {@code fromKey} cannot be compared to keys
 *         currently in the map.
 * @throws NullPointerException if {@code fromKey} is null
 *         and this map does not permit null keys
 * @throws IllegalArgumentException if this map itself has a
 *         restricted range, and {@code fromKey} lies outside the
 *         bounds of the range
 */
NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);

/**
 * {@inheritDoc}
 *
 * <p>Equivalent to {@code tailMap(fromKey, true)}.
 *
 * @throws ClassCastException       {@inheritDoc}
 * @throws NullPointerException     {@inheritDoc}
 * @throws IllegalArgumentException {@inheritDoc}
 */
SortedMap<K,V> tailMap(K fromKey);
```

取比指定 key 更大的子集，可以根据参数确定是否包含边界。

---

